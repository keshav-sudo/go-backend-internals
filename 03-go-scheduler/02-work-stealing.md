# Work Stealing

## 1️⃣ Simple Explanation (Beginner Friendly)

**Work stealing** is how Go balances work across processors. When one P runs out of work, it "steals" goroutines from other busy Ps.

```
Before stealing:
P0: [G G G G G]   P1: []  ← P1 is idle!

After stealing:
P0: [G G]         P1: [G G G]  ← P1 stole half!
```

---

## 2️⃣ Real-World Analogy

### 🏪 Checkout Lines

Imagine a supermarket: when one cashier's line is empty, they "steal" customers from busy lines. Everyone gets served faster!

---

## 3️⃣ Technical Working (Step-by-Step)

### Work Stealing Algorithm

```
P1 runs out of work:
        │
        ▼
1. Check own local queue → Empty
        │
        ▼
2. Check global queue → Empty  
        │
        ▼
3. Check netpoller → No ready Gs
        │
        ▼
4. Try to steal from random P:
   ┌──────────────┐
   │ P0's queue:  │
   │ [G1 G2 G3 G4]│
   └──────────────┘
        │
        ▼
   Steal HALF (from tail):
   ┌──────────────┐      ┌──────────────┐
   │ P0: [G1 G2]  │      │ P1: [G3 G4]  │
   └──────────────┘      └──────────────┘
```

### Stealing Order

```
P looking for work (findRunnable):
┌────────────────────────────────────────────────┐
│ 1. Local run queue (runnext first)             │
│ 2. Global run queue (1/61 of the time)         │
│ 3. Netpoller (network I/O ready)               │
│ 4. Steal from random P (half of their queue)   │
│ 5. If all fail → P goes idle                   │
└────────────────────────────────────────────────┘
```

### Why Steal Half?

```
Steal all: Victim P immediately needs to steal back
Steal one: Multiple steal attempts needed
Steal half: Balanced - reduces total steal operations
```

---

## 4️⃣ Where Used in Real Systems?

### Load Balancing HTTP Requests

```go
// 4 CPUs, requests arrive unevenly
// Work stealing ensures all cores stay busy

func handler(w http.ResponseWriter, r *http.Request) {
    // Even if all requests hit one P initially,
    // work stealing distributes them
    compute()
    respond(w)
}
```

---

## 5️⃣ Practical Examples

### Observing Work Stealing

```bash
GODEBUG=schedtrace=1000 ./myapp

# Look for steal counts in output:
# ...runqueue=0 [0 5 0 0] steal=47
#                         ^^^^^
#                         47 steals occurred
```

### Imbalanced Workload Demo

```go
package main

import (
    "runtime"
    "sync"
)

func main() {
    runtime.GOMAXPROCS(4)
    var wg sync.WaitGroup
    
    // All Gs created on P0 initially
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            work()
        }()
    }
    // Work stealing balances across all Ps
    wg.Wait()
}

func work() {
    sum := 0
    for i := 0; i < 1000000; i++ {
        sum += i
    }
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Only steals from one P | Tries random Ps until success |
| Steals entire queue | Steals half (from tail) |
| Global queue checked first | Local queue checked first |
| Work stealing is expensive | Rare operation, amortized cheap |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ • Idle P steals from busy Ps              │
│ • Steals HALF of victim's queue           │
│ • Steals from TAIL (victim runs HEAD)     │
│ • Order: local → global → netpoll → steal │
│ • Enables automatic load balancing        │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Why steal half instead of all or one?
2. Which end of the queue does stealing take from?
3. What's checked before attempting to steal?
4. How is the victim P chosen?
5. Can stealing cause cache performance issues?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Half balances load without ping-pong stealing. Stealing all would cause immediate reverse-stealing.

**A2**: Steals from TAIL. Victim runs from HEAD. This minimizes contention.

**A3**: Local queue → Global queue → Netpoller → Then steal.

**A4**: Random selection. Randomness prevents convoy effects.

**A5**: Yes, but goroutines are lightweight. The locality benefit of work stealing outweighs cache miss costs.

</details>

---

[← Previous](./01-mpg-model.md) | [Next →](./03-preemption.md)
