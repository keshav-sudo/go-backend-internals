# Unbuffered vs Buffered Channels

## 1️⃣ Simple Explanation (Beginner Friendly)

| Type | Creation | Behavior |
|------|----------|----------|
| **Unbuffered** | `make(chan T)` | Sender blocks until receiver is ready |
| **Buffered** | `make(chan T, n)` | Sender blocks only if buffer is full |

```go
// Unbuffered: synchronous
ch := make(chan int)

// Buffered: async up to capacity
ch := make(chan int, 10)
```

---

## 2️⃣ Real-World Analogy

### 📬 Mail Delivery

**Unbuffered** = Hand-to-hand delivery. Must wait for receiver.

**Buffered** = Mailbox with limited slots. Sender leaves if space exists.

---

## 3️⃣ Technical Working (Step-by-Step)

### Unbuffered Channel

```
Sender                    Channel                   Receiver
   │                         │                          │
   │── send(42) ──►          │                          │
   │                         │ (waits for receiver)     │
   │  BLOCKED                │                          │
   │                         │          ◄── receive ────│
   │                         │                          │
   │── unblocked ──          │──── 42 ────────────────►│
```

### Buffered Channel (cap=3)

```
State 1: Empty [_ _ _]
send(1) → [1 _ _] → immediate return

State 2: Partially full [1 2 _]
send(3) → [1 2 3] → immediate return

State 3: Full [1 2 3]
send(4) → BLOCKS until receive

After receive: [2 3 _]
send(4) → [2 3 4] → unblocked
```

### Internal Structure (hchan)

```go
type hchan struct {
    qcount   uint      // Items in buffer
    dataqsiz uint      // Buffer capacity  
    buf      unsafe.Pointer // Buffer array
    sendx    uint      // Send index
    recvx    uint      // Receive index
    sendq    waitq     // Blocked senders
    recvq    waitq     // Blocked receivers
    lock     mutex     // Protects all fields
}
```

---

## 4️⃣ Where Used in Real Systems?

### Use Cases

| Pattern | Channel Type | Why |
|---------|--------------|-----|
| Signal completion | Unbuffered | Guarantee synchronization |
| Job queue | Buffered | Decouple producer/consumer |
| Semaphore | Buffered | `make(chan struct{}, N)` |
| Rate limiting | Buffered | Token bucket pattern |

---

## 5️⃣ Practical Examples

### Unbuffered for Sync

```go
done := make(chan bool)

go func() {
    doWork()
    done <- true  // Signal completion
}()

<-done  // Wait for signal
```

### Buffered for Throughput

```go
jobs := make(chan Job, 100)

// Producer doesn't block (unless 100 pending)
for _, j := range allJobs {
    jobs <- j
}

// Consumers process independently
for w := 0; w < 5; w++ {
    go func() {
        for j := range jobs {
            process(j)
        }
    }()
}
```

### Semaphore Pattern

```go
// Limit concurrent operations to 10
sem := make(chan struct{}, 10)

for _, item := range items {
    sem <- struct{}{}  // Acquire
    go func(i Item) {
        defer func() { <-sem }()  // Release
        process(i)
    }(item)
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Buffered channels prevent blocking | They only delay blocking |
| Unbuffered = sync, buffered = async | Buffered is still blocking when full |
| Bigger buffer = better | Can hide backpressure problems |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Unbuffered: Synchronous handoff            │
│ • Sender waits for receiver                │
│ • Use for: synchronization, signaling      │
│                                            │
│ Buffered: Async up to capacity             │
│ • Sender blocks only when full             │
│ • Use for: queues, rate limiting           │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What happens when you send to a full buffered channel?
2. When would unbuffered be preferred over buffered?
3. How do you create a channel that acts as a semaphore?
4. What's the capacity of `make(chan int)`?
5. Can a buffered channel have capacity 0?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Sender blocks until a receiver takes a value.

**A2**: When you need guaranteed synchronization between sender and receiver.

**A3**: `make(chan struct{}, N)` - N is the concurrency limit.

**A4**: 0 - it's unbuffered.

**A5**: No. `make(chan int, 0)` is same as unbuffered.

</details>

---

[← Previous](./01-why-channels-exist.md) | [Next →](./03-blocking-behavior.md)
