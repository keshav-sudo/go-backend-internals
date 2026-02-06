# 🚀 Go Backend Internals - Complete Self-Study Curriculum

> **Master Go (Golang) Backend Engineering from Beginner to Advanced Level**

A comprehensive curriculum covering Go runtime internals, concurrency, memory management, networking, and high-performance backend systems. Designed for system design interviews and production-grade optimization.

---

## 🎯 Learning Objectives

After completing this curriculum, you will be able to:

- ✅ Understand Go runtime internals and the M:P:G scheduler model
- ✅ Write highly concurrent applications using goroutines and channels
- ✅ Avoid race conditions, deadlocks, and goroutine leaks
- ✅ Optimize memory usage and understand garbage collection
- ✅ Build high-performance HTTP servers handling 10k+ connections
- ✅ Apply advanced concurrency patterns in production systems
- ✅ Profile and debug Go applications using pprof and trace tools
- ✅ Design scalable microservices with proper observability

---

## 📚 Curriculum Index

### [Module 01: Go Runtime](./01-go-runtime/)

Understanding the foundation of Go's execution model.

| # | Topic | Description |
|---|-------|-------------|
| 01 | [What is Go Designed For?](./01-go-runtime/01-what-is-go-designed-for.md) | Go's design philosophy and target use cases |
| 02 | [Go Compiler vs Runtime](./01-go-runtime/02-go-compiler-vs-runtime.md) | Compilation process and runtime responsibilities |
| 03 | [How Go Executes Code](./01-go-runtime/03-how-go-executes-code.md) | From source code to execution |
| 04 | [GOMAXPROCS](./01-go-runtime/04-gomaxprocs.md) | Controlling parallelism in Go |
| 05 | [Comparison with Node.js & Java](./01-go-runtime/05-comparison-with-nodejs-java.md) | Threading models comparison |

---

### [Module 02: Goroutines](./02-goroutines/)

Deep dive into Go's lightweight concurrency primitives.

| # | Topic | Description |
|---|-------|-------------|
| 01 | [What is a Goroutine?](./02-goroutines/01-what-is-a-goroutine.md) | Introduction to goroutines |
| 02 | [How Goroutines are Lightweight](./02-goroutines/02-goroutines-are-lightweight.md) | Memory efficiency and performance |
| 03 | [Stack Growth & Shrinking](./02-goroutines/03-stack-growth-shrinking.md) | Dynamic stack management |
| 04 | [Goroutine Lifecycle](./02-goroutines/04-goroutine-lifecycle.md) | States and transitions |
| 05 | [Goroutine Leaks](./02-goroutines/05-goroutine-leaks.md) | Detection and prevention |

---

### [Module 03: Go Scheduler](./03-go-scheduler/)

Master the M:P:G scheduling model.

| # | Topic | Description |
|---|-------|-------------|
| 01 | [M:P:G Model](./03-go-scheduler/01-mpg-model.md) | Understanding M, P, and G components |
| 02 | [Work Stealing](./03-go-scheduler/02-work-stealing.md) | Load balancing across processors |
| 03 | [Preemption](./03-go-scheduler/03-preemption.md) | How Go preempts running goroutines |
| 04 | [Scheduler Blocking Flow](./03-go-scheduler/04-scheduler-blocking-flow.md) | What happens when goroutines block |
| 05 | [Syscalls & Scheduler Interaction](./03-go-scheduler/05-syscalls-scheduler-interaction.md) | System call handling |

---

### [Module 04: Channels](./04-channels/)

Communication between goroutines.

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Why Channels Exist](./04-channels/01-why-channels-exist.md) | CSP philosophy and motivation |
| 02 | [Unbuffered vs Buffered Channels](./04-channels/02-unbuffered-vs-buffered.md) | Channel types and use cases |
| 03 | [Blocking Behavior](./04-channels/03-blocking-behavior.md) | Understanding channel operations |
| 04 | [Select Statement](./04-channels/04-select-statement.md) | Multiplexing channel operations |
| 05 | [Channel Internals](./04-channels/05-channel-internals.md) | How channels work under the hood |

---

### [Module 05: Sync Primitives](./05-sync-primitives/)

Low-level synchronization tools.

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Mutex & RWMutex](./05-sync-primitives/01-mutex-rwmutex.md) | Mutual exclusion locks |
| 02 | [WaitGroup](./05-sync-primitives/02-waitgroup.md) | Waiting for goroutine completion |
| 03 | [Cond Variables](./05-sync-primitives/03-cond-variables.md) | Condition-based synchronization |
| 04 | [Atomic Operations](./05-sync-primitives/04-atomic-operations.md) | Lock-free programming |
| 05 | [When NOT to Use Channels](./05-sync-primitives/05-when-not-to-use-channels.md) | Choosing the right tool |

---

### [Module 06: Memory Management](./06-memory-management/)

Understanding Go's memory model.

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Stack vs Heap](./06-memory-management/01-stack-vs-heap.md) | Memory allocation strategies |
| 02 | [Escape Analysis](./06-memory-management/02-escape-analysis.md) | How Go decides allocation location |
| 03 | [Garbage Collector](./06-memory-management/03-garbage-collector.md) | GC algorithms and behavior |
| 04 | [Stop-The-World Pauses](./06-memory-management/04-stop-the-world-pauses.md) | GC latency and optimization |
| 05 | [Memory Leaks](./06-memory-management/05-memory-leaks.md) | Common causes and detection |

---

### [Module 07: Networking](./07-networking/)

Building high-performance network applications.

