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

# 09 - Introduction to Classes and Objects

> [!IMPORTANT]
> **Conceptual Foundation**: Classes are blueprints that unite data (state) with operations (behavior). They represent the cornerstone of Object-Oriented Programming—enabling code reusability, modularity, and logical abstraction of real-world entities.

---

## The Class Blueprint Pattern

A class definition specifies the structure and interface of objects that will be created from it. Think of it as an architectural blueprint—the blueprint itself isn't a building, but rather the specification for creating buildings that follow that design.

### Core Components

| Component | Purpose | Access |
|-----------|---------|--------|
| **Data Members** | Object state/attributes | Private (hidden) |
| **Methods** | Object behavior/operations | Public (exposed) |
| **Interface** | Contract with external code | Public only |
| **Implementation** | Internal logic | Can be private or public |

---

## Fundamental Example: Employee Class

### Class Definition

```cpp
#include <iostream>
#include <string>
using namespace std;

class Employee {
public:
    string name;
    string cnic;
    int id;
    
    void sign_in();
    void sign_out();
};
```

### Method Implementation (Scope Resolution)

```cpp
void Employee::sign_in() {
    cout << "Signing in the employee: " << name << endl;
}

void Employee::sign_out() {
    cout << "Signing out: " << name << endl;
}
```

> [!NOTE]
> **The `::` Operator**: This is the *scope resolution operator*. It tells the compiler that `sign_in()` and `sign_out()` belong to the `Employee` class namespace, allowing you to implement methods outside the class declaration.

---

## Object Instantiation: Stack vs. Heap

Objects can be created in two memory regions, each with distinct implications:

### Stack Allocation (Automatic)

```cpp
int main() {
    Employee e1;              // Created on stack
    e1.name = "Ali";
    cout << e1.name << endl;  // Direct member access
    e1.sign_in();
    
    return 0;
}  // e1 automatically destroyed when scope ends
```

**Characteristics:**
- 🔵 **Automatic lifecycle**: Destroyed when out of scope
- 🔵 **Automatic cleanup**: No manual deletion needed  
- 🔵 **Performance**: Stack is faster; suitable for small objects
- 🔴 **Limited lifetime**: Cannot persist beyond function scope

### Heap Allocation (Dynamic)

```cpp
int main() {
    Employee *e = new Employee();  // Created on heap
    e->name = "Usman";             // Pointer-to-member access
    e->sign_in();
    
    delete e;                       // Manual cleanup required
    e = NULL;                       // Prevent dangling pointer
    
    return 0;
}
```

**Characteristics:**
- 🟢 **Manual lifecycle**: You control creation and destruction
- 🟡 **Manual cleanup**: Must call `delete` to avoid memory leaks
- 🟡 **Slower access**: Heap allocation has indirection overhead
- 🟢 **Extended lifetime**: Persists until explicitly deleted
- 🟢 **Flexibility**: Size/quantity determined at runtime

> [!CAUTION]
> **Memory Management Discipline**: Every `new` must have a corresponding `delete`. Forgetting to delete leads to **memory leaks**—resources consumed but never returned to the system. This is a critical source of bugs in production C++ systems.

---

## Dot vs. Arrow Operators

The operator you use depends on whether you're accessing an object directly or through a pointer:

```cpp
Employee e;           // Stack object
e.name = "Ali";       // Dot operator: direct access
e.sign_in();

Employee *ptr = &e;   // Pointer to object
ptr->name = "Ali";    // Arrow operator: dereference pointer
ptr->sign_in();

// Equivalent to: (*ptr).name = "Ali";
```

> [!TIP]
> **Memory Model Clarity**: The arrow operator (`->`) is syntactic sugar. `ptr->member` is equivalent to `(*ptr).member`. Use arrow when working with pointers for clarity.

---

## Encapsulation and Data Hiding

The foundational principle of OOP is **encapsulation**: bundling data with the operations that manipulate it, while hiding internal details.

### The Principle

```
┌─────────────────────────────────┐
│   Public Interface (Contract)   │
│  ─ What external code sees      │
├─────────────────────────────────┤
│ ┌───────────────────────────────┤ 
│ │  Private Implementation       │
│ │ - Internal details hidden     │
│ │ - Can change without affecting│
│ │   external code               │
│ └───────────────────────────────┤
└─────────────────────────────────┘
```

### Implementation Example

```cpp
class BankAccount {
private:
    double balance;      // Hidden: external code cannot access
    
public:
    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;  // Only modify through controlled interface
        }
    }
    
    double getBalance() const {
        return balance;
    }
};
```

**Advantages:**
- 🔐 **Invariant Preservation**: `balance` can never become negative through public interface
- 🔄 **Implementation Flexibility**: Internal logic can change without external impact
- 📋 **Contract Clarity**: Clear what operations are safe vs. unsafe

---

## Why This Matters in Professional Software

| Scenario | Problem Without Encapsulation | Solution With Encapsulation |
|----------|-------------------------------|---------------------------|
| **Business Logic Change** | `balance = -9999;` from anywhere | Only `deposit()` and `withdraw()` can modify |
| **Refactoring Internals** | Code breaks when implementation changes | Private details can be completely rewritten |
| **Debugging** | Money disappears—where? Unknown location | Breaks only happen inside `deposit()` or `withdraw()` |
| **API Stability** | Public fields become locked contract | Can add new private fields without breaking anything |

---

## Key Concepts to Master

### 1. **Member Access**
- **Public**: Available to all code (`object.member`, `ptr->member`)
- **Private**: Accessible only within the class

### 2. **Scope Resolution**
- The `::` operator associates method implementations with their class
- Enables separation of declaration and implementation

### 3. **Memory Ownership**
- Stack allocation: scope-based cleanup
- Heap allocation: manual cleanup responsibility

