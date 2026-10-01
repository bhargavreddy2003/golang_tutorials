# Mastering DSA in Go — The Complete Guide

Sep 30, 2026 · @Bhargav Mora

## 1. How to use this guide

Read this guide top to bottom once, then use Section 20 (patterns) and Section 21 (cheat sheet) as your daily problem-solving reference. Every topic follows the same shape: the Go syntax, what each operation does, its cost, a reusable template, and a solved example.

All code targets Go 1.22 or newer: built-in `min`, `max` and `clear`, the `slices`, `maps` and `cmp` packages, and `for i := range n`. The few helpers that need Go 1.23 (iterators) are marked.

| Part | Sections | What you get |
| --- | --- | --- |
| Foundations | 1–3 | The Go syntax used in DSA, Big-O analysis |
| Data structures | 4–11 | Every structure, all its operations, their costs |
| Algorithms | 12–19 | Graphs, sorting, searching, recursion, DP, greedy, bits, math |
| Patterns | 20–21 | 30 patterns: how to spot each, Go templates, solved problems |
| Mastery | 22–23 | Go pitfalls and a 12-week practice plan |

### How to study each section

1. Read the syntax and type every code block yourself. Do not copy-paste.
2. Say the time and space cost of each operation before you check the table.
3. Close the guide and rewrite the template from memory.
4. Solve 3–5 problems that use it (Section 23 lists them).
5. Revisit after 3 days and again after 7 days.

### How to solve any problem (the 6-step loop)

1. Restate the input, output and constraints. The input size tells you the target complexity (Section 3).
2. Work two small examples by hand, including an edge case: empty, one element, duplicates, negatives.
3. Match the problem's signals to a pattern (Section 21).
4. State the brute force and its cost, then improve it with the pattern.
5. Write the Go code from the pattern's template.
6. Dry-run the code on your examples, then check the edge cases.

## 2. Go syntax essentials for DSA

These ten topics are the only Go features DSA needs; master them and every later template reads naturally. Snippets are fragments, so wrap them in a function to run them.

### 2.1 Program skeleton and fast input/output

```go
package main // every runnable program lives in package main

import (
    "bufio" // buffered I/O: far faster than raw fmt on big inputs
    "fmt"
    "os"
)

func main() {
    reader := bufio.NewReader(os.Stdin)  // buffers input in memory
    writer := bufio.NewWriter(os.Stdout) // buffers output
    defer writer.Flush()                 // runs when main returns; without it nothing prints

    var n int
    fmt.Fscan(reader, &n) // &n passes n's address so Fscan can fill it
    nums := make([]int, n)
    for i := range nums {
        fmt.Fscan(reader, &nums[i])
    }
    fmt.Fprintln(writer, n, nums)
}
```

On LeetCode-style platforms you only write the function, for example `func twoSum(nums []int, target int) []int`.

### 2.2 Variables, constants and zero values

```go
var a int                 // declared, zero value 0
var b, c = 1, "hi"        // types inferred
d := 3.5                  // short declaration (inside functions only), float64
const MOD = 1_000_000_007 // underscores are allowed in number literals
const (
    Red = iota // iota counts 0, 1, 2 ... inside a const block
    Green
    Blue
)
x, y := 1, 2
x, y = y, x // swap in one line, used constantly in DSA
_ = y       // the blank identifier discards a value
```

| Type | Zero value | Note |
| --- | --- | --- |
| int, int64, float64 | 0 | `int` is 64-bit on all modern platforms |
| bool | false |  |
| string | "" | immutable sequence of bytes |
| pointer, slice, map, func, interface | nil | writing to a nil map panics; appending to a nil slice is safe |
| struct | every field zeroed |  |

### 2.3 Types and conversions

| Type | Use in DSA | Range / note |
| --- | --- | --- |
| int | default integer | about ±9.2×10^18 (`math.MaxInt`, `math.MinInt`) |
| int32, int64 | when a problem fixes the width | `math.MaxInt32`, `math.MaxInt64` |
| uint, uint64 | bit tricks, hashing | no negatives, wraps on overflow |
| float64 | geometry, averages | `math.Inf(1)` is +infinity |
| byte (= uint8) | one ASCII character | `s[i]` returns a byte |
| rune (= int32) | one Unicode code point | `for _, r := range s` yields runes |
| bool | flags, visited arrays |  |

Go never converts types implicitly; you write `T(x)`.

```go
i := 7
f := float64(i) / 2  // 3.5; int / int would give 3
n := int(3.99)       // 3: truncates toward zero
b := byte('a' + 2)   // 'c'
idx := int(ch - 'a') // character to 0..25 index (ch is a byte)
s := string(rune(65)) // "A"; always convert an int to rune first
```

### 2.4 Control flow

```go
// if with an init statement: v exists only inside this if/else
if v, ok := m[key]; ok {
    fmt.Println(v)
} else if len(m) == 0 {
    fmt.Println("empty")
}

// for is Go's only loop, in several forms (bodies omitted)
for i := 0; i < n; i++ {}   // classic
for i := range n {}          // Go 1.22+: i = 0..n-1
for left < right {}          // while loop
for {}                       // infinite loop: exit with break or return
for i, v := range nums {}    // index + a COPY of the value
for k, v := range m {}       // map, random order every run
for i, r := range "héllo" {} // byte index + rune

// switch needs no break: cases never fall through
switch {
case x < 0:
    fmt.Println("neg")
case x == 0, x == 1: // several conditions in one case
    fmt.Println("small")
default:
    fmt.Println("big")
}

// a labeled break exits nested loops
outer:
for i := range grid {
    for j := range grid[i] {
        if grid[i][j] == target {
            break outer
        }
    }
}
```

### 2.5 Functions, closures and defer

```go
func add(a, b int) int { return a + b } // params of one type share it

func divmod(a, b int) (int, int) { // multiple return values
    return a / b, a % b
}
q, r := divmod(17, 5) // 3, 2

func minMax(nums []int) (lo, hi int) { // named results start at their zero value
    lo, hi = nums[0], nums[0]
    for _, v := range nums {
        lo, hi = min(lo, v), max(hi, v)
    }
    return // naked return sends lo, hi
}

func sum(nums ...int) int { // variadic: nums is a []int
    total := 0
    for _, v := range nums {
        total += v
    }
    return total
}
sum(1, 2, 3)
sum(mySlice...) // spread a slice into a variadic call

// A closure captures variables from the surrounding scope.
// This is the standard way to write recursive DFS helpers in Go.
func countNodes(root *TreeNode) int {
    count := 0
    var dfs func(*TreeNode) // declare first so dfs can call itself
    dfs = func(n *TreeNode) {
        if n == nil {
            return
        }
        count++ // modifies the outer variable
        dfs(n.Left)
        dfs(n.Right)
    }
    dfs(root)
    return count
}

defer fmt.Println("runs last") // deferred calls run last-in-first-out on return
```

### 2.6 Pointers

```go
x := 10
p := &x       // p is *int, holding x's address
*p = 20       // dereference: x is now 20
q := new(int) // allocates a zeroed int and returns *int

func inc(n *int) { *n++ } // a pointer lets a function change the caller's variable
inc(&x)
```

Linked-list and tree nodes are always handled as pointers (`*ListNode`, `*TreeNode`), so they can be `nil` and shared. Slices and maps already hold a pointer to their data, so a function can change their elements. But `append` inside a function needs the new slice returned, or a pointer to the slice.

`p.X` works directly on a pointer to a struct; Go inserts `(*p).X` for you.

### 2.7 Structs and methods

```go
type Point struct {
    X, Y int
}

p := Point{X: 1, Y: 2} // named fields (preferred)
p2 := Point{3, 4}      // positional
pp := &Point{5, 6}     // pointer to a new struct
pp.X = 9               // auto-dereference

// Value receiver: works on a copy. Use for read-only methods.
func (p Point) Dist2() int { return p.X*p.X + p.Y*p.Y }

// Pointer receiver: can modify the struct. Use it for every mutating data structure.
func (p *Point) Move(dx, dy int) {
    p.X += dx
    p.Y += dy
}

// Structs of comparable fields can be map keys and compared with ==
seen := map[Point]bool{}
seen[Point{0, 0}] = true

// A local type makes tidy BFS queue items
type item struct{ node, dist int }
queue := []item{{0, 0}}
```

### 2.8 Interfaces

An interface is a set of method signatures. A type satisfies it just by having those methods; there is no `implements` keyword. You will implement `sort.Interface` and `heap.Interface` most often.

```go
type Shape interface {
    Area() float64
}
type Square struct{ S float64 }

func (s Square) Area() float64 { return s.S * s.S } // Square now satisfies Shape

var sh Shape = Square{2}
fmt.Println(sh.Area())

// any (= interface{}) holds a value of any type; a type assertion gets it back
var v any = 42
n := v.(int)      // panics if v is not an int
n2, ok := v.(int) // safe form: ok is false instead of a panic

switch t := v.(type) { // type switch
case int:
    fmt.Println("int", t)
case string:
    fmt.Println("string", t)
}
```

### 2.9 Generics

```go
import "cmp"

// T is a type parameter; cmp.Ordered allows every type that supports < and >
func MaxOf[T cmp.Ordered](a []T) T {
    best := a[0]
    for _, v := range a[1:] {
        if v > best {
            best = v
        }
    }
    return best
}
MaxOf([]int{3, 9, 2})     // 9: T is inferred as int
MaxOf([]string{"b", "a"}) // "b"

// A generic data structure: a stack of any element type
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(v T) { s.items = append(s.items, v) }
func (s *Stack[T]) Pop() T {
    v := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return v
}

st := &Stack[int]{}
st.Push(1)

// A custom constraint; ~int also admits named types whose underlying type is int
type Number interface {
    ~int | ~int64 | ~float64
}

func Sum[T Number](a []T) T {
    var s T
    for _, v := range a {
        s += v
    }
    return s
}
```

| Constraint | Allows | Typical use |
| --- | --- | --- |
| `any` | every type | containers: stack, queue, deque |
| `comparable` | types usable with `==` | map keys, sets |
| `cmp.Ordered` | ints, uints, floats, strings | sorting, min/max, BST keys |

### 2.10 Built-in functions

| Built-in | What it does | Example |
| --- | --- | --- |
| `len(x)` | length of a string, slice, array, map or channel | `len(nums)` |
| `cap(s)` | capacity of a slice | `cap(nums)` |
| `make(T, ...)` | create a slice, map or channel | `make([]int, n)`, `make(map[string]int)` |
| `new(T)` | allocate a zeroed T, return \*T | `new(TreeNode)` |
| `append(s, v...)` | add elements; may reallocate, so always reassign | `s = append(s, 4, 5)` |
| `copy(dst, src)` | copy min(len(dst), len(src)) elements, return the count | `copy(b, a)` |
| `delete(m, k)` | remove a map key; no-op if missing | `delete(freq, 'a')` |
| `clear(x)` | empty a map, or zero every element of a slice | `clear(m)` |
| `min(...)`, `max(...)` | smallest / largest of ordered values | `best = max(best, cur)` |
| `panic(v)`, `recover()` | abort / catch an abort | rarely needed in DSA |

## 3. Complexity analysis

Read the input limit first: it tells you which complexity will pass before you write a line. A typical judge runs roughly 10^8 simple Go operations per second.

| Max n | Target complexity | Typical algorithm |
| --- | --- | --- |
| ≤ 10 | O(n!) | permutations, brute force |
| ≤ 20 | O(2^n · n) | subsets, bitmask DP |
| ≤ 100 | O(n^4) | four nested loops |
| ≤ 500 | O(n^3) | Floyd-Warshall, interval DP |
| ≤ 5,000 | O(n^2) | 2D DP, all pairs |
| ≤ 10^6 | O(n log n) | sorting, heap, binary search, segment tree |
| ≤ 10^8 | O(n) | two pointers, sliding window, hashing, prefix sums |
| > 10^9 | O(log n) or O(1) | binary search on the answer, a math formula |

### Growth rates

| Notation | Name | Example |
| --- | --- | --- |
| O(1) | constant | map lookup, slice index |
| O(log n) | logarithmic | binary search, heap push |
| O(n) | linear | one pass over the input |
| O(n log n) | linearithmic | sorting |
| O(n^2) | quadratic | two nested loops |
| O(2^n) | exponential | all subsets |
| O(n!) | factorial | all permutations |

### Rules for computing Big-O

1. Sequential steps add, and you keep the largest term: O(n) + O(n log n) = O(n log n).
2. Nested loops multiply: a loop over m inside a loop over n is O(n·m).
3. Drop constants: O(3n + 5) = O(n).
4. A loop that halves or doubles its variable runs O(log n) times.
5. Recursion costs (number of calls) × (work per call). Naive Fibonacci makes about 2^n calls, so O(2^n); memoized, it makes n calls, so O(n).
6. Space counts the extra memory you allocate plus the recursion depth on the call stack.

```go
// O(n log n): the outer loop runs n times, the inner loop halves j each step
for i := 0; i < n; i++ {
    for j := n; j > 0; j /= 2 {
        work()
    }
}
```

### Amortized cost

`append` occasionally copies the whole slice into a bigger backing array, an O(n) step. Capacity roughly doubles each time (about 1.25x for large slices), so those copies are rare. Averaged over many appends, each one costs O(1): this is amortized O(1).

### Master theorem (divide and conquer)

For a recurrence T(n) = a·T(n/b) + O(n^d):

```latex
T(n) = \begin{cases} O(n^d) & \text{if } d > \log_b a \\ O(n^d \log n) & \text{if } d = \log_b a \\ O(n^{\log_b a}) & \text{if } d < \log_b a \end{cases}
```

Merge sort has a = 2, b = 2, d = 1, so d = log₂2 and the cost is O(n log n). Binary search has a = 1, b = 2, d = 0, giving O(log n).

### Costs of Go operations that surprise people

| Operation | Cost | Why |
| --- | --- | --- |
| `s + t` string concatenation | O(len(s) + len(t)) | strings are immutable, so a new one is built; use `strings.Builder` in loops |
| `s[i:j]` on a string or slice | O(1) | shares memory with the original |
| `[]byte(s)`, `[]rune(s)`, `string(b)` | O(n) | each conversion copies |
| `len(x)` | O(1) | the length is stored |
| `append` | amortized O(1) | occasional reallocation |
| insert or delete in the middle of a slice | O(n) | shifts the elements after it |
| map get / set / delete | O(1) average | hash table |
| `slices.Contains`, `slices.Index` | O(n) | linear scan |
| `slices.Sort`, `sort.Slice` | O(n log n) | pattern-defeating quicksort |
| `fmt.Sprintf`, `fmt.Println` | large constant | use `strconv` and `bufio` in hot loops |

## 4. Arrays and slices

Slices are Go's workhorse: nearly every DSA solution uses them, so know each operation and its cost by heart. Arrays are fixed-size values; slices are growable views onto arrays.

### 4.1 Arrays: fixed size, value type

```go
var a [5]int           // [0 0 0 0 0]; the size is part of the type
b := [3]int{1, 2, 3}
c := [...]int{4, 5, 6} // the compiler counts the length
var freq [26]int       // ideal for letter counts
freq['c'-'a']++
d := b                 // COPIES the whole array (value semantics)
same := b == c         // arrays compare with == and can be map keys
```

### 4.2 Slices: a view onto an array

A slice is a small header holding a pointer to a backing array, a length and a capacity. Copying a slice copies only the header, so both copies share the same elements.

```go
var s []int              // nil slice: len 0, cap 0, safe to append to and range over
s1 := []int{1, 2, 3}     // literal
s2 := make([]int, 5)     // len 5, cap 5, all zeros
s3 := make([]int, 0, 10) // len 0, cap 10: preallocate, then append
fmt.Println(len(s3), cap(s3)) // 0 10

sub := s1[1:3]  // [2 3]: indexes 1 and 2, shares memory with s1
sub[0] = 99     // s1 is now [1 99 3]
head := s1[:2]  // first two elements
tail := s1[1:]  // everything but the first
fixed := s1[0:2:2] // low:high:max sets cap = max-low = 2, so the next append must reallocate
```

### 4.3 Every slice operation

| Operation | Code | Time |
| --- | --- | --- |
| Read / write | `s[i]`, `s[i] = v` | O(1) |
| Length / capacity | `len(s)`, `cap(s)` | O(1) |
| Append values | `s = append(s, v)`, `s = append(s, a, b)` | amortized O(1) each |
| Append a slice | `s = append(s, t...)` | O(len(t)) |
| Last element | `s[len(s)-1]` | O(1) |
| Pop last | `s = s[:len(s)-1]` | O(1) |
| Pop first | `s = s[1:]` | O(1) |
| Prepend | `s = append([]int{v}, s...)` | O(n) |
| Insert at i | `s = slices.Insert(s, i, v)` | O(n) |
| Delete at i, keep order | `s = slices.Delete(s, i, i+1)` or `s = append(s[:i], s[i+1:]...)` | O(n) |
| Delete at i, order not needed | `s[i] = s[len(s)-1]; s = s[:len(s)-1]` | O(1) |
| Independent copy | `t := slices.Clone(s)` | O(n) |
| Reverse | `slices.Reverse(s)` | O(n) |
| Set all to zero | `clear(s)` | O(n) |
| Swap | `s[i], s[j] = s[j], s[i]` | O(1) |

The same operations written by hand, which interviewers often ask for:

```go
// Insert v at index i (what slices.Insert does)
s = append(s, 0)     // grow by one
copy(s[i+1:], s[i:]) // shift the tail right by one
s[i] = v

// Reverse in place with two pointers
for l, r := 0, len(s)-1; l < r; l, r = l+1, r-1 {
    s[l], s[r] = s[r], s[l]
}

// Filter in place (keep even numbers) with no extra allocation
k := 0
for _, v := range s {
    if v%2 == 0 {
        s[k] = v
        k++
    }
}
s = s[:k]
```

### 4.4 The slices package

| Function | What it does | Time |
| --- | --- | --- |
| `slices.Sort(s)` | sort ascending | O(n log n) |
| `slices.SortFunc(s, cmp)` | sort with a compare func returning negative / 0 / positive | O(n log n) |
| `slices.SortStableFunc(s, cmp)` | stable sort (equal elements keep their order) | O(n log n) |
| `slices.IsSorted(s)` | report whether s is sorted | O(n) |
| `slices.BinarySearch(s, v)` | (index, found) in sorted s; index is the insertion point | O(log n) |
| `slices.BinarySearchFunc(s, t, cmp)` | the same with a comparator | O(log n) |
| `slices.Contains(s, v)` | membership test | O(n) |
| `slices.Index(s, v)` | first index of v, or -1 | O(n) |
| `slices.IndexFunc(s, f)` | first index where f is true, or -1 | O(n) |
| `slices.Insert(s, i, vs...)` | insert values at i | O(n) |
| `slices.Delete(s, i, j)` | remove s\[i:j\] | O(n) |
| `slices.DeleteFunc(s, f)` | remove every element where f is true | O(n) |
| `slices.Replace(s, i, j, vs...)` | replace s\[i:j\] with vs | O(n) |
| `slices.Reverse(s)` | reverse in place | O(n) |
| `slices.Max(s)`, `slices.Min(s)` | largest / smallest; panics on an empty slice | O(n) |
| `slices.MaxFunc`, `slices.MinFunc` | the same with a comparator | O(n) |
| `slices.Equal(a, b)` | same length and same elements | O(n) |
| `slices.Compare(a, b)` | lexicographic -1 / 0 / 1 | O(n) |
| `slices.Clone(s)` | shallow copy | O(n) |
| `slices.Compact(s)` | drop consecutive duplicates (sort first to dedupe everything) | O(n) |
| `slices.Concat(a, b, ...)` | join slices into a new one | O(total) |
| `slices.Grow(s, n)` | make room for n more elements | O(n) |
| `slices.Repeat(s, k)` | s repeated k times (Go 1.23) | O(n·k) |
| `slices.Collect(seq)`, `slices.Sorted(seq)` | build a slice from an iterator (Go 1.23) | O(n), O(n log n) |

```go
import (
    "cmp"
    "slices"
)

nums := []int{5, 2, 8, 2}
slices.Sort(nums)                        // [2 2 5 8]
i, found := slices.BinarySearch(nums, 5) // 2 true
nums = slices.Compact(nums)              // [2 5 8]

// Descending order: swap the arguments to cmp.Compare
slices.SortFunc(nums, func(a, b int) int { return cmp.Compare(b, a) })

// Sort by age, then by name; cmp.Or returns its first non-zero argument
type Person struct {
    Name string
    Age  int
}
slices.SortFunc(people, func(a, b Person) int {
    return cmp.Or(cmp.Compare(a.Age, b.Age), cmp.Compare(a.Name, b.Name))
})
```

### 4.5 2D slices: grids, matrices and DP tables

```go
rows, cols := 3, 4
grid := make([][]int, rows) // outer slice of rows
for i := range grid {
    grid[i] = make([]int, cols) // each row is allocated separately
}
grid[1][2] = 7

m := [][]int{{1, 2}, {3, 4}} // literal

// DP table filled with -1, the usual "not computed yet" marker
dp := make([][]int, n)
for i := range dp {
    dp[i] = make([]int, k)
    for j := range dp[i] {
        dp[i][j] = -1
    }
}

// Deep copy: copying only the outer slice would still share the rows
cp := make([][]int, len(grid))
for i := range grid {
    cp[i] = slices.Clone(grid[i])
}

// Transpose
t := make([][]int, cols)
for j := range t {
    t[j] = make([]int, rows)
    for i := range rows {
        t[j][i] = grid[i][j]
    }
}

// Visit the 4 neighbours of cell (r, c)
dirs := [4][2]int{{0, 1}, {1, 0}, {0, -1}, {-1, 0}}
for _, d := range dirs {
    nr, nc := r+d[0], c+d[1]
    if nr >= 0 && nr < rows && nc >= 0 && nc < cols {
        // use grid[nr][nc]
    }
}
```

### 4.6 The aliasing trap

```go
a := []int{1, 2, 3, 4}
b := a[:2]        // [1 2] with cap 4: shares a's array
b = append(b, 99) // fits in the capacity, so it OVERWRITES a[2]
fmt.Println(a)    // [1 2 99 4]
// Fix: b := slices.Clone(a[:2]) or b := a[:2:2]
```

## 5. Strings, bytes and runes

A Go string is an immutable sequence of bytes: indexing gives a byte, ranging gives runes, and any edit means converting to `[]byte` or `[]rune`. If a problem says "lowercase English letters", work with bytes and a `[26]int`; if it involves Unicode, use `[]rune`.

### 5.1 The three views of text

```go
s := "go!"
fmt.Println(len(s))    // 3 bytes
b := s[0]              // 'g' as a byte (uint8) = 103
fmt.Println(string(b)) // "g"
// s[0] = 'G'          // compile error: strings are immutable

bs := []byte(s) // mutable copy of the bytes; best for ASCII problems
bs[0] = 'G'
s = string(bs) // "Go!"

u := "héllo"
fmt.Println(len(u)) // 6: é takes 2 bytes in UTF-8
rs := []rune(u)     // one element per character
fmt.Println(len(rs)) // 5
for i, r := range u { // i = byte offset, r = rune
    fmt.Println(i, string(r))
}
```

### 5.2 Character arithmetic

```go
c := byte('e')
idx := c - 'a'          // 4: letter to 0..25
up := c - 'a' + 'A'     // 'E'
digit := int('7' - '0') // 7: digit character to int
ch := byte('0' + 5)     // '5'
isLower := c >= 'a' && c <= 'z'

var freq [26]int
for i := 0; i < len(s); i++ {
    freq[s[i]-'a']++
}
```

### 5.3 The strings package

| Function | Returns | Example → result |
| --- | --- | --- |
| `strings.Contains(s, sub)` | bool | `Contains("seafood", "foo")` → true |
| `strings.Index(s, sub)` | first index, or -1 | `Index("chicken", "ken")` → 4 |
| `strings.LastIndex(s, sub)` | last index, or -1 | `LastIndex("go gopher", "go")` → 3 |
| `strings.IndexByte(s, c)` | first index of a byte, or -1 | `IndexByte("abc", 'c')` → 2 |
| `strings.HasPrefix(s, p)`, `strings.HasSuffix(s, p)` | bool | `HasPrefix("golang", "go")` → true |
| `strings.Count(s, sub)` | non-overlapping count | `Count("cheese", "e")` → 3 |
| `strings.Split(s, sep)` | \[\]string | `Split("a,b,c", ",")` → \[a b c\] |
| `strings.Fields(s)` | split on any whitespace, no empty parts | `Fields("  a  b ")` → \[a b\] |
| `strings.Join(parts, sep)` | string | `Join([]string{"a", "b"}, "-")` → "a-b" |
| `strings.Repeat(s, n)` | string | `Repeat("ab", 3)` → "ababab" |
| `strings.Replace(s, old, new, n)` | replace the first n (-1 = all) | `Replace("aaa", "a", "b", 2)` → "bba" |
| `strings.ReplaceAll(s, old, new)` | replace all | `ReplaceAll("a-b-c", "-", "")` → "abc" |
| `strings.ToLower(s)`, `strings.ToUpper(s)` | string | `ToUpper("go")` → "GO" |
| `strings.TrimSpace(s)` | strip leading and trailing whitespace | `TrimSpace(" hi ")` → "hi" |
| `strings.Trim(s, cutset)` | strip any chars in cutset from both ends | `Trim("xxhixx", "x")` → "hi" |
| `strings.TrimPrefix(s, p)`, `strings.TrimSuffix(s, p)` | strip one prefix / suffix | `TrimSuffix("a.go", ".go")` → "a" |
| `strings.Cut(s, sep)` | before, after, found | `Cut("k=v", "=")` → "k", "v", true |
| `strings.EqualFold(a, b)` | case-insensitive equality | `EqualFold("Go", "GO")` → true |
| `strings.Map(f, s)` | apply f to every rune | lowercase, strip characters |

### 5.4 Building strings efficiently

`s += "x"` in a loop copies the whole string every time, which is O(n²) overall. `strings.Builder` appends into a growing buffer in amortized O(1).

```go
var sb strings.Builder
sb.Grow(64) // optional: reserve capacity up front
for i := 0; i < 3; i++ {
    sb.WriteString("ab") // append a string
    sb.WriteByte('-')    // append a byte
}
sb.WriteRune('é')      // append a rune
result := sb.String()  // "ab-ab-ab-é"
fmt.Println(sb.Len())  // length in bytes
sb.Reset()             // empty it for reuse

// Alternative: collect bytes, convert once at the end
buf := make([]byte, 0, n)
buf = append(buf, 'x')
res := string(buf)
```

### 5.5 strconv: numbers and strings

| Function | Does | Example |
| --- | --- | --- |
| `strconv.Itoa(i)` | int to string | `Itoa(42)` → "42" |
| `strconv.Atoi(s)` | string to int, plus an error | `n, err := Atoi("42")` |
| `strconv.ParseInt(s, base, bits)` | parse in any base | `ParseInt("ff", 16, 64)` → 255 |
| `strconv.FormatInt(i, base)` | int64 to a string in any base | `FormatInt(5, 2)` → "101" |
| `strconv.ParseFloat(s, 64)` | string to float64 | `ParseFloat("3.5", 64)` → 3.5 |
| `strconv.ParseBool(s)` | string to bool | `ParseBool("true")` → true |

### 5.6 The unicode package

`unicode.IsLetter`, `unicode.IsDigit`, `unicode.IsSpace`, `unicode.IsUpper`, `unicode.IsLower` and `unicode.IsPunct` test a rune. `unicode.ToUpper` and `unicode.ToLower` convert one. For a byte, convert first: `unicode.IsDigit(rune(b))`.

### 5.7 String recipes you will reuse

```go
// Reverse a string (Unicode-safe)
func reverse(s string) string {
    r := []rune(s)
    for i, j := 0, len(r)-1; i < j; i, j = i+1, j-1 {
        r[i], r[j] = r[j], r[i]
    }
    return string(r)
}

// Valid palindrome, ignoring non-alphanumerics and case
func isPalindrome(s string) bool {
    l, r := 0, len(s)-1
    for l < r {
        for l < r && !isAlnum(s[l]) {
            l++
        }
        for l < r && !isAlnum(s[r]) {
            r--
        }
        if lower(s[l]) != lower(s[r]) {
            return false
        }
        l++
        r--
    }
    return true
}

func isAlnum(c byte) bool {
    return c >= 'a' && c <= 'z' || c >= 'A' && c <= 'Z' || c >= '0' && c <= '9'
}

func lower(c byte) byte {
    if c >= 'A' && c <= 'Z' {
        return c + ('a' - 'A')
    }
    return c
}

// Anagram key: a [26]int array is comparable, so it works as a map key
func anagramKey(s string) [26]int {
    var k [26]int
    for i := 0; i < len(s); i++ {
        k[s[i]-'a']++
    }
    return k
}

// Sort the characters of a string
b := []byte(s)
slices.Sort(b)
sorted := string(b)

// Strings compare lexicographically with < > ==
less := "apple" < "banana" // true
```

## 6. Maps and sets

A map is Go's hash table: average O(1) insert, lookup and delete, the backbone of counting, two-sum and seen-before problems. Go has no set type; a map with empty-struct or bool values is the idiom.

### 6.1 Creating and using maps

```go
m := make(map[string]int)            // empty, ready to write
m2 := map[string]int{"a": 1, "b": 2} // literal
var nilMap map[string]int            // nil: reads return 0, writes PANIC
big := make(map[int]int, 1000)       // size hint avoids rehashing

m["apple"] = 5     // insert or overwrite
v := m["apple"]    // 5
zero := m["kiwi"]  // 0: a missing key returns the zero value
v, ok := m["kiwi"] // comma-ok form: ok is false when the key is absent
delete(m, "apple") // remove; no error if missing
n := len(m)        // number of keys
clear(m)           // remove every key

m["x"]++ // counting works because a missing key reads as 0

for k, v := range m { // iteration order is RANDOM on every run
    fmt.Println(k, v)
}
```

