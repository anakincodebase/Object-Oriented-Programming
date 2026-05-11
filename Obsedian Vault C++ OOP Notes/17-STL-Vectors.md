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

# 17 - STL Vectors: Dynamic Arrays Done Right

> [!IMPORTANT]
> **Standard Template Library**: `std::vector<T>` is a **dynamic array** that automatically manages memory. It's the modern replacement for raw `new[]`/`delete[]` arrays. Vectors are containers that grow as needed with RAII semantics.

---

## Vector Basics

### Creation and Initialization

```cpp
#include <vector>
using namespace std;

vector<int> v1;                    // Empty vector
vector<int> v2(5);                 // 5 elements, default initialized (0)
vector<int> v3(5, 10);             // 5 elements, each value 10
vector<int> v4 = {1, 2, 3, 4, 5};  // Initialized with values
```

### Adding Elements

```cpp
vector<int> v;

v.push_back(10);                   // Add to end
v.push_back(20);
v.push_back(30);

cout << v.size() << endl;          // 3
cout << v.capacity() << endl;      // Often > 3 (allocated space)
```

### Accessing Elements

```cpp
vector<int> v = {10, 20, 30, 40, 50};

cout << v[0] << endl;              // 10
cout << v.at(1) << endl;           // 20 (bounds-checked)
cout << v.front() << endl;         // 10
cout << v.back() << endl;          // 50

// Range-based for
for (int x : v) {
    cout << x << " ";
}
```

---

## Memory Management

### Size vs. Capacity

```cpp
vector<int> v;

v.push_back(1);
cout << "Size: " << v.size() << ", Capacity: " << v.capacity() << endl;
// Size: 1, Capacity: 1

v.push_back(2);
cout << "Size: " << v.size() << ", Capacity: " << v.capacity() << endl;
// Size: 2, Capacity: 2

v.push_back(3);
cout << "Size: " << v.size() << ", Capacity: " << v.capacity() << endl;
// Size: 3, Capacity: 4  ← Capacity doubled!
```

### Growth Strategy

```
Adding to vector:

Size 0, Capacity 0
  ↓ push_back
Size 1, Capacity 1    [allocate 1]
  ↓ push_back
Size 2, Capacity 2    [allocate 2, copy old]
  ↓ push_back
Size 3, Capacity 4    [allocate 4, copy old]
  ↓ push_back
Size 4, Capacity 4
  ↓ push_back
Size 5, Capacity 8    [allocate 8, copy old]
```

**Amortized O(1)**: Even though reallocation is expensive, each push_back is O(1) on average!

### Capacity Management

```cpp
vector<int> v;
v.reserve(100);                    // Pre-allocate capacity

v.push_back(1);
v.push_back(2);

cout << v.capacity() << endl;      // 100 (no reallocation)

v.shrink_to_fit();                 // Release unused capacity
```

---

## Vector Operations

### Insertion and Deletion

```cpp
vector<int> v = {1, 2, 3, 4, 5};

v.insert(v.begin() + 2, 99);       // Insert 99 at position 2
// v is now: {1, 2, 99, 3, 4, 5}

v.erase(v.begin() + 2);            // Remove element at position 2
// v is now: {1, 2, 3, 4, 5}

v.erase(v.begin() + 1, v.begin() + 3);  // Erase range
// v is now: {1, 4, 5}

v.pop_back();                      // Remove last element
v.clear();                         // Remove all elements
```

> [!CAUTION]
> **Performance**: `insert()` and `erase()` are O(n) because they shift elements. Use only when necessary!

### Iteration and Algorithms

```cpp
vector<int> v = {1, 2, 3, 4, 5};

// Index-based
for (int i = 0; i < v.size(); i++) {
    cout << v[i] << " ";
}

// Range-based (C++11+)
for (int x : v) {
    cout << x << " ";
}

// Iterator-based
for (auto it = v.begin(); it != v.end(); ++it) {
    cout << *it << " ";
}

// STL algorithms
sort(v.begin(), v.end());          // Sort
find(v.begin(), v.end(), 3);       // Find element
```

---

## Vector of Structs

