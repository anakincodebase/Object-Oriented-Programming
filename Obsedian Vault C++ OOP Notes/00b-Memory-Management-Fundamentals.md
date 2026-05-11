---
Author:
  - Afnan Ahmed
tags:
  - c
  - computer_organisation
Creation Date: 2024-08-17, 14:21
Last Date: 2024-09-03T23:10:27+08:00
References:
draft:
description:
---

# 00b - Memory Management: Stack, Heap, and Memory Organization

> [!IMPORTANT]
> **The Foundation of Everything**: How memory is organized determines what's possible in programming:
> - **Stack**: Fast, automatic cleanup, limited size → local variables
> - **Heap**: Flexible, manual cleanup (in C++), large size → dynamic structures
> - **Data Segment**: Static memory → global variables
> - **Code Segment**: Read-only → executable instructions
>
> Mastering memory is **essential** for OOP, data structures, and avoiding crashes/leaks.

---

## Process Memory Layout

When a C++ program runs, it gets memory organized into distinct regions:

```
┌─────────────────────────────┐ High Address (0xFFFFFFFF)
│     Command Line Args       │
│     Environment Variables   │
├─────────────────────────────┤
│         STACK               │ ← Grows DOWNWARD
│         (Local variables,   │   Automatic cleanup
│          function calls)    │   Limited size (~8MB)
│                             │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │ (Gap: unused memory)
│                             │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │
│         HEAP                │ ← Grows UPWARD
│     (Dynamic memory,        │   Manual cleanup
│      new/delete)            │   Large size (~GB)
├─────────────────────────────┤
│     UNINITIALIZED DATA      │ (BSS segment)
│     (Global uninitialized)  │
├─────────────────────────────┤
│     INITIALIZED DATA        │ (Data segment)
│     (Global initialized)    │
├─────────────────────────────┤
│     CODE SEGMENT            │ Low Address (0x00000000)
│     (Executable instructions)
└─────────────────────────────┘
```

---

## The Stack: LIFO (Last In, First Out)

### How Stack Works

```
Function calls create STACK FRAMES (local scope):

main() {
    int x = 5;           ← Push x onto stack
    {
        int y = 10;      ← Push y onto stack (inner scope)
        int z = 15;      ← Push z onto stack
    }
    // y, z destroyed (popped)
    // x still exists
}
// x destroyed when main exits
```

### Visual Stack Timeline

```
TIMELINE:
Step 1: Call main()
┌────────────────┐
│    Return Addr │ ← Stack pointer
│    (main)      │
└────────────────┘

Step 2: Declare int x = 5
┌────────────────┐
│    Return Addr │
│    x = 5       │ ← Stack pointer
└────────────────┘

Step 3: Enter inner block, declare int y = 10
┌────────────────┐
│    Return Addr │
│    x = 5       │
│    y = 10      │ ← Stack pointer
└────────────────┘

Step 4: Declare int z = 15
┌────────────────┐
│    Return Addr │
│    x = 5       │
│    y = 10      │
│    z = 15      │ ← Stack pointer
└────────────────┘

Step 5: Exit inner block
┌────────────────┐
│    Return Addr │
│    x = 5       │ ← Stack pointer (y, z popped)
│   [garbage]    │
│   [garbage]    │
└────────────────┘
```

### Stack Code Example

```cpp
#include <iostream>
using namespace std;

void displayAddress(int *ptr, const char *name) {
    cout << name << " address: " << ptr 
         << " value: " << *ptr << endl;
}

int main() {
    int a = 10;
    
    {
        int b = 20;
        int c = 30;
        
        cout << "\nInside inner block:\n";
        displayAddress(&a, "a");
        displayAddress(&b, "b");
        displayAddress(&c, "c");
    }
    
    cout << "\nAfter inner block exits:\n";
    cout << "a still exists: " << a << endl;
    
    // cout << b << endl;  // ✗ COMPILE ERROR: b out of scope
    
    return 0;
}
```

**Output:**
```
Inside inner block:
a address: 0x7ffc2b3ff2cc value: 10
b address: 0x7ffc2b3ff2c8 value: 20
c address: 0x7ffc2b3ff2c4 value: 30

After inner block exits:
a still exists: 10
```

> [!NOTE]
> Notice: **Higher addresses come first** (0xcc > 0xc8 > 0xc4). Stack grows **downward** toward lower addresses.

### Stack Characteristics

| Aspect | Detail |
|--------|--------|
| **Speed** | ✅ Very fast - single pointer increment |
| **Size** | ⚠️ Limited (~8 MB typical) |
| **Allocation** | ✅ Automatic at compile time (compiler knows size) |
| **Deallocation** | ✅ Automatic when scope exits |
| **Thread-safe** | ✅ Each thread has own stack |
| **Fragmentation** | ✅ None - LIFO order prevents it |
| **Lifetime** | ⏱️ Scope-based (function/block) |

