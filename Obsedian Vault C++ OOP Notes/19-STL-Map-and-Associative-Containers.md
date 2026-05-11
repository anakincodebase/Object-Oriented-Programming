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

# 19 - STL Map and Associative Containers

> [!IMPORTANT]
> **Key-Value Storage**: `std::map<Key, Value>` stores data as **key-value pairs** with **automatic sorting by key**. Perfect for dictionaries, lookups, caching, and fast searching. Like Python dictionaries or JavaScript objects, but type-safe.

---

## Map Fundamentals

### What is a Map?

```
Python: dictionary = {"name": "Ali", "age": 25}
C++:    map<string, int> m;
        m["name"] = "Ali";          ← ✗ TYPE ERROR (name is string, stores int)
        m<"age"] = 25;
```

A map enforces **type consistency**: all keys must be one type, all values must be one type.

### Creation and Initialization

```cpp
#include <map>
using namespace std;

// Empty map
map<int, string> student_names;

// Initialize with values
map<string, int> ages = {
    {"Ali", 25},
    {"Sara", 24},
    {"Usman", 26}
};

// Copy
map<string, int> ages2 = ages;
```

---

## Core Operations

### Insert (Add Key-Value)

```cpp
map<int, string> m;

// Method 1: Direct assignment (creates if not exists)
m[1] = "Alice";
m[2] = "Bob";
m[3] = "Charlie";

// Method 2: insert() with pair
m.insert({4, "David"});
m.insert(pair<int, string>(5, "Eve"));

// Method 3: insert() ignores duplicates
m.insert({1, "ALICE"});  // Ignored! Key 1 already exists
cout << m[1] << endl;    // Still "Alice"
```

### Access (Get Value)

```cpp
map<int, string> m = {{1, "Alice"}, {2, "Bob"}};

// Method 1: Bracket notation (creates key if missing!)
cout << m[1] << endl;    // "Alice"
cout << m[99] << endl;   // "" (creates m[99] = "")

// Method 2: at() throws exception if key missing
cout << m.at(1) << endl;     // "Alice"
cout << m.at(99) << endl;    // ✗ Throws std::out_of_range

// Method 3: find()
auto it = m.find(1);
if (it != m.end()) {
    cout << it->second << endl;  // "Alice"
}
```

> [!CAUTION]
> **Bracket `[]` vs `.at()`**:
> - `m[key]` creates the key if missing (can pollute map)
> - `m.at(key)` throws exception if missing (safer)
>
> Use `.at()` for lookups, `[]` only if you intend to insert.

### Delete (Remove Key-Value)

```cpp
map<int, string> m = {{1, "Alice"}, {2, "Bob"}, {3, "Charlie"}};

// Remove by key
m.erase(2);             // Removes key 2
cout << m.size() << endl;  // 2

// Remove by iterator
auto it = m.find(1);
m.erase(it);

// Clear all
m.clear();
cout << m.size() << endl;  // 0
```

---

## Iteration and Traversal

### Automatic Sorting

```cpp
map<int, string> m;
m[3] = "Charlie";
m[1] = "Alice";
m[2] = "Bob";

// Maps automatically sort by KEY!
for (auto &p : m) {
    cout << p.first << ": " << p.second << endl;
}

// Output (sorted by key):
// 1: Alice
// 2: Bob
// 3: Charlie
```

### Iterate with Pair

```cpp
map<string, int> scores = {
    {"Alice", 95},
    {"Bob", 87},
    {"Charlie", 92}
};

// Range-based for (C++11+)
for (auto &p : scores) {
    cout << p.first << " scored " << p.second << endl;
}

// Traditional iterator
for (auto it = scores.begin(); it != scores.end(); ++it) {
    cout << it->first << ": " << it->second << endl;
}

// Reverse iteration
for (auto it = scores.rbegin(); it != scores.rend(); ++it) {
    cout << it->first << ": " << it->second << endl;
}
```

---

## Searching and Checking

### Check If Key Exists

```cpp
map<string, int> ages = {{"Ali", 25}, {"Sara", 24}};

// Method 1: count() (returns 0 or 1)
if (ages.count("Ali")) {
    cout << "Ali found\n";
}

if (ages.count("Zain") == 0) {
    cout << "Zain not found\n";
}

// Method 2: find()
if (ages.find("Ali") != ages.end()) {
    cout << "Ali found\n";
}

// Method 3: contains() (C++20)
if (ages.contains("Ali")) {
    cout << "Ali found\n";
}
```

### Find Key

```cpp
map<int, string> m = {{1, "A"}, {5, "E"}, {3, "C"}};

// Find exact key
auto it = m.find(5);
if (it != m.end()) {
    cout << it->second << endl;  // "E"
}

// Find first key >= given value (lower_bound)
auto it2 = m.lower_bound(3);
cout << it2->first << ": " << it2->second << endl;  // 3: C

// Find first key > given value (upper_bound)
auto it3 = m.upper_bound(3);
cout << it3->first << ": " << it3->second << endl;  // 5: E
```

