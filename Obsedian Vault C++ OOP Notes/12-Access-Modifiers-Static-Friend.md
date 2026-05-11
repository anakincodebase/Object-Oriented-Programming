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

# 12 - Access Modifiers, Static Members, and Friend Functions

> [!IMPORTANT]
> **Encapsulation Quality**: Access modifiers (`private`, `protected`, `public`) are your primary tool for controlling how external code interacts with your classes. Correct access design prevents misuse, makes refactoring safe, and communicates intent to other developers.

---

## Access Modifiers: The Three Pillars

### Overview

| Level | Accessible From | Purpose |
|-------|-----------------|---------|
| **`public`** | Anywhere (external code) | Public interface/contract |
| **`protected`** | Same class + derived classes | Inherited members, hidden from external |
| **`private`** | Same class only | Internal implementation details |

### Visual Hierarchy

```
┌─────────────────────────────────────────────┐
│            EXTERNAL CODE                    │
│  (Can see PUBLIC only)                     │
├─────────────────────────────────────────────┤
│         DERIVED CLASSES                     │
│  (Can see PUBLIC and PROTECTED)            │
├─────────────────────────────────────────────┤
│          SAME CLASS                        │
│  (Can see PUBLIC, PROTECTED, PRIVATE)     │
└─────────────────────────────────────────────┘
```

---

## Private: The Default Access Level

**`private` members are the class's internal implementation**. External code cannot access them directly.

### The Principle: Data Hiding

```cpp
class BankAccount {
private:
    double balance;              // HIDDEN from external code
    
    void enforce_minimum_balance() {
        if (balance < 100) {
            cout << "Warning: Low balance" << endl;
        }
    }
    
public:
    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;   // Internal logic, hidden
            enforce_minimum_balance();
        }
    }
    
    double get_balance() {
        return balance;          // Read access through getter
    }
};

int main() {
    BankAccount acc;
    
    acc.deposit(500);            // ✓ Public method
    cout << acc.get_balance();   // ✓ Public getter
    
    acc.balance = -9999;         // ✗ Compile error: private
    return 0;
}
```

### Key Benefit: Invariant Preservation

The bank account **invariant**: balance should never be negative through public interface.

```cpp
// WITH private balance:
BankAccount acc;
acc.deposit(500);        // ✓ Balance = 500 (valid)

// WITHOUT private (vulnerable design):
BankAccount acc;
acc.balance = -9999;     // ✗ Breaks business logic!
```

### Getters and Setters (Accessors)

```cpp
class Employee {
private:
    int pay_rate;
    
public:
    // SETTER: Controls how pay_rate is modified
    void set_pay_rate(int rate) {
        if (rate > 14) {                    // Business rule: min $14/hr
            this->pay_rate = rate;
            cout << "Pay rate updated to $" << rate << endl;
        } else {
            cout << "Error: Pay rate too low. Minimum is $14/hr" << endl;
        }
    }
    
    // GETTER: Provides read-only access
    int get_pay_rate() {
        return pay_rate;
    }
};

int main() {
    Employee emp;
    
    emp.set_pay_rate(10);        // ✗ Rejected: less than $14
    emp.set_pay_rate(20);        // ✓ Accepted
    
    cout << emp.get_pay_rate();  // Output: 20
    return 0;
}
```

**Advantages:**
- 🔒 Validation enforced on assignment
- 🔄 Can change internal representation without changing interface
- 📋 Clear contract: what values are acceptable

---

## Protected: Inheritance-Aware Access

**`protected` members are accessible to the class and its derived classes, but not external code.**

### Use Case: Methods for Derived Classes

```cpp
class Shape {
protected:
    vector<Point> points;        // Accessible to derived classes
    
    void validate_points() {
        if (points.empty()) {
            cerr << "Error: Shape has no points" << endl;
        }
    }
    
public:
    virtual float get_area() = 0;
};

class Triangle : public Shape {
public:
    float get_area() {
        validate_points();           // ✓ Can call protected method
        // Compute area using protected 'points'
        return 0.0;
    }
};

int main() {
    Triangle t;
    // t.validate_points();         // ✗ Error: protected, not public
    t.get_area();                   // ✓ Can call public method which uses protected
    return 0;
}
```

### Protected vs. Private in Inheritance

