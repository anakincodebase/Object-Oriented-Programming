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

# 10 - Constructors, Destructors, and Object Lifecycle

> [!IMPORTANT]
> **Object Lifecycle Management**: Every C++ object follows a rigid lifecycle: **creation → initialization → use → cleanup → destruction**. Constructors manage entry; destructors manage exit. Mastering this is essential for writing production-quality C++ code.

---

## The Complete Object Lifecycle

```
┌──────────────────────────────────────────────────────┐
│ 1. CONSTRUCTION                                      │
│    - Allocated (stack/heap)                          │
│    - Constructor runs immediately                    │
├──────────────────────────────────────────────────────┤
│ 2. INITIALIZATION                                    │
│    - Members initialized to meaningful state         │
│    - Invariants established                          │
├──────────────────────────────────────────────────────┤
│ 3. ACTIVE USE                                        │
│    - Methods called, state modified                  │
│    - Object exists in valid states                   │
├──────────────────────────────────────────────────────┤
│ 4. DESTRUCTION                                       │
│    - Destructor runs automatically                   │
│    - Resources released                              │
│    - Memory freed (if heap-allocated)                │ 
└──────────────────────────────────────────────────────┘
```

---

## Constructors: Initializing State

A **constructor** is a special method that runs automatically when an object is created. It has:
- No return type (not even `void`)
- Same name as the class
- One or more implementations (overloading)

### Default Constructor

A constructor with no parameters, providing "sensible defaults."

```cpp
class Employee {
private:
    string name;
    int salary;
    int id;
    
public:
    Employee() {                    // Default constructor: no parameters
        cout << "Creating Employee with defaults..." << endl;
        salary = 10000;             // Provide default values
        id = -1;                    // Sentinel value for "not assigned"
    }
};

int main() {
    Employee e;                     // Default constructor automatically called
    // e has salary = 10000, id = -1
    return 0;
}
```

> [!NOTE]
> **Default Initialization**: If you don't explicitly define a default constructor, C++ generates one automatically (but it may leave members uninitialized). It's best practice to always define one explicitly for control.

### Parameterized Constructors

Constructors with parameters allow different initialization patterns—this is **constructor overloading**.

```cpp
class Employee {
private:
    string name;
    int salary;
    int id;
    
public:
    // Constructor 1: Default
    Employee() {
        cout << "Employee() created with defaults" << endl;
        salary = 10000;
        id = 0;
    }
    
    // Constructor 2: Salary specified
    Employee(int sal) {
        cout << "Employee(salary) created with custom salary" << endl;
        salary = sal;
        id = 0;
    }
    
    // Constructor 3: Full initialization
    Employee(string n, int sal, int emp_id) {
        cout << "Employee(name, salary, id) created fully" << endl;
        name = n;
        salary = sal;
        id = emp_id;
    }
};

int main() {
    Employee e1;                         // Calls Constructor 1
    Employee e2(5000);                   // Calls Constructor 2
    Employee e3("Ali", 8000, 101);      // Calls Constructor 3
    
    return 0;
}
```

**Output:**
```
Employee() created with defaults
Employee(salary) created with custom salary
Employee(name, salary, id) created fully
```

> [!TIP]
> **Overload Resolution**: When you call `Employee(5000)`, C++ uses **function overload resolution** to determine which constructor to call based on argument types and counts.

---

## The `this` Pointer

Every non-static method has an implicit `this` pointer pointing to the current object. It's useful for:
- Distinguishing between parameters and members
- Returning reference to current object
- Explicit self-reference

```cpp
class Employee {
private:
    int salary;
    
public:
    Employee(int salary) {
        // Without 'this':
        // salary = salary;  // AMBIGUOUS! Which salary? Member or parameter?
        
        // With 'this':
        this->salary = salary;      // CLEAR: member = parameter
    }
    
    // Return reference to self (useful for chaining)
    Employee& giveSalaryIncrease(int amount) {
        this->salary += amount;
        return *this;
    }
};
```

