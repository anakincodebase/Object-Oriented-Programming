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

# 16 - Tower of Hanoi: Recursion and Abstraction

> [!IMPORTANT]
> **Recursive Decomposition**: Tower of Hanoi is the canonical recursion problem. It teaches **problem decomposition** into simpler subproblems. Understanding how recursion "magically" solves exponential complexity is foundational for algorithm design.

---

## The Problem

### Rules

1. **Three pegs**: Source, Auxiliary, Destination
2. **Move all disks** from Source to Destination
3. **Constraints**:
   - Move only one disk at a time
   - Never place larger disk on smaller disk

### Visualization

```
Initial State:        Goal State:

Peg 1  Peg 2  Peg 3   Peg 1  Peg 2  Peg 3
  |      |      |       |      |      |
  *      |      |       |      |      |
 ***     |      |       |      |      |
*****    |      |       |      |      |
-----    -      -       -      -    -----

(All on Peg 1)                (All on Peg 3)
```

---

## Recursive Solution

### The Key Insight

**To move n disks from Source to Destination using Auxiliary**:

1. Move **n-1 disks** from Source to Auxiliary (using Destination as temporary)
2. Move **the largest disk** from Source to Destination
3. Move **n-1 disks** from Auxiliary to Destination (using Source as temporary)

```
Base case: n=1
  → Move directly

Recursive case: n>1
  → solve(n-1, from, to, temp) + move disk + solve(n-1, temp, from, to)
```

### Implementation

```cpp
#include <iostream>
using namespace std;

int move_count = 0;

void hanoi(int n, char source, char dest, char aux) {
    // Base case: single disk
    if (n == 1) {
        cout << "Move disk 1 from " << source << " to " << dest << endl;
        move_count++;
        return;
    }
    
    // Step 1: Move n-1 disks from source to auxiliary
    //         (using destination as temporary)
    hanoi(n - 1, source, aux, dest);
    
    // Step 2: Move largest disk from source to destination
    cout << "Move disk " << n << " from " << source 
         << " to " << dest << endl;
    move_count++;
    
    // Step 3: Move n-1 disks from auxiliary to destination
    //         (using source as temporary)
    hanoi(n - 1, aux, dest, source);
}

int main() {
    int n = 3;  // 3 disks
    
    hanoi(n, 'A', 'C', 'B');
    
    cout << "\nTotal moves: " << move_count << endl;
    // 2^3 - 1 = 7 moves
    
    return 0;
}
```

### Output for n=3

```
Move disk 1 from A to C
Move disk 2 from A to B
Move disk 1 from C to B
Move disk 3 from A to C
Move disk 1 from B to A
Move disk 2 from B to C
Move disk 1 from A to C

Total moves: 7
```

---

## Recursion Trace

### Call Tree for n=3

```
hanoi(3, A, C, B)
├── hanoi(2, A, B, C)           [Move 2 from A to B using C]
│   ├── hanoi(1, A, C, B)       [Move 1 from A to C]
│   │   └── Move disk 1
│   ├── Move disk 2
│   └── hanoi(1, C, B, A)       [Move 1 from C to B]
│       └── Move disk 1
├── Move disk 3
└── hanoi(2, B, C, A)           [Move 2 from B to C using A]
    ├── hanoi(1, B, A, C)       [Move 1 from B to A]
    │   └── Move disk 1
    ├── Move disk 2
    └── hanoi(1, A, C, B)       [Move 1 from A to C]
        └── Move disk 1
```

---

## Complexity Analysis

### Number of Moves

- n=1: 1 move
- n=2: 3 moves
- n=3: 7 moves
- n=n: **2^n - 1** moves

$$T(n) = 2^n - 1$$

### Exponential Growth

| n | Moves | Time (1M moves/sec) |
|---|-------|---------------------|
| 10 | 1,023 | <0.001 sec |
| 20 | 1,048,575 | ~1 sec |
| 30 | 1,073,741,823 | ~17 minutes |
| 40 | ~1 trillion | ~11 days |
| 64 | ~18 quintillion | **585 billion years** |

**Legend**: According to legend, monks at a temple solve the 64-disk problem. When they finish, the universe ends. At 1 move per second, this takes ~585 billion years!

---

## Advanced: Animated Simulation

```cpp
#include <iostream>
#include <vector>
using namespace std;

class Tower {
private:
    vector<int> pegs[3];
    
    void display() {
        cout << "\nState:\n";
        for (int i = 0; i < 3; i++) {
            cout << "Peg " << i << ": ";
            for (int disk : pegs[i]) {
                cout << disk << " ";
            }
            cout << "\n";
        }
    }
    
public:
    Tower(int n) {
        for (int i = n; i >= 1; i--) {
            pegs[0].push_back(i);
        }
    }
    
    void solve(int n, int source, int dest, int aux) {
        if (n == 1) {
            // Move disk from source to dest
            int disk = pegs[source].back();
            pegs[source].pop_back();
            pegs[dest].push_back(disk);
            
            cout << "Move disk " << disk << " from peg " << source 
                 << " to peg " << dest;
            display();
            return;
        }
        
        solve(n - 1, source, aux, dest);
        solve(1, source, dest, aux);
        solve(n - 1, aux, dest, source);
    }
};

int main() {
    Tower tower(3);
    tower.solve(3, 0, 2, 1);
    return 0;
}
```

---

## Recursive Pattern: Divide and Conquer

```
Tower of Hanoi = Divide and Conquer

Problem: Move n disks
    ↓
Divide: Split into 3 subproblems
    ↓
Conquer: Solve each subproblem (smaller n)
    ↓
Combine: Larger disk + solved subproblems
```

### Other Applications

| Problem | Divide | Conquer | Combine |
|---------|--------|---------|---------|
| **Tower of Hanoi** | Move n-1 disks to aux | Move largest | Move n-1 from aux |
| **Merge Sort** | Split array in half | Sort each half | Merge sorted halves |
| **Binary Search** | Choose left or right | Search half | Return result |
| **Quick Sort** | Partition around pivot | Sort parts | Combine |

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| **Wrong peg** | Using source/dest/aux incorrectly | Trace base case carefully |
| **Infinite recursion** | Forgot base case | Always include `if (n == 1)` |
| **Stack overflow** | Too many recursive calls | Larger n → stack limits |
| **Wrong order** | Moving steps in wrong sequence | Step 1→2→3 order is critical |

---

## Mastery Checklist

- [ ] Understand the problem constraints
- [ ] Identify the recursive structure (n-1, base, n-1)
- [ ] Trace hanoi(3) by hand
- [ ] Implement recursive solution
- [ ] Trace the call tree
- [ ] Calculate complexity: 2^n - 1 moves
- [ ] Understand exponential growth
- [ ] Implement animated version
- [ ] Recognize divide-and-conquer pattern
- [ ] Apply hanoi pattern to other problems

> [!EXAMPLE]
> **Interview Question**: "How many moves does it take to solve Tower of Hanoi with 10 disks?"
> 
> **Answer**: $2^{10} - 1 = 1023$ moves
