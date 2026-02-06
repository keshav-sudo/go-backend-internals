# M:P:G Model

## 1️⃣ Simple Explanation (Beginner Friendly)

Go's scheduler uses three entities to manage concurrency:

| Entity | Name | What It Is |
|--------|------|------------|
| **G** | Goroutine | Your concurrent task |
| **M** | Machine | OS thread that runs code |
| **P** | Processor | Scheduler context (has a run queue) |

**Key insight**: G's run on M's, but only through P's. A P connects a G to an M.

---

## 2️⃣ Real-World Analogy

### 🏭 Factory Assembly Line

- **G (Goroutine)** = Work order (task to complete)
- **M (Machine/Thread)** = Worker (actually does the work)
- **P (Processor)** = Workstation (has tools + queue of work orders)

```
Work Orders (G):  📝📝📝📝📝📝📝📝
                        ↓
Workstations (P): [Queue + Tools]  [Queue + Tools]
                        ↓               ↓
Workers (M):        👷 👷 👷         👷 👷
```

A worker needs a workstation to do work. Work orders wait at workstations.

---

## 3️⃣ Technical Working (Step-by-Step)

### The M:P:G Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                       GO SCHEDULER                              │
└─────────────────────────────────────────────────────────────────┘

                         Global Run Queue
                    ┌─────────────────────────┐
                    │  G   G   G   G   G   G  │
                    └─────────────────────────┘
                              ↑
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
   ┌─────────┐           ┌─────────┐          ┌─────────┐
   │    P0   │           │    P1   │          │    P2   │
   │┌───────┐│           │┌───────┐│          │┌───────┐│
   ││  LRQ  ││           ││  LRQ  ││          ││  LRQ  ││
   ││ G G G ││           ││ G G   ││          ││ G     ││
   │└───────┘│           │└───────┘│          │└───────┘│
   │    ↓    │           │    ↓    │          │  (idle) │
   │   G◄────┼───────────┼───►G    │          │         │
   │ running │           │ running │          │         │
   └────┬────┘           └────┬────┘          └─────────┘
        │                     │
        ▼                     ▼
   ┌─────────┐           ┌─────────┐
   │    M0   │           │    M1   │
   │(thread) │           │(thread) │
   └─────────┘           └─────────┘
        │                     │
        ▼                     ▼
   ┌─────────────────────────────────────────┐
   │              CPU CORES                  │
   └─────────────────────────────────────────┘

LRQ = Local Run Queue
```

### Relationships

```
GOMAXPROCS = 4 means:
┌────────────────────────────────────────┐
│  4 P structures created                │
│  Each P can run 1 G at a time          │
│  Max 4 Gs running Go code in parallel  │
└────────────────────────────────────────┘

G count: Unlimited (millions possible)
P count: GOMAXPROCS (typically = CPU cores)
M count: Dynamic (grows for syscalls)
```

### Data Structures

```go
// G - Goroutine (simplified)
type g struct {
    stack       stack    // Current stack
    m           *m       // Current M running this G
    sched       gobuf    // Saved registers
    atomicstatus uint32  // Grunnable, Grunning, etc.
}

// M - Machine/Thread (simplified)
type m struct {
    g0          *g       // Scheduler stack
    curg        *g       // Current user goroutine
    p           *p       // Attached P
}

// P - Processor (simplified)
type p struct {
    m           *m       // Associated M
    runq        [256]g   // Local run queue
    runnext     *g       // Next G to run
}
```

---

## 4️⃣ Where Used in Real Systems?

### HTTP Server Request Flow

```
Request arrives
      │
      ▼
net.Listener.Accept() → new G created
      │
      ▼
G added to P's local run queue
      │
      ▼
M picks up G from P → executes handler
      │
      ▼
Handler completes → G becomes Gdead
```

---

## 5️⃣ Practical Examples

### Viewing Scheduler Activity

```bash
# Enable scheduler tracing
GODEBUG=schedtrace=1000,scheddetail=1 ./myapp

# Output:
# SCHED 0ms: gomaxprocs=4 idleprocs=2 threads=5 ...
# P0: status=1 schedtick=50 ...
# M0: p=0 curg=5 ...
# G1: status=4 (waiting) ...
```

### Count M, P, G

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func main() {
    // Number of Ps
    fmt.Printf("GOMAXPROCS (P count): %d\n", runtime.GOMAXPROCS(0))
    
    // Number of Gs
    fmt.Printf("Goroutines (G count): %d\n", runtime.NumGoroutine())
    
    // Spawn some Gs
    for i := 0; i < 100; i++ {
        go func() { time.Sleep(time.Second) }()
    }
    
    fmt.Printf("After spawn - Gs: %d\n", runtime.NumGoroutine())
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| M = P = G counts equal | G >> P, M >= P typically |
| More Ps = more parallelism | P = GOMAXPROCS = CPU cores |
| G runs directly on M | G runs on M only through P |
| P count is dynamic | P count fixed at startup |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ G = Goroutine (task) - millions possible   │
│ M = Machine (OS thread) - grows as needed  │
│ P = Processor (scheduler) - fixed count    │
│                                            │
│ G runs on M, but needs P for scheduling    │
│ P count = GOMAXPROCS (usually = CPUs)      │
│ Each P has local run queue (256 capacity)  │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What happens if there are more Gs than Ps?
2. Why does M count sometimes exceed P count?
3. What is stored in P's local run queue?
4. Can a G run without a P?
5. What's the purpose of the Global Run Queue?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Gs wait in local or global run queues. Scheduler round-robins between them.

**A2**: When G makes blocking syscall, M blocks but P detaches and gets new M. Old M still exists.

**A3**: Runnable Gs waiting to execute on this P.

**A4**: No. G must be scheduled through a P to run on an M.

**A5**: Overflow from local queues, newly created Gs, and for work stealing source.

</details>

---

[← Previous Module](../02-goroutines/05-goroutine-leaks.md) | [Next →](./02-work-stealing.md)
