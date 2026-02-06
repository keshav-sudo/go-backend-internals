# Mutex & RWMutex

## 1️⃣ Simple Explanation (Beginner Friendly)

**Mutex** = Mutual Exclusion lock. Only one goroutine can hold it.

**RWMutex** = Read-Write Mutex. Multiple readers OR one writer.

```go
var mu sync.Mutex
mu.Lock()
// Critical section
mu.Unlock()

var rw sync.RWMutex
rw.RLock()   // Multiple can hold this
rw.RUnlock()
rw.Lock()    // Only one can hold this
rw.Unlock()
```

---

## 2️⃣ Real-World Analogy

### 🚪 Bathroom vs Library

**Mutex** = Single bathroom. One person at a time.

**RWMutex** = Library. Many can read books; only one can rearrange shelves.

---

## 3️⃣ Technical Working (Step-by-Step)

### Mutex States

```
Unlocked:  [      ]  → Lock() → Locked: [G1    ]
                                  ↓
                         G2 calls Lock()
                                  ↓
Locked:    [G1    ]  G2 waits in queue
                                  ↓
                         G1 calls Unlock()
                                  ↓
Locked:    [G2    ]  G2 acquires lock
```

### RWMutex Logic

```
┌────────────────────────────────────────────┐
│ Readers: 0, Writer: none → Anyone can lock │
│ Readers: 3, Writer: none → More R, no W    │
│ Readers: 0, Writer: 1    → No R, No more W │
│ Writer waiting: new R blocked              │
└────────────────────────────────────────────┘
```

---

## 4️⃣ Where Used in Real Systems?

### Thread-Safe Cache

```go
type Cache struct {
    mu   sync.RWMutex
    data map[string]string
}

func (c *Cache) Get(key string) (string, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    v, ok := c.data[key]
    return v, ok
}

func (c *Cache) Set(key, value string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.data[key] = value
}
```

---

## 5️⃣ Practical Examples

### Mutex for Counter

```go
type Counter struct {
    mu    sync.Mutex
    value int
}

func (c *Counter) Inc() {
    c.mu.Lock()
    c.value++
    c.mu.Unlock()
}
```

### Deadlock Example

```go
// DEADLOCK: Lock twice without unlock
var mu sync.Mutex
mu.Lock()
mu.Lock()  // Blocks forever!
```

### Detecting with Race Detector

```bash
go run -race main.go
# Reports if locks are used incorrectly
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Lock is reentrant | Go Mutex is NOT reentrant |
| Unlock from different G | Same G must lock and unlock |
| Always use RWMutex | For read-heavy workloads only |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Mutex: Lock() / Unlock() - exclusive       │
│ RWMutex: RLock() for readers, Lock() write │
│                                            │
│ Use defer Unlock() to avoid leaks          │
│ Not reentrant - don't double lock!         │
│ RWMutex good for read-heavy workloads      │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What happens if you call Lock() twice without Unlock()?
2. When is RWMutex better than Mutex?
3. Can a different goroutine call Unlock()?
4. What does defer Unlock() prevent?
5. Is Go's Mutex reentrant?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Deadlock. The goroutine waits forever.

**A2**: When reads heavily outnumber writes. Multiple readers can proceed.

**A3**: Technically yes, but it's bad practice. Same G should lock/unlock.

**A4**: Forgetting to unlock, especially when function returns early.

**A5**: No! Cannot lock twice from same goroutine.

</details>

---

[← Previous Module](../04-channels/05-channel-internals.md) | [Next →](./02-waitgroup.md)