```cpp
struct Student {
    int id;
    string name;
    float gpa;
};

vector<Student> students;

students.push_back({101, "Ali", 3.8});
students.push_back({102, "Sara", 3.9});
students.push_back({103, "Usman", 3.5});

for (auto &s : students) {
    cout << s.name << ": " << s.gpa << endl;
}

// Sort by GPA
sort(students.begin(), students.end(), 
     [](const Student &a, const Student &b) {
         return a.gpa > b.gpa;
     });
```

---

## 2D Vectors (Matrix)

```cpp
vector<vector<int>> matrix(3, vector<int>(4, 0));

// 3 rows, 4 columns, all initialized to 0

matrix[0][0] = 1;
matrix[1][2] = 5;
matrix[2][3] = 9;

// Jagged vector (different row sizes)
vector<vector<int>> jagged;
jagged.push_back({1, 2, 3});
jagged.push_back({4, 5});
jagged.push_back({6, 7, 8, 9});
```

---

## Complete Example: Grade Histogram

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> grades;
    
    // Read grades
    grades.push_back(85);
    grades.push_back(92);
    grades.push_back(78);
    grades.push_back(85);
    grades.push_back(95);
    
    // Display original
    cout << "Original: ";
    for (int g : grades) {
        cout << g << " ";
    }
    cout << "\n";
    
    // Sort
    sort(grades.begin(), grades.end());
    cout << "Sorted: ";
    for (int g : grades) {
        cout << g << " ";
    }
    cout << "\n";
    
    // Statistics
    cout << "Min: " << grades.front() << "\n";
    cout << "Max: " << grades.back() << "\n";
    
    double sum = 0;
    for (int g : grades) {
        sum += g;
    }
    cout << "Average: " << sum / grades.size() << "\n";
    
    return 0;
}
```

---

## Vector vs. Array vs. Linked List

| Aspect | Array | Vector | List |
|--------|-------|--------|------|
| **Access** | O(1) | O(1) | O(n) |
| **Insert** | N/A | O(n) | O(1) |
| **Delete** | N/A | O(n) | O(1) |
| **Memory** | Fixed | Dynamic | Dynamic |
| **Cache** | Excellent | Excellent | Poor |
| **Best for** | Known size | Dynamic, random access | Frequent insert/delete |

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| **Off-by-one** | Accessing v[v.size()] | Use `i < v.size()`, not `<=` |
| **Invalidated iterator** | Erase inside loop | Use `it = v.erase(it)` |
| **Empty vector access** | `v.front()` on empty | Check `if (!v.empty())` |
| **Index vs. iterator** | Confusing `[i]` and `it` | Use range-based for clarity |
| **Capacity confusion** | Expecting automatic shrinking | Use `shrink_to_fit()` explicitly |

---

## STL Algorithms with Vectors

```cpp
vector<int> v = {5, 2, 8, 1, 9, 3};

sort(v.begin(), v.end());          // {1, 2, 3, 5, 8, 9}

reverse(v.begin(), v.end());       // {9, 8, 5, 3, 2, 1}

auto it = find(v.begin(), v.end(), 5);
if (it != v.end()) {
    cout << "Found!" << endl;
}

int sum = accumulate(v.begin(), v.end(), 0);  // Sum all

int max_val = *max_element(v.begin(), v.end());  // Max
```

---

## Professional Vector Practices

### 1. Reserve Memory Upfront for Performance

```cpp
// ❌ Inefficient: multiple reallocations as vector grows
vector<int> numbers;
for (int i = 0; i < 1000000; i++) {
    numbers.push_back(i);        // May reallocate many times
}

// ✅ Efficient: allocate once
vector<int> numbers;
numbers.reserve(1000000);        // Pre-allocate without adding elements
for (int i = 0; i < 1000000; i++) {
    numbers.push_back(i);        // No reallocation needed
}
```

### 2. Move Semantics for Efficiency (C++11+)

```cpp
// ❌ Inefficient: copies heavy object
vector<string> titles;
string title = "The C++ Programming Language";
titles.push_back(title);         // Copies the string

// ✅ Efficient: moves ownership
vector<string> titles;
string title = "The C++ Programming Language";
titles.push_back(move(title));   // Moves the string (title now empty)

