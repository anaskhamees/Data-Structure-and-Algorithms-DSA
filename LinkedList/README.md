# Linked List — Complete Tutorial

A complete guide covering **Singly Linked Lists**, **Circular Linked Lists**, **Doubly Linked Lists**, **Linked Lists in STL (`std::list`, `std::forward_list`)**, and fully solved practice problems.

---

## Table of Contents
1. [Why Linked List?](#1-why-linked-list)
2. [Array vs Linked List](#2-array-vs-linked-list)
3. [Singly Linked List — Basics](#3-singly-linked-list--basics)
4. [Operations on Singly Linked List](#4-operations-on-singly-linked-list)
5. [Circular Linked List — Full Operations](#5-circular-linked-list--full-operations)
6. [Doubly Linked List — Full Operations](#6-doubly-linked-list--full-operations)
7. [`std::list` — Full STL Tutorial](#7-stdlist--full-stl-tutorial)
8. [`std::forward_list` — Full STL Tutorial](#8-stdforward_list--full-stl-tutorial)
9. [Complexity Summary](#9-complexity-summary)
10. [Practice Examples — Solved in Detail](#10-practice-examples--solved-in-detail)
11. [Quick Reference](#11-quick-reference)

---

## 1. Why Linked List?

### Problems with Arrays
- Arrays have a **fixed size at compile time**.
- We **cannot increase/decrease** the size at runtime.
- If we want to store more elements than the fixed size allows **at runtime**, we're stuck.

### When to use Array vs Linked List

| Use **Arrays** when... | Use **Linked List** when... |
|---|---|
| You know the number of elements in advance (e.g. number of students) | Size needs to grow/shrink dynamically over time |
| You need fast random access → `O(1)` | Frequent insertion/deletion → `O(1)` (at a known position) |
| Memory needs to be compact and contiguous | Memory allocated only when needed (no empty/wasted space) |
| You mostly read/write/modify elements, rarely insert/delete | — |

---

## 2. Array vs Linked List

```
Array (contiguous memory, fixed size):
┌─────┬─────┬─────┬─────┬─────┐
│  5  │  12 │  7  │  9  │  3  │   <- one fixed-size block, index-addressable
└─────┴─────┴─────┴─────┴─────┘
  idx0  idx1  idx2  idx3  idx4

Linked List (scattered memory, dynamic size):
┌─────┬──────┐    ┌─────┬──────┐    ┌─────┬──────┐
│  5  │ 270 ─┼───▶│ 12  │ 360 ─┼───▶│  7  │ NULL │
└─────┴──────┘    └─────┴──────┘    └─────┴──────┘
 addr 100          addr 270          addr 360
```

**Why insertion is `O(1)` in a linked list but `O(n)` in an array:**
```
Array — insert 99 at index 1:           Linked List — insert 99 after node "5":
[5][12][7][9][3]                        [5]──▶[12]──▶[7]──▶[9]──▶[3]
        shift ──▶                              │
[5][99][12][7][9][3]                           └──▶[99]──▶[12]──▶[7]──▶[9]──▶[3]
 Every element after index 1            Only 2 pointers change — no shifting!
 must move one slot right. O(n)         O(1), once you're at the right node.
```

---

## 3. Singly Linked List — Basics

### What is a Linked List?
A **collection of nodes**, where each node contains:
- `data` → the value
- `next` → pointer/address of the next node

```
head/first
   │
   ▼
┌─────┬──────┐    ┌─────┬──────┐    ┌─────┬──────┐    ┌─────┬──────┐
│ 200 │      │───▶│  8  │ 210 ─┼───▶│  3  │ 270 ─┼───▶│  7  │ NULL │
└─────┴──────┘    └─────┴──────┘    └─────┴──────┘    └─────┴──────┘
 addr 100          addr 200          addr 210           addr 270
```

### What is a Node? (self-referential structure)

```c
struct Node {
    int data;
    struct Node* next;
};
```

```
┌──────┬──────┐
│ data │ next │  ──▶ pointer to next node
└──────┴──────┘
```

### Using `typedef` (common shortcut)

```c
typedef struct Node {
    int data;
    struct Node* next;   // must be "struct Node*" here — the struct
                          // can't be anonymous since it refers to itself
} Node;
```

### Declaring & Allocating a Node

```c
// In C
struct Node* p;
p = (struct Node*) malloc(sizeof(struct Node));

// In C++
Node* p = new Node;
```

```
Stack:          Heap:
┌─────┐         ┌──────┬──────┐
│ p   │────────▶│  10  │  0   │
│ 200 │         └──────┴──────┘
└─────┘          addr 200
```

### Initializing / Accessing a Node

```c
p->data = 10;
p->next = 0;     // same as NULL
```

### Useful pointer checks

```c
Ptr == NULL        // same as: Ptr == 0   /  !Ptr        → Ptr points to nothing
Ptr != NULL        // same as: Ptr != 0   /  Ptr          → Ptr points to another node
Ptr->next == NULL  // Ptr points to the LAST node
Ptr->next != NULL  // there are more nodes after Ptr
```

### Visualizing Pointer Updates (the core mental model)

```
Before: A ──▶ B ──▶ C ──▶ NULL
Goal: insert X between A and B

Step 1 — X borrows B's link:
A ──▶ B ──▶ C ──▶ NULL
       ▲
       │
       X  (X->next = B)

Step 2 — A points to X instead of B:
A ──▶ X ──▶ B ──▶ C ──▶ NULL
      (A->next = X)

Order matters! If you did A->next = X FIRST,
you'd lose the only pointer to B before X could grab it.
```

---

## 4. Operations on Singly Linked List

Example list used below:

```
First
┌─────┐   ┌───┬─────┐   ┌───┬─────┐   ┌───┬─────┐   ┌───┬──────┐
│ 200 │──▶│ 8 │ 210 │──▶│ 3 │ 270 │──▶│ 7 │ 300 │──▶│ 9 │ NULL │
└─────┘   └───┴─────┘   └───┴─────┘   └───┴─────┘   └───┴──────┘
 100       200           210           270            300
```

### 4.1 Create a Node

**Steps:**
1. Allocate memory for one node (`malloc`).
2. Store the given value in `data`.
3. Set `next` to `NULL` (it's not linked to anything yet).
4. Return the pointer to this new node.

```c
Node* createNode(int data) {
    Node* newNode = (Node*) malloc(sizeof(Node));
    newNode->data = data;
    newNode->next = NULL;
    return newNode;
}
```

### 4.2 Create the Whole List (from an array)

**Steps:**
1. Create the first node from `arr[0]` and make it `head`.
2. Keep a `temp` pointer starting at `head` (it will walk forward).
3. For each remaining element `arr[1..n-1]`:
   - Create a new node for it.
   - Link `temp->next` to that new node.
   - Move `temp` forward to the new node.
4. Return `head` (the start of the list).

```c
Node* createLinkedList(int* arr, int n) {
    Node* head = createNode(arr[0]);
    Node* temp = head;

    for (int i = 1; i < n; i++) {
        temp->next = createNode(arr[i]);
        temp = temp->next;
    }
    return head;
}
```
**Time:** `O(n)` | **Space:** `O(1)` extra (besides the list itself)

### 4.3 Display (Iterative)

**Steps:**
1. Start a traversal pointer `temp` at `head`.
2. While `temp` is not `NULL`:
   - Print `temp->data`.
   - Move `temp` to `temp->next`.
3. Stop when `temp` becomes `NULL` (end of list reached).

```c
void displayList(Node* head) {
    Node* temp = head;
    while (temp != NULL) {
        printf("%d ", temp->data);
        temp = temp->next;
    }
}
```
**Time:** `O(n)` | **Space:** `O(1)`

### 4.3b Display (Recursion)

**Steps:**
1. Base case: if `head == NULL`, there's nothing to print — stop.
2. Otherwise: print `head->data` first.
3. Then recursively call `display` on `head->next`.
(Because the print happens *before* the recursive call, the output order is the natural order 8, 3, 7, 9 — not reversed.)

```c
void display(Node* head) {
    if (head != NULL) {
        printf("%d ", head->data);   // 1. print first
        display(head->next);          // 2. then recurse
    }
}
```
**Time:** `O(n)` | **Space:** `O(n)` (call stack)

```
display(200)
   8 ─┐
      └─▶ display(210)
             3 ─┐
                 └─▶ display(270)
                        7 ─┐
                            └─▶ display(300)
                                   9 ─┐
                                       └─▶ display(NULL)  ✕ base case
```

### 4.4 Counting Nodes

**Steps (Iterative):**
1. Start a `counter` at 0.
2. While `head` is not `NULL`:
   - Increment `counter`.
   - Move `head` to `head->next`.
3. Return `counter`.

```c
int count(Node* head) {
    int counter = 0;
    while (head != NULL) {
        counter++;
        head = head->next;
    }
    return counter;
}
```

**Steps (Recursive):**
1. Base case: if `node == NULL`, return 0 (an empty list has 0 nodes).
2. Otherwise: return `1 + count(node->next)` — "me, plus however many nodes come after me."

```c
int count(Node* node) {
    if (node == NULL)
        return 0;
    else
        return count(node->next) + 1;
}
```
```
count(200)=5
 └─ count(210)+1 = 4+1 = 5
     └─ count(270)+1 = 3+1 = 4
         └─ count(300)+1 = 2+1 = 3
             └─ count(350)+1 = 1+1 = 2
                 └─ count(NULL)+1 = 0+1 = 1
```
**Time:** `O(n)` | **Space:** iterative `O(1)`, recursive `O(n)`

### 4.5 Sum of Elements

**Steps (Iterative):**
1. Start `sum` at 0.
2. While `head` is not `NULL`: add `head->data` to `sum`, then move `head` forward.
3. Return `sum`.

```c
int getSum(Node* head) {
    int sum = 0;
    while (head) {
        sum += head->data;
        head = head->next;
    }
    return sum;
}
```

**Steps (Recursive):**
1. Base case: empty list (`head == NULL`) sums to 0.
2. Otherwise: return `head->data + getSum(head->next)` — "my value, plus the sum of everything after me."

```c
int getSum(Node* head) {
    if (head == NULL)
        return 0;
    else
        return getSum(head->next) + head->data;
}
```

### 4.6 Maximum Value

**Steps:**
1. Start `max` at the smallest possible integer (`MIN_INT`).
2. While `head` is not `NULL`:
   - If `head->data > max`, update `max`.
   - Move `head` forward.
3. Return `max`.

```c
int getMax(Node* head) {
    int max = MIN_INT;
    while (head) {
        if (head->data > max)
            max = head->data;
        head = head->next;
    }
    return max;
}
```

### 4.7 Search

Only **linear search** is possible in a linked list (no random access, so binary search gives no benefit).

**Steps (Iterative):**
1. Walk through the list node by node.
2. At each node, compare `head->data` to `key`.
3. If they match, return that node (found).
4. If the loop ends (`head == NULL`) without a match, return `NULL` (not found).

```c
Node* search(Node* head, int key) {
    while (head != NULL) {
        if (key == head->data)
            return head;
        head = head->next;
    }
    return NULL;
}
```

**Steps (Recursive):** same idea — base case `NULL` → not found; match → return node; else recurse on `head->next`.

```c
Node* search(Node* head, int key) {
    if (head == NULL)
        return NULL;
    if (key == head->data)
        return head;
    return search(head->next, key);
}
```

**Optimization — Move to Front:** every time a node is found, move it to the head, so frequently-searched items become faster to find next time.

**Steps:**
1. Keep two pointers: `temp` (current node) and `prev` (the one before it, starts `NULL`).
2. Walk forward comparing `temp->data` to `key`.
3. On a match:
   - If `prev == NULL`, the match is already the first node — just return it.
   - Otherwise, unlink `temp` from its current spot: `prev->next = temp->next`.
   - Re-link `temp` at the front: `temp->next = *head`, then `*head = temp`.
   - Return `temp`.
4. If no match is found by the end, return `NULL`.

```c
Node* search(Node** head, int key) {
    Node* temp = *head;
    Node* prev = NULL;

    while (temp) {
        if (temp->data == key) {
            if (prev == NULL) return temp;   // already first node
            prev->next = temp->next;          // unlink temp
            temp->next = *head;               // move temp to front
            *head = temp;
            return temp;
        }
        prev = temp;
        temp = temp->next;
    }
    return NULL;
}
```

### 4.8 Free the Whole List

**Steps:**
1. While `head` is not `NULL`:
   - Save the current node in `temp`.
   - Move `head` forward (`head = head->next`) *before* freeing — otherwise you'd lose the rest of the list.
   - Free `temp`.
2. Loop ends when every node has been freed.

```c
void freeList(Node* head) {
    Node* temp;
    while (head) {
        temp = head;
        head = head->next;
        free(temp);
    }
}
```

### 4.9 Insertion

**(A) Insert at Head**

**Steps:**
1. Create the new node.
2. Point its `next` to the current `head` (so it connects to the rest of the list).
3. Update `head` to point to the new node (it's now first).

```
Before:  head ──▶ [8] ──▶ [3] ──▶ NULL
Insert 5 at head:
  newNode[5] ──▶ [8]   (step 2: newNode->next = head)
  head = newNode       (step 3)
After:   head ──▶ [5] ──▶ [8] ──▶ [3] ──▶ NULL
```

```c
void insertAtHead(Node** head, int data) {
    Node* newNode = createNode(data);
    newNode->next = *head;
    *head = newNode;
}
```

>##### An alternative: return the new head
>
>You don't always need a double pointer. You can return the updated head instead:
>
>```c
>Node* insertAtHead(Node* head, int data) {
>    Node* newNode = createNode(data);
>
>    if (newNode == NULL)
>        return head;
>
>    newNode->next = head;
>    return newNode;
>}
>```
>
>Call it like this:
>
>```c
>head = insertAtHead(head, 5);
>```
>
>Here, the caller receives the new head through the return value.
>
>The key concept: A single pointer lets you modify the nodes it points to. A double pointer lets you modify the caller's pointer variable itself. Returning a pointer is another way to communicate a changed head back to the caller.
>
>- ##### Example with the bug: single pointer
>
>```c
>#include <stdio.h>
>#include <stdlib.h>
>
>typedef struct Node {
>    int data;
>    struct Node* next;
>} Node;
>
>Node* createNode(int data) {
>    Node* newNode = malloc(sizeof(Node));
>
>    if (newNode == NULL) {
>        printf("Memory allocation failed\n");
>        exit(1);
>    }
>
>    newNode->data = data;
>    newNode->next = NULL;
>
>    return newNode;
>}
>
>void printList(Node* head) {
>    while (head != NULL) {
>        printf("%d -> ", head->data);
>        head = head->next;
>    }
>    printf("NULL\n");
>}
>
>// BUG: head is passed by value (a copy of the pointer)
>void insertAtHead(Node* head, int data) {
>    Node* newNode = createNode(data);
>
>    newNode->next = head;
>    head = newNode;  // Changes only the local copy!
>}
>
>int main(void) {
>    Node* head = createNode(10);
>    head->next = createNode(20);
>    head->next->next = createNode(30);
>
>    printf("Before insertion: ");
>    printList(head);
>
>    insertAtHead(head, 5);
>
>    printf("After insertion:  ");
>    printList(head);
>
>    return 0;
>}
>
>```



**(B) Insert at End**

**Steps:**

1. Create the new node.
2. If the list is empty (`*head == NULL`), the new node simply becomes `head` — done.
3. Otherwise, traverse with `temp` until `temp->next == NULL` (the last node).
4. Link the last node to the new node: `temp->next = newNode`.
5. Make sure the new node's `next` is `NULL` (it's now the last one).

```c
void insertAtEnd(Node** head, int data) {
    Node* newNode = createNode(data);
    if (*head == NULL) {
        *head = newNode;
        return;
    }
    Node* temp = *head;
    while (temp->next)
        temp = temp->next;
    temp->next = newNode;
    newNode->next = NULL;
}
```

**(C) Insert at Position (1-based)**

**Steps:**
1. Special case: if `pos == 1`, just reuse "insert at head" and return.
2. Otherwise, create the new node.
3. Traverse with `temp` to the node **just before** the target position (i.e. `pos - 1` steps from head), stopping early if `temp` becomes `NULL`.
4. If `temp` is `NULL`, the position is out of range — free the unused node and stop.
5. Otherwise, link the new node in between: `newNode->next = temp->next`, then `temp->next = newNode`.

```
Insert 99 at pos=3 in: [8]→[3]→[7]→[9]→NULL
Walk to node BEFORE pos 3 → that's [3] (position 2)
  newNode->next = temp->next   → 99 ──▶ [7]
  temp->next    = newNode      → [3] ──▶ 99
Result: [8]→[3]→[99]→[7]→[9]→NULL
```

```c
void insertAtPos(Node** head, int data, int pos) {
    if (pos == 1) {
        insertAtHead(head, data);
        return;
    }

    Node* temp = *head;
    Node* newNode = createNode(data);

    for (int i = 1; i < pos - 1 && temp; i++)
        temp = temp->next;

    if (temp == NULL) {
        printf("out of range");
        free(newNode);
        return;
    }

    newNode->next = temp->next;
    temp->next = newNode;
}
```

### 4.10 Deletion (by Position, 1-based)

**Steps:**
1. **Validate `pos`:** if `pos <= 0`, the position doesn't exist (positions start at 1) — print an error and return.
2. If the list is empty, there's nothing to delete — return.
3. Special case: if `pos == 1`, the head node itself is being deleted:
   - Save old head in `temp`.
   - Move `head` to `head->next`.
   - Free `temp`.
   - Return.
4. Otherwise, traverse with `temp` to the node **just before** the target position.
5. If `temp` is `NULL` or `temp->next` is `NULL`, the position is out of range — stop.
6. Otherwise: save `temp->next` as `nodeToDelete`, re-link `temp->next = nodeToDelete->next` (skip over it), then free `nodeToDelete`.

```c
void deleteAtPos(Node** head, int pos) {
    if (pos <= 0) {                        // FIX: reject pos <= 0
        printf("Invalid position\n");
        return;
    }

    if (*head == NULL) return;             // empty list

    Node* temp = *head;

    if (pos == 1) {                        // delete head node
        *head = temp->next;
        free(temp);
        return;
    }

    for (int i = 1; i < pos - 1 && temp; i++)
        temp = temp->next;

    if (temp == NULL || temp->next == NULL)  // out of range
        return;

    Node* nodeToDelete = temp->next;
    temp->next = nodeToDelete->next;
    free(nodeToDelete);
}
```

---

## 5. Circular Linked List — Full Operations

In a circular linked list, the **last node points back to the first node** instead of `NULL`.

### Representation 1 — head points directly to first data node
```
head
  │
  ▼
┌───┬────┐   ┌───┬────┐   ┌───┬────┐
│ 7 │  ● │──▶│ 4 │  ● │──▶│ 5 │  ● │──┐
└───┴────┘   └───┴────┘   └───┴────┘  │
  ▲                                   │
  └───────────────────────────────────┘

One-node list:        Empty list:
┌──────┬───┐
│ data │ ● │──┐        head → NULL
└──────┴───┘  │
     ▲		  │
     └────────┘       (the single node points to itself)
```

### Representation 2 — head is a dummy/sentinel node
```
Head
┌───┬───┐   ┌───┬───┐   ┌───┬───┐   ┌───┬───┐
│   │ ● │──▶│ 4 │ ● │──▶│ 5 │ ● │──▶│ 7 │ ● │──┐
└───┴───┘   └───┴───┘   └───┴───┘   └───┴───┘  │
               ▲                               │
  	           └───────────────────────────────┘
```
> The rest of this tutorial uses **Representation 1** (no sentinel) — the same style as the singly linked list, just circular.

### 5.1 Create Node
Identical to the singly linked list's `createNode` (section 4.1) — a circular list uses the same node structure; only the *linking* is different.

### 5.2 Create a Circular Linked List

**Steps:**
1. Create the first node from `arr[0]`, call it `head`.
2. Walk through the rest of the array, creating and appending nodes one by one (same as a normal singly linked list).
3. The key extra step: once all nodes are linked, point the **last** node's `next` back to `head` — this is what makes it circular.

```c
Node* createCLL(int* arr, int n) {
    Node* head = createNode(arr[0]);
    Node* temp = head;

    for (int i = 1; i < n; i++) {
        Node* newNode = createNode(arr[i]);
        temp->next = newNode;
        temp = newNode;
    }
    temp->next = head;   // <-- makes it circular
    return head;
}
```

### 5.3 Display (iterative, `do...while`)

**Steps:**
1. Start `temp` at `head`.
2. Print `temp->data`, then move `temp` forward — do this **at least once** before checking the condition (that's why it's a `do...while`, not a `while`).
3. Keep looping until `temp` comes back around to `head` (a full circle).

```c
void display(Node* head) {
    if (head == NULL) return;        // empty list guard
    Node* temp = head;
    do {
        printf("%d ", temp->data);
        temp = temp->next;
    } while (temp != head);
}
```
> Must use `do...while` (not `while`) because the loop condition `temp != head` is false on the **first** iteration too (`temp` IS `head` at the start).

### 5.4 Display (recursion, using a static flag)

```c
void display(Node* head) {
    static int flag = 0;
    static Node* temp = NULL;
    if (temp == NULL) temp = head;

    if (temp != head || flag == 0) {
        flag = 1;
        printf("%d ", temp->data);
        temp = temp->next;
        display(head);
    }
    flag = 0;
    temp = NULL;
}
```

### 5.5 Counting Nodes

**Steps:** identical idea to singly — but because there's no `NULL`, you must stop when you come back to `head`, using a `do...while`.

```c
int count(Node* head) {
    if (head == NULL) return 0;
    int counter = 0;
    Node* temp = head;
    do {
        counter++;
        temp = temp->next;
    } while (temp != head);
    return counter;
}
```

### 5.6 Sum of Elements

```c
int getSum(Node* head) {
    if (head == NULL) return 0;
    int sum = 0;
    Node* temp = head;
    do {
        sum += temp->data;
        temp = temp->next;
    } while (temp != head);
    return sum;
}
```

### 5.7 Maximum Value

```c
int getMax(Node* head) {
    if (head == NULL) return MIN_INT;
    int max = head->data;
    Node* temp = head->next;
    while (temp != head) {
        if (temp->data > max) max = temp->data;
        temp = temp->next;
    }
    return max;
}
```

### 5.8 Search

**Steps:** same linear scan, but loop bounded by returning to `head` instead of hitting `NULL`.

```c
Node* search(Node* head, int key) {
    if (head == NULL) return NULL;
    Node* temp = head;
    do {
        if (temp->data == key) return temp;
        temp = temp->next;
    } while (temp != head);
    return NULL;   // not found
}
```

### 5.9 Insertion

**(A) Insert at Head**

**Steps:**
1. Create the new node.
2. If list is empty: new node points to itself, becomes `head`.
3. Otherwise: find the **last node** (the one whose `next == head`) by walking the circle.
4. New node's `next` = old `head`.
5. Last node's `next` = new node.
6. `head` = new node.

```
Before: head→[7]→[4]→[5]─┐
              ▲──────────┘
Insert 2 at head:
  last node [5]->next = newNode[2]
  newNode[2]->next = old head [7]
  head = newNode[2]
After: head→[2]→[7]→[4]→[5]─┐
              ▲─────────────┘
```

```c
void insertAtHead(Node** head, int data) {
    Node* newNode = createNode(data);
    if (*head == NULL) {
        newNode->next = newNode;   // points to itself
        *head = newNode;
        return;
    }
    Node* last = *head;
    while (last->next != *head)
        last = last->next;

    newNode->next = *head;
    last->next = newNode;
    *head = newNode;
}
```

**(B) Insert at End**

**Steps:**
1. Create the new node.
2. If empty, same as insert-at-head (single self-pointing node).
3. Otherwise, find the last node (`last->next == head`).
4. New node's `next` = `head` (closes the circle).
5. Last node's `next` = new node.

```c
void insertAtEnd(Node** head, int data) {
    Node* newNode = createNode(data);
    if (*head == NULL) {
        newNode->next = newNode;
        *head = newNode;
        return;
    }
    Node* last = *head;
    while (last->next != *head)
        last = last->next;

    last->next = newNode;
    newNode->next = *head;
}
```

**(C) Insert at Position (1-based)**

**Steps:**
1. If `pos == 1`, call insert-at-head.
2. Otherwise, traverse `pos - 2` steps from `head` to reach the node **before** the target position.
3. Link new node in between (same splicing as singly linked list), but be careful: if you wrap all the way around without finding the spot, position is invalid.

```c
void insertAtPos(Node** head, int data, int pos) {
    if (pos <= 0) { printf("Invalid position\n"); return; }
    if (pos == 1) { insertAtHead(head, data); return; }

    Node* temp = *head;
    for (int i = 1; i < pos - 1; i++) {
        temp = temp->next;
        if (temp == *head) { printf("out of range\n"); return; }
    }
    Node* newNode = createNode(data);
    newNode->next = temp->next;
    temp->next = newNode;
}
```

### 5.10 Deletion

**(A) Delete at Head**

**Steps:**
1. If empty, nothing to do.
2. If only one node (`head->next == head`), free it, set `head = NULL`.
3. Otherwise: find the last node, point `last->next` to `head->next` (skip old head), free old head, update `head`.

```c
void deleteAtHead(Node** head) {
    if (*head == NULL) return;

    if ((*head)->next == *head) {   // only one node
        free(*head);
        *head = NULL;
        return;
    }

    Node* last = *head;
    while (last->next != *head)
        last = last->next;

    Node* toDelete = *head;
    *head = (*head)->next;
    last->next = *head;
    free(toDelete);
}
```

**(B) Delete at End**

**Steps:**
1. If empty, nothing to do. If one node, same as delete-at-head.
2. Otherwise, traverse to find the **second-to-last** node (`prev`) and the **last** node.
3. `prev->next = head` (closes the circle, skipping the last node).
4. Free the last node.

```c
void deleteAtEnd(Node** head) {
    if (*head == NULL) return;
    if ((*head)->next == *head) {
        free(*head);
        *head = NULL;
        return;
    }
    Node* prev = *head;
    while (prev->next->next != *head)
        prev = prev->next;

    Node* toDelete = prev->next;
    prev->next = *head;
    free(toDelete);
}
```

**(C) Delete at Position (1-based)**

```c
void deleteAtPos(Node** head, int pos) {
    if (pos <= 0 || *head == NULL) { printf("Invalid\n"); return; }
    if (pos == 1) { deleteAtHead(head); return; }

    Node* prev = *head;
    for (int i = 1; i < pos - 1; i++) {
        prev = prev->next;
        if (prev == *head) { printf("out of range\n"); return; }
    }
    if (prev->next == *head) { printf("out of range\n"); return; }

    Node* toDelete = prev->next;
    prev->next = toDelete->next;
    free(toDelete);
}
```

### 5.11 Free the Whole Circular List

**Steps:** you cannot simply loop "while not NULL" — you must break the circle first (or count nodes and free that many times), otherwise you'd loop forever.

```c
void freeCLL(Node** head) {
    if (*head == NULL) return;
    Node* curr = (*head)->next;
    while (curr != *head) {
        Node* temp = curr;
        curr = curr->next;
        free(temp);
    }
    free(*head);
    *head = NULL;
}
```

---

## 6. Doubly Linked List — Full Operations

Each node has **two** pointers: one to the previous node, one to the next node.

```
NULL◁──┬────┬────┬───▶    ◁──┬────┬────┬───▶    ◁──┬────┬────┬───▶   ◁──┬────┬────┬──▶NULL
       │prev│ 6  │next       │prev│ 9  │next       │prev│ 10 │next      │prev│ 2  │next
       └────┴────┴───        └────┴────┴───        └────┴────┴───       └────┴────┴───
```

### Node Structure
```c
typedef struct Node {
    struct Node* prev;
    int data;
    struct Node* next;
} Node;
```

### 6.1 Create a Node

**Steps:**
1. Allocate memory for the new node.
2. If allocation failed (`malloc` returned `NULL`), return `NULL` to signal an error.
3. Store the value in `data`.
4. Set both `next` and `prev` to `NULL` (not linked to anything yet).
5. Return the new node.

```c
Node* createNode(int data) {
    Node* newNode = (Node*) malloc(sizeof(Node));
    if (!newNode) return NULL;     // error
    newNode->data = data;
    newNode->next = NULL;
    newNode->prev = NULL;
    return newNode;
}
```

### 6.2 Create the Whole DLL (from array)

**Steps:**
1. Create the first node from `arr[0]`, set it as `head`, and `temp = head`.
2. For each remaining element:
   - Create a new node.
   - Link forward: `temp->next = newNode`.
   - Link backward: `newNode->prev = temp` (the extra step compared to a singly linked list).
   - Advance `temp = newNode`.
3. Return `head`.

```c
Node* createDLL(int* arr, int n) {
    Node* head = createNode(arr[0]);
    Node* temp = head;

    for (int i = 1; i < n; i++) {
        Node* newNode = createNode(arr[i]);
        temp->next = newNode;
        newNode->prev = temp;
        temp = newNode;
    }
    return head;
}
```

### 6.3 Display

**Forward:**
```c
void display(Node* head) {
    while (head) {
        printf("%d ", head->data);
        head = head->next;
    }
}
```

**Steps (Reverse):**
1. First walk **forward** from `head` until you reach the last node (`tail`, where `tail->next == NULL`).
2. Then walk **backward** from `tail`, printing each node's data and following `prev`, until you fall off the front (`tail == NULL`).

```c
void displayReverse(Node* head) {
    if (head == NULL) return;
    Node* tail = head;
    while (tail->next)
        tail = tail->next;

    while (tail) {
        printf("%d ", tail->data);
        tail = tail->prev;
    }
}
```

### 6.4 Counting Nodes
```c
int count(Node* head) {
    int c = 0;
    while (head) { c++; head = head->next; }
    return c;
}
```

### 6.5 Sum of Elements
```c
int getSum(Node* head) {
    int sum = 0;
    while (head) { sum += head->data; head = head->next; }
    return sum;
}
```

### 6.6 Maximum Value
```c
int getMax(Node* head) {
    if (head == NULL) return MIN_INT;
    int max = head->data;
    while (head) {
        if (head->data > max) max = head->data;
        head = head->next;
    }
    return max;
}
```

### 6.7 Search (forward)
```c
Node* search(Node* head, int key) {
    while (head) {
        if (head->data == key) return head;
        head = head->next;
    }
    return NULL;
}
```
> A doubly linked list CAN also search backward from a known tail — useful if the key is more likely near the end.

### 6.8 Insertion

**(A) At Beginning**
```
If list is empty:
  1. newNode->next = NULL
  2. newNode->prev = NULL
  3. head = newNode

If list is NOT empty:
  1. newNode->next = head
  2. newNode->prev = NULL
  3. head->prev    = newNode
  4. head          = newNode
```
```
Before: head→[6]⇄[9]⇄[10]→NULL
Insert 3 at beginning:
  newNode[3]->next = head[6]
  head[6]->prev = newNode[3]
  head = newNode[3]
After: head→[3]⇄[6]⇄[9]⇄[10]→NULL
```
```c
void insertAtBeginning(Node** head, int data) {
    Node* newNode = createNode(data);
    newNode->next = *head;
    newNode->prev = NULL;
    if (*head != NULL)
        (*head)->prev = newNode;
    *head = newNode;
}
```

**(B) At End**
```
If list is empty:
  next = prev = NULL  (just insert it as the only node)

Else:
  1. Traverse to the last node
  2. last->next     = newNode
  3. newNode->prev  = last
  4. newNode->next  = NULL
```
```c
void insertAtEnd(Node** head, int data) {
    Node* newNode = createNode(data);
    newNode->next = NULL;
    if (*head == NULL) {
        newNode->prev = NULL;
        *head = newNode;
        return;
    }
    Node* last = *head;
    while (last->next)
        last = last->next;
    last->next = newNode;
    newNode->prev = last;
}
```

**(C) At Position**
```
1. Create newNode
2. Find size of list to make sure pos is within bounds
3. if pos == 0 or 1  → use "insert at beginning"
4. if pos == size    → use "insert at end"
5. Otherwise (pos is valid, not at boundaries):
     a. Traverse to the node BEFORE the desired position → prevNode
     b. newNode->next = prevNode->next
     c. newNode->prev = prevNode
     d. if (prevNode->next != NULL)
            prevNode->next->prev = newNode
     e. prevNode->next = newNode
```
```c
void insertAtPos(Node** head, int data, int pos) {
    if (pos <= 1) { insertAtBeginning(head, data); return; }

    Node* prevNode = *head;
    for (int i = 1; i < pos - 1 && prevNode; i++)
        prevNode = prevNode->next;

    if (prevNode == NULL) { printf("out of range\n"); return; }

    Node* newNode = createNode(data);
    newNode->next = prevNode->next;
    newNode->prev = prevNode;
    if (prevNode->next != NULL)
        prevNode->next->prev = newNode;
    prevNode->next = newNode;
}
```

### 6.9 Deletion

**(A) Delete at Beginning**

**Steps:**
1. If empty, nothing to do.
2. Save old `head` in `temp`.
3. Move `head` to `head->next`.
4. If new `head` is not `NULL`, set its `prev` to `NULL` (it's now first, has no predecessor).
5. Free `temp`.

```c
void deleteAtBeginning(Node** head) {
    if (*head == NULL) return;
    Node* temp = *head;
    *head = (*head)->next;
    if (*head != NULL)
        (*head)->prev = NULL;
    free(temp);
}
```

**(B) Delete at End**

**Steps:**
1. If empty, nothing to do. If only one node, delete it and set `head = NULL`.
2. Otherwise traverse to the last node.
3. Its `prev` node's `next` becomes `NULL` (new last node).
4. Free the old last node.

```c
void deleteAtEnd(Node** head) {
    if (*head == NULL) return;
    if ((*head)->next == NULL) {   // only one node
        free(*head);
        *head = NULL;
        return;
    }
    Node* last = *head;
    while (last->next)
        last = last->next;

    last->prev->next = NULL;
    free(last);
}
```

**(C) Delete at Position (1-based)**

**Steps:**
1. Validate `pos <= 0` → error.
2. If empty, nothing to do.
3. If `pos == 1`, use delete-at-beginning.
4. Traverse to the node AT the target position (`nodeToDelete`).
5. If it doesn't exist, out of range.
6. Re-link its neighbors around it:
   - If it has a `prev`, `prev->next = nodeToDelete->next`.
   - If it has a `next`, `next->prev = nodeToDelete->prev`.
   - If it was the tail, nothing extra needed for `next` side.
7. Free `nodeToDelete`.

```c
void deleteAtPos(Node** head, int pos) {
    if (pos <= 0) { printf("Invalid position\n"); return; }
    if (*head == NULL) return;
    if (pos == 1) { deleteAtBeginning(head); return; }

    Node* temp = *head;
    for (int i = 1; i < pos && temp; i++)
        temp = temp->next;

    if (temp == NULL) { printf("out of range\n"); return; }

    if (temp->prev) temp->prev->next = temp->next;
    if (temp->next) temp->next->prev = temp->prev;
    free(temp);
}
```

### 6.10 Free the Whole DLL
```c
void freeList(Node* head) {
    Node* temp;
    while (head) {
        temp = head;
        head = head->next;
        free(temp);
    }
}
```

---

## 7. `std::list` — Full STL Tutorial

`std::list` (from `<list>`) is C++'s built-in **doubly linked list**. It supports **constant-time** insertion/removal anywhere, but **no random access** (no `l[i]`).

### 7.1 Declaring a `std::list`

```cpp
#include <list>
using namespace std;

list<int> l1;                       // empty list of ints
list<int> l2 = {10, 20, 30};        // initializer-list construction
list<int> l3(5, 7);                 // 5 copies of value 7 → {7,7,7,7,7}
list<int> l4(l2);                   // copy constructor → {10,20,30}
list<int> l5(l2.begin(), l2.end()); // construct from a range (iterators)
list<string> names = {"Ali", "Sara", "Omar"};  // works with any type T
```

### 7.2 Element Access
`std::list` has **no `operator[]` and no `.at()`** — you can only access the ends directly.

```cpp
list<int> l = {10, 20, 30};
cout << l.front();   // 10 — first element
cout << l.back();    // 30 — last element
```

### 7.3 Iterators
Since there's no indexing, you traverse with **iterators** (bidirectional — can go forward and backward).

```cpp
list<int> l = {10, 20, 30};

for (auto it = l.begin(); it != l.end(); ++it)
    cout << *it << " ";          // 10 20 30

for (auto it = l.rbegin(); it != l.rend(); ++it)
    cout << *it << " ";          // 30 20 10  (reverse_iterator)

for (int x : l) cout << x << " "; // range-based for — cleanest way: 10 20 30

// const versions (read-only, guarantee no modification)
for (auto it = l.cbegin(); it != l.cend(); ++it) cout << *it << " ";
```

### 7.4 Capacity
```cpp
list<int> l = {10, 20, 30};
cout << l.empty();    // false (0)
cout << l.size();     // 3
cout << l.max_size(); // implementation-defined huge number
```

### 7.5 Modifiers

**`push_back` / `push_front`** — `O(1)` each:
```cpp
list<int> l = {10, 20};
l.push_back(30);    // {10, 20, 30}
l.push_front(5);    // {5, 10, 20, 30}
```

**`pop_back` / `pop_front`** — `O(1)` each:
```cpp
l.pop_back();    // removes 30  → {5, 10, 20}
l.pop_front();   // removes 5   → {10, 20}
```

**`emplace_back` / `emplace_front`** — construct in-place (avoids a copy/move):
```cpp
struct Point { int x, y; Point(int a, int b): x(a), y(b) {} };
list<Point> pts;
pts.emplace_back(1, 2);   // constructs Point(1,2) directly inside the list
pts.emplace_front(0, 0);
```

**`insert(iterator, value)`** — insert BEFORE the position the iterator points to:
```cpp
list<int> l = {10, 20, 30};
auto it = l.begin();
advance(it, 1);          // it now points to 20
l.insert(it, 99);        // {10, 99, 20, 30}  (99 inserted before 20)

l.insert(l.end(), 2, 77);   // insert 2 copies of 77 at the end → {..., 77, 77}
l.insert(l.begin(), {1,2,3}); // insert a whole initializer_list at front
```

**`emplace(iterator, args...)`** — like `insert`, but constructs in place:
```cpp
l.emplace(l.begin(), 500);  // {500, ...}
```

**`erase(iterator)`** / **`erase(first, last)`**:
```cpp
list<int> l = {10, 20, 30, 40};
auto it = l.begin();
++it;                 // points to 20
it = l.erase(it);     // removes 20, returns iterator to next element (30)
                       // l = {10, 30, 40}

l.erase(l.begin(), l.end());  // erase() with a range → clears everything
```

**`clear()`** — removes all elements:
```cpp
l.clear();   // l is now empty, size() == 0
```

**`resize(n)` / `resize(n, value)`**:
```cpp
list<int> l = {1, 2, 3};
l.resize(5);        // {1, 2, 3, 0, 0}   (new elements default-constructed)
l.resize(2);         // {1, 2}            (shrinks, discarding the rest)
l.resize(4, 9);       // {1, 2, 9, 9}      (new elements = 9)
```

**`swap`**:
```cpp
list<int> a = {1,2,3}, b = {9,8};
a.swap(b);   // a = {9,8}, b = {1,2,3}  — O(1), just swaps internal pointers
```

**`assign`**:
```cpp
list<int> l;
l.assign(5, 1);               // l = {1,1,1,1,1}
l.assign({10, 20, 30});       // l = {10,20,30}
```

### 7.6 Operations (the list-specific algorithms)

**`merge`** — merges two **already-sorted** lists into one sorted list (the other list becomes empty):
```cpp
list<int> a = {1, 3, 5};
list<int> b = {2, 4, 6};
a.merge(b);       // a = {1,2,3,4,5,6}, b = {} (now empty)
```

**`splice`** — moves elements from another list into this one **without copying** (`O(1)` per element):
```cpp
list<int> a = {1, 2, 3};
list<int> b = {10, 20, 30};

auto it = a.begin();
advance(it, 1);                    // points to 2
a.splice(it, b);                   // moves ALL of b before a's "2"
// a = {1, 10, 20, 30, 2, 3}, b = {} (empty)

list<int> c = {100, 200, 300};
auto itc = c.begin();
advance(itc, 1);                   // points to 200
a.splice(a.begin(), c, itc);       // move just "200" from c to front of a
// a = {200, 1, 10, 20, 30, 2, 3}, c = {100, 300}
```

**`remove(value)`** — removes ALL elements equal to `value`:
```cpp
list<int> l = {1, 2, 3, 2, 4, 2};
l.remove(2);   // l = {1, 3, 4}
```

**`remove_if(predicate)`** — removes all elements for which the predicate returns true:
```cpp
list<int> l = {1, 2, 3, 4, 5, 6};
l.remove_if([](int x){ return x % 2 == 0; });  // removes evens → {1,3,5}
```

**`reverse()`** — reverses the list in place:
```cpp
list<int> l = {1, 2, 3};
l.reverse();   // l = {3, 2, 1}
```

**`unique()`** — removes **consecutive** duplicate elements (like `uniq` in Unix):
```cpp
list<int> l = {1, 1, 2, 2, 2, 3, 1};
l.unique();    // l = {1, 2, 3, 1}   <- note: the last 1 stays, not consecutive to the first
```

**`sort()`** — sorts the list (`O(n log n)`, uses merge sort internally since lists lack random access):
```cpp
list<int> l = {5, 2, 8, 1};
l.sort();                      // {1, 2, 5, 8}
l.sort(greater<int>());        // {8, 5, 2, 1}  — custom comparator
```

### 7.7 Full Example Program

```cpp
#include <iostream>
#include <list>
using namespace std;

int main() {
    list<int> l = {10, 20, 30};

    l.push_back(40);            // 10 20 30 40
    l.push_front(5);             // 5 10 20 30 40
    l.pop_front();                 // 10 20 30 40
    l.pop_back();                   // 10 20 30

    for (int val : l) cout << val << " ";
    cout << endl;

    auto it = l.begin();
    advance(it, 1);
    l.insert(it, 99);              // 10 99 20 30

    it = l.begin();
    l.erase(it);                   // 99 20 30

    cout << "Size: " << l.size() << ", Front: " << l.front()
         << ", Back: " << l.back() << endl;

    l.push_back(20);
    l.sort();                      // 20 20 30 99
    l.unique();                    // 20 30 99
    l.reverse();                   // 99 30 20
    l.remove(30);                  // 99 20
}
```

---

## 8. `std::forward_list` — Full STL Tutorial

`std::forward_list` (from `<forward_list>`, C++11) is C++'s built-in **singly linked list** — more space-efficient than `std::list` since each node stores only one pointer, but you can only move **forward**.

### 8.1 Declaring a `std::forward_list`

```cpp
#include <forward_list>
using namespace std;

forward_list<int> f1;                     // empty
forward_list<int> f2 = {1, 2, 3};         // initializer list
forward_list<int> f3(5, 9);               // 5 copies of 9 → {9,9,9,9,9}
forward_list<int> f4(f2);                 // copy → {1,2,3}
forward_list<int> f5(f2.begin(), f2.end()); // from a range
```

### 8.2 Element Access
Only `front()` exists — **no `back()`**, because reaching the last element would require an `O(n)` walk every time, which the container intentionally does not expose.

```cpp
forward_list<int> f = {1, 2, 3};
cout << f.front();   // 1
```

### 8.3 Iterators — including the special `before_begin()`

```cpp
forward_list<int> f = {1, 2, 3};

for (auto it = f.begin(); it != f.end(); ++it)
    cout << *it << " ";            // 1 2 3

for (int x : f) cout << x << " "; // 1 2 3

auto bb = f.before_begin();       // a "phantom" iterator BEFORE the first element
                                    // used because forward_list has no "insert before"
                                    // — only "insert_after"
```

```
before_begin()        begin()
      │                 │
      ▼                 ▼
     [●] ─────────▶ [1] ──▶ [2] ──▶ [3] ──▶ end()
     (not a real       ^ first real element
      node — a marker)
```

### 8.4 Capacity
```cpp
forward_list<int> f = {1, 2, 3};
cout << f.empty();     // false
// NOTE: forward_list has NO size()! Counting requires O(n) via std::distance.
cout << distance(f.begin(), f.end());  // 3
cout << f.max_size();
```

### 8.5 Modifiers

**`push_front` / `pop_front`** — the ONLY end you can insert/remove at directly (no `push_back`!):
```cpp
forward_list<int> f = {2, 3};
f.push_front(1);   // {1, 2, 3}
f.pop_front();      // {2, 3}
```

**`emplace_front`**:
```cpp
f.emplace_front(0);   // constructs 0 in place at front
```

**`insert_after(pos, value)`** — inserts AFTER the given iterator (since you can't look backward to insert "before"):
```cpp
forward_list<int> f = {1, 3, 4};
auto it = f.begin();                // points to 1
f.insert_after(it, 2);              // {1, 2, 3, 4}

f.insert_after(f.before_begin(), 0); // insert at the very front → {0,1,2,3,4}
f.insert_after(it, 3, 99);           // insert 3 copies of 99 after "1"
```

**`emplace_after(pos, args...)`**:
```cpp
f.emplace_after(f.begin(), 42);
```

**`erase_after(pos)`** — removes the element AFTER `pos`:
```cpp
forward_list<int> f = {1, 2, 3, 4};
auto it = f.begin();            // points to 1
f.erase_after(it);              // removes 2 → {1, 3, 4}

f.erase_after(f.before_begin()); // removes the FIRST element → {3, 4}
f.erase_after(f.begin(), f.end()); // erase a whole range after "begin"
```

**`clear()`**:
```cpp
f.clear();   // empty
```

**`resize(n)` / `resize(n, value)`**:
```cpp
forward_list<int> f = {1, 2, 3};
f.resize(5);       // {1, 2, 3, 0, 0}
f.resize(1);        // {1}
```

**`swap`**:
```cpp
forward_list<int> a = {1,2}, b = {9,8,7};
a.swap(b);   // a = {9,8,7}, b = {1,2}
```

**`assign`**:
```cpp
forward_list<int> f;
f.assign(3, 5);          // {5,5,5}
f.assign({1,2,3});       // {1,2,3}
```

### 8.6 Operations

**`merge`** — merge two sorted forward_lists:
```cpp
forward_list<int> a = {1, 4, 5};
forward_list<int> b = {2, 3, 6};
a.merge(b);   // a = {1,2,3,4,5,6}, b = {} (empty)
```

**`splice_after`** — move elements from another forward_list AFTER a given position:
```cpp
forward_list<int> a = {1, 2, 3};
forward_list<int> b = {10, 20, 30};

a.splice_after(a.begin(), b);   // move ALL of b after a's first element (1)
// a = {1, 10, 20, 30, 2, 3}, b = {} (empty)

forward_list<int> c = {100, 200, 300};
auto itc = c.begin();                        // points to 100
a.splice_after(a.begin(), c, itc);           // move just the element AFTER itc (200)
// a = {1, 200, 10, 20, 30, 2, 3}, c = {100, 300}
```

**`remove(value)` / `remove_if(pred)`**:
```cpp
forward_list<int> f = {1, 2, 3, 2, 4};
f.remove(2);                                  // {1, 3, 4}
f.remove_if([](int x){ return x > 2; });      // removes 3,4 → {1}
```

**`reverse()`**:
```cpp
forward_list<int> f = {1, 2, 3};
f.reverse();   // {3, 2, 1}
```

**`unique()`** — removes consecutive duplicates:
```cpp
forward_list<int> f = {1, 1, 2, 3, 3, 3};
f.unique();    // {1, 2, 3}
```

**`sort()`**:
```cpp
forward_list<int> f = {5, 1, 4, 2};
f.sort();                 // {1, 2, 4, 5}
f.sort(greater<int>());   // {5, 4, 2, 1}
```

### 8.7 Full Example Program

```cpp
#include <iostream>
#include <forward_list>
using namespace std;

int main() {
    forward_list<int> fl = {3, 4, 5};

    fl.push_front(2);                       // 2 3 4 5
    fl.insert_after(fl.before_begin(), 1);  // 1 2 3 4 5

    for (int x : fl) cout << x << " ";
    cout << endl;

    auto it = fl.begin();
    advance(it, 2);                          // points to 3
    fl.insert_after(it, 99);                 // 1 2 3 99 4 5
    fl.erase_after(it);                      // removes 99 → 1 2 3 4 5

    fl.remove(3);                            // 1 2 4 5
    fl.reverse();                            // 5 4 2 1
    fl.sort();                               // 1 2 4 5

    cout << "Front: " << fl.front() << endl;
}
```

### 8.8 `std::list` vs `std::forward_list` — Side by Side

| Feature | `std::list` | `std::forward_list` |
|---|---|---|
| Underlying structure | Doubly linked | Singly linked |
| Iteration direction | Forward & backward | Forward only |
| `size()` member | ✅ `O(1)` | ❌ Not provided (use `std::distance`) |
| `back()` | ✅ | ❌ |
| `push_back()` / `pop_back()` | ✅ | ❌ |
| Insert/erase style | `insert`/`erase` (before a position) | `insert_after`/`erase_after` (after a position) |
| Memory per node | 2 pointers + data | 1 pointer + data |
| Typical use | Need both-direction traversal | Memory-critical, forward-only use |

---

## 9. Complexity Summary

| Operation | Singly LL | Circular LL | Doubly LL | `std::list` | `std::forward_list` | Array |
|---|---|---|---|---|---|---|
| Access by index | `O(n)` | `O(n)` | `O(n)` | `O(n)` | `O(n)` | `O(1)` |
| Search | `O(n)` | `O(n)` | `O(n)` | `O(n)` | `O(n)` | `O(n)` |
| Insert at head/front | `O(1)` | `O(1)` | `O(1)` | `O(1)` | `O(1)` | `O(n)` |
| Insert at end/back | `O(n)` (`O(1)` w/ tail ptr) | `O(n)` (`O(1)` w/ tail ptr) | `O(n)` (`O(1)` w/ tail ptr) | `O(1)` | ❌ not supported | `O(1)` amortized |
| Insert/Delete at known node | `O(1)` | `O(1)` | `O(1)` | `O(1)` | `O(1)` (after known node) | `O(n)` |
| Delete at head/front | `O(1)` | `O(1)` | `O(1)` | `O(1)` | `O(1)` | `O(n)` |
| Extra memory / node | 1 ptr | 1 ptr | 2 ptr | 2 ptr | 1 ptr | 0 |
| Traverse backward | ❌ | only via full loop | ✅ | ✅ | ❌ |✅ |
| `size()` in O(1) | manual counter needed | manual counter needed | manual counter needed | ✅ | ❌ | ✅ |

---

## 10. Practice Examples — Solved in Detail

### 10.1 Reverse a Singly Linked List (iterative, 3 pointers)

**Idea:** walk through the list once, and at each node flip its `next` pointer to point **backward** instead of forward.

**Steps:**
1. Keep three pointers: `prev = NULL`, `curr = head`, `next = NULL`.
2. While `curr != NULL`:
   - Save `next = curr->next` (don't lose the rest of the list).
   - Reverse the link: `curr->next = prev`.
   - Move both pointers forward: `prev = curr`, `curr = next`.
3. When the loop ends, `prev` is the new head (it was the old tail). Return `prev`.

```
Initial:  NULL  prev    curr                  next
                 ↑        ↑
                NULL ◀—— [1]→[2]→[3]→NULL

Iter 1:  next=curr->next (2)
         curr->next = prev  →  [1]→NULL
         prev=curr(1), curr=next(2)
         NULL ◀─[1]    [2]→[3]→NULL
                ↑prev   ↑curr

Iter 2:  next=3
         [2]->next = prev(1)  → [2]→[1]→NULL
         prev=2, curr=3
         NULL◀[1]◀[2]   [3]→NULL
                   ↑prev ↑curr

Iter 3:  next=NULL
         [3]->next = prev(2) → [3]→[2]→[1]→NULL
         prev=3, curr=NULL  → loop ends

Result:  head = prev = 3 → 2 → 1 → NULL  ✓ reversed
```

```c
Node* reverseList(Node* head) {
    Node* prev = NULL;
    Node* curr = head;
    Node* next = NULL;

    while (curr != NULL) {
        next = curr->next;   // save next before overwriting
        curr->next = prev;   // reverse the pointer
        prev = curr;          // advance prev
        curr = next;          // advance curr
    }
    return prev;   // prev is the new head
}
```
**Time:** `O(n)` | **Space:** `O(1)`

---

### 10.2 Detect a Loop (Floyd's Cycle Detection — slow/fast pointers)

**Idea:** use two pointers moving at different speeds. If there's a cycle, the faster one will eventually "lap" the slower one and they'll meet. If there's no cycle, the fast pointer reaches `NULL`.

**Steps:**
1. `slow = head`, `fast = head`.
2. Loop while `fast` and `fast->next` are not `NULL`:
   - Move `slow` one step (`slow = slow->next`).
   - Move `fast` two steps (`fast = fast->next->next`).
   - If `slow == fast`, a cycle exists — return true.
3. If the loop exits normally (fast hit `NULL`), there's no cycle — return false.

```
No cycle:  1→2→3→4→NULL
  fast always reaches NULL first → no loop.

With cycle:        ┌────────────┐
             1→2→3→4            │
                   └────────────┘ (4 points back to 2)

  slow: 1,2,3,4,2,3,...   (1 step at a time)
  fast: 1,3,2,4,3,...     (2 steps at a time)
  Eventually slow and fast land on the SAME node → cycle detected.
```

```c
int hasCycle(Node* head) {
    Node* slow = head;
    Node* fast = head;

    while (fast != NULL && fast->next != NULL) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast)
            return 1;   // cycle found
    }
    return 0;   // no cycle
}
```
**Time:** `O(n)` | **Space:** `O(1)`

---

### 10.3 Find the Middle Node (one pass, slow/fast pointers)

**Idea:** same two-speed trick — when `fast` reaches the end, `slow` will be exactly at the middle (because it moved half as far).

**Steps:**
1. `slow = head`, `fast = head`.
2. While `fast != NULL` and `fast->next != NULL`: `slow = slow->next`, `fast = fast->next->next`.
3. When the loop ends, `slow` is at the middle node. Return it.

```
1→2→3→4→5→NULL   (odd length)
slow: 1  2  3        → ends at 3 (true middle)
fast: 1  3  5(end)

1→2→3→4→NULL      (even length)
slow: 1  2  3        → ends at 3 (2nd of the two middles, by convention)
fast: 1  3  NULL(end, loop stops because fast->next is NULL after landing on 4? let's check)
```

```c
Node* findMiddle(Node* head) {
    Node* slow = head;
    Node* fast = head;

    while (fast != NULL && fast->next != NULL) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;
}
```
**Time:** `O(n)` | **Space:** `O(1)`

---

### 10.4 Merge Two Sorted Linked Lists

**Idea:** like merging two sorted arrays — repeatedly take the smaller "current" node from either list, and link it into the result. Use a **dummy node** to avoid special-casing the very first insertion.

**Steps:**
1. Create a `dummy` node (its value doesn't matter) and a `tail` pointer starting at `dummy`.
2. While both lists have nodes left:
   - Compare the current nodes of each list.
   - Attach the smaller one to `tail->next`, then advance both `tail` and that list's pointer.
3. When one list runs out, attach whatever remains of the other list directly (it's already sorted).
4. Return `dummy->next` (the dummy itself was never a real part of the result).

```
List A: 1→3→5→NULL
List B: 2→4→6→NULL

dummy→? , tail=dummy
compare 1,2 → take 1   dummy→1   tail=1
compare 3,2 → take 2   dummy→1→2
compare 3,4 → take 3   dummy→1→2→3
compare 5,4 → take 4   dummy→1→2→3→4
compare 5,6 → take 5   dummy→1→2→3→4→5
A exhausted → attach rest of B (6)  dummy→1→2→3→4→5→6
```

```c
Node* mergeSorted(Node* a, Node* b) {
    Node dummy;              // stack-allocated dummy node
    dummy.next = NULL;
    Node* tail = &dummy;

    while (a != NULL && b != NULL) {
        if (a->data <= b->data) {
            tail->next = a;
            a = a->next;
        } else {
            tail->next = b;
            b = b->next;
        }
        tail = tail->next;
    }
    tail->next = (a != NULL) ? a : b;  // attach whichever list remains
    return dummy.next;
}
```
**Time:** `O(n + m)` | **Space:** `O(1)` extra (reuses existing nodes)

---

### 10.5 Remove Duplicates from an Unsorted Linked List (using a hash set)

**Idea:** keep a set of values seen so far. Walk the list; for each node, if its value was already seen, unlink and free it; otherwise record it as seen.

**Steps:**
1. If the list is empty, return.
2. Mark `head->data` as seen. Set `curr = head`.
3. While `curr->next != NULL`:
   - If `curr->next->data` has been seen before: unlink it (`curr->next = curr->next->next`) and free the removed node. Do NOT advance `curr` (there's a new `curr->next` to check).
   - Else: mark `curr->next->data` as seen, and advance `curr = curr->next`.

```
List: 5 → 2 → 5 → 9 → 2 → NULL
seen = {5}, curr=5
  curr->next=2, not seen → seen={5,2}, curr=2
  curr->next=5, SEEN → unlink: curr->next = (5)->next = 9
    free old "5" node
    List now: 5→2→9→2→NULL   (curr stays at 2)
  curr->next=9, not seen → seen={5,2,9}, curr=9
  curr->next=2, SEEN → unlink: curr->next = (2)->next = NULL
    List now: 5→2→9→NULL
Done.
```

```c
#include <stdbool.h>
#define TABLE_SIZE 1000

void removeDuplicates(Node* head) {
    if (head == NULL) return;

    bool seen[TABLE_SIZE] = {false};   // simplistic hash set for small int ranges
    seen[head->data] = true;
    Node* curr = head;

    while (curr->next != NULL) {
        if (seen[curr->next->data]) {
            Node* duplicate = curr->next;
            curr->next = curr->next->next;
            free(duplicate);
        } else {
            seen[curr->next->data] = true;
            curr = curr->next;
        }
    }
}
```
**Time:** `O(n)` average | **Space:** `O(n)` for the hash set
> For a real hash set over arbitrary values, use `std::unordered_set<int>` in C++.

---

### 10.6 Check if a Linked List is a Palindrome

**Approach A — Using an auxiliary stack/array:**

**Steps:**
1. Traverse the list once, pushing each value into a stack (or array).
2. Traverse the list again from `head`, and for each node, pop from the stack and compare.
3. If any mismatch occurs, it's not a palindrome. If all match, it is.

```
List: 1→2→3→2→1→NULL
Stack after push: [1,2,3,2,1]  (top=1)
Compare head(1) with pop()=1 ✓
Compare head(2) with pop()=2 ✓
Compare head(3) with pop()=3 ✓
Compare head(2) with pop()=2 ✓
Compare head(1) with pop()=1 ✓
→ Palindrome!
```

```c
#include <stdbool.h>
bool isPalindrome(Node* head) {
    int arr[1000], n = 0;
    for (Node* t = head; t; t = t->next)
        arr[n++] = t->data;

    for (int i = 0, j = n - 1; i < j; i++, j--)
        if (arr[i] != arr[j]) return false;

    return true;
}
```
**Time:** `O(n)` | **Space:** `O(n)`

**Approach B — `O(1)` extra space:** find the middle (10.3), reverse the second half (10.1), compare both halves node by node, then (optionally) re-reverse the second half to restore the list.

---

### 10.7 Find the Nth Node from the End (two-pointer / gap technique)

**Idea:** move one pointer `N` steps ahead first. Then move both pointers together — when the lead pointer reaches the end, the trailing pointer is exactly `N` nodes from the end.

**Steps:**
1. Set `fast = head`. Move it `N` steps forward. (If it becomes `NULL` before `N` steps, the list is shorter than `N` — invalid input.)
2. Set `slow = head`.
3. Move `fast` and `slow` forward together, one step each, until `fast == NULL`.
4. `slow` now points to the Nth node from the end. Return it.

```
List: 1→2→3→4→5→NULL,  N=2 (2nd from end = 4)

Step1: fast moves 2 steps from head → fast at node 3
       1  2 [3] 4  5
              ▲fast

Step2: slow=head(1)

Step3: move both until fast hits NULL
  fast:3→4→5→NULL   slow:1→2→3→4
  when fast=NULL, slow=4  ✓ (2nd from end)
```

```c
Node* nthFromEnd(Node* head, int n) {
    Node* fast = head;
    for (int i = 0; i < n; i++) {
        if (fast == NULL) return NULL;  // list shorter than n
        fast = fast->next;
    }
    Node* slow = head;
    while (fast != NULL) {
        fast = fast->next;
        slow = slow->next;
    }
    return slow;
}
```
**Time:** `O(n)`, single pass | **Space:** `O(1)`

---

### 10.8 Convert a Singly Linked List to Circular, and Back

**To Circular — Steps:**
1. If the list is empty, nothing to do.
2. Traverse to the last node (where `next == NULL`).
3. Set `last->next = head`.

**Back to Singly (break the circle) — Steps:**
1. If the list is empty, nothing to do.
2. Traverse starting from `head->next` until you find the node whose `next == head` (that's the last node).
3. Set that node's `next = NULL`.

```c
void makeCircular(Node* head) {
    if (head == NULL) return;
    Node* last = head;
    while (last->next != NULL)
        last = last->next;
    last->next = head;
}

void breakCircular(Node* head) {
    if (head == NULL) return;
    Node* curr = head;
    while (curr->next != head)
        curr = curr->next;
    curr->next = NULL;
}
```
**Time:** `O(n)` for both | **Space:** `O(1)`

---

### 10.9 Implement `std::list`-like push/pop Manually (to see what STL does under the hood)

**Idea:** build a minimal doubly linked list "class" in C with a `head`, a `tail` pointer (so `push_back` is `O(1)` too — unlike our earlier manual version that had to traverse to find the end!), and the 4 basic operations.

**Steps:**
1. Keep both `head` and `tail` pointers (this is the trick STL uses to make BOTH ends `O(1)`).
2. `push_front`: create node, link it before `head`, update `head` (and `tail` if list was empty).
3. `push_back`: create node, link it after `tail`, update `tail` (and `head` if list was empty).
4. `pop_front` / `pop_back`: unlink the respective end node, update `head`/`tail`, free it.

```c
typedef struct DNode {
    int data;
    struct DNode *prev, *next;
} DNode;

typedef struct {
    DNode *head, *tail;
} MyList;

void push_back(MyList* l, int data) {
    DNode* n = (DNode*) malloc(sizeof(DNode));
    n->data = data; n->next = NULL; n->prev = l->tail;
    if (l->tail) l->tail->next = n;
    else l->head = n;          // list was empty
    l->tail = n;
}

void push_front(MyList* l, int data) {
    DNode* n = (DNode*) malloc(sizeof(DNode));
    n->data = data; n->prev = NULL; n->next = l->head;
    if (l->head) l->head->prev = n;
    else l->tail = n;           // list was empty
    l->head = n;
}

void pop_back(MyList* l) {
    if (!l->tail) return;
    DNode* old = l->tail;
    l->tail = old->prev;
    if (l->tail) l->tail->next = NULL;
    else l->head = NULL;        // list is now empty
    free(old);
}

void pop_front(MyList* l) {
    if (!l->head) return;
    DNode* old = l->head;
    l->head = old->next;
    if (l->head) l->head->prev = NULL;
    else l->tail = NULL;        // list is now empty
    free(old);
}
```
**Takeaway:** this is exactly why `std::list::push_back`/`pop_back` are `O(1)` — the container secretly maintains a `tail` pointer, unlike our earlier hand-written singly linked list examples that had to walk to the end each time.

---

## 11. Quick Reference

```c
// Singly Linked List
typedef struct Node {
    int data;
    struct Node* next;
} Node;

// Doubly Linked List
typedef struct Node {
    struct Node* prev;
    int data;
    struct Node* next;
} Node;

// Circular Linked List — same struct as singly,
// difference is only in HOW it's linked (last->next = head)
```

```cpp
// STL
#include <list>           // doubly linked list: push_back, push_front, size(), both directions
#include <forward_list>   // singly linked list: push_front only, insert_after/erase_after, no size()
```

| Need | Use |
|---|---|
| Random access by index | `std::vector` (not a linked list at all) |
| Frequent insert/delete anywhere, need backward traversal | `std::list` |
| Frequent insert/delete anywhere, memory-tight, forward-only | `std::forward_list` |
| Circular buffer behavior | custom circular linked list |
