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

# 14 - Function and Class Templates (Generic Programming)

> [!IMPORTANT]
> **Compile-Time Generics**: C++ templates enable **generic programming**—writing code once that works for many types, with type safety verified at compile time. When you use a template with a specific type, the compiler generates a specialized version for that type. This is different from runtime polymorphism and has zero runtime overhead.

---

## The Problem: Code Duplication

### Without Templates

Suppose you need a max function for different types:

```cpp
// Max function for integers
int max_int(int a, int b) {
    return (a > b) ? a : b;
}

// Max function for doubles
double max_double(double a, double b) {
    return (a > b) ? a : b;
}

// Max function for strings
string max_string(string a, string b) {
    return (a > b) ? a : b;
}

// ... and so on for each type
```

**Problem**: Same logic repeated for every type. Hard to maintain. Error-prone.

### Solution: Function Templates

```cpp
template <typename T>           // T is a type placeholder
T find_max(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    cout << find_max<int>(25, 30) << endl;              // 30
    cout << find_max<double>(3.14, 2.71) << endl;       // 3.14
    cout << find_max<string>("Ali", "Usman") << endl;   // "Usman"
    return 0;
}
```

**Output:**
```
30
3.14
Usman
```

One function, infinite types!

---

## Function Templates

### Template Syntax

```cpp
template <typename T>           // Or: template <class T>
// ↑ Declares T as a type parameter

ReturnType function_name(T param1, T param2, ...) {
    // T is substituted with actual type at compile time
}
```

### Template Instantiation

When you call `find_max<int>(5, 10)`, the compiler generates:

```cpp
// Compiler generates this for you:
int find_max(int a, int b) {
    return (a > b) ? a : b;
}
```

When you call `find_max<string>("a", "b")`, it generates:

```cpp
// Compiler generates this too:
string find_max(string a, string b) {
    return (a > b) ? a : b;
}
```

> [!NOTE]
> **Explicit Template Instantiation**: The `<int>` and `<string>` explicitly specify which type to use. The compiler needs this information to generate the correct version.

### Implicit Instantiation

Sometimes the compiler can deduce the type:

```cpp
int x = 5, y = 10;
int result = find_max(x, y);        // Compiler deduces: find_max<int>
```

But explicit instantiation is safer and clearer.

### Type Requirements

The type `T` must support the operations used in the template:

```cpp
template <typename T>
T find_max(T a, T b) {
    return (a > b) ? a : b;         // Requires operator>
}

// Works with:
int i = find_max<int>(1, 2);              // ✓ int supports >
double d = find_max<double>(1.0, 2.0);    // ✓ double supports >
string s = find_max<string>("a", "b");    // ✓ string supports >

// Fails with:
class MyClass { };
MyClass m = find_max<MyClass>(m1, m2);    // ✗ MyClass doesn't have >
```

---

## Class Templates (Generic Containers)

### Problem: Rewriting Collections for Each Type

Without templates, you'd need:

```cpp
class IntVector { ... };           // Vector of ints
class DoubleVector { ... };        // Vector of doubles
class StringVector { ... };        // Vector of strings
// ... more for each type
```

### Solution: Generic Container Template

```cpp
template <typename T>
class List {
private:
    struct node {
        T val;              // Generic value type
        node *next;
    };
    
    node *head, *last;
    
public:
    List() { head = last = NULL; }
    
    void push(T val) {
        node *temp = new node;
        temp->val = val;
        temp->next = NULL;
        
        if (last == NULL) {
            head = last = temp;
        } else {
            last->next = temp;
            last = temp;
        }
    }
    
    T pop() {
        T val = last->val;
        // ... delete node logic ...
        return val;
    }
    
    void print_list() {
        node *current = head;
        cout << "[ ";
        while (current != NULL) {
            cout << current->val << " ";
            current = current->next;
        }
        cout << "]" << endl;
    }
};
```

### Usage

```cpp
int main() {
    // Instantiation: List<int>
    List<int> int_list;
    int_list.push(5);
    int_list.push(15);
    int_list.print_list();          // Output: [ 5 15 ]
    
    int_list.pop();
    int_list.print_list();          // Output: [ 5 ]
    
    // Instantiation: List<string>
    List<string> string_list;
    string_list.push("Alice");
    string_list.push("Bob");
    string_list.print_list();       // Output: [ Alice Bob ]
    
    return 0;
}
```

**Key Insight**: Same `List` class, different types, full type safety!

---

## How Templates Work: Compile-Time Code Generation

### Template Compilation Process

