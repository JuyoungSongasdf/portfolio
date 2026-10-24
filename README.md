# Operating System Simulation Projects

운영체제의 핵심 메커니즘을 C 언어로 구현하고 시뮬레이션한 과제 아카이브입니다.  
> **Academic Integrity Notice**: 본 저장소는 소속 교육기관의 표절 규정을 준수하기 위해 소스코드는 비공개 처리되어 있으며, 설계 다이어그램, 핵심 로직 구조 및 실행 바이너리/데모 결과 위주로 아카이빙되어 있습니다.

---

## Project Index
1. [Unix Process Scheduler Simulation](#1-unix-process-scheduler-simulation)
2. *(다음 과제명 추가 예정)*

---

## 1. Unix Process Scheduler Simulation

### Overview
Unix 환경에서 프로세스의 생성, 동적 우선순위 기반 CPU 스케줄링, I/O 블로킹, 종료에 이르는 전 생명주기를 재현한 이벤트 드리븐 시뮬레이터입니다.

- **Environment**: C (C89/ANSI C), GCC, POSIX `usleep`
- **Key Concepts**: Context Switching, Preemptive Dynamic Priority, Multi-level Queue Ordering, Singly Linked List with Sentinel Nodes

---

### System Architecture & Logic Flow

#### 1) Process State Transition Diagram
```text
           [ New Process ]
                  │
                  ▼
         ┌────────────────┐
         │     READY      │◄──────────────┐
         └───────┬────────┘               │
                 │ Dispatch               │ Unblocked (P=1/20)
                 ▼                        │
         ┌────────────────┐               │
         │    RUNNING     │────────┐      │
         └───────┬────────┘        │      │
                 │                 ▼      │
   Quantum / End │         ┌──────────────┴─┐
                 │         │    BLOCKED     │
                 ▼         │   (I/O Bound)  │
           [ Terminated ]  └────────────────┘

```

#### 2) Priority & Ordering Logic

* **Sentinel Nodes**: 헤더와 트레일러 노드를 사용해 리스트의 삽입/삭제 시 예외 분기를 최소화한 정렬 링크드 리스트를 사용합니다.


* **Priority Mapping**: 음수 우선순위(I/O Unblocked 우선 처리)와 양수 우선순위(CPU-bound)를 일관되게 정렬하기 위해 `sort_key` 변환 함수를 적용합니다.


* **우선순위 재계산 공식**:

$$
\text{PRI}_{\text{new}} = \left\lceil \frac{|\text{PRI}_{\text{old}}| + \text{Quantum Used}}{2} \right\rceil
$$

* **블로킹 여부에 따른 부호 부여**:

$$
\text{PRI}_{\text{final}} = 
\begin{cases} 
-\text{PRI}_{\text{new}} & \text{if process blocked (I/O Bound)} \\ 
+\text{PRI}_{\text{new}} & \text{if fully used quantum (CPU Bound)} 
\end{cases}
$$

* I/O 바운드(블록 발생) 프로세스는 음수 부호($-\text{PRI}$)를 부여받아 언블록 시 CPU를 최우선 선점합니다.





#### 3) Simulation Tick Cycle (60Hz Clock Interrupt)

```text
[Clock Tick (1/60s)] 
   ├── 1. Context Switch Check (Running Quantum 소진 or Idle & Ready 대기)
   │        └── Context Switch 발생 시: BEFORE/AFTER 테이블 출력 및 우선순위 재배치
   ├── 2. CPU Execution (Current Process CPU/Quantum Ticks 누적)
   ├── 3. Dynamic Process Arrival (20% 확률로 PID 생성, 최대 10개 제한)
   └── 4. I/O Check (각 Blocked 프로세스 5% 확률로 Ready 전이)

```

---

### Execution & Output Example

컨텍스트 스위칭이 일어나는 순간 시스템의 상태 변화(BEFORE/AFTER)를 기록합니다.

```text
 BEFORE SCHEDULING CPU:  Next PID =  6,  Number of Processes =  5
 PID   CPU Used   MAX Time   STATE   PRI   QUANTUM USED   BLK TIME   WAIT TKS
   1          6         12       R       0              0          6          0
   2          0          8       R       0              0          2          0
   3          0         15       R       0              0          4          0
   4          0          3       R       0              0          6          0
   5          0          9       R       0              0          1          0

 AFTER SCHEDULING CPU:  Next PID =  6,  Number of Processes =  5
 PID   CPU Used   MAX Time   STATE   PRI   QUANTUM USED   BLK TIME   WAIT TKS
   1          6         12       N       3              0          6          0
   2          0          8       R       0              0          2          0
   3          0         15       R       0              0          4          0
   4          0          3       R       0              0          6          0
   5          0          9       R       0              0          1          0

```

---

### How to Run (Precompiled Binary)

본 프로젝트는 소스코드를 제공하지 않으며, 컴파일된 실행 파일만 제공됩니다.

```bash
# Clone repository
git clone [https://github.com/](https://github.com/)JuyoungSongasdf/portfolio.git
cd portfolio

# Execute binary (Linux/macOS)
chmod +x ./bin/scheduler
./bin/scheduler

# Or Windows
.\bin\scheduler.exe

```
