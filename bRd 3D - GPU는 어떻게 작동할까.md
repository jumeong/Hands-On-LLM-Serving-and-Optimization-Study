# GPU는 어떻게 작동할까?

> **핵심 흐름**
>
> CPU → 순차적/복잡한 작업에 강함
> GPU → 대규모 병렬 연산에 강함
>
> GPU 내부에서는 많은 연산 유닛이 동일한 연산을 서로 다른 데이터에 병렬로 수행한다.

---

# 1. CPU와 GPU의 차이

CPU와 GPU 모두 데이터를 연산하는 프로세서지만 **설계 목적이 다르다.**

### CPU

```text
┌──────────────────────────────┐
│        Powerful Core         │
│                              │
│  Control │ ALU │ Cache │ ... │
└──────────────────────────────┘
```

* Core 하나의 성능이 높음
* 복잡한 제어 흐름에 강함
* Branch가 많은 작업에 유리
* 다양한 종류의 작업을 빠르게 처리

### GPU

```text
┌─────────────────────────────────┐
│ Core │ Core │ Core │ Core │ ... │
│ Core │ Core │ Core │ Core │ ... │
│ Core │ Core │ Core │ Core │ ... │
└─────────────────────────────────┘
```

* 많은 연산 유닛
* 동일한 연산을 대량으로 수행하는 데 특화
* 데이터 병렬성이 높은 작업에 강함
* 그래픽, 행렬 연산, AI 등에 적합

---

# 2. GPU가 필요한 이유

예를 들어 다음과 같은 연산이 있다고 하자.

```text
A1 + B1
A2 + B2
A3 + B3
A4 + B4
...
A1000 + B1000
```

각 연산이 서로 독립적이라면 하나씩 순차적으로 계산할 필요가 없다.

```text
CPU

A1+B1
  ↓
A2+B2
  ↓
A3+B3
  ↓
...
```

GPU는 여러 연산을 동시에 수행할 수 있다.

```text
GPU

A1+B1 ──┐
A2+B2 ──┤
A3+B3 ──┤
A4+B4 ──┤──→ 동시에 처리
...     ──┤
A1000+B1000 ┘
```

즉 GPU의 핵심은 **많은 데이터를 동시에 처리하는 것**이다.

---

# 3. 병렬 처리(Parallel Processing)

GPU의 핵심 특징은 **Data Parallelism**이다.

예를 들어 벡터 연산:

```text
C[i] = A[i] + B[i]
```

가 있다고 하자.

```text
A = [A0 A1 A2 A3]
B = [B0 B1 B2 B3]

C = [A0+B0, A1+B1, A2+B2, A3+B3]
```

각 원소의 연산은 서로 독립적이다.

따라서:

```text
Thread 0 → A0 + B0
Thread 1 → A1 + B1
Thread 2 → A2 + B2
Thread 3 → A3 + B3
```

처럼 병렬화할 수 있다.

---

# 4. SIMD

GPU의 병렬 처리 개념을 이해하려면 **SIMD**를 이해해야 한다.

> **SIMD = Single Instruction, Multiple Data**

하나의 명령어로 여러 데이터를 처리한다.

```text
              Instruction
                   │
        ┌──────────┼──────────┐
        ↓          ↓          ↓
       A0         A1         A2
        │          │          │
        +          +          +
        │          │          │
       B0         B1         B2
        │          │          │
        ↓          ↓          ↓
       C0         C1         C2
```

즉,

```text
1 Instruction
      ↓
Multiple Data
```

이다.

---

# 5. SIMT

GPU에서는 SIMD 개념을 발전시킨 **SIMT**라는 방식이 중요하다.

> **SIMT = Single Instruction, Multiple Threads**

여러 Thread가 동일한 명령어 흐름을 따라가면서 서로 다른 데이터를 처리한다.

```text
             Instruction
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
     Thread 0  Thread 1  Thread 2
        ↓         ↓         ↓
      Data 0    Data 1    Data 2
```

핵심 차이는 다음과 같다.

```text
SIMD
→ 하나의 명령어로 여러 데이터 처리

SIMT
→ 여러 Thread가 동일한 명령어를 실행
```

