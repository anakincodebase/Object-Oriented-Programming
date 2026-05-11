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

# 11 - Inheritance, Method Overriding, and Dynamic Arrays

> [!IMPORTANT]
> **Inheritance Principle**: Inheritance models **is-a relationships** between classes. A derived class *is-a* specialized version of its base class, inheriting its interface and data, then extending or replacing specific behaviors. This enables code reuse and polymorphic design.

---

## Inheritance Fundamentals

### The is-a Relationship

```
Real-world hierarchy:        Code hierarchy:

Vehicle                     class Vehicle {
├── Car                         // shared members
├── Motorcycle           class Car : public Vehicle {
└── Truck                    // inherits from Vehicle
                            // adds car-specific members
                        }
```

**Not** "has-a": A `Car` does **not** *have* a `Vehicle`; it **is** a `Vehicle`.

### Syntax

```cpp
class Base {
public:
    int base_member;
    void base_method() { }
};

class Derived : public Base {     // ":" specifies inheritance
public:                           // "public" is the access level (more on this later)
    int derived_member;
    void derived_method() { }
};

int main() {
    Derived d;
    d.base_member = 5;            // Inherited from Base
    d.base_method();              // Inherited from Base
    d.derived_member = 10;        // New to Derived
    d.derived_method();           // New to Derived
    return 0;
}
```

> [!NOTE]
> **Public Inheritance**: `class Derived : public Base` means the public interface of `Base` remains public in `Derived`. (There are also `private` and `protected` inheritance modes for advanced use cases.)

---

## Real-World Example: Shape Hierarchy

Let's build a geometric shape system with inheritance:

### Base Class: Point

```cpp
class Point {
public:
    int x;
    int y;
    
    void print_point() {
        cout << "(" << x << ", " << y << ")" << endl;
    }
};
```

### Base Class: Shape

```cpp
class Shape {
public:
    int num_points;              // How many points define this shape?
    Point *points;               // Dynamic array of points
    
    Shape() {
        cout << "📐 Shape constructor" << endl;
        points = NULL;           // Initialize to safe null state
        num_points = 0;
    }
    
    virtual float get_area() {   // Virtual (more on this in next topic)
        cout << "❌ Error: Cannot compute area for abstract Shape" << endl;
        return -1.0;             // Sentinel value indicating error
    }
};
```

### Derived Class: Triangle

Triangle **is-a** Shape, so it inherits Point management and area computation responsibility:

```cpp
class Triangle : public Shape {      // Inherit from Shape
public:
    Triangle() {
        cout << "🔺 Triangle constructor" << endl;
        num_points = 3;              // Inherited from Shape, but set to 3
    }
    
    void set_points(Point *p) {
        points = p;                  // Set inherited member
    }
    
    void show_shape() {
        Point *temp = points;
        for (int i = 0; i < num_points; i++) {
            temp->print_point();
            temp++;                  // Traverse Point array
        }
    }
    
    float get_area() {               // OVERRIDE the base implementation
        Point *t = points;
        int x0 = t->x, y0 = t->y;  t++;
        int x1 = t->x, y1 = t->y;  t++;
        int x2 = t->x, y2 = t->y;
        
        // Shoelace formula for triangle area
        return abs(x0 * (y1 - y2) + x1 * (y2 - y0) + x2 * (y0 - y1)) / 2.0;
    }
};
```

---

## Method Overriding

**Overriding** means a derived class provides its own implementation of a base class method.

### Method Resolution

```cpp
class Animal {
public:
    void make_sound() {
        cout << "Generic animal sound" << endl;
    }
};

class Dog : public Animal {
public:
    void make_sound() {              // Overrides base method
        cout << "Woof! Woof!" << endl;
    }
};

int main() {
    Dog dog;
    dog.make_sound();                // Calls Dog::make_sound, not Animal::make_sound
    return 0;
}
```

**Output:** `Woof! Woof!`

### Overriding vs. Overloading

| Concept | Same Class | Different Classes | Signature |
|---------|-----------|-------------------|-----------|
| **Overloading** | Multiple methods, same name | N/A | Different signatures |
| **Overriding** | N/A | Base and derived | Same signature |

```cpp
// OVERLOADING (same class, different signatures)
class Math {
    int add(int a, int b);
    double add(double a, double b);
};

// OVERRIDING (different classes, same signature)
class Animal {
    void speak();
};
class Dog : public Animal {
    void speak();                    // Overrides Animal::speak()
};
```

