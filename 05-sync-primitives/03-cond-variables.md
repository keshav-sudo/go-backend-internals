# Cond Variables

## 1️⃣ Simple Explanation (Beginner Friendly)

`sync.Cond` allows goroutines to wait for a condition to become true. It's used with a Mutex.

```go
var mu sync.Mutex
cond := sync.NewCond(&mu)

// Waiting goroutine
mu.Lock()
for !condition {
    cond.Wait()  // Releases lock, waits, reacquires
}
mu.Unlock()

// Signaling goroutine
mu.Lock()
condition = true
cond.Signal()  // Wake one waiter
mu.Unlock()
```

---

## 2️⃣ Real-World Analogy

### 🔔 Dinner Bell

Kids (goroutines) wait for dinner. Parent rings bell (Signal/Broadcast) when ready.

---

## 3️⃣ Technical Working (Step-by-Step)

### Wait() Internals

```
cond.Wait():
┌─────────────────────────────────┐
│ 1. Release associated mutex     │
│ 2. Add goroutine to wait queue  │
│ 3. Park goroutine (sleep)       │
│ 4. On wake: reacquire mutex     │
└─────────────────────────────────┘
```

### Signal vs Broadcast

```
Signal():   Wake ONE waiting goroutine
Broadcast(): Wake ALL waiting goroutines
```

---

## 4️⃣ Where Used in Real Systems?

### Producer-Consumer Queue

```go
type Queue struct {
    items []int
    cond  *sync.Cond
    mu    sync.Mutex
}

func (q *Queue) Put(item int) {
    q.mu.Lock()
    q.items = append(q.items, item)
    q.cond.Signal()  // Wake one consumer
    q.mu.Unlock()
}

func (q *Queue) Get() int {
    q.mu.Lock()
    for len(q.items) == 0 {
        q.cond.Wait()
    }
    item := q.items[0]
    q.items = q.items[1:]
    q.mu.Unlock()
    return item
}
```

---

## 5️⃣ Practical Examples

### Basic Condition Variable

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    var mu sync.Mutex
    cond := sync.NewCond(&mu)
    ready := false
    
    go func() {
        mu.Lock()
        for !ready {
            cond.Wait()
        }
        fmt.Println("Condition met!")
        mu.Unlock()
    }()
    
    time.Sleep(100 * time.Millisecond)
    mu.Lock()
    ready = true
    cond.Signal()
    mu.Unlock()
    
    time.Sleep(100 * time.Millisecond)
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| if condition { Wait() } | MUST use for loop (spurious wakeups) |
| Wait() without holding lock | Must hold lock before Wait() |
| Forget to recheck condition | Always loop and recheck after wakeup |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ • Wait(): release lock, wait, reacquire    │
│ • Signal(): wake one waiter                │
│ • Broadcast(): wake all waiters            │
│                                            │
│ Always use: for !condition { Wait() }      │
│ Channels often simpler for Go              │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Why use `for` instead of `if` before Wait()?
2. What does Wait() do with the mutex?
3. When to use Broadcast vs Signal?
4. Can you use Cond without a Mutex?
5. Are channels preferred over Cond in Go?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Spurious wakeups can occur. Must recheck condition after wakeup.

**A2**: Releases it before sleeping, reacquires after waking.

**A3**: Signal: wake one (first waiter). Broadcast: wake all (when multiple need to check).

**A4**: No. Cond requires a Locker (Mutex or RWMutex).

**A5**: Usually yes. Channels are more idiomatic. Use Cond for specific patterns.

</details>

---

[← Previous](./02-waitgroup.md) | [Next →](./04-atomic-operations.md)
