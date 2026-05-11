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

# 15 - Queue Implementation: FIFO Data Structure

> [!IMPORTANT]
> **First-In-First-Out**: Queues model **real-world waiting lines**. First element added is first removed. Critical for breadth-first search, task scheduling, and print spooling. Two implementations: array-based (circular) and class-based (linked list).

---

## Queue Fundamentals

### FIFO Behavior

```
Enqueue 1: [1]
Enqueue 2: [1, 2]
Enqueue 3: [1, 2, 3]
Dequeue:   [2, 3]       ← 1 removed (entered first)
Dequeue:   [3]          ← 2 removed
```

### Key Operations

- **Enqueue**: Add element to rear
- **Dequeue**: Remove element from front
- **Front**: Access front element
- **Is Empty**: Check if queue has elements
- **Size**: Count elements

---

## Array-Based Queue

### Naive Implementation (Problematic)

```cpp
class Queue {
private:
    int arr[100];
    int front = 0;          // Index of first element
    int rear = -1;          // Index of last element
    
public:
    void enqueue(int x) {
        rear++;
        arr[rear] = x;
    }
    
    int dequeue() {
        return arr[front++];
    }
};
```

**Problem**: After dequeue, `front` advances but never used space is wasted!

```
Initial:    [_, _, _, _, _]  front=0, rear=-1

Enqueue 1:  [1, _, _, _, _]  front=0, rear=0
Enqueue 2:  [1, 2, _, _, _]  front=0, rear=1
Dequeue:    [1, 2, _, _, _]  front=1, rear=1  (arr[0] wasted!)
Enqueue 3:  [1, 2, 3, _, _]  front=1, rear=2  (arr[0] still wasted)
```

### Circular Queue (Efficient)

Use modular arithmetic to wrap around:

```cpp
class CircularQueue {
private:
    int arr[100];
    int front = 0;
    int rear = -1;
    int count = 0;              // Track how many elements
    static const int MAXSIZE = 100;
    
public:
    void enqueue(int x) {
        if (count == MAXSIZE) {
            cout << "Queue full!" << endl;
            return;
        }
        rear = (rear + 1) % MAXSIZE;    // Wrap around
        arr[rear] = x;
        count++;
    }
    
    int dequeue() {
        if (count == 0) {
            cout << "Queue empty!" << endl;
            return -1;
        }
        int value = arr[front];
        front = (front + 1) % MAXSIZE;   // Wrap around
        count--;
        return value;
    }
    
    bool is_empty() {
        return count == 0;
    }
    
    int size() {
        return count;
    }
    
    int peek() {
        if (count == 0) return -1;
        return arr[front];
    }
};
```

**Memory Reuse**:
```
After wrap: front→[X, 2, 3, 1, _]←rear  (circular structure)
                   ^         ^
               front wraps  rear wraps
```

---

## Class-Based Queue (Linked List)

### Node Structure

```cpp
struct Node {
    int data;
    Node *next;
};

class Queue {
private:
    Node *front;
    Node *rear;
    int count;
    
public:
    Queue() : front(NULL), rear(NULL), count(0) { }
    
    ~Queue() {
        while (!is_empty()) {
            dequeue();
        }
    }
    
    void enqueue(int x) {
        Node *new_node = new Node();
        new_node->data = x;
        new_node->next = NULL;
        
        if (front == NULL) {            // First element
            front = rear = new_node;
        } else {
            rear->next = new_node;      // Add at rear
            rear = new_node;
        }
        count++;
    }
    
    int dequeue() {
        if (front == NULL) {
            cout << "Queue empty!" << endl;
            return -1;
        }
        
        int value = front->data;
        Node *temp = front;
        front = front->next;
        
        if (front == NULL) {
            rear = NULL;                // Queue now empty
        }
        
        delete temp;
        count--;
        return value;
    }
    
    int peek() {
        return front ? front->data : -1;
    }
    
    bool is_empty() {
        return count == 0;
    }
    
    int size() {
        return count;
    }
};
```

---

## Complete Example: Print Queue Simulation

```cpp
#include <iostream>
using namespace std;

struct PrintJob {
    int job_id;
    string filename;
    int priority;
};

class PrintQueue {
private:
    queue<PrintJob> jobs;
    
public:
    void add_job(PrintJob job) {
        jobs.push(job);
        cout << "Job " << job.job_id << " added\n";
    }
    
    void print_next() {
        if (jobs.empty()) {
            cout << "No jobs!\n";
            return;
        }
        
        PrintJob job = jobs.front();
        jobs.pop();
        cout << "Printing: " << job.filename << " (Job " 
             << job.job_id << ")\n";
    }
    
    void show_queue() {
        cout << "Queue size: " << jobs.size() << "\n";
    }
};

int main() {
    PrintQueue pq;
    
    pq.add_job({1, "document.pdf", 1});
    pq.add_job({2, "image.jpg", 2});
    pq.add_job({3, "report.docx", 1});
    
    pq.show_queue();        // Queue size: 3
    
    pq.print_next();        // Printing: document.pdf (Job 1)
    pq.print_next();        // Printing: image.jpg (Job 2)
    pq.show_queue();        // Queue size: 1
    
    return 0;
}
```

---

## Array vs. Linked List Queue

| Aspect | Array | Linked List |
|--------|-------|------------|
| **Memory** | Fixed | Dynamic |
| **Circular logic** | Needed | Not needed |
| **Overflow** | Possible | No (unless out of heap) |
| **Performance** | O(1) enqueue/dequeue | O(1) enqueue/dequeue |
| **Space** | Wasteful if not circular | Overhead per node |
| **Best for** | Fixed, known size | Dynamic size |

---

## STL Queue

```cpp
#include <queue>

queue<int> q;

q.push(1);              // Enqueue
q.push(2);
q.push(3);

cout << q.front() << endl;  // 1
cout << q.back() << endl;   // 3

q.pop();                // Dequeue

cout << q.size() << endl;   // 2
cout << q.empty() << endl;  // false
```

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| **Confused FIFO** | Added to front instead of rear | FIFO = add rear, remove front |
| **Circular index confusion** | Wrong modulo application | Use `(index + 1) % SIZE` |
| **Memory leak** | Forgot `delete` in dequeue | Always delete in destructor |
| **NULL check** | Dereferencing NULL | Check `front != NULL` |
| **Off-by-one** | Circular buffer not wrapping | Test with wraparound cases |

---

## Mastery Checklist

- [ ] Understand FIFO principle
- [ ] Implement circular queue with modulo arithmetic
- [ ] Implement linked-list-based queue
- [ ] Enqueue and dequeue operations work correctly
- [ ] Handle edge cases (empty, full, wraparound)
- [ ] Implement peek and size operations
- [ ] Use STL `std::queue<T>`
- [ ] Understand when array vs. linked list is better
- [ ] Properly manage memory (new/delete)
- [ ] Test queue operations in realistic scenarios
