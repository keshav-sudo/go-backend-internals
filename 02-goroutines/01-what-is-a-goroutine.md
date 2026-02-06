# What is a Goroutine?

## 1️⃣ Simple Explanation (Beginner Friendly)

A **goroutine** is Go's way of running functions concurrently. Think of it as a lightweight "worker" that can execute code independently of other workers.

```go
// Start a goroutine with the 'go' keyword
go doSomething()

// This continues immediately, doesn't wait
fmt.Println("I don't wait!")
```

Key characteristics:
- **Lightweight**: ~2KB initial stack (vs ~1MB for OS threads)
- **Cheap to create**: Millions can exist in a single program
- **Managed by Go runtime**: Not directly by the operating system

---

## 2️⃣ Real-World Analogy

### 🍳 Restaurant Kitchen

Imagine a busy restaurant:

- **Without goroutines**: One chef cooks each dish completely before starting the next. Slow!
- **With goroutines**: Multiple tasks happen simultaneously - one chef preps, another grills, another plates.

```
Without concurrency:          With goroutines:
Order 1 ████████              Order 1 ████
Order 2         ████████      Order 2   ████
Order 3                 ████  Order 3     ████
                              ─────────────────
Time: 24 units                Time: 8 units
```

---

## 3️⃣ Technical Working (Step-by-Step)

### Goroutine Structure (G struct)

```
┌─────────────────────────────────────┐
│           G (Goroutine)             │
├─────────────────────────────────────┤
│ stack      → pointer to stack       │
│ stackguard → stack overflow check   │
│ status     → running/waiting/dead   │
│ m          → current M (thread)     │
│ sched      → saved registers (PC,SP)│
│ atomicstatus → atomic state field   │
│ goid       → goroutine ID           │
└─────────────────────────────────────┘
```

### Creating a Goroutine

```
go myFunc(args)
       │
       ▼
┌─────────────────────┐
│ runtime.newproc()   │
├─────────────────────┤
│ 1. Allocate G struct│
│ 2. Allocate 2KB stack│
│ 3. Initialize state │
│ 4. Add to run queue │
└─────────────────────┘
       │
       ▼
   Scheduler picks it up when ready
```

### Goroutine States

```
      ┌──────────┐
      │  Gidle   │  (newly created)
      └────┬─────┘
           ▼
      ┌──────────┐
 ┌───►│ Grunnable│◄─────────────┐
 │    └────┬─────┘              │
 │         ▼                    │
 │    ┌──────────┐              │
 │    │ Grunning │ (on CPU)     │
 │    └────┬─────┘              │
 │         │                    │
 │    ┌────┴────┐               │
 │    ▼         ▼               │
 │ ┌──────┐ ┌────────┐          │
 │ │Gwait │ │Gsyscall│──────────┘
 │ └──┬───┘ └────────┘
 │    │
 └────┘
      │
      ▼
   ┌──────┐
   │ Gdead│ (finished)
   └──────┘
```

---

## 4️⃣ Where Used in Real Systems?

### HTTP Server: One Goroutine per Request

```go
func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}

// Each request = new goroutine
func handler(w http.ResponseWriter, r *http.Request) {
    // This runs in its own goroutine
    data := fetchFromDB()
    json.NewEncoder(w).Respond(data)
}
```

### Real-world Usage:
- **10K concurrent requests** = 10K goroutines = ~20MB memory
- Same in Java = ~10GB memory (thread per request)

---

## 5️⃣ Practical Examples

### Basic Goroutine

```go
package main

import (
    "fmt"
    "time"
)

func sayHello(name string) {
    fmt.Printf("Hello, %s!\n", name)
}

func main() {
    go sayHello("World")  // Runs concurrently
    
    fmt.Println("Main continues...")
    time.Sleep(100 * time.Millisecond)  // Wait for goroutine
}
```

### Counting Goroutines

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func main() {
    fmt.Printf("Initial goroutines: %d\n", runtime.NumGoroutine())
    
    for i := 0; i < 100; i++ {
        go func() {
            time.Sleep(time.Second)
        }()
    }
    
    fmt.Printf("After spawning: %d\n", runtime.NumGoroutine())
}
```

### Common Mistake: Goroutine with Loop Variable

```go
// BUG: All goroutines print the same value!
for i := 0; i < 5; i++ {
    go func() {
        fmt.Println(i)  // Captures variable, not value
    }()
}

// FIX: Pass as parameter
for i := 0; i < 5; i++ {
    go func(n int) {
        fmt.Println(n)  // Each gets its own copy
    }(i)
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Goroutines are threads | Goroutines are multiplexed onto threads |
| `go func()` blocks until complete | It returns immediately |
| Goroutines share nothing | They share memory (need sync!) |
| More goroutines = faster | I/O bound: yes. CPU bound: limited by cores |
| Goroutine order is deterministic | Order is non-deterministic |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ • Created with: go functionCall()          │
│ • Initial stack: ~2KB (grows as needed)    │
│ • Managed by Go runtime scheduler          │
│ • States: runnable → running → waiting     │
│ • Use sync.WaitGroup to wait for them      │
│ • Always capture loop variables properly!  │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What's the initial stack size of a goroutine?
2. Does `go func()` block the calling code?
3. Why is this buggy? `for i := range items { go process(i) }`
4. How do you wait for a goroutine to complete?
5. What happens to running goroutines when `main()` exits?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: ~2KB initial stack, can grow dynamically up to 1GB.

**A2**: No! It returns immediately. The goroutine runs concurrently.

**A3**: Not buggy if `i` is the value (range copies). But `for i := 0; i < n; i++` with `go func() { use(i) }()` captures variable address.

**A4**: Use `sync.WaitGroup`:
```go
var wg sync.WaitGroup
wg.Add(1)
go func() { defer wg.Done(); work() }()
wg.Wait()
```

**A5**: They're killed immediately! No cleanup, no deferred functions run.

</details>

---

[← Previous Module](../01-go-runtime/05-comparison-with-nodejs-java.md) | [Next →](./02-goroutines-are-lightweight.md)
