# Semaphores

## 1️⃣ Simple Explanation (Beginner Friendly)

A **semaphore** limits concurrent access to a resource. In Go, typically implemented with buffered channels.

```go
sem := make(chan struct{}, 10)  // Max 10 concurrent

sem <- struct{}{}    // Acquire (block if full)
doWork()
<-sem               // Release
```

---

## 2️⃣ Real-World Analogy

### 🅿️ Parking Lot

Semaphore = parking lot with N spaces. Cars wait at entrance when full. Car leaving = space available.

---

## 3️⃣ Technical Working (Step-by-Step)

### Channel-Based Semaphore

```
Semaphore(3):       [_][_][_]  (3 slots)

Worker 1 acquire:   [X][_][_]  (2 left)
Worker 2 acquire:   [X][X][_]  (1 left)
Worker 3 acquire:   [X][X][X]  (0 left)
Worker 4 acquire:   BLOCKS     (waiting)

Worker 1 release:   [X][X][_]  (1 available)
Worker 4 unblocks:  [X][X][X]  (proceeds)
```

### Weighted Semaphore (x/sync/semaphore)

```go
sem := semaphore.NewWeighted(100)  // 100 units

// Acquire 10 units
sem.Acquire(ctx, 10)
defer sem.Release(10)
```

---

## 4️⃣ Where Used in Real Systems?

### Database Connection Limiting

```go
var dbSem = make(chan struct{}, 25)  // Max 25 connections

func queryDB(query string) (Result, error) {
    dbSem <- struct{}{}
    defer func() { <-dbSem }()
    
    return db.Query(query)
}
```

---

## 5️⃣ Practical Examples

### Basic Semaphore

```go
func processWithLimit(items []Item, maxConcurrent int) {
    sem := make(chan struct{}, maxConcurrent)
    var wg sync.WaitGroup
    
    for _, item := range items {
        wg.Add(1)
        sem <- struct{}{}  // Acquire
        
        go func(i Item) {
            defer wg.Done()
            defer func() { <-sem }()  // Release
            
            process(i)
        }(item)
    }
    
    wg.Wait()
}
```

### Weighted Semaphore

```go
import "golang.org/x/sync/semaphore"

func main() {
    ctx := context.Background()
    sem := semaphore.NewWeighted(100)
    
    // Large job needs 50 units
    sem.Acquire(ctx, 50)
    defer sem.Release(50)
    
    processLargeJob()
}
```

### Timeout on Acquire

```go
select {
case sem <- struct{}{}:
    defer func() { <-sem }()
    doWork()
case <-time.After(time.Second):
    return errors.New("timeout acquiring semaphore")
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Forget to release | Use defer for release |
| Release wrong amount | Weighted: release what you acquired |
| Nil channel semaphore | Must initialize with make |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Semaphore: limit concurrent access         │
│                                            │
│ Channel-based:                             │
│ • sem := make(chan struct{}, N)            │
│ • Acquire: sem <- struct{}{}               │
│ • Release: <-sem                           │
│                                            │
│ Weighted (x/sync/semaphore):               │
│ • Acquire(ctx, weight)                     │
│ • Release(weight)                          │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Why use struct{} for semaphore?
2. How to implement timeout on acquire?
3. When to use weighted semaphore?
4. What if you release more than acquire?
5. Semaphore vs worker pool difference?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Zero memory cost (empty struct = 0 bytes).

**A2**: Select with time.After or context with timeout.

**A3**: When operations have different resource costs (big queries vs small).

**A4**: Channel: can receive more (creates slack). Weighted: panics.

**A5**: Semaphore: limits access. Worker pool: has actual workers processing.

</details>

---

[← Previous](./04-rate-limiting.md) | [Next Module: Advanced Systems →](../10-advanced-systems/01-building-high-performance-servers.md)
