# Escape Analysis

## 1️⃣ Simple Explanation (Beginner Friendly)

**Escape analysis** is the compiler's process of deciding whether a variable can stay on the stack or must move to the heap.

If a variable "escapes" the function (could be accessed after function returns), it goes to heap. Otherwise, it stays on stack.

---

## 2️⃣ Real-World Analogy

### 📦 Package Decision

Escape analysis = deciding if a package can stay local (stack) or needs to be shipped out (heap).

If someone outside needs it → ship it (heap).

---

## 3️⃣ Technical Working (Step-by-Step)

### Escape Triggers

```
┌────────────────────────────────────────────┐
│ Variable ESCAPES when:                     │
├────────────────────────────────────────────┤
│ 1. Pointer is returned                     │
│ 2. Stored in a longer-lived location       │
│ 3. Captured by goroutine closure           │
│ 4. Assigned to interface{}                 │
│ 5. Sent through channel                    │
│ 6. Too large for stack                     │
└────────────────────────────────────────────┘
```

### Examples

```go
// NO ESCAPE
func noEscape() int {
    x := 42
    return x  // Value copied, x stays on stack
}

// ESCAPES
func escapes() *int {
    x := 42
    return &x  // Pointer returned, x must survive
}
```

---

## 4️⃣ Where Used in Real Systems?

### Hot Path Optimization

```go
// SLOW: Escapes every call
func processRequest() *Result {
    r := &Result{}
    r.compute()
    return r
}

// FAST: Pool reuse, reduces escapes
var pool = sync.Pool{New: func() any { return &Result{} }}

func processRequestFast() *Result {
    r := pool.Get().(*Result)
    r.compute()
    // Return to pool after use
    return r
}
```

---

## 5️⃣ Practical Examples

### Viewing Escape Decisions

```bash
# Basic escape info
go build -gcflags="-m" main.go

# Detailed analysis
go build -gcflags="-m -m" main.go

# Output example:
# ./main.go:5:6: x escapes to heap:
# ./main.go:5:6:   flow: ~r0 = &x:
# ./main.go:5:6:     from &x (address-of)
```

### Common Patterns

```go
// Interface boxing - ESCAPES
func interfaceEscape() {
    x := 42
    fmt.Println(x)  // x escapes (passed as interface{})
}

// Goroutine capture - ESCAPES
func goroutineEscape() {
    x := 42
    go func() {
        fmt.Println(x)  // x must outlive function
    }()
}

// Slice/map literals - may ESCAPE
func sliceEscape() {
    s := []int{1, 2, 3}  // Often escapes
    use(s)
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| All pointers escape | Small objects may be inlined |
| Avoid escapes always | Profile first, optimize hot paths |
| Escape = bad code | Sometimes heap is necessary |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Escape Analysis: compiler decides alloc   │
│                                            │
│ Escapes: pointer returned, closure,        │
│          interface, channel, too large     │
│                                            │
│ Check: go build -gcflags="-m"              │
│ Optimize: pools, value receivers,          │
│           avoid unnecessary pointers       │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What triggers escape analysis decision?
2. Does passing to interface{} cause escape?
3. How do sync.Pool help with escapes?
4. Is escape analysis done at runtime?
5. Does slice literal always escape?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Compiler analyzes if variable could be accessed after function returns.

**A2**: Usually yes! Interface boxing often causes escape.

**A3**: Reuse allocated objects instead of creating new ones.

**A4**: No! Compile time only. Runtime has no escape concept.

**A5**: Not always, but often. Small slices may stay on stack.

</details>

---

[← Previous](./01-stack-vs-heap.md) | [Next →](./03-garbage-collector.md)
