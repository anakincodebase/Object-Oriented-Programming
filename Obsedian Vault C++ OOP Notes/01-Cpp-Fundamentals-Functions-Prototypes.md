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
# 01 - C++ Fundamentals, Functions, and Prototypes

> [!IMPORTANT]
> **Foundation of All C++**: C++ is a *compiled, statically typed* language where the compiler enforces type safety and generates machine code **before** execution. Every later concept—OOP, templates, memory management—depends on understanding this paradigm. Functions are the atomic building blocks from which all larger abstractions are constructed.

---

## The C++ Compilation Model

### What Makes C++ Different

Unlike interpreted languages (Python, JavaScript), C++ code goes through a **multi-stage compilation pipeline**:

```
┌─────────────┐
│ Source Code │
│  (.cpp)     │
└──────┬──────┘
       │
       ▼
┌─────────────────────┐
│ Preprocessor        │ → Expands #include, #define, etc.
├─────────────────────┤
│ Compiler            │ → Type checking, generates assembly
├─────────────────────┤
│ Assembler           │ → Converts to machine code (.obj files)
├─────────────────────┤
│ Linker              │ → Combines .obj files, resolves symbols
└──────┬──────────────┘
       │
       ▼
┌─────────────────┐
│ Executable      │ → Ready to run
│  (.exe / binary)│
└─────────────────┘
```

> [!NOTE]
> **Compilation is Your Friend**: Errors that would crash a Python program at runtime are caught by C++ compiler before execution. This "fail-fast" philosophy catches bugs early.

### Key Implications

| Aspect | Consequence |
|--------|-------------|
| **Statically Typed** | Declare types upfront; compiler verifies compatibility |
| **Compiled** | Cannot modify code at runtime; all decisions made pre-execution |
| **Multi-pass** | Requires declarations before use (forward declarations) |
| **Strongly Typed** | Implicit conversions are rare; explicit casts required |

---

## Functions: Building Blocks of C++

### Function Anatomy

```cpp
┌──────────┬──────────┬──────────┬─────────┐
│ Return   │ Function │ Parameter│  Body   │
│  Type    │  Name    │  List    │         │
└──────────┴──────────┴──────────┴─────────┘
    ↓         ↓            ↓         ↓
int      sum(int a, int b) {
    return a + b;  // Returns value of type int
}
```

### Function Declaration vs. Definition

**Declaration** (Prototype):
```cpp
int sum(int a, int b);              // Just promises it exists
```

**Definition** (Implementation):
```cpp
int sum(int a, int b) {             // Actually implements it
    return a + b;
}
```

> [!NOTE]
> **Why Prototypes Matter**: In a file, C++ reads top-to-bottom. If `main()` calls `sum()` but `sum()` is defined after `main()`, the compiler doesn't know what `sum` is. The prototype tells the compiler: "Trust me, this function will exist by the time we link."

### Complete Example: Three-Tier Architecture

```cpp
#include <iostream>
using namespace std;

// ========== PROTOTYPES (Declarations) ==========
int get_input();
bool validate_age(int age);
string calculate_status(int age);
void print_result(string status);

// ========== MAIN LOGIC ==========
int main() {
    int age = get_input();              // Call 1
    
    if (!validate_age(age)) {           // Call 2
        print_result("Invalid");
        return 1;
    }
    
    string status = calculate_status(age);  // Call 3
    print_result(status);                   // Call 4
    
    return 0;
}

// ========== IMPLEMENTATIONS (Definitions) ==========
int get_input() {
    int age;
    cout << "Enter age: ";
    cin >> age;
    return age;
}

bool validate_age(int age) {
    return (age > 0 && age < 120);
}

string calculate_status(int age) {
    if (age < 13) return "Child";
    if (age < 18) return "Teen";
    if (age < 65) return "Adult";
    return "Senior";
}

void print_result(string status) {
    cout << "Status: " << status << endl;
}
```

---

## Function Signature and Overloading

### What is a Signature?

A **signature** consists of:
- Function name
- Parameter types (in order)
- Return type (partially—not always part of signature)

```cpp
// Signatures:
int add(int a, int b);              // add(int, int)
int add(double a, double b);        // add(double, double) - DIFFERENT!
int add(int a, int b, int c);       // add(int, int, int) - DIFFERENT!
```

### Function Overloading

C++ allows **same function name** with **different signatures**. The compiler picks based on argument types:

```cpp
class Math {
public:
    // Overload 1: integers
    int max(int a, int b) {
        return (a > b) ? a : b;
    }
    
    // Overload 2: doubles
    double max(double a, double b) {
        return (a > b) ? a : b;
    }
    
    // Overload 3: three integers
    int max(int a, int b, int c) {
        return max(max(a, b), c);
    }
};

int main() {
    Math m;
    cout << m.max(5, 10) << endl;           // Calls Overload 1
    cout << m.max(3.14, 2.71) << endl;     // Calls Overload 2
    cout << m.max(1, 5, 3) << endl;        // Calls Overload 3
    return 0;
}
```

