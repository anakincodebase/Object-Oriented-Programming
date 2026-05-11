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

# 13 - Virtual Functions and Polymorphism

> [!IMPORTANT]
> **The Polymorphism Paradigm**: Virtual functions enable **runtime polymorphism**—the ability to write code that works with base class pointers/references but dynamically dispatches to the correct derived class method at runtime. This is arguably the most powerful feature of OOP, enabling extensible architectures that don't require modification when you add new derived classes.

---

## The Core Problem: Method Resolution

### Compile-Time Binding (Without `virtual`)

By default, C++ resolves method calls at compile time based on the declared type:

```cpp
class Animal {
public:
    void speak() {
        cout << "Some generic animal sound" << endl;
    }
};

class Dog : public Animal {
public:
    void speak() {
        cout << "Woof! Woof!" << endl;
    }
};

int main() {
    Animal *ptr = new Dog();    // Pointer declared as Animal*
    ptr->speak();               // What happens?
}
```

**Output:** `Some generic animal sound`

**Why?** Without `virtual`, the compiler sees `ptr` is type `Animal*` and calls `Animal::speak()` at compile time, ignoring that the actual object is a `Dog`.

### Runtime Binding (With `virtual`)

The `virtual` keyword tells the compiler: "Resolve this method call at runtime based on the actual object type, not the pointer type."

```cpp
class Animal {
public:
    virtual void speak() {      // <- Add virtual keyword
        cout << "Some generic animal sound" << endl;
    }
};

class Dog : public Animal {
public:
    void speak() {              // Override virtual method
        cout << "Woof! Woof!" << endl;
    }
};

int main() {
    Animal *ptr = new Dog();    // Pointer type: Animal*
    ptr->speak();               // Actual object: Dog
}
```

**Output:** `Woof! Woof!`

Now the call is resolved at runtime: "Is the actual object a `Dog`? Then call `Dog::speak()`."

---

## Virtual Functions: Method Binding at Runtime

### Syntax and Semantics

```cpp
class Shape {
public:
    // Virtual method: can be overridden by derived classes
    virtual float get_area() {
        return 0.0;
    }
    
    // Virtual destructor: CRITICAL for polymorphic base classes
    virtual ~Shape() { }
};

class Circle : public Shape {
public:
    float radius;
    
    // Override the virtual method
    float get_area() {
        return 3.14159 * radius * radius;
    }
};

class Rectangle : public Shape {
public:
    float width, height;
    
    // Override the virtual method
    float get_area() {
        return width * height;
    }
};
```

### How Virtual Functions Work

When you call a virtual method through a base pointer/reference:

```cpp
1. Shape *s = new Circle();     // Pointer type: Shape*
                                // Actual type: Circle

2. s->get_area();               // RUNTIME dispatch:
                                // 1. Look at actual object type
                                // 2. Find Circle::get_area()
                                // 3. Call it
```

---

## Pure Virtual Functions and Abstract Classes

A **pure virtual function** is one with no implementation—it declares that derived classes **must** override it.

### Syntax

```cpp
class Shape {
public:
    // Pure virtual: no implementation, just = 0
    virtual float get_area() = 0;
    
    // Can still have other virtual methods with implementations
    virtual void print_info() {
        cout << "This is a shape" << endl;
    }
};
```

### Abstract Classes

A class with **any** pure virtual function becomes **abstract**—you cannot instantiate it directly.

```cpp
class Shape {
public:
    virtual float get_area() = 0;  // Pure virtual
};

int main() {
    // Shape s;                   // ✗ Compile error: cannot instantiate abstract class
    
    Shape *s;                      // ✓ OK: pointer to abstract class
    
    return 0;
}
```

> [!NOTE]
> **Abstract Base Classes**: Pure virtual functions force derived classes to implement them, ensuring consistent interfaces. They're a contract that says "any Shape **must** provide get_area()".

### Concrete Classes (Implementations)

Derived classes that implement all pure virtual functions become concrete (instantiable):

```cpp
class Circle : public Shape {
private:
    float radius;
    
public:
    // Must implement the pure virtual function
    float get_area() {
        return 3.14159 * radius * radius;
    }
};

int main() {
    Circle c;                      // ✓ OK: Circle is concrete
    c.get_area();
    
    return 0;
}
```

