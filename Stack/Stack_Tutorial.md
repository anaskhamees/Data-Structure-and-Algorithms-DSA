# Stack — Complete Tutorial

A complete guide covering **Array-based Stack**, **Linked-List-based Stack**, and **Stack in STL (`std::stack`)**, with fully solved practice problems — built in the same style as the Linked List tutorial.

---

## Table of Contents
1. [Why Stack?](#1-why-stack)
2. [LIFO — The Core Idea](#2-lifo--the-core-idea)
3. [Array-based Stack — Basics](#3-array-based-stack--basics)
4. [Operations on Array-based Stack](#4-operations-on-array-based-stack)
5. [Linked-List-based Stack — Full Operations](#5-linked-list-based-stack--full-operations)
6. [Array vs Linked-List Stack](#6-array-vs-linked-list-stack)
7. [`std::stack` — Full STL Tutorial](#7-stdstack--full-stl-tutorial)
8. [Complexity Summary](#8-complexity-summary)
9. [Practice Examples — Solved in Detail](#9-practice-examples--solved-in-detail)
10. [Quick Reference](#10-quick-reference)

---

## 1. Why Stack?

A **stack** is a linear data structure that stores elements the way a physical stack of plates works: you can only add a plate to the **top**, and you can only remove a plate from the **top**. You can never grab a plate from the middle or bottom without first removing everything above it.

### Real-world / CS use cases
- **Function call stack** — every function call pushes a stack frame; returning pops it.
- **Undo/Redo** in editors — each action pushed onto a stack; undo pops it.
- **Browser back button** — visited pages pushed; back pops.
- **Expression evaluation** — parsing/evaluating arithmetic expressions, matching brackets.
- **Backtracking algorithms** — maze solving, DFS, N-Queens.
- **Compilers** — syntax parsing, balancing `{}`, `()`, `[]`.

### When to use a Stack vs other structures

| Use **Stack** when... | Use something else when... |
|---|---|
| You only ever need the **most recently added** item | You need the oldest item first → use a **Queue** |
| Order of processing is "last in, first out" | You need random access by index → use an **Array** |
| You need to reverse something, or track nested structure | You need fast search → use a **Hash Table / Tree** |

---

## 2. LIFO — The Core Idea

**LIFO = Last In, First Out.** The last element pushed is the first one popped.

```
Pushing 10, 20, 30 in order:

push(10)        push(20)        push(30)
┌────┐          ┌────┐          ┌────┐
│    │          │    │          │ 30 │ ◀── top
├────┤          ├────┤          ├────┤
│    │          │ 20 │ ◀── top  │ 20 │
├────┤          ├────┤          ├────┤
│ 10 │ ◀── top  │ 10 │          │ 10 │
└────┘          └────┘          └────┘

Now pop() three times:
pop() → 30 (removed first — it was pushed LAST)
pop() → 20
pop() → 10
```

Only two ends of interaction exist:
- **push** — add to the top
- **pop** — remove from the top
- (**peek/top** — look at the top without removing it, is the 3rd essential operation)

---

## 3. Array-based Stack — Basics

### What is an Array-based Stack?
A stack implemented on top of a **dynamically allocated array**, with `capacity` storing how big that array is and `top` tracking the index of the current top element.

```
arr (heap-allocated):  [ 10 | 20 | 30 |    |    ]
Index:                    0    1    2    3    4
                                    ▲
                                  top = 2  (index of last pushed element)
capacity = 5

If top == -1             → stack is EMPTY
If top == capacity - 1   → stack is FULL
```

### Stack Structure (C)

```c
typedef struct
{
    int *arr;     // array to store elements
    int capacity; // maximum size of stack
    int top;      // index of top element
} Stack;
```

> Using a **pointer** (`int *arr`) instead of a fixed-size array (`int arr[MAX_SIZE]`) means the stack's capacity is decided **at runtime** — via `malloc` — rather than baked in at compile time. This is closer to how real libraries (including the container behind `std::stack`) work, and it also sets up section 9.8, where the array is **resized** on demand.

### Creating / Initializing a Stack

**Steps:**
1. Allocate an array of the requested `capacity` on the heap with `malloc`.
2. Store that `capacity` in the struct (needed later to detect "full").
3. Set `top = -1` (empty stack — no elements yet).

```c
Stack* createStack(int capacity) {
    Stack* s = (Stack*) malloc(sizeof(Stack));
    s->arr = (int*) malloc(capacity * sizeof(int));
    s->capacity = capacity;
    s->top = -1;
    return s;
}
```

```
createStack(5):
┌─────────────────────────────┐
│ arr ──▶ [ ?, ?, ?, ?, ? ]    │   (5 ints allocated on the heap)
│ capacity: 5                   │
│ top: -1                       │  ◀── nothing pushed yet
└─────────────────────────────┘
```

### Checking Empty / Full

```c
int isEmpty(Stack* s) {
    return s->top == -1;
}

int isFull(Stack* s) {
    return s->top == s->capacity - 1;
}
```

### Freeing the Stack

Since the array (and the `Stack` struct itself, if heap-allocated) was `malloc`'d, it must be freed when done:

```c
void freeStack(Stack* s) {
    free(s->arr);   // free the underlying array first
    free(s);         // then the struct itself
}
```

---

## 4. Operations on Array-based Stack

Example stack used below (`capacity = 5`):

```
arr:  [ 8 | 3 | 7 |   |   ]
        0   1   2   3   4
                ▲
              top=2
```

### 4.1 Push

**Steps:**
1. Check if the stack is full. If so, report overflow and stop.
2. Increment `top`.
3. Store the new value at `arr[top]`.

```
Before: [8|3|7| | ]   top=2
push(9):
  top++  → top=3
  arr[3] = 9
After:  [8|3|7|9| ]   top=3
```

```c
void push(Stack* s, int value) {
    if (isFull(s)) {
        printf("Stack Overflow\n");
        return;
    }
    s->top++;
    s->arr[s->top] = value;
}
```
**Time:** `O(1)` | **Space:** `O(1)`

### 4.2 Pop

**Steps:**
1. Check if the stack is empty. If so, report underflow and stop (or return a sentinel/error code).
2. Read the value at `arr[top]` (so it can be returned).
3. Decrement `top` (this "removes" the element — we don't need to actually erase the array slot, it'll just be overwritten next push).
4. Return the saved value.

```
Before: [8|3|7|9| ]   top=3
pop():
  value = arr[3] = 9
  top--  → top=2
After:  [8|3|7|9| ]   top=2   (9 is "gone" logically, though still in memory)
Returns: 9
```

```c
int pop(Stack* s) {
    if (isEmpty(s)) {
        printf("Stack Underflow\n");
        return -1;   // sentinel for error (or use a bool "success" out-param in real code)
    }
    int value = s->arr[s->top];
    s->top--;
    return value;
}
```
**Time:** `O(1)` | **Space:** `O(1)`

### 4.3 Peek / Top

**Steps:**
1. Check if the stack is empty. If so, report error.
2. Return `arr[top]` **without** modifying `top`.

```c
int peek(Stack* s) {
    if (isEmpty(s)) {
        printf("Stack is empty\n");
        return -1;
    }
    return s->arr[s->top];
}
```
**Time:** `O(1)` | **Space:** `O(1)`

### 4.4 isEmpty / isFull
Already shown in section 3 — both `O(1)`, just comparisons on `top`.

### 4.5 Size

**Steps:**
1. Since `top` is the index of the last element (0-based) and starts at -1 when empty, the count of elements is simply `top + 1`.

```c
int size(Stack* s) {
    return s->top + 1;
}
```

### 4.6 Display (top to bottom)

**Steps:**
1. If empty, print a message and stop.
2. Walk `i` from `top` down to `0`, printing `arr[i]` at each step (top element printed first, since that's the "active" end).

```
[8|3|7|9]  top=3
display:  9 7 3 8   (top-first order)
```

```c
void display(Stack* s) {
    if (isEmpty(s)) {
        printf("Stack is empty\n");
        return;
    }
    for (int i = s->top; i >= 0; i--)
        printf("%d ", s->arr[i]);
}
```
**Time:** `O(n)` | **Space:** `O(1)`

### 4.7 Full Array Stack Example (C)

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct
{
    int *arr;     // array to store elements
    int capacity; // maximum size of stack
    int top;      // index of top element
} Stack;

Stack* createStack(int capacity) {
    Stack* s = (Stack*) malloc(sizeof(Stack));
    s->arr = (int*) malloc(capacity * sizeof(int));
    s->capacity = capacity;
    s->top = -1;
    return s;
}

int isEmpty(Stack* s) { return s->top == -1; }
int isFull(Stack* s)  { return s->top == s->capacity - 1; }

void push(Stack* s, int value) {
    if (isFull(s)) { printf("Overflow\n"); return; }
    s->arr[++s->top] = value;
}

int pop(Stack* s) {
    if (isEmpty(s)) { printf("Underflow\n"); return -1; }
    return s->arr[s->top--];
}

int peek(Stack* s) {
    if (isEmpty(s)) { printf("Empty\n"); return -1; }
    return s->arr[s->top];
}

void freeStack(Stack* s) {
    free(s->arr);
    free(s);
}

int main() {
    Stack* s = createStack(5);
    push(s, 10);
    push(s, 20);
    push(s, 30);
    printf("Top: %d\n", peek(s));   // 30
    printf("Pop: %d\n", pop(s));    // 30
    printf("Pop: %d\n", pop(s));    // 20
    printf("Top: %d\n", peek(s));   // 10
    freeStack(s);
}
```

### 4.8 The Overflow Problem with a Fixed-Capacity Array

```
Even though the array is heap-allocated, its SIZE was fixed
at createStack(capacity) time — it has a HARD capacity limit.

[8|3|7|9|2]   top=4  (capacity=5, FULL)
push(6)?  → Stack Overflow! No room left — unlike a linked list,
            which can keep growing as long as memory is available.

(Section 9.8 shows how to lift this limit: reallocate to a
 bigger array automatically instead of reporting overflow.)
```
> This is the main motivation for building a stack on a **linked list** instead (section 5) — no fixed capacity, grows one node at a time.

---

## 5. Linked-List-based Stack — Full Operations

A stack can also be built on top of a **singly linked list**, where **the head of the list IS the top of the stack**. This removes the fixed-capacity problem entirely.

```
top/head
   │
   ▼
┌────┬──────┐    ┌────┬──────┐    ┌────┬──────┐
│ 30 │  ●───┼───▶│ 20 │  ●───┼───▶│ 10 │ NULL │
└────┴──────┘    └────┴──────┘    └────┴──────┘
 (most recently        ...            (first
  pushed = top)                       pushed = bottom)
```

**Why insert/remove at the HEAD, not the tail?** Inserting/removing at the head of a singly linked list is `O(1)` (section 4.9(A)/4.10 of the Linked List tutorial). Inserting/removing at the tail would require traversing the whole list (`O(n)`) unless we kept an extra tail pointer — and even then, removing from the tail of a *singly* linked list is still `O(n)` because you'd need the node *before* the tail. So the head is the natural, efficient choice for the stack's "top."

### Node Structure

```c
typedef struct Node {
    int data;
    struct Node* next;
} Node;

typedef struct {
    Node* top;    // top == NULL means empty stack
} Stack;
```

### 5.1 Initialize

**Steps:**
1. Set `top` to `NULL` — an empty stack has no nodes.

```c
void initStack(Stack* s) {
    s->top = NULL;
}
```

### 5.2 isEmpty

**Steps:**
1. The stack is empty exactly when `top == NULL`.

```c
int isEmpty(Stack* s) {
    return s->top == NULL;
}
```

> **No `isFull()` needed** — a linked-list stack has no fixed capacity (it's only bounded by available memory).

### 5.3 Push

**Steps:**
1. Create a new node holding `value`.
2. Point the new node's `next` to the current `top` (so it connects to the rest of the stack).
3. Update `top` to the new node (it's now the top of the stack).

This is *identical* to "insert at head" from the Linked List tutorial.

```
Before:  top → [20] → [10] → NULL
push(30):
  newNode[30]->next = top (20)
  top = newNode[30]
After:   top → [30] → [20] → [10] → NULL
```

```c
void push(Stack* s, int value) {
    Node* newNode = (Node*) malloc(sizeof(Node));
    newNode->data = value;
    newNode->next = s->top;
    s->top = newNode;
}
```
**Time:** `O(1)` | **Space:** `O(1)` per push

### 5.4 Pop

**Steps:**
1. Check if the stack is empty. If so, report underflow and stop.
2. Save the current `top` node in a temporary pointer.
3. Move `top` forward to `top->next` (the new top).
4. Read the value from the saved node (so it can be returned).
5. Free the saved node.
6. Return the value.

```
Before:  top → [30] → [20] → [10] → NULL
pop():
  temp = top (30)
  top = top->next (20)
  value = temp->data (30)
  free(temp)
After:   top → [20] → [10] → NULL
Returns: 30
```

```c
int pop(Stack* s) {
    if (isEmpty(s)) {
        printf("Stack Underflow\n");
        return -1;
    }
    Node* temp = s->top;
    int value = temp->data;
    s->top = temp->next;
    free(temp);
    return value;
}
```
**Time:** `O(1)` | **Space:** `O(1)`

### 5.5 Peek / Top

**Steps:**
1. Check if the stack is empty. If so, report error.
2. Return `top->data` without modifying anything.

```c
int peek(Stack* s) {
    if (isEmpty(s)) {
        printf("Stack is empty\n");
        return -1;
    }
    return s->top->data;
}
```
**Time:** `O(1)` | **Space:** `O(1)`

### 5.6 Size

**Steps:**
1. Start a `count` at 0.
2. Walk from `top` to `NULL`, incrementing `count` at each node.
3. Return `count`.

> Unlike the array version (`top + 1` in `O(1)`), a linked-list stack has no built-in counter, so size takes `O(n)` **unless you maintain a separate counter field that you increment/decrement on every push/pop** (the recommended approach in practice — see 5.9).

```c
int size(Stack* s) {
    int count = 0;
    Node* temp = s->top;
    while (temp != NULL) {
        count++;
        temp = temp->next;
    }
    return count;
}
```
**Time:** `O(n)` (or `O(1)` with a maintained counter) | **Space:** `O(1)`

### 5.7 Display (top to bottom)

**Steps:**
1. Start `temp` at `top`.
2. While `temp != NULL`: print `temp->data`, move `temp` forward.
3. Stops automatically at `NULL` — no need to track an index like the array version.

```c
void display(Stack* s) {
    if (isEmpty(s)) {
        printf("Stack is empty\n");
        return;
    }
    Node* temp = s->top;
    while (temp != NULL) {
        printf("%d ", temp->data);
        temp = temp->next;
    }
}
```
**Time:** `O(n)` | **Space:** `O(1)`

### 5.8 Free the Whole Stack

**Steps:** identical to freeing any singly linked list — keep popping (or freeing node by node) until `top` is `NULL`.

```c
void freeStack(Stack* s) {
    Node* temp;
    while (s->top != NULL) {
        temp = s->top;
        s->top = s->top->next;
        free(temp);
    }
}
```
**Time:** `O(n)` | **Space:** `O(1)`

### 5.9 Better Version: Maintain a `size` Field (O(1) size)

**Steps:**
1. Add a `count` field to the `Stack` struct.
2. On every `push`, increment `count`.
3. On every `pop`, decrement `count`.
4. `size()` just returns `count` directly — `O(1)`.

```c
typedef struct {
    Node* top;
    int count;
} Stack;

void push(Stack* s, int value) {
    Node* newNode = (Node*) malloc(sizeof(Node));
    newNode->data = value;
    newNode->next = s->top;
    s->top = newNode;
    s->count++;                 // <-- track size
}

int pop(Stack* s) {
    if (isEmpty(s)) { printf("Underflow\n"); return -1; }
    Node* temp = s->top;
    int value = temp->data;
    s->top = temp->next;
    free(temp);
    s->count--;                 // <-- track size
    return value;
}

int size(Stack* s) { return s->count; }   // O(1) now
```

### 5.10 Full Linked-List Stack Example (C)

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct Node {
    int data;
    struct Node* next;
} Node;

typedef struct {
    Node* top;
    int count;
} Stack;

void initStack(Stack* s)  { s->top = NULL; s->count = 0; }
int isEmpty(Stack* s)     { return s->top == NULL; }
int size(Stack* s)        { return s->count; }

void push(Stack* s, int value) {
    Node* newNode = (Node*) malloc(sizeof(Node));
    newNode->data = value;
    newNode->next = s->top;
    s->top = newNode;
    s->count++;
}

int pop(Stack* s) {
    if (isEmpty(s)) { printf("Underflow\n"); return -1; }
    Node* temp = s->top;
    int value = temp->data;
    s->top = temp->next;
    free(temp);
    s->count--;
    return value;
}

int peek(Stack* s) {
    if (isEmpty(s)) { printf("Empty\n"); return -1; }
    return s->top->data;
}

int main() {
    Stack s;
    initStack(&s);
    push(&s, 10);
    push(&s, 20);
    push(&s, 30);
    printf("Top: %d, Size: %d\n", peek(&s), size(&s));  // 30, 3
    printf("Pop: %d\n", pop(&s));                        // 30
    printf("Pop: %d\n", pop(&s));                        // 20
    printf("Size: %d\n", size(&s));                      // 1
}
```

---

## 6. Array vs Linked-List Stack

```
Array Stack:                      Linked-List Stack:
┌───┬───┬───┬───┬───┐             top → [30] → [20] → [10] → NULL
│ 10│ 20│ 30│   │   │
└───┴───┴───┴───┴───┘             Grows one malloc'd node at a time —
  top=2, capacity=5 (fixed at createStack)   no fixed upper bound (until memory runs out)
```

| Feature | Array-based | Linked-List-based |
|---|---|---|
| Max capacity | Fixed at creation (can resize with realloc, at a cost) | Dynamic — grows/shrinks per element |
| Memory layout | Contiguous (cache-friendly) | Scattered (extra pointer overhead per node) |
| Push/Pop time | `O(1)` | `O(1)` |
| Extra memory per element | 0 | 1 pointer (`next`) |
| Overflow possible? | Yes, once full | Only if system runs out of memory |
| `size()` | `O(1)` (`top + 1`) | `O(1)` with a counter field, else `O(n)` |

---

## 7. `std::stack` — Full STL Tutorial

`std::stack` (from `<stack>`) is a **container adapter** in C++ — it doesn't implement its own storage; instead it wraps another container (by default `std::deque`) and restricts access to LIFO-only operations (`push`, `pop`, `top`). This mirrors exactly what we built by hand in sections 3–5, except STL handles the underlying storage for you.

### 7.1 Declaring a `std::stack`

```cpp
#include <stack>
#include <vector>
#include <list>
#include <deque>
using namespace std;

stack<int> s1;                          // default: underlying container is std::deque<int>
stack<int, vector<int>> s2;             // use std::vector as the underlying container
stack<int, list<int>> s3;               // use std::list as the underlying container

stack<int> s4(s1);                      // copy constructor

vector<int> v = {1, 2, 3};
stack<int, vector<int>> s5(v);          // construct from an existing container
                                          // (stack's top = v's back = 3)
```

```
template<
    class T,
    class Container = std::deque<T>
> class stack;
```
- `T` — type of elements stored.
- `Container` — the underlying container doing the real work. Must support `back()`, `push_back()`, `pop_back()` — so valid choices are `std::deque` (default), `std::vector`, or `std::list`. **Not** `std::forward_list` or `std::set` (they lack `push_back`/`pop_back` or `back`).

### 7.2 Why `std::deque` Is the Default
`std::deque` ("double-ended queue") allows `O(1)` push/pop at **both** ends and doesn't need to reallocate/copy everything when it grows (unlike `std::vector`, which may reallocate its entire buffer as it grows). That makes it the safest general-purpose default for a stack adapter.

### 7.3 Element Access

**`top()`** — returns a reference to the top element (like `peek()` in our hand-built version):

```cpp
stack<int> s;
s.push(10);
s.push(20);
cout << s.top();   // 20

s.top() = 99;       // top() returns a REFERENCE — you can modify it in place!
cout << s.top();   // 99
```

### 7.4 Capacity

```cpp
stack<int> s;
cout << s.empty();    // true (1) — checks whether the stack is empty
s.push(1);
s.push(2);
cout << s.size();     // 2 — number of elements
```

### 7.5 Modifiers

**`push(value)`** — pushes a copy of `value` onto the top:
```cpp
stack<int> s;
s.push(10);
s.push(20);
s.push(30);     // top is now 30
```

**`emplace(args...)`** (C++11) — constructs the element **in place** on top, avoiding a copy/move — same idea as `emplace_back` for `std::list`:
```cpp
struct Point { int x, y; Point(int a, int b): x(a), y(b) {} };
stack<Point> s;
s.emplace(1, 2);    // constructs Point(1,2) directly on top — no temporary object made
```

**`pop()`** — removes the top element. **Important: unlike our hand-built `pop()`, `std::stack::pop()` returns `void`** — it does NOT give you back the popped value. You must call `top()` first if you need the value, then `pop()`.

```cpp
stack<int> s;
s.push(10); s.push(20); s.push(30);

int value = s.top();   // read it first: 30
s.pop();                 // then remove it — s is now {10, 20}
cout << value;          // 30
```

> **Common beginner mistake:** calling `s.pop()` expecting a return value, like `int x = s.pop();` — this is a **compile error**. Always `top()` then `pop()`.

**`swap(other)`** — swaps the entire contents with another stack of the same type, `O(1)`:
```cpp
stack<int> a, b;
a.push(1); a.push(2);
b.push(9);
a.swap(b);
// a now has just {9}, b has {1, 2}
```

### 7.6 Comparison Operators
`std::stack` supports `==`, `!=`, `<`, `<=`, `>`, `>=` — these compare the stacks **element by element** (delegating to the underlying container's comparison), as if comparing their contents from bottom to top.

```cpp
stack<int> a, b;
a.push(1); a.push(2);
b.push(1); b.push(2);
cout << (a == b);   // true — same contents in the same order
```

### 7.7 What You CANNOT Do (By Design)
`std::stack` intentionally has **no iterators, no `begin()`/`end()`, no indexing, no way to view the middle or bottom** — only `top()`. This is the whole point of an "adapter": it restricts the richer underlying container down to pure LIFO semantics, so the type system itself prevents LIFO-breaking misuse.

```cpp
stack<int> s;
s.push(1);
// for (int x : s) ...   // ❌ COMPILE ERROR — stack has no begin()/end()
// s[0]                   // ❌ COMPILE ERROR — no operator[]
```
If you need to inspect every element, you must pop them one by one (destructively), or use the underlying container directly instead of a `std::stack`.

### 7.8 Full Example Program

```cpp
#include <iostream>
#include <stack>
using namespace std;

int main() {
    stack<int> s;

    s.push(10);
    s.push(20);
    s.push(30);

    cout << "Top: " << s.top() << endl;     // 30
    cout << "Size: " << s.size() << endl;   // 3

    s.pop();                                  // removes 30
    cout << "Top after pop: " << s.top() << endl;  // 20

    while (!s.empty()) {
        cout << s.top() << " ";
        s.pop();
    }
    // prints: 20 10
    cout << endl;

    cout << "Empty now? " << s.empty() << endl;   // true
}
```

### 7.9 Member Function Summary (from cppreference)

| Function | What it does |
|---|---|
| (constructor) | constructs the stack |
| (destructor) | destructs the stack |
| `operator=` | assigns values |
| **Element access** | |
| `top` | access the top element |
| **Capacity** | |
| `empty` | checks whether the container is empty |
| `size` | returns the number of elements |
| **Modifiers** | |
| `push` | inserts an element at the top |
| `push_range` *(C++23)* | pushes a range of elements |
| `emplace` *(C++11)* | constructs an element in place at the top |
| `pop` | removes the top element (returns nothing!) |
| `swap` | swaps the contents |
| **Non-member** | |
| `operator==`, `!=`, `<`, `<=`, `>`, `>=` | lexicographically compares two stacks |
| `std::swap(std::stack)` | specializes the `std::swap` algorithm |

---

## 8. Complexity Summary

| Operation | Array Stack | Linked-List Stack | `std::stack` (deque-backed) |
|---|---|---|---|
| Push | `O(1)` | `O(1)` | `O(1)` amortized |
| Pop | `O(1)` | `O(1)` | `O(1)` |
| Top / Peek | `O(1)` | `O(1)` | `O(1)` |
| isEmpty | `O(1)` | `O(1)` | `O(1)` |
| Size | `O(1)` | `O(1)` w/ counter, else `O(n)` | `O(1)` |
| Extra memory / element | 0 | 1 pointer | small, container-dependent |
| Fixed capacity? | Yes (unless manually resized) | No | No |
| Random/middle access | ❌ | ❌ | ❌ (by design) |

---

## 9. Practice Examples — Solved in Detail

### 9.1 Balanced Parentheses / Brackets Checker

**Problem:** Given a string containing `()[]{}`, determine if the brackets are balanced and properly nested.

**Idea:** Every time we see an **opening** bracket, push it. Every time we see a **closing** bracket, it must match the bracket on **top of the stack** (the most recently opened, still-unclosed one) — that's exactly LIFO behavior.

**Steps:**
1. Create an empty stack.
2. Walk through the string character by character.
3. If the character is an opening bracket (`(`, `[`, `{`), push it.
4. If it's a closing bracket (`)`, `]`, `}`):
   - If the stack is empty, there's nothing to match → unbalanced, return false.
   - Pop the top. If it doesn't correspond to the current closing bracket, return false.
5. After processing the whole string, if the stack is **not** empty, some opening brackets were never closed → unbalanced.
6. Otherwise, it's balanced.

```
Input: "{[()]}"

char '{' → push       stack: {
char '[' → push       stack: {[
char '(' → push       stack: {[(
char ')' → pop '(' → matches ')' ✓   stack: {[
char ']' → pop '[' → matches ']' ✓   stack: {
char '}' → pop '{' → matches '}' ✓   stack: (empty)

End of string, stack is empty → BALANCED ✓
```

```
Input: "{[(])}"   (interleaved — classic unbalanced case)

'{' → push        stack: {
'[' → push        stack: {[
'(' → push        stack: {[(
']' → pop '(' → does NOT match ']'  → UNBALANCED ✗ (stop immediately)
```

```c
#include <stdbool.h>
#include <string.h>

bool isMatchingPair(char open, char close) {
    return (open == '(' && close == ')') ||
           (open == '[' && close == ']') ||
           (open == '{' && close == '}');
}

bool isBalanced(char* str) {
    Stack* s = createStack(strlen(str));   // worst case: all opening brackets

    for (int i = 0; str[i] != '\0'; i++) {
        char c = str[i];
        if (c == '(' || c == '[' || c == '{') {
            push(s, c);
        } else if (c == ')' || c == ']' || c == '}') {
            if (isEmpty(s)) { freeStack(s); return false; }   // nothing to match
            char top = pop(s);
            if (!isMatchingPair(top, c)) { freeStack(s); return false; }
        }
        // any other character is ignored
    }
    bool balanced = isEmpty(s);   // true only if every opener was matched
    freeStack(s);
    return balanced;
}
```
**Time:** `O(n)` | **Space:** `O(n)` worst case (all openers, e.g. `"((((("`)

**In C++ with `std::stack`:**
```cpp
#include <stack>
#include <string>
using namespace std;

bool isBalanced(const string& str) {
    stack<char> s;
    for (char c : str) {
        if (c == '(' || c == '[' || c == '{') {
            s.push(c);
        } else if (c == ')' || c == ']' || c == '}') {
            if (s.empty()) return false;
            char top = s.top(); s.pop();
            if ((c == ')' && top != '(') ||
                (c == ']' && top != '[') ||
                (c == '}' && top != '{'))
                return false;
        }
    }
    return s.empty();
}
```

---

### 9.2 Reverse a String Using a Stack

**Idea:** Push every character; because popping reverses insertion order (LIFO), reading characters back off the stack produces the string in reverse.

**Steps:**
1. Push every character of the string onto a stack, left to right.
2. Pop every character off (this naturally comes out right to left) and append it to a result string.

```
Input: "abc"
push 'a' → stack: a
push 'b' → stack: a b
push 'c' → stack: a b c   (top='c')

pop → 'c'  result = "c"
pop → 'b'  result = "cb"
pop → 'a'  result = "cba"
```

```c
void reverseString(char* str) {
    int len = strlen(str);
    Stack* s = createStack(len);

    for (int i = 0; i < len; i++)
        push(s, str[i]);

    for (int i = 0; i < len; i++)
        str[i] = pop(s);

    freeStack(s);
}
```
**Time:** `O(n)` | **Space:** `O(n)`

---

### 9.3 Evaluate a Postfix Expression

**Problem:** Given a postfix (Reverse Polish Notation) expression like `"53+82-*"` (meaning `(5+3) * (8-2)`), compute its value.

**Idea:** In postfix, operators come **after** their operands. Scan left to right: push numbers; when an operator is seen, it always applies to the **two most recently pushed** numbers — pop them, compute, push the result back.

**Steps:**
1. Create an empty stack (of numbers).
2. Scan the expression token by token.
3. If the token is a number, push it.
4. If the token is an operator (`+ - * /`):
   - Pop the top value → this is the **second** operand (`b`).
   - Pop again → this is the **first** operand (`a`).
   - Compute `a <op> b`, push the result back.
5. After scanning the whole expression, the single value left on the stack is the answer.

```
Expression: "5 3 + 8 2 - *"   (meaning (5+3) * (8-2))

token '5' → push       stack: [5]
token '3' → push       stack: [5,3]
token '+' → pop b=3, pop a=5 → 5+3=8 → push 8      stack: [8]
token '8' → push       stack: [8,8]
token '2' → push       stack: [8,8,2]
token '-' → pop b=2, pop a=8 → 8-2=6 → push 6      stack: [8,6]
token '*' → pop b=6, pop a=8 → 8*6=48 → push 48    stack: [48]

End of expression → answer = pop() = 48
```

```c
#include <ctype.h>

int evaluatePostfix(char* expr) {
    Stack* s = createStack(strlen(expr));

    for (int i = 0; expr[i] != '\0'; i++) {
        char c = expr[i];

        if (isspace(c)) continue;             // skip spaces

        if (isdigit(c)) {
            push(s, c - '0');                  // single-digit numbers for simplicity
        } else if (c == '+' || c == '-' || c == '*' || c == '/') {
            int b = pop(s);                     // second operand (pushed later)
            int a = pop(s);                     // first operand (pushed earlier)
            int result;
            switch (c) {
                case '+': result = a + b; break;
                case '-': result = a - b; break;
                case '*': result = a * b; break;
                case '/': result = a / b; break;
            }
            push(s, result);
        }
    }
    int answer = pop(s);   // final answer
    freeStack(s);
    return answer;
}
```
**Time:** `O(n)` | **Space:** `O(n)` worst case

**In C++ with `std::stack`:**
```cpp
#include <stack>
#include <sstream>
#include <string>
using namespace std;

int evaluatePostfix(const string& expr) {
    stack<int> s;
    istringstream iss(expr);
    string token;

    while (iss >> token) {
        if (token == "+" || token == "-" || token == "*" || token == "/") {
            int b = s.top(); s.pop();
            int a = s.top(); s.pop();
            if (token == "+") s.push(a + b);
            else if (token == "-") s.push(a - b);
            else if (token == "*") s.push(a * b);
            else s.push(a / b);
        } else {
            s.push(stoi(token));
        }
    }
    return s.top();
}
// evaluatePostfix("5 3 + 8 2 - *") → 48
```

---

### 9.4 Infix to Postfix Conversion (Shunting-Yard style, simplified)

**Problem:** Convert a human-readable infix expression like `"A+B*C"` into postfix `"ABC*+"`.

**Idea:** Use a stack to hold operators temporarily. Output operands immediately. An operator is only pushed to output once we're sure no higher/equal-precedence operator will need to come before it (which the stack enforces via precedence comparison).

**Steps:**
1. Create an empty stack (for operators) and an empty output string.
2. Scan the infix expression token by token.
3. If the token is an operand (letter/digit), append it directly to output.
4. If the token is `(`, push it.
5. If the token is `)`, pop from the stack and append to output **until** a `(` is popped (discard the `(`, don't output it).
6. If the token is an operator:
   - While the stack's top has **greater or equal precedence** (and isn't `(`), pop it to output.
   - Push the current operator.
7. After scanning, pop any remaining operators from the stack to output.

```
Infix: "A+B*C"
precedence: * and / > + and -

'A' → output        output:"A"
'+' → stack empty → push        stack:[+]
'B' → output        output:"AB"
'*' → top is '+' (lower precedence than '*') → don't pop, push '*'
                     stack:[+,*]
'C' → output        output:"ABC"
end → pop remaining: '*' then '+'   output:"ABC*+"

Result: "ABC*+"   ✓ matches evaluatePostfix example above
```

```c
int precedence(char op) {
    if (op == '*' || op == '/') return 2;
    if (op == '+' || op == '-') return 1;
    return 0;   // '(' or anything else
}

void infixToPostfix(char* infix, char* output) {
    Stack* s = createStack(strlen(infix));
    int j = 0;   // index into output

    for (int i = 0; infix[i] != '\0'; i++) {
        char c = infix[i];

        if (isalnum(c)) {
            output[j++] = c;                        // operand → straight to output
        } else if (c == '(') {
            push(s, c);
        } else if (c == ')') {
            while (!isEmpty(s) && peek(s) != '(')
                output[j++] = pop(s);
            pop(s);                                   // discard the '('
        } else {                                       // operator
            while (!isEmpty(s) && precedence(peek(s)) >= precedence(c))
                output[j++] = pop(s);
            push(s, c);
        }
    }
    while (!isEmpty(s))
        output[j++] = pop(s);

    output[j] = '\0';
    freeStack(s);
}
```
**Time:** `O(n)` | **Space:** `O(n)`

---

### 9.5 Next Greater Element

**Problem:** For each element in an array, find the first element to its right that is greater. If none exists, output -1.

**Idea:** Walk the array **right to left**, maintaining a stack of "candidates that could be the next greater element for something to their left." For each new element, pop off everything on the stack that is **smaller or equal** (they can never be the answer for anything further left, since the current element is bigger and closer). Whatever remains on top is the answer; then push the current element as a future candidate.

**Steps:**
1. Create an empty stack and a result array of the same size, initialized to -1.
2. Walk the input array from the **last** index to the **first**.
3. At each element:
   - While the stack is not empty and its top is `<=` current element, pop (it's useless now).
   - If the stack is empty after popping, there's no greater element to the right → result is -1.
   - Otherwise, the new top is the next greater element.
   - Push the current element onto the stack (it could be the answer for something further left).
4. Return the result array.

```
Input: [4, 5, 2, 10, 8]
Process right to left:

i=4, val=8:  stack empty → result[4]=-1,  push 8        stack:[8]
i=3, val=10: pop 8 (8<=10)  → stack empty → result[3]=-1, push 10   stack:[10]
i=2, val=2:  top=10 (10>2, keep)  → result[2]=10, push 2            stack:[10,2]
i=1, val=5:  pop 2 (2<=5), top=10 (10>5, keep) → result[1]=10, push 5  stack:[10,5]
i=0, val=4:  top=5 (5>4, keep) → result[0]=5, push 4                 stack:[10,5,4]

Result: [5, 10, 10, -1, -1]
```

```c
void nextGreaterElement(int* arr, int n, int* result) {
    Stack* s = createStack(n);

    for (int i = n - 1; i >= 0; i--) {
        while (!isEmpty(s) && peek(s) <= arr[i])
            pop(s);

        result[i] = isEmpty(s) ? -1 : peek(s);

        push(s, arr[i]);
    }
    freeStack(s);
}
```
**Time:** `O(n)` — each element is pushed and popped **at most once** total, even though there's a nested `while` loop (this is the classic "amortized `O(1)` per element" stack pattern) | **Space:** `O(n)`

---

### 9.6 Sort a Stack Using Only One Extra Stack (no arrays)

**Problem:** Given a stack, sort it so the smallest element ends up on top, using only one additional stack (no other data structure).

**Idea:** Use a second stack as a "staging area" kept in **sorted** order (largest at its bottom, smallest at its top... we'll pick a consistent convention). Repeatedly pop from the original stack, and insert each popped value into the correct position in the sorted stack by temporarily popping elements back from the sorted stack when they're in the way.

**Steps:**
1. Create an empty helper stack, `sortedStack`.
2. While the original stack is not empty:
   - Pop a value `temp` from the original stack.
   - While `sortedStack` is not empty AND its top is **greater than** `temp`: pop that top value from `sortedStack` and push it back onto the **original** stack (temporarily "undoing" to make room).
   - Push `temp` onto `sortedStack`.
3. After the loop, `sortedStack` holds all elements in sorted order (largest at the bottom, smallest at top). Copy it back to the original stack if needed (or just use `sortedStack` directly).

```
Original (top→bottom): [3, 1, 4, 2]   (top=3)
sortedStack: []

Pop 3 from original.  sortedStack empty → push 3.      sortedStack: [3]
Pop 1 from original.  sortedStack top=3 > 1 → pop 3 back to original.
                       sortedStack empty now → push 1.  sortedStack: [1]
                       original now: [3,4,2]  (3 went back on top)
Pop 3 from original.  sortedStack top=1, NOT > 3 → push 3 directly. sortedStack: [1,3]
Pop 4 from original.  sortedStack top=3, NOT > 4 → push 4 directly. sortedStack: [1,3,4]
Pop 2 from original.  top=4>2→pop 4 back to original; top=3>2→pop 3 back;
                       top=1, NOT>2 → push 2.           sortedStack: [1,2]
                       original now: [4,3]
Pop 3 (from original, re-added). sortedStack top=2, not>3 → push 3. sortedStack:[1,2,3]
Pop 4. sortedStack top=3, not>4 → push 4.                sortedStack:[1,2,3,4]

Final sortedStack bottom→top: 1,2,3,4 → top = 4... 
```
> Note: depending on the exact comparison direction chosen, the "smallest on top" vs "largest on top" convention flips — the C code below produces **largest on top** (so popping the sorted stack gives ascending order); flip the comparison (`<` instead of `>`) to get smallest on top instead.

```c
void sortStack(Stack* original) {
    Stack* sortedStack = createStack(original->capacity);

    while (!isEmpty(original)) {
        int temp = pop(original);

        while (!isEmpty(sortedStack) && peek(sortedStack) > temp) {
            push(original, pop(sortedStack));
        }
        push(sortedStack, temp);
    }

    // copy sortedStack back into original (now sorted, ascending from bottom)
    while (!isEmpty(sortedStack))
        push(original, pop(sortedStack));

    freeStack(sortedStack);
}
```
**Time:** `O(n²)` worst case (each of the n elements may require shifting up to n others) | **Space:** `O(n)` for the helper stack

---

### 9.7 Implement Two Stacks in One Array

**Problem:** Given a single array of size `n`, implement two stacks that both use it efficiently (without a fixed 50/50 split, which wastes space if one stack is much bigger than the other).

**Idea:** Grow **Stack 1 from the left end** (index 0 upward) and **Stack 2 from the right end** (index `n-1` downward). They only collide (overflow) when the two tops meet in the middle — this uses the full array efficiently regardless of how unevenly the two stacks are used.

**Steps:**
1. `top1 = -1` (grows rightward from here), `top2 = n` (grows leftward from here).
2. **push1(value):** if `top1 + 1 == top2` → overflow (they've met). Else `top1++`, `arr[top1] = value`.
3. **push2(value):** if `top2 - 1 == top1` → overflow. Else `top2--`, `arr[top2] = value`.
4. **pop1():** if `top1 == -1` → underflow. Else return `arr[top1--]`.
5. **pop2():** if `top2 == n` → underflow. Else return `arr[top2++]`.

```
Array (n=6):  [_|_|_|_|_|_]
               0 1 2 3 4 5
top1=-1 (grows right from -1)         top2=6 (grows left from 6)

push1(10): top1=0, arr[0]=10      [10|_|_|_|_|_]
push1(20): top1=1, arr[1]=20      [10|20|_|_|_|_]
push2(99): top2=5, arr[5]=99      [10|20|_|_|_|99]
push2(88): top2=4, arr[4]=88      [10|20|_|_|88|99]

Stack1 (bottom→top): 10,20     Stack2 (bottom→top): 99,88
They can keep growing toward each other until top1+1==top2 → overflow.
```

```c
typedef struct {
    int* arr;
    int size;
    int top1, top2;
} TwoStacks;

void initTwoStacks(TwoStacks* s, int n) {
    s->arr = (int*) malloc(n * sizeof(int));
    s->size = n;
    s->top1 = -1;
    s->top2 = n;
}

void push1(TwoStacks* s, int value) {
    if (s->top1 + 1 == s->top2) { printf("Overflow\n"); return; }
    s->arr[++s->top1] = value;
}

void push2(TwoStacks* s, int value) {
    if (s->top2 - 1 == s->top1) { printf("Overflow\n"); return; }
    s->arr[--s->top2] = value;
}

int pop1(TwoStacks* s) {
    if (s->top1 == -1) { printf("Underflow\n"); return -1; }
    return s->arr[s->top1--];
}

int pop2(TwoStacks* s) {
    if (s->top2 == s->size) { printf("Underflow\n"); return -1; }
    return s->arr[s->top2++];
}
```
**Time:** `O(1)` for every operation | **Space:** `O(n)` total, shared efficiently between both stacks

---

### 9.8 Make the Array Stack Auto-Resize (Self-Growing, Like `std::vector`)

**Idea:** our `Stack` struct (section 3) already stores `capacity` separately from `arr`, precisely so it can be **resized** instead of just reporting overflow. Instead of rejecting a `push` when `isFull(s)` is true, reallocate a bigger array (commonly **double** the capacity) and keep going. This is exactly what `std::vector` (and therefore a `vector`-backed `std::stack`) does internally, and it's why `push_back`/`push` on these containers is `O(1)` *amortized* rather than strictly `O(1)`.

**Steps:**
1. On `push`, if `isFull(s)` (i.e. `top + 1 == capacity`): **reallocate** `arr` to double the capacity with `realloc`, and update `capacity`.
2. Proceed with the normal push (increment `top`, store the value).
3. `pop` / `peek` / `isEmpty` stay exactly as in section 4 — resizing only ever happens on `push`.

```c
void push(Stack* s, int value) {
    if (isFull(s)) {                             // full → grow instead of failing
        s->capacity *= 2;
        s->arr = (int*) realloc(s->arr, s->capacity * sizeof(int));
    }
    s->arr[++s->top] = value;
}
```

```
capacity=2, top=1: [10|20]   (full)
push(30):
  isFull → true → capacity doubles to 4, realloc copies [10|20| ? | ? ]
  top++  → top=2
  arr[2] = 30
After:  [10|20|30| ]   capacity=4, top=2   (no overflow error!)
```
**Takeaway:** this is exactly why `std::stack`'s default (`std::deque`-backed) push is `O(1)` amortized — occasionally a resize happens (`O(n)` that one time, copying every element), but spread out ("amortized") over all the pushes, the average cost per push stays `O(1)`.

---

## 10. Quick Reference

```c
// Array-based Stack (dynamically allocated)
typedef struct
{
    int *arr;     // array to store elements
    int capacity; // maximum size of stack
    int top;      // index of top element; -1 = empty
} Stack;

// Linked-List-based Stack
typedef struct Node {
    int data;
    struct Node* next;
} Node;

typedef struct {
    Node* top;   // NULL = empty
    int count;
} Stack;
```

```cpp
// STL
#include <stack>
std::stack<int> s;              // default: backed by std::deque
std::stack<int, std::vector<int>> s2;  // backed by std::vector instead
```

| Operation | Array/LinkedList Stack (hand-built) | `std::stack` |
|---|---|---|
| Add to top | `push(value)` → returns nothing | `s.push(value)` |
| Remove from top | `pop()` → **returns** the removed value | `s.pop()` → returns **void**! Use `top()` first |
| Look at top | `peek()` | `s.top()` |
| Is empty? | `isEmpty()` | `s.empty()` |
| Count | `size()` | `s.size()` |

| Need | Use |
|---|---|
| Simple, fixed-capacity, max performance | Array-based stack |
| Unknown/unbounded size, no capacity limit | Linked-list-based stack |
| Production C++ code | `std::stack` (don't hand-roll one) |
| Need to also iterate/index the elements | Don't use a stack at all — use the underlying container (`vector`, `deque`) directly |