### 6.2 Operations and costs

| Operation | Syntax | Time |
| --- | --- | --- |
| Insert / update | `m[k] = v` | O(1) average |
| Lookup | `v := m[k]` | O(1) average |
| Exists | `_, ok := m[k]` | O(1) average |
| Delete | `delete(m, k)` | O(1) average |
| Size | `len(m)` | O(1) |
| Iterate | `for k, v := range m` | O(n) |
| Remove all | `clear(m)` | O(n) |

### 6.3 Keys and values

Keys must be comparable: numbers, strings, bools, pointers, and arrays or structs built from those. Slices, maps and functions cannot be keys. To key by a slice, convert it to an array, a struct or a string.

```go
type pair struct{ r, c int }
visited := map[pair]bool{}
visited[pair{1, 2}] = true

cells := map[[2]int]int{} // array key
cells[[2]int{3, 4}] = 1

// Map of slices: append works on a missing key because it reads as a nil slice
groups := map[string][]string{}
groups["fruit"] = append(groups["fruit"], "apple")

// Map of maps: create the inner map before writing to it
graph := map[int]map[int]bool{}
if graph[1] == nil {
    graph[1] = map[int]bool{}
}
graph[1][2] = true

// A struct stored by value cannot be modified in place
type stat struct{ count int }
st := map[string]stat{}
// st["a"].count++ // compile error
s := st["a"] // copy it, change it, write it back
s.count++
st["a"] = s

ptrs := map[string]*stat{"a": {}} // or store pointers instead
ptrs["a"].count++
```

### 6.4 Sorted iteration

```go
import (
    "maps"
    "slices"
)

// Go 1.23+
for _, k := range slices.Sorted(maps.Keys(m)) {
    fmt.Println(k, m[k])
}

// Any Go version
keys := make([]string, 0, len(m))
for k := range m {
    keys = append(keys, k)
}
slices.Sort(keys)
```

### 6.5 The maps package

| Function | What it does |
| --- | --- |
| `maps.Keys(m)`, `maps.Values(m)` | iterators over keys / values (Go 1.23); wrap in `slices.Collect` or `slices.Sorted` |
| `maps.All(m)` | iterator over key-value pairs (Go 1.23) |
| `maps.Clone(m)` | shallow copy |
| `maps.Copy(dst, src)` | copy every pair from src into dst |
| `maps.Equal(a, b)` | same keys with equal values |
| `maps.EqualFunc(a, b, eq)` | the same with a value comparator |
| `maps.DeleteFunc(m, f)` | delete every pair where f(k, v) is true |

### 6.6 Sets

```go
set := map[int]struct{}{} // struct{} values take zero bytes
set[5] = struct{}{}       // add
_, has := set[5]          // contains
delete(set, 5)            // remove

seen := map[int]bool{} // easier to read: a missing key reads false
if seen[x] {
    // duplicate
}
seen[x] = true
```

A reusable generic set:

```go
type Set[T comparable] map[T]struct{}

func NewSet[T comparable](items ...T) Set[T] {
    s := Set[T]{}
    for _, v := range items {
        s.Add(v)
    }
    return s
}

func (s Set[T]) Add(v T)      { s[v] = struct{}{} }
func (s Set[T]) Remove(v T)   { delete(s, v) }
func (s Set[T]) Has(v T) bool { _, ok := s[v]; return ok }
func (s Set[T]) Len() int     { return len(s) }

func (s Set[T]) Union(o Set[T]) Set[T] {
    r := Set[T]{}
    for v := range s {
        r.Add(v)
    }
    for v := range o {
        r.Add(v)
    }
    return r
}

func (s Set[T]) Intersect(o Set[T]) Set[T] {
    r := Set[T]{}
    for v := range s {
        if o.Has(v) {
            r.Add(v)
        }
    }
    return r
}

func (s Set[T]) Difference(o Set[T]) Set[T] {
    r := Set[T]{}
    for v := range s {
        if !o.Has(v) {
            r.Add(v)
        }
    }
    return r
}
```

### 6.7 Classic map problems

```go
// Two Sum: store each value's index, look up the complement
func twoSum(nums []int, target int) []int {
    idx := map[int]int{} // value -> index
    for i, v := range nums {
        if j, ok := idx[target-v]; ok {
            return []int{j, i}
        }
        idx[v] = i
    }
    return nil
}

// Group Anagrams: the letter-count array is the key
func groupAnagrams(strs []string) [][]string {
    groups := map[[26]int][]string{}
    for _, s := range strs {
        var k [26]int
        for i := 0; i < len(s); i++ {
            k[s[i]-'a']++
        }
        groups[k] = append(groups[k], s)
    }
    res := make([][]string, 0, len(groups))
    for _, g := range groups {
        res = append(res, g)
    }
    return res
}
```

When keys are small integers (0–25, or 0..n), a plain array or slice beats a map: it is faster and uses less memory.

## 7. Linked lists

A linked list gives up O(1) random access in exchange for O(1) insert and delete at a node you already hold. Almost every list problem is solved with one of three tools: a dummy head node, fast and slow pointers, or in-place reversal.

### 7.1 Singly linked list: the node

```go
type ListNode struct {
    Val  int
    Next *ListNode // nil marks the end of the list
}

// Build a list from a slice (handy for testing)
func build(vals []int) *ListNode {
    dummy := &ListNode{}
    cur := dummy
    for _, v := range vals {
        cur.Next = &ListNode{Val: v}
        cur = cur.Next
    }
    return dummy.Next
}

// Traverse and collect the values
func toSlice(head *ListNode) []int {
    var out []int
    for cur := head; cur != nil; cur = cur.Next {
        out = append(out, cur.Val)
    }
    return out
}
```

### 7.2 Every operation and its cost

| Operation | Time | Note |
| --- | --- | --- |
| Access the k-th node | O(k) | walk from the head |
| Search for a value | O(n) |  |
| Insert at head | O(1) |  |
| Insert at tail | O(n), or O(1) with a tail pointer |  |
| Insert after a known node | O(1) |  |
| Delete the head | O(1) |  |
| Delete a value | O(n) | you need the previous node |
| Length | O(n) | unless you store it |
| Reverse | O(n) time, O(1) space |  |

```go
// Insert at head: returns the new head
func pushFront(head *ListNode, v int) *ListNode {
    return &ListNode{Val: v, Next: head}
}

// Insert after a known node
func insertAfter(node *ListNode, v int) {
    node.Next = &ListNode{Val: v, Next: node.Next}
}

// Delete every node equal to v; the dummy node removes the "head is deleted" special case
func removeElements(head *ListNode, v int) *ListNode {
    dummy := &ListNode{Next: head}
    prev := dummy
    for prev.Next != nil {
        if prev.Next.Val == v {
            prev.Next = prev.Next.Next // unlink the node
        } else {
            prev = prev.Next
        }
    }
    return dummy.Next
}
```

### 7.3 The core techniques

```go
// Reverse iteratively: O(n) time, O(1) space
func reverseList(head *ListNode) *ListNode {
    var prev *ListNode
    cur := head
    for cur != nil {
        next := cur.Next // 1. save the rest of the list
        cur.Next = prev  // 2. point this node backwards
        prev = cur       // 3. advance prev
        cur = next       // 4. advance cur
    }
    return prev // prev is the new head
}

// Reverse recursively: reverse the rest, then hang this node on its end
func reverseRec(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
    newHead := reverseRec(head.Next)
    head.Next.Next = head
    head.Next = nil
    return newHead
}

// Middle node: fast moves 2, slow moves 1 (even length returns the second middle)
func middle(head *ListNode) *ListNode {
    slow, fast := head, head
    for fast != nil && fast.Next != nil {
        slow, fast = slow.Next, fast.Next.Next
    }
    return slow
}

// Floyd's cycle detection, returning the node where the cycle starts
func detectCycle(head *ListNode) *ListNode {
    slow, fast := head, head
    for fast != nil && fast.Next != nil {
        slow, fast = slow.Next, fast.Next.Next
        if slow == fast { // they met inside the cycle
            p := head
            for p != slow { // walking from head and from the meeting point meets at the entry
                p, slow = p.Next, slow.Next
            }
            return p
        }
    }
    return nil
}

// Merge two sorted lists
func mergeTwoLists(a, b *ListNode) *ListNode {
    dummy := &ListNode{}
    tail := dummy
    for a != nil && b != nil {
        if a.Val <= b.Val {
            tail.Next, a = a, a.Next
        } else {
            tail.Next, b = b, b.Next
        }
        tail = tail.Next
    }
    if a != nil {
        tail.Next = a
    } else {
        tail.Next = b
    }
    return dummy.Next
}

// Remove the n-th node from the end: fast gets n+1 steps ahead, then both move
func removeNthFromEnd(head *ListNode, n int) *ListNode {
    dummy := &ListNode{Next: head}
    fast, slow := dummy, dummy
    for i := 0; i <= n; i++ {
        fast = fast.Next
    }
    for fast != nil {
        fast, slow = fast.Next, slow.Next
    }
    slow.Next = slow.Next.Next
    return dummy.Next
}
```

### 7.4 Doubly linked list with sentinels

Sentinel head and tail nodes never hold data, so no operation ever checks for nil.

```go
type DNode struct {
    Key, Val   int
    Prev, Next *DNode
}

type DList struct {
    head, tail *DNode // sentinels
    size       int
}

func NewDList() *DList {
    h, t := &DNode{}, &DNode{}
    h.Next, t.Prev = t, h
    return &DList{head: h, tail: t}
}

func (l *DList) insertAfter(p, n *DNode) {
    n.Prev, n.Next = p, p.Next
    p.Next.Prev = n
    p.Next = n
    l.size++
}

func (l *DList) PushFront(n *DNode) { l.insertAfter(l.head, n) }
func (l *DList) PushBack(n *DNode)  { l.insertAfter(l.tail.Prev, n) }

func (l *DList) Remove(n *DNode) {
    n.Prev.Next = n.Next
    n.Next.Prev = n.Prev
    l.size--
}

func (l *DList) Back() *DNode {
    if l.size == 0 {
        return nil
    }
    return l.tail.Prev
}
```

### 7.5 container/list: the standard library's doubly linked list

Each element stores `Value any`, so reading it needs a type assertion such as `e.Value.(int)`.

| Method | Does | Time |
| --- | --- | --- |
| `list.New()` | create an empty list | O(1) |
| `l.PushFront(v)`, `l.PushBack(v)` | insert; returns the new `*list.Element` | O(1) |
| `l.InsertBefore(v, mark)`, `l.InsertAfter(v, mark)` | insert next to an element | O(1) |
| `l.Remove(e)` | remove e; returns `e.Value` | O(1) |
| `l.MoveToFront(e)`, `l.MoveToBack(e)` | move an element to an end | O(1) |
| `l.MoveBefore(e, mark)`, `l.MoveAfter(e, mark)` | move next to another element | O(1) |
| `l.Front()`, `l.Back()` | first / last element, or nil | O(1) |
| `l.Len()` | number of elements | O(1) |
| `e.Next()`, `e.Prev()` | neighbouring element, or nil | O(1) |
| `l.PushBackList(o)`, `l.PushFrontList(o)` | insert a copy of another list | O(len(o)) |
| `l.Init()` | clear the list | O(1) |

```go
import "container/list"

l := list.New()
l.PushBack(1)
l.PushBack(2)
e := l.PushFront(0)
l.MoveToBack(e) // list is now 1 2 0
for e := l.Front(); e != nil; e = e.Next() {
    fmt.Println(e.Value.(int))
}
```

### 7.6 LRU cache: container/list plus a map

The map finds a key's list element in O(1); the list keeps recency order, so every operation is O(1).

```go
type entry struct{ key, val int }

type LRUCache struct {
    cap   int
    ll    *list.List // front = most recently used
    items map[int]*list.Element
}

func Constructor(capacity int) LRUCache {
    return LRUCache{cap: capacity, ll: list.New(), items: map[int]*list.Element{}}
}

func (c *LRUCache) Get(key int) int {
    if e, ok := c.items[key]; ok {
        c.ll.MoveToFront(e)
        return e.Value.(*entry).val
    }
    return -1
}

func (c *LRUCache) Put(key, val int) {
    if e, ok := c.items[key]; ok {
        e.Value.(*entry).val = val
        c.ll.MoveToFront(e)
        return
    }
    c.items[key] = c.ll.PushFront(&entry{key, val})
    if c.ll.Len() > c.cap {
        last := c.ll.Back()
        c.ll.Remove(last)
        delete(c.items, last.Value.(*entry).key)
    }
}
```

## 8. Stacks, queues, deques and rings

Go has no built-in stack or queue: a slice is your stack, a slice or ring buffer is your queue, and `container/list` is a ready-made deque.

| Structure | Push | Pop | Peek | Best Go implementation |
| --- | --- | --- | --- | --- |
| Stack (LIFO) | O(1) amortized | O(1) | O(1) | slice |
| Queue (FIFO) | O(1) amortized | O(1) | O(1) | slice with `q[1:]`, or a ring buffer |
| Deque | O(1) | O(1) | O(1) | `container/list` or a ring buffer |
| Priority queue | O(log n) | O(log n) | O(1) | `container/heap` (Section 9) |

### 8.1 Stack with a slice

```go
stack := []int{}
stack = append(stack, 1, 2, 3) // push
top := stack[len(stack)-1]     // peek: 3
stack = stack[:len(stack)-1]   // pop
empty := len(stack) == 0
```

A generic stack you can reuse:

```go
type Stack[T any] struct{ data []T }

func (s *Stack[T]) Push(v T)    { s.data = append(s.data, v) }
func (s *Stack[T]) Len() int    { return len(s.data) }
func (s *Stack[T]) Empty() bool { return len(s.data) == 0 }
func (s *Stack[T]) Peek() T     { return s.data[len(s.data)-1] }

func (s *Stack[T]) Pop() T {
    v := s.data[len(s.data)-1]
    s.data = s.data[:len(s.data)-1]
    return v
}
```

Example, valid parentheses:

```go
func isValid(s string) bool {
    pairs := map[byte]byte{')': '(', ']': '[', '}': '{'}
    var st []byte
    for i := 0; i < len(s); i++ {
        c := s[i]
        if open, isClose := pairs[c]; isClose {
            if len(st) == 0 || st[len(st)-1] != open {
                return false
            }
            st = st[:len(st)-1]
        } else {
            st = append(st, c)
        }
    }
    return len(st) == 0
}
```

### 8.2 Queue

The slice queue is what you will write most often, especially for BFS.

```go
queue := []int{start}
for len(queue) > 0 {
    cur := queue[0]   // front
    queue = queue[1:] // dequeue: O(1); old slots are reclaimed when the slice reallocates
    for _, next := range adj[cur] {
        queue = append(queue, next) // enqueue
    }
}

// Level-by-level processing: swap in a fresh slice per level
level := 0
for len(queue) > 0 {
    var next []int
    for _, cur := range queue {
        // process cur, then: next = append(next, children of cur...)
    }
    queue = next
    level++
}
```

A ring-buffer queue never leaks the popped slots, which matters for long-running queues:

```go
type Queue[T any] struct {
    buf        []T
    head, size int
}

func NewQueue[T any](capacity int) *Queue[T] {
    return &Queue[T]{buf: make([]T, max(capacity, 1))}
}

func (q *Queue[T]) Len() int { return q.size }
func (q *Queue[T]) Front() T { return q.buf[q.head] }

func (q *Queue[T]) Push(v T) {
    if q.size == len(q.buf) {
        q.grow()
    }
    q.buf[(q.head+q.size)%len(q.buf)] = v
    q.size++
}

func (q *Queue[T]) Pop() T {
    v := q.buf[q.head]
    var zero T
    q.buf[q.head] = zero // drop the reference so the GC can free it
    q.head = (q.head + 1) % len(q.buf)
    q.size--
    return v
}

func (q *Queue[T]) grow() {
    nb := make([]T, 2*len(q.buf))
    for i := 0; i < q.size; i++ {
        nb[i] = q.buf[(q.head+i)%len(q.buf)]
    }
    q.buf, q.head = nb, 0
}
```

A queue built from two stacks is a classic interview question; each element moves at most once, so every operation is amortized O(1).

```go
type MyQueue struct{ in, out []int }

func (q *MyQueue) Push(x int) { q.in = append(q.in, x) }

func (q *MyQueue) move() { // refill out only when it is empty
    if len(q.out) == 0 {
        for len(q.in) > 0 {
            q.out = append(q.out, q.in[len(q.in)-1])
            q.in = q.in[:len(q.in)-1]
        }
    }
}

func (q *MyQueue) Pop() int {
    q.move()
    v := q.out[len(q.out)-1]
    q.out = q.out[:len(q.out)-1]
    return v
}

func (q *MyQueue) Peek() int   { q.move(); return q.out[len(q.out)-1] }
func (q *MyQueue) Empty() bool { return len(q.in) == 0 && len(q.out) == 0 }
```

### 8.3 Deque

```go
// container/list: O(1) at both ends
dq := list.New()
dq.PushBack(1)  // push right
dq.PushFront(0) // push left
front := dq.Front().Value.(int)
back := dq.Back().Value.(int)
dq.Remove(dq.Front()) // pop left
dq.Remove(dq.Back())  // pop right

// Slice deque: fastest when you only push at the back (the monotonic deque case)
idx := []int{}
idx = append(idx, i)      // push back
idx = idx[:len(idx)-1]    // pop back
idx = idx[1:]             // pop front
first, last := idx[0], idx[len(idx)-1]
```

Pushing to the front of a slice is O(n), so use `container/list` or a ring buffer when both ends see heavy pushes.

### 8.4 container/ring: a circular list

| Function / method | Does | Time |
| --- | --- | --- |
| `ring.New(n)` | create a ring of n elements (values nil) | O(n) |
| `r.Value` | this element's value (`any`) | O(1) |
| `r.Next()`, `r.Prev()` | step one element | O(1) |
| `r.Move(k)` | step k elements; negative steps backwards | O(k) |
| `r.Len()` | number of elements | O(n) |
| `r.Do(f)` | call f on every value | O(n) |
| `r.Link(s)` | splice ring s in after r | O(1) |
| `r.Unlink(k)` | remove the k elements after r | O(k) |

```go
import "container/ring"

// Josephus problem: n people in a circle, remove every k-th, return the survivor
func josephus(n, k int) int {
    r := ring.New(n)
    for i := 1; i <= n; i++ {
        r.Value = i
        r = r.Next()
    }
    r = r.Prev() // stand just before person 1
    for r.Len() > 1 {
        r = r.Move(k - 1) // step to just before the k-th person
        r.Unlink(1)       // remove the k-th person
    }
    return r.Value.(int)
}
```

## 9. Heaps and priority queues

A heap gives O(1) access to the smallest (or largest) item and O(log n) push and pop. In Go you get one by implementing the five methods of `heap.Interface` on your own slice type, or by writing a small generic heap once.

### 9.1 How a binary heap works

A heap is a complete binary tree stored in a slice. For index i, the parent is (i-1)/2, the left child 2i+1 and the right child 2i+2. In a min-heap every parent is ≤ its children, so the minimum always sits at index 0.

Push appends at the end and sifts the new item up. Pop moves the last item to the root, shrinks the slice, and sifts the root down.

### 9.2 container/heap

```go
// heap.Interface, from the standard library
type Interface interface {
    sort.Interface // Len() int; Less(i, j int) bool; Swap(i, j int)
    Push(x any)    // append x as element Len()
    Pop() any      // remove and return element Len()-1
}
```

Your `Push` and `Pop` are plain append and truncate; the package calls them and does all the sifting. Always call `heap.Push(h, x)` and `heap.Pop(h)`, never `h.Push` directly.

| Function | Does | Time |
| --- | --- | --- |
| `heap.Init(h)` | turn any slice into a valid heap | O(n) |
| `heap.Push(h, x)` | insert x | O(log n) |
| `heap.Pop(h)` | remove and return the minimum, as `any` | O(log n) |
| `(*h)[0]` | peek at the minimum | O(1) |
| `heap.Fix(h, i)` | restore order after element i changed | O(log n) |
| `heap.Remove(h, i)` | remove and return element i | O(log n) |
| `h.Len()` | number of items | O(1) |

### 9.3 Min-heap and max-heap of ints

```go
import "container/heap"

type IntHeap []int

func (h IntHeap) Len() int           { return len(h) }
func (h IntHeap) Less(i, j int) bool { return h[i] < h[j] } // < gives a min-heap; > gives a max-heap
func (h IntHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

// Push and Pop change the slice's length, so they need pointer receivers
func (h *IntHeap) Push(x any) { *h = append(*h, x.(int)) }
func (h *IntHeap) Pop() any {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[:n-1]
    return x
}

func main() {
    h := &IntHeap{5, 2, 8}
    heap.Init(h) // O(n)
    heap.Push(h, 1)
    fmt.Println((*h)[0]) // 1: peek
    for h.Len() > 0 {
        fmt.Print(heap.Pop(h).(int), " ") // 1 2 5 8
    }
}
```

For a max-heap, flip `Less` to `h[i] > h[j]`. A quick alternative is to push negated values into a min-heap and negate them again on pop.

### 9.4 Priority queue of structs, with priority updates

```go
type Item struct {
    value    string
    priority int
    index    int // position in the heap, kept current by Swap; needed for Fix and Remove
}

type PQ []*Item

func (pq PQ) Len() int           { return len(pq) }
func (pq PQ) Less(i, j int) bool { return pq[i].priority > pq[j].priority } // highest first
func (pq PQ) Swap(i, j int) {
    pq[i], pq[j] = pq[j], pq[i]
    pq[i].index = i
    pq[j].index = j
}

func (pq *PQ) Push(x any) {
    it := x.(*Item)
    it.index = len(*pq)
    *pq = append(*pq, it)
}

func (pq *PQ) Pop() any {
    old := *pq
    n := len(old)
    it := old[n-1]
    old[n-1] = nil // avoid holding a stale pointer
    it.index = -1
    *pq = old[:n-1]
    return it
}

// Change an item's priority in O(log n)
func (pq *PQ) update(it *Item, priority int) {
    it.priority = priority
    heap.Fix(pq, it.index)
}
```

For ties, compare several keys inside `Less`:

```go
func (h H) Less(i, j int) bool {
    if h[i].dist != h[j].dist {
        return h[i].dist < h[j].dist
    }
    return h[i].node < h[j].node
}
```

### 9.5 A generic heap: write once, reuse everywhere

```go
type Heap[T any] struct {
    data []T
    less func(a, b T) bool
}

func NewHeap[T any](less func(a, b T) bool) *Heap[T] { return &Heap[T]{less: less} }

func (h *Heap[T]) Len() int { return len(h.data) }
func (h *Heap[T]) Peek() T  { return h.data[0] }

func (h *Heap[T]) Push(v T) {
    h.data = append(h.data, v)
    i := len(h.data) - 1
    for i > 0 { // sift up
        p := (i - 1) / 2
        if !h.less(h.data[i], h.data[p]) {
            break
        }
        h.data[i], h.data[p] = h.data[p], h.data[i]
        i = p
    }
}

func (h *Heap[T]) Pop() T {
    top := h.data[0]
    n := len(h.data) - 1
    h.data[0] = h.data[n]
    h.data = h.data[:n]
    i := 0
    for { // sift down
        l, r, best := 2*i+1, 2*i+2, i
        if l < n && h.less(h.data[l], h.data[best]) {
            best = l
        }
        if r < n && h.less(h.data[r], h.data[best]) {
            best = r
        }
        if best == i {
            break
        }
        h.data[i], h.data[best] = h.data[best], h.data[i]
        i = best
    }
    return top
}

// Usage: T is inferred from the comparator
minH := NewHeap(func(a, b int) bool { return a < b })
maxH := NewHeap(func(a, b int) bool { return a > b })
type pair struct{ dist, node int }
pq := NewHeap(func(a, b pair) bool { return a.dist < b.dist })
```

### 9.6 When to reach for a heap

The signals are "k largest / smallest", "k-th", "top k frequent", "merge k sorted", "median of a stream", and "always process the cheapest or earliest next" (Dijkstra, Prim, scheduling). For the k largest, keep a min-heap of size k, which costs O(n log k).

```go
// K-th largest element: a min-heap that never holds more than k items
func findKthLargest(nums []int, k int) int {
    h := &IntHeap{}
    for _, v := range nums {
        heap.Push(h, v)
        if h.Len() > k {
            heap.Pop(h) // drop the smallest
        }
    }
    return (*h)[0]
}
```

## 10. Trees: binary trees and BSTs

Most tree problems are one recursive function that asks each child for an answer and combines the two. Learn the four traversals and this "return info up" idea and you can solve the majority of tree questions.

### 10.1 Vocabulary

| Term | Meaning |
| --- | --- |
| Root / leaf | top node / node with no children |
| Depth of a node | edges from the root down to it |
| Height of a tree | edges (or nodes, by convention) on the longest root-to-leaf path |
| Balanced | at every node, left and right heights differ by at most 1 |
| Complete | every level full except possibly the last, filled left to right (heaps) |
| Full / perfect | every node has 0 or 2 children / every level completely full |
| BST | every left-subtree value < node < every right-subtree value |

### 10.2 The node, and building a tree from level order

```go
type TreeNode struct {
    Val         int
    Left, Right *TreeNode
}

// Build from LeetCode-style level order; the marker null stands for a missing child
const null = math.MinInt

func buildTree(vals []int) *TreeNode {
    if len(vals) == 0 || vals[0] == null {
        return nil
    }
    root := &TreeNode{Val: vals[0]}
    queue := []*TreeNode{root}
    i := 1
    for len(queue) > 0 && i < len(vals) {
        node := queue[0]
        queue = queue[1:]
        if i < len(vals) && vals[i] != null {
            node.Left = &TreeNode{Val: vals[i]}
            queue = append(queue, node.Left)
        }
        i++
        if i < len(vals) && vals[i] != null {
            node.Right = &TreeNode{Val: vals[i]}
            queue = append(queue, node.Right)
        }
        i++
    }
    return root
}

// root := buildTree([]int{3, 9, 20, null, null, 15, 7})
```

### 10.3 Traversals

| Order | Visit sequence | Use it for |
| --- | --- | --- |
| Preorder | node, left, right | copying, serializing, paths from the root |
| Inorder | left, node, right | a BST in sorted order |
| Postorder | left, right, node | answers built from the children: height, deletion, subtree sums |
| Level order (BFS) | level by level | per-level answers, minimum depth, right-side view |

```go
// Recursive DFS: move the append to switch between the three orders
func inorderTraversal(root *TreeNode) []int {
    var res []int
    var dfs func(*TreeNode)
    dfs = func(n *TreeNode) {
        if n == nil {
            return
        }
        // preorder: append here
        dfs(n.Left)
        res = append(res, n.Val) // inorder: append here
        dfs(n.Right)
        // postorder: append here
    }
    dfs(root)
    return res
}

// Iterative inorder with an explicit stack
func inorderIter(root *TreeNode) []int {
    var res []int
    var st []*TreeNode
    cur := root
    for cur != nil || len(st) > 0 {
        for cur != nil { // go left as far as possible
            st = append(st, cur)
            cur = cur.Left
        }
        cur = st[len(st)-1]
        st = st[:len(st)-1]
        res = append(res, cur.Val) // visit
        cur = cur.Right            // then the right subtree
    }
    return res
}

// Iterative preorder: push right before left so left is processed first.
// Iterative postorder = this with left/right swapped, then reverse the result.
func preorderIter(root *TreeNode) []int {
    if root == nil {
        return nil
    }
    var res []int
    st := []*TreeNode{root}
    for len(st) > 0 {
        n := st[len(st)-1]
        st = st[:len(st)-1]
        res = append(res, n.Val)
        if n.Right != nil {
            st = append(st, n.Right)
        }
        if n.Left != nil {
            st = append(st, n.Left)
        }
    }
    return res
}

// Level order (BFS), one slice per level
func levelOrder(root *TreeNode) [][]int {
    var res [][]int
    if root == nil {
        return res
    }
    queue := []*TreeNode{root}
    for len(queue) > 0 {
        level := make([]int, 0, len(queue))
        var next []*TreeNode
        for _, n := range queue {
            level = append(level, n.Val)
            if n.Left != nil {
                next = append(next, n.Left)
            }
            if n.Right != nil {
                next = append(next, n.Right)
            }
        }
        res = append(res, level)
        queue = next
    }
    return res
}
```

### 10.4 "Return info up" problems

Each call returns a local answer to its parent and, when needed, updates a global answer held in a closure variable.

```go
func maxDepth(n *TreeNode) int {
    if n == nil {
        return 0
    }
    return 1 + max(maxDepth(n.Left), maxDepth(n.Right))
}

// Diameter: the longest path bends at some node, with length = left height + right height
func diameterOfBinaryTree(root *TreeNode) int {
    best := 0
    var height func(*TreeNode) int
    height = func(n *TreeNode) int {
        if n == nil {
            return 0
        }
        l, r := height(n.Left), height(n.Right)
        best = max(best, l+r) // global answer
        return 1 + max(l, r)  // local answer for the parent
    }
    height(root)
    return best
}

// Balanced check: -1 signals "unbalanced somewhere below"
func isBalanced(root *TreeNode) bool {
    var h func(*TreeNode) int
    h = func(n *TreeNode) int {
        if n == nil {
            return 0
        }
        l, r := h(n.Left), h(n.Right)
        if l < 0 || r < 0 || l-r > 1 || r-l > 1 {
            return -1
        }
        return 1 + max(l, r)
    }
    return h(root) >= 0
}

func invertTree(n *TreeNode) *TreeNode {
    if n == nil {
        return nil
    }
    n.Left, n.Right = invertTree(n.Right), invertTree(n.Left)
    return n
}

func isSameTree(p, q *TreeNode) bool {
    if p == nil || q == nil {
        return p == q // true only if both are nil
    }
    return p.Val == q.Val && isSameTree(p.Left, q.Left) && isSameTree(p.Right, q.Right)
}

// Lowest common ancestor in any binary tree
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    if root == nil || root == p || root == q {
        return root
    }
    l := lowestCommonAncestor(root.Left, p, q)
    r := lowestCommonAncestor(root.Right, p, q)
    if l != nil && r != nil {
        return root // p and q sit on different sides
    }
    if l != nil {
        return l
    }
    return r
}
```

