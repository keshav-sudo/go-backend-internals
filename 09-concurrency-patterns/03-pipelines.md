# Pipelines

## 1️⃣ Simple Explanation (Beginner Friendly)

A **pipeline** is a series of stages connected by channels. Each stage:
1. Receives values from upstream
2. Performs processing
3. Sends results downstream

```
Stage 1 ──► Stage 2 ──► Stage 3
(generate)   (square)   (print)
```

---

## 2️⃣ Real-World Analogy

### 🏭 Assembly Line

Car factory: chassis → engine → paint → wheels → quality check.
Each station processes and passes to next.

---

## 3️⃣ Technical Working (Step-by-Step)

### Pipeline Structure

```
┌──────────────────────────────────────────────────────────┐
│                    PIPELINE                              │
│                                                          │
│  ┌───────────┐     ┌───────────┐     ┌───────────┐      │
│  │   GEN     │────►│  PROCESS  │────►│  OUTPUT   │      │
│  │ (source)  │ ch1 │ (transform)│ ch2 │  (sink)   │      │
│  └───────────┘     └───────────┘     └───────────┘      │
│                                                          │
│  Each stage owns its output channel                      │
└──────────────────────────────────────────────────────────┘
```

### Stage Template

```go
func stage(input <-chan T) <-chan T {
    output := make(chan T)
    go func() {
        defer close(output)  // Owner closes
        for v := range input {
            output <- transform(v)
        }
    }()
    return output
}
```

---

## 4️⃣ Where Used in Real Systems?

### Data Processing Pipeline

```go
func main() {
    // Pipeline: read → decode → validate → store
    files := listFiles(dir)
    decoded := decode(files)
    validated := validate(decoded)
    results := store(validated)
    
    for r := range results {
        log.Println(r)
    }
}
```

---

## 5️⃣ Practical Examples

### Number Processing Pipeline

```go
func gen(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            out <- n
        }
    }()
    return out
}

func square(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * n
        }
    }()
    return out
}

func main() {
    // Build pipeline
    nums := gen(1, 2, 3, 4, 5)
    squared := square(nums)
    
    // Consume
    for s := range squared {
        fmt.Println(s)  // 1, 4, 9, 16, 25
    }
}
```

### With Cancellation

```go
func gen(ctx context.Context, nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            select {
            case <-ctx.Done():
                return
            case out <- n:
            }
        }
    }()
    return out
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Forget to close channels | Each stage closes its output |
| No cancellation support | Add ctx.Done() to all stages |
| Goroutine leak on error | Use context for cleanup |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Pipeline: chain of processing stages       │
│                                            │
│ Rules:                                     │
│ • Each stage owns its output channel       │
│ • Stage closes channel when done           │
│ • Support cancellation via context         │
│ • Stages run concurrently                  │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Who closes the output channel of a stage?
2. How to stop a pipeline early?
3. Do pipeline stages run in parallel?
4. How to add buffering?
5. Pipeline vs fan-out difference?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: The stage that created it (owner closes).

**A2**: Use context cancellation. All stages check ctx.Done().

**A3**: Yes! Each stage runs in its own goroutine.

**A4**: Create buffered channels: `make(chan T, size)`

**A5**: Pipeline = sequential stages. Fan-out = parallel workers.

</details>

---

[← Previous](./02-fan-in-fan-out.md) | [Next →](./04-rate-limiting.md)
