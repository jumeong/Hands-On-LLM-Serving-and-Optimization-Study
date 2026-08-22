# NVIDIA H100 GPU 스펙 및 시스템 토폴로지 비교 분석 (H100 SXM vs H100 NVL)

NVIDIA Hopper 아키텍처(GH100) 기반의 **H100 SXM**과 **H100 NVL**의 세부 사양, 연산 성능, 토폴로지 구성 및 활용 목적별 차이점을 정리한 문서입니다.

---

## 1. 핵심 개요 및 포지셔닝

| 구분 | **NVIDIA H100 SXM (SXM5)** | **NVIDIA H100 NVL** |
| :--- | :--- | :--- |
| **주요 목적** | **초거대 AI 모델 학습(Training) & 대규모 HPC 클러스터** | **대규모 언어 모델(LLM) 실시간 추론(Inference) & 서빙** |
| **배포 인프라** | NVIDIA HGX / DGX H100 전용 베이스보드 (8-GPU / 4-GPU) | 범용 데이터센터 표준 PCIe 서버 랙 (Dual-Slot PCIe) |
| **핵심 강점** | 900 GB/s 풀 NVLink 대역폭, 최대 700W TDP 기반 최고 피크 연산력 | 94 GB 대용량 HBM3, 3.9 TB/s 초고속 메모리 대역폭, 전력 효율성 |

---

## 2. 하드웨어 세부 사양 비교 (Specifications)

| 사양 항목 (Specification) | **H100 SXM5** | **H100 NVL** |
| :--- | :--- | :--- |
| **아키텍처 (Architecture)** | NVIDIA Hopper (GH100) | NVIDIA Hopper (GH100) |
| **공정 (Fabrication)** | TSMC 4N (NVIDIA 맞춤형 5nm) | TSMC 4N (NVIDIA 맞춤형 5nm) |
| **폼 팩터 (Form Factor)** | SXM5 모듈 (소켓 장착형) | Dual-Slot PCIe Gen 5 (FHFL) |
| **스트리밍 멀티프로세서 (SM)** | **132개** | **132개** |
| **CUDA 코어 수** | **16,896개** | **16,896개** |
| **Tensor Core 수** | 528개 (4세대) | 528개 (4세대) |
| **GPU 메모리 용량** | **80 GB** | **94 GB** |
| **메모리 유형** | HBM3 | HBM3 |
| **메모리 구성** | 5개 HBM3 스택 활성화 | 6개 HBM3 스택 활성화 |
| **메모리 버스 폭** | 5,120-bit | 6,144-bit |
| **메모리 대역폭 (Bandwidth)** | **3.35 TB/s (3,350 GB/s)** | **3.9 TB/s (3,900 GB/s)** |
| **인터커넥트 대역폭 (NVLink)** | **900 GB/s** (4세대, 18 링크) | **600 GB/s** (3x NVLink Bridge) |
| **호스트 인터페이스** | PCIe Gen 5 (HGX 보드 연동) | PCIe Gen 5 x16 (128 GB/s) |
| **소비 전력 (Max TDP)** | **최대 700W** (설정 가능) | **350W ~ 400W** (설정 가능) |
| **냉각 방식** | 서버 섀시 공랭 / Direct-to-Chip 수랭 | 서버 섀시 공랭 (패시브) |
| **Multi-Instance GPU (MIG)** | 최대 7개 인스턴스 | 최대 7개 인스턴스 |

---

## 3. 연산 성능 비교 (Peak Throughput)

NVIDIA Hopper 아키텍처는 **Transformer Engine**과 **2:4 Structural Sparsity(희소성)**를 지원합니다. 아래 수치 중 `*` 표시는 Sparsity 활성화 기준 피크 성능입니다.

| 정밀도 / 연산 포맷 (Precision) | **H100 SXM** | **H100 NVL** |
| :--- | :--- | :--- |
| **FP64 (Standard)** | 34 TFLOPS | 30 TFLOPS |
| **FP64 Tensor Core** | 67 TFLOPS | 60 TFLOPS |
| **FP32 (Standard)** | 67 TFLOPS | 60 TFLOPS |
| **TF32 Tensor Core** | 495 TFLOPS / **989 TFLOPS\*** | 417.5 TFLOPS / **835 TFLOPS\*** |
| **BFLOAT16 Tensor Core** | 989.5 TFLOPS / **1,979 TFLOPS\*** | 835.5 TFLOPS / **1,671 TFLOPS\*** |
| **FP16 Tensor Core** | 989.5 TFLOPS / **1,979 TFLOPS\*** | 835.5 TFLOPS / **1,671 TFLOPS\*** |
| **FP8 Tensor Core** | 1,979 TFLOPS / **3,958 TFLOPS\*** | 1,670.5 TFLOPS / **3,341 TFLOPS\*** |
| **INT8 Tensor Core** | 1,979 TOPS / **3,958 TOPS\*** | 1,670.5 TOPS / **3,341 TOPS\*** |

---

## 4. 시스템 구성별 토폴로지(Topology) 비교