### 10.5 Binary search tree operations

An inorder walk of a BST visits values in sorted order. Search, insert and delete cost O(h): O(log n) when balanced, O(n) when skewed.

```go
func searchBST(n *TreeNode, v int) *TreeNode {
    for n != nil && n.Val != v {
        if v < n.Val {
            n = n.Left
        } else {
            n = n.Right
        }
    }
    return n
}

func insertBST(n *TreeNode, v int) *TreeNode {
    if n == nil {
        return &TreeNode{Val: v}
    }
    if v < n.Val {
        n.Left = insertBST(n.Left, v)
    } else {
        n.Right = insertBST(n.Right, v)
    }
    return n
}

func deleteBST(n *TreeNode, key int) *TreeNode {
    if n == nil {
        return nil
    }
    switch {
    case key < n.Val:
        n.Left = deleteBST(n.Left, key)
    case key > n.Val:
        n.Right = deleteBST(n.Right, key)
    default: // found it
        if n.Left == nil { // zero or one child: splice it out
            return n.Right
        }
        if n.Right == nil {
            return n.Left
        }
        succ := n.Right // two children: copy in the inorder successor
        for succ.Left != nil {
            succ = succ.Left
        }
        n.Val = succ.Val
        n.Right = deleteBST(n.Right, succ.Val)
    }
    return n
}

// Validate by passing down the open range (lo, hi) each node must fall in
func isValidBST(root *TreeNode) bool {
    var ok func(n *TreeNode, lo, hi int) bool
    ok = func(n *TreeNode, lo, hi int) bool {
        if n == nil {
            return true
        }
        if n.Val <= lo || n.Val >= hi {
            return false
        }
        return ok(n.Left, lo, n.Val) && ok(n.Right, n.Val, hi)
    }
    return ok(root, math.MinInt, math.MaxInt)
}

// K-th smallest: iterative inorder that stops at the k-th visit
func kthSmallest(root *TreeNode, k int) int {
    var st []*TreeNode
    cur := root
    for {
        for cur != nil {
            st = append(st, cur)
            cur = cur.Left
        }
        cur = st[len(st)-1]
        st = st[:len(st)-1]
        k--
        if k == 0 {
            return cur.Val
        }
        cur = cur.Right
    }
}

// LCA in a BST: walk down until p and q fall on different sides
func lcaBST(root, p, q *TreeNode) *TreeNode {
    for {
        switch {
        case p.Val < root.Val && q.Val < root.Val:
            root = root.Left
        case p.Val > root.Val && q.Val > root.Val:
            root = root.Right
        default:
            return root
        }
    }
}
```

| Operation | Balanced BST | Skewed BST | Unordered binary tree |
| --- | --- | --- | --- |
| Search | O(log n) | O(n) | O(n) |
| Insert / delete | O(log n) | O(n) | not defined |
| Any traversal | O(n) | O(n) | O(n) |
| Recursion stack | O(log n) | O(n) | O(height) |

### 10.6 Go has no TreeMap

The standard library has no balanced BST or ordered map (the equivalent of Java's `TreeMap` or C++'s `std::map`). Use a sorted slice with `slices.BinarySearch` when inserts are rare, a heap when you only need the min or max, or a Fenwick / segment tree over compressed values for rank queries (Section 11). Where third-party code is allowed, `github.com/emirpasic/gods` (red-black tree, treemap) and `github.com/google/btree` are common choices.

## 11. Advanced structures: trie, union-find, Fenwick tree, segment tree

These four answer questions that plain structures cannot answer fast: prefix lookups (trie), dynamic connectivity (union-find), and range queries under updates (Fenwick and segment trees).

| Structure | Answers | Build | Query | Update |
| --- | --- | --- | --- | --- |
| Trie | does a word or prefix exist; autocomplete | O(total characters) | O(word length) | O(word length) |
| Union-find | are a and b connected; how many groups | O(n) | nearly O(1) | nearly O(1) |
| Fenwick tree | prefix and range sums with point updates | O(n log n) | O(log n) | O(log n) |
| Segment tree | range sum / min / max, point or range updates | O(n) | O(log n) | O(log n) |

With no updates at all, plain prefix sums (Section 20) beat all of these. Choose Fenwick for sums with point updates, and a segment tree for min / max or range updates.

### 11.1 Trie (prefix tree)

```go
type TrieNode struct {
    children [26]*TrieNode // fastest for lowercase letters; use map[rune]*TrieNode otherwise
    isEnd    bool          // a word ends at this node
    count    int           // words passing through this node (for prefix counts)
}

type Trie struct{ root *TrieNode }

func NewTrie() *Trie { return &Trie{root: &TrieNode{}} }

func (t *Trie) Insert(word string) {
    n := t.root
    for i := 0; i < len(word); i++ {
        c := word[i] - 'a'
        if n.children[c] == nil {
            n.children[c] = &TrieNode{}
        }
        n = n.children[c]
        n.count++
    }
    n.isEnd = true
}

// walk returns the node reached by following s, or nil
func (t *Trie) walk(s string) *TrieNode {
    n := t.root
    for i := 0; i < len(s); i++ {
        n = n.children[s[i]-'a']
        if n == nil {
            return nil
        }
    }
    return n
}

func (t *Trie) Search(word string) bool {
    n := t.walk(word)
    return n != nil && n.isEnd
}

func (t *Trie) StartsWith(prefix string) bool { return t.walk(prefix) != nil }

func (t *Trie) CountPrefix(prefix string) int {
    if n := t.walk(prefix); n != nil {
        return n.count
    }
    return 0
}

// Autocomplete: collect every word under a prefix with DFS + backtracking
func (t *Trie) WithPrefix(prefix string) []string {
    var res []string
    start := t.walk(prefix)
    if start == nil {
        return res
    }
    buf := []byte(prefix)
    var dfs func(*TrieNode)
    dfs = func(n *TrieNode) {
        if n.isEnd {
            res = append(res, string(buf))
        }
        for c, child := range n.children {
            if child != nil {
                buf = append(buf, byte('a'+c)) // choose
                dfs(child)                     // explore
                buf = buf[:len(buf)-1]         // un-choose
            }
        }
    }
    dfs(start)
    return res
}
```

### 11.2 Union-find (disjoint set union)

Path compression plus union by size makes each operation O(α(n)), where α is the inverse Ackermann function; it never exceeds 4 in practice.

```go
type DSU struct {
    parent, size []int
    count        int // number of separate groups
}

func NewDSU(n int) *DSU {
    d := &DSU{parent: make([]int, n), size: make([]int, n), count: n}
    for i := range n {
        d.parent[i] = i // every element starts as its own root
        d.size[i] = 1
    }
    return d
}

// Find the root, halving the path as it goes
func (d *DSU) Find(x int) int {
    for d.parent[x] != x {
        d.parent[x] = d.parent[d.parent[x]]
        x = d.parent[x]
    }
    return x
}

// Union attaches the smaller tree under the larger; false means already connected
func (d *DSU) Union(a, b int) bool {
    ra, rb := d.Find(a), d.Find(b)
    if ra == rb {
        return false
    }
    if d.size[ra] < d.size[rb] {
        ra, rb = rb, ra
    }
    d.parent[rb] = ra
    d.size[ra] += d.size[rb]
    d.count--
    return true
}

func (d *DSU) Connected(a, b int) bool { return d.Find(a) == d.Find(b) }
func (d *DSU) SizeOf(x int) int        { return d.size[d.Find(x)] }

// Example, Redundant Connection: the first edge joining two already-connected nodes closes a cycle
func findRedundantConnection(edges [][]int) []int {
    d := NewDSU(len(edges) + 1)
    for _, e := range edges {
        if !d.Union(e[0], e[1]) {
            return e
        }
    }
    return nil
}
```

### 11.3 Fenwick tree (binary indexed tree)

The tree is 1-indexed. Position i stores the sum of a block whose length is the lowest set bit of i, which `i & -i` isolates.

```go
type Fenwick struct{ tree []int }

func NewFenwick(n int) *Fenwick { return &Fenwick{tree: make([]int, n+1)} }

// Add delta to element i (1-based)
func (f *Fenwick) Add(i, delta int) {
    for ; i < len(f.tree); i += i & -i {
        f.tree[i] += delta
    }
}

// Sum of elements 1..i
func (f *Fenwick) Sum(i int) int {
    s := 0
    for ; i > 0; i -= i & -i {
        s += f.tree[i]
    }
    return s
}

// Sum of elements l..r (1-based, inclusive)
func (f *Fenwick) RangeSum(l, r int) int { return f.Sum(r) - f.Sum(l-1) }

// Example: count inversions (i < j with a[i] > a[j]) in O(n log n)
func countInversions(a []int) int {
    ranks := slices.Clone(a)
    slices.Sort(ranks)
    ranks = slices.Compact(ranks) // coordinate compression: value -> rank
    f := NewFenwick(len(ranks))
    inv := 0
    for i := len(a) - 1; i >= 0; i-- { // walk right to left
        r, _ := slices.BinarySearch(ranks, a[i])
        inv += f.Sum(r) // smaller values already seen on the right
        f.Add(r+1, 1)
    }
    return inv
}
```

### 11.4 Segment tree

The iterative version is short and fast for point updates with range queries. Change `+` to `min` or `max` (and the starting `res`) to get range min or max.

```go
type SegTree struct {
    n    int
    tree []int
}

func NewSegTree(nums []int) *SegTree {
    n := len(nums)
    t := make([]int, 2*n)
    copy(t[n:], nums) // leaves live at n..2n-1
    for i := n - 1; i > 0; i-- {
        t[i] = t[2*i] + t[2*i+1] // parent = merge of its children
    }
    return &SegTree{n, t}
}

// Set nums[i] = v
func (s *SegTree) Update(i, v int) {
    i += s.n
    s.tree[i] = v
    for i > 1 {
        i /= 2
        s.tree[i] = s.tree[2*i] + s.tree[2*i+1]
    }
}

// Sum of nums[l..r], 0-based and inclusive
func (s *SegTree) Query(l, r int) int {
    res := 0
    for l, r = l+s.n, r+s.n+1; l < r; l, r = l/2, r/2 {
        if l&1 == 1 {
            res += s.tree[l]
            l++
        }
        if r&1 == 1 {
            r--
            res += s.tree[r]
        }
    }
    return res
}
```

For range updates ("add v to every element in \[l, r\]"), use a recursive tree with lazy propagation: pending additions are stored at a node and pushed to its children only when needed.

```go
type LazySeg struct {
    sum, lazy []int
    n         int
}

func NewLazySeg(nums []int) *LazySeg {
    n := len(nums)
    s := &LazySeg{sum: make([]int, 4*n), lazy: make([]int, 4*n), n: n}
    var build func(node, l, r int)
    build = func(node, l, r int) {
        if l == r {
            s.sum[node] = nums[l]
            return
        }
        m := (l + r) / 2
        build(2*node, l, m)
        build(2*node+1, m+1, r)
        s.sum[node] = s.sum[2*node] + s.sum[2*node+1]
    }
    if n > 0 {
        build(1, 0, n-1)
    }
    return s
}

func (s *LazySeg) apply(node, l, r, v int) {
    s.sum[node] += v * (r - l + 1)
    s.lazy[node] += v
}

func (s *LazySeg) push(node, l, r int) {
    if s.lazy[node] != 0 {
        m := (l + r) / 2
        s.apply(2*node, l, m, s.lazy[node])
        s.apply(2*node+1, m+1, r, s.lazy[node])
        s.lazy[node] = 0
    }
}

func (s *LazySeg) add(node, l, r, ql, qr, v int) {
    if qr < l || r < ql {
        return // no overlap
    }
    if ql <= l && r <= qr {
        s.apply(node, l, r, v) // fully covered: store it lazily
        return
    }
    s.push(node, l, r)
    m := (l + r) / 2
    s.add(2*node, l, m, ql, qr, v)
    s.add(2*node+1, m+1, r, ql, qr, v)
    s.sum[node] = s.sum[2*node] + s.sum[2*node+1]
}

func (s *LazySeg) query(node, l, r, ql, qr int) int {
    if qr < l || r < ql {
        return 0
    }
    if ql <= l && r <= qr {
        return s.sum[node]
    }
    s.push(node, l, r)
    m := (l + r) / 2
    return s.query(2*node, l, m, ql, qr) + s.query(2*node+1, m+1, r, ql, qr)
}

func (s *LazySeg) RangeAdd(l, r, v int)  { s.add(1, 0, s.n-1, l, r, v) }
func (s *LazySeg) RangeSum(l, r int) int { return s.query(1, 0, s.n-1, l, r) }
```

## 12. Graphs: representations, BFS and DFS

Build an adjacency list, then use BFS for shortest paths in unweighted graphs and DFS for exploring, components and cycles. Both run in O(V + E) time and O(V) space.

### 12.1 Terms

| Term | Meaning |
| --- | --- |
| Directed / undirected | edges have a direction / edges go both ways |
| Weighted | each edge carries a cost |
| Degree | number of edges at a node (in-degree and out-degree when directed) |
| Connected component | a maximal set of nodes that can reach each other |
| Cycle | a path that returns to its start |
| DAG | directed acyclic graph: has a topological order |
| Bipartite | nodes split into two sides with every edge crossing sides |

### 12.2 Representations

| Representation | Go type | Space | Edge u→v exists? | Best for |
| --- | --- | --- | --- | --- |
| Adjacency list | `[][]int` | O(V + E) | O(degree) | almost everything |
| Weighted adjacency list | `[][]Edge` | O(V + E) | O(degree) | Dijkstra, Prim |
| Map adjacency | `map[string][]string` | O(V + E) | O(degree) | non-integer node labels |
| Adjacency matrix | `[][]bool`, `[][]int` | O(V²) | O(1) | dense graphs, Floyd-Warshall |
| Edge list | `[][3]int{u, v, w}` | O(E) | O(E) | Kruskal, Bellman-Ford |
| Implicit grid | `[][]byte` | O(rows × cols) | O(1) | islands, mazes |

```go
// n nodes labelled 0..n-1, edges given as [][]int{{u, v}, ...}
func buildGraph(n int, edges [][]int, directed bool) [][]int {
    adj := make([][]int, n)
    for _, e := range edges {
        u, v := e[0], e[1]
        adj[u] = append(adj[u], v)
        if !directed {
            adj[v] = append(adj[v], u)
        }
    }
    return adj
}

// Weighted: edges given as {u, v, w}
type Edge struct{ to, w int }

wadj := make([][]Edge, n)
for _, e := range edges {
    wadj[e[0]] = append(wadj[e[0]], Edge{e[1], e[2]})
}

// String labels
g := map[string][]string{}
g["a"] = append(g["a"], "b")

// Adjacency matrix
mat := make([][]bool, n)
for i := range mat {
    mat[i] = make([]bool, n)
}
mat[u][v] = true
```

### 12.3 Breadth-first search

BFS explores in rings of increasing distance, so the first time it reaches a node is along a shortest path.

```go
// Distance in edges from src to every node (-1 = unreachable), plus parents for paths
func bfs(adj [][]int, src int) (dist, parent []int) {
    dist = make([]int, len(adj))
    parent = make([]int, len(adj))
    for i := range dist {
        dist[i], parent[i] = -1, -1
    }
    dist[src] = 0
    queue := []int{src}
    for len(queue) > 0 {
        u := queue[0]
        queue = queue[1:]
        for _, v := range adj[u] {
            if dist[v] == -1 { // mark when ENQUEUED, never when dequeued, or nodes repeat
                dist[v] = dist[u] + 1
                parent[v] = u
                queue = append(queue, v)
            }
        }
    }
    return dist, parent
}

// Rebuild the path src -> dst from the parent slice (check dist[dst] != -1 first)
func pathTo(parent []int, dst int) []int {
    var p []int
    for v := dst; v != -1; v = parent[v] {
        p = append(p, v)
    }
    slices.Reverse(p)
    return p
}
```

### 12.4 Depth-first search

```go
// Recursive DFS over every component
func dfsAll(adj [][]int) {
    visited := make([]bool, len(adj))
    var dfs func(int)
    dfs = func(u int) {
        visited[u] = true
        // pre-order work on u
        for _, v := range adj[u] {
            if !visited[v] {
                dfs(v)
            }
        }
        // post-order work on u (e.g. record topological order)
    }
    for u := range adj {
        if !visited[u] {
            dfs(u) // one call per connected component
        }
    }
}

// Iterative DFS with an explicit stack (no recursion depth to worry about)
func dfsIter(adj [][]int, src int) []int {
    visited := make([]bool, len(adj))
    var order []int
    stack := []int{src}
    for len(stack) > 0 {
        u := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        if visited[u] {
            continue
        }
        visited[u] = true
        order = append(order, u)
        for _, v := range adj[u] {
            if !visited[v] {
                stack = append(stack, v)
            }
        }
    }
    return order
}
```

### 12.5 Grids as graphs

Each cell is a node and its 4 (or 8) neighbours are its edges. When you need one integer per cell, encode (r, c) as `r*cols + c`.

```go
var dirs = [4][2]int{{-1, 0}, {1, 0}, {0, -1}, {0, 1}}

// Number of Islands: flood-fill each new island with DFS
func numIslands(grid [][]byte) int {
    rows, cols := len(grid), len(grid[0])
    var sink func(r, c int)
    sink = func(r, c int) {
        if r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] != '1' {
            return
        }
        grid[r][c] = '0' // mark visited by overwriting the cell
        for _, d := range dirs {
            sink(r+d[0], c+d[1])
        }
    }
    count := 0
    for r := range rows {
        for c := range cols {
            if grid[r][c] == '1' {
                count++
                sink(r, c)
            }
        }
    }
    return count
}

// Shortest path through open cells (0) of a maze with BFS
func shortestPath(grid [][]int, sr, sc, tr, tc int) int {
    rows, cols := len(grid), len(grid[0])
    dist := make([][]int, rows)
    for i := range dist {
        dist[i] = make([]int, cols)
        for j := range dist[i] {
            dist[i][j] = -1
        }
    }
    dist[sr][sc] = 0
    queue := [][2]int{{sr, sc}}
    for len(queue) > 0 {
        cur := queue[0]
        queue = queue[1:]
        r, c := cur[0], cur[1]
        if r == tr && c == tc {
            return dist[r][c]
        }
        for _, d := range dirs {
            nr, nc := r+d[0], c+d[1]
            if nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc] == 0 && dist[nr][nc] == -1 {
                dist[nr][nc] = dist[r][c] + 1
                queue = append(queue, [2]int{nr, nc})
            }
        }
    }
    return -1
}
```

### 12.6 Cycle detection and bipartite check

```go
// Undirected: a visited neighbour that is not our parent means a cycle
func hasCycleUndirected(adj [][]int) bool {
    visited := make([]bool, len(adj))
    var dfs func(u, parent int) bool
    dfs = func(u, parent int) bool {
        visited[u] = true
        for _, v := range adj[u] {
            if !visited[v] {
                if dfs(v, u) {
                    return true
                }
            } else if v != parent {
                return true
            }
        }
        return false
    }
    for u := range adj {
        if !visited[u] && dfs(u, -1) {
            return true
        }
    }
    return false
}

// Directed: three colours; meeting a GRAY node (still on the current path) means a cycle
func hasCycleDirected(adj [][]int) bool {
    const white, gray, black = 0, 1, 2
    color := make([]int, len(adj))
    var dfs func(int) bool
    dfs = func(u int) bool {
        color[u] = gray
        for _, v := range adj[u] {
            if color[v] == gray {
                return true
            }
            if color[v] == white && dfs(v) {
                return true
            }
        }
        color[u] = black // finished: every path from u is explored
        return false
    }
    for u := range adj {
        if color[u] == white && dfs(u) {
            return true
        }
    }
    return false
}

// Bipartite: 2-colour with BFS; an edge joining two equal colours fails
func isBipartite(adj [][]int) bool {
    color := make([]int, len(adj)) // 0 = uncoloured, 1 and -1 = the two sides
    for s := range adj {
        if color[s] != 0 {
            continue
        }
        color[s] = 1
        queue := []int{s}
        for len(queue) > 0 {
            u := queue[0]
            queue = queue[1:]
            for _, v := range adj[u] {
                if color[v] == 0 {
                    color[v] = -color[u]
                    queue = append(queue, v)
                } else if color[v] == color[u] {
                    return false
                }
            }
        }
    }
    return true
}
```

### 12.7 BFS or DFS?

| You need | Use |
| --- | --- |
| Shortest path, unweighted | BFS |
| Minimum number of steps, level by level | BFS |
| All paths, or any path with backtracking | DFS |
| Connected components | DFS, BFS or union-find |
| Cycle detection | DFS with colours; union-find for undirected |
| Topological order | DFS postorder or Kahn's BFS (Section 18) |
| Shortest path with weights | Dijkstra or Bellman-Ford (Section 18) |

## 13. Sorting

In real solutions, call `slices.Sort` or `slices.SortFunc`: O(n log n) pattern-defeating quicksort. Implement the classic algorithms yourself to learn divide and conquer, partitioning and stability, which interviews test directly.

### 13.1 Sorting with the standard library

```go
nums := []int{5, 1, 4}
slices.Sort(nums) // ascending; works for ints, floats and strings
slices.SortFunc(nums, func(a, b int) int { return cmp.Compare(b, a) }) // descending

sort.Ints(nums) // older API, still common: sort.Strings, sort.Float64s
sort.Slice(people, func(i, j int) bool { return people[i].Age < people[j].Age }) // less by index
sort.SliceStable(people, func(i, j int) bool { return people[i].Age < people[j].Age })
sort.Sort(sort.Reverse(sort.IntSlice(nums))) // descending via sort.Interface

// Intervals by start, then by end
slices.SortFunc(intervals, func(a, b []int) int {
    return cmp.Or(cmp.Compare(a[0], b[0]), cmp.Compare(a[1], b[1]))
})

// Argsort: order the indices by value without moving the data
idx := make([]int, len(nums))
for i := range idx {
    idx[i] = i
}
slices.SortFunc(idx, func(a, b int) int { return cmp.Compare(nums[a], nums[b]) })

// A custom type that implements sort.Interface
type ByLen []string

func (s ByLen) Len() int           { return len(s) }
func (s ByLen) Less(i, j int) bool { return len(s[i]) < len(s[j]) }
func (s ByLen) Swap(i, j int)      { s[i], s[j] = s[j], s[i] }

sort.Sort(ByLen(words))
```

| API | Comparator | Stable | Note |
| --- | --- | --- | --- |
| `slices.Sort(s)` | natural order | no | generic and fastest |
| `slices.SortFunc(s, cmp)` | `func(a, b T) int`: negative, 0, positive | no | use `cmp.Compare` to avoid overflow from `a - b` |
| `slices.SortStableFunc(s, cmp)` | `func(a, b T) int` | yes |  |
| `sort.Slice(s, less)` | `func(i, j int) bool` by index | no | slower (reflection) |
| `sort.SliceStable(s, less)` | `func(i, j int) bool` | yes |  |
| `sort.Sort(x)`, `sort.Stable(x)` | `sort.Interface` | no / yes |  |

### 13.2 Algorithm comparison

| Algorithm | Best | Average | Worst | Extra space | Stable | Use when |
| --- | --- | --- | --- | --- | --- | --- |
| Bubble | O(n) | O(n²) | O(n²) | O(1) | yes | teaching only |
| Selection | O(n²) | O(n²) | O(n²) | O(1) | no | you must minimise swaps |
| Insertion | O(n) | O(n²) | O(n²) | O(1) | yes | small or nearly sorted input |
| Merge | O(n log n) | O(n log n) | O(n log n) | O(n) | yes | linked lists, stability, counting inversions |
| Quick | O(n log n) | O(n log n) | O(n²) | O(log n) | no | fastest in-place in practice |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | no | guaranteed bound with O(1) space |
| Counting | O(n + k) | O(n + k) | O(n + k) | O(k) | yes | small integer range k |
| Radix | O(d·(n + b)) | O(d·(n + b)) | O(d·(n + b)) | O(n + b) | yes | fixed-width integers (d digits, base b) |

### 13.3 Implementations

```go
func bubbleSort(a []int) {
    for i := 0; i < len(a)-1; i++ {
        swapped := false
        for j := 0; j < len(a)-1-i; j++ { // the largest i items are already in place
            if a[j] > a[j+1] {
                a[j], a[j+1] = a[j+1], a[j]
                swapped = true
            }
        }
        if !swapped {
            return // no swaps: already sorted, the O(n) best case
        }
    }
}

func selectionSort(a []int) {
    for i := range a {
        m := i
        for j := i + 1; j < len(a); j++ {
            if a[j] < a[m] {
                m = j
            }
        }
        a[i], a[m] = a[m], a[i]
    }
}

func insertionSort(a []int) {
    for i := 1; i < len(a); i++ {
        key, j := a[i], i-1
        for j >= 0 && a[j] > key { // shift bigger items one step right
            a[j+1] = a[j]
            j--
        }
        a[j+1] = key
    }
}

func mergeSort(a []int) []int {
    if len(a) <= 1 {
        return a
    }
    mid := len(a) / 2
    return merge(mergeSort(a[:mid]), mergeSort(a[mid:]))
}

func merge(l, r []int) []int {
    res := make([]int, 0, len(l)+len(r))
    i, j := 0, 0
    for i < len(l) && j < len(r) {
        if l[i] <= r[j] { // <= keeps equal items in order: stable
            res = append(res, l[i])
            i++
        } else {
            res = append(res, r[j])
            j++
        }
    }
    res = append(res, l[i:]...)
    return append(res, r[j:]...)
}

func quickSort(a []int) {
    if len(a) <= 1 {
        return
    }
    p := partition(a)
    quickSort(a[:p])
    quickSort(a[p+1:])
}

// Lomuto partition around a random pivot (math/rand); returns the pivot's final index
func partition(a []int) int {
    r := rand.Intn(len(a)) // a random pivot avoids O(n²) on sorted input
    last := len(a) - 1
    a[r], a[last] = a[last], a[r]
    pivot, i := a[last], 0
    for j := 0; j < last; j++ {
        if a[j] < pivot {
            a[i], a[j] = a[j], a[i]
            i++
        }
    }
    a[i], a[last] = a[last], a[i]
    return i
}

func heapSort(a []int) {
    n := len(a)
    for i := n/2 - 1; i >= 0; i-- { // build a max-heap in O(n)
        siftDown(a, i, n)
    }
    for end := n - 1; end > 0; end-- {
        a[0], a[end] = a[end], a[0] // move the max to the end
        siftDown(a, 0, end)
    }
}

func siftDown(a []int, i, n int) {
    for {
        l, r, big := 2*i+1, 2*i+2, i
        if l < n && a[l] > a[big] {
            big = l
        }
        if r < n && a[r] > a[big] {
            big = r
        }
        if big == i {
            return
        }
        a[i], a[big] = a[big], a[i]
        i = big
    }
}

// Counting sort for values in 0..maxVal
func countingSort(a []int, maxVal int) {
    count := make([]int, maxVal+1)
    for _, v := range a {
        count[v]++
    }
    i := 0
    for v, c := range count {
        for ; c > 0; c-- {
            a[i] = v
            i++
        }
    }
}

// LSD radix sort for non-negative ints, base 10
func radixSort(a []int) {
    if len(a) == 0 {
        return
    }
    maxV := slices.Max(a)
    buf := make([]int, len(a))
    for exp := 1; maxV/exp > 0; exp *= 10 {
        var count [10]int
        for _, v := range a {
            count[(v/exp)%10]++
        }
        for d := 1; d < 10; d++ {
            count[d] += count[d-1] // prefix sums give each digit's end position
        }
        for i := len(a) - 1; i >= 0; i-- { // walking backwards keeps it stable
            d := (a[i] / exp) % 10
            count[d]--
            buf[count[d]] = a[i]
        }
        copy(a, buf)
    }
}
```

### 13.4 Quickselect: the k-th smallest in O(n) average

```go
func quickSelect(a []int, k int) int { // k is 0-based; reorders a
    lo, hi := 0, len(a)-1
    for {
        p := lo + partition(a[lo:hi+1]) // partition a sub-slice, then shift back by lo
        switch {
        case p == k:
            return a[p]
        case p < k:
            lo = p + 1
        default:
            hi = p - 1
        }
    }
}
```

### 13.5 Custom-order sorting

Sorting first often unlocks two pointers, greedy interval scheduling, and dedupe by comparing neighbours. Some problems are nothing but the right comparator:

```go
// Largest Number: put a before b when a+b > b+a
func largestNumber(nums []int) string {
    s := make([]string, len(nums))
    for i, v := range nums {
        s[i] = strconv.Itoa(v)
    }
    slices.SortFunc(s, func(a, b string) int { return strings.Compare(b+a, a+b) })
    if s[0] == "0" {
        return "0"
    }
    return strings.Join(s, "")
}
```

## 14. Searching and binary search

Binary search works on any monotonic yes/no question, not only on sorted arrays. Master one template, "find the first index where a condition becomes true", and every variant follows from it.

Linear search (`slices.Index`, a plain loop) is O(n) and right for unsorted data queried once. Binary search is O(log n) and needs sorted data or a monotonic condition.

### 14.1 The one template: first true

```go
// Smallest i in [lo, hi) with ok(i) == true, or hi if there is none.
// ok must be monotonic: false false ... false true true ... true
func firstTrue(lo, hi int, ok func(int) bool) int {
    for lo < hi {
        mid := lo + (hi-lo)/2 // avoids the overflow of (lo+hi)/2
        if ok(mid) {
            hi = mid // mid may be the answer: keep it
        } else {
            lo = mid + 1 // mid is not the answer: drop it
        }
    }
    return lo
}
```