```
┌──────────────────────────┐
│  Source Code             │
│  (template definition)   │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│  Compiler sees:          │
│  List<int> int_list;     │
│  List<string> str_list;  │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│  Generate two versions:  │
│  1. List<int>            │
│  2. List<string>         │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│  Compile each version    │
│  separately (type-safe)  │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│  Executable code with    │
│  both versions linked in │
└──────────────────────────┘
```

> [!TIP]
> **Key Advantage**: Type checking happens at compile time for each instantiation. If you try `List<int>` and later do `int_list.push("string")`, the compiler catches the error immediately.

---

## Template Method Definitions

### Member Functions Must Be in Header or Separated

Templates require the full definition visible at instantiation time:

```cpp
// ✗ WRONG: Won't work
// list.h
template <class T>
class List {
public:
    void push(T val);  // Declaration only
};

// list.cpp
template <class T>
void List<T>::push(T val) { ... }  // Implementation in .cpp


// ✓ CORRECT: Method in header
// list.h
template <class T>
class List {
public:
    void push(T val) {
        // ... implementation ...
    }
};
```

> [!CAUTION]
> **Why?** The compiler needs the full template definition to generate code for `List<int>`, `List<string>`, etc. If the implementation is in a separate .cpp file, the compiler can't see it during instantiation.

---

## Template Specialization

### Explicit Specialization (Advanced)

You can provide a custom implementation for a specific type:

```cpp
// General template
template <typename T>
void print_value(T val) {
    cout << "Generic: " << val << endl;
}

// Specialization for strings (custom behavior)
template <>
void print_value<string>(string val) {
    cout << "STRING: [" << val << "]" << endl;
}

int main() {
    print_value<int>(42);           // Generic: 42
    print_value<string>("hello");   // STRING: [hello]
    return 0;
}
```

---

## Practical Example: Generic Stack

```cpp
#include <iostream>
using namespace std;

template <typename T>
class Stack {
private:
    static const int MAX = 100;
    T data[MAX];
    int top_index;
    
public:
    Stack() : top_index(-1) { }
    
    void push(T value) {
        if (top_index < MAX - 1) {
            data[++top_index] = value;
        } else {
            cout << "Stack overflow!" << endl;
        }
    }
    
    T pop() {
        if (top_index >= 0) {
            return data[top_index--];
        }
        return T();  // Default value
    }
    
    bool is_empty() {
        return top_index == -1;
    }
    
    void print() {
        cout << "[ ";
        for (int i = 0; i <= top_index; i++) {
            cout << data[i] << " ";
        }
        cout << "]" << endl;
    }
};

int main() {
    // Integer stack
    Stack<int> int_stack;
    int_stack.push(1);
    int_stack.push(2);
    int_stack.push(3);
    int_stack.print();              // [ 1 2 3 ]
    
    cout << "Popped: " << int_stack.pop() << endl;  // 3
    int_stack.print();              // [ 1 2 ]
    
    // String stack
    Stack<string> str_stack;
    str_stack.push("Alice");
    str_stack.push("Bob");
    str_stack.print();              // [ Alice Bob ]
    
    return 0;
}
```

**Output:**
```
[ 1 2 3 ]
Popped: 3
[ 1 2 ]
[ Alice Bob ]
```

---

## Template Metaprogramming (Advanced Concept)

Templates are evaluated at compile time, enabling compile-time computation:

```cpp
// Factorial calculated at compile time
template <int N>
class Factorial {
public:
    static const int value = N * Factorial<N-1>::value;
};

// Specialization: base case
template <>
class Factorial<0> {
public:
    static const int value = 1;
};

int main() {
    // Calculated at compile time, not runtime!
    int result = Factorial<5>::value;  // = 120
    cout << result << endl;
    return 0;
}
```

---

## STL: The Ultimate Template Library

C++'s Standard Template Library uses extensively templates:

```cpp
#include <vector>
#include <map>

vector<int> v;              // vector<T> is a template
v.push_back(5);

map<string, int> ages;      // map<K, V> is a template
ages["Ali"] = 25;
```

You're already using templates constantly with STL!

---

## Modern C++ Template Practices (C++11 and Beyond)

### 1. Template Specialization: Customizing for Specific Types

Sometimes you need special behavior for certain types:

```cpp
// Generic template
template <typename T>
void print(T value) {
    cout << "Value: " << value << "\n";
}

// Specialization for boolean
template <>
void print<bool>(bool value) {
    cout << "Boolean: " << (value ? "TRUE" : "FALSE") << "\n";
}

// Specialization for C-string
template <>
void print<const char*>(const char* value) {
    cout << "String: \"" << value << "\"\n";
}

int main() {
    print(42);              // Uses generic: "Value: 42"
    print(true);            // Uses bool specialization: "Boolean: TRUE"
    print("Hello");         // Uses const char* specialization
}
```

