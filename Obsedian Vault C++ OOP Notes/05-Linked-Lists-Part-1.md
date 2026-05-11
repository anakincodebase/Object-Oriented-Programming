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

# 05 - Linked Lists Part 1: Foundation

> [!IMPORTANT]
> **Non-Contiguous Storage**: Unlike arrays (contiguous memory), linked lists use **pointers to connect dynamically allocated nodes**. This enables efficient insertion/deletion but loses random access. Linked lists introduce the critical pattern: navigating data structures through pointers.

---

## Linked List Fundamentals

### Node Structure

```cpp
struct Node {
    int data;           // The actual value
    Node *next;         // Pointer to next node
};
```

**Memory Layout**:
```
Node 1              Node 2              Node 3
┌──────┬────────┐  ┌──────┬────────┐  ┌──────┬────────┐
│data:1│next ──────→│data:2│next ──────→│data:3│next:NULL│
└──────┴────────┘  └──────┴────────┘  └──────┴────────┘
 (0x1000)           (0x2000)           (0x3000)
```

### Creating a List

```cpp
// Create head pointer
Node *head = NULL;              // Empty list

// Add first node
head = new Node();
head->data = 10;
head->next = NULL;

// Add second node
Node *new_node = new Node();
new_node->data = 20;
new_node->next = NULL;
head->next = new_node;
```

---

## Core Operations

### 1. Insert at Head (O(1))

**Complexity**: O(1) because we just change pointers, no traversal needed.

```cpp
void insert_at_head(Node *&head, int value) {
    Node *new_node = new Node();
    new_node->data = value;
    new_node->next = head;      // New node points to old head
    head = new_node;            // New node becomes head
}

int main() {
    Node *head = NULL;
    
    insert_at_head(head, 10);
    insert_at_head(head, 20);
    insert_at_head(head, 30);
    
    // List is now: 30 → 20 → 10 → NULL
}
```

**Step-by-step**:
```
Before: head → 10 → 20 → NULL

Insert 30:
1. new_node = {data: 30, next: NULL}
2. new_node->next = head (point to 10)
3. head = new_node

After:  head → 30 → 10 → 20 → NULL
```

### 2. Traverse and Display (O(n))

```cpp
void display(Node *head) {
    Node *current = head;
    
    while (current != NULL) {
        cout << current->data << " → ";
        current = current->next;
    }
    cout << "NULL" << endl;
}

// Usage
display(head);  // Output: 30 → 20 → 10 → NULL
```

### 3. Count Elements (O(n))

```cpp
int count_nodes(Node *head) {
    int count = 0;
    Node *current = head;
    
    while (current != NULL) {
        count++;
        current = current->next;
    }
    
    return count;
}

// Usage
cout << "List has " << count_nodes(head) << " nodes" << endl;
```

### 4. Search for Value (O(n))

```cpp
bool search(Node *head, int target) {
    Node *current = head;
    
    while (current != NULL) {
        if (current->data == target) {
            return true;
        }
        current = current->next;
    }
    
    return false;
}

// Usage
if (search(head, 20)) {
    cout << "Found!" << endl;
}
```

---

## Pass by Reference: Critical for Head Pointer

### The Problem

```cpp
void bad_insert(Node *head, int value) {  // Passed by value
    Node *new_node = new Node();
    new_node->data = value;
    new_node->next = head;
    head = new_node;                      // Only local head changed!
}

int main() {
    Node *head = NULL;
    bad_insert(head, 10);
    // head is STILL NULL! New node was lost.
}
```

### The Solution

```cpp
void good_insert(Node *&head, int value) {  // Passed by REFERENCE
    Node *new_node = new Node();
    new_node->data = value;
    new_node->next = head;
    head = new_node;                      // Original head changed!
}

int main() {
    Node *head = NULL;
    good_insert(head, 10);
    // head NOW points to the new node!
}
```

> [!IMPORTANT]
> **Pass Head by Reference**: When inserting at head, the head pointer itself changes. Use `Node *&head` (reference to pointer) to modify the original head.

---

## Complete Example: Simple Linked List

```cpp
#include <iostream>
using namespace std;

struct Node {
    int data;
    Node *next;
};

// Insert at head
void insert_head(Node *&head, int value) {
    Node *new_node = new Node();
    new_node->data = value;
    new_node->next = head;
    head = new_node;
}

// Display
void display(Node *head) {
    cout << "List: ";
    while (head != NULL) {
        cout << head->data << " → ";
        head = head->next;
    }
    cout << "NULL\n";
}

// Count
int count_nodes(Node *head) {
    int count = 0;
    while (head != NULL) {
        count++;
        head = head->next;
    }
    return count;
}

int main() {
    Node *head = NULL;
    
    insert_head(head, 10);
    insert_head(head, 20);
    insert_head(head, 30);
    
    display(head);                       // List: 30 → 20 → 10 → NULL
    cout << "Size: " << count_nodes(head) << "\n";  // Size: 3
    
    return 0;
}
```

---

## Complexity Analysis

| Operation | Time | Why |
|-----------|------|-----|
| **Insert at head** | O(1) | No traversal needed |
| **Delete head** | O(1) | Just change head pointer |
| **Search** | O(n) | Must traverse to find |
| **Insert at tail** | O(n) | Must find tail first |
| **Access element at index k** | O(n) | Must traverse k nodes |

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| **Forgot new** | `Node n;` on stack, lost when scope ends | Always use `new` |
| **Forgot delete** | Memory leak | Use `delete` in destructor |
| **Pass head by value** | Modifications don't affect original | Pass by reference: `Node *&` |
| **Infinite loop** | Forgot to move `current = current->next` | Always advance pointer |
| **Null check** | Dereferencing NULL → crash | Check `if (ptr != NULL)` |

---

## Mastery Checklist

- [ ] Define a Node struct with data and next pointer
- [ ] Create a linked list by inserting at head
- [ ] Traverse a list without infinite loop
- [ ] Count nodes in a list
- [ ] Search for a value in a list
- [ ] Understand why head is passed by reference
- [ ] Identify memory management points (new/delete)
- [ ] Avoid common pitfalls: forgetting new, infinite loops, crashes on NULL
