llm-d

1. 한 줄 요약

llm-d = Kubernetes 환경에서 여러 LLM inference worker를 지능적으로 라우팅·스케줄링하고, KV Cache / Prefill-Decode Disaggregation / Autoscaling 등을 활용해 대규모 LLM Serving을 최적화하는 오픈소스 프로젝트

기본적인 모델 실행은 vLLM/SGLang 같은 inference engine이 담당하고, llm-d는 그 위에서 **“어떤 요청을 어느 worker에서 처리할 것인가”**를 최적화하는 역할에 가깝다.

⸻

2. 왜 필요한가?

일반적인 vLLM Serving:

Client
  ↓
Load Balancer
  ↓
vLLM Pod
  ├─ Prefill
  └─ Decode

Pod가 많아지면 단순 Round-Robin만으로는 비효율적이다.

* 특정 Pod에 KV Cache가 이미 있는데 다른 Pod로 요청 → KV Cache 재계산
* 긴 Prompt의 Prefill이 Decode 요청을 방해
* GPU별 Load가 다른데 동일하게 분배
* 대형 MoE 모델에서 GPU 간 통신/병렬화 고려 필요

llm-d는 이런 문제를 LLM-aware routing/scheduling으로 해결하려는 계층이다. (GitHub)

⸻

3. 핵심 기능

① Intelligent Routing

단순 Round-Robin 대신

* KV Cache / Prefix Cache locality
* 현재 Load
* Queue 상태
* Model / Hardware 정보
* 예상 latency

등을 고려해 어느 inference worker로 보낼지 결정한다. (GitHub)

Request
   ↓
llm-d Router / EPP
   ↓
┌────────┬────────┬────────┐
│ vLLM 1 │ vLLM 2 │ vLLM 3 │
└────────┴────────┴────────┘
       ↑
 KV Cache / Load 등을 고려

⸻

② Prefill / Decode Disaggregation

LLM inference의

* Prefill → 주로 compute/FLOPs-bound
* Decode → 주로 memory-bandwidth-bound

특성을 이용해 둘을 서로 다른 worker로 분리한다. (GitHub)

                 ┌─ Prefill Worker
Request → Router ┤
                 └─ Decode Worker
                       ↑
                    KV Cache

예를 들어:

Prefill : TP=1 × 여러 Pod
Decode  : TP=5 × 적은 Pod

처럼 P와 D를 서로 다른 규모로 확장할 수 있다. (GitHub)

P에서 생성된 KV Cache는 NIXL + RDMA/IB/RoCE/EFA 등을 이용해 D로 전달한다.

⸻

③ KV Cache Management

단순히 각 vLLM Pod가 자기 KV Cache만 가지고 있는 구조에서 벗어나

* Prefix-aware routing
* CPU/NVMe 등으로 KV Cache offloading
* 다른 worker의 cache 활용

등을 지원한다. (GitHub)

즉,

“어떤 GPU가 현재 이 요청의 KV Cache를 가지고 있는가?”

까지 routing 판단에 포함시키는 것이 핵심이다.

⸻

④ Autoscaling / Flow Control

LLM 특유의

* 요청 길이
* TTFT
* ITL
* Queue
* GPU utilization

등을 고려해 일반적인 CPU/GPU utilization 기반 HPA보다 inference workload에 맞는 scaling을 수행하는 방향이다. (GitHub)

⸻

4. 전체 구조

대략 다음과 같이 이해하면 된다.

                 Client
                   │
                   ▼
            Gateway / Envoy
                   │
                   ▼
          ┌─────────────────┐
          │   llm-d Router   │
          │      (EPP)       │
          └────────┬────────┘
                   │
          Intelligent Routing
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
  Prefill Pool           Decode Pool
   (vLLM)                  (vLLM)
        │                     ▲
        └──── KV Cache ───────┘
              NIXL/RDMA

Kubernetes 관점에서는 Gateway API Inference Extension + EPP(Endpoint Picker) + model-server Pods를 조합하는 구조다. (GitHub)

⸻

5. vLLM과의 관계

중요한 부분:

vLLM
 └─ 실제 LLM inference 실행
      └─ GPU에서 Attention / GEMM / KV Cache 등 수행
llm-d
 └─ 여러 vLLM을 어떻게 운영할지 결정
      ├─ Routing
      ├─ Scheduling
      ├─ KV Cache-aware routing
      ├─ Prefill / Decode 분리
      └─ Autoscaling

따라서 llm-d가 vLLM을 대체하는 것은 아니다.

오히려

llm-d = distributed LLM serving orchestration/scheduling layer

라고 이해하는 게 가장 쉽다.

⸻

6. 핵심적으로 기억할 것

                    llm-d
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
 Intelligent      KV Cache       P/D
  Routing        Management    Disaggregation
       │              │              │
       └──────────────┼──────────────┘
                      ▼
             Efficient LLM Serving
                      │
                  vLLM/SGLang
                      │
                     GPU

한마디로:

vLLM이 “LLM을 어떻게 빠르게 실행할까”라면,
llm-d는 “수많은 LLM worker를 어떻게 배치·라우팅·스케줄링해야 전체 시스템이 빠를까”에 초점을 둔다.