# Goroutine Lifecycle

## 1️⃣ Simple Explanation (Beginner Friendly)

A goroutine goes through several states during its lifetime:

1. **Created** → Added to scheduler queue
2. **Runnable** → Waiting to be picked up by a thread
3. **Running** → Actively executing on a CPU
4. **Waiting** → Blocked on I/O, channel, lock, etc.
5. **Dead** → Finished execution

---

## 2️⃣ Real-World Analogy

### 🎢 Theme Park Ride Queue

- **Created**: You buy a ticket (goroutine created)
- **Runnable**: You're in line (waiting for CPU)
- **Running**: You're on the ride! (executing)
- **Waiting**: Ride paused for photo (blocked on I/O)
- **Dead**: Ride over, exit (goroutine finished)

---

## 3️⃣ Technical Working (Step-by-Step)

### State Transitions

```
                    go func()
                        │
                        ▼
                   ┌─────────┐
                   │ Gidle   │
                   └────┬────┘
                        │ newproc()
                        ▼
    ┌─────────────►┌─────────┐◄────────────────┐
    │              │Grunnable│                 │
    │              └────┬────┘                 │
    │                   │ scheduled            │
    │                   ▼                      │
    │              ┌─────────┐                 │
    │              │Grunning │                 │
    │              └────┬────┘                 │
    │                   │                      │
    │     ┌─────────────┼─────────────┐        │
    │     ▼             ▼             ▼        │
    │ ┌───────┐   ┌──────────┐   ┌────────┐   │
    │ │Gwaiting│   │Gsyscall  │   │finished│   │
    │ └───┬───┘   └────┬─────┘   └────┬───┘   │
    │     │            │              │        │
    │     │ wake       │ return       ▼        │
    └─────┘            └──────────►┌──────┐    │
                                   │Gdead │    │
                                   └──────┘    │
```

### Key States (runtime/runtime2.go)

| State | Value | Meaning |
|-------|-------|---------|
| `_Gidle` | 0 | Just allocated |
| `_Grunnable` | 1 | In run queue, waiting |
| `_Grunning` | 2 | Executing on M |
| `_Gsyscall` | 3 | In system call |
| `_Gwaiting` | 4 | Blocked (channel, lock) |
| `_Gdead` | 6 | Finished, can be reused |

---

## 4️⃣ Where Used in Real Systems?

### Understanding State for Debugging

```bash
# See goroutine states in pprof
curl http://localhost:6060/debug/pprof/goroutine?debug=2

# Output shows:
# goroutine 1 [running]:
# goroutine 5 [chan receive]:
# goroutine 8 [IO wait]:
```

---

## 5️⃣ Practical Examples

### Tracking Goroutine States

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func main() {
    fmt.Printf("Initial: %d goroutines\n", runtime.NumGoroutine())
    
    // Create waiting goroutine
    ch := make(chan int)
    go func() {
        <-ch  // Gwaiting (blocked on channel)
    }()
    
    time.Sleep(10 * time.Millisecond)
    fmt.Printf("After spawn: %d goroutines\n", runtime.NumGoroutine())
    
    ch <- 1  // Unblock
    time.Sleep(10 * time.Millisecond)
    fmt.Printf("After unblock: %d goroutines\n", runtime.NumGoroutine())
}
```

### Visualizing with pprof

```go
import _ "net/http/pprof"

func main() {
    go http.ListenAndServe(":6060", nil)
    // Visit: http://localhost:6060/debug/pprof/goroutine?debug=1
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Goroutines always finish | Can get stuck in Gwaiting forever (leak) |
| Dead goroutines are freed | G structs are pooled and reused |
| All states visible to user | Internal states; observe via pprof |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ States: Idle → Runnable → Running          │
│         ↓ waiting     ↓ finished           │
│      Gwaiting      Gdead (pooled)          │
│                                            │
│ Debug: pprof/goroutine?debug=2             │
│ Track: runtime.NumGoroutine()              │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What state is a goroutine in when blocked on a channel receive?
2. What happens to the G struct when a goroutine finishes?
3. How can you see the state of all goroutines?
4. What triggers a transition from Grunning to Grunnable?
5. Can a goroutine go from Gwaiting to Grunning directly?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: `Gwaiting` - waiting for channel operation.

**A2**: Marked as `Gdead`. The G struct is returned to a free pool for reuse.

**A3**: pprof: `curl localhost:6060/debug/pprof/goroutine?debug=2`

**A4**: Preemption, `runtime.Gosched()`, time slice expiry, or blocking operation.

**A5**: No. It goes Gwaiting → Grunnable → Grunning. Must be scheduled.

</details>

---

[← Previous](./03-stack-growth-shrinking.md) | [Next →](./05-goroutine-leaks.md)
