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

# 08 - Preprocessor Directives and Macros

> [!IMPORTANT]
> **Compile-Time Processing**: The preprocessor runs **before** compilation, performing text substitutions and conditional inclusion. Macros are powerful but dangerous (no type safety). Use modern C++ alternatives whenever possible.

---

## The Two-Stage Compilation

```
C++ Source Code (.cpp)
    ↓
[PREPROCESSOR]  ← Handles #include, #define, #ifdef
    ↓
Expanded Source
    ↓
[COMPILER]      ← Checks types, generates object code
    ↓
Object Code (.obj)
```

---

## #include Directive

### System Headers

```cpp
#include <iostream>      // Searches system paths
#include <vector>
#include <string>
#include <fstream>
```

### Local Headers

```cpp
#include "myfile.h"      // Searches current directory first
#include "utils/helper.h"
```

### Difference

| `#include <...>` | `#include "..."` |
|------------------|------------------|
| System headers | Local/project headers |
| Standard library | Your code |
| Searched in `/usr/include` | Searched in current dir first |

---

## #define Macro

### Simple Text Replacement

```cpp
#define PI 3.14159
#define MAX_SIZE 100
#define DEBUG 1

cout << PI << endl;             // Substituted: 3.14159
int arr[MAX_SIZE];              // Substituted: int arr[100]

if (DEBUG) {
    cout << "Debug mode\n";     // if (1) after substitution
}
```

### Function-Like Macros

```cpp
#define SQUARE(x) ((x) * (x))
#define MAX(a, b) ((a) > (b) ? (a) : (b))
#define ABS(x) ((x) < 0 ? -(x) : (x))

cout << SQUARE(5) << endl;      // Expands to: ((5) * (5)) = 25
cout << MAX(3, 7) << endl;      // Expands to: ((3) > (7) ? (3) : (7)) = 7
```

> [!CAUTION]
> **Macro Pitfalls**: Macros are dumb text replacement. They don't understand C++ syntax.
>
> ```cpp
> #define DOUBLE(x) x * 2
> cout << DOUBLE(3 + 4) << endl;  // ✗ Expands to: 3 + 4 * 2 = 11 (wrong!)
> 
> // Fix: Add parentheses
> #define DOUBLE(x) ((x) * 2)     // Expands to: ((3 + 4) * 2) = 14 ✓
> ```

---

## Header Guards: Prevent Multiple Inclusion

### The Problem

```cpp
// math.h
struct Vector {
    double x, y;
};

// program.cpp
#include "math.h"
#include "math.h"       // Included twice!
                        // Error: Vector redefined
```

### Solution 1: #ifndef

```cpp
// math.h
#ifndef MATH_H
#define MATH_H

struct Vector {
    double x, y;
};

#endif  // MATH_H
```

### Solution 2: #pragma once (Modern)

```cpp
// math.h
#pragma once    // Included only once per compilation unit

struct Vector {
    double x, y;
};
```

---

## Conditional Compilation

### #ifdef / #endif

```cpp
#define DEBUG 1

#ifdef DEBUG
    cout << "Debug enabled\n";   // Compiled if DEBUG defined
#endif

#ifndef DEBUG
    cout << "Release mode\n";    // Compiled if DEBUG NOT defined
#endif
```

### #if / #else

```cpp
#define VERSION 2

#if VERSION == 1
    cout << "Version 1 code\n";
#elif VERSION == 2
    cout << "Version 2 code\n";
#else
    cout << "Unknown version\n";
#endif
```

### Platform-Specific Code

```cpp
#ifdef _WIN32
    #include <windows.h>
    cout << "Windows\n";
#elif __linux__
    #include <unistd.h>
    cout << "Linux\n";
#endif
```

---

## Multi-Line Macros

```cpp
#define PRINT_ARRAY(arr, size) \
    do { \
        for (int i = 0; i < size; i++) { \
            cout << arr[i] << " "; \
        } \
        cout << endl; \
    } while(0)

int arr[] = {1, 2, 3, 4, 5};
PRINT_ARRAY(arr, 5);            // Prints: 1 2 3 4 5
```