```cpp
class Base {
private:
    int priv;                   // Derived cannot access
    
protected:
    int prot;                   // Derived CAN access
    
public:
    int pub;                    // Everyone accesses
};

class Derived : public Base {
public:
    void example() {
        pub = 1;                // ✓ Can access public
        prot = 2;               // ✓ Can access protected
        priv = 3;               // ✗ Compile error: private
    }
};
```

---

## Public: The Contract

**`public` members form the interface that external code can rely on.**

```cpp
class Shape {
public:
    virtual float get_area() = 0;
    virtual void print_info() = 0;
};

// External code depends on this interface
Shape *shape = create_shape();
shape->get_area();             // ✓ Public
shape->print_info();           // ✓ Public
```

**Design Principle**: Make public interface as minimal and stable as possible.

---

## Static Members: Class-Level Data

### Problem: Sharing Data Across Instances

Without `static`, each object has its own copy:

```cpp
class User {
public:
    int id;
};

User u1;
u1.id = 1;

User u2;
u2.id = 2;

// Each has separate 'id' member
```

With `static`, all objects share one copy:

```cpp
class User {
public:
    static int total_users;     // Shared by ALL User objects
};

int User::total_users = 0;      // Initialize outside class

User u1;
User u2;

User::total_users = 2;          // Affects ALL instances
```

### Static for Auto-Incrementing IDs

```cpp
class User {
private:
    int id;
    static int next_id;         // Shared across all instances
    
public:
    static int get_next_user_id() {
        next_id++;
        return next_id;
    }
    
    User() {
        id = User::get_next_user_id();  // Assign unique ID
    }
    
    int get_id() {
        return id;
    }
};

// CRITICAL: Initialize static member outside class
int User::next_id = 0;

int main() {
    User u1;
    cout << "User 1 ID: " << u1.get_id() << endl;    // Output: 1
    
    User u2;
    cout << "User 2 ID: " << u2.get_id() << endl;    // Output: 2
    
    User u3;
    cout << "User 3 ID: " << u3.get_id() << endl;    // Output: 3
    
    return 0;
}
```

**Output:**
```
User 1 ID: 1
User 2 ID: 2
User 3 ID: 3
```

> [!NOTE]
> **Static Member Initialization**: Static members exist outside any object instance. They **must** be initialized outside the class definition at namespace scope (usually in the .cpp file).

### Static Methods

**Static methods** can access only static data (no `this` pointer).

```cpp
class BankAccount {
private:
    static int total_accounts;
    static double total_deposits;
    double balance;
    
public:
    BankAccount() {
        balance = 0.0;
        total_accounts++;          // Increment account counter
    }
    
    void deposit(double amount) {
        balance += amount;
        total_deposits += amount;  // Update total deposits
    }
    
    // Static method: no 'this' pointer
    static void print_statistics() {
        cout << "Total accounts: " << total_accounts << endl;
        cout << "Total deposits: $" << total_deposits << endl;
        // cout << balance;  // ✗ Error: no 'this', can't access instance member
    }
};

// Initialize statics
int BankAccount::total_accounts = 0;
double BankAccount::total_deposits = 0.0;

int main() {
    BankAccount acc1, acc2, acc3;
    acc1.deposit(1000);
    acc2.deposit(2000);
    
    BankAccount::print_statistics();   // Call static method without object
    
    return 0;
}
```

**Output:**
```
Total accounts: 3
Total deposits: $3000
```

### Static Local Variables

Static variables inside functions persist across function calls:

```cpp
int count_calls() {
    static int call_count = 0;      // Initialized once
    call_count++;
    return call_count;
}

int main() {
    cout << count_calls() << endl;  // Output: 1
    cout << count_calls() << endl;  // Output: 2
    cout << count_calls() << endl;  // Output: 3
    return 0;
}
```

> [!TIP]
> **Recursive Counting**: Use static local variables to count function invocations in recursive algorithms for debugging.

---

## Friend Functions: Breaking Encapsulation (Carefully)

A **friend function** is a non-member function that gets special access to a class's private members.

> [!CAUTION]
> **Warning**: Friend functions break encapsulation. Use sparingly and only when truly necessary.

### Syntax

