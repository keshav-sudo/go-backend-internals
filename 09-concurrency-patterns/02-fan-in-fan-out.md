# Fan-In / Fan-Out

## 1️⃣ Simple Explanation (Beginner Friendly)

**Fan-Out**: One channel splits into multiple workers (parallelism).
**Fan-In**: Multiple channels merge into one (aggregation).

```
Fan-Out:           Fan-In:
  [in]              [ch1]
   │                 │
   ├─► worker1       ├─►
   ├─► worker2       │   [merged]
   └─► worker3       ├─►
                    [ch2]
```

---

## 2️⃣ Real-World Analogy

### 📬 Mail System

**Fan-Out**: Post office distributes mail to carriers.
**Fan-In**: All carriers return to central sorting facility.

---

## 3️⃣ Technical Working (Step-by-Step)

### Fan-Out Pattern

```
┌─────────────────────────────────────────────┐
│              FAN-OUT                        │
│                                             │
│   ┌───────┐                                 │
│   │ Input │───┬───► Worker 1 ───► Result 1  │
│   │ Chan  │   ├───► Worker 2 ───► Result 2  │
│   └───────┘   └───► Worker 3 ───► Result 3  │
└─────────────────────────────────────────────┘
```

### Fan-In Pattern

```
┌─────────────────────────────────────────────┐
│              FAN-IN                         │
│                                             │
│   Chan 1 ────┐                              │
│              ├───► ┌────────┐               │
│   Chan 2 ────┤     │ Merged │               │
│              ├───► │ Chan   │               │
│   Chan 3 ────┘     └────────┘               │
└─────────────────────────────────────────────┘
```

---

## 4️⃣ Where Used in Real Systems?

### Parallel API Calls

```go
func fetchAllUsers(ids []int) []User {
    // Fan-out: parallel fetches
    results := make([]<-chan User, len(ids))
    for i, id := range ids {
        results[i] = fetchUser(id)  // Returns channel
    }
    
    // Fan-in: merge results
    var users []User
    for _, ch := range results {
        users = append(users, <-ch)
    }
    return users
}
```

---

## 5️⃣ Practical Examples

### Fan-Out

```go
func fanOut(input <-chan int, workers int) []<-chan int {
    outputs := make([]<-chan int, workers)
    for i := 0; i < workers; i++ {
        outputs[i] = worker(input)
    }
    return outputs
}

func worker(input <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range input {
            out <- process(n)
        }
    }()
    return out
}
```

### Fan-In

```go
func fanIn(channels ...<-chan int) <-chan int {
    merged := make(chan int)
    var wg sync.WaitGroup
    
    for _, ch := range channels {
        wg.Add(1)
        go func(c <-chan int) {
            defer wg.Done()
            for v := range c {
                merged <- v
            }
        }(ch)
    }
    
    go func() {
        wg.Wait()
        close(merged)
    }()
    
    return merged
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Forget to close merged | Use WaitGroup + close |
| Order matters | Fan-in loses ordering |
| Unbounded fan-out | Use worker pool |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Fan-Out: 1 channel → N workers             │
│ Fan-In: N channels → 1 channel             │
│                                            │
│ Use cases:                                 │
│ • Parallel processing (fan-out)            │
│ • Result aggregation (fan-in)              │
│ • Pipeline stages                          │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. When to use fan-out?
2. Does fan-in preserve order?
3. How to close merged channel properly?
4. Fan-out vs worker pool?
5. How to handle errors in fan-in?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: When you need parallel processing of a single stream.

**A2**: No! Results arrive in completion order.

**A3**: WaitGroup on all feeders, close when all done.

**A4**: Worker pool: bounded workers. Fan-out: one worker per input.

**A5**: Include error in result type or separate error channel.

</details>

---

[← Previous](./01-worker-pools.md) | [Next →](./03-pipelines.md)