// ✅ Simplest: use std::move_back (pushes and moves automatically)
titles.push_back("The C++ Programming Language");  // Temporary is moved
```

### 3. Avoid Invalidated Iterators

```cpp
// ❌ WRONG: Iterator becomes invalid after push_back
vector<int> v = {1, 2, 3};
auto it = v.begin();
v.push_back(4);                  // May reallocate, invalidating it!
cout << *it;                     // ❌ UNDEFINED BEHAVIOR

// ✅ RIGHT: Recalculate iterator or use reserve
vector<int> v = {1, 2, 3};
v.reserve(10);                   // Prevent reallocation
auto it = v.begin();
v.push_back(4);                  // Safe: no reallocation
cout << *it;                     // ✅ Valid
```

### 4. Erase-Remove Idiom for Efficient Removal

```cpp
// ❌ Inefficient: O(n) for each erase
vector<int> v = {1, 2, 3, 2, 4, 2, 5};
for (auto it = v.begin(); it != v.end(); ) {
    if (*it == 2) {
        it = v.erase(it);        // Each erase is O(n)
    } else {
        ++it;
    }
}

// ✅ Efficient: single pass erase-remove
vector<int> v = {1, 2, 3, 2, 4, 2, 5};
v.erase(remove(v.begin(), v.end(), 2), v.end());  // O(n) total
```

### 5. Use `.data()` for C-style Array Interop

```cpp
// Pass vector to C-style functions
vector<int> values = {1, 2, 3, 4, 5};

// ✅ Get pointer to underlying array
legacy_c_function(values.data(), values.size());

// Equivalent to but safer than: (int *)&values[0]
```

### 6. Range-based Loops (Preferred)

```cpp
vector<Student> students;
// ... populate students ...

// ❌ Old C-style loop
for (int i = 0; i < students.size(); i++) {
    students[i].printInfo();
}

// ✅ Modern range-based loop (C++11)
for (auto& student : students) {
    student.printInfo();
}

// ✅ For read-only access
for (const auto& student : students) {
    student.printInfo();  // Can't modify
}
```

### 7. Vector of Vectors for 2D Data

```cpp
// ✅ Safe dynamic 2D array
vector<vector<int>> matrix(rows, vector<int>(cols, 0));

// Access with safety checks
for (size_t i = 0; i < matrix.size(); i++) {
    for (size_t j = 0; j < matrix[i].size(); j++) {
        cout << matrix[i][j] << " ";
    }
}

// Use .at() for bounds checking
try {
    int val = matrix.at(i).at(j);  // Throws exception if out of bounds
} catch (const out_of_range& e) {
    cerr << "Index out of range: " << e.what() << "\n";
}
```

> [!IMPORTANT]
> **Professional Vector Guidelines**:
> 1. **Use `reserve()` before loops** that add elements (performance)
> 2. **Use range-based for loops** (C++11+) instead of indices
> 3. **Use `move()` for efficiency** when passing temporaries or using `std::move`
> 4. **Avoid iterator invalidation** - understand when iterators become invalid
> 5. **Use erase-remove idiom** for efficient element removal
> 6. **Prefer `vector<T>` over arrays** - automatic memory management, no memory leaks
> 7. **Use `.at()` for bounds checking**, `[]` for performance-critical code

---

## Mastery Checklist

- [ ] Create vectors with various initialization methods
- [ ] Use `push_back()`, `pop_back()`, `clear()`
- [ ] Understand size vs. capacity and growth strategy
- [ ] Access elements with `[]`, `.at()`, `.front()`, `.back()`
- [ ] Iterate with for loops, range-based for, and iterators
- [ ] Use `insert()` and `erase()` (know they're O(n))
- [ ] Create 2D vectors and jagged vectors
- [ ] Use STL algorithms: `sort()`, `find()`, `reverse()`
- [ ] Store complex types: vectors of structs
- [ ] Know when to use vector vs. array vs. list
- [ ] Handle memory safely with RAII (automatic cleanup)
- [ ] Use `reserve()` for performance optimization
- [ ] Use move semantics for efficiency (C++11)
- [ ] Apply erase-remove idiom for deletion

> [!EXAMPLE]
> **Interview Question**: "Why is vector faster than linked list for random access?"
> 
> **Answer**: Vectors store elements **contiguously** in memory, so accessing element at index k is O(1) array lookup. Linked lists must traverse k nodes, which is O(n) and causes cache misses.
