# Stack vs Heap

## 1️⃣ Simple Explanation (Beginner Friendly)

Go allocates memory in two places:

| Location | Characteristics |
|----------|-----------------|
| **Stack** | Fast, automatic, per-goroutine |
| **Heap** | Slower, managed by GC, shared |

**Key Insight**: Go's compiler decides where to allocate (escape analysis).

---

## 2️⃣ Real-World Analogy

### 📚 Desk vs Library

**Stack** = Your desk. Quick access, limited space, you clean up.

**Heap** = Library. More space, slower access, librarian (GC) cleans up.

---

## 3️⃣ Technical Working (Step-by-Step)

### Memory Layout

```
┌─────────────────────────────────────┐
│              HEAP                   │
│  (garbage collected, shared)        │
│  ┌─────┐ ┌─────┐ ┌─────┐           │
│  │ obj │ │ obj │ │ obj │           │
│  └─────┘ └─────┘ └─────┘           │
└─────────────────────────────────────┘
           ↑ (grows up)
           
           ↓ (grows down)
┌─────────────────────────────────────┐
│         GOROUTINE STACKS            │
│ ┌─────────┐ ┌─────────┐ ┌─────────┐│
│ │ G1 Stack│ │ G2 Stack│ │ G3 Stack││
│ └─────────┘ └─────────┘ └─────────┘│
└─────────────────────────────────────┘
```

### What Goes Where?

```
STACK ALLOCATION (fast, no GC):
• Local variables that don't escape
• Function parameters
• Return values (small)

HEAP ALLOCATION (needs GC):
• Variables that escape function scope
• Large allocations
• Variables accessed via pointers
• Closures capturing variables
```

### Escape Analysis Decision

```go
func stackAlloc() int {
    x := 42        // Stack: local, doesn't escape
    return x
}

func heapAlloc() *int {
    x := 42        // Heap: returned pointer escapes
    return &x      // Compiler: "x escapes to heap"
}
```

---

## 4️⃣ Where Used in Real Systems?

### Performance Optimization

```go
// GOOD: Stack allocation (fast)
func process(data [1000]int) {
    result := transform(data)  // Stack if possible
    use(result)
}

// CAREFUL: Heap allocation
func process() *Result {
    result := &Result{}  // Escapes to heap
    return result
}
```

---

## 5️⃣ Practical Examples

### Checking Escape Analysis

```bash
go build -gcflags="-m" main.go 2>&1 | grep escape

# Output:
# ./main.go:10:6: x escapes to heap
# ./main.go:15:6: y does not escape
```

### Escape Examples

```go
package main

// Does NOT escape (stays on stack)
func noEscape() int {
    x := 10
    return x
}

// ESCAPES to heap
func escapes() *int {
    x := 10
    return &x  // &x escapes
}

// ESCAPES - captured by closure
func closure() func() int {
    x := 10
    return func() int { return x }  // x escapes
}

// ESCAPES - interface{}
func toInterface() interface{} {
    x := 10
    return x  // x escapes (interface boxing)
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| All heap = slower | Heap is fine, GC is efficient |
| Avoid all allocations | Optimize only hot paths |
| Pointers always escape | Small objects may be inlined |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Stack: fast, auto-freed, per-goroutine     │
│ Heap: GC-managed, shared, escaping vars    │
│                                            │
│ Escape triggers:                           │
│ • Returning pointer                        │
│ • Closure capture                          │
│ • Interface conversion                     │
│ • Too large for stack                      │
│                                            │
│ Check with: go build -gcflags="-m"         │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. How does Go decide stack vs heap?
2. What makes a variable escape?
3. Why is stack allocation faster?
4. How do you check if a variable escapes?
5. Does using pointers always mean heap?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Escape analysis at compile time. If variable could outlive function, it goes to heap.

**A2**: Returned pointer, closure capture, interface boxing, too large.

**A3**: No GC involvement, just stack pointer adjustment. LIFO structure.

**A4**: `go build -gcflags="-m"` shows escape decisions.

**A5**: No! Small objects may stay on stack even with pointers (compiler optimizes).

</details>

---

[← Previous Module](../05-sync-primitives/05-when-not-to-use-channels.md) | [Next →](./02-escape-analysis.md)
