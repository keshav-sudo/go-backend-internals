# Netpoller

## 1️⃣ Simple Explanation (Beginner Friendly)

The **netpoller** is Go's way of handling network I/O without blocking OS threads. It uses OS mechanisms (epoll/kqueue) to efficiently wait on many connections.

```
Without netpoller: 1 thread per connection (expensive)
With netpoller: 1 thread monitors ALL connections
```

---

## 2️⃣ Real-World Analogy

### 📟 Pager System

Instead of each doctor waiting by their patient's bed, all doctors carry pagers. When a patient needs help, their pager beeps.

Netpoller = pager system for network events.

---

## 3️⃣ Technical Working (Step-by-Step)

### Architecture

```
┌────────────────────────────────────────────────┐
│                  USER CODE                     │
│        conn.Read() / conn.Write()              │
└───────────────────┬────────────────────────────┘
                    │ (would block)
                    ▼
┌────────────────────────────────────────────────┐
│              RUNTIME NETPOLLER                 │
│  ┌─────────────────────────────────────────┐   │
│  │ Register FD with epoll/kqueue           │   │
│  │ Park goroutine (Gwaiting)               │   │
│  │ When data ready → unpark goroutine      │   │
│  └─────────────────────────────────────────┘   │
└────────────────────────────────────────────────┘
                    │
                    ▼
┌────────────────────────────────────────────────┐
│    OS: epoll (Linux) / kqueue (macOS/BSD)      │
│    One thread monitors thousands of FDs        │
└────────────────────────────────────────────────┘
```

### How It Works

```
1. Goroutine calls conn.Read()
2. Runtime checks: data available?
3. No → Register FD with netpoller
        Park goroutine (Gwaiting)
4. Netpoller thread runs epoll_wait()
5. Data arrives → epoll signals runtime
6. Runtime marks goroutine Grunnable
7. Goroutine scheduled, continues execution
```

---

## 4️⃣ Where Used in Real Systems?

### Why C10K is Easy in Go

```go
// 10,000 connections, minimal threads!
for {
    conn, _ := listener.Accept()
    go handleConn(conn)  // Goroutine parks on I/O
}
// Most goroutines waiting in netpoller
// Only few OS threads actually running
```

---

## 5️⃣ Practical Examples

### Seeing Netpoller in Action

```bash
# Monitor network I/O blocking
GODEBUG=netpoll=1 ./server

# In pprof, look for:
# runtime.netpoll
# internal/poll.runtime_pollWait
```

### Non-Blocking Socket Under the Hood

```go
// This doesn't block the OS thread!
data, err := conn.Read(buf)
// Goroutine parks, thread does other work
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Each conn blocks a thread | Netpoller: 1 thread for all I/O |
| Go uses blocking I/O | Network I/O is async via netpoller |
| epoll in user code | Automatic, hidden by runtime |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Netpoller: async I/O with sync API         │
│                                            │
│ Linux: epoll                               │
│ macOS/BSD: kqueue                          │
│ Windows: IOCP                              │
│                                            │
│ • Goroutine parks on I/O                   │
│ • Thread freed for other work              │
│ • Wakes when data ready                    │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What OS mechanism does netpoller use on Linux?
2. What happens to a goroutine during network wait?
3. Does network I/O block OS threads?
4. How is netpoller different from Node.js?
5. What about file I/O?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: epoll (Linux), kqueue (macOS/BSD), IOCP (Windows).

**A2**: Parked (Gwaiting), added to netpoller's watch list.

**A3**: No! That's the magic. Goroutine parks, thread does other work.

**A4**: Similar async I/O, but Go has multi-threaded runtime + goroutines. Node is single-threaded.

**A5**: File I/O is blocking in Go. It causes M/P handoff, not netpoller.

</details>

---

[← Previous](./02-goroutine-per-connection.md) | [Next →](./04-connection-pooling.md)
