# Stop-The-World Pauses

## 1️⃣ Simple Explanation (Beginner Friendly)

**Stop-The-World (STW)** is when the GC briefly pauses ALL goroutines. Go has minimized these to **<1 millisecond** typically.

```
Program running...
    │
    ▼
STW START (~100μs - 1ms)
    │ - Write barrier enable/disable
    │ - Stack scanning
    ▼
Program continues...
```

---

## 2️⃣ Real-World Analogy

### 🚦 Red Light

STW = brief red light. Everyone stops for a moment so the crossing guard (GC) can do quick work safely.

---

## 3️⃣ Technical Working (Step-by-Step)

### STW in GC Cycle

```
┌────────────────────────────────────────────┐
│ GC Cycle:                                  │
│                                            │
│ 1. STW: Mark Start     (~100μs)            │
│    └─ Stop all Gs, enable write barrier    │
│                                            │
│ 2. CONCURRENT MARK     (runs with program) │
│                                            │
│ 3. STW: Mark Terminate (~100μs)            │
│    └─ Finish marking, disable barrier      │
│                                            │
│ 4. CONCURRENT SWEEP    (runs with program) │
└────────────────────────────────────────────┘
```

### What Causes Long STW

```
• Many goroutines (more stacks to scan)
• Large number of stack frames
• Objects with finalizers
• Goroutines in tight loops (pre-1.14)
```

---

## 4️⃣ Where Used in Real Systems?

### Monitoring P99 Latency

```go
// Latency-sensitive systems measure STW
import "runtime/metrics"

func getSTWStats() {
    samples := []metrics.Sample{{Name: "/gc/pauses:seconds"}}
    metrics.Read(samples)
}
```

---

## 5️⃣ Practical Examples

### Measuring STW

```bash
GODEBUG=gctrace=1 ./myapp

# Output includes:
# gc 5 @1.23s 2%: 0.044+2.5+0.057 ms clock
#                  ↑STW1  ↑MARK  ↑STW2
```

### Reducing STW Impact

```go
// 1. Reduce allocations
var buf = make([]byte, 4096)  // Reuse

// 2. Use sync.Pool
var pool = sync.Pool{New: func() any { return new(Object) }}

// 3. Lower GOGC for more frequent, smaller GC
debug.SetGCPercent(50)

// 4. Set memory limit (Go 1.19+)
debug.SetMemoryLimit(2 << 30)
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| STW takes seconds | <1ms in modern Go |
| All GC is STW | Only short phases are STW |
| GOGC=off eliminates STW | Just delays until OOM |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ STW: Brief pause of all goroutines         │
│ Duration: ~100μs - 1ms                     │
│                                            │
│ Occurs at:                                 │
│ • GC mark start                            │
│ • GC mark termination                      │
│                                            │
│ Reduce: fewer allocs, pools, tune GOGC     │
│ Measure: gctrace=1, runtime/metrics        │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What phases of GC require STW?
2. What's typical STW duration in Go?
3. How does async preemption help STW?
4. What makes STW longer?
5. How to measure STW in production?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Mark start and mark termination phases.

**A2**: Under 1ms, often 100-500μs.

**A3**: Allows GC to stop tight-loop goroutines quickly (Go 1.14+).

**A4**: Many goroutines, deep stacks, many finalizers.

**A5**: GODEBUG=gctrace=1, runtime/metrics, pprof.

</details>

---

[← Previous](./03-garbage-collector.md) | [Next →](./05-memory-leaks.md)
