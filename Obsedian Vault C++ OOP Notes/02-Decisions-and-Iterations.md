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

# 02 - Decisions and Iterations: Control Flow

> [!IMPORTANT]
> **Logic Foundation**: Decision structures and loops are the foundation of all program logic. Every algorithm—sorting, searching, parsing—is built from `if`/`switch` and `for`/`while`. Mastering these patterns is essential before understanding OOP abstractions.

---

## Decision Structures

### The `if-else` Hierarchy

```cpp
if (condition1) {
    // Execute if condition1 is true
} else if (condition2) {
    // Execute if condition1 is false AND condition2 is true
} else if (condition3) {
    // Execute if condition1 and condition2 are false AND condition3 is true
} else {
    // Execute if all conditions are false
}
```

**Execution Flow**:
```
┌─────────────────────┐
│ condition1 true?    │─ YES → Execute block 1 → DONE
└──────┬──────────────┘
       │ NO
       ▼
┌─────────────────────┐
│ condition2 true?    │─ YES → Execute block 2 → DONE
└──────┬──────────────┘
       │ NO
       ▼
┌─────────────────────┐
│ condition3 true?    │─ YES → Execute block 3 → DONE
└──────┬──────────────┘
       │ NO
       ▼
Execute else block → DONE
```

### Real-World Example: Grade Calculator

```cpp
int get_letter_grade(int score) {
    if (score >= 90) {
        return 'A';
    } else if (score >= 80) {
        return 'B';
    } else if (score >= 70) {
        return 'C';
    } else if (score >= 60) {
        return 'D';
    } else {
        return 'F';
    }
}
```

### The `switch` Statement (Discrete Values)

Use `switch` when comparing one value against many discrete options:

```cpp
char get_day_name(int day) {
    switch (day) {
        case 1: return "Monday";
        case 2: return "Tuesday";
        case 3: return "Wednesday";
        case 4: return "Thursday";
        case 5: return "Friday";
        case 6: return "Saturday";
        case 7: return "Sunday";
        default: return "Invalid";
    }
}
```

> [!NOTE]
> **Switch vs. If-Else**: Use `switch` for discrete values (days, menu options). Use `if-else` for ranges and boolean logic.

### Boolean Logic: `&&`, `||`, `!`

```cpp
// AND operator: both must be true
if (age >= 18 && has_license) {
    cout << "Can drive" << endl;
}

// OR operator: at least one must be true
if (day == "Saturday" || day == "Sunday") {
    cout << "Weekend" << endl;
}

// NOT operator: reverses boolean
if (!is_raining) {
    cout << "Go outside" << endl;
}

// Combined
if ((score >= 90 && attendance >= 80) || extra_credit) {
    cout << "Pass with honors" << endl;
}
```

**Truth Tables**:

| A | B | A && B | A ↔ B |
|---|---|--------|--------|
| T | T | T      | T      |
| T | F | F      | F      |
| F | T | F      | F      |
| F | F | F      | T      |

> [!CAUTION]
> **Short-Circuit Evaluation**: 
> - `A && B`: If A is false, B is never evaluated
> - `A || B`: If A is true, B is never evaluated
> This can affect behavior if B has side effects!

---

## Iteration Structures

### The `for` Loop: Count-Based

Use `for` when you know **exactly** how many times to iterate:

```cpp
for (int i = 0; i < 10; i++) {
    cout << i << " ";  // Prints: 0 1 2 3 4 5 6 7 8 9
}

// Breakdown:
// ┌─────────────────┐
// │ Initialization  │ i = 0 (runs once, before loop)
// │ Condition       │ i < 10 (checked each iteration)
// │ Update          │ i++ (runs after each iteration)
// └─────────────────┘
```

**Execution Timeline**:
```
1. Initialize: i = 0
2. Check: 0 < 10? YES
3. Execute body: cout << 0
4. Update: i++
5. Check: 1 < 10? YES
6. Execute body: cout << 1
...continue...
N. Check: 10 < 10? NO → Exit loop
```

### The `while` Loop: Condition-Based

Use `while` when **stop condition is data-dependent** or based on input:

```cpp
int sum = 0;
int num;

while (true) {
    cout << "Enter a number (0 to stop): ";
    cin >> num;
    
    if (num == 0) break;        // Exit loop
    
    sum += num;
}

cout << "Sum: " << sum << endl;
```

### The `do-while` Loop: Execute First, Check Later

Use `do-while` when you must execute the body **at least once**:

```cpp
int choice;

do {
    cout << "===== MENU =====" << endl;
    cout << "1. Add" << endl;
    cout << "2. Subtract" << endl;
    cout << "3. Exit" << endl;
    cout << "Enter choice: ";
    cin >> choice;
    
    switch (choice) {
        case 1: add_numbers(); break;
        case 2: subtract_numbers(); break;
        case 3: cout << "Goodbye!" << endl; break;
        default: cout << "Invalid choice" << endl;
    }
} while (choice != 3);
```

**Key Difference**:
```cpp
// while: may never execute
while (false) {
    cout << "Never prints" << endl;
}

// do-while: always executes at least once
do {
    cout << "Prints once" << endl;
} while (false);
```

---

## Nested Loops: Multi-Dimensional Iteration

### Multiplication Table

```cpp
for (int i = 1; i <= 10; i++) {
    for (int j = 1; j <= 10; j++) {
        cout << (i * j) << "\t";   // \t = tab
    }
    cout << endl;                   // New line after each row
}
```

**Output**:
```
1       2       3       4   ... 10
2       4       6       8   ... 20
3       6       9       12  ... 30
...
10      20      30      40  ... 100
```

### Complexity: O(n²)

```
Outer loop iterations: 10
Inner loop iterations per outer: 10
Total iterations: 10 × 10 = 100

General: n nested loops = O(n^n) complexity
```

---

## Loop Control: `break` and `continue`

### `break`: Exit Loop Immediately

```cpp
for (int i = 0; i < 10; i++) {
    if (i == 5) break;          // Exit when i reaches 5
    cout << i << " ";           // Output: 0 1 2 3 4
}
```

### `continue`: Skip to Next Iteration

```cpp
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) continue;   // Skip even numbers
    cout << i << " ";           // Output: 1 3 5 7 9
}
```

---

## Complete Example: Input Validation Loop

```cpp
#include <iostream>
using namespace std;

int main() {
    int age;
    bool valid = false;
    
    do {
        cout << "Enter your age (1-120): ";
        cin >> age;
        
        if (age < 1 || age > 120) {
            cout << "❌ Invalid! Age must be between 1 and 120" << endl;
        } else if (cin.fail()) {
            cout << "❌ Please enter a number" << endl;
            cin.clear();
            cin.ignore(10000, '\n');
        } else {
            valid = true;
        }
    } while (!valid);
    
    cout << "✓ You are " << age << " years old" << endl;
    
    // Classify age
    if (age < 13) {
        cout << "Category: Child" << endl;
    } else if (age < 18) {
        cout << "Category: Teen" << endl;
    } else if (age < 65) {
        cout << "Category: Adult" << endl;
    } else {
        cout << "Category: Senior" << endl;
    }
    
    return 0;
}
```

---

## Loop Invariants: Reasoning About Correctness

A **loop invariant** is a statement that remains true:
- Before the loop
- After each iteration
- After the loop exits

```cpp
// Sum all numbers from 1 to n
int sum = 0;                    // Invariant: sum = 0+1+2+...+i-1
for (int i = 1; i <= n; i++) {
    sum += i;                   // Invariant: sum = 0+1+2+...+i
}
// After loop: sum = 0+1+2+...+n ✓
```

---

## Common Loop Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| **Off-by-one** | Loop runs 1 extra/fewer time | Check boundary conditions carefully |
| **Infinite loop** | Loop never terminates | Verify increment/update condition |
| **Missing break** | `switch` falls through | Add `break` after each case |
| **Wrong condition** | Loop exits early/late | Test boundary values |
| **Variable scope** | Loop variable undefined outside | Declare outside loop if needed |

---

## Performance Considerations

### Loop Optimization

```cpp
// INEFFICIENT: Calls size() each iteration
for (int i = 0; i < vec.size(); i++) { // size() called 10x
    cout << vec[i] << endl;
}

// EFFICIENT: Call size() once
int n = vec.size();
for (int i = 0; i < n; i++) {
    cout << vec[i] << endl;
}

// MODERN C++: Range-based for
for (int value : vec) {             // Cleaner, same performance
    cout << value << endl;
}
```

---

## Mastery Checklist

- [ ] Use `if-else` for range-based decisions
- [ ] Use `switch` for discrete value comparisons
- [ ] Understand `&&`, `||`, `!` boolean operators
- [ ] Use `for` loops for count-based iteration
- [ ] Use `while` for condition-based iteration
- [ ] Use `do-while` for at-least-once execution
- [ ] Implement nested loops for 2D problems
- [ ] Use `break` and `continue` appropriately
- [ ] Write input validation loops
- [ ] Identify and fix off-by-one errors
- [ ] Reason about loop invariants
