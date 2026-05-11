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

# 04 - Pointers: Indirection and Memory Access

> [!IMPORTANT]
> **The Core of C++**: Pointers are the foundation of **all advanced C++ features**—linked data structures, dynamic allocation, inheritance via base pointers, polymorphism. Mastering pointers is non-negotiable for professional C++.

---

## The Memory Model

### How Memory Works

Every variable occupies **memory** at a specific **address**:

```
Memory Address    Variable        Value
0x1000           int x           10
0x1004           char c          'A'
0x1008           double d        3.14
0x100C           int y           42
```

### Address-of Operator: `&`

Get the memory address of a variable:

```cpp
int x = 10;
cout << x << endl;      // Output: 10 (the VALUE)
cout << &x << endl;     // Output: 0x7fff5fbff8ac (the ADDRESS)
```

### Pointer: Variable Holding an Address

```cpp
int x = 10;
int *ptr = &x;          // ptr stores ADDRESS of x

cout << ptr << endl;    // Address: 0x7fff5fbff8ac
cout << *ptr << endl;   // Value: 10 (dereference)
```

**Memory Visualization**:
```
Memory Address    Variable        Value
0x1000           x               10
0x2000           ptr             0x1000 (stores address of x)
```

---

## Pointer Operators

### `&` (Address-of)

```cpp
int num = 42;
int *address = &num;    // Get address
// address now holds the memory address of num
```

### `*` (Dereference)

```cpp
int num = 42;
int *ptr = &num;
cout << *ptr << endl;   // 42 (dereference: access what ptr points to)

*ptr = 100;             // Change value through pointer
cout << num << endl;    // 100 (num was modified!)
```

### `->`  (Arrow: Dereference + Member Access)

```cpp
struct Point {
    int x, y;
};

Point p = {10, 20};
Point *ptr = &p;

// These are equivalent:
cout << ptr->x << endl;         // 10 (arrow operator)
cout << (*ptr).x << endl;       // 10 (dereference then dot)
```

---

## Pointer Types

### Pointer-to-int

```cpp
int studentScore = 85;
int *scorePtr = &studentScore;  // scorePtr is a "pointer to int"
```

### Pointer-to-double

```cpp
double gpa = 3.75;
double *gpaPtr = &gpa;  // gpaPtr is a "pointer to double"
```

### Pointer-to-struct

```cpp
struct Student {
    int id;
    string name;
};

Student s = {101, "Ali"};
Student *ptr = &s;
cout << ptr->name << endl;  // "Ali"
```

### Void Pointer (Generic)

```cpp
int studentAge = 20;
void *genericPtr = &studentAge;     // Can point to any type
// BUT: must cast to use
int *agePtr = (int *)genericPtr;
cout << *agePtr << endl;   // 20
```

> [!CAUTION]
> Void pointers lose type information. Use sparingly; prefer typed pointers.

---

## Dynamic Memory Allocation

### The `new` Operator

Allocate memory at **runtime** (on the heap):

```cpp
int *studentCount = new int;  // Allocate single int on heap
*studentCount = 42;
cout << *studentCount << endl;  // 42

delete studentCount;           // FREE memory when done
studentCount = NULL;           // Mark as invalid
```

**Memory Layout**:
```
Stack                   Heap
┌──────┐
│ ptr  │──────────────→┌──────┐
│(addr)│              │  42  │ (allocated by new)
└──────┘              └──────┘
```

### Arrays on Heap

```cpp
int size = 10;
int *arr = new int[size];   // Dynamic array

arr[0] = 1;
arr[9] = 10;

delete[] arr;               // Must use delete[] for arrays!
arr = NULL;
```

> [!CAUTION]
> **Critical**: 
> - `new` requires `delete`
> - `new[]` requires `delete[]`
> - Forgetting `delete` = **memory leak**
> - Using `delete` instead of `delete[]` = **undefined behavior**

### Stack vs. Heap Allocation

| Aspect | Stack | Heap |
|--------|-------|------|
| **Allocation** | Automatic | Manual (`new`) |
| **Deallocation** | Automatic | Manual (`delete`) |
| **Speed** | Fast | Slower |
| **Size** | Limited | Large |
| **Lifetime** | Scope-based | Until `delete` |
| **Best for** | Known, small data | Dynamic, large data |

```cpp
// Stack: automatic cleanup (RAII principle)
{
    int score = 85;
}  // score destroyed automatically

// Heap: manual cleanup required (modern approach: use smart pointers)
{
    int *dynScore = new int(85);
    // delete dynScore;   // If you forget: MEMORY LEAK!
    // Better: use std::unique_ptr<int> dynScore(new int(85));
}
```

---

## Pointer Arithmetic

### Incrementing Pointers

```cpp
int arr[] = {10, 20, 30, 40, 50};
int *ptr = arr;                 // Points to arr[0]

cout << *ptr << endl;           // 10
ptr++;
cout << *ptr << endl;           // 20 (pointer moved to next int)
ptr++;
cout << *ptr << endl;           // 30

// Equivalent to:
ptr = &arr[2];
```

### Pointer Differences

```cpp
int arr[] = {10, 20, 30, 40, 50};
int *p1 = &arr[1];
int *p2 = &arr[4];

cout << (p2 - p1) << endl;      // 3 (distance in elements, not bytes)
```

---

## Complete Example: Swap Function

### Without Pointers (Doesn't Work)

