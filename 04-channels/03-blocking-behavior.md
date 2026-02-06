# Blocking Behavior

## 1️⃣ Simple Explanation (Beginner Friendly)

Channel operations block under specific conditions:

| Operation | Blocks When |
|-----------|-------------|
| `ch <- v` (send) | Channel is full or no receiver ready |
| `<-ch` (receive) | Channel is empty |
| `close(ch)` | Never blocks |

---

## 2️⃣ Real-World Analogy

### 🚪 Doorway

- **Send to full channel** = Waiting at busy doorway
- **Receive from empty** = Waiting for someone to arrive
- **Close** = Putting up "no more entries" sign

---

## 3️⃣ Technical Working (Step-by-Step)

### Send Blocking Conditions

```
Unbuffered channel:
ch <- value  →  Blocks until receiver calls <-ch

Buffered channel (cap=3):
[1 2 3]  ← Full
ch <- 4  →  Blocks until space available
```

### Receive Blocking Conditions

```
ch := make(chan int)
<-ch  →  Blocks forever (no sender!)

ch := make(chan int, 3)
<-ch  →  Blocks until sender adds value
```

### Deadlock Example

```go
func main() {
    ch := make(chan int)
    ch <- 1  // DEADLOCK: no receiver, blocks forever
    fmt.Println(<-ch)
}
// fatal error: all goroutines are asleep - deadlock!
```

### Non-Blocking with Select

```go
select {
case ch <- value:
    // Sent successfully
default:
    // Couldn't send (would block)
}
```

---

## 4️⃣ Where Used in Real Systems?

### Timeout Pattern

```go
select {
case result := <-ch:
    process(result)
case <-time.After(5 * time.Second):
    return errors.New("timeout")
}
```

---

## 5️⃣ Practical Examples

### Detecting Deadlock

```go
package main

func main() {
    ch := make(chan int)
    
    // This causes deadlock
    // ch <- 1  
    
    // Fix: use goroutine
    go func() { ch <- 1 }()
    println(<-ch)
}
```

### Non-Blocking Send

```go
select {
case ch <- data:
    fmt.Println("Sent!")
default:
    fmt.Println("Channel full, dropping")
}
```

### Receiving from Closed Channel

```go
ch := make(chan int, 2)
ch <- 1
ch <- 2
close(ch)

fmt.Println(<-ch)  // 1
fmt.Println(<-ch)  // 2
fmt.Println(<-ch)  // 0 (zero value, channel closed)

// Better: check if closed
val, ok := <-ch
if !ok {
    fmt.Println("Channel closed")
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Closing channel unblocks senders | Sending to closed channel panics! |
| Receive from closed returns error | Returns zero value, ok=false |
| Deadlock always panics | Go runtime detects and panics |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Send blocks: full buffer or no receiver    │
│ Receive blocks: empty channel              │
│ Close: never blocks, signals completion    │
│                                            │
│ Avoid deadlock: use goroutines or select   │
│ Non-blocking: select with default          │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What causes a channel deadlock?
2. How do you make a non-blocking send?
3. What does receiving from closed channel return?
4. Can you send to a closed channel?
5. How does Go detect deadlocks?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: All goroutines blocked on channel operations with no way to proceed.

**A2**: Use select with default:
```go
select {
case ch <- v:
default:
}
```

**A3**: Zero value of the channel type, with ok=false.

**A4**: No! It panics. Only receiver should wait for close.

**A5**: Runtime checks if all goroutines are blocked with no way to wake. Panics with "deadlock" message.

</details>

---

[← Previous](./02-unbuffered-vs-buffered.md) | [Next →](./04-select-statement.md)