### 4. **Invariant Guarantees**
- Well-designed classes ensure objects are always in valid states
- Achieved through controlled access to data

---

## Common Pitfalls & Solutions

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **Public data members** | Can be modified arbitrarily from anywhere | Keep data private; use getters/setters |
| **Forgetting `delete`** | Memory leaks accumulate | Match every `new` with `delete` |
| **Dangling pointers** | Using deleted memory causes crashes | Set pointer to `NULL` after `delete` |
| **Confusing `.` and `->`** | Compilation errors or segfaults | Use `.` for objects, `->` for pointers |

---

## Practical Exercise

### Problem
Design a `Student` class with:
- Private data: `name`, `id`, `gpa`
- Public methods: `enroll()`, `printTranscript()`
- Ensure GPA cannot exceed 4.0 or go below 0.0

### Solution Template

```cpp
class Student {
private:
    string name;
    int id;
    double gpa;
    
    void validateGPA(double g) {
        if (g < 0.0 || g > 4.0) {
            cout << "Invalid GPA" << endl;
            gpa = 0.0;
        } else {
            gpa = g;
        }
    }
    
public:
    void enroll(string n, int i, double g) {
        name = n;
        id = i;
        validateGPA(g);
    }
    
    void printTranscript() {
        cout << "Name: " << name << endl;
        cout << "ID: " << id << endl;
        cout << "GPA: " << gpa << endl;
    }
};
```

### What You're Learning
- ✓ Data hiding protects invariants
- ✓ Private validation methods enforce business rules
- ✓ Public interface is simple and safe

---

## Professional C++ Class Design

### 1. Getter/Setter Pattern with Validation

Professional classes use **getter/setter methods** to control read/write access and enforce invariants:

```cpp
class BankAccount {
private:
    double balance;
    string accountNumber;
    
public:
    // Constructor
    BankAccount(string accNum) : accountNumber(accNum), balance(0.0) { }
    
    // Getter: read-only access
    double getBalance() const {
        return balance;
    }
    
    // Setter with validation
    bool deposit(double amount) {
        if (amount <= 0) {
            cerr << "Error: Deposit must be positive\n";
            return false;
        }
        balance += amount;
        cout << "✅ Deposited $" << amount << "\n";
        return true;
    }
    
    // Setter with constraints
    bool withdraw(double amount) {
        if (amount > balance) {
            cerr << "Error: Insufficient funds\n";
            return false;
        }
        balance -= amount;
        cout << "✅ Withdrew $" << amount << "\n";
        return true;
    }
};

int main() {
    BankAccount account("123456");
    
    account.deposit(1000);      // ✅ Safe: validation happens
    account.withdraw(500);      // ✅ Safe: balance checked
    
    cout << "Balance: $" << account.getBalance() << "\n";
    // account.balance = -100;  // ❌ COMPILE ERROR: private
}
```

### 2. Const-Correctness: Promise Not to Modify

Mark methods that don't modify state as `const`:

```cpp
class Temperature {
private:
    double celsius;
    
public:
    Temperature(double c) : celsius(c) { }
    
    // Const method: promises not to modify member variables
    double fahrenheit() const {
        return celsius * 9.0 / 5.0 + 32.0;
    }
    
    // Const method: read-only access
    double getCelsius() const {
        return celsius;
    }
    
    // Non-const method: CAN modify
    void setCelsius(double c) {
        celsius = c;
    }
};

// Benefits:
// 1. Compiler prevents accidental modifications
// 2. Communicates intent to developers
// 3. Allows passing to const references
void displayTemperature(const Temperature& temp) {
    cout << temp.fahrenheit() << "°F\n";  // ✅ Calls const method
    // temp.setCelsius(0);  // ❌ COMPILE ERROR: attempts to call non-const method
}
```

### 3. Encapsulation Best Practices

| Practice | Good | Bad | Why |
|----------|------|-----|-----|
| **Data hiding** | `private int id;` | `public int id;` | Protects invariants |
| **Getter/setter** | `int getId() const { return id; }` | Direct access | Allows future validation |
| **Const-correctness** | `void print() const` | `void print()` | Communicates safety |
| **Defensive copies** | Return `const&` or copy | Return pointer | Prevents external modification |

### 4. Inline Methods for Performance

Small methods can be inlined to eliminate function call overhead:

```cpp
class Point {
private:
    int x, y;
    
public:
    // Inline: compiler substitutes code directly
    int getX() const { return x; }
    int getY() const { return y; }
    
    void setCoordinates(int newX, int newY) {
        x = newX;
        y = newY;
    }
};
```

> [!IMPORTANT]
> **Professional Principles**:
> 1. **Keep data members private** - always
> 2. **Provide public getter/setter methods** - with validation
> 3. **Mark read-only methods with `const`** - for safety and optimization
> 4. **Validate input in setters** - enforce business rules
> 5. **Use meaningful names** - `getBalance()` not `getX()`

---

## Next Steps

Before proceeding to the next topic, make sure you can:
- [ ] Explain the difference between stack and heap allocation
- [ ] Use `.` operator with objects and `->` with pointers fluently
- [ ] Design a class where data members are private by default
- [ ] Implement a method outside the class using `::`
- [ ] Identify when to make a field private vs. public
- [ ] Write getter/setter methods with validation
- [ ] Mark read-only methods with `const`
- [ ] Use defensive copying to prevent external modification

> [!EXAMPLE]
> **Real-world analogy**: Think of a car manufacturer. The internal engine mechanism (private) can change completely—from V6 to V8, carburator to fuel injection. The interface (public)—gas pedal, steering wheel—remains the same. Drivers don't need to know engine details; they use the consistent public interface.