GPU 프로그래밍에서는 **Thread라는 개념이 매우 중요**하다.

---

# 6. GPU Thread

GPU에서는 하나의 큰 작업을 수많은 Thread로 쪼갤 수 있다.

예:

```text
1000개의 데이터 처리
        ↓
1000개의 Thread
```

각 Thread가 자신의 데이터를 처리한다.

```text
Thread 0 → Data 0
Thread 1 → Data 1
Thread 2 → Data 2
...
Thread 999 → Data 999
```

이렇게 하면 GPU의 병렬 처리 능력을 활용할 수 있다.

---

# 7. Thread → Block → Grid

GPU에서는 Thread를 단순히 무한정 나열하지 않고 계층적으로 관리한다.

```text
Grid
│
├── Block 0
│   ├── Thread 0
│   ├── Thread 1
│   ├── Thread 2
│   └── ...
│
├── Block 1
│   ├── Thread 0
│   ├── Thread 1
│   └── ...
│
└── Block 2
    ├── Thread 0
    ├── Thread 1
    └── ...
```

### Thread

가장 작은 실행 단위.

### Block

여러 Thread의 그룹.

### Grid

Kernel 실행 전체를 구성하는 Block들의 집합.

```text
Grid
 ↓
Blocks
 ↓
Threads
```

이 구조가 CUDA 프로그래밍의 핵심적인 실행 모델이다.

---

# 8. Warp

NVIDIA GPU에서는 Thread를 다시 **Warp** 단위로 묶어서 실행한다.

일반적으로:

```text
1 Warp = 32 Threads
```

이다.

```text
Warp
├── Thread 0
├── Thread 1
├── Thread 2
├── ...
└── Thread 31
```

GPU 하드웨어는 Warp 단위로 Thread들을 함께 실행한다.

따라서 GPU에서 실제 성능을 이해하려면

```text
Thread
  ↓
Warp
  ↓
Block
  ↓
Grid
```

구조를 이해하는 것이 중요하다.

---

# 9. Branch Divergence

GPU에서는 모든 Thread가 동일한 명령어 흐름을 따라갈 때 가장 효율적이다.

예를 들어:

```c
if (x > 0)
    y = x + 1;
else
    y = x - 1;
```

Thread마다 조건이 다르면 문제가 발생한다.

```text
Thread 0 → if
Thread 1 → if
Thread 2 → else
Thread 3 → else
```

같은 Warp 안에서 서로 다른 Branch를 실행하게 된다.

이를 **Branch Divergence**라고 한다.

개념적으로:

```text
          Warp
            │
       ┌────┴────┐
       ↓         ↓
      IF        ELSE
       │         │
       └────┬────┘
            ↓
         계속 실행
```

결과적으로 GPU의 병렬성이 떨어질 수 있다.

따라서 GPU에서는 **Thread 간 Branch를 최대한 동일하게 만드는 것이 중요**하다.

---

# 10. GPU 내부 구조

GPU는 일반적인 CPU와 달리 매우 많은 연산 유닛을 가지고 있다.

NVIDIA GPU를 예로 들면:

```text
GPU
│
├── SM 0
│   ├── CUDA Core
│   ├── Register
│   ├── Shared Memory
│   └── ...
│
├── SM 1
│   ├── CUDA Core
│   ├── Register
│   ├── Shared Memory
│   └── ...
│
└── ...
```

> **SM = Streaming Multiprocessor**

GPU에서 Thread Block이 실제로 실행되는 핵심 실행 단위다.

---

# 11. CUDA Core

CUDA Core는 NVIDIA GPU에서 일반적인 연산을 수행하는 연산 유닛이다.

개념적으로:

```text
SM
│
├── CUDA Core
├── CUDA Core
├── CUDA Core
├── CUDA Core
└── ...
```

하나의 SM에는 여러 CUDA Core가 존재한다.

따라서:

```text
GPU
 ↓
많은 SM
 ↓
많은 CUDA Core
 ↓
많은 Thread 병렬 실행
```

이라는 구조를 갖는다.

---

# 12. GPU Memory Hierarchy

GPU에서도 메모리 계층 구조가 중요하다.

대표적으로:

```text
          GPU Core
             │
          Register
             │
       Shared Memory
             │
         L1 Cache
             │
         L2 Cache
             │
        Global Memory
             │
           DRAM
```

속도와 용량 사이의 Trade-off가 존재한다.

---

# 13. Register

각 Thread가 사용하는 가장 빠른 메모리이다.

```text
Thread
  │
  └── Register
```

Thread마다 독립적인 Register 공간을 가진다.

따라서 Register 사용량이 많아지면 GPU에서 동시에 실행할 수 있는 Thread 수가 줄어들 수 있다.

---

# 14. Shared Memory

Shared Memory는 같은 Block에 속한 Thread들이 공유할 수 있는 빠른 메모리이다.

```text
       Block
    ┌───────────┐
    │ Thread 0  │
    │ Thread 1  │
    │ Thread 2  │
    │    ...    │
    ├───────────┤
    │  Shared   │
    │  Memory   │
    └───────────┘
```

특징:

* Block 내부 Thread들이 공유
* Global Memory보다 빠름
* Thread 간 데이터 공유에 활용
* CUDA에서 성능 최적화에 매우 중요

---

# 15. Global Memory

GPU의 대용량 메모리이다.

일반적으로 GPU의 VRAM에 해당한다.

```text
GPU
 │
 ├── SM
 │
 └── Global Memory
          │
        VRAM
```

용량은 크지만 Register나 Shared Memory에 비해 접근 latency가 크다.

따라서 GPU 성능에서는 **Memory Access를 어떻게 줄이고 효율적으로 가져오느냐**가 중요하다.

---

# 16. Memory Coalescing

GPU에서는 여러 Thread가 메모리를 어떻게 접근하는지가 중요하다.

예를 들어:

```text
Thread 0 → Memory 0
Thread 1 → Memory 1
Thread 2 → Memory 2
Thread 3 → Memory 3
```

처럼 연속적인 주소에 접근하면 효율적으로 데이터를 가져올 수 있다.

반대로:

```text
Thread 0 → Memory 0
Thread 1 → Memory 1000
Thread 2 → Memory 500
Thread 3 → Memory 9000
```

처럼 불규칙하게 접근하면 메모리 효율이 떨어질 수 있다.

이를 **Memory Coalescing**과 연결해서 이해할 수 있다.

---

# 17. GPU에서 Kernel

GPU에서 실행되는 함수를 **Kernel**이라고 한다.

CPU가 GPU에게 Kernel 실행을 요청하면:

```text
CPU
 │
 │ Kernel Launch
 ↓
GPU
 │
 ├── Block 0
 ├── Block 1
 ├── Block 2
 └── ...
```

GPU가 수많은 Thread를 생성하여 병렬 실행한다.

---

# 18. CPU와 GPU의 협력

GPU는 CPU를 완전히 대체하는 것이 아니다.

일반적인 구조는:

```text
             CPU
              │
       ┌──────┴──────┐
       │             │
       ↓             ↓
   일반적인 작업    GPU 작업
                     │
                     ↓
                    GPU
                     │
               병렬 연산 수행
                     │
                     ↓
                   Result
                     │
                     ↓
                    CPU
```

CPU가 전체 프로그램의 흐름을 제어하고 GPU가 대규모 병렬 연산을 담당한다.

---

# 19. GPU가 그래픽에 강한 이유

3D 그래픽에서는 수많은 픽셀에 대해 유사한 연산을 반복해야 한다.

예를 들어:

```text
Pixel 0 → 연산
Pixel 1 → 연산
Pixel 2 → 연산
Pixel 3 → 연산
...
Pixel 1,000,000 → 연산
```

각 픽셀에 대해 유사한 연산을 수행하기 때문에 GPU의 병렬 구조와 매우 잘 맞는다.

```text
                    GPU
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
    Pixel 0       Pixel 1       Pixel 2
       ↓             ↓             ↓
      연산           연산           연산
```

---

# 20. GPU와 AI

AI 연산 역시 GPU와 잘 맞는다.

특히 Neural Network에서는 다음과 같은 연산이 반복된다.

```text
Matrix × Matrix
Matrix × Vector
Vector + Vector
```

예:

```text
C = A × B
```