> [!EXAMPLE]
> **Why `this` Matters**: In competitive code without `this`, the compiler might treat `salary = salary` as assigning a variable to itself (no-op). Using `this->salary = salary` removes all ambiguity—the left side is the member, the right side is the parameter.

---

## Destructors: Releasing Resources

A **destructor** runs automatically when an object is destroyed (goes out of scope or is deleted). It has:
- No return type
- Same name as class with `~` prefix
- Always takes no parameters
- Exactly one destructor per class

### Destructor Purpose

Destructors are critical for **Resource Acquisition Is Initialization (RAII)**—a key C++ idiom where constructor acquires a resource and destructor releases it.

```cpp
class Employee {
private:
    string name;
    int salary;
    
public:
    Employee(string n, int sal) {
        cout << "Constructor: Creating employee " << n << endl;
        name = n;
        salary = sal;
    }
    
    ~Employee() {
        cout << "Destructor: Cleaning up employee " << name << endl;
        // Release resources here (though this example is simple)
    }
};

int main() {
    {
        Employee e("Ali", 5000);    // Constructor runs
        cout << "Employee created" << endl;
    }                               // Destructor automatically runs here
    
    cout << "End of scope" << endl;
    return 0;
}
```

**Output:**
```
Constructor: Creating employee Ali
Employee created
Destructor: Cleaning up employee Ali
End of scope
```

### Destructor with Dynamic Memory

This is where destructors become critical:

```cpp
class EmployeeRecord {
private:
    string name;
    int *salary_history;    // Pointer to dynamically allocated array
    int history_size;
    
public:
    EmployeeRecord(string n, int size) {
        name = n;
        history_size = size;
        salary_history = new int[size];  // ACQUIRE: allocate memory
        cout << "Allocated " << size << " salary records" << endl;
    }
    
    ~EmployeeRecord() {
        delete[] salary_history;         // RELEASE: free memory
        salary_history = NULL;           // Prevent dangling pointer
        cout << "Freed salary records" << endl;
    }
};

int main() {
    {
        EmployeeRecord rec("Ali", 10);
        // ... use rec ...
    }  // Destructor runs: memory automatically freed
    
    return 0;
}
```

> [!CAUTION]
> **Critical Pattern**: 
> - Constructor uses `new` → Destructor uses `delete`
> - Constructor uses `new[]` → Destructor uses `delete[]`
> - Every allocation must have a corresponding deallocation
> - Forgetting this causes memory leaks (invisible bugs that waste system resources)

---

## Stack vs. Heap: Constructor/Destructor Timing

### Stack Allocation (Automatic)

```cpp
{
    Employee e(5000);          // Constructor runs
    cout << "Using employee..." << endl;
}  // Destructor runs automatically when scope ends
// After }, e no longer exists
```

**Timeline:**
```
1. new Employee() → Constructor
2. Execute code in scope
3. Exit scope } → Destructor runs
4. Memory returned to stack
```

### Heap Allocation (Manual Control)

```cpp
Employee *e = new Employee(5000);      // Constructor runs
cout << "Using employee..." << endl;
delete e;                               // Destructor runs explicitly
e = NULL;                               // Clear dangling pointer
```

**Timeline:**
```
1. new Employee() → Constructor
2. Execute code (e is still valid)
3. delete e → Destructor runs
4. Memory returned to heap
5. e is now a dangling pointer (points to freed memory)
6. Setting e = NULL prevents accidental use
```

---

## Complete Real-World Example