> [!CAUTION]
> **Overload Resolution Complexity**: The compiler uses sophisticated rules to match calls to overloads. Ambiguous overloads cause compile errors. Keep overloads simple and distinguish clearly.

---

## Parameter Passing: By Value vs. By Reference

### By Value (Primitive Types)

```cpp
void increment_by_value(int x) {
    x++;                    // Only modifies local copy
}

int main() {
    int num = 5;
    increment_by_value(num);
    cout << num << endl;    // Still 5: original unchanged
    return 0;
}
```

**Copy Process**:
```
Calling:    num = 5
            ↓ (copies value)
Function:   x = 5 (local copy)
            x++ (local copy modified)
            ↓ (local copy destroyed)
Calling:    num = 5 (unchanged)
```

### By Reference (Modify Original)

```cpp
void increment_by_reference(int &x) {   // & means reference
    x++;                    // Modifies original
}

int main() {
    int num = 5;
    increment_by_reference(num);
    cout << num << endl;    // Now 6: original changed
    return 0;
}
```

**Reference Process**:
```
Calling:    num = 5
            ↓ (passes reference/alias)
Function:   x = reference to num (same variable!)
            x++ (modifies original num)
            ↓ (reference destroyed, but num persists)
Calling:    num = 6 (changed!)
```

### When to Use What

| Type | By Value | By Reference |
|------|----------|--------------|
| **`int`, `double`** | ✓ Fast, safe | ✗ Unusual |
| **Array/String** | ✗ Expensive copy | ✓ Efficient |
| **Object** | ✗ Copy overhead | ✓ Preferred |
| **Want to modify?** | ✗ Can't | ✓ By reference |
| **Const data** | ✓ Copy | ✓ Const reference |

### Const Reference: Read-Only, No Copy

```cpp
void print_long_string(const string &s) {   // No copy, read-only
    cout << s << endl;
}

int main() {
    string message = "Hello World";
    print_long_string(message);              // Efficient: no copy made
    return 0;
}
```

> [!TIP]
> **Professional Rule**: For anything larger than an `int` or `double`, use references. For modification, use `&`. For read-only, use `const &`.

---

## Return Values and Early Exit

### Simple Return

```cpp
int divide_safe(int numerator, int denominator) {
    if (denominator == 0) {
        return -1;              // Error signal
    }
    return numerator / denominator;
}
```

### Multiple Return Paths

```cpp
string grade_letter(int score) {
    if (score >= 90) return "A";
    if (score >= 80) return "B";
    if (score >= 70) return "C";
    if (score >= 60) return "D";
    return "F";
}
```

> [!CAUTION]
> **Dangling Reference Danger**: Never return a reference to a local variable—it's destroyed when function exits!
> ```cpp
> string &bad_return() {
>     string local = "hello";
>     return local;           // ✗ WRONG: returns reference to destroyed variable
> }
> ```

---

## Scope: Lifetime and Visibility

### Local Scope (Function Scope)

```cpp
int main() {
    int x = 5;              // x created, scope begins
    {
        int y = 10;         // y created, inner scope
        cout << x << y;     // Both visible here
    }                       // y destroyed here
    
    // cout << y;           // ✗ Error: y out of scope
    cout << x;              // x still visible
    return 0;
}                           // x destroyed here
```

### Global vs. Local

```cpp
int global_var = 100;       // Global: visible everywhere

int main() {
    int local_var = 5;      // Local: visible only in main
    {
        int local_var = 10; // Shadows outer local_var
        cout << local_var;  // 10: uses inner scope version
        cout << global_var; // 100: still accessible
    }
    cout << local_var;      // 5: back to outer version
    return 0;
}
```

> [!CAUTION]
> **Global Variables Are Evil**: Avoid global state. It makes code hard to test and reason about. Pass data as parameters instead.

---

## Best Practices for Function Design

### 1. Single Responsibility

```cpp
// ✓ GOOD: Each function does one thing
int get_user_age() { /* input */ }
bool is_valid_age(int age) { /* validation */ }
void display_age_category(int age) { /* output */ }
```

### 2. Meaningful Names

```cpp
// ✓ GOOD: Clear intent
int calculate_total_price(vector<int> prices) { }
bool is_adult(int age) { }
void print_formatted_receipt(double total) { }
```

---

## Professional C++ Function Design

### 1. Use `const` for Read-Only Parameters