`sort.Search(n, f)` from the standard library is exactly `firstTrue(0, n, f)`.

### 14.2 Variants on a sorted slice

```go
// Exact match, the classic form with an inclusive hi
func search(a []int, t int) int {
    lo, hi := 0, len(a)-1
    for lo <= hi {
        mid := lo + (hi-lo)/2
        switch {
        case a[mid] == t:
            return mid
        case a[mid] < t:
            lo = mid + 1
        default:
            hi = mid - 1
        }
    }
    return -1
}

// Lower bound: first index with a[i] >= t (also the insert position)
lower := sort.Search(len(a), func(i int) bool { return a[i] >= t })
// Upper bound: first index with a[i] > t
upper := sort.Search(len(a), func(i int) bool { return a[i] > t })

count := upper - lower        // occurrences of t
first, last := lower, upper-1 // first and last position, valid when lower < len(a) && a[lower] == t
floorIdx := upper - 1         // last index with a[i] <= t (-1 if none)
ceilIdx := lower              // first index with a[i] >= t (len(a) if none)
```

| Function | Returns |
| --- | --- |
| `sort.Search(n, f)` | smallest i in \[0, n) where f(i) is true, else n |
| `sort.SearchInts(a, x)` | lower bound of x in a sorted `[]int` (also `SearchStrings`, `SearchFloat64s`) |
| `slices.BinarySearch(a, x)` | (lower bound, whether x was found) |
| `slices.BinarySearchFunc(a, t, cmp)` | the same with a comparator |

### 14.3 Binary search on the answer

The signals are "minimise the maximum", "maximise the minimum", "the smallest speed / capacity / time such that it is possible", and an answer range as large as 10^9. Guess an answer x, test `feasible(x)` in O(n), and binary search over x because feasibility is monotonic. Total cost: O(n log(range)).

```go
// Koko Eating Bananas: the slowest speed that finishes within h hours
func minEatingSpeed(piles []int, h int) int {
    canFinish := func(k int) bool {
        hours := 0
        for _, p := range piles {
            hours += (p + k - 1) / k // ceil(p / k) in integer maths
        }
        return hours <= h
    }
    return firstTrue(1, slices.Max(piles)+1, canFinish)
}

// Capacity to Ship Packages Within D Days
func shipWithinDays(weights []int, days int) int {
    fits := func(limit int) bool {
        need, load := 1, 0
        for _, w := range weights {
            if load+w > limit {
                need++
                load = 0
            }
            load += w
        }
        return need <= days
    }
    lo, hi := slices.Max(weights), 0
    for _, w := range weights {
        hi += w
    }
    return firstTrue(lo, hi+1, fits)
}
```

### 14.4 Rotated arrays, peaks and matrices

```go
// Search in a rotated sorted array: one half is always sorted
func searchRotated(a []int, t int) int {
    lo, hi := 0, len(a)-1
    for lo <= hi {
        mid := lo + (hi-lo)/2
        if a[mid] == t {
            return mid
        }
        if a[lo] <= a[mid] { // left half is sorted
            if a[lo] <= t && t < a[mid] {
                hi = mid - 1
            } else {
                lo = mid + 1
            }
        } else { // right half is sorted
            if a[mid] < t && t <= a[hi] {
                lo = mid + 1
            } else {
                hi = mid - 1
            }
        }
    }
    return -1
}

// Minimum of a rotated sorted array (no duplicates)
func findMin(a []int) int {
    lo, hi := 0, len(a)-1
    for lo < hi {
        mid := lo + (hi-lo)/2
        if a[mid] > a[hi] {
            lo = mid + 1 // the minimum is right of mid
        } else {
            hi = mid // the minimum is at mid or to its left
        }
    }
    return a[lo]
}

// Peak element: always climb toward the higher neighbour
func findPeakElement(a []int) int {
    lo, hi := 0, len(a)-1
    for lo < hi {
        mid := lo + (hi-lo)/2
        if a[mid] < a[mid+1] {
            lo = mid + 1
        } else {
            hi = mid
        }
    }
    return lo
}

// Sorted matrix where each row continues the previous one: treat it as one flat array
func searchMatrix(m [][]int, t int) bool {
    cols := len(m[0])
    lo, hi := 0, len(m)*cols-1
    for lo <= hi {
        mid := lo + (hi-lo)/2
        v := m[mid/cols][mid%cols] // flat index -> (row, col)
        switch {
        case v == t:
            return true
        case v < t:
            lo = mid + 1
        default:
            hi = mid - 1
        }
    }
    return false
}

// Integer square root: the last m with m*m <= x (safe for x up to about 3×10^18)
func mySqrt(x int) int {
    return firstTrue(0, x+1, func(m int) bool { return m > x/max(m, 1) }) - 1
}

// Binary search on real numbers: run a fixed number of iterations
func sqrtFloat(x float64) float64 {
    lo, hi := 0.0, max(1.0, x)
    for range 100 {
        mid := (lo + hi) / 2
        if mid*mid < x {
            lo = mid
        } else {
            hi = mid
        }
    }
    return lo
}
```

### 14.5 Binary search bugs and their fixes

| Bug | Fix |
| --- | --- |
| Infinite loop from `lo = mid` | in the first-true template always write `lo = mid + 1`; for "last true" use `mid := lo + (hi-lo+1)/2` |
| Overflow in `(lo + hi) / 2` | `lo + (hi-lo)/2` |
| Mixing inclusive and exclusive bounds | pick one: `[lo, hi)` with `lo < hi`, or `[lo, hi]` with `lo <= hi` |
| Condition not monotonic | binary search does not apply; rethink the predicate |

## 15. Recursion and backtracking

Recursion solves a problem through smaller copies of itself. Backtracking is recursion that builds a candidate step by step and undoes each choice, always in the order choose → explore → un-choose.

### 15.1 Anatomy of a recursive function

1. A base case that stops the recursion.
2. A recursive case that makes the problem strictly smaller.
3. Trust that the recursive call returns the right answer for the smaller problem, then combine.

```go
func factorial(n int) int {
    if n <= 1 {
        return 1 // base case
    }
    return n * factorial(n-1) // smaller subproblem
}

// Fast exponentiation in O(log n): halve the exponent each call
func power(x float64, n int) float64 {
    if n == 0 {
        return 1
    }
    if n < 0 {
        return 1 / power(x, -n)
    }
    half := power(x, n/2)
    if n%2 == 0 {
        return half * half
    }
    return half * half * x
}

// Memoization: cache results so each subproblem is solved once (leads into DP, Section 16)
func fib(n int, memo map[int]int) int {
    if n < 2 {
        return n
    }
    if v, ok := memo[n]; ok {
        return v
    }
    memo[n] = fib(n-1, memo) + fib(n-2, memo)
    return memo[n]
}
```

Go stacks grow dynamically (up to 1 GB by default on 64-bit), so recursion depth of 10^5 to 10^6 usually works. Beyond that, rewrite the recursion with an explicit stack.

### 15.2 The backtracking template

The signals are "return all", "generate every", "list all combinations / permutations / subsets / partitions", small inputs (n ≤ 20), and constraint puzzles like Sudoku or N-Queens.

```go
// Pseudocode shape of every backtracking solution
var res [][]int
var path []int
var dfs func(state)
dfs = func(state) {
    if isComplete(state) {
        res = append(res, slices.Clone(path)) // COPY: path keeps changing after this
        return
    }
    for _, c := range choices(state) {
        if !valid(c) {
            continue // prune dead branches early
        }
        path = append(path, c)    // choose
        dfs(next(state, c))       // explore
        path = path[:len(path)-1] // un-choose
    }
}
```

The most common Go bug here is `res = append(res, path)` without the clone. Every stored result then shares one backing array and ends up overwritten.

### 15.3 Subsets

```go
func subsets(nums []int) [][]int {
    var res [][]int
    var path []int
    var dfs func(start int)
    dfs = func(start int) {
        res = append(res, slices.Clone(path)) // every node of the tree is a subset
        for i := start; i < len(nums); i++ {
            path = append(path, nums[i])
            dfs(i + 1) // only look forward: no reuse, no reordering
            path = path[:len(path)-1]
        }
    }
    dfs(0)
    return res
}
```

With duplicate values (Subsets II), sort `nums` first and skip repeats at the same depth: `if i > start && nums[i] == nums[i-1] { continue }`.

### 15.4 Permutations

```go
func permute(nums []int) [][]int {
    var res [][]int
    path := make([]int, 0, len(nums))
    used := make([]bool, len(nums))
    var dfs func()
    dfs = func() {
        if len(path) == len(nums) {
            res = append(res, slices.Clone(path))
            return
        }
        for i, v := range nums {
            if used[i] {
                continue
            }
            used[i] = true
            path = append(path, v)
            dfs()
            path = path[:len(path)-1]
            used[i] = false
        }
    }
    dfs()
    return res
}
```

With duplicates (Permutations II), sort first and skip with `if used[i] || (i > 0 && nums[i] == nums[i-1] && !used[i-1]) { continue }`.

### 15.5 Combinations, combination sum and parentheses

```go
// Every k-sized combination of 1..n
func combine(n, k int) [][]int {
    var res [][]int
    var path []int
    var dfs func(start int)
    dfs = func(start int) {
        if len(path) == k {
            res = append(res, slices.Clone(path))
            return
        }
        need := k - len(path)
        for i := start; i <= n-need+1; i++ { // prune: leave room for the remaining picks
            path = append(path, i)
            dfs(i + 1)
            path = path[:len(path)-1]
        }
    }
    dfs(1)
    return res
}

// Combination Sum: a candidate may be reused, so recurse with i instead of i+1
func combinationSum(cands []int, target int) [][]int {
    slices.Sort(cands)
    var res [][]int
    var path []int
    var dfs func(start, remain int)
    dfs = func(start, remain int) {
        if remain == 0 {
            res = append(res, slices.Clone(path))
            return
        }
        for i := start; i < len(cands) && cands[i] <= remain; i++ { // sorted, so stop early
            path = append(path, cands[i])
            dfs(i, remain-cands[i])
            path = path[:len(path)-1]
        }
    }
    dfs(0, target)
    return res
}

// Generate Parentheses: add '(' while any remain, ')' only when it keeps the string valid
func generateParenthesis(n int) []string {
    var res []string
    buf := make([]byte, 0, 2*n)
    var dfs func(open, closed int)
    dfs = func(open, closed int) {
        if len(buf) == 2*n {
            res = append(res, string(buf))
            return
        }
        if open < n {
            buf = append(buf, '(')
            dfs(open+1, closed)
            buf = buf[:len(buf)-1]
        }
        if closed < open {
            buf = append(buf, ')')
            dfs(open, closed+1)
            buf = buf[:len(buf)-1]
        }
    }
    dfs(0, 0)
    return res
}
```

### 15.6 Constraint problems: N-Queens and Word Search

```go
func solveNQueens(n int) [][]string {
    var res [][]string
    cols := make([]bool, n)
    diag := make([]bool, 2*n) // cells on one diagonal share r+c
    anti := make([]bool, 2*n) // cells on one anti-diagonal share r-c (shifted by n)
    board := make([][]byte, n)
    for i := range board {
        board[i] = []byte(strings.Repeat(".", n))
    }
    var place func(r int)
    place = func(r int) {
        if r == n {
            snap := make([]string, n)
            for i := range board {
                snap[i] = string(board[i])
            }
            res = append(res, snap)
            return
        }
        for c := 0; c < n; c++ {
            if cols[c] || diag[r+c] || anti[r-c+n] {
                continue
            }
            cols[c], diag[r+c], anti[r-c+n] = true, true, true
            board[r][c] = 'Q'
            place(r + 1)
            board[r][c] = '.'
            cols[c], diag[r+c], anti[r-c+n] = false, false, false
        }
    }
    place(0)
    return res
}

// Word Search: mark the cell while it is on the current path, restore it afterwards
func exist(board [][]byte, word string) bool {
    rows, cols := len(board), len(board[0])
    var dfs func(r, c, i int) bool
    dfs = func(r, c, i int) bool {
        if i == len(word) {
            return true
        }
        if r < 0 || r >= rows || c < 0 || c >= cols || board[r][c] != word[i] {
            return false
        }
        tmp := board[r][c]
        board[r][c] = '#' // choose
        found := dfs(r+1, c, i+1) || dfs(r-1, c, i+1) || dfs(r, c+1, i+1) || dfs(r, c-1, i+1)
        board[r][c] = tmp // un-choose
        return found
    }
    for r := range rows {
        for c := range cols {
            if dfs(r, c, 0) {
                return true
            }
        }
    }
    return false
}
```

### 15.7 Costs

| Problem | Time | Extra space |
| --- | --- | --- |
| Subsets | O(n · 2^n) | O(n) recursion |
| Permutations | O(n · n!) | O(n) |
| Combinations C(n, k) | O(k · C(n, k)) | O(k) |
| N-Queens | O(n!) | O(n) |
| Word Search, word length L | O(rows · cols · 3^L) | O(L) |

## 16. Dynamic programming

DP is recursion plus caching. When a problem asks for a count, a minimum / maximum, or a yes / no over a series of choices, and brute-force recursion repeats the same subproblems, define a state, write its transition, and fill a table.

### 16.1 How to recognise DP

The wording gives it away: "number of ways", "minimum cost", "maximum profit", "longest / shortest subsequence", "is it possible". Each step has a choice (take or skip, go right or down), and a greedy choice can be shown wrong by a small counterexample. Typical limits are n ≤ 10^4 for O(n²) tables, or n ≤ 10^5 for O(n) and O(n log n) DP.

### 16.2 The 5-step method

1. State: write in words what `dp[i]` (or `dp[i][j]`) means.
2. Transition: how `dp[i]` is built from smaller states.
3. Base cases: the smallest states you know directly.
4. Order: fill the table so every dependency is computed first.
5. Answer: name the cell that holds the result.

Then shrink memory: if a row depends only on the previous row, keep one or two rows.

### 16.3 Top-down or bottom-up

```go
// Top-down: memoized recursion that follows the recurrence directly
func climbStairsMemo(n int) int {
    memo := make([]int, n+1) // 0 means "not computed" here, since every answer is >= 1
    var f func(int) int
    f = func(i int) int {
        if i <= 1 {
            return 1
        }
        if memo[i] != 0 {
            return memo[i]
        }
        memo[i] = f(i-1) + f(i-2)
        return memo[i]
    }
    return f(n)
}

// Bottom-up: fill the table in dependency order
func climbStairs(n int) int {
    dp := make([]int, max(n+1, 2))
    dp[0], dp[1] = 1, 1
    for i := 2; i <= n; i++ {
        dp[i] = dp[i-1] + dp[i-2]
    }
    return dp[n]
}

// Space-optimised: only the last two values matter
func climbStairsO1(n int) int {
    a, b := 1, 1
    for i := 2; i <= n; i++ {
        a, b = b, a+b
    }
    return b
}
```

For memo keys with several parts, use a 2D slice filled with -1, or a map keyed by an array or struct: `memo := map[[2]int]int{}`.

|  | Top-down (memo) | Bottom-up (table) |
| --- | --- | --- |
| Writing it | mirrors the recurrence | you must work out the fill order |
| States computed | only the reachable ones | all of them |
| Risk | deep recursion | none |
| Space optimisation | hard | easy with rolling rows |

### 16.4 1D DP

```go
// House Robber: dp[i] = max(dp[i-1], dp[i-2] + nums[i]), kept in two variables
func rob(nums []int) int {
    prev2, prev1 := 0, 0
    for _, v := range nums {
        prev2, prev1 = prev1, max(prev1, prev2+v)
    }
    return prev1
}

// Word Break: dp[i] = the prefix s[:i] can be split into dictionary words
func wordBreak(s string, words []string) bool {
    dict := map[string]bool{}
    for _, w := range words {
        dict[w] = true
    }
    dp := make([]bool, len(s)+1)
    dp[0] = true
    for i := 1; i <= len(s); i++ {
        for j := 0; j < i; j++ {
            if dp[j] && dict[s[j:i]] {
                dp[i] = true
                break
            }
        }
    }
    return dp[len(s)]
}

// Longest Increasing Subsequence, O(n²): dp[i] = LIS length ending at i
func lengthOfLIS(nums []int) int {
    dp := make([]int, len(nums))
    best := 0
    for i := range nums {
        dp[i] = 1
        for j := 0; j < i; j++ {
            if nums[j] < nums[i] {
                dp[i] = max(dp[i], dp[j]+1)
            }
        }
        best = max(best, dp[i])
    }
    return best
}

// LIS in O(n log n): tails[k] = smallest tail of any increasing subsequence of length k+1
func lengthOfLISFast(nums []int) int {
    var tails []int
    for _, v := range nums {
        i, _ := slices.BinarySearch(tails, v) // first tail >= v
        if i == len(tails) {
            tails = append(tails, v)
        } else {
            tails[i] = v
        }
    }
    return len(tails)
}
```

### 16.5 The knapsack family

| Variant | Each item | Capacity loop | Examples |
| --- | --- | --- | --- |
| 0/1 knapsack | used at most once | downward | partition equal subset sum, target sum |
| Unbounded knapsack | used any number of times | upward | coin change, perfect squares |
| Count combinations | any number of times | items outside, capacity inside | coin change II |
| Count permutations | any number of times | capacity outside, items inside | combination sum IV |

```go
// 0/1 knapsack: dp[c] = best value using capacity c
func knapsack01(weights, values []int, W int) int {
    dp := make([]int, W+1)
    for i := range weights {
        for c := W; c >= weights[i]; c-- { // DOWNWARD, so item i is counted at most once
            dp[c] = max(dp[c], dp[c-weights[i]]+values[i])
        }
    }
    return dp[W]
}

// Partition Equal Subset Sum: 0/1 knapsack on booleans
func canPartition(nums []int) bool {
    total := 0
    for _, v := range nums {
        total += v
    }
    if total%2 == 1 {
        return false
    }
    target := total / 2
    dp := make([]bool, target+1)
    dp[0] = true
    for _, v := range nums {
        for s := target; s >= v; s-- {
            dp[s] = dp[s] || dp[s-v]
        }
    }
    return dp[target]
}

// Coin Change: fewest coins for amount, unlimited coins, so loop UPWARD
func coinChange(coins []int, amount int) int {
    const inf = math.MaxInt / 2 // halved so inf+1 cannot overflow
    dp := make([]int, amount+1)
    for i := 1; i <= amount; i++ {
        dp[i] = inf
    }
    for _, c := range coins {
        for a := c; a <= amount; a++ {
            dp[a] = min(dp[a], dp[a-c]+1)
        }
    }
    if dp[amount] >= inf {
        return -1
    }
    return dp[amount]
}

// Coin Change II: number of combinations (coins in the outer loop)
func change(amount int, coins []int) int {
    dp := make([]int, amount+1)
    dp[0] = 1
    for _, c := range coins {
        for a := c; a <= amount; a++ {
            dp[a] += dp[a-c]
        }
    }
    return dp[amount]
}
```

### 16.6 2D DP: grids and two strings

```go
// Unique Paths: one reused row; dp[j] += dp[j-1] means "from above + from the left"
func uniquePaths(m, n int) int {
    dp := make([]int, n)
    for j := range dp {
        dp[j] = 1
    }
    for i := 1; i < m; i++ {
        for j := 1; j < n; j++ {
            dp[j] += dp[j-1]
        }
    }
    return dp[n-1]
}

// Longest Common Subsequence: dp[i][j] = LCS of a[:i] and b[:j]
func longestCommonSubsequence(a, b string) int {
    dp := make([][]int, len(a)+1)
    for i := range dp {
        dp[i] = make([]int, len(b)+1)
    }
    for i := 1; i <= len(a); i++ {
        for j := 1; j <= len(b); j++ {
            if a[i-1] == b[j-1] {
                dp[i][j] = dp[i-1][j-1] + 1
            } else {
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
            }
        }
    }
    return dp[len(a)][len(b)]
}

// Edit Distance: fewest inserts, deletes and replaces turning a into b
func minDistance(a, b string) int {
    dp := make([][]int, len(a)+1)
    for i := range dp {
        dp[i] = make([]int, len(b)+1)
        dp[i][0] = i // delete all of a[:i]
    }
    for j := range dp[0] {
        dp[0][j] = j // insert all of b[:j]
    }
    for i := 1; i <= len(a); i++ {
        for j := 1; j <= len(b); j++ {
            if a[i-1] == b[j-1] {
                dp[i][j] = dp[i-1][j-1]
            } else {
                dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) // delete, insert, replace
            }
        }
    }
    return dp[len(a)][len(b)]
}
```

### 16.7 Interval DP and palindromes

```go
// Longest Palindromic Subsequence: dp[i][j] over s[i..j]; i runs downward so dp[i+1] is ready
func longestPalindromeSubseq(s string) int {
    n := len(s)
    dp := make([][]int, n)
    for i := range dp {
        dp[i] = make([]int, n)
        dp[i][i] = 1
    }
    for i := n - 2; i >= 0; i-- {
        for j := i + 1; j < n; j++ {
            if s[i] == s[j] {
                dp[i][j] = dp[i+1][j-1] + 2
            } else {
                dp[i][j] = max(dp[i+1][j], dp[i][j-1])
            }
        }
    }
    return dp[0][n-1]
}

// Longest Palindromic Substring: expand around every centre, O(n²) time and O(1) space
func longestPalindrome(s string) string {
    start, best := 0, 0
    expand := func(l, r int) {
        for l >= 0 && r < len(s) && s[l] == s[r] {
            l--
            r++
        }
        if r-l-1 > best {
            start, best = l+1, r-l-1
        }
    }
    for i := range len(s) {
        expand(i, i)   // odd length
        expand(i, i+1) // even length
    }
    return s[start : start+best]
}

// Burst Balloons: dp[i][j] = best coins bursting everything strictly between i and j
func maxCoins(nums []int) int {
    a := append(append([]int{1}, nums...), 1) // pad with 1s at both ends
    n := len(a)
    dp := make([][]int, n)
    for i := range dp {
        dp[i] = make([]int, n)
    }
    for length := 2; length < n; length++ {
        for i := 0; i+length < n; i++ {
            j := i + length
            for k := i + 1; k < j; k++ { // k is the LAST balloon burst between i and j
                dp[i][j] = max(dp[i][j], dp[i][k]+dp[k][j]+a[i]*a[k]*a[j])
            }
        }
    }
    return dp[0][n-1]
}
```

### 16.8 State-machine DP

When each step moves between a few states, keep one variable per state and update them together.

```go
// Best Time to Buy and Sell Stock with Cooldown
func maxProfit(prices []int) int {
    hold, sold, rest := math.MinInt/2, 0, 0
    for _, p := range prices {
        // the right side uses the old values, so all three update at once
        hold, sold, rest = max(hold, rest-p), hold+p, max(rest, sold)
    }
    return max(sold, rest)
}
```

### 16.9 Bitmask DP and tree DP

```go
// Travelling salesman (n <= 20): dp[mask][i] = cheapest route visiting set mask, ending at i
func tsp(dist [][]int) int {
    n := len(dist)
    const inf = math.MaxInt / 2
    dp := make([][]int, 1<<n)
    for m := range dp {
        dp[m] = make([]int, n)
        for i := range dp[m] {
            dp[m][i] = inf
        }
    }
    dp[1][0] = 0 // start at city 0
    for mask := 1; mask < 1<<n; mask++ {
        for i := 0; i < n; i++ {
            if dp[mask][i] == inf || mask&(1<<i) == 0 {
                continue
            }
            for j := 0; j < n; j++ {
                if mask&(1<<j) != 0 {
                    continue // j already visited
                }
                next := mask | 1<<j
                dp[next][j] = min(dp[next][j], dp[mask][i]+dist[i][j])
            }
        }
    }
    best := inf
    for i := 1; i < n; i++ {
        best = min(best, dp[1<<n-1][i]+dist[i][0])
    }
    return best
}

// House Robber III: each node returns (best if taken, best if skipped)
func robTree(root *TreeNode) int {
    var dfs func(*TreeNode) (int, int)
    dfs = func(n *TreeNode) (int, int) {
        if n == nil {
            return 0, 0
        }
        lt, ls := dfs(n.Left)
        rt, rs := dfs(n.Right)
        return n.Val + ls + rs, max(lt, ls) + max(rt, rs)
    }
    take, skip := dfs(root)
    return max(take, skip)
}
```

### 16.10 DP families at a glance

| Family | State | Classic problems |
| --- | --- | --- |
| Linear | `dp[i]` over a prefix | climbing stairs, house robber, decode ways, word break |
| Ending-at-i | `dp[i]` for sequences ending at i | LIS, number of LIS, Russian doll envelopes |
| 0/1 knapsack | `dp[capacity]`, downward loop | partition equal subset sum, target sum |
| Unbounded knapsack | `dp[capacity]`, upward loop | coin change, coin change II, perfect squares |
| Grid | `dp[r][c]` | unique paths, minimum path sum, maximal square |
| Two sequences | `dp[i][j]` over two prefixes | LCS, edit distance, distinct subsequences, regex matching |
| Interval | `dp[i][j]` over `s[i..j]` | palindromes, burst balloons, matrix chain |
| State machine | a few variables per step | the stock series |
| Bitmask | `dp[mask][...]` | TSP, assignments, partition into k equal subsets |
| Tree | values returned by children | house robber III, binary tree maximum path sum |

## 17. Greedy algorithms

A greedy algorithm makes the locally best choice at each step and never revisits it. It is correct only when you can argue that the choice never hurts, which usually means sorting by the right key first.

### 17.1 When greedy works

The signals are intervals and scheduling, "minimum number of X to cover Y", "can you reach the end", and merging cheapest-first. Prove it with an exchange argument: swapping an optimal solution's choice for the greedy choice never makes it worse.

Always test a small counterexample before trusting greedy. With coins {1, 3, 4} and amount 6, greedy takes 4 + 1 + 1 (3 coins) but 3 + 3 (2 coins) is optimal, so that problem needs DP.

| Problem | Sort by | Greedy choice |
| --- | --- | --- |
| Non-overlapping intervals, activity selection | end time | keep the interval that finishes first |
| Minimum arrows to burst balloons | end | shoot at the current group's end |
| Meeting rooms II | start | reuse the room that frees up earliest (min-heap of ends) |
| Fractional knapsack | value / weight, descending | take the best ratio first |
| Assign cookies | both ascending | the smallest cookie that satisfies each child |
| Huffman coding, connect sticks | min-heap | merge the two smallest |
| Jump game | none | track the farthest reachable index |

### 17.2 Interval greedy

```go
// Non-overlapping Intervals: remove the fewest so the rest never overlap.
// Keeping the interval that ENDS earliest leaves the most room for the others.
func eraseOverlapIntervals(iv [][]int) int {
    slices.SortFunc(iv, func(a, b []int) int { return cmp.Compare(a[1], b[1]) })
    removed, end := 0, math.MinInt
    for _, x := range iv {
        if x[0] >= end {
            end = x[1] // keep it
        } else {
            removed++ // it overlaps the last kept interval
        }
    }
    return removed
}

// Minimum Number of Arrows to Burst Balloons
func findMinArrowShots(points [][]int) int {
    slices.SortFunc(points, func(a, b []int) int { return cmp.Compare(a[1], b[1]) })
    arrows, end := 0, 0
    for _, p := range points {
        if arrows == 0 || p[0] > end {
            arrows++
            end = p[1]
        }
    }
    return arrows
}

// Meeting Rooms II: the heap holds the end times of rooms in use (IntHeap from Section 9)
func minMeetingRooms(iv [][]int) int {
    slices.SortFunc(iv, func(a, b []int) int { return cmp.Compare(a[0], b[0]) })
    h := &IntHeap{}
    for _, x := range iv {
        if h.Len() > 0 && (*h)[0] <= x[0] {
            heap.Pop(h) // a room freed up before this meeting starts
        }
        heap.Push(h, x[1])
    }
    return h.Len()
}

// Partition Labels: stretch each part to the last occurrence of every letter in it
func partitionLabels(s string) []int {
    var last [26]int
    for i := 0; i < len(s); i++ {
        last[s[i]-'a'] = i
    }
    var res []int
    start, end := 0, 0
    for i := 0; i < len(s); i++ {
        end = max(end, last[s[i]-'a'])
        if i == end {
            res = append(res, end-start+1)
            start = i + 1
        }
    }
    return res
}
```

### 17.3 Reach, balance and counting greedy

```go
// Jump Game: track the farthest index reachable so far
func canJump(nums []int) bool {
    far := 0
    for i, v := range nums {
        if i > far {
            return false
        }
        far = max(far, i+v)
    }
    return true
}

// Jump Game II: fewest jumps; each jump covers a window of indices, like BFS levels
func jump(nums []int) int {
    jumps, curEnd, far := 0, 0, 0
    for i := 0; i < len(nums)-1; i++ {
        far = max(far, i+nums[i])
        if i == curEnd {
            jumps++
            curEnd = far
        }
    }
    return jumps
}

// Gas Station: an answer exists if total gas >= total cost; restart after any empty tank
func canCompleteCircuit(gas, cost []int) int {
    total, tank, start := 0, 0, 0
    for i := range gas {
        d := gas[i] - cost[i]
        total += d
        tank += d
        if tank < 0 {
            start, tank = i+1, 0
        }
    }
    if total < 0 {
        return -1
    }
    return start
}

// Assign Cookies: sort both, hand out the smallest cookie that satisfies
func findContentChildren(greed, cookies []int) int {
    slices.Sort(greed)
    slices.Sort(cookies)
    child := 0
    for _, c := range cookies {
        if child < len(greed) && c >= greed[child] {
            child++
        }
    }
    return child
}

// Task Scheduler: the most frequent task fixes the frame size
func leastInterval(tasks []byte, n int) int {
    var freq [26]int
    for _, t := range tasks {
        freq[t-'A']++
    }
    maxF, countMax := 0, 0
    for _, f := range freq {
        if f > maxF {
            maxF, countMax = f, 1
        } else if f == maxF {
            countMax++
        }
    }
    return max(len(tasks), (maxF-1)*(n+1)+countMax)
}

// Connect Sticks / Huffman cost: always merge the two smallest
func connectSticks(sticks []int) int {
    h := IntHeap(slices.Clone(sticks))
    heap.Init(&h)
    cost := 0
    for h.Len() > 1 {
        a, b := heap.Pop(&h).(int), heap.Pop(&h).(int)
        cost += a + b
        heap.Push(&h, a+b)
    }
    return cost
}
```

