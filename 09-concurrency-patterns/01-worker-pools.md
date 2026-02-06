# Worker Pools

## 1️⃣ Simple Explanation (Beginner Friendly)

A **worker pool** limits concurrent work by having a fixed number of workers processing jobs from a queue.

```go
// Instead of unlimited goroutines:
for _, job := range jobs {
    go process(job)  // 1 million goroutines!
}

// Use worker pool:
for w := 0; w < 10; w++ {
    go worker(jobs)  // Only 10 workers
}
```

---

## 2️⃣ Real-World Analogy

### 🍳 Restaurant Kitchen

Without pool: Hire a new chef for each order = chaos!
With pool: 5 chefs process orders from queue = controlled.

---

## 3️⃣ Technical Working (Step-by-Step)

### Worker Pool Pattern

```
                    ┌────────────────────────────────┐
                    │          Job Queue             │
                    │    [job][job][job][job]        │
                    └──────────────┬─────────────────┘
                                   │
        ┌──────────────┬───────────┼───────────┬──────────────┐
        ▼              ▼           ▼           ▼              ▼
   ┌─────────┐   ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
   │Worker 1 │   │Worker 2 │  │Worker 3 │  │Worker 4 │  │Worker 5 │
   └────┬────┘   └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘
        │              │           │           │              │
        └──────────────┴───────────┼───────────┴──────────────┘
                                   ▼
                    ┌────────────────────────────────┐
                    │        Results Channel         │
                    └────────────────────────────────┘
```

---

## 4️⃣ Where Used in Real Systems?

### Image Processing Service

```go
func processImages(images []Image) []Result {
    jobs := make(chan Image, len(images))
    results := make(chan Result, len(images))
    
    // Start workers
    for w := 0; w < runtime.GOMAXPROCS(0); w++ {
        go func() {
            for img := range jobs {
                results <- resize(img)
            }
        }()
    }
    
    // Send jobs
    for _, img := range images {
        jobs <- img
    }
    close(jobs)
    
    // Collect results
    var out []Result
    for range images {
        out = append(out, <-results)
    }
    return out
}
```

---

## 5️⃣ Practical Examples

### Basic Worker Pool

```go
func worker(id int, jobs <-chan int, results chan<- int) {
    for j := range jobs {
        fmt.Printf("Worker %d processing job %d\n", id, j)
        time.Sleep(time.Second)
        results <- j * 2
    }
}

func main() {
    jobs := make(chan int, 100)
    results := make(chan int, 100)
    
    // Start 3 workers
    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }
    
    // Send 9 jobs
    for j := 1; j <= 9; j++ {
        jobs <- j
    }
    close(jobs)
    
    // Collect results
    for a := 1; a <= 9; a++ {
        <-results
    }
}
```

### With Context Cancellation

```go
func worker(ctx context.Context, jobs <-chan Job) {
    for {
        select {
        case <-ctx.Done():
            return
        case job, ok := <-jobs:
            if !ok {
                return
            }
            process(job)
        }
    }
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Forget to close jobs | Close after sending all jobs |
| Wrong result count | Match results to jobs count |
| No context support | Add ctx.Done() select |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Worker Pool:                               │
│ • Fixed number of workers                  │
│ • Jobs channel as queue                    │
│ • Results channel for output               │
│                                            │
│ Benefits:                                  │
│ • Bounded concurrency                      │
│ • Controlled resource usage                │
│ • Backpressure via channel blocking        │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Why use worker pool vs unlimited goroutines?
2. What happens if you forget to close jobs channel?
3. How to choose number of workers?
4. How does backpressure work?
5. How to handle worker errors?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Control resources, prevent exhaustion, bounded memory.

**A2**: Workers block forever on range loop.

**A3**: CPU-bound: GOMAXPROCS. IO-bound: higher (10x-100x CPU count).

**A4**: Full jobs channel blocks sender until workers catch up.

**A5**: Send errors via results channel or dedicated error channel.

</details>

---

[← Previous Module](../08-context/05-context-in-real-applications.md) | [Next →](./02-fan-in-fan-out.md)
