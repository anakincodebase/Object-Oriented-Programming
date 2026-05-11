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

# 06 - Linked Lists Part 2: Advanced Operations

> [!IMPORTANT]
> **List Manipulation**: Deletion is the most complex linked list operation because you must track both the node to delete and its predecessor. Mastering pointer re-linking is essential for all advanced data structures (trees, graphs).

---

## Deletion Operations

### Delete Head (O(1))

```cpp
void delete_head(Node *&head) {
    if (head == NULL) {
        cout << "List empty!" << endl;
        return;
    }
    
    Node *temp = head;          // Save old head
    head = head->next;          // New head is second node
    delete temp;                // Free old head
    temp = NULL;
}
```

**Process**:
```
Before: head → [10] → [20] → [30] → NULL

1. temp = head (points to 10)
2. head = head->next (head now points to 20)
3. delete temp (free the node with 10)

After:  head → [20] → [30] → NULL
```

### Delete by Value (O(n))

This is **complex** because we must track both current and previous nodes:

```cpp
void delete_value(Node *&head, int target) {
    if (head == NULL) return;
    
    // Special case: delete head
    if (head->data == target) {
        delete_head(head);
        return;
    }
    
    // General case: find and delete middle/tail node
    Node *prev = head;
    Node *current = head->next;
    
    while (current != NULL) {
        if (current->data == target) {
            // Found! Re-link and delete
            prev->next = current->next;     // Skip current
            delete current;                 // Free current
            current = NULL;
            return;
        }
        
        // Move forward
        prev = current;
        current = current->next;
    }
    
    cout << "Value not found!" << endl;
}
```

**Pointer Re-linking**:
```
Before: prev → [10] → current[20] → [30] → NULL
                       (target = 20)

1. Check if current->data == target?  YES
2. Re-link: prev->next = current->next
   (Now: prev -> [10] -> [30] -> NULL)
3. Delete: delete current

After:  [10] → [30] → NULL
```

### Delete Tail (O(n))

Must traverse to find the second-to-last node:

```cpp
void delete_tail(Node *&head) {
    if (head == NULL) return;
    
    // Single node
    if (head->next == NULL) {
        delete head;
        head = NULL;
        return;
    }
    
    // Multiple nodes: find second-to-last
    Node *current = head;
    while (current->next->next != NULL) {
        current = current->next;
    }
    
    // Now current points to second-to-last node
    delete current->next;       // Delete last
    current->next = NULL;       // Point to nothing
}
```

---

## Insert at Tail (O(n))

Must traverse to find the tail:

```cpp
void insert_tail(Node *&head, int value) {
    Node *new_node = new Node();
    new_node->data = value;
    new_node->next = NULL;
    
    if (head == NULL) {
        head = new_node;
        return;
    }
    
    Node *current = head;
    while (current->next != NULL) {
        current = current->next;
    }
    
    current->next = new_node;
}
```

---

## Insert at Position (O(n))

```cpp
void insert_at_position(Node *&head, int value, int position) {
    Node *new_node = new Node();
    new_node->data = value;
    
    // Position 0: insert at head
    if (position == 0) {
        new_node->next = head;
        head = new_node;
        return;
    }
    
    // Find node at position-1
    Node *current = head;
    for (int i = 0; i < position - 1 && current != NULL; i++) {
        current = current->next;
    }
    
    if (current == NULL) {
        cout << "Invalid position!" << endl;
        delete new_node;
        return;
    }
    
    new_node->next = current->next;
    current->next = new_node;
}
```

---

## Complete List Class (CRUD Operations)

```cpp
#include <iostream>
using namespace std;

struct Node {
    int data;
    Node *next;
};

class LinkedList {
private:
    Node *head;
    
public:
    LinkedList() : head(NULL) { }
    
    ~LinkedList() {
        while (head != NULL) {
            delete_head();
        }
    }
    
    void insert_head(int value) {
        Node *new_node = new Node();
        new_node->data = value;
        new_node->next = head;
        head = new_node;
    }
    
    void insert_tail(int value) {
        Node *new_node = new Node();
        new_node->data = value;
        new_node->next = NULL;
        
        if (head == NULL) {
            head = new_node;
            return;
        }
        
        Node *current = head;
        while (current->next != NULL) {
            current = current->next;
        }
        current->next = new_node;
    }
    
    void delete_head() {
        if (head == NULL) return;
        Node *temp = head;
        head = head->next;
        delete temp;
    }
    
    void delete_value(int target) {
        if (head == NULL) return;
        
        if (head->data == target) {
            delete_head();
            return;
        }
        
        Node *prev = head;
        Node *current = head->next;
        
        while (current != NULL) {
            if (current->data == target) {
                prev->next = current->next;
                delete current;
                return;
            }
            prev = current;
            current = current->next;
        }
    }
    
    void display() {
        cout << "List: ";
        Node *current = head;
        while (current != NULL) {
            cout << current->data << " → ";
            current = current->next;
        }
        cout << "NULL\n";
    }
};

int main() {
    LinkedList list;
    
    list.insert_tail(10);
    list.insert_tail(20);
    list.insert_tail(30);
    list.display();              // List: 10 → 20 → 30 → NULL
    
    list.delete_value(20);
    list.display();              // List: 10 → 30 → NULL
    
    list.insert_head(5);
    list.display();              // List: 5 → 10 → 30 → NULL
    
    return 0;
}
```

---

## Edge Cases to Handle

| Case | Issue | Fix |
|------|-------|-----|
| **Empty list** | Operations on NULL | Check `if (head == NULL)` |
| **Single node delete** | Head becomes NULL | Handle specially |
| **Value not found** | Loop completes without match | Set flag, return error |
| **Invalid position** | Index out of bounds | Validate position |

---

## Memory Management Pattern

```cpp
// CRITICAL: Every new needs a delete

~LinkedList() {
    Node *current = head;
    while (current != NULL) {
        Node *temp = current;
        current = current->next;
        delete temp;                // Free each node
    }
}
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| **Insert head** | O(1) | O(1) |
| **Insert tail** | O(n) | O(1) |
| **Insert position** | O(n) | O(1) |
| **Delete head** | O(1) | O(1) |
| **Delete value** | O(n) | O(1) |
| **Delete all** | O(n) | O(1) |
| **Search** | O(n) | O(1) |

---

## Mastery Checklist

- [ ] Delete the head node correctly
- [ ] Delete a node by value using prev/current pattern
- [ ] Delete the tail node
- [ ] Insert at tail (requires traversal)
- [ ] Insert at specific position
- [ ] Handle all edge cases (empty, single node, not found)
- [ ] Implement destructor to prevent memory leaks
- [ ] Trace pointer movements during re-linking
- [ ] Understand the prev/current two-pointer pattern
