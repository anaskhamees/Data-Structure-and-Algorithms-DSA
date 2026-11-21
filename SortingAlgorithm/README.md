# Sorting Techniques in C++

Lecture companion for the **Sorting** chapter of Data Structures.
Every algorithm follows the same pattern:

> **Idea → Steps → Worked example (trace) → Implementation → Analysis**

---

## Table of Contents

0. [Overview and Classification](#0-overview-and-classification)
1. [Criteria for Analysis](#1-criteria-for-analysis)
2. [Helper Functions](#2-helper-functions)
3. [Bubble Sort](#3-bubble-sort)
4. [Insertion Sort](#4-insertion-sort)
5. [Bubble vs Insertion](#5-bubble-sort-vs-insertion-sort)
6. [Selection Sort](#6-selection-sort)
7. [Quick Sort](#7-quick-sort)
8. [Merge Sort (iterative + recursive)](#8-merge-sort)
9. [Count Sort](#9-count-sort)
10. [Bucket / Bin Sort](#10-bucket--bin-sort)
11. [Radix Sort](#11-radix-sort)
12. [Shell Sort](#12-shell-sort)
13. [Heap Sort](#13-heap-sort)
14. [Tree Sort](#14-tree-sort)
15. [Summary Table](#15-summary-table)
16. [Sorting in the STL](#16-sorting-in-the-stl)
17. [Which One Should I Use?](#17-which-one-should-i-use)

---

## 0. Overview and Classification

| # | Algorithm | Family | Typical time |
|---|-----------|--------|--------------|
| 1 | Bubble | Comparison | O(n²) |
| 2 | Insertion | Comparison | O(n²) |
| 3 | Selection | Comparison | O(n²) |
| 4 | Heap | Comparison | O(n log n) |
| 5 | Merge | Comparison | O(n log n) |
| 6 | Quick | Comparison | O(n log n) average, **O(n²) worst** |
| 7 | Tree | Comparison | O(n log n) average, **O(n²) worst** |
| 8 | Shell | Comparison | between O(n log n) and O(n²), depends on the gap sequence |
| 9 | Count | Index-based | O(n + k) |
| 10 | Bucket / Bin | Index-based | O(n + k) |
| 11 | Radix | Index-based | O(d · n) |

- **Comparison-based sorts** decide the order only by comparing two elements
  (`a[i] > a[j]`). No comparison sort can beat **O(n log n)** in the worst case.
- **Index-based sorts** use the *value itself* as an index (or a digit of it as an index).
  That is how they reach linear time, but they only work on **non-negative integers** in
  a known range (or need extra work for other data).

> `k` = maximum value in the array, `d` = number of digits of the maximum value.

---

## 1. Criteria for Analysis

| # | Criterion | Meaning |
|---|-----------|---------|
| 1 | **Number of comparisons** | How many times two elements are compared (drives the running time). |
| 2 | **Number of swaps** | How many times elements are moved or exchanged. |
| 3 | **Adaptive** | If the array is *already sorted*, the algorithm should finish faster (ideally O(n)). |
| 4 | **Stable** | Equal elements keep their **original relative order**. Example: if `A=10` comes before `B=10` in the input, `A` must still come before `B` in the output. |
| 5 | **Extra memory** | Memory used besides the input array. *In-place* means O(1) extra. |

Why stability matters: when sorting records by a second key (e.g. sort students by name,
then *stably* by grade), stable sorting keeps the first ordering inside equal grades.
Radix sort depends on this: each pass must be stable.

---

## 2. Helper Functions

```cpp
#include <iostream>
#include <vector>
#include <utility>   // std::swap
#include <algorithm> // std::min
using namespace std;

void printArr(const int* arr, int n)
{
    for (int i = 0; i < n; i++)
        cout << arr[i] << (i + 1 < n ? " " : "\n");
}

int findMax(const int* arr, int n)
{
    int max = arr[0];
    for (int i = 1; i < n; i++)
        if (arr[i] > max) max = arr[i];
    return max;
}
```

In C++ we use `std::swap(a, b)`. The C equivalent is `swap(&a, &b)`:

```c
void swap(int* x, int* y) { int t = *x; *x = *y; *y = t; }
```

---

## 3. Bubble Sort

### Idea
Compare every element with the **next** one and swap them if they are in the wrong order.
After each pass, the **largest remaining element "bubbles up"** to the end of the array.

### Steps
1. Start from the first element and compare it with the next one.
2. If the first is greater, swap them.
3. Keep comparing neighbours up to the end of the unsorted part. After pass 1 the
   largest element is at the end.
4. Repeat for the rest of the array, ignoring the last sorted elements.
5. **Improvement (flag):** if a whole pass makes **no swap**, the array is already sorted, so stop.

### Trace: `A = [8, 5, 7, 3, 2]`, n = 5

| Pass | Array after the pass | Comparisons | Swaps |
|------|----------------------|-------------|-------|
| 1 | 5 7 3 2 **8** | 4 | 4 |
| 2 | 5 3 2 **7 8** | 3 | 2 (max 3) |
| 3 | 3 2 **5 7 8** | 2 | 2 |
| 4 | **2 3 5 7 8** | 1 | 1 |

- Number of passes = **n − 1**
- Comparisons = 1 + 2 + … + (n−1) = **n(n−1)/2 = O(n²)**
- Max swaps = **n(n−1)/2 = O(n²)**

> **k passes property:** after *k* passes the **k largest** elements are in their final
> places. To get the 3 largest elements, run only 3 passes.

### Implementation

```cpp
void bubbleSort(int* arr, int n)
{
    for (int i = 0; i < n - 1; i++)            // passes: n-1
    {
        int flag = 0;
        for (int j = 0; j < n - 1 - i; j++)    // one pass
        {
            if (arr[j] > arr[j + 1])
            {
                swap(arr[j], arr[j + 1]);
                flag = 1;
            }
        }
        if (flag == 0)                          // no swaps -> already sorted
            break;
    }
}
```

### Analysis
| | |
|---|---|
| Best (already sorted) | O(n): one pass, no swaps, `flag` stops it |
| Worst / Average | O(n²) |
| Extra memory | O(1) |
| Stable | ✅ uses `>` (not `>=`), so equal elements are never swapped |
| Adaptive | ✅ only **with** the flag |

---

## 4. Insertion Sort

### (A) Inserting into a sorted array

**Idea:** start from the **end**, shift every element greater than the key one step right,
then put the key in the gap.

Insert **18** into `[2, 6, 10, 15, 20, 25, 30]`:

```
2 6 10 15 20 25 30 _      30 > 18 -> shift right
2 6 10 15 20 25 _ 30      25 > 18 -> shift right
2 6 10 15 20 _ 25 30      20 > 18 -> shift right
2 6 10 15 _ 20 25 30      15 < 18 -> stop
2 6 10 15 18 20 25 30     put 18 in the gap
```

```cpp
// n = current number of elements; the array must have room for n+1
void insertSorted(int* arr, int n, int key)
{
    int i = n - 1;
    while (i >= 0 && arr[i] > key)
    {
        arr[i + 1] = arr[i];
        i--;
    }
    arr[i + 1] = key;
}
```

### (B) Insertion sort

**Idea:** split the array into a **sorted part** (initially just the first element) and an
**unsorted part**. Take elements from the unsorted part one by one and **insert** each into
its correct place in the sorted part, using (A).

### Steps
1. Assume the first element is sorted.
2. Take the next element `x = arr[i]` (the first element of the unsorted part).
3. Compare it with the elements before it, shifting every element greater than `x`
   one step right.
4. Put `x` in the gap. Repeat for `i = 1 … n−1`.

### Trace: `[8 | 5, 7, 3, 2]`

| Pass | Insert | Result | Max comparisons | Max shifts |
|------|--------|--------|-----------------|------------|
| 1 | 5 | 5 8 \| 7 3 2 | 1 | 1 |
| 2 | 7 | 5 7 8 \| 3 2 | 2 | 2 |
| 3 | 3 | 3 5 7 8 \| 2 | 3 | 3 |
| 4 | 2 | 2 3 5 7 8 | 4 | 4 |

- Passes = **n − 1**
- Comparisons (worst) = n(n−1)/2 = **O(n²)**
- Shifts (worst) = n(n−1)/2 = **O(n²)**

### Implementation

```cpp
void insertionSort(int* arr, int n)
{
    for (int i = 1; i < n; i++)      // i -> passes
    {
        int j = i - 1;
        int x = arr[i];
        while (j >= 0 && arr[j] > x) // j >= 0 : x may be smaller than everything
        {                            // '>'     : keeps it stable
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = x;
    }
}
```

Example inside a pass: `[6 8 12 14 18 20 | 10]`, `x = 10`. Here 20, 18, 14 and 12 shift
right, the loop stops at 8, and 10 goes at index 2.

> **Why `j >= 0`?** Try `x = 2` with sorted part `[6 8 12]`: without it, `j` becomes −1
> and `arr[-1]` is read (out of bounds).
> **Why `>` and not `>=`?** With `>=`, an element would jump over an equal one, so the
> algorithm would no longer be stable.

### Analysis
| | |
|---|---|
| Best (already sorted) | O(n): one comparison per pass, no shifts |
| Worst (reverse sorted) / Average | O(n²) |
| Extra memory | O(1) |
| Stable | ✅ |
| Adaptive | ✅ by nature, no flag needed |
| Linked list | ✅ well suited: insertion is just relinking, no shifting |

---

## 5. Bubble Sort vs Insertion Sort

| | Bubble (with flag) | Insertion |
|---|---|---|
| Min comparisons | O(n) | O(n) |
| Max comparisons | O(n²) | O(n²) |
| Min swaps | 0, i.e. O(1) | 0, i.e. O(1) |
| Max swaps | O(n²) | O(n²) |
| Adaptive | ✅ | ✅ |
| Stable | ✅ | ✅ |
| Suited to linked list | ❌ | ✅ |
| Useful result after k passes | ✅ the k largest are at the end | ❌ only the first k+1 sorted among themselves |

> Min case = input already **ascending**. Max case = input **descending**.

---

## 6. Selection Sort

### Idea
**Select a position, then find the correct element for it.**
For position `i`, find the **minimum** of `arr[i … n−1]` and swap it into position `i`.

### Steps
1. Let `i` be the current position (start at 0).
2. Let `k = i`. `k` is the index of the smallest element found so far.
3. Scan `j = i … n−1`. If `arr[j] < arr[k]`, set `k = j`.
4. Swap `arr[i]` with `arr[k]`.
5. Move to `i + 1` and repeat **n − 1** times.

### Trace: `[8, 6, 3, 2, 5, 4]`

| Pass | i | Minimum found | Array after swap |
|------|---|---------------|------------------|
| 1 | 0 | 2 (index 3) | **2** 6 3 8 5 4 |
| 2 | 1 | 3 (index 2) | **2 3** 6 8 5 4 |
| 3 | 2 | 4 (index 5) | **2 3 4** 8 5 6 |
| 4 | 3 | 5 (index 4) | **2 3 4 5** 8 6 |
| 5 | 4 | 6 (index 5) | **2 3 4 5 6 8** |

> **k passes property:** after *k* passes the **k smallest** elements are in place.

### Implementation

```cpp
void selectionSort(int* arr, int n)
{
    for (int i = 0; i < n - 1; i++)
    {
        int k = i;                           // index of current minimum
        for (int j = i; j < n; j++)
        {
            if (arr[j] < arr[k])
                k = j;
        }
        swap(arr[i], arr[k]);
    }
}
```

### Analysis
| | |
|---|---|
| Comparisons | always n(n−1)/2 = O(n²), even on sorted input |
| Swaps | exactly n − 1 = **O(n)**, the fewest of all the O(n²) sorts |
| Extra memory | O(1) |
| Stable | ❌ e.g. `[5a, 5b, 2]`: the swap moves 5a after 5b |
| Adaptive | ❌ |

---

## 7. Quick Sort

### Idea (Divide and Conquer)
**Select an element, then find the correct position for it.**
An element is in its final sorted position when **everything before it is smaller (or equal)**
and **everything after it is greater**.

### Steps
1. Choose a **pivot** (here: the **first** element; it could also be the last, the middle or a random one).
2. **Partition:**
   - `i` moves **right** and stops at the first element **> pivot**.
   - `j` moves **left** and stops at the first element **≤ pivot**.
   - If `i < j`, swap `arr[i]` and `arr[j]` and continue.
   - When `i` and `j` cross, swap the pivot with `arr[j]`.
     The pivot is now **fixed at its sorted position `j`**, the *partitioning position*.
3. Recursively apply the same steps to the **left part** and the **right part**.

### Trace: `[50, 70, 60, 90, 40, 80, 10, 20, 30]`, pivot = 50

```
50 | 70 60 90 40 80 10 20 30        i -> 70 (>50), j -> 30 (<=50)  swap
50 | 30 60 90 40 80 10 20 70        i -> 60,       j -> 20         swap
50 | 30 20 90 40 80 10 60 70        i -> 90,       j -> 10         swap
50 | 30 20 10 40 80 90 60 70        i -> 80 (idx 5), j -> 40 (idx 4): crossed, stop
swap pivot with arr[j]:
(40 30 20 10) 50 (80 90 60 70)      50 is at its final position (index 4)
 do the same            do the same
```

### Implementation

```cpp
int partition(int* arr, int low, int high)
{
    int pivot = arr[low];
    int i = low;
    int j = high;

    while (i < j)
    {
        while (arr[i] <= pivot && i < high)  // first element > pivot
            i++;
        while (arr[j] > pivot && j > low)    // first element <= pivot
            j--;
        if (i < j)
            swap(arr[i], arr[j]);
    }
    swap(arr[low], arr[j]);                  // pivot to its sorted place
    return j;                                // partitioning position
}

void quickSort(int* arr, int low, int high)
{
    if (low < high)
    {
        int j = partition(arr, low, high);
        quickSort(arr, low, j - 1);          // left part
        quickSort(arr, j + 1, high);         // right part
    }
}

int main()
{
    int arr[] = {8, 2, 5, 4, 0};
    int n = sizeof(arr) / sizeof(arr[0]);
    quickSort(arr, 0, n - 1);
}
```

> On paper, an **∞ end marker** is often placed after the last element so `i` can never run
> past the array. The code uses the bound `i < high` instead, so no sentinel is needed.

### Analysis
| Case | When | Time |
|------|------|------|
| Best | pivot lands in the **middle** every time | O(n log n) |
| Average | random data | O(n log n) |
| Worst | pivot lands at **one end** every time, e.g. array **already sorted** asc/desc with first-element pivot | O(n²) |

- Extra memory: recursion stack, O(log n) on average and O(n) in the worst case.
- Stable ❌, Adaptive ❌.
- **Avoiding the worst case:** pick the **middle element** (swap it to `arr[low]` first),
  a **random** element, or the **median of three** (first, middle, last). Then sorted input
  gives O(n log n) too. A bad pivot is still *possible*, just very unlikely.

### Selection Sort vs Quick Sort
| Selection sort | Quick sort |
|---|---|
| Select a **position**, find the correct **element** for it | Select an **element**, find the correct **position** for it |

---

## 8. Merge Sort

### (A) Merging two sorted sub-arrays

**Idea:** two sorted halves `arr[low…mid]` and `arr[mid+1…high]` are merged into **one**
sorted array using a temporary array of size `high − low + 1`.

```
 sub-arr 1          sub-arr 2
[2  5  8  12]  |  [3  6  7  10]
 i                 j
take the smaller of arr[i], arr[j] -> tempArr[k++]
tempArr = [2 3 5 6 7 8 10 12]  -> copy back into arr[low..high]
```

```cpp
void merge(int* arr, int low, int mid, int high)
{
    vector<int> tempArr(high - low + 1);
    int i = low;          // walks sub-array 1
    int j = mid + 1;      // walks sub-array 2
    int k = 0;            // walks tempArr (starts at 0!)

    while (i <= mid && j <= high)
    {
        if (arr[i] <= arr[j])                 // '<=' keeps it stable
            tempArr[k++] = arr[i++];
        else
            tempArr[k++] = arr[j++];
    }
    for (; i <= mid;  i++) tempArr[k++] = arr[i];   // leftovers of sub-array 1
    for (; j <= high; j++) tempArr[k++] = arr[j];   // leftovers of sub-array 2

    for (i = low; i <= high; i++)                   // copy back
        arr[i] = tempArr[i - low];
}
```

### (B) Iterative (bottom-up) merge sort

**Idea:** treat the array of `n` elements as **n lists of one element each** (one element is
already sorted). Merge pairs of size 1 into lists of 2, then pairs of 2 into lists of 4,
then 8, …

### Trace: `[8, 3, 7, 4, 9, 2, 6, 5]`, n = 8

```
p = 2 :  8 3 | 7 4 | 9 2 | 6 5   ->  3 8 | 4 7 | 2 9 | 5 6     (n work)
p = 4 :  3 8 4 7 | 2 9 5 6       ->  3 4 7 8 | 2 5 6 9         (n work)
p = 8 :  3 4 7 8 2 5 6 9         ->  2 3 4 5 6 7 8 9           (n work)
```
Passes = log₂ 8 = **3**, each pass does **n** work, so the total is **O(n log n)**.

| Variable | Meaning |
|---|---|
| `p` | size of the block being produced (2, 4, 8, …) |
| `i` | start index of the current pair of sub-arrays |
| `left` | `i` |
| `mid` | end of the first half = `i + p/2 − 1` |
| `right` | end of the second half = `i + p − 1` |

```cpp
void mergeSortIterative(int* arr, int n)
{
    for (int p = 2; p / 2 < n; p *= 2)          // block sizes 2, 4, 8, ...
    {
        for (int i = 0; i < n; i += p)          // step = block size
        {
            int left  = i;
            int mid   = min(i + p / 2 - 1, n - 1);
            int right = min(i + p - 1,     n - 1);  // last block may be short
            if (mid < right)                    // a right half exists
                merge(arr, left, mid, right);
        }
    }
}
```

> When **n is not a power of 2**, the last block is shorter than `p`. Clamping `mid` and
> `right` to `n − 1` handles that inside the loop, so no extra merge is needed after it.
> Example, n = 7: `p = 2` merges (0,1) (2,3) (4,5) and leaves 6 alone; `p = 4` merges
> (0..3) and (4..6); `p = 8` merges (0..3) with (4..6).

### (C) Recursive (top-down) merge sort

### Steps
1. **Base case:** if the (sub)array has 0 or 1 element (`low >= high`), it is already sorted, so stop.
2. **Divide:** `mid = (low + high) / 2`.
3. **Conquer:** recursively sort the left half `[low … mid]` and the right half `[mid+1 … high]`.
4. **Merge** the two sorted halves.

### Trace: `[8, 2, 9, 6, 5, 3, 7, 4]`

```
                [8 2 9 6 | 5 3 7 4]
           [8 2 | 9 6]          [5 3 | 7 4]
         [8|2]   [9|6]         [5|3]   [7|4]
        [8] [2] [9] [6]       [5] [3] [7] [4]       <- base cases
         [2 8]   [6 9]         [3 5]   [4 7]        <- merge
           [2 6 8 9]             [3 4 5 7]          <- merge
                [2 3 4 5 6 7 8 9]                   <- merge
```
Merge order in the recursion tree:
`(0,1) → (2,3) → (0,3) → (4,5) → (6,7) → (4,7) → (0,7)`.

```cpp
void mergeSort(int* arr, int low, int high)
{
    if (low < high)
    {
        int mid = (low + high) / 2;
        mergeSort(arr, low, mid);         // sort left half
        mergeSort(arr, mid + 1, high);    // sort right half
        merge(arr, low, mid, high);       // merge both
    }
}
```

### Analysis
| | |
|---|---|
| Time | **O(n log n)** in best, average and worst cases: log n levels with n elements merged per level |
| Extra memory | auxiliary array **O(n)** + recursion stack **O(log n)**, so **O(n)** overall |
| Stable | ✅ (with `<=` in `merge`) |
| Adaptive | ❌ |
| Linked lists | ✅ excellent: merging lists needs no extra array |

---

## 9. Count Sort

*Simple and fast*, for **non-negative integers** with a small maximum value.

### Steps
1. Find the **max** element of the array.
2. Create `countArr` of size **max + 1** (indices 0 … max).
3. Initialise it with **zeros**.
4. For every element, `countArr[arr[i]]++` (count frequencies).
5. Scan `countArr` from left to right. While `countArr[i] > 0`, write `i` into the
   original array and decrement `countArr[i]`.

### Trace: `[6, 3, 9, 10, 15, 6, 8, 12, 3, 6]`, max = 15

```
index    : 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
countArr : 0 0 0 2 0 0 3 0 1 1  1  0  1  0  0  1

scan -> 3 3 6 6 6 8 9 10 12 15
```

### Implementation

```cpp
void countSort(int* arr, int n)
{
    int max = findMax(arr, n);
    vector<int> countArr(max + 1, 0);        // size max+1, all zeros

    for (int i = 0; i < n; i++)
        countArr[arr[i]]++;                  // count frequencies

    int i = 0, j = 0;                        // i -> countArr, j -> arr
    while (i < max + 1)
    {
        if (countArr[i] > 0)
        {
            arr[j++] = i;                    // the index is the value
            countArr[i]--;
        }
        else
            i++;
    }
}
```

Equivalent C version with `malloc`:

```c
int* countArr = (int*)malloc((max + 1) * sizeof(int));
for (i = 0; i < max + 1; i++) countArr[i] = 0;     /* or use calloc */
/* ... */
free(countArr);
```

### Analysis
| | |
|---|---|
| Time | **O(n + k)**, where k = max. This is O(n) only when k = O(n) |
| Extra memory | O(k): bad when the values are large (max = 1,000,000 means a 1,000,001-int array) |
| Restriction | non-negative integers only (shift by `min` to support negatives) |
| Stable | this version rebuilds values, so stability doesn't apply. The *prefix-sum* version of count sort (used inside radix sort on records) is stable |

---

## 10. Bucket / Bin Sort

### Idea
1. **Divide** the elements into groups called **buckets (bins)**.
2. Each bucket must be sorted.
3. **Gather** the buckets in order to form the sorted array.

In this version there is **one bin per value** (like count sort), but each bin is a
**linked list** that stores the elements themselves instead of a counter.

### Steps
1. Create an **array of pointers** (bins) of size **max + 1**, all `NULL`.
2. Scan the original array and put each element in the bin whose **index equals its value**.
3. Scan the bins from left to right and empty each bin **FIFO** back into the array.

### Trace: `[6, 8, 3, 10, 15, 6, 9, 12, 6, 3]`, max = 15

```
bin:  0  1  2  3      4  5  6        7  8   9   10   11  12   13 14 15
      /  /  /  3->3   /  /  6->6->6  /  8   9   10   /   12   /  /  15

gather -> 3 3 6 6 6 8 9 10 12 15
```

### FIFO bin (linked-list queue)

Insert at the **tail**, delete from the **head**: first in, first out.
Keeping a `tail` pointer makes insertion **O(1)** instead of walking the whole list.

```cpp
struct Node
{
    int   data;
    Node* next;
};

struct Bin
{
    Node* head = nullptr;
    Node* tail = nullptr;
};

void insert(Bin& bin, int value)          // insert at tail: O(1)
{
    Node* t = new Node{value, nullptr};
    if (bin.head == nullptr)
        bin.head = bin.tail = t;
    else
    {
        bin.tail->next = t;
        bin.tail = t;
    }
}

int deleteHead(Bin& bin)                  // delete from head: O(1)
{
    Node* t = bin.head;
    int value = t->data;
    bin.head = t->next;
    if (bin.head == nullptr)
        bin.tail = nullptr;
    delete t;
    return value;
}
```

### Implementation

```cpp
void binSort(int* arr, int n)
{
    int max = findMax(arr, n);
    vector<Bin> bins(max + 1);               // all empty (NULL)

    for (int i = 0; i < n; i++)
        insert(bins[arr[i]], arr[i]);

    int i = 0;                               // for array of bins
    int j = 0;                               // for original array
    while (i < max + 1)
    {
        while (bins[i].head != nullptr)
            arr[j++] = deleteHead(bins[i]);
        i++;
    }
}
```

### Analysis
| | |
|---|---|
| Time | O(n + k) with one bin per value and O(1) tail insertion |
| Extra memory | O(n + k): k bins + n nodes |
| Stable | ✅ (FIFO) |

> **General bucket sort** (e.g. for floats in [0, 1)) uses *fewer* buckets, each covering a
> **range** of values, and sorts each bucket (usually with insertion sort). There the
> average is O(n + k), and the **worst case is O(n²)** when all elements fall into
> **one bucket**.

---

## 11. Radix Sort

### Idea
Like bin sort, but uses only **10 bins (digits 0–9)** instead of max + 1 bins, so it needs
**much less space**. Elements are distributed **digit by digit**, starting from the
**least significant digit (LSD)**.

### Steps
1. Find the **max** element. Its **number of digits = number of passes**.
   max = 348 means 3 digits, so 3 passes.
2. Create an array of bins whose size = number of possible digit values
   (base 10 means 10 bins, 0…9; base 8 means 8 bins, 0…7). All start `NULL`.
3. **Pass 1** (units digit): insert each element into bin `(arr[i] / 1) % 10`, then
   collect the bins back **FIFO**.
4. **Pass 2** (tens digit): bin `(arr[i] / 10) % 10`, collect FIFO.
5. **Pass 3** (hundreds digit): bin `(arr[i] / 100) % 10`, collect FIFO. The array is now sorted.

> **FIFO is essential.** Each pass must be **stable** so the order from the previous digit is kept.

### Trace: `[237, 146, 259, 348, 152, 163, 235, 48, 36, 62]`

**Pass 1: units digit**
```
bin 2: 152 -> 62
bin 3: 163
bin 5: 235
bin 6: 146 -> 36
bin 7: 237
bin 8: 348 -> 48
bin 9: 259
collect: 152 62 163 235 146 36 237 348 48 259        (not sorted yet)
```

**Pass 2: tens digit**
```
bin 3: 235 -> 36 -> 237
bin 4: 146 -> 348 -> 48
bin 5: 152 -> 259
bin 6: 62 -> 163
collect: 235 36 237 146 348 48 152 259 62 163        (not sorted yet)
```

**Pass 3: hundreds digit** (036, 048, 062 have hundreds digit 0)
```
bin 0: 36 -> 48 -> 62
bin 1: 146 -> 152 -> 163
bin 2: 235 -> 237 -> 259
bin 3: 348
collect: 36 48 62 146 152 163 235 237 259 348        sorted ✔
```

### Implementation (uses the same `Bin` from section 10)

```cpp
void radixSort(int* arr, int n)
{
    const int BASE = 10;
    int max = findMax(arr, n);
    vector<Bin> bins(BASE);

    for (int exp = 1; max / exp > 0; exp *= BASE)   // one pass per digit
    {
        for (int i = 0; i < n; i++)                  // distribute
            insert(bins[(arr[i] / exp) % BASE], arr[i]);

        int j = 0;
        for (int i = 0; i < BASE; i++)               // collect back (FIFO)
            while (bins[i].head != nullptr)
                arr[j++] = deleteHead(bins[i]);
    }
}
```

`exp` = 1, 10, 100, … gives the digit `(arr[i] / exp) % 10`.

### Analysis
| | |
|---|---|
| Time | **O(d · (n + b))**, written **O(d·n)** for base b = 10. d = digits of max = number of passes |
| Extra memory | O(n + b): n nodes + b bins |
| Stable | ✅ |
| Restriction | non-negative integers (or fixed-length keys such as strings of equal length) |

---

## 12. Shell Sort

### Why?
Insertion sort moves an element **one position at a time**. In a large array, an element
far from its correct place needs **many shifts**, which is very slow.

Shell sort is an **enhanced insertion sort** for large lists:
- **Move elements long distances early.**
- **Fix the large disorder first, then fine-tune the rest later.**

### Steps
1. Choose a **gap**: compare and move elements that are `gap` positions apart.
2. Large elements jump close to their correct positions quickly.
3. After each pass, **halve the gap**. The array becomes more and more sorted.
4. When **gap = 1**, it is plain insertion sort on an almost-sorted array, so it is **very fast**.

Gap sequence used here (Shell's original): `n/2, n/4, …, 1`.

### Trace: `[19, 22, 63, 105, 2, 46, 12, 31]`, n = 8

**Pass 1: gap = 8/2 = 4**
```
19 <-> 2     compare, swap
22  -  46    compare
63 <-> 12    compare, swap
105 <-> 31   compare, swap
result: 2 22 12 31 19 46 63 105
```

**Pass 2: gap = 4/2 = 2** (compare all elements 2 apart)
```
2-12, 22-31, 12-19, 31-46, 19-63, 46-105   all in order, no change
result: 2 22 12 31 19 46 63 105
```

**Pass 3: gap = 2/2 = 1**: simple insertion sort
```
result: 2 12 19 22 31 46 63 105   sorted ✔
```

### Implementation

```cpp
void shellSort(int* arr, int n)
{
    int gap, i, j, temp;

    // loop for passes: gap decides how far apart compared elements are
    for (gap = n / 2; gap >= 1; gap = gap / 2)
    {
        // pick the element to be inserted in its correct place
        for (i = gap; i < n; i++)
        {
            temp = arr[i];          // element being inserted
            j = i - gap;            // element 'gap' positions before temp

            // while the element 'gap' before temp is larger than it
            while (j >= 0 && arr[j] > temp)
            {
                arr[j + gap] = arr[j];  // move the larger element gap forward
                j = j - gap;            // keep comparing further back by gap
            }
            arr[j + gap] = temp;    // put temp in its correct place
        }
    }
}
```

> It is exactly insertion sort with `1` replaced by `gap`. `sorting.cpp` also has
> `shellSortTrace()`, which prints the array after every pass.

### Analysis
| | |
|---|---|
| Passes | about log₂ n |
| Time | **depends on the gap sequence**. With n/2, n/4, …: worst **O(n²)**. With Hibbard (1, 3, 7, 15, …) or Knuth (1, 4, 13, 40, …) gaps: worst **O(n^{3/2})**. Typical runs are much faster than insertion sort |
| Best | O(n log n): already sorted, about n comparisons per pass |
| Extra memory | O(1) |
| Stable | ❌ long jumps can reorder equal elements |
| Adaptive | ✅ |

---

## 13. Heap Sort

### Idea
Store the array as a **complete binary tree**: the children of index `i` are `2i+1` and
`2i+2`. In a **max heap** every parent is ≥ its children, so the **largest element is at the root**.

### Steps
1. **Build a max heap**: call `heapify` on every non-leaf node from `n/2 − 1` down to `0`.
2. Swap the root (the max) with the **last** element. The max is now in its final place.
3. Shrink the heap by one and `heapify` the root to restore the heap.
4. Repeat steps 2–3 until the heap has one element.

### Trace: `[4, 10, 3, 5, 1]`
```
build max heap        -> 10 5 3 4 1
swap 10 with 1        -> 1 5 3 4 | 10   heapify -> 5 4 3 1 | 10
swap 5 with 1         -> 1 4 3 | 5 10   heapify -> 4 1 3 | 5 10
swap 4 with 3         -> 3 1 | 4 5 10   heapify -> 3 1 | 4 5 10
swap 3 with 1         -> 1 | 3 4 5 10
sorted: 1 3 4 5 10
```

### Implementation

```cpp
void heapify(int* arr, int n, int i)     // push arr[i] down to its place
{
    int largest = i;
    int left  = 2 * i + 1;
    int right = 2 * i + 2;

    if (left  < n && arr[left]  > arr[largest]) largest = left;
    if (right < n && arr[right] > arr[largest]) largest = right;

    if (largest != i)
    {
        swap(arr[i], arr[largest]);
        heapify(arr, n, largest);
    }
}

void heapSort(int* arr, int n)
{
    for (int i = n / 2 - 1; i >= 0; i--)  // 1) build max heap
        heapify(arr, n, i);

    for (int i = n - 1; i > 0; i--)       // 2) move max to the end
    {
        swap(arr[0], arr[i]);
        heapify(arr, i, 0);               //    heap size is now i
    }
}
```

### Analysis
| | |
|---|---|
| Time | **O(n log n)** in best, average and worst cases (building the heap is O(n)) |
| Extra memory | O(1), in place |
| Stable | ❌ |
| Adaptive | ❌ |

---

## 14. Tree Sort

### Idea
Insert all elements into a **Binary Search Tree (BST)**; an **in-order traversal**
(left, root, right) visits them in sorted order.

### Steps
1. Insert each element into the BST: smaller goes **left**, greater or equal goes **right**.
2. Traverse the tree **in-order** and write each visited value back into the array.

### Trace: `[50, 70, 60, 90, 40, 80, 10, 20, 30]`
```
              50
            /    \
          40      70
         /       /  \
        10      60   90
          \         /
          20       80
            \
            30
in-order: 10 20 30 40 50 60 70 80 90
```

### Implementation

```cpp
struct TreeNode
{
    int       data;
    TreeNode* left;
    TreeNode* right;
};

TreeNode* bstInsert(TreeNode* root, int value)
{
    if (root == nullptr)
        return new TreeNode{value, nullptr, nullptr};
    if (value < root->data)
        root->left  = bstInsert(root->left, value);
    else
        root->right = bstInsert(root->right, value);   // equal -> right (stable)
    return root;
}

void inorder(TreeNode* root, int* arr, int& j)   // also frees the tree
{
    if (root == nullptr) return;
    inorder(root->left, arr, j);
    arr[j++] = root->data;
    inorder(root->right, arr, j);
    delete root;
}

void treeSort(int* arr, int n)
{
    TreeNode* root = nullptr;
    for (int i = 0; i < n; i++)
        root = bstInsert(root, arr[i]);

    int j = 0;
    inorder(root, arr, j);
}
```

### Analysis
| | |
|---|---|
| Average | O(n log n): balanced tree, height ≈ log n |
| Worst | **O(n²)**: sorted input makes the tree a "linked list" of height n. A self-balancing tree (AVL / Red-Black) guarantees O(n log n) |
| Extra memory | O(n) nodes |
| Stable | ✅ (equal values go right and are visited after) |

---

## 15. Summary Table

| Algorithm | Best | Average | Worst | Extra memory | Stable | Adaptive |
|-----------|------|---------|-------|--------------|:------:|:--------:|
| Bubble (flag) | O(n) | O(n²) | O(n²) | O(1) | ✅ | ✅ |
| Insertion | O(n) | O(n²) | O(n²) | O(1) | ✅ | ✅ |
| Selection | O(n²) | O(n²) | O(n²) | O(1) | ❌ | ❌ |
| Quick | O(n log n) | O(n log n) | O(n²) | O(log n) stack | ❌ | ❌ |
| Merge | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ | ❌ |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ | ❌ |
| Tree (plain BST) | O(n log n) | O(n log n) | O(n²) | O(n) | ✅ | ❌ |
| Shell (n/2 gaps) | O(n log n) | depends on gaps | O(n²) | O(1) | ❌ | ✅ |
| Count | O(n + k) | O(n + k) | O(n + k) | O(k) | — | ❌ |
| Bin (1 bin/value) | O(n + k) | O(n + k) | O(n + k) | O(n + k) | ✅ | ❌ |
| Radix (base 10) | O(d·n) | O(d·n) | O(d·n) | O(n + 10) | ✅ | ❌ |

`k` = max value, `d` = number of digits of max.

---

## 16. Sorting in the STL

In real programs you rarely write a sort by hand: the C++ Standard Library already
provides fast, well-tested implementations of the same ideas from this chapter.
Most live in `<algorithm>`.

### 16.1 Main sorting functions

| Function | What it does | Algorithm inside | Time | Stable |
|----------|--------------|------------------|------|:------:|
| `std::sort(first, last)` | Sorts the range ascending | **Introsort** = quick sort, switching to **heap sort** if recursion gets too deep, and **insertion sort** for small parts | O(n log n) guaranteed | ❌ |
| `std::stable_sort(first, last)` | Sorts, keeping equal elements in order | **Merge sort** (falls back to an in-place merge if no memory) | O(n log n) with extra memory, O(n log² n) without | ✅ |
| `std::partial_sort(first, middle, last)` | Only the smallest `k = middle − first` elements, sorted, at the front | **Heap** | O(n log k) | ❌ |
| `std::nth_element(first, nth, last)` | Puts the element that belongs at `nth` there; smaller ones before it, greater ones after (unordered) | **Quick-select** (the partition step of quick sort) | O(n) average | ❌ |
| `std::is_sorted(first, last)` | Checks whether a range is sorted | Linear scan | O(n) | — |
| `list.sort()` / `forward_list.sort()` | Member function for linked lists | **Merge sort** on nodes | O(n log n) | ✅ |
| `std::ranges::sort(v)` (C++20) | Same as `std::sort`, takes the container and an optional **projection** | Introsort | O(n log n) | ❌ |
| `qsort(arr, n, size, cmp)` (C, `<cstdlib>`) | C library sort with a `void*` comparator | Implementation-defined | no guarantee | ❌ |

> `std::sort` and `std::stable_sort` need **random-access iterators** (array, `vector`,
> `deque`, `string`). A `std::list` cannot jump to the middle, so it has its own
> member `list.sort()`. This is the same reason merge sort suits linked lists.

### 16.2 Building blocks of the chapter's algorithms

| Chapter algorithm | STL equivalent |
|-------------------|----------------|
| Merge step of merge sort | `std::merge` (two ranges into a third), `std::inplace_merge` |
| Partition step of quick sort | `std::partition`, `std::stable_partition` |
| Heap sort | `std::make_heap` (O(n)) + `std::sort_heap` (O(n log n)); `std::priority_queue` is a ready-made heap |
| Tree sort | `std::set` / `std::multiset` / `std::map`: balanced BSTs (red-black trees); iterating them visits keys in sorted order, O(n log n) total |
| Insertion into a sorted array | `std::lower_bound` / `std::upper_bound` to find the position + `vector::insert` |
| Count / Bin / Radix | no STL version, written by hand (or count with `std::map<int,int>`) |

### 16.3 Comparators

By default the STL sorts with `operator<` (ascending). Pass a **comparator** to change the order:

```cpp
sort(v.begin(), v.end());                                   // ascending
sort(v.begin(), v.end(), greater<int>());                   // descending
sort(v.begin(), v.end(), [](int a, int b) { return a > b; });  // descending (lambda)
```

The comparator must answer **"should `a` come before `b`?"** and must be a
**strict weak ordering**:
- Use `<` or `>`, **never `<=` or `>=`**: `comp(x, x)` must be `false`.
- Breaking this rule is **undefined behaviour**: `std::sort` may crash or read past the array.

Sorting structs, e.g. by grade (descending) while keeping the original order of equal grades:

```cpp
struct Student { string name; int grade; };

vector<Student> s = {{"Ali", 90}, {"Mona", 85}, {"Omar", 90}, {"Sara", 85}};
stable_sort(s.begin(), s.end(),
            [](const Student& a, const Student& b) { return a.grade > b.grade; });
// Ali(90) Omar(90) Mona(85) Sara(85)  -> equal grades keep input order

ranges::sort(s, {}, &Student::name);    // C++20: sort by the 'name' member
```

> C++17 also offers parallel sorting: `sort(std::execution::par, v.begin(), v.end());`
> (header `<execution>`; with GCC it needs Intel TBB, linked with `-ltbb`).

### 16.4 Complete example

The full program is in [`stl_sorting.cpp`](stl_sorting.cpp):

```bash
g++ -std=c++20 -Wall -Wextra -o stl_sorting stl_sorting.cpp
./stl_sorting
```

```cpp
#include <iostream>
#include <vector>
#include <list>
#include <set>
#include <string>
#include <queue>
#include <algorithm>   // sort, stable_sort, partial_sort, nth_element, heaps...
#include <functional>  // std::greater
#include <cstdlib>     // qsort
using namespace std;

void print(const vector<int>& v)
{
    for (int x : v) cout << x << ' ';
    cout << '\n';
}

int cmpInt(const void* a, const void* b)          // comparator for C qsort
{
    int x = *(const int*)a, y = *(const int*)b;
    return (x > y) - (x < y);                     // -1, 0, 1 (no overflow)
}

int main()
{
    // std::sort: ascending, then descending
    vector<int> v = {50, 70, 60, 90, 40, 80, 10, 20, 30};
    sort(v.begin(), v.end());
    sort(v.begin(), v.end(), greater<int>());

    // plain C array: pointers are iterators
    int arr[] = {8, 5, 7, 3, 2};
    int n = sizeof(arr) / sizeof(arr[0]);
    sort(arr, arr + n);

    // partial_sort: only the 3 smallest, sorted, at the front
    vector<int> p = {8, 6, 3, 10, 9, 4, 12, 5, 2, 7};
    partial_sort(p.begin(), p.begin() + 3, p.end());   // 2 3 4 | rest unordered

    // nth_element: the median in O(n) average
    vector<int> m = {8, 6, 3, 10, 9, 4, 12, 5, 2, 7};
    nth_element(m.begin(), m.begin() + m.size() / 2, m.end());
    int median = m[m.size() / 2];                       // 7

    // list has its own sort (merge sort)
    list<int> lst = {8, 3, 7, 4, 9, 2, 6, 5};
    lst.sort();

    // heap sort with heap algorithms
    vector<int> h = {4, 10, 3, 5, 1};
    make_heap(h.begin(), h.end());
    sort_heap(h.begin(), h.end());                      // 1 3 4 5 10

    // priority_queue as a min-heap
    priority_queue<int, vector<int>, greater<int>> pq;
    for (int x : {6, 3, 9, 10, 15}) pq.push(x);        // pops 3 6 9 10 15

    // tree sort with multiset (keeps duplicates)
    multiset<int> ms = {6, 3, 9, 10, 15, 6, 8, 12, 3, 6};   // 3 3 6 6 6 8 9 10 12 15

    // merge step and partition step
    vector<int> a = {2, 5, 8, 12}, b = {3, 6, 7, 10}, out(8);
    merge(a.begin(), a.end(), b.begin(), b.end(), out.begin());   // 2 3 5 6 7 8 10 12

    vector<int> q = {50, 70, 60, 90, 40, 80, 10, 20, 30};
    partition(q.begin(), q.end(), [](int x) { return x <= 50; }); // <=50 first, then >50

    // C++20 ranges
    vector<int> r = {237, 146, 259, 348, 152, 163, 235, 48, 36, 62};
    ranges::sort(r);

    // C qsort
    int c[] = {6, 3, 9, 10, 15};
    qsort(c, 5, sizeof(int), cmpInt);

    print(v); print(p); print(h); print(out); print(r);
    cout << "median = " << median << '\n';
}
```

Output of the full `stl_sorting.cpp`:

```
sort ascending      : 10 20 30 40 50 60 70 80 90
sort descending     : 90 80 70 60 50 40 30 20 10
sort with lambda    : 10 20 30 40 50 60 70 80 90
sort C array        : 2 3 5 7 8
stable_sort by grade: Ali(90) Omar(90) Mona(85) Sara(85)
partial_sort (3)    : 2 3 4 10 9 8 12 6 5 7
median (nth_element): 7
is_sorted(v)?       : true
list::sort          : 2 3 4 5 6 7 8 9
make_heap+sort_heap : 1 3 4 5 10
priority_queue pops : 3 6 9 10 15
multiset (tree sort): 3 3 6 6 6 8 9 10 12 15
merge two sorted    : 2 3 5 6 7 8 10 12
partition (<= 50)   : 50 30 20 10 40 80 90 60 70
  first element > 50 at index 5
ranges::sort        : 36 48 62 146 152 163 235 237 259 348
ranges::sort by name: Ali Mona Omar Sara
qsort (C)           : 3 6 9 10 15
```

> `partial_sort` and `partition` only guarantee the part they promise. The order of the
> remaining elements can differ between compilers.

---

## 17. Which One Should I Use?

| Situation | Good choice |
|-----------|-------------|
| Small array (< ~20 elements) or nearly sorted | **Insertion sort** |
| Need the fewest writes/swaps (e.g. flash memory) | **Selection sort** |
| General purpose, fast in practice, in-place | **Quick sort** (random / median-of-three pivot) |
| Guaranteed O(n log n) and stable, or a linked list | **Merge sort** |
| Guaranteed O(n log n) with O(1) extra memory | **Heap sort** |
| Medium-size array, no recursion, little memory (embedded) | **Shell sort** |
| Integers in a small range | **Count sort** / **Bin sort** |
| Many integers with a fixed number of digits | **Radix sort** |

| Real production code | **`std::sort`**, or **`std::stable_sort`** when stability matters (see section 16) |