---

## The Heap: Dynamic Memory

### How Heap Works

```
Heap is a FREE POOL where you request memory at runtime:

┌─────────────────────┐
│  Free Memory Pool   │ (unallocated)
│                     │
│  ┌───────────────┐  │
│  │   Used Block  │  │
│  │   (object)    │  │
│  └───────────────┘  │
│  ┌───────────────┐  │
│  │   Free Space  │  │
│  └───────────────┘  │
│  ┌───────────────┐  │
│  │   Used Block  │  │
│  │   (array)     │  │
│  └───────────────┘  │
│                     │
└─────────────────────┘

Allocate: new → Find free block large enough
Deallocate: delete → Mark block as free
```

### Heap Code Example

```cpp
#include <iostream>
using namespace std;

int main() {
    // Stack allocation (automatic)
    int stackVar = 100;
    
    // Heap allocation (manual)
    int *heapVar = new int(200);
    
    cout << "Stack variable: " << stackVar 
         << " at address " << &stackVar << endl;
    cout << "Heap variable: " << *heapVar 
         << " at address " << heapVar << endl;
    
    // Must manually free heap memory
    delete heapVar;
    heapVar = NULL;  // Good practice
    
    // Stack memory freed automatically
    return 0;
}
```

**Output:**
```
Stack variable: 100 at address 0x7ffc2b3ff2cc
Heap variable: 200 at address 0x55555556a8e0
```

> [!NOTE]
> Heap addresses are typically much smaller than stack addresses, and non-contiguous (notice the gap: 0x55... vs 0x7ff...).

### Heap Allocation: `new` and `delete`

```cpp
// Allocate single object
int *ptr = new int;           // Allocate space
*ptr = 42;                    // Initialize
delete ptr;                   // Free space
ptr = NULL;                   // Prevent dangling pointer

// Allocate array
int *arr = new int[100];      // Allocate 100 ints
arr[0] = 10;
delete[] arr;                 // Must use delete[] for arrays!
arr = NULL;

// Allocate struct/class
struct Student {
    string name;
    int id;
};

Student *student = new Student();
student->name = "Ali";
delete student;
student = NULL;
```

### Heap Characteristics

| Aspect | Detail |
|--------|--------|
| **Speed** | ⚠️ Slower - must find free block, maintain metadata |
| **Size** | ✅ Large (GBs available on modern systems) |
| **Allocation** | ⏱️ Dynamic at runtime (you request with `new`) |
| **Deallocation** | ⚠️ Manual (`delete` required or memory leaks) |
| **Thread-safe** | ❌ Not thread-safe without synchronization |
| **Fragmentation** | ❌ Possible - scattered allocations cause gaps |
| **Lifetime** | ⏱️ Until you explicitly `delete` |

---

## Stack vs. Heap: When to Use Each

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    // ✅ USE STACK for:
    // - Known, small size at compile time
    // - Local variables in functions
    // - Fixed-size arrays
    
    int count = 10;
    double scores[100];          // Fixed size
    
    // ✅ USE HEAP for:
    // - Unknown size at compile time
    // - Need data to persist beyond function
    // - Large data that might overflow stack
    // - Shared data between functions
    
    cout << "How many students? ";
    cin >> count;
    
    // Size determined at runtime → use heap
    int *grades = new int[count];
    
    for (int i = 0; i < count; i++) {
        grades[i] = 0;
    }
    
    // ... use grades ...
    
    delete[] grades;
    grades = NULL;
    
    // ✅ MODERN C++: Use containers (automatic)
    vector<int> modernGrades(count);  // Heap + automatic cleanup!
    
    return 0;
}
```

| Scenario | Use | Why |
|----------|-----|-----|
| Local variable in function | Stack | Automatic cleanup |
| Fixed array (size known) | Stack | Fast access, no overhead |
| Size unknown at compile time | Heap | Flexible size |
| Data needs to outlive function | Heap | Lives beyond scope |
| Large data (>1MB) | Heap | Stack too small |
| Many allocations/deallocations | Heap | Not stack LIFO |
| Modern C++ | Vector/String/Smart ptrs | Automatic, safe |

---

## Memory Leaks: The Silent Killer

### Leak: Forgetting `delete`

```cpp
// ❌ MEMORY LEAK: Allocate but don't delete
void leakMemory() {
    int *ptr = new int(42);
    // Forgot to delete!
}