## 18. Graph algorithms

These algorithms sit on top of the representations and BFS/DFS from Section 12. Pick one by answering three questions: is the graph directed, is it weighted, and can a weight be negative?

### 18.1 Choosing the algorithm

| Problem | Algorithm | Time | Watch out for |
| --- | --- | --- | --- |
| Order tasks with dependencies | Topological sort (Kahn or DFS) | O(V+E) | Only works on a DAG; leftover nodes mean a cycle |
| Shortest path, unweighted | BFS (12.3) | O(V+E) | Count edges, not weights |
| Shortest path, weights 0 or 1 | 0-1 BFS | O(V+E) | Needs a deque |
| Shortest path, non-negative weights | Dijkstra | O((V+E) log V) | Wrong answers with negative edges |
| Shortest path, negative weights or at most k edges | Bellman-Ford | O(V·E) | Also detects negative cycles |
| All-pairs shortest paths | Floyd-Warshall | O(V³) | Only for V up to about 400 |
| Minimum spanning tree from an edge list | Kruskal | O(E log E) | Needs DSU (11.2) |
| Minimum spanning tree, dense graph | Prim | O(E log V) or O(V²) | Heap or plain arrays |
| Critical edges or cut vertices | Tarjan bridges | O(V+E) | Undirected graphs |
| Groups that can reach each other | Kosaraju SCC | O(V+E) | Directed graphs |

All examples use `adj [][]int` for unweighted graphs and `adj [][]Edge` with `type Edge struct{ to, w int }` for weighted ones, exactly as built in 12.2.

### 18.2 Topological sort

A topological order lists every node before the nodes it points to. The signals are prerequisites, build order, course schedule, "X must happen before Y", or ordering letters of an alien alphabet.

Kahn's algorithm is BFS on in-degrees: start with nodes that have no incoming edges, remove them, and push any neighbor whose in-degree drops to zero. If the output is shorter than n, a cycle blocked the rest.

```go
// Kahn's algorithm: repeatedly remove nodes with no incoming edges
func topoSort(n int, edges [][]int) ([]int, bool) {
    adj := make([][]int, n)
    indeg := make([]int, n)
    for _, e := range edges { // e = [from, to]
        adj[e[0]] = append(adj[e[0]], e[1])
        indeg[e[1]]++
    }
    queue := []int{}
    for v := range n {
        if indeg[v] == 0 {
            queue = append(queue, v)
        }
    }
    order := make([]int, 0, n)
    for len(queue) > 0 {
        u := queue[0]
        queue = queue[1:]
        order = append(order, u)
        for _, v := range adj[u] {
            indeg[v]--
            if indeg[v] == 0 {
                queue = append(queue, v)
            }
        }
    }
    return order, len(order) == n // false means a cycle blocked some nodes
}

// Course Schedule II: prerequisites[i] = [course, pre], so the edge is pre -> course
func findOrder(numCourses int, prerequisites [][]int) []int {
    edges := make([][]int, len(prerequisites))
    for i, p := range prerequisites {
        edges[i] = []int{p[1], p[0]}
    }
    order, ok := topoSort(numCourses, edges)
    if !ok {
        return []int{}
    }
    return order
}
```

The DFS version appends a node after all its descendants finish, then reverses. It reuses the three-color cycle check from 12.6.

```go
// DFS topological sort: reverse of the finish order
func topoDFS(n int, adj [][]int) ([]int, bool) {
    color := make([]int, n) // 0 = unvisited, 1 = on the current path, 2 = done
    order := make([]int, 0, n)
    var dfs func(u int) bool
    dfs = func(u int) bool {
        color[u] = 1
        for _, v := range adj[u] {
            if color[v] == 1 {
                return false // back edge: cycle
            }
            if color[v] == 0 && !dfs(v) {
                return false
            }
        }
        color[u] = 2
        order = append(order, u)
        return true
    }
    for v := range n {
        if color[v] == 0 && !dfs(v) {
            return nil, false
        }
    }
    slices.Reverse(order)
    return order, true
}
```

Two useful variations: replace Kahn's queue with a min-heap to get the lexicographically smallest order, and count BFS layers of Kahn's algorithm to get the minimum number of semesters (Parallel Courses).

### 18.3 Dijkstra

Dijkstra finds shortest paths from one source when every weight is zero or positive. It always expands the closest unfinished node, which is why a min-heap drives it. Go has no built-in priority queue, so this uses `container/heap` from Section 9 with lazy deletion: push a new entry on every improvement and skip stale ones when they pop.

```go
type Item struct{ node, dist int }
type MinDist []Item

func (h MinDist) Len() int           { return len(h) }
func (h MinDist) Less(i, j int) bool { return h[i].dist < h[j].dist }
func (h MinDist) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *MinDist) Push(x any)        { *h = append(*h, x.(Item)) }
func (h *MinDist) Pop() any {
    old := *h
    it := old[len(old)-1]
    *h = old[:len(old)-1]
    return it
}

const INF = math.MaxInt / 2 // halved so INF + w never overflows

func dijkstra(adj [][]Edge, src int) []int {
    dist := make([]int, len(adj))
    for i := range dist {
        dist[i] = INF
    }
    dist[src] = 0
    h := &MinDist{{src, 0}}
    for h.Len() > 0 {
        cur := heap.Pop(h).(Item)
        if cur.dist > dist[cur.node] {
            continue // stale entry: a shorter path was already found
        }
        for _, e := range adj[cur.node] {
            if nd := cur.dist + e.w; nd < dist[e.to] {
                dist[e.to] = nd
                heap.Push(h, Item{e.to, nd})
            }
        }
    }
    return dist
}

// Network Delay Time: times[i] = [u, v, w], nodes are 1..n
func networkDelayTime(times [][]int, n, k int) int {
    adj := make([][]Edge, n+1)
    for _, t := range times {
        adj[t[0]] = append(adj[t[0]], Edge{t[1], t[2]})
    }
    dist := dijkstra(adj, k)
    ans := slices.Max(dist[1:])
    if ans == INF {
        return -1
    }
    return ans
}
```

To rebuild the path, set `parent[e.to] = cur.node` whenever you relax an edge and walk back as in `pathTo` (12.3). For "minimize the largest step" problems such as Path With Minimum Effort, keep the same loop and change the new distance to `max(cur.dist, e.w)`. On a grid, the node is a cell and the neighbors come from the four directions in 12.5.

### 18.4 0-1 BFS

When every weight is 0 or 1, a deque replaces the heap: zero-weight edges go to the front, one-weight edges to the back. The typical signal is "minimum number of obstacles to remove" or "minimum cells to flip", where moving is free and breaking costs 1.

```go
// 0-1 BFS with container/list as the deque
func zeroOneBFS(adj [][]Edge, src int) []int {
    dist := make([]int, len(adj))
    for i := range dist {
        dist[i] = INF
    }
    dist[src] = 0
    dq := list.New()
    dq.PushBack(src)
    for dq.Len() > 0 {
        u := dq.Remove(dq.Front()).(int)
        for _, e := range adj[u] {
            if nd := dist[u] + e.w; nd < dist[e.to] {
                dist[e.to] = nd
                if e.w == 0 {
                    dq.PushFront(e.to)
                } else {
                    dq.PushBack(e.to)
                }
            }
        }
    }
    return dist
}
```

### 18.5 Bellman-Ford

Bellman-Ford relaxes every edge V−1 times, because a shortest path has at most V−1 edges. It is slower than Dijkstra but handles negative weights, and if a V-th pass still improves something, a negative cycle is reachable.

```go
// edges[i] = [u, v, w]; ok is false when a negative cycle is reachable from src
func bellmanFord(n int, edges [][]int, src int) (dist []int, ok bool) {
    dist = make([]int, n)
    for i := range dist {
        dist[i] = INF
    }
    dist[src] = 0
    for range n - 1 {
        changed := false
        for _, e := range edges {
            u, v, w := e[0], e[1], e[2]
            if dist[u] != INF && dist[u]+w < dist[v] {
                dist[v] = dist[u] + w
                changed = true
            }
        }
        if !changed {
            break // nothing moved, so nothing will
        }
    }
    for _, e := range edges {
        if dist[e[0]] != INF && dist[e[0]]+e[2] < dist[e[1]] {
            return dist, false
        }
    }
    return dist, true
}

// Cheapest Flights Within K Stops: at most k+1 edges, so exactly k+1 rounds
func findCheapestPrice(n int, flights [][]int, src, dst, k int) int {
    dist := make([]int, n)
    for i := range dist {
        dist[i] = INF
    }
    dist[src] = 0
    for range k + 1 {
        next := slices.Clone(dist) // read old, write new: each round adds one edge only
        for _, f := range flights {
            u, v, w := f[0], f[1], f[2]
            if dist[u] != INF && dist[u]+w < next[v] {
                next[v] = dist[u] + w
            }
        }
        dist = next
    }
    if dist[dst] == INF {
        return -1
    }
    return dist[dst]
}
```

The `slices.Clone` line is the whole trick for "at most k edges": without the copy, one round could chain several edges together.

### 18.6 Floyd-Warshall

Floyd-Warshall computes every pair's shortest distance by allowing one more intermediate node per outer iteration. The loop order matters: the intermediate node `k` must be the outermost loop.

```go
// edges[i] = [u, v, w] for a directed graph; add both directions if undirected
func floydWarshall(n int, edges [][]int) [][]int {
    d := make([][]int, n)
    for i := range d {
        d[i] = make([]int, n)
        for j := range d[i] {
            if i != j {
                d[i][j] = INF
            }
        }
    }
    for _, e := range edges {
        d[e[0]][e[1]] = min(d[e[0]][e[1]], e[2]) // keep the cheapest parallel edge
    }
    for k := range n {
        for i := range n {
            if d[i][k] == INF {
                continue
            }
            for j := range n {
                d[i][j] = min(d[i][j], d[i][k]+d[k][j])
            }
        }
    }
    return d // d[i][i] < 0 means i lies on a negative cycle
}
```

Use it when n is small and you need many pairs, as in Find the City With the Smallest Number of Neighbors. The same triple loop with `||` and `&&` instead of `min` and `+` computes reachability (transitive closure).

### 18.7 Minimum spanning tree

A minimum spanning tree connects all V nodes with V−1 edges at the lowest total weight. The signals are "connect all cities, points or houses at minimum cost".

Kruskal sorts edges by weight and keeps each one that joins two different components, using the DSU from 11.2.

```go
// edges[i] = [u, v, w]; ok is false when the graph is disconnected
func kruskal(n int, edges [][]int) (total int, ok bool) {
    slices.SortFunc(edges, func(a, b []int) int { return cmp.Compare(a[2], b[2]) })
    d := NewDSU(n)
    used := 0
    for _, e := range edges {
        if d.Union(e[0], e[1]) { // false means u and v are already connected
            total += e[2]
            used++
            if used == n-1 {
                break
            }
        }
    }
    return total, used == n-1
}
```

Prim grows one tree from node 0 and always adds the cheapest edge leaving it. With a heap it reuses `MinDist` from 18.3, where `dist` now means the weight of the edge into the tree.

```go
func prim(adj [][]Edge) (total int, ok bool) {
    n := len(adj)
    inTree := make([]bool, n)
    h := &MinDist{{0, 0}}
    count := 0
    for h.Len() > 0 && count < n {
        cur := heap.Pop(h).(Item)
        if inTree[cur.node] {
            continue
        }
        inTree[cur.node] = true
        total += cur.dist
        count++
        for _, e := range adj[cur.node] {
            if !inTree[e.to] {
                heap.Push(h, Item{e.to, e.w})
            }
        }
    }
    return total, count == n
}
```

On a complete graph (every pair connected, E ≈ V²) skip the heap. The O(V²) array version is both simpler and faster.

```go
// Min Cost to Connect All Points: Manhattan distance between every pair
func minCostConnectPoints(points [][]int) int {
    n := len(points)
    best := make([]int, n) // cheapest known edge from the tree to each node
    for i := range best {
        best[i] = INF
    }
    best[0] = 0
    inTree := make([]bool, n)
    total := 0
    for range n {
        u := -1
        for v := range n {
            if !inTree[v] && (u == -1 || best[v] < best[u]) {
                u = v
            }
        }
        inTree[u] = true
        total += best[u]
        for v := range n {
            if !inTree[v] {
                dist := abs(points[u][0]-points[v][0]) + abs(points[u][1]-points[v][1])
                best[v] = min(best[v], dist)
            }
        }
    }
    return total
}

// Go has no integer abs; math.Abs works on float64 only
func abs(x int) int {
    if x < 0 {
        return -x
    }
    return x
}
```

### 18.8 Bridges and strongly connected components

A bridge is an edge whose removal disconnects the graph. Tarjan's algorithm records each node's discovery time `disc` and the earliest discovery time its subtree can reach, `low`. Edge u→v is a bridge when `low[v] > disc[u]`, meaning nothing below v climbs back above u.

```go
// Critical Connections in a Network
func criticalConnections(n int, connections [][]int) [][]int {
    adj := make([][]int, n)
    for _, c := range connections {
        adj[c[0]] = append(adj[c[0]], c[1])
        adj[c[1]] = append(adj[c[1]], c[0])
    }
    disc := make([]int, n) // 0 = unvisited
    low := make([]int, n)
    timer := 0
    var res [][]int
    var dfs func(u, parent int)
    dfs = func(u, parent int) {
        timer++
        disc[u], low[u] = timer, timer
        for _, v := range adj[u] {
            if v == parent {
                continue // do not climb back along the edge we came in on
            }
            if disc[v] == 0 {
                dfs(v, u)
                low[u] = min(low[u], low[v])
                if low[v] > disc[u] {
                    res = append(res, []int{u, v})
                }
            } else {
                low[u] = min(low[u], disc[v]) // back edge
            }
        }
    }
    for v := range n {
        if disc[v] == 0 {
            dfs(v, -1)
        }
    }
    return res
}
```

For cut vertices (articulation points), change the test to `low[v] >= disc[u]` for non-root nodes and treat the root as a cut vertex only if it has two or more DFS children. If the input can contain parallel edges, skip the parent by edge index rather than by node.

A strongly connected component is a maximal set of nodes in a directed graph where each can reach every other. Kosaraju runs DFS twice: once to record finish order, then on the reversed graph in reverse finish order. Each second-pass tree is one component.

```go
func scc(n int, adj [][]int) [][]int {
    radj := make([][]int, n)
    for u := range n {
        for _, v := range adj[u] {
            radj[v] = append(radj[v], u)
        }
    }
    seen := make([]bool, n)
    order := make([]int, 0, n)
    var dfs1 func(u int)
    dfs1 = func(u int) {
        seen[u] = true
        for _, v := range adj[u] {
            if !seen[v] {
                dfs1(v)
            }
        }
        order = append(order, u) // finished
    }
    for v := range n {
        if !seen[v] {
            dfs1(v)
        }
    }
    clear(seen)
    var comps [][]int
    var comp []int
    var dfs2 func(u int)
    dfs2 = func(u int) {
        seen[u] = true
        comp = append(comp, u)
        for _, v := range radj[u] {
            if !seen[v] {
                dfs2(v)
            }
        }
    }
    for i := n - 1; i >= 0; i-- {
        if u := order[i]; !seen[u] {
            comp = nil
            dfs2(u)
            comps = append(comps, comp)
        }
    }
    return comps
}
```

### 18.9 Complexity summary

| Algorithm | Time | Space | Negative weights | Go building blocks |
| --- | --- | --- | --- | --- |
| Kahn topological sort | O(V+E) | O(V) | n/a | slice queue + in-degree slice |
| DFS topological sort | O(V+E) | O(V) | n/a | recursive closure + colors |
| BFS | O(V+E) | O(V) | unweighted only | slice queue |
| 0-1 BFS | O(V+E) | O(V) | weights 0 and 1 only | `container/list` deque |
| Dijkstra | O((V+E) log V) | O(V+E) | No | `container/heap` |
| Bellman-Ford | O(V·E) | O(V) | Yes, detects cycles | edge list + `slices.Clone` |
| Floyd-Warshall | O(V³) | O(V²) | Yes | 2D slice |
| Kruskal | O(E log E) | O(V) | Yes | `slices.SortFunc` + DSU |
| Prim with heap | O(E log V) | O(V+E) | Yes | `container/heap` |
| Prim with arrays | O(V²) | O(V) | Yes | two slices |
| Tarjan bridges | O(V+E) | O(V) | n/a | `disc` and `low` slices |
| Kosaraju SCC | O(V+E) | O(V+E) | n/a | two DFS passes |

## 19. Bit manipulation and math

Bits give O(1) tricks and let one integer stand in for a whole set. Number theory shows up whenever a problem says "return the answer modulo 10⁹+7" or mentions primes, divisors or combinations.

### 19.1 Bitwise operators

Go has six bitwise operators. Two differ from C: bitwise NOT is unary `^x` (there is no `~`), and `&^` is a dedicated "AND NOT" that clears bits.

```go
a, b := 0b1100, 0b1010 // binary literals; also 0o17 (octal), 0xFF (hex), 1_000_000

a & b   // 0b1000  AND: 1 where both are 1
a | b   // 0b1110  OR: 1 where either is 1
a ^ b   // 0b0110  XOR: 1 where they differ
^a      // -13     NOT (unary ^): flips every bit, so ^x == -x-1
a &^ b  // 0b0100  AND NOT: clear the bits of a that are set in b
1 << 3  // 8       shift left: multiply by 2^k
-8 >> 1 // -4      shift right: signed values keep their sign (floor division by 2^k)

// Printing and parsing binary
fmt.Printf("%b %08b\n", 5, 5)        // 101 00000101
s := strconv.FormatInt(10, 2)          // "1010"
v, _ := strconv.ParseInt("1011", 2, 64) // 11
```

Precedence is friendlier than in C. In Go, `<<`, `>>`, `&` and `&^` bind like `*`, while `|` and `^` bind like `+`, and all of them bind tighter than `==`. So `x>>i&1 == 1` means `((x>>i)&1) == 1` and `1<<n - 1` means `(1<<n) - 1`. A negative shift count panics at run time.

### 19.2 Bit tricks

```go
x>>i&1 == 1        // is bit i set?
x |= 1 << i        // set bit i
x &^= 1 << i       // clear bit i
x ^= 1 << i        // toggle bit i
x & -x             // isolate the lowest set bit (12 -> 4)
x & (x - 1)        // drop the lowest set bit (12 -> 8)
x > 0 && x&(x-1) == 0 // x is a power of two
1<<n - 1           // mask of the lowest n bits
x&1 == 1           // x is odd (also works for negatives, unlike x%2 == 1)
(a ^ b) < 0        // a and b have opposite signs
x ^ x == 0; x ^ 0 == x // XOR cancels pairs; the basis of every "single number" trick
```

### 19.3 The math/bits package

`math/bits` compiles to single CPU instructions. Every function takes unsigned values, so convert first: `bits.OnesCount(uint(x))`. Each has sized variants such as `OnesCount8`, `OnesCount32` and `OnesCount64`.

| Function | Returns | Example |
| --- | --- | --- |
| `bits.OnesCount(x)` | Number of 1 bits (popcount) | `OnesCount(0b1011) == 3` |
| `bits.TrailingZeros(x)` | Index of the lowest set bit; bit width if x is 0 | `TrailingZeros(0b1000) == 3` |
| `bits.LeadingZeros(x)` | Zeros above the highest set bit | `LeadingZeros64(1) == 63` |
| `bits.Len(x)` | Bits needed to write x, i.e. floor(log₂x)+1; 0 for 0 | `Len(8) == 4` |
| `bits.Reverse32(x)` | Bits in reverse order | Reverse Bits problem |
| `bits.RotateLeft(x, k)` | Rotate left by k; negative k rotates right | Circular shifts |
| `bits.UintSize` | 32 or 64 (64 on every judge you will use) | Constant |

```go
// Hamming Distance: count the bits where x and y differ
func hammingDistance(x, y int) int {
    return bits.OnesCount(uint(x ^ y))
}

// Counting Bits: i has the bits of i/2 plus its own last bit
func countBits(n int) []int {
    dp := make([]int, n+1)
    for i := 1; i <= n; i++ {
        dp[i] = dp[i>>1] + i&1
    }
    return dp
}

// Reverse Bits of a 32-bit unsigned integer
func reverseBits(num uint32) uint32 {
    return bits.Reverse32(num)
}
```

### 19.4 XOR problems

The signal is "every element appears twice (or k times) except one" together with O(1) extra space.

```go
// Single Number: pairs cancel, the loner survives
func singleNumber(nums []int) int {
    x := 0
    for _, v := range nums {
        x ^= v
    }
    return x
}

// Missing Number: XOR all indices 0..n with all values; the missing one is left
func missingNumber(nums []int) int {
    x := len(nums)
    for i, v := range nums {
        x ^= i ^ v
    }
    return x
}

// Single Number III: two loners; split the array by any bit where they differ
func singleNumberIII(nums []int) []int {
    x := 0
    for _, v := range nums {
        x ^= v // x = a ^ b
    }
    low := x & -x // a and b differ at this bit
    a := 0
    for _, v := range nums {
        if v&low != 0 {
            a ^= v
        }
    }
    return []int{a, x ^ a}
}

// Single Number II: others appear three times; count each bit modulo 3
func singleNumberII(nums []int) int {
    var res int32 // 32-bit so the sign bit lands in the right place
    for i := range 32 {
        count := 0
        for _, v := range nums {
            count += int(int32(v) >> i & 1)
        }
        if count%3 != 0 {
            res |= 1 << i
        }
    }
    return int(res)
}
```

### 19.5 Bitmasks as sets

With n up to about 20, an int can hold a subset: bit i says whether item i is in. This powers subset enumeration and the bitmask DP in 16 (TSP).

```go
// All 2^n subsets
func subsetsMask(nums []int) [][]int {
    n := len(nums)
    res := make([][]int, 0, 1<<n)
    for mask := range 1 << n {
        var sub []int
        for i := range n {
            if mask>>i&1 == 1 {
                sub = append(sub, nums[i])
            }
        }
        res = append(res, sub)
    }
    return res
}

// Visit only the set bits of a mask
for m := mask; m > 0; m &= m - 1 {
    i := bits.TrailingZeros(uint(m))
    _ = i // bit i is set
}

// Visit every non-empty submask of mask; O(3^n) summed over all masks
for sub := mask; sub > 0; sub = (sub - 1) & mask {
    _ = sub
}

// Letters as a 26-bit mask: two words share no letter when their masks AND to 0
func letterMask(s string) int {
    m := 0
    for i := 0; i < len(s); i++ {
        m |= 1 << (s[i] - 'a')
    }
    return m
}
```

### 19.6 GCD, LCM, primes and divisors

```go
// Euclid: gcd(a, b) = gcd(b, a mod b); pass non-negative values
func gcd(a, b int) int {
    for b != 0 {
        a, b = b, a%b
    }
    return a
}

func lcm(a, b int) int {
    return a / gcd(a, b) * b // divide first to delay overflow
}

// Trial division: O(√n)
func isPrime(n int) bool {
    if n < 2 {
        return false
    }
    for i := 2; i*i <= n; i++ {
        if n%i == 0 {
            return false
        }
    }
    return true
}

// Sieve of Eratosthenes: prime[i] for every i <= n in O(n log log n)
func sieve(n int) []bool {
    prime := make([]bool, n+1)
    for i := 2; i <= n; i++ {
        prime[i] = true
    }
    for i := 2; i*i <= n; i++ {
        if prime[i] {
            for j := i * i; j <= n; j += i { // smaller multiples were already crossed out
                prime[j] = false
            }
        }
    }
    return prime
}

// Prime factors with repetition: 12 -> [2 2 3]
func factorize(n int) []int {
    var fs []int
    for p := 2; p*p <= n; p++ {
        for n%p == 0 {
            fs = append(fs, p)
            n /= p
        }
    }
    if n > 1 {
        fs = append(fs, n) // whatever is left is itself prime
    }
    return fs
}

// All divisors in ascending order: O(√n)
func divisors(n int) []int {
    var small, large []int
    for i := 1; i*i <= n; i++ {
        if n%i == 0 {
            small = append(small, i)
            if i != n/i {
                large = append(large, n/i)
            }
        }
    }
    slices.Reverse(large)
    return append(small, large...)
}
```

Count Primes asks for primes strictly less than n, so call `sieve(n - 1)` and count the `true` entries.

### 19.7 Modular arithmetic

Reduce after every operation so values stay below the modulus. Two values below 10⁹+7 multiply to about 10¹⁸, which fits in int64 (max 9.2×10¹⁸), but a third factor would overflow, so take `% MOD` after every single multiply.

```go
const MOD = 1_000_000_007

func addMod(a, b int) int { return (a + b) % MOD }
func mulMod(a, b int) int { return a * b % MOD }
func subMod(a, b int) int { return ((a-b)%MOD + MOD) % MOD } // Go's % can be negative

// Fast exponentiation by squaring: O(log exp)
func powMod(base, exp, mod int) int {
    base %= mod
    res := 1
    for exp > 0 {
        if exp&1 == 1 {
            res = res * base % mod
        }
        base = base * base % mod
        exp >>= 1
    }
    return res
}

// Modular inverse by Fermat's little theorem; the modulus must be prime
func invMod(a int) int { return powMod(a, MOD-2, MOD) }

// nCr mod p: O(n) precomputation, O(1) per query
type Comb struct{ fact, inv []int }

func NewComb(n int) *Comb {
    fact := make([]int, n+1)
    inv := make([]int, n+1)
    fact[0] = 1
    for i := 1; i <= n; i++ {
        fact[i] = fact[i-1] * i % MOD
    }
    inv[n] = invMod(fact[n])
    for i := n; i > 0; i-- {
        inv[i-1] = inv[i] * i % MOD
    }
    return &Comb{fact, inv}
}

func (c *Comb) NCr(n, r int) int {
    if r < 0 || r > n {
        return 0
    }
    return c.fact[n] * c.inv[r] % MOD * c.inv[n-r] % MOD
}

// Small nCr without a modulus: each step is an exact integer C(n-r+i, i)
func nCr(n, r int) int {
    r = min(r, n-r)
    res := 1
    for i := 1; i <= r; i++ {
        res = res * (n - r + i) / i
    }
    return res
}
```

For numbers beyond 64 bits, `math/big` provides `big.Int` with `Add`, `Mul`, `Exp`, `GCD` and `ProbablyPrime`. It is slow but exact.

### 19.8 Integer behavior in Go

| Topic | Go behavior | What to do |
| --- | --- | --- |
| Size of `int` | 64 bits on every judge, max about 9.2×10¹⁸ | Use `int`; use `int32` only when the problem demands 32-bit behavior |
| Overflow | Wraps silently, no panic | Check before multiplying: `a > math.MaxInt/b` |
| Division | Truncates toward zero: `-7/2 == -3` | Use `floorDiv` below for floors |
| Remainder | Takes the sign of the dividend: `-7%3 == -1` | `((a%m)+m)%m` for a non-negative result |
| Absolute value | `math.Abs` takes and returns `float64` | Write `abs(int)` (18.7) |
| Limits | `math.MaxInt`, `math.MinInt`, `math.MaxInt32`, `math.MinInt64`, … | Use as ±infinity; `math.MaxInt/2` if you will add to it |
| Powers | No `**` operator; `math.Pow` returns `float64` | Integer loop or `powMod` |
| Square root | `int(math.Sqrt(float64(x)))` can be off by one for large x | Correct it as in `isqrt` below |
| Ceiling division | No built-in | `(a + b - 1) / b` for positive a and b |
| Floats | Rounding error makes `==` unreliable | Compare with a small epsilon, or stay in integers |

```go
// Reverse Integer: int is 64-bit, so check the 32-bit range after each step
func reverse(x int) int {
    res := 0
    for x != 0 {
        res = res*10 + x%10 // x%10 keeps the sign, so negatives just work
        x /= 10
        if res > math.MaxInt32 || res < math.MinInt32 {
            return 0
        }
    }
    return res
}

// Floor division for operands of any sign
func floorDiv(a, b int) int {
    q := a / b
    if a%b != 0 && (a < 0) != (b < 0) {
        q--
    }
    return q
}

// Exact integer square root
func isqrt(x int) int {
    r := int(math.Sqrt(float64(x)))
    for r*r > x {
        r--
    }
    for (r+1)*(r+1) <= x {
        r++
    }
    return r
}

// Digit loop: sum of digits
func digitSum(x int) int {
    s := 0
    for ; x > 0; x /= 10 {
        s += x % 10
    }
    return s
}
```

### 19.9 Formulas worth memorizing

| Formula | Where it appears |
| --- | --- |
| 1 + 2 + … + n = n(n+1)/2 | Missing number, counting subarrays (an array of n has n(n+1)/2) |
| 2ⁿ subsets, n! permutations, C(n, r) combinations | Sizing backtracking output (Section 15) |
| Catalan number C(2n, n)/(n+1) | Unique BSTs, counting valid parentheses |
| Pigeonhole: n+1 items in n boxes forces a repeat | Find the Duplicate Number |
| Manhattan distance abs(x1−x2) + abs(y1−y2) | Grid moves, Min Cost to Connect All Points |
| Compare squared distances x² + y² | K Closest Points without `math.Sqrt` |
| Cross product (bx−ax)(cy−ay) − (by−ay)(cx−ax) | Sign gives turn direction; convex hull, collinear points |
| Floor of log₂x = `bits.Len(uint(x)) - 1` | Sparse tables, powers of two |