```cpp
#include <iostream>
#include <string>
using namespace std;

class Employee {
private:
    string name;
    string cnic;
    int salary;
    int id;
    
public:
    // Constructor 1: Default
    Employee() {
        cout << "📌 Default Constructor: Creating employee..." << endl;
        salary = 10000;
        id = 0;
        name = "Unknown";
    }
    
    // Constructor 2: With salary
    Employee(int sal) {
        cout << "📌 Parameterized Constructor: Creating with salary..." << endl;
        this->salary = sal;
        id = 0;
        name = "Unknown";
    }
    
    // Constructor 3: Full initialization
    Employee(string n, int sal, int emp_id) {
        cout << "📌 Full Constructor: Creating " << n << "..." << endl;
        this->name = n;
        this->salary = sal;
        this->id = emp_id;
    }
    
    ~Employee() {
        cout << "🗑️  Destructor: Cleaning up " << name << endl;
    }
    
    void sign_in() {
        cout << "✅ " << name << " signed in" << endl;
    }
};

int main() {
    cout << "=== Stack Allocation ===" << endl;
    {
        Employee e1;                          // Calls Constructor 1
        e1.sign_in();
        
        Employee e2(5000);                    // Calls Constructor 2
        
        Employee e3("Ali", 8000, 101);        // Calls Constructor 3
        e3.sign_in();
    }  // All three destructors run here in reverse order
    
    cout << "\n=== Heap Allocation ===" << endl;
    Employee *emp = new Employee("Usman", 12000, 102);
    emp->sign_in();
    delete emp;                               // Destructor runs here
    emp = NULL;
    
    cout << "End of main" << endl;
    return 0;
}
```

**Output:**
```
=== Stack Allocation ===
📌 Default Constructor: Creating employee...
✅ Unknown signed in
📌 Parameterized Constructor: Creating with salary...
📌 Full Constructor: Creating Ali...
✅ Ali signed in
🗑️  Cleaning up Ali
🗑️  Cleaning up Unknown
🗑️  Cleaning up Unknown

=== Heap Allocation ===
📌 Full Constructor: Creating Usman...
✅ Usman signed in
🗑️  Cleaning up Usman
End of main
```

---

## Function/Constructor Overloading

**Overloading** means having multiple functions with the same name but different signatures.

### Overload Rules

```cpp
class Math {
public:
    // Overload 1: int version
    int add(int a, int b) {
        return a + b;
    }
    
    // Overload 2: double version
    double add(double a, double b) {
        return a + b;
    }
    
    // Overload 3: three parameters
    int add(int a, int b, int c) {
        return a + b + c;
    }
};

int main() {
    Math m;
    cout << m.add(5, 10) << endl;           // Calls Overload 1: int
    cout << m.add(5.5, 10.5) << endl;      // Calls Overload 2: double
    cout << m.add(5, 10, 15) << endl;      // Calls Overload 3: three params
    return 0;
}
```

### Constructor Overloading (Example from Lectures)

```cpp
class Queue {
public:
    // Overload 1: Default size
    Queue() {
        size = 10;
    }
    
    // Overload 2: Custom size
    Queue(int custom_size) {
        size = custom_size;
    }
};
```

> [!NOTE]
> **Overload Resolution**: The compiler picks the "best match" based on:
> 1. Exact type match (preferred)
> 2. Implicit conversion (less preferred)
> 3. Ambiguity → Compile error

---

## Common Patterns & Pitfalls

| Pattern | Code | Problem |
|---------|------|---------|
| **Resource Leak** | `new Employee(...)` without `delete` | Memory never freed |
| **Double Delete** | `delete e; delete e;` | Undefined behavior, crash |
| **Dangling Pointer** | `delete e;` then `e->sign_in()` | Use-after-free bug |
| **Forgetting Destructor** | Dynamic member allocated in constructor, no destructor | Memory leaked |
| **Ambiguous Overload** | `void func(int)` and `void func(double)` called with no args | Compile error |

---

## Key Principles

### RAII (Resource Acquisition Is Initialization)

```cpp
// GOOD: Resources tied to object lifetime
class FileReader {
private:
    FILE *file;
public:
    FileReader(const char *filename) {
        file = fopen(filename, "r");    // Acquire in constructor
    }
    
    ~FileReader() {
        if (file) fclose(file);          // Release in destructor
    }
};

// Correct usage:
{
    FileReader reader("data.txt");       // File opened
    // Use file...
}  // File automatically closed by destructor
```

---

## Rule of Five (C++11+ Modern C++)

When a class manages resources (pointers, file handles, etc.), you must define **all five** special members:

