
# CPU는 어떻게 작동할까?

---

# 1. Logic Gate

컴퓨터의 모든 연산은 **논리 연산**에서 시작된다.

Logic Gate는 0과 1을 입력으로 받아 특정 규칙에 따라 출력을 만든다.

### NOT

```text
0 → 1
1 → 0
```

### AND

```text
A B | OUT
---------
0 0 |  0
0 1 |  0
1 0 |  0
1 1 |  1
```

### OR

```text
A B | OUT
---------
0 0 |  0
0 1 |  1
1 0 |  1
1 1 |  1
```

### XOR

```text
A B | OUT
---------
0 0 |  0
0 1 |  1
1 0 |  1
1 1 |  0
```

---

# 2. Binary와 디지털 연산

컴퓨터는 모든 정보를 **0과 1**로 표현한다.

```text
0 = OFF
1 = ON
```

이진 표현을 통해 다음 연산들이 가능하다:

* 덧셈
* 뺄셈
* 비교
* 논리 연산

이 모든 것은 결국 **Logic Gate 조합**으로 구현된다.

---

# 3. ALU (Arithmetic Logic Unit)

ALU는 CPU의 핵심 연산 장치이다.

> **ALU = 산술 + 논리 연산 수행 장치**

### 수행 기능

* ADD
* SUB
* AND
* OR
* XOR
* 비교 연산

### 구조

```text
Input A →        ┌──────────┐
                 │          │
Input B →        │   ALU    │ → Result
                 │          │
Operation  →     └──────────┘
```

---

# 4. Register

Register는 CPU 내부의 **가장 빠른 저장 공간**이다.

CPU가 즉시 사용하는 데이터가 저장된다.

### 주요 종류

* Program Counter (PC)
* Instruction Register (IR)
* General Purpose Register
* Address Register
* Data Register

---

# 5. CPU와 Memory 계층 구조

CPU는 단독으로 동작하지 않고 메모리 계층과 함께 작동한다.

```text
        CPU
         │
      Register
         │
       L1 Cache
         │
       L2 Cache
         │
       L3 Cache
         │
        RAM
         │
       SSD / HDD
```

### 특징

* 위로 갈수록 빠름 / 작음 / 비쌈
* 아래로 갈수록 느림 / 큼 / 저렴

---

# 6. Cache Memory

CPU와 RAM 사이 속도 차이를 줄이기 위한 구조이다.

```text
CPU
 │
 ├── L1 (가장 빠름, 가장 작음)
 ├── L2
 └── L3 (상대적으로 느림, 큼)
        │
       RAM
```

### Locality 원리

#### Temporal Locality

최근 사용한 데이터는 다시 사용될 가능성이 높다.

#### Spatial Locality

근처 데이터도 함께 사용될 가능성이 높다.

---

# 7. Memory Hierarchy

전체 메모리 구조는 다음과 같다.

```text
Fast / Small / Expensive
        ▲
        │
    Register
        │
       L1
        │
       L2
        │
       L3
        │
       RAM
        │
    SSD / HDD
        │
        ▼
Slow / Large / Cheap
```

---

# 8. CPU Instruction Cycle

CPU는 모든 명령을 반복적으로 처리한다.

```text
Fetch
  ↓
Decode
  ↓
Execute
  ↓
Write Back
```

### ① Fetch

메모리에서 명령어를 가져온다.

### ② Decode

명령어를 해석한다.

### ③ Execute

ALU에서 연산을 수행한다.

### ④ Write Back

결과를 Register 또는 Memory에 저장한다.

---

# 9. Instruction과 Data

CPU는 두 가지 정보를 처리한다.

### Instruction

무엇을 할지 정의

```text
ADD
LOAD
STORE
JUMP
```

### Data

연산 대상 값

```text
10
20
100
```

---

# 10. C언어 실행 과정

CPU는 C 코드를 직접 이해하지 못한다.

```text
C / C++
   ↓
Compiler
   ↓
Assembly
   ↓
Machine Code
   ↓
CPU 실행
```

최종적으로 CPU는 기계어를 Fetch → Decode → Execute 한다.

---

# 11. CPU Bit 구조

CPU는 한 번에 처리할 수 있는 데이터 크기를 기준으로 구분된다.

```text
8-bit
16-bit
32-bit
64-bit
```

비트 수가 증가할수록:

* 처리 가능한 데이터 범위 증가
* 주소 공간 증가

---

# 12. CPU와 Memory 통신

CPU는 Memory와 다음 신호를 주고받는다.

```text
Address
Data
Control
```

### 과정

1. 주소 지정
2. Read 요청
3. 데이터 전달
4. Register 저장
5. 연산 수행

---

# 13. Core

CPU 내부의 독립 실행 단위이다.

```text
CPU
├── Core 0
├── Core 1
├── Core 2
└── Core 3
```

각 Core는 독립적으로 명령어를 처리한다.

---

# 14. Thread

Thread는 실행 흐름 단위이다.

```text
CPU
│
├── Core 0
│    ├── Thread 0
│    └── Thread 1
│
├── Core 1
│    ├── Thread 2
│    └── Thread 3
```

* Thread는 논리적 실행 단위
* Core는 물리적 실행 단위

---

# 15. Processor vs Architecture

### Processor

실제 명령어를 실행하는 하드웨어

### Architecture

명령어 구조와 동작 규칙

```text
Architecture
   ↓
Instruction Set
   ↓
Microarchitecture
   ↓
Hardware
```