```cpp
// ❌ WRONG: implies you might modify the string (you won't)
void printUserInfo(string name) {
    cout << "Name: " << name << "\n";
}

// ✅ RIGHT: clearly states you won't modify it
void printUserInfo(const string& name) {
    cout << "Name: " << name << "\n";
}

// ✅ Even better for objects: use const reference to avoid copying
void processLargeData(const vector<int>& data) {
    // Tells compiler and reader: data won't be modified
    int sum = 0;
    for (int val : data) sum += val;
    return sum;
}
```

### 2. Return by Const Reference When Appropriate

```cpp
// ❌ INEFFICIENT: returns copy
string getName() {
    static string name = "Alice";
    return name;  // Copied!
}

// ✅ EFFICIENT: returns reference (only if returning static or member variable)
const string& getName() {
    static string name = "Alice";
    return name;  // Reference, no copy
}

// ⚠️ WARNING: Don't return reference to local variable!
const int& badFunction() {
    int x = 5;
    return x;  // ✗ WRONG: x is destroyed at function exit!
}
```

### 3. Use Meaningful Default Parameters

```cpp
// ✗ BAD: what does true mean? What is 100?
void initialize(bool flag, int size);
initialize(true, 100);  // Unclear intent

// ✓ GOOD: Named parameters with defaults (requires wrapper or structure)
void initialize(bool enableLogging = false, int size = 50) { }
initialize();                      // Uses defaults
initialize(true);                  // Custom logging
initialize(true, 200);             // Custom logging and size
```

### 4. Avoid Side Effects: Pure Functions

```cpp
// ❌ BAD: function modifies global state
int global_counter = 0;
int increment() {
    return ++global_counter;  // Side effect: modifies global
}

// ✓ GOOD: function has no side effects (pure)
int increment(int &counter) {
    return ++counter;  // Explicitly receives what to modify
}

// ✓ BETTER: function has no side effects at all
int add(int a, int b) {
    return a + b;  // No side effects, predictable
}
```

### 5. Error Handling: Return Status or Throw

```cpp
// ✗ OLD C-style: magic return values
int readFile(string filename) {
    // Returns -1 for error, file size otherwise
    // Caller must know -1 means error
}

// ✓ C++: Use exceptions for truly exceptional conditions
int readFile(string filename) {
    if (!fileExists(filename)) {
        throw runtime_error("File not found: " + filename);
    }
    // ... read and return file size
}

// ✓ OR: Return optional result (C++17)
#include <optional>
optional<int> readFile(string filename) {
    if (!fileExists(filename)) {
        return {};  // Empty optional = error
    }
    return fileSize;  // Non-empty optional = success
}
```

### 6. Function Overloading Best Practices

```cpp
// ✓ GOOD: Clear overloads with obvious differences
int add(int a, int b) { return a + b; }
double add(double a, double b) { return a + b; }
string add(string a, string b) { return a + b; }  // Concatenation

// ✗ CONFUSING: Overloads with different behavior
void process(int x) { x *= 2; }          // Multiplies
void process(double x) { x += 1; }       // Adds one
// Caller might be confused which will be called

// ✓ BETTER: Use templates for this pattern
template <typename T>
T process(T value) { /* consistent behavior */ }
```

### 7. Avoid Using Global Variables

```cpp
// ✗ ANTI-PATTERN: Global state makes testing/reasoning difficult
int global_total = 0;
void addToTotal(int value) {
    global_total += value;
}

// ✓ PATTERN: Pass what you need
int addToTotal(int currentTotal, int value) {
    return currentTotal + value;
}

// ✓ PATTERN: Use classes for related state
class TotalTracker {
private:
    int total;
public:
    void add(int value) { total += value; }
    int getTotal() const { return total; }
};
```

> [!IMPORTANT]
> **Professional Function Guidelines**:
> 1. **Use `const` liberally** - communicate intent and enable optimizations
> 2. **Prefer pass-by-const-reference** over pass-by-value for large objects
> 3. **Keep functions small and focused** - single responsibility principle
> 4. **Use meaningful names** that describe what the function does
> 5. **Avoid global state** - pass dependencies explicitly
> 6. **Prefer exceptions for errors** over magic return values (or use `std::optional`)
> 7. **Document preconditions and postconditions** for complex functions

---

## Mastery Checklist

- [ ] Explain the C++ compilation pipeline
- [ ] Write a function prototype and definition separately
- [ ] Overload a function with different parameter types
- [ ] Use `&` to pass by reference and modify originals
- [ ] Use `const &` for read-only efficient passing
- [ ] Understand scope: local vs. global
- [ ] Design functions with single responsibility
- [ ] Avoid dangling references
- [ ] Use `const` correctly for parameters and return types
- [ ] Write pure functions with no side effects
- [ ] Handle errors with exceptions or `std::optional`
- [ ] Avoid global variables
