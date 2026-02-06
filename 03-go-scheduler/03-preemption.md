# Preemption

## 1️⃣ Simple Explanation (Beginner Friendly)

**Preemption** means the scheduler can interrupt a running goroutine to give other goroutines a chance to run.

Before Go 1.14: Only **cooperative** - goroutine had to yield voluntarily.
After Go 1.14: Also **asynchronous** - scheduler can interrupt any goroutine.

---

## 2️⃣ Real-World Analogy

### 🎤 Microphone at Meeting

- **Cooperative**: Speaker talks until they pass the mic
- **Preemptive**: Someone can tap speaker's shoulder to share time

---

## 3️⃣ Technical Working (Step-by-Step)

### Cooperative Preemption Points

```
┌────────────────────────────────────────────────┐
│ Goroutine yields at:                           │
│ • Function calls (stack check)                 │
│ • Channel operations                           │
│ • Blocking syscalls                            │
│ • runtime.Gosched()                            │
│ • Memory allocation                            │
└────────────────────────────────────────────────┘
```

### Asynchronous Preemption (Go 1.14+)

```
Problem (pre-1.14):
for { compute() }  ← Never yields, starves other Gs!

Solution (1.14+):
OS Signal (SIGURG)
       │
       ▼
Signal handler checks if G should yield
       │
       ▼
Saves state, switches to scheduler
```

### How It Works

```
sysmon goroutine (runs independently):
┌─────────────────────────────────────┐
│ Loop every 10ms:                    │
│   For each P:                       │
│     If G running > 10ms:            │
│       Send preemption signal        │
└─────────────────────────────────────┘
         │
         ▼
Signal received by M:
┌─────────────────────────────────────┐
│ Save G's registers to stack         │
│ Mark G as preempted                 │
│ Return to scheduler                 │
└─────────────────────────────────────┘
```

---

## 4️⃣ Where Used in Real Systems?

### Preventing Starvation

```go
// Without preemption, this starves other goroutines:
go func() {
    for {
        heavyComputation()  // No function calls = no yield
    }
}()

// With async preemption (Go 1.14+):
// Scheduler interrupts after ~10ms, other Gs run
```

---

## 5️⃣ Practical Examples

### Testing Preemption

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func main() {
    runtime.GOMAXPROCS(1)  // Force single P
    
    go func() {
        for i := 0; ; i++ {
            // Pre-1.14: would never yield
            // Post-1.14: preempted after ~10ms
        }
    }()
    
    go func() {
        fmt.Println("I can run now!")
    }()
    
    time.Sleep(100 * time.Millisecond)
}
```

### Manual Yield

```go
for {
    doWork()
    runtime.Gosched()  // Voluntarily yield
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Go was always preemptive | Async preemption added in Go 1.14 |
| Time.Sleep yields | Correct! It does yield |
| Heavy loops never preempt | They do now (post-1.14) |
| CGO respects preemption | CGO can block preemption |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Cooperative: yields at function calls,     │
│   channels, syscalls, Gosched()            │
│                                            │
│ Asynchronous (Go 1.14+): sysmon sends      │
│   SIGURG after ~10ms runtime               │
│                                            │
│ Purpose: Prevent starvation, ensure        │
│   fairness among goroutines                │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What was the main limitation before Go 1.14?
2. How does async preemption work technically?
3. What signal is used for preemption?
4. Can CGO code be preempted?
5. What is the typical preemption interval?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Tight loops without function calls could starve other goroutines forever.

**A2**: sysmon sends SIGURG signal, handler saves state, returns to scheduler.

**A3**: SIGURG (urgent socket data signal, rarely used otherwise).

**A4**: No. CGO calls block preemption. Long CGO calls can still starve.

**A5**: ~10ms. sysmon checks every 10ms if G has run too long.

</details>

---

[← Previous](./02-work-stealing.md) | [Next →](./04-scheduler-blocking-flow.md)
