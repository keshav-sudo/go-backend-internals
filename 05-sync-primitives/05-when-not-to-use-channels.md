# When NOT to Use Channels

## 1️⃣ Simple Explanation (Beginner Friendly)

Channels are great, but not always the best tool:

| Use Case | Best Tool |
|----------|-----------|
| Protecting shared state | Mutex |
| Simple counter | Atomic |
| Waiting for goroutines | WaitGroup |
| Passing ownership | Channel ✓ |
| Signaling events | Channel ✓ |

---

## 2️⃣ Real-World Analogy

### 🔧 Right Tool for the Job

Channel = mail system. Great for sending packages.
Mutex = lock on a safe. Better for protecting valuables in place.

Don't mail the safe back and forth!

---

## 3️⃣ Technical Working (Step-by-Step)

### Decision Matrix

```
┌────────────────────────────────────────────┐
│ Need to:                  Use:             │
├────────────────────────────────────────────┤
│ Pass data between Gs     → Channel         │
│ Protect shared variable  → Mutex           │
│ Count completions        → WaitGroup       │
│ Increment counter        → Atomic          │
│ Signal event             → Channel/Context │
│ Limit concurrency        → Semaphore chan  │
└────────────────────────────────────────────┘
```

### Performance Comparison

```
Mutex operation:   ~20 ns (uncontended)
Channel operation: ~200 ns
Atomic operation:  ~5 ns

Channel has overhead of:
• Lock acquisition
• Potential goroutine parking
• Memory for hchan struct
```

---

## 4️⃣ Where Used in Real Systems?

### Cache: Use Mutex, Not Channel

```go
// GOOD: Mutex for shared state
type Cache struct {
    mu   sync.RWMutex
    data map[string]string
}

// BAD: Channel overcomplicates this
type CacheBad struct {
    get  chan getRequest
    set  chan setRequest
    // Requires separate goroutine to process...
}
```

---

## 5️⃣ Practical Examples

### Counter: Atomic > Channel

```go
// GOOD: Atomic for counter
var counter int64
atomic.AddInt64(&counter, 1)

// BAD: Channel for simple counter
counterChan := make(chan int, 1)
counterChan <- 0
go func() {
    for delta := range incrementChan {
        v := <-counterChan
        counterChan <- v + delta
    }
}()
```

### Waiting: WaitGroup > Channel

```go
// GOOD: WaitGroup for waiting
var wg sync.WaitGroup
for i := 0; i < n; i++ {
    wg.Add(1)
    go func() { defer wg.Done(); work() }()
}
wg.Wait()

// OKAY but complex: Channel counting
done := make(chan bool, n)
for i := 0; i < n; i++ {
    go func() { work(); done <- true }()
}
for i := 0; i < n; i++ {
    <-done
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| "Always use channels" | Use right tool for pattern |
| Channels are faster | Often slower than mutex |
| Channel for protecting map | Use RWMutex or sync.Map |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Use Mutex: protecting shared state         │
│ Use Atomic: simple counters/flags          │
│ Use WaitGroup: waiting for group of Gs     │
│ Use Channel: passing data, signaling       │
│                                            │
│ "Don't communicate by sharing memory,      │
│  share memory by communicating" applies    │
│  to DATA FLOW, not state protection        │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What's faster for a simple counter: channel or atomic?
2. When protecting a map, what should you use?
3. Why is WaitGroup better than counting done signals?
4. When IS a channel the right choice?
5. What's the overhead of channels vs mutex?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Atomic (~5ns vs ~200ns for channel).

**A2**: sync.RWMutex or sync.Map (if suitable).

**A3**: Cleaner API, no need to track count, handles Add/Done/Wait pattern.

**A4**: Passing data between stages, pipelines, signaling completion, select multiplexing.

**A5**: Channel: ~200ns, hchan struct, potential parking. Mutex: ~20ns uncontended.

</details>

---

[← Previous](./04-atomic-operations.md) | [Next Module: Memory Management →](../06-memory-management/01-stack-vs-heap.md)