int main() {
    for (int i = 0; i < 1000000; i++) {
        leakMemory();  // Allocates 1M blocks of memory
    }
    // Program uses GB of RAM for nothing!
    return 0;
}
```

### Leak: Lost Pointer

```cpp
// ❌ MEMORY LEAK: Pointer lost before delete
void losePointer() {
    int *ptr1 = new int(10);
    int *ptr2 = new int(20);
    
    ptr1 = ptr2;  // ptr1 now points to second block
    // Original first block is now unreachable!
    
    delete ptr2;  // Only deletes second block
    // First block leaked forever
}
```

### Leak: Exception Before Delete

```cpp
// ❌ MEMORY LEAK: Exception before delete
void processData(int *data) {
    if (data == NULL) {
        throw runtime_error("Null data");
        // delete data never runs!
    }
    // ... use data ...
    delete data;
}

int main() {
    int *myData = new int(100);
    try {
        processData(nullptr);  // Throws exception
        delete myData;  // Never reached!
    } catch (const exception& e) {
        cerr << e.what() << endl;
    }
}
```

### ✅ Fix: Modern C++ (Smart Pointers)

```cpp
#include <memory>
using namespace std;

void noLeaks() {
    // std::unique_ptr automatically deletes
    unique_ptr<int> ptr(new int(42));
    
    if (ptr == nullptr) {
        throw runtime_error("Error");
        // ptr destroyed automatically
    }
    
    // ... use ptr ...
    // ptr destroyed automatically at scope exit
}  // Memory automatically freed!
```

---

## Dangling Pointers: Using Deleted Memory

### Dangling After Delete

```cpp
// ❌ DANGLING POINTER: Using memory after delete
int *ptr = new int(42);
cout << *ptr << endl;      // OK: 42

delete ptr;

cout << *ptr << endl;      // ✗ UNDEFINED BEHAVIOR!
ptr->someMethod();         // ✗ UNDEFINED BEHAVIOR!
// Might crash, might print garbage, might work
```

### ✅ Fix: Set to NULL

```cpp
// ✓ GOOD: Set to NULL after delete
int *ptr = new int(42);
delete ptr;
ptr = NULL;

if (ptr != NULL) {
    cout << *ptr << endl;  // Will never execute
}
```

### ✅ Fix: Use Smart Pointers

```cpp
unique_ptr<int> ptr(new int(42));
// ptr automatically set to valid or invalid state
// Can't accidentally use deleted memory
```

---

## Stack Overflow: Exceeding Stack Limits

### Recursive Infinite Loop

```cpp
// ❌ STACK OVERFLOW: Infinite recursion
void infinite() {
    int localData[1000];
    infinite();  // Calls itself infinitely
    // Stack grows until overflow crash!
}
```

**Result**: `Segmentation fault (core dumped)`

### ✅ Fix: Base Case

```cpp
// ✓ GOOD: Recursion with base case
int factorial(int n) {
    if (n <= 1) return 1;  // Base case
    return n * factorial(n - 1);
}
```

### Stack Overflow: Large Local Variables

```cpp
// ❌ STACK OVERFLOW: Too much stack data
void process() {
    int hugeArray[10000000];  // 40MB on stack!
    // Stack is usually ~8MB → OVERFLOW
}

// ✓ GOOD: Use heap for large data
void processFixed() {
    vector<int> hugeArray(10000000);  // Uses heap
}
```

---

## Heap Fragmentation: Lost Efficiency

### Fragmentation Problem

```
Allocations:
new int[1000]    → [████████████████]
new int[500]     → [████████]
new int[1000]    → [████████████████]

Deallocations:
delete first     → [            ][████████][████████████████]
delete third     → [            ][████████][                ]

Now want: new int[1500]
Free space: 1000 + 1000 = 2000, but in separate blocks!
Can't allocate contiguous 1500 → ALLOCATION FAILS or SLOW
```

### Prevention: Use Container Classes

```cpp
// Container classes (std::vector, etc.) manage fragmentation
vector<int> data;
data.push_back(10);  // Allocator handles fragmentation
data.push_back(20);
data.push_back(30);

// Better than manual new/delete for many objects
```

---

## Static Memory: Global Variables

### How Statics Work

```cpp
#include <iostream>
using namespace std;

// Global variable (data segment)
int globalCount = 0;

void incrementGlobal() {
    globalCount++;
}

int main() {
    cout << "Global: " << globalCount << endl;  // 0
    
    incrementGlobal();
    cout << "Global: " << globalCount << endl;  // 1
    
    incrementGlobal();
    cout << "Global: " << globalCount << endl;  // 2
    
    return 0;
}
```

### Static Characteristics

| Aspect | Detail |
|--------|--------|
| **Lifetime** | Entire program run |
| **Scope** | Global (all functions) or file (static keyword) |
| **Speed** | ✅ Fast - known at compile time |
| **Size** | Limited by data segment |
| **Initialization** | Before main() runs |
| **Thread-safe** | ❌ Shared across threads |

### ⚠️ Warning: Avoid Global Variables

```cpp
// ❌ ANTI-PATTERN: Global state
int globalCounter = 0;