```cpp
void swap_wrong(int score1, int score2) {
    int temp = score1;
    score1 = score2;
    score2 = temp;
    // Swaps local copies, original unchanged (pass-by-value)
}

int main() {
    int mathScore = 85, englishScore = 90;
    swap_wrong(mathScore, englishScore);
    cout << mathScore << " " << englishScore << endl;  // Still 85 90 (unchanged!)
}
```

### With Pointers (Works!)

```cpp
void swap_correct(int *score1Ptr, int *score2Ptr) {
    int temp = *score1Ptr;
    *score1Ptr = *score2Ptr;
    *score2Ptr = temp;
}

int main() {
    int mathScore = 85, englishScore = 90;
    swap_correct(&mathScore, &englishScore);  // Pass addresses
    cout << mathScore << " " << englishScore << endl;  // Now 90 85 (swapped!)
}
```

---

## Dangling Pointers and Memory Leaks

### Memory Leak: Forgot `delete`

```cpp
void leak() {
    int *ptr = new int(42);
    // forgot: delete ptr;
}  // Memory not freed! Leak.

int main() {
    for (int i = 0; i < 1000000; i++) {
        leak();  // Allocates millions of unfreed blocks
    }
    // Program consumes all available memory
    return 0;
}
```

### Dangling Pointer: Using After Delete

```cpp
int *ptr = new int(42);
cout << *ptr << endl;   // 42
delete ptr;
cout << *ptr << endl;   // ✗ UNDEFINED BEHAVIOR: ptr is dangling
ptr = NULL;             // Prevent accidental use
```

### Best Practice: RAII (Resource Acquisition Is Initialization)

```cpp
// ✓ Better: use smart pointers (advanced)
unique_ptr<int> ptr(new int(42));
// Automatically deletes when ptr goes out of scope
```

---

## Professional C++ Practices with Pointers

### 1. **Modern Approach: Smart Pointers (C++11+)**

Raw pointers require manual management (error-prone). Professional C++ uses smart pointers:

```cpp
// Modern C++: Use smart pointers instead of raw new/delete
#include <memory>

// std::unique_ptr: Exclusive ownership (can't copy)
std::unique_ptr<int> score(new int(85));
// Automatically deleted when score goes out of scope

// std::shared_ptr: Shared ownership (reference counted)
std::shared_ptr<Student> student = std::make_shared<Student>();
// Deleted when all references are gone

// Instead of:
int *ptr = new int(42);
delete ptr;

// Do:
auto ptr = std::make_unique<int>(42);
// No manual delete needed!
```

### 2. **RAII Principle (Resource Acquisition Is Initialization)**

Bind resource lifetimes to object lifetimes:

```cpp
class FileHandler {
    FILE* handle;
public:
    FileHandler(const string& filename) {
        handle = fopen(filename.c_str(), "r");  // Acquire
    }
    ~FileHandler() {
        if (handle) fclose(handle);  // Release in destructor
    }
};

// Safe: file automatically closes when handler is destroyed
{
    FileHandler handler("data.txt");
}  // destructor called here
```

### 3. **Pointer Naming Convention**

Use descriptive names that indicate purpose:

```cpp
// Professional naming:
int *studentCountPtr;          // What it points to
int *dataBuffer;               // What it buffers
int *nodePtr;                  // Often part of linked list

// Avoid:
int *p;                        // Too generic
int *ptr;                      // No indication of purpose
```

### 4. **Null Check Pattern**

Always validate pointers before use:

```cpp
bool processData(int *dataPtr) {
    if (!dataPtr) {            // Guard clause
        cerr << "Error: null pointer\n";
        return false;
    }
    // Safe to use dataPtr here
    *dataPtr = 42;
    return true;
}
```

### 5. **Pointer-to-Pointer (Advanced Pattern)**

When you need to modify what a pointer points to:

```cpp
void createStudent(Student **studentPtr) {
    *studentPtr = new Student();  // Modify the caller's pointer
}

int main() {
    Student *student = nullptr;
    createStudent(&student);  // Pass address of pointer
    // student now points to newly created Student
    delete student;
    return 0;
}
```

---

## Common Pointer Errors

| Error | Problem | Fix |
|-------|---------|-----|
| **Using null pointer** | Dereference NULL → crash | Check `if (ptr != NULL)` |
| **Memory leak** | Allocate but no delete | Use `delete` in destructor or smart pointers |
| **Dangling pointer** | Using deleted memory | Set `ptr = NULL` after delete or use smart pointers |
| **Wrong operator** | `delete` on `new[]` array | Use `delete[]` for arrays or `unique_ptr<int[]>` |
| **Uninitialized pointer** | Garbage address | Initialize: `ptr = NULL` or `ptr = &var` |

---

## Mastery Checklist

- [ ] Understand memory addresses with `&`
- [ ] Dereference pointers with `*`
- [ ] Declare pointer variables (`int *p`, `double *q`)
- [ ] Use `->`  for pointer-to-struct member access
- [ ] Allocate memory with `new`, deallocate with `delete`
- [ ] Use `new[]` and `delete[]` for arrays
- [ ] Perform pointer arithmetic (increment, subtraction)
- [ ] Avoid memory leaks and dangling pointers
- [ ] Understand stack vs. heap allocation
- [ ] Write swap functions using pointers

> [!EXAMPLE]
> **Interview Question**: "What's the difference between stack and heap allocation?"
>
> **Answer**:
> - **Stack**: Automatic, scope-based cleanup, limited size, fast
> - **Heap**: Manual cleanup with `delete`, large size, slower
> Use **heap for dynamic size or long lifetime**, **stack for known small size**.