> [!IMPORTANT]
> **Method Overriding Rule**: To override, the derived method must have:
> - ✓ Same name
> - ✓ Same parameter list
> - ✓ Same return type (or covariant—advanced topic)

---

## Dynamic Arrays: The `new[]` and `delete[]` Operators

### Problem: Runtime-Determined Size

When you don't know the array size at compile time:

```cpp
// ❌ WRONG: Compile error
int n;
cin >> n;
int arr[n];                     // Size must be known at compile time!

// ✓ CORRECT: Dynamic allocation
int *arr = new int[n];          // Allocate n ints on the heap at runtime
// ... use arr ...
delete[] arr;                   // Free when done
```

### Syntax & Semantics

```cpp
Point *points;
int count = 3;

// Allocate an array of 3 Points on the heap
points = new Point[count];

// Access like a normal array
points[0].x = 0;
points[1].x = 10;
points[2].x = 25;

// Must use delete[] (not just delete!)
delete[] points;
points = NULL;                  // Clear dangling pointer
```

> [!CAUTION]
> **Critical Distinction**:
> - `new` allocates single object → use `delete`
> - `new[]` allocates array → use `delete[]`
> - Using `delete` on `new[]` memory → **undefined behavior, likely crash**

### Complete Triangle Example

```cpp
int main() {
    // Create triangle
    Triangle *triangle = new Triangle;
    
    // Allocate array of 3 points on heap
    Point *triangle_points = new Point[3];
    
    // Set coordinates
    triangle_points[0] = {0, 0};
    triangle_points[1] = {10, 10};
    triangle_points[2] = {25, 0};
    
    // Give points to triangle
    triangle->set_points(triangle_points);
    
    // Use triangle
    triangle->show_shape();
    cout << "Area: " << triangle->get_area() << endl;
    
    // Clean up (order doesn't matter, but be consistent)
    delete[] triangle_points;
    triangle_points = NULL;
    
    delete triangle;
    triangle = NULL;
    
    return 0;
}
```

**Output:**
```
🔺 Triangle constructor
(0, 0)
(10, 10)
(25, 0)
Area: 125
```

---

## Constructor Chaining in Inheritance

When a derived object is created, constructors run in a specific order:

```cpp
class Base {
public:
    Base() { cout << "1. Base constructor" << endl; }
};

class Derived : public Base {
public:
    Derived() { cout << "3. Derived constructor" << endl; }
};

int main() {
    Derived d;    // What order?
    return 0;
}
```

**Output:**
```
1. Base constructor
3. Derived constructor
```

> [!NOTE]
> **Constructor Call Order**: Base constructor runs first, then derived. This ensures the base part of the object is properly initialized before derived-specific code runs.

---

## Access Levels in Inheritance

The `public` keyword in `class Derived : public Base` controls what base members are visible:

```cpp
class Base {
public:
    int pub;
protected:
    int prot;
private:
    int priv;
};

class Derived : public Base {
    // pub is public in Derived ✓
    // prot is protected in Derived ✓
    // priv is NOT accessible to Derived ✗
};

int main() {
    Derived d;
    d.pub = 5;        // ✓ Public
    d.prot = 10;      // ✗ Error: protected
    d.priv = 15;      // ✗ Error: private
    return 0;
}
```

---

## Complete Geometric Shape System

```cpp
#include <iostream>
#include <string>
#include <cmath>
using namespace std;

// ========== BASE CLASSES ==========

class Point {
public:
    int x, y;
    void print() { cout << "(" << x << "," << y << ")"; }
};

class Shape {
protected:                    // Protected: accessible to derived classes
    int num_points;
    Point *points;
    
public:
    Shape() { 
        points = NULL; 
        num_points = 0; 
    }
    
    virtual float get_area() { 
        return -1.0;           // Default: cannot compute
    }
    
    virtual void print_points() {
        for (int i = 0; i < num_points; i++) {
            points[i].print();
            cout << " ";
        }
        cout << endl;
    }
};

// ========== DERIVED CLASSES ==========

class Triangle : public Shape {
public:
    Triangle() { num_points = 3; }
    
    void set_points(Point *p) {
        points = p;
    }
    
    float get_area() override {        // C++11 override keyword (optional)
        Point *t = points;
        int x0 = t->x, y0 = t->y;  t++;
        int x1 = t->x, y1 = t->y;  t++;
        int x2 = t->x, y2 = t->y;
        
        return abs(x0 * (y1 - y2) + x1 * (y2 - y0) + x2 * (y0 - y1)) / 2.0;
    }
};

class Rectangle : public Shape {
public:
    Rectangle() { num_points = 4; }
    
    void set_points(Point *p) {
        points = p;
    }
    
    float get_area() override {
        // Assuming axis-aligned rectangle
        int width = abs(points[1].x - points[0].x);
        int height = abs(points[2].y - points[1].y);
        return width * height;
    }
};

// ========== USAGE ==========

int main() {
    // Create triangle
    Triangle tri;
    Point *tri_pts = new Point[3];
    tri_pts[0] = {0, 0};
    tri_pts[1] = {10, 0};
    tri_pts[2] = {5, 8};
    tri.set_points(tri_pts);
    
    cout << "Triangle points: ";
    tri.print_points();
    cout << "Triangle area: " << tri.get_area() << " square units" << endl;
    
    delete[] tri_pts;
    tri_pts = NULL;
    
    return 0;
}
```