---

## Plugin Architecture: The Real Power

Virtual functions enable **plugin systems**—extensible architectures where new types can be added without modifying existing code.

### Example: Image Filter System

```cpp
// ===== BASE CLASS: PLUGIN INTERFACE =====
class Plugin {
public:
    // Pure virtual: every plugin must implement apply_filter
    virtual void apply_filter(int img[5][5]) = 0;
    virtual ~Plugin() { }
};

// ===== CONCRETE PLUGINS =====

class MakeItWhite : public Plugin {
public:
    void apply_filter(int img[5][5]) {
        for (int i = 0; i < 5; i++) {
            for (int j = 0; j < 5; j++) {
                img[i][j] = 255;  // White
            }
        }
    }
};

class MakeItBlack : public Plugin {
public:
    void apply_filter(int img[5][5]) {
        for (int i = 0; i < 5; i++) {
            for (int j = 0; j < 5; j++) {
                img[i][j] = 0;    // Black
            }
        }
    }
};

// ===== FACTORY: SELECT PLUGIN BY NAME =====

Plugin* select_plugin(string plugin_name) {
    if (plugin_name == "white")
        return new MakeItWhite();
    else if (plugin_name == "black")
        return new MakeItBlack();
    else
        return NULL;
}

// ===== USAGE: CODE WORKS WITH ANY PLUGIN =====

int main() {
    int image[5][5] = {...};
    
    // Get a plugin dynamically (could be any Plugin subclass)
    Plugin *filter = select_plugin("black");
    
    // Call apply_filter: compiler doesn't know which plugin
    // But at runtime, it correctly calls MakeItBlack::apply_filter!
    filter->apply_filter(image);
    
    delete filter;
    return 0;
}
```

> [!EXAMPLE]
> **Why This Matters**: You can add a new plugin (e.g., `MakeItGray`) without changing the selection logic or any existing code. The system is **open for extension, closed for modification**.

---

## Polymorphic Collections

One of the most practical uses of virtual functions:

```cpp
class Shape {
public:
    virtual float get_area() = 0;
    virtual ~Shape() { }
};

class Circle : public Shape {
    float radius = 5.0;
public:
    float get_area() { return 3.14159 * radius * radius; }
};

class Rectangle : public Shape {
    float width = 4.0, height = 6.0;
public:
    float get_area() { return width * height; }
};

int main() {
    // Vector of pointers to base class
    vector<Shape*> shapes;
    
    // Add different types
    shapes.push_back(new Circle());
    shapes.push_back(new Rectangle());
    shapes.push_back(new Circle());
    
    // Single loop: correct method called for each type
    float total = 0;
    for (Shape *s : shapes) {
        total += s->get_area();      // Polymorphic call!
    }
    
    cout << "Total area: " << total << endl;
    
    // Clean up
    for (Shape *s : shapes) {
        delete s;
    }
    
    return 0;
}
```

**Output:**
```
Total area: 115.265
```

> [!NOTE]
> Notice: We never explicitly check "Is this a Circle or Rectangle?". The virtual function mechanism handles it automatically.

---

## Virtual Destructors: Critical!

**Every polymorphic base class must have a virtual destructor.**

### Problem Without Virtual Destructor

```cpp
class Shape {
public:
    virtual float get_area() = 0;
    ~Shape() { cout << "Destroying Shape" << endl; }  // NOT virtual!
};

class Circle : public Shape {
public:
    float get_area() { return 0; }
    ~Circle() { cout << "Destroying Circle" << endl; }
};

int main() {
    Shape *s = new Circle();
    delete s;                  // What happens?
}
```

**Output:** `Destroying Shape`

**Problem**: Only the base destructor runs! If `Circle` allocated resources, they leak.

### Solution: Virtual Destructor

```cpp
class Shape {
public:
    virtual float get_area() = 0;
    virtual ~Shape() { cout << "Destroying Shape" << endl; }  // Virtual!
};

class Circle : public Shape {
public:
    float get_area() { return 0; }
    ~Circle() { cout << "Destroying Circle" << endl; }
};

int main() {
    Shape *s = new Circle();
    delete s;                  // Now both run
}
```

