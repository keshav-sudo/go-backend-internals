# How Goroutines are Lightweight

## 1️⃣ Simple Explanation (Beginner Friendly)

Goroutines are "lightweight" because they use far fewer resources than traditional threads:

| Resource | OS Thread | Goroutine |
|----------|-----------|-----------|
| Initial Stack | ~1-8 MB | ~2 KB |
| Creation Time | ~1ms | ~1μs (1000x faster) |
| Context Switch | Kernel mode | User mode (cheaper) |
| Max Count (8GB RAM) | ~8,000 | ~4,000,000 |

---

## 2️⃣ Real-World Analogy

### 📦 Shipping Containers vs Envelopes

**OS Threads = Shipping Containers**: Big, heavy, expensive. Each needs a truck (OS resources).

**Goroutines = Envelopes**: Lightweight, cheap. Thousands fit in one mailbag (OS thread).

---

## 3️⃣ Technical Working (Step-by-Step)

### Stack Size Comparison

```
OS Thread Stack (Fixed ~2MB):
┌────────────────────────────────────┐
│                                    │
│     Allocated but unused           │
│     (wasted memory)                │
│                                    │
├────────────────────────────────────┤
│     Actual stack usage             │
└────────────────────────────────────┘

Goroutine Stack (Dynamic ~2KB initial):
┌────────┐
│ Used   │ ← Grows only when needed
└────────┘
     ↓ (can grow to 1GB if needed)
┌──────────────────┐
│   Grown stack    │
└──────────────────┘
```

### Context Switch Cost

```
OS Thread Context Switch:
┌──────────┐    ┌──────────┐    ┌──────────┐
│ User Mode│ →  │  Kernel  │ →  │ User Mode│
└──────────┘    │  Mode    │    └──────────┘
                └──────────┘
Cost: ~1-10 microseconds + cache flush

Goroutine Context Switch:
┌──────────────────────────────────────┐
│           User Mode Only             │
│   Save PC/SP → Load PC/SP → Continue │
└──────────────────────────────────────┘
Cost: ~200 nanoseconds
```

### Memory Layout

```
Single OS Thread running multiple Goroutines:

Thread M1
┌─────────────────────────────────────────────┐
│  Thread Stack (~8KB used by runtime)        │
├─────────────────────────────────────────────┤
│  Currently executing: G1                    │
│  ┌─────────┐                                │
│  │ G1 Stack│ (2KB-32KB)                     │
│  └─────────┘                                │
├─────────────────────────────────────────────┤
│  Waiting goroutines (stored in heap):       │
│  G2, G3, G4, G5... (only G struct + stack)  │
└─────────────────────────────────────────────┘
```

---

## 4️⃣ Where Used in Real Systems?

### High-Connection Servers

```go
// Handle 1 million WebSocket connections
// Memory: 1M × 2KB = 2GB (vs 1M × 2MB = 2TB for threads!)
func main() {
    for {
        conn, _ := listener.Accept()
        go handleConnection(conn)  // Millions possible
    }
}
```

---

## 5️⃣ Practical Examples

### Measuring Goroutine Memory

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func main() {
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    before := m.Alloc
    
    // Create 100,000 goroutines
    for i := 0; i < 100000; i++ {
        go func() { time.Sleep(10 * time.Second) }()
    }
    
    runtime.ReadMemStats(&m)
    after := m.Alloc
    
    fmt.Printf("Goroutines: %d\n", runtime.NumGoroutine())
    fmt.Printf("Memory per goroutine: %.2f KB\n", 
        float64(after-before)/100000/1024)
}
// Output: ~2-4 KB per goroutine
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Goroutines are free | They cost ~2KB+ memory each |
| Unlimited goroutines is fine | Can exhaust memory; use worker pools |
| Stack never grows | Stacks grow/shrink dynamically |

---

## 7️⃣ Quick Summary Box

```
┌──────────────────────────────────────────┐
│ • 2KB initial stack (vs 2MB for threads) │
│ • ~1μs creation (vs ~1ms for threads)    │
│ • User-space scheduling (faster switches)│
│ • Can run millions per process           │
│ • Stacks grow dynamically as needed      │
└──────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Why can't you create millions of OS threads?
2. How does goroutine context switching differ from thread switching?
3. What makes goroutine creation faster than thread creation?
4. If goroutines are cheap, why use worker pools?
5. How much memory would 1 million goroutines use (minimum)?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Each thread needs ~2MB stack. 1M threads = 2TB RAM. Plus kernel overhead.

**A2**: Goroutine switches in user space (save/restore PC, SP). Thread switches require kernel transition and cache flush.

**A3**: No kernel call needed. Just allocate small stack from heap and add to scheduler queue.

**A4**: Control concurrency, prevent resource exhaustion (connections, file handles), limit memory usage.

**A5**: 1M × 2KB = 2GB minimum (just stacks, plus G struct overhead).

</details>

---

[← Previous](./01-what-is-a-goroutine.md) | [Next →](./03-stack-growth-shrinking.md)