---

## Common Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **Using `delete` on `new[]`** | Undefined behavior | Always use `delete[]` for arrays |
| **Forgetting base destructor** | Derived resources leak | Implement destructor that calls base |
| **Object slicing** | Passing derived by value to base | Pass by reference or pointer |
| **Shadowing** | Derived method hides base | Mark with `override` keyword |
| **Missing `virtual`** | Wrong method called at runtime | Use `virtual` for polymorphic methods |

---

## Key Patterns

### Pattern: Enforcing Base Interface

```cpp
class Database {
public:
    virtual void connect() = 0;        // Must be overridden
    virtual void query(string sql) = 0;
};

class MySQLDB : public Database {
public:
    void connect() { cout << "MySQL connected" << endl; }
    void query(string sql) { cout << "MySQL executing: " << sql << endl; }
};
```

### Pattern: Protected Utilities

```cpp
class Shape {
protected:                         // Only derived classes can use
    float compute_area_helper() {
        // Complex shared logic
        return 0.0;
    }
};

class Triangle : public Shape {
public:
    float get_area() {
        return compute_area_helper();  // Can call protected method
    }
};
```

---

## Professional Inheritance Design

### 1. Access Level Hierarchy

```cpp
class Base {
private:
    int privateData;                   // Only Base can access
    
protected:
    int protectedData;                 // Base and derived can access
    
public:
    int publicData;                    // Everyone can access
    
    // Private methods: internal only
private:
    void internalHelper() { }
    
    // Protected methods: derived classes can override
protected:
    virtual void processData() {
        // Base implementation
    }
    
    // Public interface: everyone uses
public:
    virtual void execute() = 0;
};

class Derived : public Base {
public:
    void foo() {
        // privateData = 5;          // ❌ COMPILE ERROR: can't access private
        protectedData = 5;            // ✅ OK: access protected
        publicData = 5;               // ✅ OK: access public
    }
    
    void processData() override {
        // ✅ OK: override protected virtual method
    }
};
```

### 2. Virtual Destructors: CRITICAL!

Always use virtual destructors in base classes:

```cpp
// ❌ WRONG: No virtual destructor
class BadBase {
public:
    ~BadBase() { cout << "Destroying BadBase\n"; }
};

class BadDerived : public BadBase {
public:
    ~BadDerived() { cout << "Destroying BadDerived\n"; }
};

int main() {
    BadBase *ptr = new BadDerived();
    delete ptr;             // Only calls BadBase::~BadBase()!
    // BadDerived destructor NEVER runs → memory leak
}

// ✅ RIGHT: Virtual destructor
class GoodBase {
public:
    virtual ~GoodBase() { cout << "Destroying GoodBase\n"; }
};

class GoodDerived : public GoodBase {
public:
    ~GoodDerived() { cout << "Destroying GoodDerived\n"; }
};

int main() {
    GoodBase *ptr = new GoodDerived();
    delete ptr;             // Calls GoodDerived::~GoodDerived() then GoodBase::~GoodBase()
    // ✅ Both destructors run in correct order
}
```

### 3. The `override` Keyword (C++11)

Use `override` to document and verify you're overriding:

```cpp
class Base {
public:
    virtual void process(int x) { }
    virtual void cleanup() { }
};

class Derived : public Base {
public:
    // Explicitly marked as override - compiler will error if base doesn't have this
    void process(int x) override {
        // Code here
    }
    
    // ❌ COMPILE ERROR: Base has no virtual func with signature void(double)
    // void process(double x) override { }
    
    // ❌ COMPILE ERROR: Base has cleanup() but we're not overriding it
    // void cleanup(int x) override { }
};
```

