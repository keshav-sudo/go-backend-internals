# Goroutine Leaks

## 1️⃣ Simple Explanation (Beginner Friendly)

A **goroutine leak** occurs when a goroutine is started but never finishes. It stays blocked forever, consuming memory and resources.

```go
// LEAK: This goroutine waits forever!
go func() {
    ch := make(chan int)
    val := <-ch  // Nobody sends, blocks forever
    fmt.Println(val)
}()
```

---

## 2️⃣ Real-World Analogy

### 🚿 Dripping Faucet

A goroutine leak is like a dripping faucet - each drip (goroutine) is small, but over time your water bill (memory) explodes.

---

## 3️⃣ Technical Working (Step-by-Step)

### Common Leak Patterns

```
Pattern 1: Blocked Channel Receive
┌──────────────────────┐
│  go func() {         │
│      <-ch            │ ← No sender, waits forever
│  }()                 │
└──────────────────────┘

Pattern 2: Blocked Channel Send
┌──────────────────────┐
│  go func() {         │
│      ch <- value     │ ← No receiver, waits forever
│  }()                 │
└──────────────────────┘

Pattern 3: Forgotten Goroutine
┌──────────────────────┐
│  go func() {         │
│      for {           │
│          // work     │ ← No exit condition
│      }               │
│  }()                 │
└──────────────────────┘
```

### Memory Impact

```
Each leaked goroutine costs:
┌────────────────────────┐
│ G struct:    ~400 bytes│
│ Stack:       ~2KB min  │
│ Total:       ~2.5KB    │
└────────────────────────┘

10,000 leaks = 25MB wasted
1,000,000 leaks = 2.5GB wasted → OOM!
```

---

## 4️⃣ Where Used in Real Systems?

### HTTP Request Handler Leak

```go
// LEAK: If client disconnects, goroutine waits forever
func handler(w http.ResponseWriter, r *http.Request) {
    result := make(chan string)
    
    go func() {
        result <- slowOperation()  // Client gone, nobody reads
    }()
    
    // If timeout, we return but goroutine stays!
    select {
    case res := <-result:
        fmt.Fprint(w, res)
    case <-time.After(5 * time.Second):
        http.Error(w, "timeout", 504)
    }
}
```

---

## 5️⃣ Practical Examples

### Detecting Leaks

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func leakyFunction() {
    ch := make(chan int)
    go func() {
        <-ch  // LEAK: blocks forever
    }()
}

func main() {
    fmt.Printf("Before: %d goroutines\n", runtime.NumGoroutine())
    
    for i := 0; i < 100; i++ {
        leakyFunction()
    }
    
    time.Sleep(100 * time.Millisecond)
    fmt.Printf("After: %d goroutines\n", runtime.NumGoroutine())
    // Output: 101 goroutines (100 leaked!)
}
```

### Fixing with Context

```go
func fixedFunction(ctx context.Context) {
    ch := make(chan int)
    
    go func() {
        select {
        case val := <-ch:
            fmt.Println(val)
        case <-ctx.Done():
            return  // Clean exit!
        }
    }()
}
```

### Detection Tools

```bash
# Monitor goroutine count
watch -n 1 'curl -s localhost:6060/debug/pprof/goroutine?debug=0 | head -1'

# Dump all goroutine stacks
curl localhost:6060/debug/pprof/goroutine?debug=2 > goroutines.txt

# Use goleak in tests
go get go.uber.org/goleak
```

```go
// In tests
func TestNoLeak(t *testing.T) {
    defer goleak.VerifyNone(t)
    // ... test code
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| GC collects blocked goroutines | GC cannot collect waiting goroutines |
| Only channels cause leaks | Mutexes, WaitGroup misuse also leak |
| Small leaks are fine | Leaks accumulate over time |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Common causes:                             │
│ • Blocked channel operations               │
│ • Missing context cancellation             │
│ • Infinite loops without exit              │
│                                            │
│ Prevention:                                │
│ • Always use context.Context               │
│ • Use select with done channels            │
│ • Test with goleak                         │
│ • Monitor runtime.NumGoroutine()           │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Why can't the garbage collector clean up leaked goroutines?
2. How do you detect a goroutine leak in production?
3. What's the pattern to prevent channel-based leaks?
4. Can a goroutine blocked on mutex cause a leak?
5. How does uber/goleak detect leaks in tests?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: GC only collects unreachable memory. Blocked goroutines are still "alive" (referenced by scheduler).

**A2**: Monitor `runtime.NumGoroutine()` over time. Use pprof goroutine endpoint. Set up alerts.

**A3**: Use `select` with `ctx.Done()` or timeout:
```go
select {
case <-ch:
case <-ctx.Done():
    return
}
```

**A4**: Yes! Goroutine waiting on `Lock()` that's never released = leak.

**A5**: Compares goroutine count before/after test. Extra goroutines (except background) = leak.

</details>

---

[← Previous](./04-goroutine-lifecycle.md) | [Next Module: Go Scheduler →](../03-go-scheduler/01-mpg-model.md)