```cpp
class Employee {
private:
    int pay_rate;
    
public:
    friend void print_full_record(Employee emp);  // Friend declaration
    
    void set_pay_rate(int rate) {
        if (rate > 14) {
            this->pay_rate = rate;
        }
    }
};

// Friend function definition (NOT a member function)
void print_full_record(Employee emp) {
    cout << "Pay rate (private access): " << emp.pay_rate << endl;  // ✓ Can access private!
}

int main() {
    Employee e;
    e.set_pay_rate(20);
    print_full_record(e);
    
    return 0;
}
```

**Output:**
```
Pay rate (private access): 20
```

### Why Avoid Friends

```cpp
// ❌ BAD: Friend breaks encapsulation
class BankAccount {
private:
    double balance;
    friend void hack_balance(BankAccount &acc);
};

void hack_balance(BankAccount &acc) {
    acc.balance = 9999999;  // Can manipulate directly!
}

// ✓ BETTER: Use public interface
class BankAccount {
private:
    double balance;
public:
    void deposit(double amount) {
        if (amount > 0) balance += amount;
    }
};
```

### Legitimate Use Case: Operator Overloading

```cpp
class Vector {
private:
    double x, y;
    
public:
    friend ostream& operator<<(ostream &out, const Vector &v);
};

ostream& operator<<(ostream &out, const Vector &v) {
    out << "(" << v.x << ", " << v.y << ")";  // Needs access to private members
    return out;
}

int main() {
    Vector v = {3, 4};
    cout << v << endl;
    return 0;
}
```

---

## Complete Example: Multi-Layered Design

```cpp
#include <iostream>
using namespace std;

class Employee {
private:
    string name;
    int salary;
    static int employee_count;
    
public:
    // Constructor
    Employee(string n, int sal) : name(n), salary(sal) {
        employee_count++;
    }
    
    // Setter with validation
    void set_salary(int sal) {
        if (sal >= 25000) {
            salary = sal;
        } else {
            cout << "Salary too low" << endl;
        }
    }
    
    // Getter
    int get_salary() {
        return salary;
    }
    
    // Static method for class-level info
    static void print_employee_count() {
        cout << "Total employees: " << employee_count << endl;
    }
};

int Employee::employee_count = 0;

class Manager : public Employee {
private:
    int team_size;
    
public:
    Manager(string n, int sal, int team) : Employee(n, sal), team_size(team) { }
    
    void set_team_size(int size) {
        if (size > 0) {
            team_size = size;
        }
    }
};

int main() {
    Employee e1("Ali", 30000);
    Employee e2("Usman", 35000);
    Manager m1("Sara", 50000, 5);
    
    e1.set_salary(28000);
    e2.set_salary(8000);      // Rejected: too low
    
    Employee::print_employee_count();  // Output: Total employees: 3
    
    return 0;
}
```

---

## Access Modifier Decision Matrix

| Situation | Choice | Reason |
|-----------|--------|--------|
| User-facing method | `public` | External code needs it |
| Internal helper | `private` | Hide implementation |
| To-be-inherited method | `protected` | Derived classes need it |
| Class-level counter | `static` | Shared across all instances |
| Special access needed | `friend` | Rare exceptions (operators, etc.) |

---

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| All public members | No encapsulation | Make most fields `private` |
| Static initialization | Linker error | Initialize outside class |
| Friend overuse | Breaks encapsulation | Use getters/setters instead |
| Forgetting `public:` | Everything private | Explicitly mark public |
| Not validating in setters | Invalid states possible | Add range checks |

---

## Mastery Checklist

- [ ] Distinguish `public`, `protected`, `private`
- [ ] Write getters/setters that enforce invariants
- [ ] Use `static` to share data across instances
- [ ] Call static methods without objects
- [ ] Initialize static members outside class
- [ ] Recognize when `friend` is appropriate (rarely)
- [ ] Design classes with minimal public interface
- [ ] Prevent unauthorized object state changes

> [!EXAMPLE]
> **Interview Question**: "Why make data `private` when getters/setters do the same thing?"
>
> **Answer**: Because it gives you *flexibility*. Example:
> ```cpp
> // Private with getter allows refactoring:
> class User {
>     string full_name;  // Can change internal representation
> public:
>     string get_name() { return full_name; }
> };
> 
> // Later, you could refactor to:
> class User {
>     string first_name, last_name;  // Implementation changes
> public:
>     string get_name() {
>         return first_name + " " + last_name;  // Same interface!
>     }
> };
> 
> // External code doesn't break because it used the getter
> ```