### 4. Avoiding Object Slicing

```cpp
// ❌ WRONG: Pass by value slices the object
void processDerived(Base b) {
    // If 'b' is a Derived object, only Base part is passed!
    // Derived-specific data is lost
}

// ✅ RIGHT: Pass by pointer or reference
void processDerived(Base *b) {
    // Safe: passes actual object identity
}

void processDerived(Base& b) {
    // Safe: passes reference to actual object
}

int main() {
    Derived d;
    processDerived(d);              // ❌ Slicing!
    processDerived(&d);             // ✅ Correct
    processDerived(d);              // ✅ Correct (reference param)
}
```

### 5. Use `dynamic_cast` for Safe Downcasting

```cpp
class Shape { 
public: 
    virtual ~Shape() { }
};
class Circle : public Shape { };
class Triangle : public Shape { };

void process(Shape *shape) {
    // ❌ DANGEROUS: Assume it's Circle (runtime error if wrong)
    // Circle *circle = (Circle *)shape;
    
    // ✅ SAFE: Check type before cast
    Circle *circle = dynamic_cast<Circle*>(shape);
    if (circle) {
        // Safe to use circle
        cout << "It's a Circle\n";
    } else {
        cout << "Not a Circle\n";
    }
}
```

### 6. Composition vs. Inheritance

Sometimes composition is better than inheritance:

```cpp
// ❌ WRONG: Over-use of inheritance
class Employee : public Person, public Payable, public Taxable {
    // Multiple inheritance is complex
};

// ✅ RIGHT: Composition when appropriate
class Employee {
private:
    Person personalInfo;            // Composition: has-a
    Payable payrollInfo;            // Composition: has-a
    Taxable taxInfo;                // Composition: has-a
};
```

> [!IMPORTANT]
> **Professional Inheritance Rules**:
> 1. **Always use virtual destructors** in polymorphic base classes
> 2. **Mark overrides with `override` keyword** (C++11+) for clarity and safety
> 3. **Pass derived objects by pointer/reference**, never by value (prevents slicing)
> 4. **Use `protected` for shared implementation**, `private` for internals
> 5. **Prefer composition over inheritance** when possible (simpler, more flexible)
> 6. **Use `dynamic_cast` for safe downcasting** (not C-style casts)
> 7. **Consider depth: 2-3 levels is good, 10+ levels is a code smell**

---

## Practice Problems

### Problem 1: Vehicle Hierarchy

```cpp
// TODO: Create Vehicle base class with:
// - brand, model, year
// - get_info() method
// TODO: Create Car derived class with:
// - num_doors
// - override get_info() to include door count
// TODO: Create Motorcycle with:
// - has_sidecar boolean
// - override get_info()
```

### Problem 2: Dynamic Grade Array

```cpp
class Student {
private:
    string name;
    int *grades;          // Dynamic array
    int num_grades;
    
public:
    // TODO: Constructor allocates grades array
    // TODO: Destructor frees grades array
    // TODO: add_grade(int g) stores a grade
    // TODO: get_average() computes average
};
```

---

## Mastery Checklist

- [ ] Explain is-a vs. has-a relationships
- [ ] Write a derived class that inherits from a base
- [ ] Override a base method in a derived class
- [ ] Allocate and deallocate dynamic arrays with `new[]` and `delete[]`
- [ ] Access base class members from derived class
- [ ] Understand constructor call ordering in inheritance
- [ ] Avoid object slicing
- [ ] Mark derived methods with `override` keyword
- [ ] Always use virtual destructors in polymorphic bases
- [ ] Use `dynamic_cast` for safe type checking
- [ ] Choose composition over inheritance when appropriate
- [ ] Identify when methods should be virtual (preview of next topic)

> [!EXAMPLE]
> **Interview Question**: "If I have `Triangle *t = new Triangle[10];`, how do I clean up?"
>
> **Answer**: `delete[] t;` — because we allocated an array of triangles with `new[]`, we must use `delete[]`. The correct sequence is:
> ```cpp
> Triangle *t = new Triangle[10];     // Array of 10 triangles
> // ... use t ...
> delete[] t;                          // MUST use delete[], not delete
> t = NULL;                            // Clear dangling pointer
> ```