void incrementCounter() {
    globalCounter++;
}

// Hard to test, understand, maintain
// Multiple functions depend on global state

// ✓ BETTER: Pass parameters
void incrementCounter(int &counter) {
    counter++;
}
```

---

## Code Segment: Read-Only Memory

### How Code Works

```cpp
#include <iostream>
using namespace std;

// Code is stored in read-only segment
void printMessage() {
    cout << "This is in code segment" << endl;  // Instructions
}

int main() {
    printMessage();  // Call instruction
    
    // ✗ Can't modify code at runtime
    // (C doesn't allow this; bad security)
    
    return 0;
}
```

### String Literals in Code Segment

```cpp
const char *message = "Hello";  // "Hello" in code segment

// Careful: Don't modify string literals!
// message[0] = 'J';  // ✗ Undefined behavior

// Use std::string for mutable strings
string mutableMsg = "Hello";
mutableMsg[0] = 'J';  // ✓ OK
```

---

## Professional Best Practices

### 1. RAII (Resource Acquisition Is Initialization)

```cpp
// ✓ GOOD: Resource tied to object lifetime
class FileHandle {
    FILE *file;
public:
    FileHandle(const char *filename) {
        file = fopen(filename, "r");  // ACQUIRE
    }
    
    ~FileHandle() {
        if (file) fclose(file);       // RELEASE
    }
};

{
    FileHandle handle("data.txt");
    // File automatically closed when handle destroyed
}
```

### 2. Modern Memory Management

```cpp
// ✗ OLD: Raw pointers
int *ptr = new int(42);
delete ptr;

// ✓ MODERN: Smart pointers
auto ptr = make_unique<int>(42);
// Deleted automatically at scope exit!

// ✓ MODERN: Containers
vector<int> data;
data.push_back(42);
// Automatic allocation/deallocation
```

### 3. Memory Bounds Checking

```cpp
// ✓ GOOD: Check bounds
vector<int> data = {1, 2, 3};

try {
    int value = data.at(10);  // Throws exception if out of bounds
} catch (const out_of_range& e) {
    cerr << "Index out of range: " << e.what() << endl;
}

// ✗ RISKY: No bounds check
int value = data[10];  // Undefined behavior!
```

---

## Common Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **Forget `delete`** | Memory leak | Always pair `new` with `delete` |
| **Use `delete` instead of `delete[]`** | Undefined behavior | Use `delete[]` for arrays |
| **Use after delete** | Dangling pointer | Set `ptr = NULL` after delete |
| **Stack overflow** | Crash | Use heap for large data |
| **Heap fragmentation** | Performance | Use containers (vector, etc.) |
| **Lost pointer** | Memory leak | Never reassign without delete |
| **Global variable hell** | Hard to debug | Use local scope, pass parameters |
| **Exception before delete** | Memory leak | Use RAII / smart pointers |

---

## Mastery Checklist

- [ ] Explain memory layout: stack, heap, data, code segments
- [ ] Understand LIFO stack behavior and automatic cleanup
- [ ] Understand dynamic heap allocation with `new`/`delete`
- [ ] Pair every `new` with corresponding `delete` or `delete[]`
- [ ] Set pointers to `NULL` after `delete`
- [ ] Identify when to use stack vs. heap
- [ ] Recognize and prevent memory leaks
- [ ] Identify dangling pointers
- [ ] Avoid stack overflow with proper recursion base cases
- [ ] Understand heap fragmentation and its costs
- [ ] Appreciate RAII principle for resource management
- [ ] Use modern C++ (smart pointers, containers) instead of raw pointers
- [ ] Know about static memory (global variables) and their pitfalls

> [!EXAMPLE]
> **Interview Question**: "Explain the difference between stack and heap memory. When would you use each?"
>
> **Answer**: 
> - **Stack**: Automatic LIFO allocation/deallocation, fast, limited size (~8MB), used for local variables with known size at compile time
> - **Heap**: Manual allocation with `new`, deallocation with `delete`, slow, large size (~GBs), used for dynamic size determined at runtime
>
> **When to use:**
> - Stack: Integers, small structs, local variables
> - Heap: Vectors, strings, dynamic arrays, large objects
> 
> **Modern C++**: Use `std::vector`, `std::unique_ptr`, `std::shared_ptr` instead of manual `new`/`delete` to avoid leaks.