GPU 수량과 인터커넥트 방식(NVLink Bridge vs 직접 NVLink Full Mesh vs NVSwitch Fabric)에 따른 토폴로지 구조 및 통신 특성 비교입니다.

### 📊 구성별 토폴로지 요약 비교표

| 구분 | **1. H100 PCIe 2대** | **2. H100 NVL 4대** | **3. H100 SXM 8대 w/ NVLink** | **4. H100 SXM 8대 w/ NVLink + NVSwitch** |
| :--- | :--- | :--- | :--- | :--- |
| **토폴로지 형태** | **1:1 Point-to-Point (P2P)** | **Dual-Island (2+2 페어 구조)** | **Direct Full Mesh (스위치 없는 풀 메시)** | **Full Non-blocking Crossbar Fabric** |
| **인터커넥트 방식** | 3x NVLink Bridge | 3x NVLink Bridge (2쌍) + PCIe Switch | 18개 NVLink 포트를 7개 피어에 직접 1:1 분할 배선 | 4x 3세대 NVSwitch 전용 크로스바 스위치 칩 |
| **GPU 간 대역폭** | **600 GB/s** (P2P) | • 페어 내부: **600 GB/s**<br>• 페어 간: **128 GB/s (PCIe)** | • 개별 1:1 쌍 간: **~100~150 GB/s** (정적 분할)<br>• 7개 피어 동시 통신 총합: **900 GB/s** | **어떤 GPU 간이든 1:1 통신 시 900 GB/s 풀 대역폭 보장** |
| **통신 홉 (Hop) 수** | 단일 홉 (Direct) | • 내부: 1홉<br>• 외부: PCIe/CPU 경유 (2+ 홉) | **모든 GPU 간 1홉 (Direct P2P)** | **모든 GPU 간 1홉 (Full Any-to-Any)** |
| **통신 특성 및 병목** | 없음 (2-GPU 완벽 P2P) | 페어 간 통신 시 PCIe 버스 병목 (128 GB/s) | 모든 GPU가 직결되어 1홉이나, 1:1 집중 통신 시 대역폭 제한 | 시스템 내부 병목 없음 (총 3.6 TB/s 바이섹션 대역폭) |
| **권장 병렬화 전략** | • TP = 2<br>• 70B 모델 추론 | • 2x 독립 인스턴스 (각 TP=2)<br>• 또는 TP=2 + PP/DP=2 | • 전체 All-to-All 통신 시 900 GB/s 활용<br>• 비대칭/특정 GPU 간 대용량 전송에는 한계 | **• TP = 8 (초거대 LLM 8-way TP All-Reduce 최적)<br>• 대규모 분산 학습 & 고부하 서빙** |

---

### 🔍 구성별 상세 토폴로지 구조

#### 1. H100 PCIe 2대 (1:1 P2P NVLink Bridge)
```
+---------------+                      +---------------+
|   H100 PCIe   |<====================>|   H100 PCIe   |
|     (GPU 0)   |  NVLink Bridge 600GB/s|     (GPU 1)   |
+---------------+                      +---------------+
        |                                      |
        +------------- PCIe Gen 5 x16 ---------+
                      (Host / CPU)
```
* **구조**: 2장의 PCIe 카드가 3개의 물리적 NVLink Bridge로 직결.
* **특징**: 호스트 CPU/PCIe 버스를 거치지 않고 GPU 0과 GPU 1이 600 GB/s의 고속으로 직접 통신(P2P).
* **적용**: 2-GPU 텐서 병렬화(TP=2)를 통한 중간 규모 모델 서빙 및 미세조정.

---

#### 2. H100 NVL 4대 (2+2 Dual-Island 구조)
```
  [ NVLink Island 1 ]                     [ NVLink Island 2 ]
+---------+         +---------+         +---------+         +---------+
| H100 NVL|<=======>| H100 NVL|         | H100 NVL|<=======>| H100 NVL|
| (GPU 0) | 600GB/s | (GPU 1) |         | (GPU 2) | 600GB/s | (GPU 3) |
+---------+ NVLink  +---------+         +---------+ NVLink  +---------+
     |                   |                   |                   |
     +---------+---------+                   +---------+---------+
               |                                       |
               +========== PCIe Switch / Host =========+
                               (128 GB/s)
```
* **구조**: 2대씩 NVLink Bridge로 묶여 독립된 섬(Island) 2개를 형성하고, 섬 간 통신은 PCIe Gen 5 버스(128 GB/s)를 이용.
* **특징**: 4대 전체 All-to-All 통신 시 페어 간 통신 구간에서 PCIe 128 GB/s 병목 발생.
* **적용**: 
  * 독립적인 2개의 서빙 인스턴스 운용 (각 인스턴스당 TP=2)
  * 또는 노드 내 하이브리드 병렬화 (`TP=2` [NVLink 활용] + `DP=2` or `PP=2` [PCIe 활용]).

---