| # | Topic | Description |
|---|-------|-------------|
| 01 | [net/http Server Internals](./07-networking/01-net-http-internals.md) | HTTP server architecture |
| 02 | [Handling 10k+ Connections](./07-networking/02-handling-10k-connections.md) | High concurrency patterns |
| 03 | [Netpoller](./07-networking/03-netpoller.md) | Go's network poller internals |
| 04 | [Goroutines Per Request Model](./07-networking/04-goroutines-per-request.md) | Request handling architecture |
| 05 | [gRPC Basics](./07-networking/05-grpc-basics.md) | Introduction to gRPC in Go |

---

### [Module 08: Concurrency Patterns](./08-concurrency-patterns/)

Battle-tested patterns for concurrent Go.

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Worker Pools](./08-concurrency-patterns/01-worker-pools.md) | Bounded concurrency pattern |
| 02 | [Fan-In / Fan-Out](./08-concurrency-patterns/02-fan-in-fan-out.md) | Parallel processing patterns |
| 03 | [Pipelines](./08-concurrency-patterns/03-pipelines.md) | Stream processing architecture |
| 04 | [Context Cancellation](./08-concurrency-patterns/04-context-cancellation.md) | Request-scoped cancellation |
| 05 | [Backpressure Handling](./08-concurrency-patterns/05-backpressure-handling.md) | Load management strategies |

---

### [Module 09: Performance Tuning](./09-performance-tuning/)

Profiling and optimizing Go applications.

| # | Topic | Description |
|---|-------|-------------|
| 01 | [pprof CPU Profiling](./09-performance-tuning/01-pprof-cpu-profiling.md) | Finding CPU bottlenecks |
| 02 | [Memory Profiling](./09-performance-tuning/02-memory-profiling.md) | Heap and allocation analysis |
| 03 | [Race Detector](./09-performance-tuning/03-race-detector.md) | Finding data races |
| 04 | [Benchmarking in Go](./09-performance-tuning/04-benchmarking.md) | Writing and running benchmarks |
| 05 | [Tracing Tools](./09-performance-tuning/05-tracing-tools.md) | Execution tracing and analysis |

---

### [Module 10: Advanced Systems](./10-advanced-systems/)

Production-grade backend engineering.

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Building Microservices](./10-advanced-systems/01-building-microservices.md) | Microservice architecture in Go |
| 02 | [Caching Strategies](./10-advanced-systems/02-caching-strategies.md) | In-memory and distributed caching |
| 03 | [Rate Limiting](./10-advanced-systems/03-rate-limiting.md) | Protecting services from overload |
| 04 | [Load Shedding](./10-advanced-systems/04-load-shedding.md) | Graceful degradation under load |
| 05 | [Graceful Shutdown](./10-advanced-systems/05-graceful-shutdown.md) | Safe service termination |
| 06 | [Observability](./10-advanced-systems/06-observability.md) | Logs, metrics, and tracing |

---

## 📖 Topic Format

Each topic follows a consistent **9-step structure**:

```
1️⃣ Simple Explanation (Beginner Friendly)
2️⃣ Real-World Analogy
3️⃣ Technical Working (Step-by-Step with ASCII Diagrams)
4️⃣ Where Used in Real Systems?
5️⃣ Practical Examples (Code + Debugging)
6️⃣ Common Mistakes & Interview Traps
7️⃣ Quick Summary Box
8️⃣ Quiz Questions (5)
9️⃣ Answer Key (with detailed explanations)
```

---

## 🗺️ Recommended Learning Path

### Path 1: Complete Beginner
```
Module 01 → Module 02 → Module 04 → Module 05 → Module 03 → 
Module 06 → Module 07 → Module 08 → Module 09 → Module 10
```

### Path 2: Experienced Developer (Focus on Internals)
```
Module 03 → Module 06 → Module 07 (Netpoller) → Module 09 → Module 10
```

### Path 3: Interview Preparation
```
Module 02 → Module 03 → Module 04 → Module 08 → 
Module 09 (Race Detector) → Module 10 (Rate Limiting, Graceful Shutdown)
```

---

## 🛠️ Prerequisites

- Basic Go syntax knowledge (variables, functions, structs)
- Familiarity with command line
- Basic understanding of concurrent programming concepts

---

## 💻 Tools You'll Use

| Tool | Purpose |
|------|---------|
| `go build` | Compile Go programs |
| `go run -race` | Run with race detector |
| `go tool pprof` | CPU and memory profiling |
| `go tool trace` | Execution tracing |
| `top`, `htop` | System monitoring |
| `strace`, `lsof` | System call tracing |

---

## 📊 Progress Tracker

- [ ] Module 01: Go Runtime (0/5)
- [ ] Module 02: Goroutines (0/5)
- [ ] Module 03: Go Scheduler (0/5)
- [ ] Module 04: Channels (0/5)
- [ ] Module 05: Sync Primitives (0/5)
- [ ] Module 06: Memory Management (0/5)
- [ ] Module 07: Networking (0/5)
- [ ] Module 08: Concurrency Patterns (0/5)
- [ ] Module 09: Performance Tuning (0/5)
- [ ] Module 10: Advanced Systems (0/6)

**Total Progress: 0/51 topics**

---

## 🎓 About This Curriculum

This curriculum is designed for:
- **Backend Engineers** looking to master Go internals
- **System Design Interview** preparation
- **Performance Engineers** optimizing Go applications
- **Self-learners** building production-grade systems

---

> **Start your journey: [Module 01 - What is Go Designed For?](./01-go-runtime/01-what-is-go-designed-for.md)**

---

*Created with ❤️ for Go developers who want to go beyond the surface.*
# go-backend-internals
