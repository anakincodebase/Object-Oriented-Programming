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

# 00a - Number Systems: Binary, Hexadecimal, and Bitwise Operations

> [!IMPORTANT]
> **Why This Matters**: Computers store everything as binary digits (0 and 1). Understanding number systems is **essential** for:
> - **Memory addresses**: Displayed in hexadecimal (0x7FFF...)
> - **Bitwise operations**: AND, OR, XOR, shifts used in data structures
> - **Debugging**: Reading memory dumps, understanding pointers
> - **Optimization**: Bit manipulation for performance
> - **Data representation**: How integers/floats actually occupy memory

---

## The Three Essential Number Systems

### Decimal (Base 10): What We Know
```
Normal everyday numbers: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9

Example: 
  (5 × 10²) + (3 × 10¹) + (7 × 10⁰) = 500 + 30 + 7 = 537
   │ hundreds      │ tens      │ ones
```

### Binary (Base 2): What Computers Know
```
Only two digits: 0 and 1

Example: 1011 in binary
  (1 × 2³) + (0 × 2²) + (1 × 2¹) + (1 × 2⁰)
  = 8 + 0 + 2 + 1 = 11 in decimal

Conversion table:
Decimal  →  Binary
0        →  0000
1        →  0001
2        →  0010
3        →  0011
4        →  0100
5        →  0101
6        →  0110
7        →  0111
8        →  1000
9        →  1001
10       →  1010
11       →  1011
12       →  1100
13       →  1101
14       →  1110
15       →  1111
```

### Hexadecimal (Base 16): The Programmer's Shorthand
```
Digits: 0-9, then A(10), B(11), C(12), D(13), E(14), F(15)

Why use hex?
- 4 binary digits = 1 hex digit (compact!)
- Memory addresses shown in hex
- Color codes in graphics (#FF00FF)

Example: 0xAB7F in hexadecimal
  (A × 16³) + (B × 16²) + (7 × 16¹) + (F × 16⁰)
  = (10 × 4096) + (11 × 256) + (7 × 16) + (15 × 1)
  = 40960 + 2816 + 112 + 15 = 43903 in decimal

Hex Digit  →  Binary  →  Decimal
0          →  0000    →  0
1          →  0001    →  1
2          →  0010    →  2
3          →  0011    →  3
4          →  0100    →  4
5          →  0101    →  5
6          →  0110    →  6
7          →  0111    →  7
8          →  1000    →  8
9          →  1001    →  9
A          →  1010    →  10
B          →  1011    →  11
C          →  1100    →  12
D          →  1101    →  13
E          →  1110    →  14
F          →  1111    →  15
```

---

## Practical Conversions in C++

### Binary ↔ Decimal
```cpp
#include <iostream>
using namespace std;

// Decimal to binary string
string decimalToBinary(int num) {
    if (num == 0) return "0";
    
    string binary = "";
    while (num > 0) {
        binary = (num % 2 == 0 ? "0" : "1") + binary;
        num /= 2;
    }
    return binary;
}

// Binary string to decimal
int binaryToDecimal(string binary) {
    int decimal = 0;
    int power = 0;
    
    for (int i = binary.length() - 1; i >= 0; i--) {
        if (binary[i] == '1') {
            decimal += (1 << power);  // 1 * 2^power
        }
        power++;
    }
    return decimal;
}

int main() {
    cout << "587 in binary: " << decimalToBinary(587) << endl;
    // Output: 1001001011
    
    cout << "1001001011 in decimal: " << binaryToDecimal("1001001011") << endl;
    // Output: 587
    
    return 0;
}
```

### Decimal ↔ Hexadecimal
```cpp
#include <iostream>
#include <iomanip>
using namespace std;

int main() {
    int number = 43903;
    
    // Decimal to hex (C++ built-in)
    cout << "Decimal " << number << " in hex: ";
    cout << "0x" << hex << number << endl;  // Output: 0xab7f
    
    // Back to decimal
    cout << dec << "0xAB7F in decimal: ";
    cout << number << endl;  // Output: 43903
    
    // Without 0x prefix
    cout << hex << uppercase << number << endl;  // Output: AB7F
    
    return 0;
}
```

**Output:**
```
Decimal 43903 in hex: 0xab7f
0xAB7F in decimal: 43903
AB7F
```

---

## Bitwise Operations: Manipulating Individual Bits

### The Four Basic Operations

| Operation | Symbol | What It Does | Example |
|-----------|--------|-------------|---------|
| **AND** | `&` | Both bits must be 1 → 1 | `0101 & 0011 = 0001` |
| **OR** | `\|` | At least one bit is 1 → 1 | `0101 \| 0011 = 0111` |
| **XOR** | `^` | Bits are different → 1 | `0101 ^ 0011 = 0110` |
| **NOT** | `~` | Flip all bits | `~0101 = 1010` (with sign) |

### Truth Tables

#### AND (`&`)
```
A  B  A & B
0  0   0
0  1   0
1  0   0
1  1   1  ← Both must be 1
```