For randomness use `math/rand/v2`: `rand.IntN(n)` picks a random pivot and `rand.Shuffle(len(a), func(i, j int) { a[i], a[j] = a[j], a[i] })` shuffles in place.

## 20. The DSA patterns

A pattern is a reusable shape of solution. Most interview problems are one pattern, occasionally two stacked together, so the real skill is spotting which one from the problem statement. Every entry below follows the same order: the signals that point to it, the idea, a Go template, solved problems, and the cost. Patterns already covered in depth earlier point back to that section instead of repeating the code.

### 20.1 Two pointers: opposite ends

Signals: the array is sorted (or sorting is allowed), you need a pair or triplet with a target sum, a palindrome check, or "the widest/largest container". Two pointers turn an O(n²) pair search into O(n).

The idea: start at both ends and move the pointer whose move can improve the answer. Sortedness guarantees that the discarded side can never be part of a better answer.

```go
// Template
l, r := 0, len(a)-1
for l < r {
    s := a[l] + a[r]
    switch {
    case s == target:
        // record the answer, then l++ and r-- (or return)
    case s < target:
        l++ // need a bigger sum
    default:
        r-- // need a smaller sum
    }
}
```

```go
// Two Sum II (sorted input, 1-indexed answer)
func twoSumSorted(numbers []int, target int) []int {
    l, r := 0, len(numbers)-1
    for l < r {
        s := numbers[l] + numbers[r]
        switch {
        case s == target:
            return []int{l + 1, r + 1}
        case s < target:
            l++
        default:
            r--
        }
    }
    return nil
}

// 3Sum: fix one number, run two pointers on the rest, skip duplicates at every level
func threeSum(nums []int) [][]int {
    slices.Sort(nums)
    var res [][]int
    for i := 0; i < len(nums)-2; i++ {
        if i > 0 && nums[i] == nums[i-1] {
            continue
        }
        l, r := i+1, len(nums)-1
        for l < r {
            s := nums[i] + nums[l] + nums[r]
            switch {
            case s == 0:
                res = append(res, []int{nums[i], nums[l], nums[r]})
                for l < r && nums[l] == nums[l+1] {
                    l++
                }
                for l < r && nums[r] == nums[r-1] {
                    r--
                }
                l++
                r--
            case s < 0:
                l++
            default:
                r--
            }
        }
    }
    return res
}

// Container With Most Water: the shorter wall limits the area, so move it
func maxArea(height []int) int {
    l, r, best := 0, len(height)-1, 0
    for l < r {
        best = max(best, min(height[l], height[r])*(r-l))
        if height[l] < height[r] {
            l++
        } else {
            r--
        }
    }
    return best
}

// Trapping Rain Water: water over a bar = min(maxLeft, maxRight) - bar
func trap(height []int) int {
    l, r := 0, len(height)-1
    leftMax, rightMax, water := 0, 0, 0
    for l < r {
        if height[l] < height[r] { // the left side is the bottleneck
            leftMax = max(leftMax, height[l])
            water += leftMax - height[l]
            l++
        } else {
            rightMax = max(rightMax, height[r])
            water += rightMax - height[r]
            r--
        }
    }
    return water
}
```

Cost: O(n) time and O(1) space per scan; 3Sum is O(n²) because of the outer loop, 4Sum O(n³).

### 20.2 Two pointers: same direction (read and write)

Signals: modify the array in place, "return the new length", remove duplicates or a value, move zeroes, merge sorted arrays, check a subsequence.

The idea: a read pointer scans everything, a write pointer marks where the next kept element goes. Everything before the write pointer is the finished answer.

```go
// Remove Duplicates from Sorted Array: keep the first of each run
func removeDuplicates(nums []int) int {
    w := 1
    for r := 1; r < len(nums); r++ {
        if nums[r] != nums[w-1] {
            nums[w] = nums[r]
            w++
        }
    }
    return w // nums[:w] is the answer
}

// Move Zeroes: swap every non-zero into the write slot, keeping order
func moveZeroes(nums []int) {
    w := 0
    for r := range nums {
        if nums[r] != 0 {
            nums[w], nums[r] = nums[r], nums[w]
            w++
        }
    }
}

// Is Subsequence: advance through s only on a match
func isSubsequence(s, t string) bool {
    i := 0
    for j := 0; j < len(t) && i < len(s); j++ {
        if s[i] == t[j] {
            i++
        }
    }
    return i == len(s)
}

// Merge Sorted Array: fill nums1 from the back so nothing is overwritten
func mergeInPlace(nums1 []int, m int, nums2 []int, n int) {
    i, j := m-1, n-1
    for w := m + n - 1; j >= 0; w-- {
        if i >= 0 && nums1[i] > nums2[j] {
            nums1[w] = nums1[i]
            i--
        } else {
            nums1[w] = nums2[j]
            j--
        }
    }
}

// Sort Colors (Dutch national flag): [0, lo) = 0s, [lo, mid) = 1s, (hi, end] = 2s
func sortColors(nums []int) {
    lo, mid, hi := 0, 0, len(nums)-1
    for mid <= hi {
        switch nums[mid] {
        case 0:
            nums[lo], nums[mid] = nums[mid], nums[lo]
            lo++
            mid++
        case 1:
            mid++
        case 2:
            nums[mid], nums[hi] = nums[hi], nums[mid]
            hi-- // do not advance mid: the swapped-in value is unchecked
        }
    }
}
```

Cost: O(n) time, O(1) space. In Go, the idiomatic way to return the result of an in-place filter is the re-sliced `nums[:w]`.

### 20.3 Fast and slow pointers

Signals: a linked list with a possible cycle, the middle of a list, "happy number" style sequences that either end or loop, or an array whose values are indices into itself (Find the Duplicate Number).

The idea: the fast pointer moves two steps per slow step. If there is a loop they must meet; if not, fast reaches the end when slow is at the middle. `middle` and `detectCycle` are in Section 7.

```go
// Happy Number: the digit-square sequence either reaches 1 or cycles
func isHappy(n int) bool {
    next := func(x int) int {
        s := 0
        for ; x > 0; x /= 10 {
            d := x % 10
            s += d * d
        }
        return s
    }
    slow, fast := n, next(n)
    for fast != 1 && slow != fast {
        slow = next(slow)
        fast = next(next(fast))
    }
    return fast == 1
}

// Find the Duplicate Number: treat i -> nums[i] as a linked list; the duplicate is the cycle start
func findDuplicate(nums []int) int {
    slow, fast := nums[0], nums[nums[0]]
    for slow != fast {
        slow = nums[slow]
        fast = nums[nums[fast]]
    }
    slow = 0
    for slow != fast {
        slow = nums[slow]
        fast = nums[fast]
    }
    return slow
}

// Palindrome Linked List: find the middle, reverse the second half, compare
func isPalindromeList(head *ListNode) bool {
    slow, fast := head, head
    for fast != nil && fast.Next != nil {
        slow, fast = slow.Next, fast.Next.Next
    }
    second := reverseList(slow) // from Section 7
    for p, q := head, second; q != nil; p, q = p.Next, q.Next {
        if p.Val != q.Val {
            return false
        }
    }
    return true
}
```

Cost: O(n) time, O(1) space, which is the reason to prefer it over a visited set.

### 20.4 Sliding window: fixed size

Signals: "every subarray or substring of length k", maximum or average of k consecutive elements, "find all anagrams" or "permutation in string".

The idea: when the window slides by one, add the element entering on the right and remove the one leaving on the left, instead of recomputing the whole window.

```go
// Template
for i := range a {
    // add a[i] to the window state
    if i >= k {
        // remove a[i-k] from the window state
    }
    if i >= k-1 {
        // window a[i-k+1 : i+1] is full: record the answer
    }
}
```

```go
// Maximum Average Subarray I
func findMaxAverage(nums []int, k int) float64 {
    sum, best := 0, math.MinInt
    for i, v := range nums {
        sum += v
        if i >= k {
            sum -= nums[i-k]
        }
        if i >= k-1 {
            best = max(best, sum)
        }
    }
    return float64(best) / float64(k)
}

// Find All Anagrams in a String: Go arrays are comparable, so == compares all 26 counts
func findAnagrams(s, p string) []int {
    if len(p) > len(s) {
        return nil
    }
    var need, have [26]int
    for i := 0; i < len(p); i++ {
        need[p[i]-'a']++
    }
    var res []int
    for i := 0; i < len(s); i++ {
        have[s[i]-'a']++
        if i >= len(p) {
            have[s[i-len(p)]-'a']--
        }
        if have == need {
            res = append(res, i-len(p)+1)
        }
    }
    return res
}
```

Cost: O(n) time; O(1) or O(alphabet) space.

### 20.5 Sliding window: variable size

Signals: the longest or shortest subarray/substring that satisfies a condition, "at most k distinct", "without repeating characters", "minimum window containing", or "sum at least target" with non-negative numbers. The condition must be monotonic: shrinking a valid window keeps it valid (for longest) or growing an invalid one can fix it (for shortest). With negative numbers that breaks, so use prefix sums (20.6) instead.

```go
// Template for LONGEST: grow right, shrink left while invalid, record after
l := 0
for r := range len(a) {
    // add a[r]
    for /* window is invalid */ false {
        // remove a[l]
        l++
    }
    best = max(best, r-l+1)
}

// Template for SHORTEST: grow right, record and shrink left while valid
l := 0
for r := range len(a) {
    // add a[r]
    for /* window is valid */ false {
        best = min(best, r-l+1)
        // remove a[l]
        l++
    }
}
```

```go
// Longest Substring Without Repeating Characters: jump l past the last copy
func lengthOfLongestSubstring(s string) int {
    last := map[byte]int{} // byte -> last index seen
    best, l := 0, 0
    for r := 0; r < len(s); r++ {
        if i, ok := last[s[r]]; ok && i >= l {
            l = i + 1
        }
        last[s[r]] = r
        best = max(best, r-l+1)
    }
    return best
}

// Minimum Size Subarray Sum (positive numbers)
func minSubArrayLen(target int, nums []int) int {
    best, sum, l := math.MaxInt, 0, 0
    for r, v := range nums {
        sum += v
        for sum >= target {
            best = min(best, r-l+1)
            sum -= nums[l]
            l++
        }
    }
    if best == math.MaxInt {
        return 0
    }
    return best
}

// Longest Repeating Character Replacement: valid while (size - most frequent) <= k
func characterReplacement(s string, k int) int {
    var cnt [26]int
    best, maxF, l := 0, 0, 0
    for r := 0; r < len(s); r++ {
        cnt[s[r]-'A']++
        maxF = max(maxF, cnt[s[r]-'A'])
        for r-l+1-maxF > k {
            cnt[s[l]-'A']--
            l++
        }
        best = max(best, r-l+1)
    }
    return best
}

// Minimum Window Substring: count how many required characters are fully covered
func minWindow(s, t string) string {
    var need [128]int
    for i := 0; i < len(t); i++ {
        need[t[i]]++
    }
    missing := len(t)
    start, best, l := 0, math.MaxInt, 0
    for r := 0; r < len(s); r++ {
        if need[s[r]] > 0 {
            missing--
        }
        need[s[r]]-- // negative means surplus
        for missing == 0 {
            if r-l+1 < best {
                start, best = l, r-l+1
            }
            need[s[l]]++
            if need[s[l]] > 0 {
                missing++
            }
            l++
        }
    }
    if best == math.MaxInt {
        return ""
    }
    return s[start : start+best]
}

// "Exactly k" = atMost(k) - atMost(k-1): Subarrays with K Different Integers
func subarraysWithKDistinct(nums []int, k int) int {
    atMost := func(k int) int {
        cnt := map[int]int{}
        res, l := 0, 0
        for r, v := range nums {
            cnt[v]++
            for len(cnt) > k {
                cnt[nums[l]]--
                if cnt[nums[l]] == 0 {
                    delete(cnt, nums[l]) // so len(cnt) counts distinct values
                }
                l++
            }
            res += r - l + 1 // every subarray ending at r and starting in [l, r]
        }
        return res
    }
    return atMost(k) - atMost(k-1)
}
```

Cost: O(n) time, because each index enters and leaves the window once, even with the inner loop.

### 20.6 Prefix sum

Signals: many range-sum queries, "number of subarrays whose sum equals k" or "is divisible by k", negative numbers present (so the sliding window fails), equal counts of two kinds (0s and 1s), or sums over 2D rectangles.

The idea: `pre[i]` is the sum of the first i elements, so any subarray sum is a difference of two prefixes. Storing earlier prefixes in a map turns "find a subarray" into "find an earlier prefix", the same trick as Two Sum.

```go
// Build once in O(n): sum(a[l..r]) = pre[r+1] - pre[l]
pre := make([]int, len(a)+1)
for i, v := range a {
    pre[i+1] = pre[i] + v
}
```

```go
// Subarray Sum Equals K: count earlier prefixes equal to current - k
func subarraySum(nums []int, k int) int {
    seen := map[int]int{0: 1} // the empty prefix
    sum, count := 0, 0
    for _, v := range nums {
        sum += v
        count += seen[sum-k] // a missing key reads as 0
        seen[sum]++
    }
    return count
}

// Subarray Sums Divisible by K: equal remainders mean the gap is divisible
func subarraysDivByK(nums []int, k int) int {
    seen := map[int]int{0: 1}
    sum, count := 0, 0
    for _, v := range nums {
        sum += v
        m := (sum%k + k) % k // Go's % can be negative
        count += seen[m]
        seen[m]++
    }
    return count
}

// Contiguous Array: longest subarray with equal 0s and 1s; map 0 to -1, look for sum 0
func findMaxLength(nums []int) int {
    first := map[int]int{0: -1} // prefix sum -> first index it appeared
    sum, best := 0, 0
    for i, v := range nums {
        if v == 0 {
            sum--
        } else {
            sum++
        }
        if j, ok := first[sum]; ok {
            best = max(best, i-j)
        } else {
            first[sum] = i
        }
    }
    return best
}

// Product of Array Except Self: prefix products from the left, suffix from the right
func productExceptSelf(nums []int) []int {
    n := len(nums)
    res := make([]int, n)
    res[0] = 1
    for i := 1; i < n; i++ {
        res[i] = res[i-1] * nums[i-1]
    }
    suffix := 1
    for i := n - 1; i >= 0; i-- {
        res[i] *= suffix
        suffix *= nums[i]
    }
    return res
}

// Range Sum Query 2D: pre[i][j] = sum of the rectangle above-left of (i, j)
type NumMatrix struct{ pre [][]int }

func NewNumMatrix(matrix [][]int) NumMatrix {
    m, n := len(matrix), len(matrix[0])
    pre := make([][]int, m+1)
    for i := range pre {
        pre[i] = make([]int, n+1)
    }
    for i := range m {
        for j := range n {
            pre[i+1][j+1] = matrix[i][j] + pre[i][j+1] + pre[i+1][j] - pre[i][j]
        }
    }
    return NumMatrix{pre}
}

func (nm *NumMatrix) SumRegion(r1, c1, r2, c2 int) int {
    p := nm.pre
    return p[r2+1][c2+1] - p[r1][c2+1] - p[r2+1][c1] + p[r1][c1]
}
```

Cost: O(n) to build, O(1) per query; the hashmap versions are O(n) time and space. Use a Fenwick tree (11.3) when values change between queries.

### 20.7 Hashing and frequency counting

Signals: "have we seen this before", duplicates, counting occurrences, anagrams, the first unique element, or any inner loop that only searches for a value (a map makes that O(1)). Two Sum and Group Anagrams are in Section 6.

```go
// Contains Duplicate: map[int]struct{} is Go's set
func containsDuplicate(nums []int) bool {
    seen := make(map[int]struct{}, len(nums))
    for _, v := range nums {
        if _, ok := seen[v]; ok {
            return true
        }
        seen[v] = struct{}{}
    }
    return false
}

// First Unique Character: a [26]int beats a map for lowercase letters
func firstUniqChar(s string) int {
    var cnt [26]int
    for i := 0; i < len(s); i++ {
        cnt[s[i]-'a']++
    }
    for i := 0; i < len(s); i++ {
        if cnt[s[i]-'a'] == 1 {
            return i
        }
    }
    return -1
}

// Longest Consecutive Sequence: only start counting at the first number of a run
func longestConsecutive(nums []int) int {
    set := make(map[int]bool, len(nums))
    for _, v := range nums {
        set[v] = true
    }
    best := 0
    for v := range set {
        if set[v-1] {
            continue // not the start of a run
        }
        length := 1
        for set[v+length] {
            length++
        }
        best = max(best, length)
    }
    return best
}
```

Cost: O(n) expected time and O(n) space. For small alphabets prefer a fixed array: it is faster, and arrays can be compared with `==` or used as map keys.

### 20.8 Kadane's algorithm

Signals: maximum (or minimum) sum of a contiguous subarray, the best single stretch, or a circular variant.

The idea: at each index, either extend the best subarray that ends at the previous index or start fresh here, whichever is larger. It is a one-variable DP.

```go
// Maximum Subarray
func maxSubArray(nums []int) int {
    cur, best := nums[0], nums[0]
    for _, v := range nums[1:] {
        cur = max(v, cur+v) // extend or restart
        best = max(best, cur)
    }
    return best
}

// Maximum Product Subarray: a negative flips min and max, so track both
func maxProduct(nums []int) int {
    hi, lo, best := nums[0], nums[0], nums[0]
    for _, v := range nums[1:] {
        if v < 0 {
            hi, lo = lo, hi
        }
        hi = max(v, hi*v)
        lo = min(v, lo*v)
        best = max(best, hi)
    }
    return best
}

// Maximum Sum Circular Subarray: best wraps around = total - worst middle stretch
func maxSubarraySumCircular(nums []int) int {
    total := 0
    curMax, bestMax := 0, nums[0]
    curMin, bestMin := 0, nums[0]
    for _, v := range nums {
        total += v
        curMax = max(v, curMax+v)
        bestMax = max(bestMax, curMax)
        curMin = min(v, curMin+v)
        bestMin = min(bestMin, curMin)
    }
    if bestMax < 0 {
        return bestMax // all negative: wrapping would pick an empty array
    }
    return max(bestMax, total-bestMin)
}
```

Cost: O(n) time, O(1) space.

### 20.9 Difference array and sweep line

Signals: many range updates ("add v to every index from l to r") followed by one read, bookings or passengers over a range, or "the maximum number of overlapping intervals at any moment".

The idea: a difference array records only where a change starts and stops, so each range update is O(1). One prefix-sum pass then rebuilds the final values. A sweep line does the same with sorted events when coordinates are large or sparse.

```go
// Template: updates[i] = [l, r, v] on indices 0..n-1
diff := make([]int, n+1)
for _, u := range updates {
    diff[u[0]] += u[2]
    diff[u[1]+1] -= u[2] // the extra slot at n absorbs r = n-1
}
for i := 1; i < n; i++ {
    diff[i] += diff[i-1] // diff[0..n-1] now holds the final values
}
```

```go
// Corporate Flight Bookings: bookings[i] = [first, last, seats], flights are 1..n
func corpFlightBookings(bookings [][]int, n int) []int {
    diff := make([]int, n+1)
    for _, b := range bookings {
        diff[b[0]-1] += b[2]
        diff[b[1]] -= b[2]
    }
    for i := 1; i < n; i++ {
        diff[i] += diff[i-1]
    }
    return diff[:n]
}

// Car Pooling: trips[i] = [passengers, from, to]; locations are 0..1000
func carPooling(trips [][]int, capacity int) bool {
    var diff [1001]int
    for _, t := range trips {
        diff[t[1]] += t[0]
        diff[t[2]] -= t[0] // passengers leave at 'to'
    }
    load := 0
    for _, d := range diff {
        load += d
        if load > capacity {
            return false
        }
    }
    return true
}

// Sweep line: maximum overlapping meetings (Meeting Rooms II without a heap)
func minMeetingRoomsSweep(intervals [][]int) int {
    type event struct{ t, d int }
    ev := make([]event, 0, 2*len(intervals))
    for _, in := range intervals {
        ev = append(ev, event{in[0], +1}, event{in[1], -1})
    }
    slices.SortFunc(ev, func(a, b event) int {
        // cmp.Or returns the first non-zero comparison; -1 sorts before +1 at equal times,
        // so a room is freed before the next meeting takes it
        return cmp.Or(cmp.Compare(a.t, b.t), cmp.Compare(a.d, b.d))
    })
    cur, best := 0, 0
    for _, e := range ev {
        cur += e.d
        best = max(best, cur)
    }
    return best
}
```

Cost: O(n + q) for the difference array; O(n log n) for the sweep because of the sort. The 2D version marks four corners and takes a 2D prefix sum (20.6).

### 20.10 Cyclic sort

Signals: n numbers in the range 1..n (or 0..n), and you must find the missing, duplicated or first missing positive number in O(1) extra space.

The idea: every value v belongs at index v−1. Keep swapping the current value into its home until the right value sits here, then move on. Afterwards, any index i that does not hold i+1 exposes the answer.

```go
// Template: place each v in 1..n at index v-1; ignore values out of range
func cyclicSort(nums []int) {
    i := 0
    for i < len(nums) {
        v := nums[i]
        if v >= 1 && v <= len(nums) && nums[v-1] != v {
            nums[i], nums[v-1] = nums[v-1], v
        } else {
            i++
        }
    }
}

// First Missing Positive: O(n) time, O(1) space
func firstMissingPositive(nums []int) int {
    cyclicSort(nums)
    for i, v := range nums {
        if v != i+1 {
            return i + 1
        }
    }
    return len(nums) + 1
}

// Find All Numbers Disappeared in an Array
func findDisappearedNumbers(nums []int) []int {
    cyclicSort(nums)
    var res []int
    for i, v := range nums {
        if v != i+1 {
            res = append(res, i+1)
        }
    }
    return res
}

// Find All Duplicates in an Array: the misplaced values are the duplicates
func findDuplicates(nums []int) []int {
    cyclicSort(nums)
    var res []int
    for i, v := range nums {
        if v != i+1 {
            res = append(res, v)
        }
    }
    return res
}
```

Cost: O(n) time even with the inner swapping, because each swap puts one value home for good; O(1) space.

### 20.11 Merge intervals

Signals: a list of \[start, end\] pairs with "merge overlapping", "insert an interval", "intersection of two lists" or "free time". Sort by start first; then two intervals overlap exactly when the next one starts before the current one ends. Choosing which intervals to keep or remove is greedy (Section 17).

```go
// Merge Intervals
func mergeIntervals(intervals [][]int) [][]int {
    slices.SortFunc(intervals, func(a, b []int) int { return cmp.Compare(a[0], b[0]) })
    res := [][]int{intervals[0]}
    for _, in := range intervals[1:] {
        last := res[len(res)-1] // shares memory with res, so writing last[1] updates res
        if in[0] <= last[1] {
            last[1] = max(last[1], in[1])
        } else {
            res = append(res, in)
        }
    }
    return res
}

// Insert Interval into a sorted, non-overlapping list: before, overlapping, after
func insertInterval(intervals [][]int, newInterval []int) [][]int {
    var res [][]int
    i, n := 0, len(intervals)
    for i < n && intervals[i][1] < newInterval[0] {
        res = append(res, intervals[i])
        i++
    }
    lo, hi := newInterval[0], newInterval[1]
    for i < n && intervals[i][0] <= hi {
        lo = min(lo, intervals[i][0])
        hi = max(hi, intervals[i][1])
        i++
    }
    res = append(res, []int{lo, hi})
    return append(res, intervals[i:]...)
}

// Interval List Intersections: two pointers; advance whichever ends first
func intervalIntersection(a, b [][]int) [][]int {
    var res [][]int
    i, j := 0, 0
    for i < len(a) && j < len(b) {
        lo := max(a[i][0], b[j][0])
        hi := min(a[i][1], b[j][1])
        if lo <= hi {
            res = append(res, []int{lo, hi})
        }
        if a[i][1] < b[j][1] {
            i++
        } else {
            j++
        }
    }
    return res
}
```

Cost: O(n log n) for the sort, O(n) after it. `mergeIntervals` modifies the input's inner slices; clone them first if the caller still needs the original.

### 20.12 In-place linked list reversal

Signals: reverse a whole list, a sublist between two positions, every k nodes, swap pairs, or reorder a list by interleaving its ends. The whole-list versions are in Section 7.

The idea: a dummy node in front of the head removes the special case of reversing from position 1, and the "move the next node to the front" loop reverses a stretch without extra memory.

```go
// Reverse Linked List II: reverse positions left..right (1-indexed)
func reverseBetween(head *ListNode, left, right int) *ListNode {
    dummy := &ListNode{Next: head}
    prev := dummy
    for range left - 1 {
        prev = prev.Next // prev stops just before the stretch
    }
    cur := prev.Next
    for range right - left {
        next := cur.Next // move next to the front of the stretch
        cur.Next = next.Next
        next.Next = prev.Next
        prev.Next = next
    }
    return dummy.Next
}

// Reverse Nodes in k-Group
func reverseKGroup(head *ListNode, k int) *ListNode {
    dummy := &ListNode{Next: head}
    groupPrev := dummy
    for {
        kth := groupPrev
        for range k {
            kth = kth.Next
            if kth == nil {
                return dummy.Next // fewer than k nodes left: leave them as they are
            }
        }
        groupNext := kth.Next
        prev, cur := groupNext, groupPrev.Next
        for cur != groupNext {
            // Go evaluates every right-hand value first, so this swap needs no temp
            cur.Next, prev, cur = prev, cur, cur.Next
        }
        tail := groupPrev.Next // the old first node is now the group's tail
        groupPrev.Next = kth
        groupPrev = tail
    }
}

// Swap Nodes in Pairs
func swapPairs(head *ListNode) *ListNode {
    dummy := &ListNode{Next: head}
    prev := dummy
    for prev.Next != nil && prev.Next.Next != nil {
        a, b := prev.Next, prev.Next.Next
        a.Next = b.Next
        b.Next = a
        prev.Next = b
        prev = a
    }
    return dummy.Next
}

// Reorder List: L0 -> Ln -> L1 -> Ln-1 ... (middle, reverse, weave)
func reorderList(head *ListNode) {
    if head == nil {
        return
    }
    slow, fast := head, head
    for fast.Next != nil && fast.Next.Next != nil {
        slow, fast = slow.Next, fast.Next.Next
    }
    second := reverseList(slow.Next)
    slow.Next = nil // cut the list in two
    for first := head; second != nil; {
        n1, n2 := first.Next, second.Next
        first.Next = second
        second.Next = n1
        first, second = n1, n2
    }
}
```

Cost: O(n) time, O(1) space. Draw the pointers on paper for three nodes before coding; almost every bug here is a lost `Next`.

### 20.13 Monotonic stack

Signals: "next greater element", "next smaller", "previous smaller", "how many days until a warmer day", stock span, the largest rectangle in a histogram, or "remove k digits to make the smallest number".

The idea: keep a stack of indices whose values are sorted. When a new value breaks the order, every popped index has just found its answer: the new value is its next greater (or smaller) element. Each index is pushed and popped once, so the whole scan is O(n).

| You want | Keep the stack | Pop while | Answer comes from |
| --- | --- | --- | --- |
| Next greater | Decreasing | top value < current | current, for each popped index |
| Next smaller | Increasing | top value > current | current, for each popped index |
| Previous greater | Decreasing | top value <= current | the top after popping |
| Previous smaller | Increasing | top value >= current | the top after popping |

```go
// Next Greater Element for every index, -1 if none
func nextGreater(nums []int) []int {
    res := make([]int, len(nums))
    for i := range res {
        res[i] = -1
    }
    stack := []int{} // indices with decreasing values
    for i, v := range nums {
        for len(stack) > 0 && nums[stack[len(stack)-1]] < v {
            res[stack[len(stack)-1]] = v
            stack = stack[:len(stack)-1]
        }
        stack = append(stack, i)
    }
    return res
}

// Daily Temperatures: the answer is the distance, not the value
func dailyTemperatures(t []int) []int {
    res := make([]int, len(t))
    stack := []int{}
    for i, v := range t {
        for len(stack) > 0 && t[stack[len(stack)-1]] < v {
            j := stack[len(stack)-1]
            stack = stack[:len(stack)-1]
            res[j] = i - j
        }
        stack = append(stack, i)
    }
    return res
}

// Next Greater Element II (circular): walk the array twice, push only on the first lap
func nextGreaterElements(nums []int) []int {
    n := len(nums)
    res := make([]int, n)
    for i := range res {
        res[i] = -1
    }
    stack := []int{}
    for i := range 2 * n {
        v := nums[i%n]
        for len(stack) > 0 && nums[stack[len(stack)-1]] < v {
            res[stack[len(stack)-1]] = v
            stack = stack[:len(stack)-1]
        }
        if i < n {
            stack = append(stack, i)
        }
    }
    return res
}

// Largest Rectangle in Histogram: a popped bar's rectangle spans between its smaller neighbors
func largestRectangleArea(heights []int) int {
    stack := []int{}
    best := 0
    for i := 0; i <= len(heights); i++ {
        h := 0 // a height-0 sentinel at the end flushes the stack
        if i < len(heights) {
            h = heights[i]
        }
        for len(stack) > 0 && heights[stack[len(stack)-1]] >= h {
            top := stack[len(stack)-1]
            stack = stack[:len(stack)-1]
            left := -1 // index of the previous smaller bar
            if len(stack) > 0 {
                left = stack[len(stack)-1]
            }
            best = max(best, heights[top]*(i-left-1))
        }
        stack = append(stack, i)
    }
    return best
}

// Remove K Digits: drop a digit whenever a smaller one follows it
func removeKdigits(num string, k int) string {
    stack := make([]byte, 0, len(num))
    for i := 0; i < len(num); i++ {
        for k > 0 && len(stack) > 0 && stack[len(stack)-1] > num[i] {
            stack = stack[:len(stack)-1]
            k--
        }
        stack = append(stack, num[i])
    }
    stack = stack[:len(stack)-k] // any removals left come off the end
    res := strings.TrimLeft(string(stack), "0")
    if res == "" {
        return "0"
    }
    return res
}
```

