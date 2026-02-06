# Atomic Operations

## 1️⃣ Simple Explanation (Beginner Friendly)

Atomic operations are **indivisible** - they complete entirely or not at all. No locks needed!

```go
import "sync/atomic"

var counter int64

atomic.AddInt64(&counter, 1)      // Atomic increment
val := atomic.LoadInt64(&counter) // Atomic read
atomic.StoreInt64(&counter, 0)    // Atomic write
```

---

## 2️⃣ Real-World Analogy

### 💡 Light Switch

Atomic = light switch. It's either on or off. You can't catch it "partially flipped."

---

## 3️⃣ Technical Working (Step-by-Step)

### Common Atomic Operations

```go
// sync/atomic package
AddInt64(&val, delta)    // Add and return new value
LoadInt64(&val)          // Atomic read
StoreInt64(&val, new)    // Atomic write
SwapInt64(&val, new)     // Swap and return old
CompareAndSwapInt64(&val, old, new)  // CAS
```

### CAS (Compare-And-Swap)

```
CompareAndSwap(addr, old, new):
┌─────────────────────────────────────┐
│ if *addr == old {                   │
│     *addr = new                     │
│     return true                     │
│ }                                   │
│ return false                        │
└─────────────────────────────────────┘
All done atomically by CPU
```

---

## 4️⃣ Where Used in Real Systems?

### Lock-Free Counter

```go
type Counter struct {
    value int64
}

func (c *Counter) Inc() {
    atomic.AddInt64(&c.value, 1)
}

func (c *Counter) Get() int64 {
    return atomic.LoadInt64(&c.value)
}
```

---

## 5️⃣ Practical Examples

### Atomic Counter

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

func main() {
    var counter int64
    var wg sync.WaitGroup
    
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            atomic.AddInt64(&counter, 1)
        }()
    }
    
    wg.Wait()
    fmt.Println("Counter:", counter)  // Always 1000
}
```

### atomic.Value for Any Type

```go
var config atomic.Value

// Store
config.Store(map[string]string{"key": "value"})

// Load
cfg := config.Load().(map[string]string)
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| counter++ is atomic | Not atomic! Use AddInt64 |
| All reads are safe | Use Load for concurrent reads |
| Atomics replace all locks | Only for simple operations |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ • Load/Store: atomic read/write            │
│ • Add: atomic increment                    │
│ • CAS: conditional update                  │
│ • atomic.Value: any type storage           │
│                                            │
│ Faster than mutex for simple counters      │
│ But limited to single-variable operations  │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Is `counter++` atomic in Go?
2. What does CompareAndSwap return?
3. When to use atomic vs mutex?
4. How to atomically update a struct?
5. Are atomic operations lock-free?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: No! It's read-modify-write (not atomic). Use atomic.AddInt64.

**A2**: bool - true if swap succeeded, false if current value didn't match expected.

**A3**: Atomic: simple counters, flags. Mutex: multiple values, complex operations.

**A4**: Use atomic.Value or atomic.Pointer[T] (Go 1.19+).

**A5**: Yes, they use CPU instructions without locks (CAS, LOCK prefix).

</details>

---

[← Previous](./03-cond-variables.md) | [Next →](./05-when-not-to-use-channels.md)
