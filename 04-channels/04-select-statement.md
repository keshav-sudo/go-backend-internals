# Select Statement

## 1️⃣ Simple Explanation (Beginner Friendly)

`select` lets you wait on **multiple channel operations** simultaneously. It's like a switch statement for channels.

```go
select {
case msg := <-ch1:
    fmt.Println("From ch1:", msg)
case msg := <-ch2:
    fmt.Println("From ch2:", msg)
case <-time.After(5 * time.Second):
    fmt.Println("Timeout!")
}
```

---

## 2️⃣ Real-World Analogy

### 🎧 Customer Service

Select is like waiting for calls on multiple lines - answer whichever rings first!

---

## 3️⃣ Technical Working (Step-by-Step)

### Select Behavior

```
┌────────────────────────────────────────────┐
│ 1. Evaluate all channels                   │
│ 2. If multiple ready → pick random one    │
│ 3. If none ready → block (or run default) │
│ 4. Execute chosen case                     │
└────────────────────────────────────────────┘
```

### Patterns

```go
// Non-blocking (with default)
select {
case v := <-ch:
    process(v)
default:
    // No value ready
}

// Timeout
select {
case v := <-ch:
    process(v)
case <-time.After(time.Second):
    handleTimeout()
}

// Cancellation
select {
case <-ctx.Done():
    return ctx.Err()
case result := <-resultCh:
    return result, nil
}
```

---

## 4️⃣ Where Used in Real Systems?

### HTTP Handler with Timeout

```go
func handler(w http.ResponseWriter, r *http.Request) {
    result := make(chan Data)
    go func() { result <- fetchData() }()
    
    select {
    case data := <-result:
        json.NewEncoder(w).Encode(data)
    case <-r.Context().Done():
        http.Error(w, "cancelled", 499)
    case <-time.After(10 * time.Second):
        http.Error(w, "timeout", 504)
    }
}
```

---

## 5️⃣ Practical Examples

### Fan-In (Merge Channels)

```go
func fanIn(ch1, ch2 <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for {
            select {
            case v := <-ch1:
                out <- v
            case v := <-ch2:
                out <- v
            }
        }
    }()
    return out
}
```

### Random Selection Demo

```go
ch1 := make(chan string, 1)
ch2 := make(chan string, 1)
ch1 <- "from 1"
ch2 <- "from 2"

// Both ready: random selection
select {
case m := <-ch1:
    fmt.Println(m)
case m := <-ch2:
    fmt.Println(m)
}
// Run multiple times: different results!
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| First case has priority | Random selection when multiple ready |
| Empty select blocks forever | Correct! `select {}` never returns |
| Default always runs | Only runs if all cases would block |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ • Wait on multiple channels simultaneously │
│ • Random choice if multiple ready          │
│ • default: non-blocking operation          │
│ • time.After: timeout pattern              │
│ • ctx.Done(): cancellation pattern         │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What happens with `select {}`?
2. When does the default case execute?
3. How is a case chosen when multiple are ready?
4. How do you implement a timeout with select?
5. Can select have send cases?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Blocks forever. Used to keep main() alive.

**A2**: When all other cases would block.

**A3**: Random selection (uniform distribution).

**A4**: `case <-time.After(duration):` as one of the cases.

**A5**: Yes! `case ch <- value:` is valid.

</details>

---

[← Previous](./03-blocking-behavior.md) | [Next →](./05-channel-internals.md)