**Output:**
```
Destroying Circle
Destroying Shape
```

> [!CAUTION]
> **Critical Rule**: If a class has any virtual methods (meaning it's meant for polymorphism), its destructor must be virtual. Otherwise, you create resource leaks.

---

## Complete Example: Image Processing System

```cpp
#include <iostream>
#include <vector>
using namespace std;

// ===== BASE CLASS =====
class Filter {
public:
    virtual void apply(int img[5][5]) = 0;
    virtual string get_name() = 0;
    virtual ~Filter() { }
};

// ===== CONCRETE FILTERS =====

class Brighten : public Filter {
public:
    void apply(int img[5][5]) {
        for (int i = 0; i < 5; i++) {
            for (int j = 0; j < 5; j++) {
                img[i][j] = min(255, img[i][j] + 50);
            }
        }
    }
    string get_name() { return "Brighten"; }
};

class Invert : public Filter {
public:
    void apply(int img[5][5]) {
        for (int i = 0; i < 5; i++) {
            for (int j = 0; j < 5; j++) {
                img[i][j] = 255 - img[i][j];
            }
        }
    }
    string get_name() { return "Invert"; }
};

// ===== FILTER PIPELINE =====

class Image {
private:
    int data[5][5];
    
public:
    void apply_filters(vector<Filter*> filters) {
        for (Filter *f : filters) {
            cout << "Applying " << f->get_name() << "..." << endl;
            f->apply(data);           // Polymorphic call
        }
    }
    
    void print() {
        for (int i = 0; i < 5; i++) {
            for (int j = 0; j < 5; j++) {
                cout << data[i][j] << " ";
            }
            cout << endl;
        }
    }
};

// ===== USAGE =====

int main() {
    Image img;
    
    vector<Filter*> pipeline;
    pipeline.push_back(new Brighten());
    pipeline.push_back(new Invert());
    pipeline.push_back(new Brighten());
    
    img.apply_filters(pipeline);
    
    // Clean up
    for (Filter *f : pipeline) {
        delete f;
    }
    
    return 0;
}
```

---

## Virtual vs. Non-Virtual: Decision Guide

| Situation | Use Virtual | Reason |
|-----------|------------|--------|
| Base method will be overridden | ✓ Virtual | Enable runtime polymorphism |
| Utility helper method | ✗ Non-virtual | No need for polymorphic dispatch |
| Interface contract | ✓ Pure virtual | Force derived classes to implement |
| Base class won't be inherited | ✗ Non-virtual | No polymorphism needed |
| Destructor in polymorphic base | ✓ Virtual | Prevent resource leaks |

---

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| **Non-virtual destructor** | Resource leaks on deletion | Make destructor virtual |
| **Forgetting derived override** | Wrong method called at runtime | Override all pure virtual methods |
| **Virtual from constructor** | Derived method not called | Don't rely on virtual calls in constructor |
| **Slicing** | Passing derived by value | Pass by pointer/reference |
| **Wrong signature** | Method not overridden | Match base signature exactly |

---

## Professional Design Patterns with Polymorphism

### 1. Strategy Pattern: Plug-and-Play Algorithms

Use virtual functions to allow runtime algorithm selection:

```cpp
// Base: Algorithm interface
class PaymentStrategy {
public:
    virtual bool process(double amount) = 0;
    virtual ~PaymentStrategy() { }
};

// Concrete strategies
class CreditCardPayment : public PaymentStrategy {
public:
    bool process(double amount) {
        cout << "Processing $" << amount << " via Credit Card\n";
        return true;
    }
};

class CryptoCurrencyPayment : public PaymentStrategy {
public:
    bool process(double amount) {
        cout << "Processing $" << amount << " via Crypto\n";
        return true;
    }
};

// Client: Doesn't care which strategy is used
class ShoppingCart {
private:
    PaymentStrategy *paymentMethod;
public:
    void setPaymentMethod(PaymentStrategy *method) {
        paymentMethod = method;
    }
    
    void checkout(double total) {
        if (paymentMethod->process(total)) {
            cout << "✅ Payment successful\n";
        }
    }
};

// Runtime selection
int main() {
    ShoppingCart cart;
    
    CreditCardPayment cardPayment;
    cart.setPaymentMethod(&cardPayment);
    cart.checkout(99.99);  // Uses credit card
    
    CryptoCurrencyPayment cryptoPayment;
    cart.setPaymentMethod(&cryptoPayment);
    cart.checkout(99.99);  // Uses crypto
}
```

### 2. Factory Pattern: Dynamic Object Creation

Let polymorphism handle which type to create:

```cpp
class Vehicle {
public:
    virtual void drive() = 0;
    virtual ~Vehicle() { }
};

class Car : public Vehicle {
public:
    void drive() { cout << "🚗 Driving car on road\n"; }
};

class Boat : public Vehicle {
public:
    void drive() { cout << "🚤 Driving boat on water\n"; }
};

// Factory: Creates appropriate type
Vehicle* createVehicle(string type) {
    if (type == "car") return new Car();
    if (type == "boat") return new Boat();
    return nullptr;
}

int main() {
    // Client doesn't know exact type at compile time
    vector<Vehicle*> fleet;
    fleet.push_back(createVehicle("car"));
    fleet.push_back(createVehicle("boat"));
    
    for (Vehicle *v : fleet) {
        v->drive();  // Polymorphism: correct method called
    }
    
    // Cleanup
    for (auto v : fleet) delete v;
}
```

### 3. Template Method Pattern: Skeleton with Customizable Steps

Base class defines algorithm structure; derived classes customize specific steps:

```cpp
// Base: Algorithm template
class DataProcessor {
public:
    // Template method: defines algorithm flow
    void process() {
        loadData();
        transformData();    // Let derived class customize
        validateData();
        saveData();         // Let derived class customize
    }
    
private:
    void loadData() { cout << "Loading...\n"; }
    void validateData() { cout << "Validating...\n"; }
    
    // Virtual hooks: customization points
    virtual void transformData() = 0;
    virtual void saveData() = 0;
    virtual ~DataProcessor() { }
};

// Concrete implementation
class CSVProcessor : public DataProcessor {
private:
    void transformData() { cout << "Parsing CSV rows...\n"; }
    void saveData() { cout << "Writing to CSV file...\n"; }
};

class JSONProcessor : public DataProcessor {
private:
    void transformData() { cout << "Parsing JSON objects...\n"; }
    void saveData() { cout << "Writing to JSON file...\n"; }
};

int main() {
    CSVProcessor csv;
    csv.process();  // Runs template method
}
```

> [!IMPORTANT]
> **Professional Guidelines**:
> 1. **Design for extension, not modification** (Open/Closed Principle)
> 2. **Always use virtual destructors** in polymorphic classes
> 3. **Prefer composition over inheritance** when possible (Dependency Injection)
> 4. **Use smart pointers** (`unique_ptr`, `shared_ptr`) instead of raw `new`/`delete`
> 5. **Document virtual contracts** - what derived classes must implement

---

## Mastery Checklist

- [ ] Explain compile-time vs. runtime method resolution
- [ ] Identify when to mark a method `virtual`
- [ ] Create a pure virtual function and abstract class
- [ ] Implement a concrete class overriding pure virtual methods
- [ ] Write a virtual destructor
- [ ] Build a polymorphic collection (vector of base pointers)
- [ ] Design a plugin system architecture
- [ ] Avoid object slicing with derived classes
- [ ] Understand dynamic_cast for type checking (advanced)
- [ ] Implement Strategy Pattern (runtime algorithm selection)
- [ ] Implement Factory Pattern (type-safe creation)
- [ ] Implement Template Method Pattern (customizable algorithm)

> [!EXAMPLE]
> **Interview Question**: "What's the difference between a virtual function and a pure virtual function?"
>
> **Answer**: 
> - **Virtual function**: Has an implementation that derived classes **can** override (optional)
>   ```cpp
>   virtual void print() { cout << "Base version"; }  // Has implementation
>   ```
> - **Pure virtual function**: Has no implementation; derived classes **must** override it (required)
>   ```cpp
>   virtual void print() = 0;  // No implementation, must be overridden
>   ```
> Pure virtual functions make a class abstract (cannot be instantiated), forcing a contract on derived classes.