> [!NOTE]
> **Why `do { ... } while(0)`?** This pattern ensures the macro behaves like a single statement and works correctly inside `if` blocks without braces.

---

## Complete Example: Cross-Platform Code

```cpp
#ifndef CONFIG_H
#define CONFIG_H

#include <iostream>
using namespace std;

// Platform detection
#ifdef _WIN32
    #define PLATFORM "Windows"
    #define NEWLINE "\r\n"
#elif __APPLE__
    #define PLATFORM "macOS"
    #define NEWLINE "\n"
#elif __linux__
    #define PLATFORM "Linux"
    #define NEWLINE "\n"
#else
    #define PLATFORM "Unknown"
    #define NEWLINE "\n"
#endif

// Debug logging
#ifdef DEBUG
    #define LOG(msg) cout << "[DEBUG] " << msg << NEWLINE
#else
    #define LOG(msg)  // No-op in release
#endif

#endif  // CONFIG_H

// main.cpp
#define DEBUG 1
#include "config.h"

int main() {
    LOG("Program starting");
    cout << "Running on: " << PLATFORM << endl;
    LOG("Program ending");
    return 0;
}
```

---

## Modern Alternatives to Macros

### Instead of Macro Constants: `constexpr`

```cpp
// ✗ Old: Macro
#define PI 3.14159

// ✓ Modern: constexpr
constexpr double PI = 3.14159;
```

### Instead of Function Macros: Inline Functions

```cpp
// ✗ Old: Macro (no type safety)
#define SQUARE(x) ((x) * (x))

// ✓ Modern: Inline (type safe)
template <typename T>
inline T square(T x) {
    return x * x;
}
```

### Instead of Conditional Inclusion: `if constexpr`

```cpp
// ✗ Old: Preprocessor
#ifdef DEBUG
    cout << "Debug\n";
#endif

// ✓ Modern: Compile-time if
if constexpr (DEBUG) {
    cout << "Debug\n";
}
```

---

## Common Preprocessor Symbols

| Symbol | Meaning |
|--------|---------|
| `__LINE__` | Current line number |
| `__FILE__` | Current filename |
| `__DATE__` | Compilation date |
| `__TIME__` | Compilation time |
| `_WIN32` | Windows platform |
| `__linux__` | Linux platform |
| `__APPLE__` | macOS platform |

```cpp
cout << "File: " << __FILE__ << ", Line: " << __LINE__ << endl;
```

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| **Missing parentheses** | Operator precedence wrong | Always wrap: `((x))` |
| **Macro name collision** | Unexpected substitution | Use prefixes: `APP_ASSERT` |
| **Side effects** | `SQUARE(x++)` evaluates twice | Use `inline` functions |
| **Multiple inclusion** | Duplicate definitions | Use header guards |
| **Debugging difficulty** | Errors show expanded code | Use modern `constexpr` |

---

## Best Practices

> [!IMPORTANT]
> **When to Use Macros**:
> - Header guards (`#ifndef`)
> - Platform detection (`#ifdef _WIN32`)
> - Debug logging (disabled in release)
>
> **When NOT to Use Macros**:
> - Constants → use `constexpr`
> - Functions → use `inline` or `template`
> - Conditional logic → use `if constexpr`

---

## Mastery Checklist

- [ ] Understand preprocessor vs. compiler stages
- [ ] Use `#include` for both system and local headers
- [ ] Create proper header guards or use `#pragma once`
- [ ] Define simple macros with `#define`
- [ ] Avoid function-like macro pitfalls
- [ ] Use conditional compilation for platform code
- [ ] Write multi-line macros with `\` continuation
- [ ] Know when to replace macros with `constexpr` or `inline`
- [ ] Debug by examining preprocessor output
- [ ] Understand `__FILE__`, `__LINE__`, platform macros