#### OR (`|`)
```
A  B  A | B
0  0   0
0  1   1
1  0   1
1  1   1  ← At least one is 1
```

#### XOR (`^`)
```
A  B  A ^ B
0  0   0
0  1   1  ← Different
1  0   1  ← Different
1  1   0
```

### Practical Bitwise Examples
```cpp
#include <iostream>
#include <bitset>
using namespace std;

int main() {
    int a = 0b1010;  // 10 in decimal
    int b = 0b1100;  // 12 in decimal
    
    cout << "a = " << bitset<4>(a) << " (decimal: " << a << ")" << endl;
    cout << "b = " << bitset<4>(b) << " (decimal: " << b << ")" << endl;
    
    // AND: Both must be 1
    cout << "a & b = " << bitset<4>(a & b) << " (decimal: " << (a & b) << ")" << endl;
    // Output: 1000 (8)
    
    // OR: At least one is 1
    cout << "a | b = " << bitset<4>(a | b) << " (decimal: " << (a | b) << ")" << endl;
    // Output: 1110 (14)
    
    // XOR: Bits are different
    cout << "a ^ b = " << bitset<4>(a ^ b) << " (decimal: " << (a ^ b) << ")" << endl;
    // Output: 0110 (6)
    
    // NOT: Flip all bits
    cout << "~a = " << bitset<4>(~a & 0x0F) << endl;
    // Output: 0101 (5) - with masking to show only 4 bits
    
    return 0;
}
```

**Output:**
```
a = 1010 (decimal: 10)
b = 1100 (decimal: 12)
a & b = 1000 (decimal: 8)
a | b = 1110 (decimal: 14)
a ^ b = 0110 (decimal: 6)
~a = 0101
```

---

## Bit Shifting: Moving Bits Left and Right

### Left Shift (`<<`)
```
Each position left = multiply by 2

Example: 0101 << 2 = 10100
         5   × 4  = 20

5 << 1 = 10   (multiply by 2)
5 << 2 = 20   (multiply by 4)
5 << 3 = 40   (multiply by 8)
```

### Right Shift (`>>`)
```
Each position right = divide by 2

Example: 1100 >> 2 = 0011
         12   ÷ 4  = 3

12 >> 1 = 6   (divide by 2)
12 >> 2 = 3   (divide by 4)
12 >> 3 = 1   (divide by 8)
```

### Practical Shift Examples
```cpp
#include <iostream>
#include <bitset>
using namespace std;

int main() {
    int value = 5;  // 0101 in binary
    
    cout << "Original: " << bitset<8>(value) 
         << " (decimal: " << value << ")" << endl;
    
    cout << "value << 1: " << bitset<8>(value << 1) 
         << " (decimal: " << (value << 1) << ")" << endl;
    // Output: 00001010 (10) - multiply by 2
    
    cout << "value << 3: " << bitset<8>(value << 3) 
         << " (decimal: " << (value << 3) << ")" << endl;
    // Output: 00101000 (40) - multiply by 8
    
    cout << "value >> 1: " << bitset<8>(value >> 1) 
         << " (decimal: " << (value >> 1) << ")" << endl;
    // Output: 00000010 (2) - divide by 2
    
    return 0;
}
```

**Output:**
```
Original: 00000101 (decimal: 5)
value << 1: 00001010 (decimal: 10)
value << 3: 00101000 (decimal: 40)
value >> 1: 00000010 (decimal: 2)
```

---

## Real-World Uses of Bitwise Operations

### 1. Check if Bit is Set
```cpp
// Check if bit at position `pos` is set to 1
bool isBitSet(int num, int pos) {
    return (num & (1 << pos)) != 0;
}

// Example: Is bit 2 of 0101 (5) set?
// 1 << 2 = 0100
// 0101 & 0100 = 0100 (non-zero, so YES)
cout << isBitSet(5, 2) << endl;  // Output: 1 (true)
```

### 2. Set a Specific Bit
```cpp
// Set bit at position `pos` to 1
int setBit(int num, int pos) {
    return num | (1 << pos);
}

// Example: Set bit 3 of 1010 (10)
// 1 << 3 = 1000
// 1010 | 1000 = 1010 (already set)
cout << setBit(10, 3) << endl;  // Output: 10
```

### 3. Clear a Specific Bit
```cpp
// Clear bit at position `pos` (set to 0)
int clearBit(int num, int pos) {
    return num & ~(1 << pos);
}

// Example: Clear bit 2 of 1101 (13)
// 1 << 2 = 0100
// ~0100 = 1011
// 1101 & 1011 = 1001 (9)
cout << clearBit(13, 2) << endl;  // Output: 9
```

### 4. Toggle a Specific Bit
```cpp
// Toggle bit at position `pos`
int toggleBit(int num, int pos) {
    return num ^ (1 << pos);
}

// Example: Toggle bit 0 of 1010 (10)
// 1 << 0 = 0001
// 1010 ^ 0001 = 1011 (11)
cout << toggleBit(10, 0) << endl;  // Output: 11
```