### 2. Variadic Templates (C++11): Variable Number of Arguments

Handle unlimited template parameters:

```cpp
// Base case: no more arguments
void printAll() {
    // Do nothing
}

// Recursive case: first parameter + rest
template <typename First, typename... Rest>
void printAll(First first, Rest... rest) {
    cout << first << " ";
    printAll(rest...);      // Recursively handle remaining
}

int main() {
    printAll(1, 2.5, "hello", 42);  // Prints: 1 2.5 hello 42
}
```

### 3. Concepts (C++20): Type Constraints

Define requirements for template types:

```cpp
// C++20: Concept defines type requirements
template <typename T>
concept Printable = requires(T t) {
    { cout << t } -> void;  // Type must support cout
};

template <Printable T>
void safePrint(T value) {
    cout << "Safe print: " << value << "\n";
}

int main() {
    safePrint(42);              // ✅ int is Printable
    safePrint("hello");         // ✅ const char* is Printable
    // safePrint(someNonPrintable);  // ❌ Compile error: doesn't satisfy concept
}
```

### 4. Move Semantics with Templates (C++11)

Use perfect forwarding to preserve argument properties:

```cpp
template <typename T>
void process(T&& value) {
    // std::forward preserves whether value is lvalue or rvalue
    useValue(std::forward<T>(value));
}

int main() {
    int x = 10;
    process(x);             // Lvalue forwarded as lvalue
    process(20);            // Rvalue forwarded as rvalue (moved, not copied)
}
```

### 5. Non-Type Template Parameters

Templates can also take compile-time constants:

```cpp
template <typename T, int MaxSize>
class StaticArray {
private:
    T data[MaxSize];        // MaxSize is a compile-time constant
    
public:
    void fill(T value) {
        for (int i = 0; i < MaxSize; i++) {
            data[i] = value;
        }
    }
};

int main() {
    StaticArray<int, 10> arr1;   // Array of 10 ints
    StaticArray<double, 5> arr2; // Array of 5 doubles
    
    arr1.fill(42);
}
```

> [!IMPORTANT]
> **Professional Template Guidelines**:
> 1. **Define templates in headers** - compiler needs full definition for instantiation
> 2. **Document type requirements** - what operations must T support?
> 3. **Use template specialization** for type-specific optimizations
> 4. **Prefer concepts (C++20)** over SFINAE for clarity
> 5. **Use `std::forward`** for perfect forwarding in wrapper functions
> 6. **Watch out for instantiation bloat** - templates increase binary size
> 7. **Use explicit instantiation** in translation units to control binary size in large projects

---

## Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| **Syntax Error in Template** | Template not visible at instantiation | Define template in header |
| **Type Doesn't Support Operation** | Using operator not supported by type | Choose different type or add operator |
| **Implicit Instantiation Fails** | Compiler can't deduce type | Use explicit template syntax `<Type>` |
| **Large Binary Size** | Multiple instantiations of same template | Accept this; can't be avoided |

---

## Template vs. Virtual Functions: When to Use

| Feature | Compile-Time? | Runtime Cost? | Type Safe? | Use When |
|---------|---------------|---------------|-----------|----------|
| **Templates** | Yes | Zero | Full | Generic code, reusable components |
| **Virtual** | No | Small | Full | Runtime polymorphism, plugin systems |

---

## Mastery Checklist

- [ ] Write a template function with type parameter `T`
- [ ] Explicitly instantiate a template with `<Type>`
- [ ] Create a generic container class template
- [ ] Understand why template definitions must be in headers
- [ ] Define member functions of template classes
- [ ] Identify type requirements (e.g., operators needed)
- [ ] Use STL containers like `vector<T>`, `map<K, V>`
- [ ] Avoid common instantiation errors
- [ ] Know the difference between templates and virtual functions
- [ ] Implement template specialization for specific types (C++11)
- [ ] Use variadic templates for variable arguments (C++11)
- [ ] Understand perfect forwarding with `std::forward` (C++11)

> [!EXAMPLE]
> **Interview Question**: "Why do template definitions need to be in header files?"
>
> **Answer**: Because the compiler needs to generate a specific version of the template for each type it's used with. When instantiating `List<int>`, the compiler needs to see the full template definition to generate the `List<int>` code. If the implementation is in a .cpp file (which is compiled separately), the compiler can't see it during instantiation, leading to linker errors.
> 
> ```cpp
> // list.h
> template <typename T>
> class List {
>     void push(T val) { /* full implementation here */ }
> };
> ```
