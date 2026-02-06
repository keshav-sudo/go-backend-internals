# WaitGroup

## 1️⃣ Simple Explanation (Beginner Friendly)

`sync.WaitGroup` waits for a collection of goroutines to finish.

```go
var wg sync.WaitGroup

wg.Add(1)        // Increment counter
go func() {
    defer wg.Done()  // Decrement counter
    doWork()
}()

wg.Wait()  // Block until counter is 0
```

---

## 2️⃣ Real-World Analogy

### 🚌 School Bus

The bus (main goroutine) waits until all students (goroutines) are on board before leaving.

---

## 3️⃣ Technical Working (Step-by-Step)

### Counter Mechanism

```
wg.Add(3):  counter = 3
go work1()
go work2()
go work3()

work1 done: wg.Done() → counter = 2
work2 done: wg.Done() → counter = 1
work3 done: wg.Done() → counter = 0

wg.Wait() unblocks when counter = 0
```

### State Diagram

```
           Add(n)
    [0] ──────────► [n]
     ▲                │
     │                │ Done() × n
     │                ▼
     └──────────── [0] ← Wait() unblocks
```

---

## 4️⃣ Where Used in Real Systems?

### Parallel Processing

```go
func processItems(items []Item) {
    var wg sync.WaitGroup
    
    for _, item := range items {
        wg.Add(1)
        go func(i Item) {
            defer wg.Done()
            process(i)
        }(item)
    }
    
    wg.Wait()  // All items processed
}
```

---

## 5️⃣ Practical Examples

### Basic Usage

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    var wg sync.WaitGroup
    
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go func(n int) {
            defer wg.Done()
            time.Sleep(time.Duration(n) * 100 * time.Millisecond)
            fmt.Printf("Worker %d done\n", n)
        }(i)
    }
    
    wg.Wait()
    fmt.Println("All workers done")
}
```

### Common Bug: Add in Goroutine

```go
// BUG: Add after goroutine might race
for i := 0; i < n; i++ {
    go func() {
        wg.Add(1)  // WRONG: might happen after Wait()
        wg.Done()
    }()
}
wg.Wait()

// FIX: Add before goroutine
for i := 0; i < n; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
    }()
}
wg.Wait()
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Add() inside goroutine | Add() before starting goroutine |
| Forget Done() | Use defer wg.Done() |
| Negative counter | Panics! Ensure Add/Done balance |
| Copy WaitGroup | Pass by pointer only |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ • Add(n): increment counter by n           │
│ • Done(): decrement counter by 1           │
│ • Wait(): block until counter = 0          │
│                                            │
│ Always: Add() before goroutine starts      │
│ Always: defer wg.Done() for safety         │
│ Never: copy WaitGroup (pass pointer)       │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What happens if Done() is called more than Add()?
2. Why should Add() be called before starting the goroutine?
3. Can WaitGroup be reused after Wait() returns?
4. What's the correct way to pass WaitGroup to functions?
5. Why use defer for Done()?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Panic! Counter becomes negative.

**A2**: Otherwise Wait() might return before Add() is called (race condition).

**A3**: Yes, once counter is 0 and Wait() returns, you can Add() again.

**A4**: Pass by pointer: `func worker(wg *sync.WaitGroup)`

**A5**: Ensures Done() is called even if function panics or returns early.

</details>

---

[← Previous](./01-mutex-rwmutex.md) | [Next →](./03-cond-variables.md)
