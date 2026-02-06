# Scheduler Blocking Flow

## 1️⃣ Simple Explanation (Beginner Friendly)

When a goroutine blocks (channel, lock, I/O), the scheduler doesn't waste the CPU. It:
1. Parks the blocked goroutine
2. Picks another runnable goroutine
3. Continues execution

---

## 2️⃣ Real-World Analogy

### 🚗 Valet Parking

When a customer (G) says "I'm waiting for my friend," the valet (scheduler):
1. Parks their car (saves G state)
2. Helps the next customer
3. Brings their car back when friend arrives

---

## 3️⃣ Technical Working (Step-by-Step)

### Blocking on Channel Receive

```
G1 executes: <-ch (empty channel)
       │
       ▼
┌─────────────────────────────────────┐
│ 1. G1 state → Gwaiting              │
│ 2. G1 added to channel's wait queue │
│ 3. M detaches from G1               │
│ 4. M picks new G from P's queue     │
│ 5. M runs new G                     │
└─────────────────────────────────────┘

Later: G2 sends to ch
       │
       ▼
┌─────────────────────────────────────┐
│ 1. G1 removed from wait queue       │
│ 2. G1 state → Grunnable             │
│ 3. G1 added to P's run queue        │
│ 4. G1 will be scheduled next        │
└─────────────────────────────────────┘
```

### Blocking on Mutex

```
G tries: mutex.Lock() (already locked)
       │
       ▼
┌─────────────────────────────────────┐
│ 1. G state → Gwaiting               │
│ 2. G added to mutex's wait queue    │
│ 3. Scheduler runs another G         │
└─────────────────────────────────────┘

Owner calls: mutex.Unlock()
       │
       ▼
┌─────────────────────────────────────┐
│ 1. Wake one G from wait queue       │
│ 2. G state → Grunnable              │
│ 3. G added to run queue             │
└─────────────────────────────────────┘
```

### State Diagram

```
          Block (chan/mutex/cond)
  Grunning ──────────────────────► Gwaiting
     ▲                                  │
     │                                  │ Wake (data/unlock/signal)
     │                                  ▼
     └────────────────────────────── Grunnable
              Scheduled
```

---

## 4️⃣ Where Used in Real Systems?

### Efficient I/O Handling

```go
func handler(w http.ResponseWriter, r *http.Request) {
    data := fetchFromDB()  // G blocks, M runs others
    process(data)          // G resumes
    respond(w)             // G blocks again
}
// M stays busy even when Gs wait!
```

---

## 5️⃣ Practical Examples

### Watching Block/Unblock

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func main() {
    runtime.GOMAXPROCS(1)
    ch := make(chan int)
    
    go func() {
        fmt.Println("G1: waiting...")
        <-ch  // Block here
        fmt.Println("G1: received!")
    }()
    
    go func() {
        fmt.Println("G2: sleeping...")
        time.Sleep(100 * time.Millisecond)
        fmt.Println("G2: sending...")
        ch <- 1
    }()
    
    time.Sleep(200 * time.Millisecond)
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| M waits with blocked G | M detaches, runs other Gs |
| Blocking wastes CPU | CPU used for other work |
| Wake order is FIFO | Implementation-specific |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Blocking G: state → Gwaiting, M detaches   │
│ M picks another G from run queue           │
│ When unblocked: G → Grunnable → run queue  │
│ No CPU wasted during blocking!             │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What happens to M when G blocks on a channel?
2. Where is a blocked G stored?
3. What state is a blocked goroutine in?
4. Who wakes a blocked goroutine?
5. Is blocking on a channel expensive?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: M detaches from G, picks another G from P's run queue, continues work.

**A2**: On the wait queue of the blocking primitive (channel, mutex, etc.).

**A3**: `Gwaiting` - waiting for some event.

**A4**: The goroutine that provides the resource (sender for channel, unlocker for mutex).

**A5**: No! G parks efficiently. M stays productive. This is Go's efficiency secret.

</details>

---

[← Previous](./03-preemption.md) | [Next →](./05-syscalls-scheduler-interaction.md)
