# Garbage Collector

## 1️⃣ Simple Explanation (Beginner Friendly)

Go's **Garbage Collector (GC)** automatically frees memory you're no longer using. You don't need to manually free memory like in C.

Go uses a **concurrent, tri-color mark-and-sweep** algorithm.

---

## 2️⃣ Real-World Analogy

### 🧹 Office Cleaning Crew

GC = cleaning crew that works while you're still in the office (concurrent). They mark what's in use (mark phase) and clean the rest (sweep phase).

---

## 3️⃣ Technical Working (Step-by-Step)

### Tri-Color Marking

```
┌────────────────────────────────────────────┐
│ WHITE: Potentially garbage                 │
│ GRAY:  Being scanned                       │
│ BLACK: Definitely reachable                │
└────────────────────────────────────────────┘

Start:
All objects WHITE
      │
      ▼
Mark roots (globals, stacks) as GRAY
      │
      ▼
While GRAY objects exist:
  Pick GRAY object → Mark its references GRAY
  Mark object BLACK
      │
      ▼
Sweep: Free all WHITE objects
```

### GC Phases

```
1. MARK START   (STW ~1ms)
   └─ Enable write barrier

2. CONCURRENT MARK
   └─ Scan stacks, heap (runs with program)

3. MARK TERMINATION   (STW ~1ms)
   └─ Finish marking, disable write barrier

4. CONCURRENT SWEEP
   └─ Free unreachable objects
```

---

## 4️⃣ Where Used in Real Systems?

### Tuning for Latency

```go
import "runtime/debug"

func init() {
    // More frequent GC = lower memory, more CPU
    debug.SetGCPercent(50)  // Default is 100
    
    // Set target memory limit
    debug.SetMemoryLimit(1 << 30)  // 1GB
}
```

---

## 5️⃣ Practical Examples

### Manual GC Trigger

```go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    var m runtime.MemStats
    
    // Allocate
    data := make([]byte, 100*1024*1024)
    _ = data
    
    runtime.ReadMemStats(&m)
    fmt.Printf("Before GC: %d MB\n", m.HeapAlloc/1024/1024)
    
    data = nil
    runtime.GC()  // Force GC
    
    runtime.ReadMemStats(&m)
    fmt.Printf("After GC: %d MB\n", m.HeapAlloc/1024/1024)
}
```

### GC Tracing

```bash
GODEBUG=gctrace=1 ./myapp

# Output:
# gc 1 @0.012s 2%: 0.5+1.2+0.3 ms clock, ...
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| GC pauses are long | Go GC pauses <1ms typically |
| disable.GC() for performance | Usually harmful |
| GC runs constantly | Triggered by heap growth |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ • Tri-color mark-and-sweep                 │
│ • Concurrent (runs with your code)         │
│ • Sub-millisecond pauses                   │
│ • Triggered by heap growth (GOGC %)        │
│                                            │
│ Tune: GOGC, SetMemoryLimit, sync.Pool     │
│ Debug: GODEBUG=gctrace=1                   │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What triggers a GC cycle?
2. What are the three colors in tri-color marking?
3. Is Go's GC concurrent or stop-the-world?
4. What does GOGC=100 mean?
5. How do you reduce GC pressure?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Heap growth exceeds threshold (GOGC% of previous heap).

**A2**: White (potential garbage), Gray (scanning), Black (reachable).

**A3**: Mostly concurrent with short STW pauses (<1ms).

**A4**: GC triggers when heap grows 100% beyond live data size.

**A5**: Reduce allocations, use sync.Pool, reuse buffers.

</details>

---

[← Previous](./02-escape-analysis.md) | [Next →](./04-stop-the-world-pauses.md)