---

## Why This Matters for OOP and Data Structures

### Memory Addresses (Hexadecimal)
```cpp
int variable = 42;
int *ptr = &variable;

// Memory address shown in hex
cout << "Address of variable: " << ptr << endl;
// Output: 0x7ffe5fbff8ac  ← This is hexadecimal!
```

### Flags and Bitmasks (Bitwise Operations)
```cpp
// Using bits to store multiple boolean flags efficiently
class FilePermissions {
private:
    unsigned char permissions;  // 8 bits = 8 flags
    
public:
    const int READ = 1 << 0;    // Bit 0
    const int WRITE = 1 << 1;   // Bit 1
    const int EXECUTE = 1 << 2; // Bit 2
    
    void grantPermission(int perm) {
        permissions |= perm;  // Set bit
    }
    
    bool hasPermission(int perm) {
        return (permissions & perm) != 0;  // Check bit
    }
};
```

### Hash Functions and Checksums (Bitwise XOR)
```cpp
// Simple hash using XOR
int simpleHash(int a, int b) {
    return a ^ b;  // XOR for mixing bits
}
```

### Optimization: Powers of 2
```cpp
// Check if number is power of 2
bool isPowerOfTwo(int num) {
    return (num > 0) && ((num & (num - 1)) == 0);
}

// 8 = 1000, 8-1 = 0111
// 1000 & 0111 = 0000 → YES, power of 2!

// 6 = 0110, 6-1 = 0101
// 0110 & 0101 = 0100 → NO, not power of 2
```

---

## Professional Best Practices

### 1. Use Meaningful Bit Names
```cpp
// ✗ WRONG: Magic numbers
int status = 0b0110;

// ✓ RIGHT: Named constants
const int ACTIVE = 1 << 0;
const int READY = 1 << 1;
const int ERROR = 1 << 2;

int status = ACTIVE | READY;
```

### 2. Document Bit Layout
```cpp
// ✓ GOOD: Clear documentation of bit meanings
class NetworkPacket {
    // Byte layout:
    // [0:3]   version (4 bits)
    // [4:7]   headerLength (4 bits)
    // [8:9]   priority (2 bits)
    // [10:12] reserved (3 bits)
    // [13:15] flags (3 bits)
    
    unsigned short header;
};
```

### 3. Use Explicit Bit Width Types
```cpp
// ✓ RIGHT: Clear bit width
uint8_t flags;      // 8 bits guaranteed
uint16_t data;      // 16 bits guaranteed
uint32_t address;   // 32 bits guaranteed

// Use std::bitset for larger bit collections
std::bitset<256> largeFlags;
```

### 4. Avoid Undefined Behavior
```cpp
// ✗ WRONG: Undefined for negative numbers
int value = -5;
cout << (value >> 1) << endl;  // Implementation-defined!

// ✓ RIGHT: Use unsigned for bit operations
unsigned int value = 5;
cout << (value >> 1) << endl;  // Well-defined: 2
```

---

## Common Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **Confusing `&` with `&&`** | Bitwise AND vs logical AND | Use bitwise for bits, logical for conditions |
| **Forgetting operator precedence** | `a & b == 0` evaluates as `a & (b == 0)` | Use parentheses: `(a & b) == 0` |
| **Shifting too far** | Undefined behavior if shift >= word size | Check shift amount or use min() |
| **Signed right shift** | Implementation-defined behavior | Use unsigned types for bit operations |
| **Not masking bits** | Garbage bits from other operations | Always mask: `value & 0xFF` for 8 bits |

---

## Mastery Checklist

- [ ] Convert between decimal, binary, and hexadecimal fluently
- [ ] Understand why computers use binary (2 states per bit)
- [ ] Read and interpret memory addresses in hexadecimal
- [ ] Perform AND, OR, XOR operations on bits
- [ ] Understand truth tables for each bitwise operation
- [ ] Use left shift (`<<`) and right shift (`>>`) correctly
- [ ] Implement bit checking, setting, clearing, toggling
- [ ] Recognize when bitwise operations are appropriate
- [ ] Use meaningful names for bit flags
- [ ] Avoid undefined behavior with signed shifts
- [ ] Understand bit fields for compact storage
- [ ] Know binary operations for power-of-2 checks

> [!EXAMPLE]
> **Interview Question**: "How would you check if a number is a power of 2 using bitwise operations?"
>
> **Answer**: A power of 2 in binary has exactly one bit set:
> - 1 = 0001
> - 2 = 0010
> - 4 = 0100
> - 8 = 1000
>
> If you subtract 1 from a power of 2, all lower bits become 1, so AND with the original becomes 0:
> ```cpp
> bool isPowerOfTwo(int num) {
>     return (num > 0) && ((num & (num - 1)) == 0);
> }
> 
> // 8 & 7: 1000 & 0111 = 0000 → YES
> // 6 & 5: 0110 & 0101 = 0100 → NO
> ```
