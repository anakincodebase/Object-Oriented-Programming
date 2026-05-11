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

# 03 - Arrays, Strings, and Structs: Organizing Data

> [!IMPORTANT]
> **Data Organization**: Before you can design objects (classes), you need to understand how C++ organizes collections of data. Arrays provide homogeneous collections. Strings handle text. Structs group heterogeneous fields. Classes are just structs with methods and access control added.

---

## Arrays: Fixed-Size Collections

### Array Fundamentals

```cpp
int scores[10];             // Array of 10 integers
                            // Valid indices: 0, 1, 2, ..., 9

scores[0] = 85;
scores[1] = 90;
scores[9] = 78;

// Access elements
cout << scores[0] << endl;  // Output: 85

// WRONG: out-of-bounds
scores[10] = 100;           // ✗ Undefined behavior! Only 0-9 valid
```

### Memory Layout

```
scores[0]  → Address: 0x1000 → Value: 85
scores[1]  → Address: 0x1004 → Value: 90  (assuming 4-byte int)
scores[2]  → Address: 0x1008 → Value: ?
...
scores[9]  → Address: 0x1028 → Value: 78
```

**Contiguous memory**: Arrays store elements consecutively, enabling fast O(1) random access.

### Initialization

```cpp
int arr1[5];                // Uninitialized
int arr2[5] = {1, 2, 3, 4, 5};  // Initialized
int arr3[5] = {10, 20};     // {10, 20, 0, 0, 0}
int arr4[] = {1, 2, 3, 4, 5};   // Size deduced: 5
```

### Loop Over Array

```cpp
int arr[] = {10, 20, 30, 40, 50};

// Range-based for (C++11+)
for (int value : arr) {
    cout << value << " ";
}
```

---

## Strings: Text Representation

### `std::string`: Recommended

```cpp
#include <string>
using namespace std;

string name = "Ali";
string city = "Karachi";

string full = name + " from " + city;  // Concatenation
cout << name.length() << endl;         // Length: 3
name[0] = 'a';                         // Modify
```

### C-String (Avoid)

```cpp
char name[20] = "Ali";      // Older style, risky
cout << strlen(name) << endl;  // Must search for \0
```

### Why Use `std::string`

| Feature | C-String | std::string |
|---------|----------|------------|
| **Safety** | Risky (overflow) | Safe (auto resize) |
| **Length** | O(n) | O(1) |
| **Ease** | Complex | Easy |

---

## Structs: Grouping Heterogeneous Data

### Struct Definition and Usage

```cpp
struct Student {
    int id;
    string name;
    float gpa;
    int year;
};

Student s1 = {101, "Ali", 3.8, 2};
cout << s1.name << endl;            // Ali

// Arrays of structs
Student class_roster[30];
class_roster[0].id = 102;

// Pointer to struct
Student *ptr = &s1;
cout << ptr->name << endl;          // ptr->member (arrow operator)
```

### Struct with Functions

```cpp
struct Rectangle {
    double width, height;
};

double area(Rectangle r) {
    return r.width * r.height;
}

Rectangle r = {5.0, 3.0};
cout << area(r) << endl;            // 15.0
```

---

## Real-World Example: Student Record System

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

struct Student {
    int id;
    string name;
    float gpa;
};

int find_topper(vector<Student> students) {
    float max_gpa = 0.0;
    int topper_idx = -1;
    for (int i = 0; i < students.size(); i++) {
        if (students[i].gpa > max_gpa) {
            max_gpa = students[i].gpa;
            topper_idx = i;
        }
    }
    return topper_idx;
}

int main() {
    vector<Student> roster;
    roster.push_back({101, "Ali", 3.8});
    roster.push_back({102, "Usman", 3.5});
    roster.push_back({103, "Sara", 3.9});
    
    int topper = find_topper(roster);
    cout << "Topper: " << roster[topper].name << endl;
    return 0;
}
```

---

## Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| **Array out-of-bounds** | Use `i < n`, not `i <= n` |
| **Uninitialized array** | Initialize with `= {0}` |
| **C-string buffer overflow** | Use `std::string` |
| **Forgetting struct init** | Use `{}` or constructor |

---

## Mastery Checklist

- [ ] Declare and initialize arrays
- [ ] Access array elements with indexing
- [ ] Loop through arrays
- [ ] Use `std::string` instead of C-strings
- [ ] Create and use struct types
- [ ] Access struct members with `.` and `->`
- [ ] Work with arrays of structs
- [ ] Pass structs by reference efficiently
