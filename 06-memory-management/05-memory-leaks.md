# Memory Leaks in Go

## 1️⃣ Simple Explanation (Beginner Friendly)

Even with GC, Go programs can "leak" memory:
- Memory that's still referenced but not needed
- Growing goroutine count
- Accumulating data in maps/slices

---

## 2️⃣ Real-World Analogy

### 📚 Hoarding Books

GC only cleans what you're not referencing. If you keep adding books to your shelf but never remove any, you'll run out of space eventually!

---

## 3️⃣ Technical Working (Step-by-Step)

### Common Leak Patterns

```
1. GOROUTINE LEAKS
   ┌─────────────────────┐
   │ go func() {         │
   │     <-ch  // waits  │  No sender = waits forever
   │ }()                 │
   └─────────────────────┘

2. SLICE MEMORY RETENTION
   ┌─────────────────────┐
   │ s = s[:1]           │  Underlying array still allocated
   └─────────────────────┘

3. GROWING MAPS
   ┌─────────────────────┐
   │ cache[key] = val    │  Never deleted = grows forever
   └─────────────────────┘

4. TIME.AFTER IN LOOPS
   ┌─────────────────────┐
   │ for {               │
   │   <-time.After(d)   │  Creates new timer each iteration
   │ }                   │
   └─────────────────────┘
```

---

## 4️⃣ Where Used in Real Systems?

### Detecting Leaks

```bash
# Monitor goroutine count
curl localhost:6060/debug/pprof/goroutine?debug=0

# Heap profile
go tool pprof http://localhost:6060/debug/pprof/heap
```

---

## 5️⃣ Practical Examples

### Goroutine Leak Fix

```go
// LEAKY
func leaky() {
    ch := make(chan int)
    go func() { <-ch }()  // Blocks forever
}

// FIXED
func fixed(ctx context.Context) {
    ch := make(chan int)
    go func() {
        select {
        case <-ch:
        case <-ctx.Done():
            return
        }
    }()
}
```

### Slice Memory Fix

```go
// LEAKY: underlying array kept
func leaky(data []byte) []byte {
    return data[:10]  // Still references original
}

// FIXED: copy to new slice
func fixed(data []byte) []byte {
    result := make([]byte, 10)
    copy(result, data[:10])
    return result
}
```

### Timer Leak Fix

```go
// LEAKY
for {
    <-time.After(time.Second)  // New timer each iteration
}

// FIXED
ticker := time.NewTicker(time.Second)
defer ticker.Stop()
for range ticker.C {
    // work
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| GC prevents all leaks | Only frees unreferenced memory |
| Maps shrink automatically | Maps never shrink, only grow |
| Closed channels are freed | Only when no references exist |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Common leaks:                              │
│ • Blocked goroutines                       │
│ • Growing maps/slices                      │
│ • time.After in loops                      │
│ • Slice sub-slice references               │
│                                            │
│ Detect: pprof heap, goroutine profiles     │
│ Prevent: context, ticker.Stop(), copy      │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Why don't maps shrink in Go?
2. How do blocked goroutines cause leaks?
3. What's wrong with time.After in loops?
4. How to properly sub-slice without leaking?
5. How do you detect goroutine leaks?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Map's bucket array only grows. Deleting keys doesn't shrink it.

**A2**: Each goroutine uses ~2KB minimum. Thousands blocked = GB of memory.

**A3**: Creates new timer (and goroutine) each iteration, never garbage collected until fires.

**A4**: Copy only needed data to new slice: `result := make(...); copy(result, src)`

**A5**: Monitor runtime.NumGoroutine(), use pprof goroutine profile, use goleak in tests.

</details>

---

[← Previous](./04-stop-the-world-pauses.md) | [Next Module: Networking →](../07-networking/01-net-http-internals.md)