---

## Complete Example: Student Grade System

```cpp
#include <iostream>
#include <map>
#include <string>
using namespace std;

int main() {
    // Map: student name → grade
    map<string, int> grades;
    
    // Insert grades
    grades["Ali"] = 85;
    grades["Sara"] = 92;
    grades["Usman"] = 78;
    grades.insert({"Zainab", 88});
    
    // Display all (automatically sorted by name)
    cout << "All Students:\n";
    for (auto &p : grades) {
        cout << "  " << p.first << ": " << p.second << endl;
    }
    
    // Find specific student
    cout << "\nSearch for 'Sara': ";
    if (grades.count("Sara")) {
        cout << grades["Sara"] << endl;
    } else {
        cout << "Not found\n";
    }
    
    // Find highest grade
    int max_grade = 0;
    string topper;
    for (auto &p : grades) {
        if (p.second > max_grade) {
            max_grade = p.second;
            topper = p.first;
        }
    }
    cout << "\nTopper: " << topper << " with grade " << max_grade << endl;
    
    // Remove a student
    grades.erase("Usman");
    cout << "\nAfter removing Usman:\n";
    for (auto &p : grades) {
        cout << "  " << p.first << ": " << p.second << endl;
    }
    
    // Size and empty check
    cout << "\nTotal students: " << grades.size() << endl;
    cout << "Empty? " << (grades.empty() ? "Yes" : "No") << endl;
    
    return 0;
}

// Output:
// All Students:
//   Ali: 85
//   Sara: 92
//   Usman: 78
//   Zainab: 88
//
// Search for 'Sara': 92
//
// Topper: Sara with grade 92
//
// After removing Usman:
//   Ali: 85
//   Sara: 92
//   Zainab: 88
//
// Total students: 3
// Empty? No
```

---

## Map vs. Vector: When to Use Each

| Operation | Vector | Map |
|-----------|--------|-----|
| **Access by index** | O(1) | N/A |
| **Access by value** | O(n) | N/A |
| **Access by key** | N/A | O(log n) |
| **Search** | O(n) | O(log n) |
| **Insert** | O(n) | O(log n) |
| **Delete** | O(n) | O(log n) |
| **Sorted** | No (must sort) | Yes (by key) |
| **Memory** | Compact | Overhead per pair |

```cpp
// Use VECTOR for:
vector<int> scores;  // Just a list, access by index

// Use MAP for:
map<string, int> phone_book;  // Lookup by name
map<int, User> users_by_id;   // Lookup by ID
```

---

## Professional Map Practices

### 1. Prefer `.at()` Over `[]` for Safety

```cpp
map<string, int> scores = {
    {"Alice", 95},
    {"Bob", 87}
};

// ❌ RISKY: [] creates key if not found
int score = scores["Charlie"];  // Now map has {"Charlie", 0} ← unexpected!

// ✅ SAFE: .at() throws exception if not found
try {
    int score = scores.at("Charlie");
} catch (const out_of_range& e) {
    cerr << "Student not found: " << e.what() << "\n";
}

// ✅ CHECK FIRST: Use find()
auto it = scores.find("Charlie");
if (it != scores.end()) {
    int score = it->second;  // Safe to access
} else {
    cerr << "Student not found\n";
}
```

### 2. Efficient Insertion: Use `emplace()` or `insert()`

```cpp
map<string, int> ages;

// ❌ Less efficient: creates pair first, then inserts
ages[string("Alice")] = 25;

// ✅ More efficient: constructs in-place
ages.emplace("Bob", 30);

// ✅ Also efficient: insert with initializer list
ages.insert({"Charlie", 28});

// ✅ BEST for complex values: use make_pair or initializer list
map<int, vector<string>> userGroups;
userGroups.emplace(1, vector<string>{"admin", "moderator"});
```

### 3. Iterate Safely with Range-Based For

```cpp
map<int, string> employees;
// ... populate map ...

// ❌ Less readable: iterator-based
for (auto it = employees.begin(); it != employees.end(); ++it) {
    cout << it->first << ": " << it->second << "\n";
}

// ✅ Modern C++11: range-based for
for (const auto& [id, name] : employees) {  // Structured binding
    cout << id << ": " << name << "\n";
}

// ✅ Without structured binding (if not using C++17)
for (const auto& entry : employees) {
    cout << entry.first << ": " << entry.second << "\n";
}
```

### 4. Use `contains()` (C++20) or `find()`

```cpp
map<string, int> cache;

// ❌ Pre-C++20: use find()
if (cache.find("key") != cache.end()) {
    cout << "Found\n";
}

// ✅ C++20: use contains()
if (cache.contains("key")) {
    cout << "Found\n";
}

// ✅ Fallback: use count()
if (cache.count("key")) {
    cout << "Found\n";
}
```