행렬의 각 원소 계산은 대규모 병렬화가 가능하다.

```text
C00 ← A Row 0 × B Column 0
C01 ← A Row 0 × B Column 1
C02 ← A Row 0 × B Column 2
...
```

따라서 GPU는 AI의 Matrix 연산을 매우 효율적으로 수행할 수 있다.

---

# 21. Tensor Core

AI 연산을 위해 NVIDIA GPU에는 **Tensor Core**와 같은 특화 연산 유닛도 존재한다.

```text
SM
│
├── CUDA Core
├── Tensor Core
├── Register
├── Shared Memory
└── ...
```

Tensor Core는 특히 Matrix Multiply-Accumulate와 같은 AI 핵심 연산을 높은 처리량으로 수행하도록 설계되어 있다.

즉 GPU 내부에서도

```text
General Compute
      ↓
CUDA Core

AI / Matrix Compute
      ↓
Tensor Core
```

처럼 역할이 분화된다.

---

# 22. GPU 성능의 핵심

GPU의 성능은 단순히 **CUDA Core 개수**만으로 결정되지 않는다.

중요한 요소는:

```text
Compute
 ├── 연산 유닛 수
 ├── Clock
 └── 연산 효율

Memory
 ├── Bandwidth
 ├── Latency
 └── Cache

Parallelism
 ├── Thread 수
 ├── Warp
 ├── Occupancy
 └── 병렬성

Control
 └── Branch Divergence
```

즉,

> **GPU 성능 = 연산 성능 + 메모리 성능 + 병렬 실행 효율**

로 이해하는 것이 좋다.

---

# 23. CPU vs GPU 핵심 비교

|                  | CPU             | GPU               |
| ---------------- | --------------- | ----------------- |
| 핵심 목적            | 범용 연산           | 대규모 병렬 연산         |
| Core             | 적음 / 강력         | 매우 많음             |
| Control          | 복잡한 제어에 강함      | 동일한 연산에 유리        |
| Branch           | 상대적으로 유리        | Divergence에 취약    |
| Parallelism      | 상대적으로 낮음        | 매우 높음             |
| Cache            | 매우 중요           | 중요                |
| Memory Bandwidth | 상대적으로 낮음        | 매우 중요             |
| 대표 작업            | OS, Application | Graphics, AI, HPC |

---

# 24. 전체 구조

GPU의 전체적인 실행 구조를 하나로 연결하면:

```text
                 CPU
                  │
             Kernel Launch
                  │
                  ↓
                GPU
                  │
                Grid
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
      Block     Block     Block
        │         │         │
       Warp      Warp      Warp
        │         │         │
     Threads   Threads   Threads
        │         │         │
        ↓         ↓         ↓
      CUDA      CUDA      CUDA
      Core      Core      Core
        │         │         │
        └─────────┼─────────┘
                  ↓
              Memory
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
    Register   Shared      L1/L2
               Memory      Cache
                  │
                  ↓
             Global Memory
                  │
                 VRAM
```

---

# ⭐ 핵심 개념 정리

### ① GPU의 본질

```text
많은 연산 유닛
      +
많은 Thread
      +
높은 Memory Bandwidth
      ↓
대규모 병렬 처리
```

### ② 실행 계층

```text
Grid
 ↓
Block
 ↓
Warp
 ↓
Thread
 ↓
CUDA Core / Execution Unit
```

### ③ 메모리 계층

```text
Register
   ↓
Shared Memory
   ↓
L1 / L2 Cache
   ↓
Global Memory
   ↓
VRAM
```

### ④ GPU가 강한 작업

```text
Graphics
Matrix Operation
AI
HPC
Scientific Computing
```

### ⑤ GPU가 상대적으로 약한 작업

```text
복잡한 Branch
순차적 Dependency
불규칙한 Memory Access
```

---

# 🎯 CPU와 GPU를 한 문장으로 비교

> **CPU는 적은 수의 강력한 Core로 복잡하고 다양한 작업을 빠르게 처리하는 데 최적화되어 있고, GPU는 많은 연산 유닛과 Thread를 이용해 동일하거나 유사한 연산을 대규모 데이터에 병렬로 수행하는 데 최적화되어 있다.**

---