### The Rule of Five

```cpp
class DynamicArray {
private:
    int *data;
    int size;
    
public:
    // 1. Constructor (acquire resources)
    DynamicArray(int n) {
        size = n;
        data = new int[n];
    }
    
    // 2. Destructor (release resources)
    ~DynamicArray() {
        delete[] data;
        data = nullptr;
    }
    
    // 3. Copy Constructor (deep copy for other DynamicArray)
    DynamicArray(const DynamicArray& other) {
        size = other.size;
        data = new int[size];
        for (int i = 0; i < size; i++)
            data[i] = other.data[i];
    }
    
    // 4. Copy Assignment Operator
    DynamicArray& operator=(const DynamicArray& other) {
        if (this == &other) return *this;  // Self-check
        delete[] data;                      // Free old
        size = other.size;
        data = new int[size];
        for (int i = 0; i < size; i++)
            data[i] = other.data[i];
        return *this;
    }
    
    // 5. Move Constructor (C++11: efficient transfer of ownership)
    DynamicArray(DynamicArray&& other) noexcept {
        size = other.size;
        data = other.data;              // Steal the pointer
        other.data = nullptr;           // Leave other empty
        other.size = 0;
    }
    
    // BONUS 6. Move Assignment Operator (C++11)
    DynamicArray& operator=(DynamicArray&& other) noexcept {
        if (this == &other) return *this;
        delete[] data;                  // Clean up old
        size = other.size;
        data = other.data;              // Steal
        other.data = nullptr;
        other.size = 0;
        return *this;
    }
};
```

### Modern Alternative: Rule of Zero

Use standard library containers (which manage their own resources):

```cpp
// Modern C++ (PREFERRED)
class DynamicArray {
private:
    std::vector<int> data;  // Vector manages memory automatically
public:
    DynamicArray(int n) : data(n) {}
    // No need to write constructor, destructor, copy, move!
    // Compiler generates correct versions automatically
};
```

> [!IMPORTANT]
> **Professional Practice**:
> - **If your class uses `new`**: Define all five (or six) special members
> - **If your class doesn't use `new`**: Use `= default` for compiler-generated versions
> - **Modern C++**: Use `std::vector`, `std::unique_ptr`, `std::shared_ptr` instead of raw `new`/`delete`

---

## Practice Problems

### Problem 1: Bank Account with Validation

```cpp
class BankAccount {
private:
    string account_holder;
    double balance;
    
public:
    // TODO: Implement default constructor (balance = 0)
    // TODO: Implement constructor with initial balance (validate > 0)
    // TODO: Implement destructor with cleanup message
    // TODO: Print account statement
};
```

### Problem 2: Dynamic Vector Management

```cpp
class SimpleVector {
private:
    int *data;
    int size;
    
public:
    // TODO: Constructor allocates array of size n
    // TODO: Destructor frees array
    // TODO: Copy constructor (handle deep copy)
    // TODO: Methods to access elements safely
};
```

---

## Mastery Checklist

Before moving to the next topic, ensure you can:

- [ ] Write a default constructor with meaningful initialization
- [ ] Overload a constructor with different parameter signatures
- [ ] Use `this` pointer correctly
- [ ] Implement a destructor that properly deallocates memory
- [ ] Explain the difference between stack and heap cleanup timing
- [ ] Identify and fix memory leaks in code
- [ ] Distinguish between `delete` and `delete[]`
- [ ] Prevent dangling pointer bugs

> [!EXAMPLE]
> **Interview Question**: "How would you write a constructor for a `DynamicArray` class that prevents memory leaks?"
> 
> **Answer**: 
> ```cpp
> class DynamicArray {
>     int *arr;
>     int size;
> public:
>     DynamicArray(int n) : size(n) {      // Constructor initializes size
>         arr = new int[n];                 // Acquire memory
>     }
>     
>     ~DynamicArray() {
>         delete[] arr;                     // Release memory
>         arr = NULL;                       // Prevent dangling pointer
>     }
> };
> ```