### 5. Efficient Batch Operations

```cpp
map<string, int> grades;

// ❌ INEFFICIENT: multiple lookups
for (const auto& student : students) {
    if (grades.find(student) != grades.end()) {
        cout << student << ": " << grades[student] << "\n";  // lookup!
    }
}

// ✅ EFFICIENT: single lookup per student
for (const auto& student : students) {
    auto it = grades.find(student);
    if (it != grades.end()) {
        cout << student << ": " << it->second << "\n";  // reuse iterator
    }
}
```

### 6. unordered_map for Performance When Order Doesn't Matter

```cpp
// ✅ Use map when you need SORTED keys
map<string, int> phone_book;  // O(log n) lookup, maintains order

// ✅ Use unordered_map when order doesn't matter and need speed
unordered_map<string, int> cache;  // O(1) average lookup, no guaranteed order

// Map: good for "all users in alphabetical order"
// unordered_map: good for "check if user exists"
```

### 7. Multimap for Duplicate Keys

```cpp
// Regular map: keys are unique
map<string, int> one_address_per_name;
one_address_per_name["Alice"] = 123;
one_address_per_name["Alice"] = 456;  // Overwrites!

// Multimap: allows duplicate keys
multimap<string, int> multiple_addresses;
multiple_addresses.insert({"Alice", 123});
multiple_addresses.insert({"Alice", 456});  // Both stored!

// Find all addresses for Alice
auto range = multiple_addresses.equal_range("Alice");
for (auto it = range.first; it != range.second; ++it) {
    cout << "Address: " << it->second << "\n";
}
```

> [!IMPORTANT]
> **Professional Map Guidelines**:
> 1. **Use `.at()` or `.find()` for safety**, avoid `[]` for lookups
> 2. **Use `emplace()` or `.insert()`** for efficient insertion
> 3. **Use range-based for loops** with structured binding (C++17+)
> 4. **Prefer `unordered_map` for O(1) lookups** when order doesn't matter
> 5. **Use `multimap` for duplicate keys**, not multiple maps
> 6. **Understand memory overhead** - maps use more memory than vectors due to tree structure
> 7. **Consider `lower_bound()`/`upper_bound()`** for range queries

---

## Advanced: Custom Comparators

### Sort Map by Value (Not Key)

By default, map sorts by key. To sort by value, use a custom comparator:

```cpp
map<string, int> grades = {
    {"Ali", 85},
    {"Sara", 92},
    {"Usman", 78}
};

// Create vector of pairs sorted by value
vector<pair<string, int>> sorted_grades(
    grades.begin(), 
    grades.end()
);

sort(sorted_grades.begin(), sorted_grades.end(),
    [](auto &a, auto &b) {
        return a.second > b.second;  // Sort by grade (descending)
    });

cout << "Sorted by grade (highest first):\n";
for (auto &p : sorted_grades) {
    cout << "  " << p.first << ": " << p.second << endl;
}
```

---

## multimap: Allowing Duplicate Keys

```cpp
#include <map>

// Regular map: each key unique
map<string, int> single;
single["Ali"] = 1;
single["Ali"] = 2;  // Overwrites, count = 1

// multimap: keys can repeat
multimap<string, int> multi;
multi.insert({"Ali", 1});
multi.insert({"Ali", 2});
multi.insert({"Ali", 3});

// Find all values for key "Ali"
auto range = multi.equal_range("Ali");
for (auto it = range.first; it != range.second; ++it) {
    cout << it->second << endl;  // 1, 2, 3
}
```

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| **Bracket creates key** | `m[missing_key]` adds to map | Use `.count()` or `.find()` first |
| **No duplicate keys** | Second insert ignored | Use `multimap` for duplicates |
| **Sorting by value** | Map only sorts by key | Copy to vector, sort manually |
| **Iterator invalidation** | Erase invalidates iterators | Use return value: `it = m.erase(it)` |
| **Key type restriction** | Key must support `<` operator | Use custom `operator<` |

---

## Mastery Checklist

- [ ] Create maps with various key/value types
- [ ] Insert and access elements with `[]` and `.insert()`
- [ ] Understand bracket `[]` creates missing keys
- [ ] Use `.at()` for safe access with exception handling
- [ ] Remove elements with `.erase()`
- [ ] Iterate with range-based for loops
- [ ] Search with `.find()`, `.count()`, `.contains()`
- [ ] Use `lower_bound()` and `upper_bound()` for range queries
- [ ] Know automatic sorting by key
- [ ] Decide between vector and map for each use case
- [ ] Use multimap for duplicate keys

> [!EXAMPLE]
> **Interview Question**: "Why is searching a map O(log n) instead of O(n)?"
>
> **Answer**: Maps are typically implemented as **red-black trees** (self-balancing BSTs). Since keys are sorted, binary search narrows down the search space by half each step, giving O(log n) complexity. Unlike linear search through a vector which is O(n).