Cost: O(n) time and O(n) space. Maximal Rectangle in a binary matrix is Largest Rectangle run once per row on accumulated column heights.

### 20.14 Monotonic deque

Signals: the maximum or minimum of every window of size k, the shortest subarray with sum at least k when negatives are present, or a DP whose transition takes the best value from the last k states.

The idea: a deque of indices kept in decreasing value order. The front is always the current window's maximum. Pop from the back anything the new value beats (it can never be a maximum again), and pop from the front any index that has slid out of the window.

```go
// Sliding Window Maximum
func maxSlidingWindow(nums []int, k int) []int {
    dq := []int{} // indices; values decrease from front to back
    res := make([]int, 0, len(nums)-k+1)
    for i, v := range nums {
        if len(dq) > 0 && dq[0] <= i-k {
            dq = dq[1:] // the front index left the window
        }
        for len(dq) > 0 && nums[dq[len(dq)-1]] <= v {
            dq = dq[:len(dq)-1]
        }
        dq = append(dq, i)
        if i >= k-1 {
            res = append(res, nums[dq[0]])
        }
    }
    return res
}

// Shortest Subarray with Sum at Least K (negatives allowed): deque over prefix sums
func shortestSubarray(nums []int, k int) int {
    n := len(nums)
    pre := make([]int, n+1)
    for i, v := range nums {
        pre[i+1] = pre[i] + v
    }
    best := n + 1
    dq := []int{} // indices with increasing prefix sums
    for i := range n + 1 {
        for len(dq) > 0 && pre[i]-pre[dq[0]] >= k {
            best = min(best, i-dq[0]) // the shortest window starting at dq[0]
            dq = dq[1:]
        }
        for len(dq) > 0 && pre[dq[len(dq)-1]] >= pre[i] {
            dq = dq[:len(dq)-1] // a later, smaller prefix is always a better start
        }
        dq = append(dq, i)
    }
    if best == n+1 {
        return -1
    }
    return best
}
```

Cost: O(n) time, O(k) or O(n) space. A slice works as the deque here because items leave only from the ends; `dq = dq[1:]` is O(1).

### 20.15 Top-K elements

Signals: "k largest", "k smallest", "k most frequent", "k closest", or "the kth largest". These examples use the generic `Heap[T]` from 9.5, which takes a comparator and offers `Push`, `Pop`, `Peek` and `Len`.

The idea: to keep the k largest, hold a min-heap of size k and evict the top whenever it grows past k; the top is then the kth largest. For the k smallest, flip it and use a max-heap. It feels backwards at first: the heap's top is the element you are ready to throw away.

```go
// Top K Frequent Elements with a size-k min-heap: O(n log k)
func topKFrequent(nums []int, k int) []int {
    freq := map[int]int{}
    for _, v := range nums {
        freq[v]++
    }
    type pair struct{ val, cnt int }
    h := NewHeap(func(a, b pair) bool { return a.cnt < b.cnt })
    for v, c := range freq {
        h.Push(pair{v, c})
        if h.Len() > k {
            h.Pop() // evict the least frequent
        }
    }
    res := make([]int, 0, k)
    for h.Len() > 0 {
        res = append(res, h.Pop().val)
    }
    return res
}

// Same problem with bucket sort: counts are at most n, so index buckets by count, O(n)
func topKFrequentBucket(nums []int, k int) []int {
    freq := map[int]int{}
    for _, v := range nums {
        freq[v]++
    }
    buckets := make([][]int, len(nums)+1) // buckets[c] = values seen c times
    for v, c := range freq {
        buckets[c] = append(buckets[c], v)
    }
    res := make([]int, 0, k)
    for c := len(buckets) - 1; c > 0 && len(res) < k; c-- {
        res = append(res, buckets[c]...)
    }
    return res[:k]
}

// K Closest Points to Origin: size-k max-heap on squared distance (no sqrt needed)
func kClosest(points [][]int, k int) [][]int {
    d := func(p []int) int { return p[0]*p[0] + p[1]*p[1] }
    h := NewHeap(func(a, b []int) bool { return d(a) > d(b) })
    for _, p := range points {
        h.Push(p)
        if h.Len() > k {
            h.Pop() // evict the farthest
        }
    }
    res := make([][]int, 0, k)
    for h.Len() > 0 {
        res = append(res, h.Pop())
    }
    return res
}
```

Cost: O(n log k) time, O(k) space. Alternatives: sort everything for O(n log n) when n is small, or `quickSelect` (Section 13) for O(n) average when you only need the kth value.

### 20.16 K-way merge

Signals: k sorted lists or arrays to merge, the kth smallest in a row- and column-sorted matrix, the smallest range that covers k lists, or the k pairs with the smallest sums.

The idea: put the head of every list in a min-heap. Pop the smallest, output it, and push the next element from the same list. The heap never holds more than k items.

```go
// Merge k Sorted Lists
func mergeKLists(lists []*ListNode) *ListNode {
    h := NewHeap(func(a, b *ListNode) bool { return a.Val < b.Val })
    for _, l := range lists {
        if l != nil {
            h.Push(l)
        }
    }
    dummy := &ListNode{}
    tail := dummy
    for h.Len() > 0 {
        n := h.Pop()
        tail.Next = n
        tail = n
        if n.Next != nil {
            h.Push(n.Next) // the next candidate from the same list
        }
    }
    return dummy.Next
}

// Kth Smallest Element in a Sorted Matrix: every row is a sorted list
func kthSmallestMatrix(matrix [][]int, k int) int {
    type cell struct{ val, r, c int }
    h := NewHeap(func(a, b cell) bool { return a.val < b.val })
    for r := range min(len(matrix), k) {
        h.Push(cell{matrix[r][0], r, 0})
    }
    for range k - 1 {
        x := h.Pop()
        if x.c+1 < len(matrix[0]) {
            h.Push(cell{matrix[x.r][x.c+1], x.r, x.c + 1})
        }
    }
    return h.Peek().val
}

// Smallest Range Covering Elements from K Lists: heap gives the low end, track the high end
func smallestRange(nums [][]int) []int {
    type item struct{ val, list, idx int }
    h := NewHeap(func(a, b item) bool { return a.val < b.val })
    hi := math.MinInt
    for i, l := range nums {
        h.Push(item{l[0], i, 0})
        hi = max(hi, l[0])
    }
    best := []int{h.Peek().val, hi}
    for {
        x := h.Pop()
        if x.idx+1 == len(nums[x.list]) {
            return best // one list ran out, so no later range can cover every list
        }
        next := nums[x.list][x.idx+1]
        h.Push(item{next, x.list, x.idx + 1})
        hi = max(hi, next)
        if lo := h.Peek().val; hi-lo < best[1]-best[0] {
            best = []int{lo, hi}
        }
    }
}
```

Cost: O(N log k) where N is the total number of elements and k the number of lists.

### 20.17 Two heaps

Signals: a running median of a stream, the median of every window, or a problem that repeatedly needs the best item among those that just became available (IPO).

The idea for medians: a max-heap holds the smaller half and a min-heap holds the larger half. Keep their sizes equal or the low half one bigger; the median then sits on the tops.

```go
type MedianFinder struct {
    low  *Heap[int] // max-heap: the smaller half
    high *Heap[int] // min-heap: the larger half
}

func NewMedianFinder() MedianFinder {
    return MedianFinder{
        low:  NewHeap(func(a, b int) bool { return a > b }),
        high: NewHeap(func(a, b int) bool { return a < b }),
    }
}

func (m *MedianFinder) AddNum(num int) {
    m.low.Push(num)
    m.high.Push(m.low.Pop()) // the largest of the low half moves up
    if m.high.Len() > m.low.Len() {
        m.low.Push(m.high.Pop()) // rebalance so low is never smaller
    }
}

func (m *MedianFinder) FindMedian() float64 {
    if m.low.Len() > m.high.Len() {
        return float64(m.low.Peek())
    }
    return float64(m.low.Peek()+m.high.Peek()) / 2
}

// IPO: projects sorted by capital play the role of the second heap
func findMaximizedCapital(k, w int, profits, capital []int) int {
    idx := make([]int, len(profits))
    for i := range idx {
        idx[i] = i
    }
    slices.SortFunc(idx, func(a, b int) int { return cmp.Compare(capital[a], capital[b]) })
    affordable := NewHeap(func(a, b int) bool { return a > b }) // max-heap of profits
    i := 0
    for range k {
        for i < len(idx) && capital[idx[i]] <= w {
            affordable.Push(profits[idx[i]])
            i++
        }
        if affordable.Len() == 0 {
            break
        }
        w += affordable.Pop()
    }
    return w
}
```

Cost: O(log n) per insert and O(1) per median. Sliding Window Median needs removals from the middle of a heap; use lazy deletion (a map of values to skip when they reach the top) or an ordered structure.

### 20.18 Modified binary search

Signals: the input is sorted or rotated, or the answer lies in a range where a yes/no test flips exactly once. The full templates (`firstTrue`, lower and upper bound, search on answer, rotated arrays, peaks) are in Section 14; this is how to spot each variant, plus three problems not covered there.

| Clue in the problem | Variant | Where |
| --- | --- | --- |
| Sorted array, find a target or insert position | Lower bound | 14 |
| First and last position of a value | Lower bound of x and of x+1 | below |
| Rotated sorted array | Compare mid with the right end | 14 |
| "Minimize the maximum", "maximize the minimum", smallest speed or capacity | Binary search on the answer | 14, below |
| Peak or mountain array | Compare `a[mid]` with `a[mid+1]` | 14 |
| Latest value at or before a time | Upper bound minus one | below |
| Median or kth of two sorted arrays | Binary search a partition | below |

```go
// Find First and Last Position of Element in Sorted Array
func searchRange(nums []int, target int) []int {
    lo, found := slices.BinarySearch(nums, target) // first index with nums[i] >= target
    if !found {
        return []int{-1, -1}
    }
    hi, _ := slices.BinarySearch(nums, target+1) // first index with nums[i] > target
    return []int{lo, hi - 1}
}

// Split Array Largest Sum: search the smallest limit that allows at most k parts
func splitArray(nums []int, k int) int {
    lo, hi := slices.Max(nums), 0
    for _, v := range nums {
        hi += v
    }
    fits := func(limit int) bool { // greedy: start a new part only when forced
        parts, sum := 1, 0
        for _, v := range nums {
            if sum+v > limit {
                parts++
                sum = 0
            }
            sum += v
        }
        return parts <= k
    }
    return lo + sort.Search(hi-lo+1, func(i int) bool { return fits(lo + i) })
}

// Time Based Key-Value Store: timestamps arrive in increasing order, so each list is sorted
type entry struct {
    t int
    v string
}

type TimeMap struct{ m map[string][]entry }

func NewTimeMap() TimeMap { return TimeMap{m: map[string][]entry{}} }

func (tm *TimeMap) Set(key, value string, timestamp int) {
    tm.m[key] = append(tm.m[key], entry{timestamp, value})
}

func (tm *TimeMap) Get(key string, timestamp int) string {
    es := tm.m[key]
    i := sort.Search(len(es), func(i int) bool { return es[i].t > timestamp })
    if i == 0 {
        return "" // nothing at or before timestamp
    }
    return es[i-1].v
}

// Median of Two Sorted Arrays: choose how many elements the left half takes from a
func findMedianSortedArrays(a, b []int) float64 {
    if len(a) > len(b) {
        a, b = b, a // search the shorter array
    }
    m, n := len(a), len(b)
    half := (m + n + 1) / 2
    lo, hi := 0, m
    for lo <= hi {
        i := (lo + hi) / 2 // taken from a
        j := half - i      // taken from b
        aL, aR, bL, bR := math.MinInt, math.MaxInt, math.MinInt, math.MaxInt
        if i > 0 {
            aL = a[i-1]
        }
        if i < m {
            aR = a[i]
        }
        if j > 0 {
            bL = b[j-1]
        }
        if j < n {
            bR = b[j]
        }
        switch {
        case aL > bR:
            hi = i - 1 // took too many from a
        case bL > aR:
            lo = i + 1 // took too few from a
        default:
            if (m+n)%2 == 1 {
                return float64(max(aL, bL))
            }
            return float64(max(aL, bL)+min(aR, bR)) / 2
        }
    }
    return 0
}
```

Cost: O(log n) per search; search on the answer costs O(n log range) because each test is a linear scan.

### 20.19 Tree DFS

Signals: anything about paths (root-to-leaf, any downward path, maximum path), a value that depends on subtrees (height, diameter, balance), BST order, or building and serializing trees. Section 10 has the basic traversals, depth, diameter and LCA.

First decide which way information flows.

| Style | Use when | Examples |
| --- | --- | --- |
| Top-down: pass state in the parameters | The answer depends on the path from the root | Path Sum, Count Good Nodes, Sum Root to Leaf Numbers |
| Bottom-up: return values from children | A node's answer depends on its subtrees | Height, diameter, balanced, Maximum Path Sum |
| Inorder | Sorted BST order matters | Validate BST, kth smallest (Section 10) |
| Path collection with backtracking | Every qualifying path must be listed | Path Sum II, Binary Tree Paths |

```go
// Top-down: Path Sum
func hasPathSum(root *TreeNode, target int) bool {
    if root == nil {
        return false
    }
    if root.Left == nil && root.Right == nil {
        return target == root.Val
    }
    rest := target - root.Val
    return hasPathSum(root.Left, rest) || hasPathSum(root.Right, rest)
}

// Top-down with carried state: Count Good Nodes (no bigger value above on the path)
func goodNodes(root *TreeNode) int {
    var dfs func(n *TreeNode, maxAbove int) int
    dfs = func(n *TreeNode, maxAbove int) int {
        if n == nil {
            return 0
        }
        good := 0
        if n.Val >= maxAbove {
            good = 1
        }
        m := max(maxAbove, n.Val)
        return good + dfs(n.Left, m) + dfs(n.Right, m)
    }
    return dfs(root, math.MinInt)
}

// Bottom-up: Binary Tree Maximum Path Sum
func maxPathSum(root *TreeNode) int {
    best := math.MinInt
    var gain func(n *TreeNode) int // best downward path that starts at n
    gain = func(n *TreeNode) int {
        if n == nil {
            return 0
        }
        l := max(gain(n.Left), 0) // a negative branch is better left out
        r := max(gain(n.Right), 0)
        best = max(best, n.Val+l+r) // the path that bends at n
        return n.Val + max(l, r)    // a parent can extend only one side
    }
    gain(root)
    return best
}

// Path collection: Path Sum II (clone the path when you save it)
func pathSum(root *TreeNode, target int) [][]int {
    var res [][]int
    var path []int
    var dfs func(n *TreeNode, rest int)
    dfs = func(n *TreeNode, rest int) {
        if n == nil {
            return
        }
        path = append(path, n.Val)
        rest -= n.Val
        if n.Left == nil && n.Right == nil && rest == 0 {
            res = append(res, slices.Clone(path))
        }
        dfs(n.Left, rest)
        dfs(n.Right, rest)
        path = path[:len(path)-1] // undo before returning to the parent
    }
    dfs(root, target)
    return res
}

// Prefix sums on a tree: Path Sum III (any downward path)
func pathSumIII(root *TreeNode, target int) int {
    seen := map[int]int{0: 1}
    var dfs func(n *TreeNode, sum int) int
    dfs = func(n *TreeNode, sum int) int {
        if n == nil {
            return 0
        }
        sum += n.Val
        count := seen[sum-target]
        seen[sum]++
        count += dfs(n.Left, sum) + dfs(n.Right, sum)
        seen[sum]-- // this prefix is no longer on the path
        return count
    }
    return dfs(root, 0)
}

// Construct Binary Tree from Preorder and Inorder Traversal
func buildFromPreIn(preorder, inorder []int) *TreeNode {
    pos := make(map[int]int, len(inorder))
    for i, v := range inorder {
        pos[v] = i
    }
    next := 0 // index of the next root in preorder
    var build func(lo, hi int) *TreeNode
    build = func(lo, hi int) *TreeNode { // builds from inorder[lo..hi]
        if lo > hi {
            return nil
        }
        v := preorder[next]
        next++
        n := &TreeNode{Val: v}
        n.Left = build(lo, pos[v]-1) // left first: preorder lists it first
        n.Right = build(pos[v]+1, hi)
        return n
    }
    return build(0, len(inorder)-1)
}

// Serialize and Deserialize: preorder with "#" for nil
func serialize(root *TreeNode) string {
    var sb strings.Builder
    var dfs func(n *TreeNode)
    dfs = func(n *TreeNode) {
        if n == nil {
            sb.WriteString("#,")
            return
        }
        sb.WriteString(strconv.Itoa(n.Val))
        sb.WriteByte(',')
        dfs(n.Left)
        dfs(n.Right)
    }
    dfs(root)
    return sb.String()
}

func deserialize(data string) *TreeNode {
    tokens := strings.Split(strings.TrimSuffix(data, ","), ",")
    i := 0
    var dfs func() *TreeNode
    dfs = func() *TreeNode {
        t := tokens[i]
        i++
        if t == "#" {
            return nil
        }
        v, _ := strconv.Atoi(t)
        n := &TreeNode{Val: v}
        n.Left = dfs()
        n.Right = dfs()
        return n
    }
    return dfs()
}
```

Cost: O(n) time, O(h) stack space where h is the height (O(log n) balanced, O(n) for a chain).

### 20.20 Tree BFS

Signals: "level by level", "each level", right or left side view, zigzag order, the minimum depth, level averages, or the width of a level.

The idea: process the tree one level at a time. In Go the cleanest form swaps two slices: the current level and the next one.

```go
// Template: one outer iteration per level
level := []*TreeNode{root}
for len(level) > 0 {
    var next []*TreeNode
    for _, n := range level {
        // visit n
        if n.Left != nil {
            next = append(next, n.Left)
        }
        if n.Right != nil {
            next = append(next, n.Right)
        }
    }
    level = next
}
```

```go
// Binary Tree Right Side View: the last node of each level
func rightSideView(root *TreeNode) []int {
    var res []int
    if root == nil {
        return res
    }
    level := []*TreeNode{root}
    for len(level) > 0 {
        res = append(res, level[len(level)-1].Val)
        var next []*TreeNode
        for _, n := range level {
            if n.Left != nil {
                next = append(next, n.Left)
            }
            if n.Right != nil {
                next = append(next, n.Right)
            }
        }
        level = next
    }
    return res
}

// Zigzag Level Order: write odd levels from the back instead of reversing
func zigzagLevelOrder(root *TreeNode) [][]int {
    var res [][]int
    if root == nil {
        return res
    }
    level := []*TreeNode{root}
    for depth := 0; len(level) > 0; depth++ {
        vals := make([]int, len(level))
        var next []*TreeNode
        for i, n := range level {
            if depth%2 == 0 {
                vals[i] = n.Val
            } else {
                vals[len(level)-1-i] = n.Val
            }
            if n.Left != nil {
                next = append(next, n.Left)
            }
            if n.Right != nil {
                next = append(next, n.Right)
            }
        }
        res = append(res, vals)
        level = next
    }
    return res
}

// Minimum Depth: BFS can stop at the first leaf, DFS cannot
func minDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    level := []*TreeNode{root}
    for depth := 1; ; depth++ {
        var next []*TreeNode
        for _, n := range level {
            if n.Left == nil && n.Right == nil {
                return depth
            }
            if n.Left != nil {
                next = append(next, n.Left)
            }
            if n.Right != nil {
                next = append(next, n.Right)
            }
        }
        level = next
    }
}

// Maximum Width: number nodes like a heap (children 2i and 2i+1)
func widthOfBinaryTree(root *TreeNode) int {
    if root == nil {
        return 0
    }
    type item struct {
        n   *TreeNode
        idx int
    }
    level := []item{{root, 0}}
    best := 0
    for len(level) > 0 {
        first := level[0].idx
        best = max(best, level[len(level)-1].idx-first+1)
        var next []item
        for _, it := range level {
            i := it.idx - first // re-base each level so indices never overflow
            if it.n.Left != nil {
                next = append(next, item{it.n.Left, 2 * i})
            }
            if it.n.Right != nil {
                next = append(next, item{it.n.Right, 2*i + 1})
            }
        }
        level = next
    }
    return best
}
```

Cost: O(n) time, O(w) space where w is the widest level (up to n/2 for a full tree).

### 20.21 Multi-source and implicit-graph BFS

Signals: "minimum minutes until every cell is", "distance to the nearest X for every cell", several starting points at once, or a puzzle where each state (a word, a lock combination, a board) changes into neighbor states by one move. The graph is never given; you generate neighbors on the fly.

The idea: push every source into the queue before the first step. BFS then expands all of them in parallel, and the level count is the time or distance. The grid examples share one direction table.

```go
var dirs = [4][2]int{{1, 0}, {-1, 0}, {0, 1}, {0, -1}}

// Rotting Oranges: all rotten oranges spread at the same time
func orangesRotting(grid [][]int) int {
    m, n := len(grid), len(grid[0])
    type cell struct{ r, c int }
    var level []cell
    fresh := 0
    for r := range m {
        for c := range n {
            switch grid[r][c] {
            case 2:
                level = append(level, cell{r, c}) // every rotten orange is a source
            case 1:
                fresh++
            }
        }
    }
    minutes := 0
    for len(level) > 0 && fresh > 0 {
        var next []cell
        for _, p := range level {
            for _, d := range dirs {
                r, c := p.r+d[0], p.c+d[1]
                if r >= 0 && r < m && c >= 0 && c < n && grid[r][c] == 1 {
                    grid[r][c] = 2 // mark when pushed, not when popped
                    fresh--
                    next = append(next, cell{r, c})
                }
            }
        }
        level = next
        minutes++
    }
    if fresh > 0 {
        return -1
    }
    return minutes
}

// 01 Matrix: distance from every cell to the nearest 0, starting from all zeros
func updateMatrix(mat [][]int) [][]int {
    m, n := len(mat), len(mat[0])
    dist := make([][]int, m)
    var queue [][2]int
    for r := range m {
        dist[r] = make([]int, n)
        for c := range n {
            if mat[r][c] == 0 {
                queue = append(queue, [2]int{r, c})
            } else {
                dist[r][c] = -1 // not reached yet
            }
        }
    }
    for len(queue) > 0 {
        p := queue[0]
        queue = queue[1:]
        for _, d := range dirs {
            r, c := p[0]+d[0], p[1]+d[1]
            if r >= 0 && r < m && c >= 0 && c < n && dist[r][c] == -1 {
                dist[r][c] = dist[p[0]][p[1]] + 1
                queue = append(queue, [2]int{r, c})
            }
        }
    }
    return dist
}

// Word Ladder: words are nodes, one changed letter is an edge
func ladderLength(beginWord, endWord string, wordList []string) int {
    words := make(map[string]bool, len(wordList))
    for _, w := range wordList {
        words[w] = true
    }
    if !words[endWord] {
        return 0
    }
    delete(words, beginWord)
    level := []string{beginWord}
    for steps := 1; len(level) > 0; steps++ {
        var next []string
        for _, w := range level {
            if w == endWord {
                return steps
            }
            b := []byte(w)
            for i := range b {
                orig := b[i]
                for ch := byte('a'); ch <= 'z'; ch++ {
                    b[i] = ch
                    if s := string(b); words[s] {
                        delete(words, s) // removing from the set marks it visited
                        next = append(next, s)
                    }
                }
                b[i] = orig
            }
        }
        level = next
    }
    return 0
}
```

Cost: O(V+E); for Word Ladder that is O(N·L·26) for N words of length L. Searching from both ends at once (bidirectional BFS) cuts the explored states sharply on large inputs.

### 20.22 Matrix patterns: flood fill, border sweeps and traversal

Signals: islands, regions, connected cells, "cells that can reach the border", or walking a matrix in a special order (spiral, diagonal, rotated). Counting islands and the maze BFS are in 12.5.

The idea for regions: DFS from a cell and mark every cell you reach, either by overwriting the grid or with a `seen` matrix. For "reaches the border" problems, start from the border and search inward instead of testing every cell.

```go
// Max Area of Island: DFS returns the size of the region it floods
func maxAreaOfIsland(grid [][]int) int {
    m, n := len(grid), len(grid[0])
    var dfs func(r, c int) int
    dfs = func(r, c int) int {
        if r < 0 || r >= m || c < 0 || c >= n || grid[r][c] != 1 {
            return 0
        }
        grid[r][c] = 0 // sink the cell so it is counted once
        return 1 + dfs(r+1, c) + dfs(r-1, c) + dfs(r, c+1) + dfs(r, c-1)
    }
    best := 0
    for r := range m {
        for c := range n {
            best = max(best, dfs(r, c))
        }
    }
    return best
}

// Surrounded Regions: any 'O' connected to the border survives; flip the rest
func solve(board [][]byte) {
    m, n := len(board), len(board[0])
    var mark func(r, c int)
    mark = func(r, c int) {
        if r < 0 || r >= m || c < 0 || c >= n || board[r][c] != 'O' {
            return
        }
        board[r][c] = '#' // safe
        for _, d := range dirs {
            mark(r+d[0], c+d[1])
        }
    }
    for r := range m {
        mark(r, 0)
        mark(r, n-1)
    }
    for c := range n {
        mark(0, c)
        mark(m-1, c)
    }
    for r := range m {
        for c := range n {
            switch board[r][c] {
            case 'O':
                board[r][c] = 'X'
            case '#':
                board[r][c] = 'O'
            }
        }
    }
}

// Pacific Atlantic Water Flow: climb uphill from each ocean; answer = cells both reach
func pacificAtlantic(heights [][]int) [][]int {
    m, n := len(heights), len(heights[0])
    grid := func() [][]bool {
        g := make([][]bool, m)
        for i := range g {
            g[i] = make([]bool, n)
        }
        return g
    }
    pac, atl := grid(), grid()
    var dfs func(r, c int, seen [][]bool)
    dfs = func(r, c int, seen [][]bool) {
        seen[r][c] = true
        for _, d := range dirs {
            nr, nc := r+d[0], c+d[1]
            if nr >= 0 && nr < m && nc >= 0 && nc < n && !seen[nr][nc] &&
                heights[nr][nc] >= heights[r][c] {
                dfs(nr, nc, seen)
            }
        }
    }
    for r := range m {
        dfs(r, 0, pac)
        dfs(r, n-1, atl)
    }
    for c := range n {
        dfs(0, c, pac)
        dfs(m-1, c, atl)
    }
    var res [][]int
    for r := range m {
        for c := range n {
            if pac[r][c] && atl[r][c] {
                res = append(res, []int{r, c})
            }
        }
    }
    return res
}

// Spiral Matrix: peel the outer ring, shrink the four bounds, repeat
func spiralOrder(matrix [][]int) []int {
    top, bottom, left, right := 0, len(matrix)-1, 0, len(matrix[0])-1
    res := make([]int, 0, len(matrix)*len(matrix[0]))
    for top <= bottom && left <= right {
        for c := left; c <= right; c++ {
            res = append(res, matrix[top][c])
        }
        top++
        for r := top; r <= bottom; r++ {
            res = append(res, matrix[r][right])
        }
        right--
        if top <= bottom {
            for c := right; c >= left; c-- {
                res = append(res, matrix[bottom][c])
            }
            bottom--
        }
        if left <= right {
            for r := bottom; r >= top; r-- {
                res = append(res, matrix[r][left])
            }
            left++
        }
    }
    return res
}

// Rotate Image 90 degrees clockwise in place: transpose, then reverse each row
func rotate(matrix [][]int) {
    n := len(matrix)
    for i := range n {
        for j := i + 1; j < n; j++ {
            matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
        }
    }
    for _, row := range matrix {
        slices.Reverse(row) // row shares memory with matrix, so this is in place
    }
}
```

Cost: O(m·n) time. Recursive flood fill uses O(m·n) stack in the worst case; Go grows goroutine stacks on demand, so this is fine up to about a million cells, and BFS avoids the question entirely.

### 20.23 Backtracking

Signals: "return all" combinations, subsets, permutations, partitions or placements, and small limits (n up to about 10 for permutations, about 20 for subsets). The full template and the classic problems are in Section 15; use this table to pick the right choice rule.

| Problem shape | Choice at each step | Section 15 model |
| --- | --- | --- |
| All subsets | Take or skip; or loop from a start index | `subsets` |
| Combinations of size k or with a target sum | Loop from a start index; pass i (reuse) or i+1 (no reuse) | `combine`, `combinationSum` |
| Permutations | Any unused element, tracked with `used []bool` | `permute` |
| Input has duplicates | Sort, then skip `nums[i] == nums[i-1]` at the same depth | Subsets II, Permutations II |
| Constrained placement | One row at a time, prune with sets of used columns and diagonals | `solveNQueens` |
| Paths in a grid | Four directions, mark before recursing, unmark after | `exist` |
| Cutting a string | Cut at every end index that keeps the piece valid | `partitionPalindromes` below |