#### 3. H100 SXM 8대 w/ NVLink (Direct Full Mesh - NVSwitch 없음)
```
                  (GPU 0)
               ／ ｜ ｜ ｜ ＼
             ／   ｜ ｜ ｜   ＼
        (GPU 7)---+-+-+-+---(GPU 1)
         ｜ ＼   ／ ｜ ｜ ＼   ／ ｜
         ｜   ＼    ｜ ｜    ／   ｜
        (GPU 6)---+-+-+-+---(GPU 2)
             ＼   ｜ ｜ ｜   ／
               ＼ ｜ ｜ ｜ ／
             (GPU 5)---(GPU 4)---(GPU 3)
   * 8개 GPU 간 물리적 NVLink 링크를 1:1로 직접 상호 연결 (Full Mesh P2P)
```
* **구조**: 별도의 NVSwitch 없이, 각 H100 GPU가 가진 18개의 4세대 NVLink 포트(총 900 GB/s)를 나머지 7개 피어 GPU에 1:1로 직접 분할 배선한 풀 메시(Full Mesh) 구조.
* **통신 특성**:
  * **모든 GPU 쌍 간 1홉(Direct P2P)**으로 직접 통신 가능.
  * 18개 링크를 7개 GPU에 나누어 배정하므로, 특정 단일 GPU 쌍(1:1 P2P) 간에는 물리 링크가 2~3개로 고정되어 **실제 1:1 대역폭은 ~100~150 GB/s로 제한**.
  * 7개 피어 모두와 동시에 데이터를 주고받는 완전 분산 All-to-All / All-Reduce 상황에서만 각 GPU의 총 900 GB/s 대역폭이 온전히 활용됨.

---

#### 4. H100 SXM 8대 w/ NVLink + NVSwitch (Full Non-blocking Fabric, HGX/DGX H100)
```
  +-------+ +-------+ +-------+ +-------+ +-------+ +-------+ +-------+ +-------+
  | GPU 0 | | GPU 1 | | GPU 2 | | GPU 3 | | GPU 4 | | GPU 5 | | GPU 6 | | GPU 7 |
  +---+---+ +---+---+ +---+---+ +---+---+ +---+---+ +---+---+ +---+---+ +---+---+
      |         |         |         |         |         |         |         |
  =============================================================================
   [ 4x 3세대 NVSwitch 칩 (Non-blocking Crossbar Switch Fabric, 900 GB/s per GPU) ]
  =============================================================================
```
* **구조**: 8개의 SXM5 GPU가 4개의 온보드 NVSwitch를 통해 완전 교차 연결된 넌블로킹(Non-blocking) 스위치 패브릭 구성.
* **통신 특성**:
  * **어떤 GPU끼리 1:1 통신을 하든 동적 스위칭을 통해 900 GB/s 풀 대역폭을 온전히 집중(Burst) 가능**.
  * 총 3.6 TB/s의 노드 내부 바이섹션 대역폭을 제공하여 통신 병목 완전 제거.
  * 8개 GPU 전체가 물리적 제약 없이 하나의 거대한 단일 가상 GPU처럼 동작.
* **적용**: 
  * **초거대 LLM 8-way 텐서 병렬화 (`TP=8`)**
  * 파운데이션 모델 대규모 분산 사전 학습(Pre-training) 및 최고 부하 서빙 클러스터.

---

## 5. LLM 서빙 및 워크로드 관점의 선택 가이드

```
+-------------------------------------------------------------------------------+
|                             NVIDIA H100 선택 가이드                           |
+------------------------------------+------------------------------------------+
|          H100 SXM 선택             |              H100 NVL 선택               |
+------------------------------------+------------------------------------------+
| - 파운데이션 모델 사전 학습 (Pre-train)  | - 대규모 LLM 실시간 추론 & 서빙 (Inference)   |
| - 수천 장 단위의 대규모 클러스터 스케일아웃 | - 기존 PCIe 랙 서버 기반 유연한 인프라 확장   |
| - 최고 수준의 FP64 HPC 시뮬레이션     | - LLM 서빙을 위한 대용량 메모리 & 대역폭 확보 |
| - HGX/DGX 전용 랙 인프라 구축 가능 환경   | - 전력 소모 및 쿨링 효율이 중요한 엔터프라이즈 |
+------------------------------------+------------------------------------------+
```

1. **LLM 추론(Inference) 서빙 시 NVL의 우위**:
   * LLM 생성(Decoding) 단계는 주로 **메모리 대역폭 바운드(Memory Bandwidth Bound)** 특성을 보임.
   * H100 NVL은 **3.9 TB/s**의 초고속 대역폭과 **94 GB** 대용량 VRAM을 제공하여, 긴 컨텍스트(KV Cache)와 큰 배치(Batch) 처리 시 높은 처리량(TPS) 달성 가능.
2. **대규모 학습(Training) 시 SXM의 우위**:
   * 대규모 분산 학습 시 GPU 간 통신 병목이 발생하므로 **900 GB/s NVLink + NVSwitch** 기반의 고출력 연산력을 제공하는 SXM5가 최적.
