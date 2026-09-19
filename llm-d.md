# llm-d

## 1. 한 줄 요약

**llm-d = Kubernetes 환경에서 여러 LLM inference worker를 효율적으로 운영하기 위한 LLM-aware serving/scheduling layer**

실제 LLM 연산은 **vLLM/SGLang** 등이 담당하고, llm-d는 그 위에서 **Routing, Scheduling, KV Cache 활용, Prefill/Decode 분리, Autoscaling** 등을 담당한다.

---

## 2. 왜 필요한가?

일반적인 vLLM Serving:

Client
  ↓
Load Balancer
  ↓
vLLM Pod
  ├─ Prefill
  └─ Decode

Pod가 많아지면 단순 Round-Robin 방식은 비효율적일 수 있다.

- KV Cache가 이미 있는 Pod를 활용하지 못함 → KV Cache 재계산
- 긴 Prompt의 Prefill이 Decode 요청에 영향을 줌
- Pod마다 Load가 다른데 동일하게 요청을 분배
- GPU/모델 병렬화 구조를 고려하지 못함

→ llm-d는 이런 문제를 **LLM workload를 고려한 routing/scheduling**으로 최적화한다.

---

## 3. 핵심 기능

### ① Intelligent Routing

단순 Round-Robin 대신 다음 정보를 고려하여 요청을 적절한 worker로 전달한다.

- KV Cache / Prefix Cache locality
- 현재 Load
- Queue 상태
- Model / Hardware 정보
- 예상 latency

```text
Request
   ↓
llm-d Router / EPP
   ↓
┌────────┬────────┬────────┐
│ vLLM 1 │ vLLM 2 │ vLLM 3 │
└────────┴────────┴────────┘
       ↑
 KV Cache / Load 등을 고려