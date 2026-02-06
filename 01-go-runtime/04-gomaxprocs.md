# GOMAXPROCS

## 1️⃣ Simple Explanation (Beginner Friendly)

`GOMAXPROCS` tells Go: **"How many CPU cores can run Go code simultaneously?"**

```go
import "runtime"

// Get current value
current := runtime.GOMAXPROCS(0)  // 0 = query only

// Set to specific value
runtime.GOMAXPROCS(4)  // Use 4 cores

// Default: uses all CPUs
runtime.GOMAXPROCS(runtime.NumCPU())
```

---

## 2️⃣ Real-World Analogy

### 🍳 Restaurant Kitchen

- **Cooking Stations (GOMAXPROCS)** = How many chefs can cook simultaneously
- **Order Tickets (Goroutines)** = Tasks waiting to be done
- Even with 100 tickets, only 4 chefs can cook at once if GOMAXPROCS=4

---

## 3️⃣ Technical Working (Step-by-Step)

### GOMAXPROCS in M:P:G Model

```
GOMAXPROCS = 4 (Four P structures)

     P0          P1          P2          P3
   ┌────┐      ┌────┐      ┌────┐      ┌────┐
   │ M  │      │ M  │      │ M  │      │idle│
   │ G  │      │ G  │      │ G  │      │    │
   │Queue│     │Queue│     │Queue│     │Queue│
   └────┘      └────┘      └────┘      └────┘
```

### CPU-Bound vs I/O-Bound

| Workload Type | GOMAXPROCS Impact |
|--------------|-------------------|
| CPU-bound | Higher = faster (up to NumCPU) |
| I/O-bound | Minimal impact (goroutines wait anyway) |

---

## 4️⃣ Where Used in Real Systems?

### Container CPU Limits Problem

```bash
# On 64-core host with 2 CPU limit:
NumCPU(): 64      # Sees all host CPUs!
GOMAXPROCS: 64    # Inefficient!
```

**Solution**: 
```go
import _ "go.uber.org/automaxprocs"
// Automatically respects container limits
```

---

## 5️⃣ Practical Examples

### Benchmarking GOMAXPROCS

```go
package main

import (
    "fmt"
    "runtime"
    "sync"
    "time"
)

func cpuWork() {
    for i := 0; i < 10_000_000; i++ {}
}

func main() {
    for _, procs := range []int{1, 2, 4, 8} {
        runtime.GOMAXPROCS(procs)
        var wg sync.WaitGroup
        start := time.Now()
        
        for i := 0; i < 8; i++ {
            wg.Add(1)
            go func() { defer wg.Done(); cpuWork() }()
        }
        wg.Wait()
        fmt.Printf("GOMAXPROCS=%d: %v\n", procs, time.Since(start))
    }
}
```

### Debug Scheduler
```bash
GODEBUG=schedtrace=1000 ./myapp
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| GOMAXPROCS = thread count | Threads can exceed GOMAXPROCS |
| Higher = always faster | Only helps CPU-bound work |
| Default is 1 | Default is NumCPU() since Go 1.5 |
| Can't change at runtime | Can call GOMAXPROCS() anytime |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ • Controls concurrent Go code execution   │
│ • Default: NumCPU()                       │
│ • Containers: Use automaxprocs            │
│ • CPU-bound: Keep at NumCPU()             │
│ • I/O-bound: Default is fine              │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. If GOMAXPROCS=4, what's the max number of OS threads?
2. Why might GOMAXPROCS=100 on 8-core machine hurt performance?
3. Why doesn't increasing GOMAXPROCS help I/O-bound servers?
4. When would you set GOMAXPROCS=1 intentionally?
5. How do you handle GOMAXPROCS in Kubernetes containers?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: No limit! GOMAXPROCS limits Go code execution, not thread count. Syscall-blocked threads don't count.

**A2**: More Ps than cores causes context switching overhead. Best: GOMAXPROCS = NumCPU().

**A3**: I/O-bound goroutines spend time waiting. Bottleneck is DB/network, not CPU.

**A4**: Deterministic testing, race debugging, single-core containers.

**A5**: Use `go.uber.org/automaxprocs` to read cgroup limits automatically.

</details>

---

[← Previous](./03-how-go-executes-code.md) | [Next →](./05-comparison-with-nodejs-java.md)