```go
// Palindrome Partitioning
func partitionPalindromes(s string) [][]string {
    isPal := func(l, r int) bool {
        for l < r {
            if s[l] != s[r] {
                return false
            }
            l++
            r--
        }
        return true
    }
    var res [][]string
    var path []string
    var dfs func(start int)
    dfs = func(start int) {
        if start == len(s) {
            res = append(res, slices.Clone(path))
            return
        }
        for end := start; end < len(s); end++ {
            if isPal(start, end) {
                path = append(path, s[start:end+1])
                dfs(end + 1)
                path = path[:len(path)-1]
            }
        }
    }
    dfs(0)
    return res
}

// Letter Combinations of a Phone Number: a fixed-size buffer, one slot per depth
func letterCombinations(digits string) []string {
    if digits == "" {
        return nil
    }
    keys := [10]string{"", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"}
    var res []string
    buf := make([]byte, len(digits))
    var dfs func(i int)
    dfs = func(i int) {
        if i == len(digits) {
            res = append(res, string(buf)) // string() copies, so no clone is needed
            return
        }
        letters := keys[digits[i]-'0']
        for j := 0; j < len(letters); j++ {
            buf[i] = letters[j]
            dfs(i + 1)
        }
    }
    dfs(0)
    return res
}
```

Cost: exponential by nature (2ⁿ subsets, n! permutations, times the O(n) copy per answer). Pruning early is the only real speed-up.

### 20.24 Topological sort

Signals: prerequisites, dependencies, build or course order, "X must come before Y", or an order you must infer from sorted examples. Kahn's algorithm and the DFS version are in 18.2.

```go
// Alien Dictionary: each adjacent pair of words reveals at most one letter ordering
func alienOrder(words []string) string {
    adj := map[byte]map[byte]bool{}
    for _, w := range words {
        for i := 0; i < len(w); i++ {
            if adj[w[i]] == nil {
                adj[w[i]] = map[byte]bool{} // every letter becomes a node
            }
        }
    }
    indeg := map[byte]int{} // a missing key reads as 0
    for i := 0; i+1 < len(words); i++ {
        a, b := words[i], words[i+1]
        if len(a) > len(b) && strings.HasPrefix(a, b) {
            return "" // "abc" before "ab" is impossible
        }
        for j := 0; j < min(len(a), len(b)); j++ {
            if a[j] != b[j] {
                if !adj[a[j]][b[j]] {
                    adj[a[j]][b[j]] = true
                    indeg[b[j]]++
                }
                break // only the first difference carries information
            }
        }
    }
    var queue []byte
    for ch := range adj {
        if indeg[ch] == 0 {
            queue = append(queue, ch)
        }
    }
    var sb strings.Builder
    for len(queue) > 0 {
        ch := queue[0]
        queue = queue[1:]
        sb.WriteByte(ch)
        for nb := range adj[ch] {
            indeg[nb]--
            if indeg[nb] == 0 {
                queue = append(queue, nb)
            }
        }
    }
    if sb.Len() < len(adj) {
        return "" // a cycle: the examples contradict each other
    }
    return sb.String()
}
```

Cost: O(C) where C is the total number of characters.

### 20.25 Union-find

Signals: "are these connected", counting groups, merging items that share something (an email, a letter swap, an equation), edges arriving one by one, detecting the edge that closes a cycle, or Kruskal's MST. The `DSU` type is in 11.2; it tracks `count`, the number of groups.

```go
// Number of Provinces from an adjacency matrix
func findCircleNum(isConnected [][]int) int {
    n := len(isConnected)
    d := NewDSU(n)
    for i := range n {
        for j := i + 1; j < n; j++ {
            if isConnected[i][j] == 1 {
                d.Union(i, j)
            }
        }
    }
    return d.count
}

// Graph Valid Tree: exactly n-1 edges and none of them closes a cycle
func validTree(n int, edges [][]int) bool {
    if len(edges) != n-1 {
        return false
    }
    d := NewDSU(n)
    for _, e := range edges {
        if !d.Union(e[0], e[1]) {
            return false
        }
    }
    return true
}

// Accounts Merge: union accounts that share an email, then group emails by root
func accountsMerge(accounts [][]string) [][]string {
    d := NewDSU(len(accounts))
    owner := map[string]int{} // email -> first account that listed it
    for i, acc := range accounts {
        for _, email := range acc[1:] {
            if j, ok := owner[email]; ok {
                d.Union(i, j)
            } else {
                owner[email] = i
            }
        }
    }
    groups := map[int][]string{} // root account -> its emails
    for email, i := range owner {
        root := d.Find(i)
        groups[root] = append(groups[root], email)
    }
    res := make([][]string, 0, len(groups))
    for root, emails := range groups {
        slices.Sort(emails)
        res = append(res, append([]string{accounts[root][0]}, emails...))
    }
    return res
}
```

Cost: nearly O(1) per operation (inverse Ackermann) with path compression and union by size. Prefer DSU over DFS when edges keep arriving; prefer DFS or BFS when you also need paths or distances.

### 20.26 Trie

Signals: prefix queries, autocomplete, "starts with", searching many words at once (especially in a grid), replacing words by their shortest root, or maximizing XOR (a trie over bits). The general `Trie` is in 11.1.

```go
// Word Search II: walk the board and the trie together; prune when no word continues
type wordNode struct {
    next [26]*wordNode
    word string // non-empty on the node where a word ends
}

func findWords(board [][]byte, words []string) []string {
    root := &wordNode{}
    for _, w := range words {
        n := root
        for i := 0; i < len(w); i++ {
            c := w[i] - 'a'
            if n.next[c] == nil {
                n.next[c] = &wordNode{}
            }
            n = n.next[c]
        }
        n.word = w
    }
    rows, cols := len(board), len(board[0])
    var res []string
    var dfs func(r, c int, n *wordNode)
    dfs = func(r, c int, n *wordNode) {
        if r < 0 || r >= rows || c < 0 || c >= cols || board[r][c] == '#' {
            return
        }
        ch := board[r][c]
        n = n.next[ch-'a']
        if n == nil {
            return // no word goes this way
        }
        if n.word != "" {
            res = append(res, n.word)
            n.word = "" // report each word once
        }
        board[r][c] = '#' // mark visited
        for _, d := range dirs {
            dfs(r+d[0], c+d[1], n)
        }
        board[r][c] = ch // unmark
    }
    for r := range rows {
        for c := range cols {
            dfs(r, c, root)
        }
    }
    return res
}

// Maximum XOR of Two Numbers: a binary trie; at each bit try to go the opposite way
func findMaximumXOR(nums []int) int {
    type node struct{ child [2]*node }
    root := &node{}
    best := 0
    for _, v := range nums {
        n := root
        for b := 30; b >= 0; b-- { // insert v, highest bit first
            bit := v >> b & 1
            if n.child[bit] == nil {
                n.child[bit] = &node{}
            }
            n = n.child[bit]
        }
        n, x := root, 0
        for b := 30; b >= 0; b-- {
            bit := v >> b & 1
            if n.child[1-bit] != nil {
                x |= 1 << b // this bit of the XOR can be 1
                n = n.child[1-bit]
            } else {
                n = n.child[bit]
            }
        }
        best = max(best, x)
    }
    return best
}
```

Cost: O(L) per insert or lookup for a word of length L; the bit trie is O(31) per number.

### 20.27 Dynamic programming

Signals: "number of ways", "minimum or maximum cost", "is it possible", "longest or shortest" combined with choices that overlap, where a greedy counterexample exists. Section 16 has the method and the classic problems; this table maps clues to the family and its state.

| Clue | Family | State | Section 16 example |
| --- | --- | --- | --- |
| Steps along a line, each depends on the last few | Linear | `dp[i]` = answer for the prefix ending at i | Climbing stairs, house robber |
| Pick items under a capacity or target | Knapsack | `dp[c]` over capacity | 0/1 knapsack, coin change, partition |
| Two strings or arrays | Two sequences | `dp[i][j]` for prefixes `a[:i]`, `b[:j]` | LCS, edit distance |
| Moves right or down in a grid | Grid | `dp[r][c]` | Unique paths |
| Substrings, merging or bursting ranges | Interval | `dp[l][r]`, filled by increasing length | Palindromic subsequence, burst balloons |
| Holding, sold, cooldown, k transactions | State machine | `dp[i][state]` | Stock with cooldown |
| n up to about 20, visit everything | Bitmask | `dp[mask][last]` | TSP |
| Answer at a node depends on children | Tree DP | Return a small tuple from DFS | House robber III |
| Longest increasing chain | LIS | `dp[i]`, or patience tails with binary search | LIS |
| Paths on a grid or DAG without a clear order | Memoized DFS | `memo[node]` | Longest Increasing Path below |

```go
// Decode Ways: linear DP that only needs the last two values
func numDecodings(s string) int {
    prev2, prev1 := 0, 1 // dp[i-2], dp[i-1]; dp of the empty prefix is 1
    for i := 0; i < len(s); i++ {
        cur := 0
        if s[i] != '0' {
            cur = prev1 // s[i] as one letter
        }
        if i > 0 && s[i-1] != '0' {
            if two := (s[i-1]-'0')*10 + (s[i] - '0'); two <= 26 {
                cur += prev2 // s[i-1..i] as one letter
            }
        }
        prev2, prev1 = prev1, cur
    }
    return prev1
}

// Maximal Square: a square ending here is limited by its three neighbors
func maximalSquare(matrix [][]byte) int {
    m, n := len(matrix), len(matrix[0])
    dp := make([][]int, m+1) // dp[i][j] = side of the largest square ending at (i-1, j-1)
    for i := range dp {
        dp[i] = make([]int, n+1)
    }
    best := 0
    for i := 1; i <= m; i++ {
        for j := 1; j <= n; j++ {
            if matrix[i-1][j-1] == '1' {
                dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
                best = max(best, dp[i][j])
            }
        }
    }
    return best * best
}

// Longest Increasing Path in a Matrix: strictly increasing moves form a DAG, so memo DFS works
func longestIncreasingPath(matrix [][]int) int {
    m, n := len(matrix), len(matrix[0])
    memo := make([][]int, m) // 0 = not computed yet
    for i := range memo {
        memo[i] = make([]int, n)
    }
    var dfs func(r, c int) int
    dfs = func(r, c int) int {
        if memo[r][c] != 0 {
            return memo[r][c]
        }
        best := 1
        for _, d := range dirs {
            nr, nc := r+d[0], c+d[1]
            if nr >= 0 && nr < m && nc >= 0 && nc < n && matrix[nr][nc] > matrix[r][c] {
                best = max(best, 1+dfs(nr, nc))
            }
        }
        memo[r][c] = best
        return best
    }
    ans := 0
    for r := range m {
        for c := range n {
            ans = max(ans, dfs(r, c))
        }
    }
    return ans
}
```

Cost: number of states times the work per state, for example O(n) for Decode Ways and O(m·n) for the two grid problems.

### 20.28 Greedy

Signals: a locally best choice can be proven never to hurt (an exchange argument), intervals to keep or remove, reachability, or "minimum number of" something with a natural ordering. Section 17 covers the proofs, the sort-key table and the classic problems. The shortcut for intervals: sort by end to keep as many as possible, sort by start to merge. When a greedy idea fails on a small counterexample, switch to DP.

```go
// Boats to Save People: pair the heaviest remaining person with the lightest if they fit
func numRescueBoats(people []int, limit int) int {
    slices.Sort(people)
    boats := 0
    for l, r := 0, len(people)-1; l <= r; r-- {
        if people[l]+people[r] <= limit {
            l++ // the lightest rides along
        }
        boats++ // the heaviest always takes this boat
    }
    return boats
}
```

### 20.29 Bitwise XOR and bitmasks

Signals: "every element appears twice except one", O(1) extra space with no sorting, a missing number, toggling states, or subsets of at most about 20 items. XOR cancels equal pairs (`a ^ a == 0`), so the survivors are the answer; a bitmask stores a whole subset in one int. The problems are in 19.4 and the subset loops in 19.5.

### 20.30 Combined patterns

Hard problems usually stack two patterns. Recognizing the pair is often the whole trick.

| Combination | Typical problem |
| --- | --- |
| Sliding window + hashmap | Longest substring with at most k distinct characters |
| Prefix sum + hashmap | Subarray sum equals k, contiguous array |
| Binary search on the answer + greedy check | Split array largest sum, Koko eating bananas |
| Sorting + two pointers | 3Sum, boats to save people |
| Heap + greedy | Task scheduler, IPO, meeting rooms II |
| BFS + bitmask state | Shortest path visiting all nodes |
| DFS + memoization | Longest increasing path in a matrix, word break |
| Monotonic deque + DP | Jump game VI, constrained subsequence sum |
| Trie + backtracking | Word search II |
| Union-find + sorting | Kruskal MST, accounts merge by time |
| Binary search + DP | LIS in O(n log n) |

## 21. Pattern recognition cheat sheet

Use this section before you write any code: read the problem, find its strongest signal in the first table, and check the limits against the second. If two rows match, the problem probably combines both patterns (20.30).

### 21.1 Signal to pattern

| When the problem says or implies | Reach for | Section |
| --- | --- | --- |
| Sorted array, find a pair or triplet with a sum | Two pointers from both ends | 20.1 |
| Modify in place, "return the new length" | Read and write pointers | 20.2 |
| Linked list cycle or middle; duplicate in 1..n with O(1) space | Fast and slow pointers | 20.3 |
| Every subarray or substring of length k | Fixed sliding window | 20.4 |
| Longest or shortest substring or subarray with a condition (no negatives) | Variable sliding window | 20.5 |
| Count subarrays with sum k, or negatives are present | Prefix sum + hashmap | 20.6 |
| Many range-sum queries on a fixed array | Prefix sums (1D or 2D) | 20.6 |
| Range queries mixed with point or range updates | Fenwick tree or segment tree | 11.3, 11.4 |
| Many range updates, one read at the end | Difference array | 20.9 |
| Most overlapping intervals, rooms needed | Sweep line or min-heap of end times | 20.9, 17 |
| Seen before, counts, duplicates, anagrams | Hashmap, set or `[26]int` | 20.7, 6 |
| Best contiguous sum or product | Kadane | 20.8 |
| n numbers in the range 1..n, find missing or duplicated | Cyclic sort | 20.10 |
| Merge, insert or intersect intervals | Sort by start, then sweep | 20.11 |
| Keep the most non-overlapping intervals | Greedy, sort by end | 17 |
| Reverse part of a list, k-groups, reorder | In-place reversal with a dummy node | 20.12 |
| Next or previous greater or smaller element, histogram | Monotonic stack | 20.13 |
| Maximum or minimum of every window | Monotonic deque | 20.14 |
| Matching brackets, nested structures, undo | Stack | 8 |
| k largest, smallest, most frequent or closest | Size-k heap, or quickselect | 20.15, 13 |
| Merge k sorted lists or arrays | K-way merge with a heap | 20.16 |
| Median of a stream | Two heaps | 20.17 |
| Sorted or rotated input, O(log n) required | Binary search | 14, 20.18 |
| "Minimize the maximum", "smallest speed or capacity that works" | Binary search on the answer | 14, 20.18 |
| Path sums, height, anything computed from subtrees | Tree DFS | 10, 20.19 |
| Level by level, side views, minimum depth | Tree BFS | 20.20 |
| Fewest steps, unweighted shortest path | BFS | 12.3 |
| Spreading from several places at once | Multi-source BFS | 20.21 |
| Islands, regions, cells connected to the border | Flood fill | 12.5, 20.22 |
| Groups, "are they connected", merging by shared items | Union-find | 11.2, 20.25 |
| Prerequisites, build order, dependencies | Topological sort | 18.2, 20.24 |
| Weighted shortest path, non-negative weights | Dijkstra | 18.3 |
| Negative weights, or at most k edges | Bellman-Ford | 18.5 |
| Connect everything at minimum total cost | Minimum spanning tree | 18.7 |
| Return all subsets, combinations, permutations or placements | Backtracking | 15, 20.23 |
| Prefix lookups, autocomplete, many words in a grid | Trie | 11.1, 20.26 |
| Number of ways, minimum cost, "is it possible" with overlapping choices | Dynamic programming | 16, 20.27 |
| Two strings compared character by character | Two-sequence DP | 16 |
| A locally best move can never hurt | Greedy | 17 |
| Everything appears twice except one, O(1) space | XOR | 19.4 |
| Up to about 20 items and "visit or use all of them" | Bitmask DP | 16, 19.5 |
| "Return the answer modulo 10⁹+7" | Counting DP or combinatorics with modular arithmetic | 19.7 |
| Design a cache with O(1) get and put | Hashmap + doubly linked list | 7 |
| kth smallest or sorted output from a BST | Inorder traversal | 10 |

### 21.2 Limits to complexity

The input limits tell you how fast the answer must be. Assume about 10⁸ simple operations per second (Section 3).

| Largest n | Target complexity | Patterns that fit |
| --- | --- | --- |
| 10 to 12 | O(n!) | Permutation backtracking |
| 20 to 25 | O(2ⁿ) or O(2ⁿ·n) | Subsets, bitmask DP, meet in the middle |
| 100 to 500 | O(n³) | Floyd-Warshall, interval DP |
| 2,000 to 5,000 | O(n²) | Two-sequence DP, all pairs, O(n²) LIS |
| 10⁵ to 10⁶ | O(n log n) or O(n) | Sorting, heaps, binary search, two pointers, sliding window, prefix sums, hashing, monotonic stack |
| Values up to 10⁹ (the answer range is huge) | An extra O(log V) factor | Binary search on the answer |
| n up to 10¹⁸ | O(log n) or O(1) | Math, fast power, binary search |

### 21.3 First patterns to try, by input type

| Input | Try first |
| --- | --- |
| Array or string | Two pointers, sliding window, prefix sums, hashing, sorting, binary search |
| Linked list | Dummy node, fast and slow pointers, in-place reversal |
| Binary tree | DFS (top-down or bottom-up), BFS by level; inorder if it is a BST |
| Graph given as edges | Build an adjacency list, then BFS, DFS, topological sort, DSU or Dijkstra |
| Grid | Flood fill, BFS for distances, grid DP for counting paths |
| Intervals | Sort, then merge, greedy by end, sweep line or heap |
| Stream of values | Heaps, two heaps, monotonic deque, DSU |
| List of words | Trie, hashing with a canonical key, BFS over a word graph |

### 21.4 Six questions that find the pattern

1. What is the input shape? Use 21.3 to narrow the candidates.
2. Is it sorted, or does sorting lose nothing? Sorting unlocks two pointers, binary search and most greedy solutions.
3. Is the answer a contiguous piece? Then think sliding window, prefix sums, Kadane or a monotonic structure.
4. Do they want all answers or one best answer? All answers means backtracking; a count or a best value means DP or greedy.
5. What do the limits allow? Check 21.2 before investing in an approach that is too slow.
6. What does the brute force repeat? The repeated work names the fix: repeated lookups need a hashmap, repeated range sums need prefix sums, repeated min or max needs a heap or deque, and repeated subproblems need memoization.

## 22. Go gotchas and performance

Most wrong answers in Go come from a handful of language behaviors, not from the algorithm. Learn these once and you will stop losing time to them.

### 22.1 Correctness gotchas

| Gotcha | What goes wrong | Fix |
| --- | --- | --- |
| Slices share memory | `b := a[1:3]` or an `append` with spare capacity writes into `a` | `slices.Clone(a)`, or cap the slice with `a[1:3:3]` |
| Saving a backtracking path | Every saved result shows the last path | `res = append(res, slices.Clone(path))` |
| `range` copies elements | `for _, v := range items { v.x = 1 }` changes a copy | Index instead: `items[i].x = 1` |
| Map iteration order | Random on every run, so output order changes | Collect the keys, `slices.Sort`, then iterate |
| Writing to a nil map | Panics (reading one is fine) | `m := map[K]V{}` or `make(map[K]V)` |
| Struct values in a map | `m[k].field = 1` does not compile | Store pointers, or copy out, modify, write back |
| Integer overflow | Wraps silently, no panic | `int` is 64-bit; use `math.MaxInt/2` as infinity; `lo + (hi-lo)/2` |
| Division and remainder | `-7/2 == -3`, `-7%3 == -1` | `((a%m)+m)%m`; `floorDiv` in 19.8 |
| `math.Abs`, `math.Max`, `math.Min` | Take and return `float64` | Built-in `min` and `max` (Go 1.21+); your own `abs` |
| Indexing a string | `s[i]` is a byte and `len(s)` counts bytes, not characters | `[]rune(s)` for Unicode; bytes are fine for ASCII problems |
| `string(65)` | Gives "A", not "65" | `strconv.Itoa(65)` |
| Strings are immutable | `s[i] = 'x'` does not compile; `s += ...` in a loop is O(n²) | Convert to `[]byte`, or use `strings.Builder` |
| Comparing slices | `a == b` does not compile for slices | `slices.Equal(a, b)`; arrays such as `[26]int` compare with `==` |
| Shadowing with `:=` | An inner `x := ...` hides the outer `x`, which never changes | Use `=` when the variable already exists |
| `heap.Pop` result | Returns `any` | Assert the type: `heap.Pop(h).(int)` |
| Calling `h.Push(x)` directly | Skips the heap reordering | Always go through `heap.Push(h, x)` and `heap.Pop(h)` |
| Heap methods on a value receiver | `Push` and `Pop` change nothing | Pointer receivers: `func (h *IntHeap) Push(x any)` |
| Sort comparator | A `less` using `<=` or an inconsistent `cmp` corrupts the order | Strict `<` for `less`; `cmp.Compare` for `SortFunc` |
| Recursive closure | `dfs := func() { dfs() }` does not compile | Declare first: `var dfs func(); dfs = func() { ... }` |
| `defer` inside a loop | Runs when the function returns, not each iteration | Move the loop body into its own function |
| Unused variables and imports | Compile error | Delete them, or assign to `_` while debugging |
| Older Go versions | `for i := range n` needs 1.22, `min`/`max` need 1.21, the `slices` package needs 1.21 | Check the judge's Go version; fall back to classic loops and helpers |

The first two cause the most bugs, so here they are in code.

```go
// 1. A sub-slice shares the backing array
a := []int{1, 2, 3, 4}
b := a[:2]        // b = [1 2], cap 4
b = append(b, 99) // fits in the capacity, so it overwrites a[2]
fmt.Println(a)    // [1 2 99 4]

c := a[:2:2]      // full slice expression: cap is 2
c = append(c, 7)  // no room, so append copies; a is untouched

// 2. Backtracking without a clone
var res [][]int
path := []int{}
res = append(res, path)               // WRONG: aliases path, later changes leak in
res = append(res, slices.Clone(path)) // RIGHT: an independent snapshot

// 3. Loop variables are per-iteration since Go 1.22, so closures capture each value
var fs []func() int
for i := range 3 {
    fs = append(fs, func() int { return i }) // 0, 1, 2 (before 1.22: 3, 3, 3)
}
```

### 22.2 Performance habits

| Habit | Why it helps |
| --- | --- |
| Preallocate: `make([]int, 0, n)`, `make(map[int]int, n)` | Avoids repeated growth and rehashing |
| Arrays over maps for small key ranges: `[26]int`, `[]bool` for visited | Direct indexing is several times faster than hashing |
| Struct or array map keys: `map[[2]int]bool` | Much faster than building keys with `fmt.Sprintf("%d,%d", r, c)` |
| Encode a cell as one int: `id := r*n + c` | One slice instead of a 2D structure, and cheap keys |
| `strings.Builder`, `strconv.Itoa`, `strconv.AppendInt` | `fmt` is slow in hot loops; `+=` on strings is O(n²) |
| Reuse buffers: `buf = buf[:0]`, `clear(m)` | Keeps the memory and skips reallocation |
| Generic `Heap[T]` (9.5) over `container/heap` in hot loops | No interface boxing or type assertions |
| `slices.Sort` over `sort.Slice` | Generic code avoids reflection-style swaps |
| Iterative BFS instead of very deep recursion | Recursion is fine for most inputs because Go stacks grow, but a million frames is slow |
| Buffered I/O for large inputs | `fmt.Scan` without buffering can time out on 10⁶ numbers |

For competitive judges that read from standard input, this reader is roughly ten times faster than `fmt.Fscan`.

```go
var reader = bufio.NewReaderSize(os.Stdin, 1<<20)
var writer = bufio.NewWriterSize(os.Stdout, 1<<20)

// readInt parses the next integer; call it only as many times as there are numbers
func readInt() int {
    c, _ := reader.ReadByte()
    for c != '-' && (c < '0' || c > '9') {
        c, _ = reader.ReadByte() // skip spaces and newlines
    }
    neg := c == '-'
    if neg {
        c, _ = reader.ReadByte()
    }
    n := 0
    for c >= '0' && c <= '9' {
        n = n*10 + int(c-'0')
        c, _ = reader.ReadByte()
    }
    if neg {
        return -n
    }
    return n
}

func main() {
    defer writer.Flush() // without this, buffered output is lost
    n := readInt()
    sum := 0
    for range n {
        sum += readInt()
    }
    writer.WriteString(strconv.Itoa(sum))
    writer.WriteByte('\n')
}
```

### 22.3 Before you submit

- [ ] Empty input, a single element, and two elements
- [ ] All equal values, duplicates, and already sorted or reverse-sorted input
- [ ] Negative numbers, zero, and the largest values the limits allow (overflow)
- [ ] Off-by-one at both ends of every loop and binary search
- [ ] 0-indexed versus 1-indexed input and output
- [ ] Output order: sorted when required, not map order
- [ ] Results cloned in backtracking, and inputs you mutated that the caller still needs
- [ ] Complexity checked against the limits (21.2)

## 23. Practice roadmap

Twelve weeks at about 10 hours a week takes you from Go syntax to recognizing every pattern in this guide. Each week, read the listed sections, type each template once from memory, then solve the problems without looking at the code.

### 23.1 Twelve-week plan

| Week | Focus | Read | Problems (LeetCode names) |
| --- | --- | --- | --- |
| 1 | Go syntax, complexity, arrays, strings | 2–5 | Two Sum, Valid Anagram, Contains Duplicate, Best Time to Buy and Sell Stock, Valid Palindrome, Reverse String |
| 2 | Hashing, two pointers, prefix sums | 6, 20.1–20.2, 20.6–20.7 | Group Anagrams, Two Sum II, 3Sum, Container With Most Water, Subarray Sum Equals K, Product of Array Except Self, Longest Consecutive Sequence |
| 3 | Sliding window, Kadane, intervals, cyclic sort | 20.4–20.5, 20.8–20.11 | Longest Substring Without Repeating Characters, Permutation in String, Minimum Window Substring, Maximum Subarray, Merge Intervals, Insert Interval, First Missing Positive |
| 4 | Linked lists, stacks, queues, monotonic structures | 7, 8, 20.3, 20.12–20.14 | Reverse Linked List, Linked List Cycle II, Merge Two Sorted Lists, Reorder List, LRU Cache, Valid Parentheses, Daily Temperatures, Largest Rectangle in Histogram, Sliding Window Maximum |
| 5 | Sorting and binary search | 13, 14, 20.18 | Sort Colors, Kth Largest Element in an Array, Binary Search, Search in Rotated Sorted Array, Find Minimum in Rotated Sorted Array, Koko Eating Bananas, Split Array Largest Sum, Time Based Key-Value Store |
| 6 | Heaps | 9, 20.15–20.17 | Top K Frequent Elements, K Closest Points to Origin, Merge k Sorted Lists, Find Median from Data Stream, Task Scheduler, IPO |
| 7 | Trees | 10, 20.19–20.20 | Maximum Depth, Invert Binary Tree, Same Tree, Level Order Traversal, Right Side View, Validate BST, Kth Smallest in a BST, Lowest Common Ancestor, Binary Tree Maximum Path Sum, Construct Binary Tree from Preorder and Inorder |
| 8 | Graphs: BFS, DFS, grids, topological sort | 12, 18.2, 20.21–20.22, 20.24 | Number of Islands, Max Area of Island, Rotting Oranges, Pacific Atlantic Water Flow, Clone Graph, Course Schedule II, Word Ladder, Alien Dictionary |
| 9 | Weighted graphs, union-find, tries | 11, 18, 20.25–20.26 | Network Delay Time, Cheapest Flights Within K Stops, Path With Minimum Effort, Min Cost to Connect All Points, Redundant Connection, Accounts Merge, Implement Trie, Word Search II |
| 10 | Backtracking, bits, math | 15, 19, 20.23, 20.29 | Subsets, Permutations, Combination Sum, N-Queens, Word Search, Palindrome Partitioning, Single Number, Counting Bits, Pow(x, n), Count Primes |
| 11 | DP part 1: linear, knapsack, grids | 16, 20.27 | Climbing Stairs, House Robber, House Robber II, Coin Change, Coin Change II, Partition Equal Subset Sum, Word Break, Decode Ways, Unique Paths, Longest Increasing Subsequence |
| 12 | DP part 2, greedy, mixed review | 16, 17, 20.28, 21 | Longest Common Subsequence, Edit Distance, Longest Palindromic Substring, Best Time to Buy and Sell Stock with Cooldown, Burst Balloons, Jump Game II, Gas Station, Non-overlapping Intervals, then 10 random mediums under a timer |

### 23.2 How to work each problem

1. Read the statement and constraints, then name the pattern with Section 21 before writing code.
2. Give yourself 25 minutes for a medium. If you are stuck, reread only the matching pattern's signals and idea, not its code, and try again.
3. After solving, read one strong solution and note what it did better.
4. Log the problem below with its pattern and the one insight that unlocked it.
5. Re-solve it from scratch a week later. If you cannot, it goes back into next week's list.

### 23.3 Problem log

Add a row for every problem you solve. The pattern and insight columns are what you review before an interview.

| Problem | Pattern | Key insight | Time taken | Re-solved |
| --- | --- | --- | --- | --- |
| Two Sum | Hashing (20.7) | Look up target − v before storing v | 10 min |  |

### 23.4 Milestones

- [ ] Write BFS, DFS, `firstTrue` binary search and a heap from memory in under 5 minutes each
- [ ] Name the pattern for 20 random problems from their statements alone
- [ ] Solve an unseen easy in 15 minutes and an unseen medium in 30 minutes
- [ ] Finish every problem in the twelve-week table
- [ ] Re-solve your 20 hardest problems a week after the first attempt
- [ ] Complete 5 timed mock sessions of two problems in 60 minutes, explaining your reasoning out loud
