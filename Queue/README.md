# Queue — Complete Tutorial

A complete guide covering **Array-based Queue (Linear & Circular)**, **Linked-List-based Queue**, **Deque & Priority Queue variants**, and **Queue in STL (`std::queue`, `std::deque`, `std::priority_queue`)**, with fully solved practice problems — built in the same style as the Linked List and Stack tutorials.

---

## Table of Contents
1. [Why Queue?](#1-why-queue)
2. [FIFO — The Core Idea](#2-fifo--the-core-idea)
3. [Array-based Queue (Linear) — Basics](#3-array-based-queue-linear--basics)
4. [The Wasted-Space Problem → Circular Queue](#4-the-wasted-space-problem--circular-queue)
5. [Operations on Circular Array Queue](#5-operations-on-circular-array-queue)
6. [Linked-List-based Queue — Full Operations](#6-linked-list-based-queue--full-operations)
7. [Array vs Linked-List Queue](#7-array-vs-linked-list-queue)
8. [Other Queue Variants: Deque & Priority Queue](#8-other-queue-variants-deque--priority-queue)
9. [`std::queue` — Full STL Tutorial](#9-stdqueue--full-stl-tutorial)
10. [`std::deque` — Full STL Tutorial](#10-stddeque--full-stl-tutorial)
11. [`std::priority_queue` — Full STL Tutorial](#11-stdpriority_queue--full-stl-tutorial)
12. [Complexity Summary](#12-complexity-summary)
13. [Practice Examples — Solved in Detail](#13-practice-examples--solved-in-detail)
14. [Quick Reference](#14-quick-reference)

---

## 1. Why Queue?

A **queue** is a linear data structure that works like a line of people at a ticket counter: the person who arrives **first** is served **first**, and new people join at the **back** of the line. You can't cut in line or be served out of order.

### Real-world / CS use cases
- **Task scheduling** — CPU scheduling, print spoolers, job queues.
- **Breadth-First Search (BFS)** — graph/tree traversal level by level.
- **Request handling** — web server request queues, message queues (Kafka, RabbitMQ).
- **Buffering** — IO buffers, keyboard input buffers, streaming data.
- **Call center systems** — "calls answered in the order received."

### When to use a Queue vs other structures

| Use **Queue** when... | Use something else when... |
|---|---|
| Elements should be processed in arrival order | You need "most recent first" → use a **Stack** |
| You're doing level-order/BFS traversal | You're doing depth-first exploration → use a **Stack** |
| You need fairness (first come, first served) | You need priority-based ordering → use a **Priority Queue** |

---

## 2. FIFO — The Core Idea

**FIFO = First In, First Out.** The first element added is the first one removed.

```
Enqueue 10, 20, 30 in order (adds happen at the REAR):

enqueue(10)         enqueue(20)              enqueue(30)
front  rear         front      rear          front          rear
  ▼     ▼              ▼         ▼              ▼              ▼
┌────┐              ┌────┬────┐              ┌────┬────┬────┐
│ 10 │              │ 10 │ 20 │              │ 10 │ 20 │ 30 │
└────┘              └────┴────┘              └────┴────┴────┘

Now dequeue() three times (removals happen at the FRONT):
dequeue() → 10 (removed first — it was added FIRST)
dequeue() → 20
dequeue() → 30
```

Two ends of interaction exist:
- **enqueue** — add to the **rear**
- **dequeue** — remove from the **front**
- (**peek/front** — look at the front element without removing it, is the 3rd essential operation)

```
Stack vs Queue, side by side:

STACK (LIFO)            QUEUE (FIFO)
   push/pop                enqueue →          → dequeue
   both at TOP           at REAR    [ ...queue... ]   at FRONT
   ┌───┐
   │ 30│◀─ push/pop
   ├───┤
   │ 20│
   ├───┤
   │ 10│
   └───┘
```

---

## 3. Array-based Queue (Linear) — Basics

### What is an Array-based Queue?
A queue implemented on top of an array, with two indices: `front` (points to the first/oldest element) and `rear` (points to the last/newest element).

```
arr:  [ 10 | 20 | 30 |    |    ]
Index:   0    1    2    3    4
         ▲              ▲
       front            rear

front == -1 (or front > rear)  → queue is EMPTY
rear == capacity - 1            → queue is FULL (in the naive/linear version)
```

### Queue Structure (C) — heap-allocated, like the Stack tutorial's pattern

```c
typedef struct
{
    int *arr;     // array to store elements
    int capacity; // maximum size of queue
    int front;    // index of the front (oldest) element
    int rear;     // index of the rear (newest) element
} Queue;
```

### Creating / Initializing a Queue

**Steps:**
1. Allocate an array of the requested `capacity` on the heap.
2. Store `capacity`.
3. Set `front = -1` and `rear = -1` — both signal an empty queue.

```c
Queue* createQueue(int capacity) {
    Queue* q = (Queue*) malloc(sizeof(Queue));
    q->arr = (int*) malloc(capacity * sizeof(int));
    q->capacity = capacity;
    q->front = -1;
    q->rear = -1;
    return q;
}
```

### Checking Empty / Full

```c
int isEmpty(Queue* q) {
    return q->front == -1;
}

int isFull(Queue* q) {
    return q->rear == q->capacity - 1;
}
```

### Enqueue (Linear Version)

**Steps:**
1. Check if the queue is full. If so, report overflow.
2. If the queue was empty (`front == -1`), this is the first element: set `front = 0`.
3. Increment `rear`.
4. Store the value at `arr[rear]`.

```
Before:  front=-1, rear=-1 (empty)
enqueue(10): front was -1 → front=0; rear++ → rear=0; arr[0]=10
After:   [10| | | | ]   front=0, rear=0

enqueue(20): rear++ → rear=1; arr[1]=20
After:   [10|20| | | ]   front=0, rear=1
```

```c
void enqueue(Queue* q, int value) {
    if (isFull(q)) {
        printf("Queue Overflow\n");
        return;
    }
    if (q->front == -1)
        q->front = 0;
    q->rear++;
    q->arr[q->rear] = value;
}
```
**Time:** `O(1)` | **Space:** `O(1)`

### Dequeue (Linear Version)

**Steps:**
1. Check if the queue is empty. If so, report underflow.
2. Read the value at `arr[front]` (so it can be returned).
3. If `front == rear` (only one element was left), reset both to -1 — the queue is now empty.
4. Otherwise, increment `front` (the next element becomes the new front).
5. Return the saved value.

```
Before: [10|20|30| | ]  front=0, rear=2
dequeue():
  value = arr[0] = 10
  front != rear → front++  → front=1
After:  [10|20|30| | ]  front=1, rear=2   (10 is logically gone)
Returns: 10
```

```c
int dequeue(Queue* q) {
    if (isEmpty(q)) {
        printf("Queue Underflow\n");
        return -1;
    }
    int value = q->arr[q->front];
    if (q->front == q->rear) {
        q->front = -1;
        q->rear = -1;
    } else {
        q->front++;
    }
    return value;
}
```
**Time:** `O(1)` | **Space:** `O(1)`

### Peek / Front

```c
int peek(Queue* q) {
    if (isEmpty(q)) {
        printf("Queue is empty\n");
        return -1;
    }
    return q->arr[q->front];
}
```

### The Fatal Flaw of the Linear Queue

```
capacity = 5
[10|20|30|40|50]   front=0, rear=4  (FULL)

dequeue() x3:
front moves to 3:  [10|20|30|40|50]   front=3, rear=4
                     ▲ ▲ ▲  these 3 slots are now WASTED — unusable!

enqueue(60)?  rear == capacity-1 (4) → isFull() says TRUE
              → "Overflow" reported, even though 3 slots are free!

This is the LINEAR queue's fatal flaw: once rear hits the end,
it can NEVER enqueue again, even if front has moved far ahead
and left plenty of empty space behind it.
```
> This is exactly why real implementations use a **circular queue** instead (next section) — it reuses the freed-up space at the front of the array.

---

## 4. The Wasted-Space Problem → Circular Queue

### The Fix: Wrap Around

A **circular queue** treats the array as if its end connects back to its beginning — like a clock face. When `rear` reaches the last index, the *next* enqueue wraps around to index 0 (if that slot is free).

```
Circular view of the same array (capacity=5):

        index 0
       ┌───────┐
index4 │       │ index 1
       │   ◯   │
index3 │       │ index 2
       └───────┘

Moving "forward" from index 4 goes to index 0, not out of bounds.
```

### The Modulo Trick

```c
rear = (rear + 1) % capacity;
front = (front + 1) % capacity;
```

```
capacity=5, rear=4 (last index)
next rear = (4 + 1) % 5 = 0   → wraps back to index 0!
```

### Visualizing the Wrap-Around

```
Start: front=0, rear=-1 (empty), capacity=5
enqueue(10,20,30,40,50):  [10|20|30|40|50]  front=0, rear=4 (FULL)

dequeue() x2:  front moves to 2
               [10|20|30|40|50]  front=2, rear=4
                XX  XX              (10,20 logically removed)

enqueue(60):  rear = (4+1)%5 = 0 → WRAPS to index 0!
               [60|20|30|40|50]  front=2, rear=0
                ▲ written here (old "10" slot reused)

enqueue(70):  rear = (0+1)%5 = 1 → WRAPS to index 1!
               [60|70|30|40|50]  front=2, rear=1
               FULL again (front is right after rear, all 5 slots logically used)
```

### The New Problem: Telling "Full" Apart from "Empty"

With wrap-around, `front == rear` can mean **either** "one element left" or "completely empty" — ambiguous! There are 3 common fixes:

| Approach | How it distinguishes full vs. empty |
|---|---|
| **A. Keep a `count` field** | `count == 0` → empty; `count == capacity` → full. (Used below — simplest and clearest.) |
| **B. Sacrifice one slot** | Queue is "full" when it would use all N slots but really only fills N-1; `front == rear` always means empty. |
| **C. Use a boolean flag** | A separate `isFull` flag toggled on enqueue/dequeue. |

This tutorial uses **Approach A** (a `count` field) — it's the clearest to reason about and also gives `O(1)` `size()` for free.

---

## 5. Operations on Circular Array Queue

### Queue Structure (C) — Circular, with a `count` field

```c
typedef struct
{
    int *arr;      // array to store elements
    int capacity;  // maximum size of queue
    int front;     // index of the front (oldest) element
    int rear;      // index of the rear (newest) element
    int count;     // current number of elements
} Queue;
```

### 5.1 Create / Initialize

**Steps:**
1. Allocate the array of size `capacity`.
2. Set `front = 0`, `rear = -1`, `count = 0` (all three conventions work; this one keeps `rear` "one behind" the next write position).

```c
Queue* createQueue(int capacity) {
    Queue* q = (Queue*) malloc(sizeof(Queue));
    q->arr = (int*) malloc(capacity * sizeof(int));
    q->capacity = capacity;
    q->front = 0;
    q->rear = -1;
    q->count = 0;
    return q;
}
```

### 5.2 isEmpty / isFull / size

**Steps:**
1. `isEmpty`: true when `count == 0`.
2. `isFull`: true when `count == capacity`.
3. `size`: simply return `count` — `O(1)`, no counting loop needed.

```c
int isEmpty(Queue* q) { return q->count == 0; }
int isFull(Queue* q)  { return q->count == q->capacity; }
int size(Queue* q)    { return q->count; }
```

### 5.3 Enqueue (Circular)

**Steps:**
1. Check if the queue is full. If so, report overflow.
2. Move `rear` forward **with wrap-around**: `rear = (rear + 1) % capacity`.
3. Store the value at `arr[rear]`.
4. Increment `count`.

```
Before: [60|70|30|40|50]  front=2, rear=1, count=5 → FULL, can't enqueue.

After dequeue() x1:  front=(2+1)%5=3, count=4
                      [60|70| |40|50]  front=3, rear=1, count=4

enqueue(80): rear=(1+1)%5=2 → arr[2]=80 → count=5
              [60|70|80|40|50]  front=3, rear=2, count=5 (full again)
```

```c
void enqueue(Queue* q, int value) {
    if (isFull(q)) {
        printf("Queue Overflow\n");
        return;
    }
    q->rear = (q->rear + 1) % q->capacity;
    q->arr[q->rear] = value;
    q->count++;
}
```
**Time:** `O(1)` | **Space:** `O(1)`

### 5.4 Dequeue (Circular)

**Steps:**
1. Check if the queue is empty. If so, report underflow.
2. Read the value at `arr[front]` (so it can be returned).
3. Move `front` forward **with wrap-around**: `front = (front + 1) % capacity`.
4. Decrement `count`.
5. Return the saved value.

```
Before: [60|70|80|40|50]  front=3, rear=2, count=5
dequeue():
  value = arr[3] = 40
  front = (3+1)%5 = 4
  count-- → 4
After:  front=4, rear=2, count=4
Returns: 40
```

```c
int dequeue(Queue* q) {
    if (isEmpty(q)) {
        printf("Queue Underflow\n");
        return -1;
    }
    int value = q->arr[q->front];
    q->front = (q->front + 1) % q->capacity;
    q->count--;
    return value;
}
```
**Time:** `O(1)` | **Space:** `O(1)`

### 5.5 Peek / Front and Rear

```c
int peekFront(Queue* q) {
    if (isEmpty(q)) { printf("Empty\n"); return -1; }
    return q->arr[q->front];
}

int peekRear(Queue* q) {
    if (isEmpty(q)) { printf("Empty\n"); return -1; }
    return q->arr[q->rear];
}
```
**Time:** `O(1)` each

### 5.6 Display (front to rear)

**Steps:**
1. If empty, print a message and stop.
2. Starting at `front`, print `count` elements, advancing the index with wrap-around (`(i + 1) % capacity`) each time — NOT simply `i < rear`, since `rear` may be numerically smaller than `front` after wrapping!

```c
void display(Queue* q) {
    if (isEmpty(q)) {
        printf("Queue is empty\n");
        return;
    }
    int i = q->front;
    for (int c = 0; c < q->count; c++) {
        printf("%d ", q->arr[i]);
        i = (i + 1) % q->capacity;
    }
}
```
**Time:** `O(n)` | **Space:** `O(1)`

### 5.7 Free the Queue

```c
void freeQueue(Queue* q) {
    free(q->arr);
    free(q);
}
```

### 5.8 Full Circular Queue Example (C)

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int *arr, capacity, front, rear, count;
} Queue;

Queue* createQueue(int capacity) {
    Queue* q = (Queue*) malloc(sizeof(Queue));
    q->arr = (int*) malloc(capacity * sizeof(int));
    q->capacity = capacity;
    q->front = 0; q->rear = -1; q->count = 0;
    return q;
}

int isEmpty(Queue* q) { return q->count == 0; }
int isFull(Queue* q)  { return q->count == q->capacity; }

void enqueue(Queue* q, int value) {
    if (isFull(q)) { printf("Overflow\n"); return; }
    q->rear = (q->rear + 1) % q->capacity;
    q->arr[q->rear] = value;
    q->count++;
}

int dequeue(Queue* q) {
    if (isEmpty(q)) { printf("Underflow\n"); return -1; }
    int value = q->arr[q->front];
    q->front = (q->front + 1) % q->capacity;
    q->count--;
    return value;
}

void freeQueue(Queue* q) { free(q->arr); free(q); }

int main() {
    Queue* q = createQueue(5);
    enqueue(q, 10);
    enqueue(q, 20);
    enqueue(q, 30);
    printf("Dequeue: %d\n", dequeue(q));  // 10
    printf("Dequeue: %d\n", dequeue(q));  // 20
    enqueue(q, 40);
    enqueue(q, 50);
    enqueue(q, 60);   // wraps around internally
    while (!isEmpty(q))
        printf("%d ", dequeue(q));   // 30 40 50 60
    freeQueue(q);
}
```

---

## 6. Linked-List-based Queue — Full Operations

A queue can also be built on a **singly linked list**, where **enqueue adds at the tail** and **dequeue removes from the head**. This removes the fixed-capacity problem entirely, just like the linked-list stack did.

```
front/head                                    rear/tail
   │                                               │
   ▼                                               ▼
┌────┬──────┐    ┌────┬──────┐    ┌────┬──────┐
│ 10 │  ●───┼───▶│ 20 │  ●───┼───▶│ 30 │ NULL │
└────┴──────┘    └────┴──────┘    └────┴──────┘
 (oldest = front                   (newest = rear
  → dequeue here)                   → enqueue here)
```

**Why keep BOTH `front` and `rear` pointers?** With only a `front`(head) pointer, enqueueing at the tail would require walking the entire list every time (`O(n)`). Keeping a separate `rear` pointer makes enqueue `O(1)` too — this mirrors exactly why `std::list` keeps a tail pointer internally (see the Linked List tutorial, section 10.9).

### Node & Queue Structure

```c
typedef struct Node {
    int data;
    struct Node* next;
} Node;

typedef struct {
    Node* front;   // oldest element (dequeue from here)
    Node* rear;    // newest element (enqueue here)
    int count;
} Queue;
```

### 6.1 Initialize

**Steps:**
1. Set both `front` and `rear` to `NULL` — an empty queue has no nodes.
2. Set `count = 0`.

```c
void initQueue(Queue* q) {
    q->front = NULL;
    q->rear = NULL;
    q->count = 0;
}
```

### 6.2 isEmpty

```c
int isEmpty(Queue* q) {
    return q->front == NULL;
}
```
> No `isFull()` needed — same reasoning as the linked-list stack: bounded only by available memory.

### 6.3 Enqueue

**Steps:**
1. Create a new node holding `value`, with `next = NULL` (it will be the new last node).
2. If the queue is empty, both `front` and `rear` become this new node.
3. Otherwise, link the current `rear` node to the new node (`rear->next = newNode`), then move `rear` to the new node.
4. Increment `count`.

```
Before: front→[10]→[20]→NULL,  rear→[20]
enqueue(30):
  newNode[30]->next = NULL
  rear(20)->next = newNode[30]     ([20] now points to [30])
  rear = newNode[30]
After:  front→[10]→[20]→[30]→NULL,  rear→[30]
```

```c
void enqueue(Queue* q, int value) {
    Node* newNode = (Node*) malloc(sizeof(Node));
    newNode->data = value;
    newNode->next = NULL;

    if (q->rear == NULL) {          // queue was empty
        q->front = newNode;
        q->rear = newNode;
    } else {
        q->rear->next = newNode;
        q->rear = newNode;
    }
    q->count++;
}
```
**Time:** `O(1)` | **Space:** `O(1)` per enqueue

### 6.4 Dequeue

**Steps:**
1. Check if the queue is empty. If so, report underflow.
2. Save the current `front` node in a temporary pointer.
3. Move `front` forward to `front->next`.
4. If `front` becomes `NULL` (the queue is now empty), also set `rear = NULL` — otherwise `rear` would dangle, pointing at the freed node.
5. Read the value from the saved node, free it, decrement `count`, and return the value.

```
Before: front→[10]→[20]→[30]→NULL, rear→[30], count=3
dequeue():
  temp = front (10)
  front = front->next (20)
  front is not NULL → rear stays [30]
  value = temp->data (10); free(temp); count-- → 2
After:  front→[20]→[30]→NULL, rear→[30], count=2
Returns: 10
```

```c
int dequeue(Queue* q) {
    if (isEmpty(q)) {
        printf("Queue Underflow\n");
        return -1;
    }
    Node* temp = q->front;
    int value = temp->data;

    q->front = q->front->next;
    if (q->front == NULL)           // list just became empty
        q->rear = NULL;

    free(temp);
    q->count--;
    return value;
}
```
**Time:** `O(1)` | **Space:** `O(1)`

### 6.5 Peek / Front and Rear

```c
int peekFront(Queue* q) {
    if (isEmpty(q)) { printf("Empty\n"); return -1; }
    return q->front->data;
}

int peekRear(Queue* q) {
    if (isEmpty(q)) { printf("Empty\n"); return -1; }
    return q->rear->data;
}
```
**Time:** `O(1)` each (thanks to the `rear` pointer — without it, `peekRear` would be `O(n)`)

### 6.6 Size

```c
int size(Queue* q) {
    return q->count;   // O(1) thanks to the maintained counter
}
```

### 6.7 Display (front to rear)

```c
void display(Queue* q) {
    if (isEmpty(q)) {
        printf("Queue is empty\n");
        return;
    }
    Node* temp = q->front;
    while (temp != NULL) {
        printf("%d ", temp->data);
        temp = temp->next;
    }
}
```
**Time:** `O(n)` | **Space:** `O(1)`

### 6.8 Free the Whole Queue

```c
void freeQueue(Queue* q) {
    Node* temp;
    while (q->front != NULL) {
        temp = q->front;
        q->front = q->front->next;
        free(temp);
    }
    q->rear = NULL;
    q->count = 0;
}
```
**Time:** `O(n)` | **Space:** `O(1)`

### 6.9 Full Linked-List Queue Example (C)

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct Node {
    int data;
    struct Node* next;
} Node;

typedef struct {
    Node* front;
    Node* rear;
    int count;
} Queue;

void initQueue(Queue* q) { q->front = q->rear = NULL; q->count = 0; }
int isEmpty(Queue* q)    { return q->front == NULL; }

void enqueue(Queue* q, int value) {
    Node* newNode = (Node*) malloc(sizeof(Node));
    newNode->data = value;
    newNode->next = NULL;
    if (q->rear == NULL) { q->front = q->rear = newNode; }
    else { q->rear->next = newNode; q->rear = newNode; }
    q->count++;
}

int dequeue(Queue* q) {
    if (isEmpty(q)) { printf("Underflow\n"); return -1; }
    Node* temp = q->front;
    int value = temp->data;
    q->front = q->front->next;
    if (q->front == NULL) q->rear = NULL;
    free(temp);
    q->count--;
    return value;
}

int main() {
    Queue q;
    initQueue(&q);
    enqueue(&q, 10);
    enqueue(&q, 20);
    enqueue(&q, 30);
    printf("Dequeue: %d\n", dequeue(&q));  // 10
    printf("Dequeue: %d\n", dequeue(&q));  // 20
    printf("Size: %d\n", q.count);          // 1
}
```

---

## 7. Array vs Linked-List Queue

```
Circular Array Queue:             Linked-List Queue:
┌───┬───┬───┬───┬───┐             front→[10]→[20]→[30]→NULL ←rear
│ 10│ 20│ 30│   │   │
└───┴───┴───┴───┴───┘             Grows one malloc'd node at a time —
 front=0,rear=2,capacity=5          no fixed upper bound
 (fixed at createQueue)
```

| Feature | Array-based (circular) | Linked-List-based |
|---|---|---|
| Max capacity | Fixed at creation (resizable with extra work) | Dynamic — grows/shrinks per element |
| Memory layout | Contiguous (cache-friendly) | Scattered (extra pointer per node) |
| Enqueue/Dequeue time | `O(1)` | `O(1)` |
| Extra memory per element | 0 | 1 pointer (`next`) |
| Needs wrap-around logic | Yes (`% capacity`) | No |
| Overflow possible? | Yes, once full | Only if system runs out of memory |
| `size()` | `O(1)` with `count` field | `O(1)` with `count` field |

---

## 8. Other Queue Variants: Deque & Priority Queue

### 8.1 Deque (Double-Ended Queue)
A **deque** allows insertion AND removal at **both ends** — front and rear — unlike a regular queue (insert rear / remove front only).

```
        insert/remove           insert/remove
             │                        │
             ▼                        ▼
      ┌────┬────┬────┬────┐
      │ 10 │ 20 │ 30 │ 40 │
      └────┴────┴────┴────┘
       ▲ front                  rear ▲
```
4 core operations: `insertFront`, `insertRear`, `deleteFront`, `deleteRear`. A regular queue is really just a deque with half its operations unused; a stack can even be simulated with a deque using only one end. C++'s `std::deque` is covered in full in section 10.

### 8.2 Priority Queue
A **priority queue** is NOT strictly FIFO — instead, each element has a **priority**, and the element with the **highest priority** is dequeued first, regardless of arrival order. Ties are usually broken by arrival order (FIFO among equal priorities).

```
Insert (5, "low"), (1, "high"), (3, "medium")   [lower number = higher priority here]

Regular Queue would dequeue: "low", "high", "medium"  (arrival order)
Priority Queue dequeues:     "high", "medium", "low"  (priority order!)
```
Typically implemented with a **heap** (binary heap) for `O(log n)` insert/extract — far better than a naive sorted array/list (`O(n)` insert) or an unsorted one (`O(n)` extract-max). C++'s `std::priority_queue` is covered in full in section 11.

### 8.3 Circular Queue — Already Covered
Sections 4–5 above. Worth noting: a circular queue is also the standard way to implement a fixed-size **ring buffer**, widely used in streaming/IO systems.

---

## 9. `std::queue` — Full STL Tutorial

`std::queue` (from `<queue>`) is a **container adapter** — like `std::stack`, it doesn't implement its own storage. By default it wraps `std::deque` and restricts access to pure FIFO operations.

### 9.1 Declaring a `std::queue`

```cpp
#include <queue>
#include <list>
#include <deque>
using namespace std;

queue<int> q1;                       // default: underlying container is std::deque<int>
queue<int, list<int>> q2;            // use std::list as the underlying container

queue<int> q3(q1);                   // copy constructor

deque<int> d = {1, 2, 3};
queue<int> q4(d);                    // construct from an existing container
                                       // (front = d.front() = 1, back = d.back() = 3)
```

```
template<
    class T,
    class Container = std::deque<T>
> class queue;
```
- `Container` must support `front()`, `back()`, `push_back()`, `pop_front()` — so valid choices are `std::deque` (default) or `std::list`. **Not** `std::vector` (no efficient `pop_front`), and **not** `std::forward_list` (no `back()`).

### 9.2 Element Access

```cpp
queue<int> q;
q.push(10);
q.push(20);
q.push(30);

cout << q.front();   // 10 — oldest element (next to be removed)
cout << q.back();    // 30 — newest element (just added)

q.front() = 99;       // both return REFERENCES — modifiable in place
```

### 9.3 Capacity

```cpp
queue<int> q;
cout << q.empty();   // true
q.push(1);
cout << q.size();    // 1
```

### 9.4 Modifiers

**`push(value)`** — adds to the back (the "enqueue"):
```cpp
queue<int> q;
q.push(10);
q.push(20);   // q: front=10 ... back=20
```

**`emplace(args...)`** (C++11) — constructs in place at the back:
```cpp
struct Point { int x, y; Point(int a, int b): x(a), y(b) {} };
queue<Point> q;
q.emplace(1, 2);   // constructs Point(1,2) directly — no temporary
```

**`pop()`** — removes the front element. **Like `std::stack::pop()`, this returns `void`!** Read with `front()` first if you need the value.

```cpp
queue<int> q;
q.push(10); q.push(20); q.push(30);

int value = q.front();   // read first: 10
q.pop();                   // then remove — q now has {20, 30}
cout << value;            // 10
```

**`swap(other)`** — `O(1)` swap of entire contents:
```cpp
queue<int> a, b;
a.push(1); a.push(2);
b.push(9);
a.swap(b);   // a now has just {9}, b has {1, 2}
```

### 9.5 Comparison Operators
Like `std::stack`, supports `==`, `!=`, `<`, `<=`, `>`, `>=`, comparing contents element-by-element (front to back).

### 9.6 What You CANNOT Do (By Design)
No iterators, no indexing, no way to peek the middle — only `front()`/`back()`. Same reasoning as `std::stack`: the adapter enforces FIFO discipline at the type level.

```cpp
queue<int> q;
q.push(1);
// for (int x : q) ...   // ❌ COMPILE ERROR — no begin()/end()
// q[0]                   // ❌ COMPILE ERROR — no operator[]
```

### 9.7 Full Example Program

```cpp
#include <iostream>
#include <queue>
using namespace std;

int main() {
    queue<int> q;
    q.push(10);
    q.push(20);
    q.push(30);

    cout << "Front: " << q.front() << ", Back: " << q.back() << endl;  // 10, 30
    cout << "Size: " << q.size() << endl;                               // 3

    q.pop();   // removes 10
    cout << "Front after pop: " << q.front() << endl;  // 20

    while (!q.empty()) {
        cout << q.front() << " ";
        q.pop();
    }
    // prints: 20 30
}
```

### 9.8 Member Function Summary

| Function | What it does |
|---|---|
| (constructor) / (destructor) / `operator=` | construct / destruct / assign |
| **Element access** | |
| `front` | access the first (oldest) element |
| `back` | access the last (newest) element |
| **Capacity** | |
| `empty` | checks whether empty |
| `size` | returns number of elements |
| **Modifiers** | |
| `push` | inserts at the back |
| `push_range` *(C++23)* | pushes a range of elements |
| `emplace` *(C++11)* | constructs in place at the back |
| `pop` | removes the front element (returns nothing!) |
| `swap` | swaps the contents |
| **Non-member** | `operator==`/`!=`/`<`/`<=`/`>`/`>=`, `std::swap` |

---

## 10. `std::deque` — Full STL Tutorial

`std::deque` ("double-ended queue", pronounced "deck") is a **real container** (not an adapter) that supports fast `O(1)` insertion/removal at **both** ends, plus **random access** (`d[i]`) — something neither `std::list` nor the queue/stack adapters offer.

```
         O(1) push_front/pop_front        O(1) push_back/pop_back
                   │                               │
                   ▼                               ▼
            ┌────┬────┬────┬────┬────┐
            │ 10 │ 20 │ 30 │ 40 │ 50 │  ← also supports d[i] random access, O(1)!
            └────┴────┴────┴────┴────┘
```
> Internally, a deque is usually implemented as a sequence of fixed-size "chunks" (not one contiguous block like `vector`), which is what makes both-end `O(1)` insertion possible without the reallocate-and-copy-everything cost a `vector`'s `push_front` would have.

### 10.1 Declaring a `std::deque`

```cpp
#include <deque>
using namespace std;

deque<int> d1;                        // empty
deque<int> d2 = {1, 2, 3};            // initializer list
deque<int> d3(5, 7);                  // 5 copies of 7 → {7,7,7,7,7}
deque<int> d4(d2);                    // copy
deque<int> d5(d2.begin(), d2.end());  // from a range
```

### 10.2 Element Access

```cpp
deque<int> d = {10, 20, 30};
cout << d[1];          // 20 — random access, O(1), no bounds check
cout << d.at(1);       // 20 — bounds-checked version (throws std::out_of_range if invalid)
cout << d.front();     // 10
cout << d.back();      // 30
```

### 10.3 Iterators
Deques support **random-access iterators** (richer than `std::list`'s bidirectional ones):
```cpp
deque<int> d = {10, 20, 30};
for (auto it = d.begin(); it != d.end(); ++it) cout << *it << " ";   // 10 20 30
for (int x : d) cout << x << " ";                                     // 10 20 30
for (auto it = d.rbegin(); it != d.rend(); ++it) cout << *it << " ";  // 30 20 10
```

### 10.4 Capacity
```cpp
deque<int> d = {1, 2, 3};
cout << d.empty();     // false
cout << d.size();      // 3
cout << d.max_size();
d.shrink_to_fit();     // request to reduce unused reserved memory (C++11)
```

### 10.5 Modifiers

**`push_back` / `push_front`** — `O(1)` each (this is the headline feature):
```cpp
deque<int> d = {20, 30};
d.push_back(40);    // {20, 30, 40}
d.push_front(10);   // {10, 20, 30, 40}
```

**`pop_back` / `pop_front`** — `O(1)` each:
```cpp
d.pop_back();    // removes 40 → {10, 20, 30}
d.pop_front();   // removes 10 → {20, 30}
```

**`emplace_back` / `emplace_front`**:
```cpp
d.emplace_back(99);
d.emplace_front(1);
```

**`insert(iterator, value)`** — insert before a given position (not just at the ends):
```cpp
deque<int> d = {10, 20, 30};
auto it = d.begin() + 1;    // deque supports iterator arithmetic! (random access)
d.insert(it, 15);           // {10, 15, 20, 30}
```

**`erase(iterator)`** / **`erase(first,last)`**:
```cpp
deque<int> d = {10, 15, 20, 30};
d.erase(d.begin() + 1);     // removes 15 → {10, 20, 30}
```

**`clear()`**, **`resize(n)`**, **`swap()`**, **`assign()`** — same semantics as `std::list`/`std::vector`.

### 10.6 Full Example Program

```cpp
#include <iostream>
#include <deque>
using namespace std;

int main() {
    deque<int> d;
    d.push_back(20);
    d.push_back(30);
    d.push_front(10);     // {10, 20, 30}

    cout << d[0] << " " << d[1] << " " << d[2] << endl;  // 10 20 30 (random access!)

    d.pop_front();          // {20, 30}
    d.pop_back();            // {20}

    cout << "Front: " << d.front() << ", Back: " << d.back() << endl;  // 20, 20
}
```

### 10.7 `std::deque` vs `std::queue` vs `std::list`

| Feature | `std::deque` | `std::queue` (adapter) | `std::list` |
|---|---|---|---|
| Random access `d[i]` | ✅ `O(1)` | ❌ | ❌ |
| Push/pop both ends | ✅ `O(1)` | Only front(pop)/back(push) exposed | ✅ `O(1)` |
| Iterators | ✅ random-access | ❌ none | ✅ bidirectional |
| Underlying storage | chunked array | wraps a deque (by default) | doubly linked nodes |
| Typical use | general-purpose both-ends container | strict FIFO discipline | frequent splicing/mid-list ops |

---

## 11. `std::priority_queue` — Full STL Tutorial

`std::priority_queue` (from `<queue>`) is another **container adapter**, built on top of a **heap** (by default a max-heap over `std::vector`). The **largest** element (by default `operator<`) is always at the top — unlike `std::queue`, this is NOT FIFO; it's priority-ordered.

```
template<
    class T,
    class Container = std::vector<T>,
    class Compare = std::less<typename Container::value_type>
> class priority_queue;
```

### 11.1 Declaring a `std::priority_queue`

```cpp
#include <queue>
#include <vector>
#include <functional>   // for std::greater
using namespace std;

priority_queue<int> maxHeap;                              // default: max-heap (largest on top)
priority_queue<int, vector<int>, greater<int>> minHeap;    // min-heap (smallest on top)

priority_queue<int> pq2({3, 1, 4, 1, 5});                 // C++17: construct from initializer_list
vector<int> v = {5, 2, 8, 1};
priority_queue<int> pq3(v.begin(), v.end());               // construct from a range (heapifies it)
```

```
Max-heap conceptually (not the real array layout, just the priority order):
            8
          /   \
         5     4
        / \
       2   1
Top (highest priority) is always 8.
```

### 11.2 Element Access

**Only `top()`** — you can see the highest-priority element, nothing else:
```cpp
priority_queue<int> pq;
pq.push(5); pq.push(1); pq.push(8); pq.push(3);
cout << pq.top();   // 8 — the maximum, regardless of insertion order
```
> Unlike `std::stack`/`std::queue`'s `top()`/`front()`, `priority_queue::top()` returns a **const reference** — you cannot modify it in place, because doing so could silently break the heap's internal ordering.

### 11.3 Capacity
```cpp
priority_queue<int> pq;
cout << pq.empty();   // true
pq.push(1);
cout << pq.size();    // 1
```

### 11.4 Modifiers

**`push(value)`** — inserts and re-heapifies, `O(log n)`:
```cpp
priority_queue<int> pq;
pq.push(5);
pq.push(10);   // 10 becomes new top
pq.push(1);    // top still 10
```

**`emplace(args...)`** — constructs in place, `O(log n)`:
```cpp
priority_queue<pair<int,int>> pq;
pq.emplace(3, 4);   // constructs pair(3,4) directly
```

**`pop()`** — removes the top (highest-priority) element and re-heapifies, `O(log n)`. Returns `void`, same gotcha as `stack`/`queue`:
```cpp
priority_queue<int> pq;
pq.push(5); pq.push(1); pq.push(8);

int top = pq.top();   // 8
pq.pop();                // removes it, re-heapifies → new top is 5
```

**`swap(other)`**:
```cpp
priority_queue<int> a, b;
a.push(1); b.push(9);
a.swap(b);
```

### 11.5 Custom Comparators — Min-Heap and Custom Objects

**Min-heap (smallest on top) using `std::greater`:**
```cpp
#include <queue>
#include <functional>
priority_queue<int, vector<int>, greater<int>> minHeap;
minHeap.push(5); minHeap.push(1); minHeap.push(8);
cout << minHeap.top();   // 1 — smallest is "highest priority" here
```

**Custom struct with a custom comparator (lambda):**
```cpp
struct Task { string name; int priority; };

auto cmp = [](const Task& a, const Task& b) {
    return a.priority < b.priority;   // higher "priority" value = more important
};
priority_queue<Task, vector<Task>, decltype(cmp)> pq(cmp);

pq.push({"Low", 1});
pq.push({"Urgent", 10});
pq.push({"Medium", 5});

cout << pq.top().name;   // "Urgent" (priority 10 is highest)
```

### 11.6 Full Example Program

```cpp
#include <iostream>
#include <queue>
using namespace std;

int main() {
    priority_queue<int> pq;   // max-heap
    pq.push(3);
    pq.push(1);
    pq.push(4);
    pq.push(1);
    pq.push(5);

    cout << "Elements in priority order: ";
    while (!pq.empty()) {
        cout << pq.top() << " ";
        pq.pop();
    }
    // prints: 5 4 3 1 1   (descending — this is essentially heap sort!)
}
```

### 11.7 Member Function Summary

| Function | What it does |
|---|---|
| (constructor) / (destructor) / `operator=` | construct / destruct / assign |
| **Element access** | |
| `top` | access the highest-priority (by default, largest) element |
| **Capacity** | |
| `empty` | checks whether empty |
| `size` | returns number of elements |
| **Modifiers** | |
| `push` | inserts an element, maintains heap property, `O(log n)` |
| `push_range` *(C++23)* | pushes a range of elements |
| `emplace` *(C++11)* | constructs in place, `O(log n)` |
| `pop` | removes the top element, `O(log n)` (returns nothing!) |
| `swap` | swaps the contents |

---

## 12. Complexity Summary

| Operation | Linear Array Queue | Circular Array Queue | Linked-List Queue | `std::queue` | `std::deque` | `std::priority_queue` |
|---|---|---|---|---|---|---|
| Enqueue/push | `O(1)` (but wastes slots) | `O(1)` | `O(1)` | `O(1)` | `O(1)` both ends | `O(log n)` |
| Dequeue/pop | `O(1)` | `O(1)` | `O(1)` | `O(1)` | `O(1)` both ends | `O(log n)` |
| Peek front/top | `O(1)` | `O(1)` | `O(1)` | `O(1)` | `O(1)` | `O(1)` |
| Random access by index | ❌ | ❌ | ❌ | ❌ | ✅ `O(1)` | ❌ |
| `size()` | `O(1)` w/ tracking | `O(1)` (`count`) | `O(1)` (`count`) | `O(1)` | `O(1)` | `O(1)` |
| Fixed capacity? | Yes | Yes (unless resized) | No | No | No | No |
| Reuses freed space | ❌ (fatal flaw) | ✅ (wrap-around) | n/a (always frees) | n/a | n/a | n/a |
| Ordering | FIFO | FIFO | FIFO | FIFO | insertion order | priority order |

---

## 13. Practice Examples — Solved in Detail

### 13.1 Implement a Stack Using Two Queues

**Problem:** Using only queue operations (`enqueue`/`dequeue`), implement a stack's `push`/`pop` (LIFO behavior from FIFO building blocks).

**Idea:** Make `push` expensive and `pop` cheap: after enqueueing the new element into an empty helper queue `q2`, dump everything from the "main" queue `q1` behind it, then swap their roles — so the **most recently pushed** element always ends up at the **front** of `q1`, making `pop` a simple `dequeue`.

**Steps (push(x)):**
1. Enqueue `x` into `q2` (initially empty).
2. Move all elements from `q1` into `q2` (dequeue from `q1`, enqueue into `q2`) — this puts `x` at the front of `q2`, followed by all older elements in their original order.
3. Swap the names/roles of `q1` and `q2` (so `q1` is now the "live" queue, with `x` at its front).

**Steps (pop()):** simply `dequeue` from `q1` — the front is always the most-recently-pushed element.

```
q1 = [1, 2, 3]  (1 is front, pushed first; 3 pushed last)
push(4):
  q2 = []
  enqueue(q2, 4)            → q2 = [4]
  move all of q1 into q2:
    dequeue q1(1) → enqueue q2 → q2=[4,1]
    dequeue q1(2) → enqueue q2 → q2=[4,1,2]
    dequeue q1(3) → enqueue q2 → q2=[4,1,2,3]
  swap q1 ↔ q2   → q1 = [4,1,2,3]   (4 is now FRONT = "top of stack")

pop() → dequeue(q1) → 4   ✓ (last pushed, first out — LIFO achieved!)
```

```c
typedef struct {
    Queue* q1;
    Queue* q2;
} StackUsingQueues;

void push(StackUsingQueues* s, int value) {
    enqueue(s->q2, value);
    while (!isEmpty(s->q1))
        enqueue(s->q2, dequeue(s->q1));

    Queue* temp = s->q1;    // swap q1 and q2
    s->q1 = s->q2;
    s->q2 = temp;
}

int pop(StackUsingQueues* s) {
    return dequeue(s->q1);   // front of q1 is always the most recent push
}
```
**Time:** `push` is `O(n)`, `pop` is `O(1)` | **Space:** `O(n)`

---

### 13.2 Implement a Queue Using Two Stacks

**Problem:** The reverse of 13.1 — build FIFO behavior out of two LIFO stacks.

**Idea:** Use one stack (`inStack`) for incoming elements, and another (`outStack`) for outgoing ones. When `outStack` is empty and a `dequeue` is requested, dump all of `inStack` into `outStack` — this **reverses** the order, turning "last pushed, last in `inStack`" into "first pushed, now on top of `outStack`," which is exactly the FIFO order we need.

**Steps (enqueue(x)):** just push `x` onto `inStack` — `O(1)`.

**Steps (dequeue()):**
1. If `outStack` is empty, transfer everything from `inStack` to `outStack` (pop from `inStack`, push onto `outStack`) — this reverses the order.
2. Pop and return the top of `outStack` (now the oldest element).

```
enqueue(1), enqueue(2), enqueue(3):
  inStack (top→bottom): [3,2,1]    outStack: []

dequeue():
  outStack empty → transfer:
    pop inStack(3) → push outStack → outStack:[3]
    pop inStack(2) → push outStack → outStack:[3,2]
    pop inStack(1) → push outStack → outStack:[3,2,1]  (top=1, bottom=3)
  pop outStack → 1   ✓ (first enqueued, first out — FIFO achieved!)

enqueue(4):  inStack:[4]   outStack:[3,2] (unchanged — 1 already removed)

dequeue():
  outStack NOT empty (top=2) → just pop it → 2   ✓ (still correct FIFO order)
```

```c
typedef struct {
    Stack* inStack;
    Stack* outStack;
} QueueUsingStacks;

void enqueue(QueueUsingStacks* q, int value) {
    push(q->inStack, value);
}

int dequeue(QueueUsingStacks* q) {
    if (isEmpty(q->outStack)) {
        while (!isEmpty(q->inStack))
            push(q->outStack, pop(q->inStack));
    }
    return pop(q->outStack);
}
```
**Time:** `enqueue` is `O(1)`; `dequeue` is `O(1)` **amortized** — each element is moved from `inStack` to `outStack` at most once across its whole lifetime, even though a single `dequeue` call can occasionally be `O(n)` | **Space:** `O(n)`

---

### 13.3 Reverse the First K Elements of a Queue

**Problem:** Given a queue and an integer `k`, reverse the order of the first `k` elements, leaving the rest in their original order.

**Idea:** Use a stack as a temporary "order flipper" for just the first `k` elements, then carefully re-append the untouched remainder in its original relative order.

**Steps:**
1. Create an empty stack.
2. Dequeue the first `k` elements from the queue, pushing each onto the stack (this will naturally reverse their order when popped).
3. Pop everything off the stack and enqueue it back into the queue — these `k` elements are now at the **back** of the queue, in reversed order.
4. The remaining `n - k` original elements are still at the front, untouched. Move them, one at a time, from front to back (`dequeue` then `enqueue`) exactly `n - k` times — this rotates them to the end **without changing their relative order**, placing the (already-reversed) first `k` elements back at the front.

```
Queue: [1,2,3,4,5]  k=3   (front=1)

Step 2: dequeue 1,2,3 onto stack:
  stack (top→bottom): [3,2,1]
  queue remaining: [4,5]

Step 3: pop stack, enqueue each:
  pop 3→enqueue   queue: [4,5,3]
  pop 2→enqueue   queue: [4,5,3,2]
  pop 1→enqueue   queue: [4,5,3,2,1]
  (first k are now reversed, but sitting at the BACK)

Step 4: rotate the remaining (n-k=2) original front elements to the back:
  dequeue 4 → enqueue 4:  queue: [5,3,2,1,4]
  dequeue 5 → enqueue 5:  queue: [3,2,1,4,5]

Final: [3,2,1,4,5]  ✓ first 3 reversed (3,2,1), rest unchanged (4,5)
```

```c
void reverseFirstK(Queue* q, int k) {
    Stack* s = createStack(k);

    for (int i = 0; i < k; i++)
        push(s, dequeue(q));

    while (!isEmpty(s))
        enqueue(q, pop(s));

    int remaining = q->count - k;    // the untouched original elements, now at the front
    for (int i = 0; i < remaining; i++)
        enqueue(q, dequeue(q));

    freeStack(s);
}
```
**Time:** `O(n)` | **Space:** `O(k)`

---

### 13.4 Generate Binary Numbers from 1 to N Using a Queue

**Problem:** Print the binary representations of 1 through N, in order, without converting each number with division/modulo.

**Idea:** Binary representations form naturally via a queue: each number's binary string, when you append `"0"` or `"1"` to it, gives you the binary of a later number. Start the queue with `"1"`; each time you dequeue a string, print it, then enqueue `(string + "0")` and `(string + "1")` — this generates the next level of binary strings in increasing numeric order, breadth-first.

**Steps:**
1. Create an empty queue of strings; enqueue `"1"`.
2. Repeat `N` times:
   - Dequeue the front string; print it.
   - Enqueue `(front + "0")`.
   - Enqueue `(front + "1")`.

```
N=5, queue=["1"]

dequeue "1" → print "1"      enqueue "10", "11"   queue=["10","11"]
dequeue "10" → print "10"    enqueue "100","101"  queue=["11","100","101"]
dequeue "11" → print "11"    enqueue "110","111"  queue=["100","101","110","111"]
dequeue "100" → print "100"  enqueue ...           (5 numbers printed, stop here)
dequeue "101" → print "101"

Output: 1, 10, 11, 100, 101   (binary of 1,2,3,4,5 ✓)
```

```c
#include <string.h>

void generateBinaryNumbers(int n) {
    // using a simple char* queue (conceptually — adapt to your Queue-of-strings impl)
    char buffer[2][20];
    int frontIdx = 0;
    strcpy(buffer[0], "1");
    // (illustrative — in real code use a proper queue of malloc'd strings)

    // Conceptual loop (pseudocode-level, since C string-queues need extra plumbing):
    // queue.enqueue("1")
    // for i in 1..n:
    //     s = queue.dequeue(); print(s)
    //     queue.enqueue(s + "0")
    //     queue.enqueue(s + "1")
}
```

**Cleaner in C++ with `std::queue<string>`:**
```cpp
#include <iostream>
#include <queue>
#include <string>
using namespace std;

void generateBinaryNumbers(int n) {
    queue<string> q;
    q.push("1");

    for (int i = 0; i < n; i++) {
        string front = q.front(); q.pop();
        cout << front << " ";

        q.push(front + "0");
        q.push(front + "1");
    }
}
// generateBinaryNumbers(5) → 1 10 11 100 101
```
**Time:** `O(n)` | **Space:** `O(n)`

---

### 13.5 Sliding Window Maximum Using a Deque

**Problem:** Given an array and window size `k`, find the maximum element in every window of size `k` as it slides from left to right.

**Idea:** Use a `std::deque` to store **indices** of array elements, kept in an order such that the **front always holds the index of the current window's maximum**. Maintain two invariants as you slide:
- Remove indices from the **front** that have fallen out of the current window (too old).
- Remove indices from the **back** whose values are `<=` the new element — they can never be the max of any future window once a bigger element has appeared to their right.

**Steps:**
1. Create an empty deque (will store array **indices**, not values).
2. For each index `i` in the array:
   - While the deque's front index is `<= i - k` (i.e., outside the window), pop it from the front.
   - While the deque's back holds a value `<= arr[i]`, pop it from the back (it's now useless).
   - Push `i` onto the back.
   - Once `i >= k - 1` (the window is fully formed), the front of the deque holds the index of the max — record `arr[front]`.

```
arr = [1, 3, -1, -3, 5, 3, 6, 7],  k = 3

i=0 (val=1): deque empty → push 0.               deque:[0]
i=1 (val=3): back(0)->val=1<=3 → pop; push 1.     deque:[1]
i=2 (val=-1): back(1)->val=3, NOT<=-1 → keep; push 2.  deque:[1,2]
   window [1,3,-1] complete (i>=k-1=2) → max = arr[front=1] = 3

i=3 (val=-3): front(1) <= i-k=0? 1<=0? no, keep.
   back(2)->val=-1, NOT<=-3 → keep; push 3.         deque:[1,2,3]
   window [3,-1,-3] → max = arr[front=1] = 3

i=4 (val=5): front(1) <= i-k=1? 1<=1? YES → pop front.  deque:[2,3]
   back(3)->val=-3<=5→pop; back(2)->val=-1<=5→pop; deque empty → push 4.  deque:[4]
   window [-1,-3,5] → max = arr[front=4] = 5

... and so on.
Result so far: [3, 3, 5, ...]
```

```cpp
#include <iostream>
#include <deque>
#include <vector>
using namespace std;

vector<int> maxSlidingWindow(vector<int>& arr, int k) {
    deque<int> dq;     // stores INDICES
    vector<int> result;

    for (int i = 0; i < (int)arr.size(); i++) {
        // remove indices out of this window
        while (!dq.empty() && dq.front() <= i - k)
            dq.pop_front();

        // remove indices whose values are useless (smaller than current)
        while (!dq.empty() && arr[dq.back()] <= arr[i])
            dq.pop_back();

        dq.push_back(i);

        if (i >= k - 1)
            result.push_back(arr[dq.front()]);
    }
    return result;
}
// maxSlidingWindow([1,3,-1,-3,5,3,6,7], 3) → [3,3,5,5,6,7]
```
**Time:** `O(n)` — each index is pushed and popped from the deque **at most once** total (same amortized argument as "Next Greater Element" in the Stack tutorial) | **Space:** `O(k)`

---

### 13.6 Level-Order Traversal of a Binary Tree (BFS) Using a Queue

**Problem:** Given a binary tree, print its nodes level by level (top to bottom, left to right within each level) — the textbook motivating example for queues.

**Idea:** A queue naturally preserves "discovery order." Start by enqueueing the root. Repeatedly dequeue a node, visit it, then enqueue its children (left, then right) — children are always enqueued **after** all nodes of the current level, which is exactly what makes the traversal come out level by level.

**Steps:**
1. If the tree is empty, nothing to print.
2. Create a queue; enqueue the root node.
3. While the queue is not empty:
   - Dequeue a node; print/visit it.
   - If it has a left child, enqueue it.
   - If it has a right child, enqueue it.

```
        1
       / \
      2   3
     / \   \
    4   5   6

queue=[1]
dequeue 1, visit "1", enqueue 2,3     queue=[2,3]
dequeue 2, visit "2", enqueue 4,5     queue=[3,4,5]
dequeue 3, visit "3", enqueue 6       queue=[4,5,6]
dequeue 4, visit "4" (no children)    queue=[5,6]
dequeue 5, visit "5" (no children)    queue=[6]
dequeue 6, visit "6" (no children)    queue=[]

Output: 1 2 3 4 5 6   ✓ level by level, left to right
```

```c
typedef struct TreeNode {
    int data;
    struct TreeNode *left, *right;
} TreeNode;

void levelOrderTraversal(TreeNode* root) {
    if (root == NULL) return;

    Queue q;
    initQueue(&q);
    enqueue(&q, (long)root);   // (in real code, store TreeNode* in the queue, not int!)

    while (!isEmpty(&q)) {
        TreeNode* node = (TreeNode*) dequeue(&q);
        printf("%d ", node->data);

        if (node->left)  enqueue(&q, (long)node->left);
        if (node->right) enqueue(&q, (long)node->right);
    }
}
```
> **Note:** our earlier `Queue` examples stored `int`s for simplicity. For a real tree-traversal queue, you'd define `Queue` to hold `void*` or `TreeNode*` elements directly — shown as a cast here just to reuse the same queue structure.

**Cleaner in C++ with `std::queue<TreeNode*>`:**
```cpp
#include <queue>
void levelOrderTraversal(TreeNode* root) {
    if (!root) return;
    queue<TreeNode*> q;
    q.push(root);

    while (!q.empty()) {
        TreeNode* node = q.front(); q.pop();
        cout << node->data << " ";
        if (node->left)  q.push(node->left);
        if (node->right) q.push(node->right);
    }
}
```
**Time:** `O(n)` — every node enqueued and dequeued exactly once | **Space:** `O(n)` worst case (a very wide tree, e.g. a complete tree's last level)

---

## 14. Quick Reference

```c
// Array-based Circular Queue (dynamically allocated)
typedef struct
{
    int *arr;      // array to store elements
    int capacity;  // maximum size of queue
    int front;     // index of front (oldest) element
    int rear;      // index of rear (newest) element
    int count;     // current number of elements
} Queue;

// Linked-List-based Queue
typedef struct Node {
    int data;
    struct Node* next;
} Node;

typedef struct {
    Node* front;   // oldest element (dequeue here)
    Node* rear;    // newest element (enqueue here)
    int count;
} Queue;
```

```cpp
// STL
#include <queue>   // std::queue (FIFO adapter) and std::priority_queue (heap adapter)
#include <deque>   // std::deque (real container, O(1) both ends + random access)
```

| Operation | Hand-built Queue | `std::queue` | `std::deque` | `std::priority_queue` |
|---|---|---|---|---|
| Add | `enqueue(value)` | `q.push(value)` | `d.push_back(value)` / `push_front` | `pq.push(value)` |
| Remove | `dequeue()` → returns value | `q.pop()` → **void!** use `front()` first | `d.pop_back()`/`pop_front()` → void | `pq.pop()` → **void!** use `top()` first |
| Look at front | `peekFront()` | `q.front()` | `d.front()` | `pq.top()` |
| Is empty? | `isEmpty()` | `q.empty()` | `d.empty()` | `pq.empty()` |
| Count | `size()` | `q.size()` | `d.size()` | `pq.size()` |

| Need | Use |
|---|---|
| Strict FIFO discipline, nothing else exposed | `std::queue` |
| Both-end insert/remove AND random access | `std::deque` |
| Highest-priority-first processing (not arrival order) | `std::priority_queue` |
| Fixed-capacity, cache-friendly, no malloc per element | Circular array queue (hand-built) |
| Unknown/unbounded size | Linked-list queue (hand-built) |
