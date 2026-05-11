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

# 07 - File Handling: Text and Binary I/O

> [!IMPORTANT]
> **Persistent Storage**: Files allow programs to store data permanently. Text mode suits human-readable data (strings, numbers). Binary mode suits structured data (images, objects). Proper error handling is **mandatory** in production code.

---

## Stream Hierarchy

```
     istream (input)
        ↓
    ifstream (file input)
    
     ostream (output)
        ↓
    ofstream (file output)
    
    iostream (input + output)
        ↓
    fstream (file input + output)
```

---

## Text File Operations

### Writing Text

```cpp
#include <fstream>
using namespace std;

ofstream outfile("data.txt");   // Open for writing

if (!outfile) {
    cout << "Error opening file!" << endl;
    return 1;
}

outfile << "Hello World\n";
outfile << "Line 2\n";
outfile << "Score: " << 95 << "\n";

outfile.close();                // Close file
```

### Reading Text

```cpp
ifstream infile("data.txt");

if (!infile) {
    cout << "Error opening file!" << endl;
    return 1;
}

string line;
while (getline(infile, line)) {     // Read line by line
    cout << line << endl;
}

infile.close();
```

### Line vs. Word Reading

```cpp
// Read entire line (includes spaces)
string line;
getline(infile, line);              // "Hello World"

// Read single word (stops at whitespace)
string word;
infile >> word;                     // "Hello"
```

---

## Binary File Operations

### Writing Binary

```cpp
struct Student {
    int id;
    char name[50];
    float gpa;
};

ofstream outfile("students.dat", ios::binary);

Student s = {101, "Ali", 3.8};
outfile.write(reinterpret_cast<char*>(&s), sizeof(Student));

outfile.close();
```

### Reading Binary

```cpp
ifstream infile("students.dat", ios::binary);

Student s;
infile.read(reinterpret_cast<char*>(&s), sizeof(Student));

cout << "ID: " << s.id << "\n";
cout << "Name: " << s.name << "\n";
cout << "GPA: " << s.gpa << "\n";

infile.close();
```

---

## File Modes

| Mode | Purpose | Behavior |
|------|---------|----------|
| `ios::in` | Read | File must exist |
| `ios::out` | Write | Creates or overwrites |
| `ios::app` | Append | Adds to end |
| `ios::binary` | Binary | No text conversions |
| `in \| binary` | Read binary | Binary input |
| `out \| binary` | Write binary | Binary output |

```cpp
// Append to existing file
ofstream outfile("data.txt", ios::app);
outfile << "New line at end\n";

// Read binary
ifstream infile("data.bin", ios::binary);
```

---

## Complete Example: Student Database

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <vector>
using namespace std;

struct Student {
    int id;
    string name;
    float gpa;
};

// Write all students to file
void save_students(vector<Student> &students, string filename) {
    ofstream outfile(filename);
    
    if (!outfile) {
        cout << "Error opening file for writing!" << endl;
        return;
    }
    
    for (auto &s : students) {
        outfile << s.id << "|" << s.name << "|" << s.gpa << "\n";
    }
    
    outfile.close();
    cout << "Saved " << students.size() << " students\n";
}

// Read all students from file
vector<Student> load_students(string filename) {
    vector<Student> students;
    ifstream infile(filename);
    
    if (!infile) {
        cout << "Error opening file for reading!" << endl;
        return students;
    }
    
    int id;
    string name;
    float gpa;
    char separator;
    
    while (infile >> id >> separator >> name >> separator >> gpa) {
        students.push_back({id, name, gpa});
    }
    
    infile.close();
    cout << "Loaded " << students.size() << " students\n";
    return students;
}

int main() {
    vector<Student> students;
    
    // Create sample data
    students.push_back({101, "Ali", 3.8});
    students.push_back({102, "Sara", 3.9});
    students.push_back({103, "Usman", 3.5});
    
    // Save to file
    save_students(students, "students.txt");
    
    // Load from file
    vector<Student> loaded = load_students("students.txt");
    
    // Display
    for (auto &s : loaded) {
        cout << "ID: " << s.id << ", Name: " << s.name 
             << ", GPA: " << s.gpa << "\n";
    }
    
    return 0;
}
```

---

## Error Handling Checklist

```cpp
ifstream infile("data.txt");

// Check if file opened successfully
if (!infile) {
    cerr << "Error: Cannot open file!" << endl;
    return 1;
}

// Check for read errors
if (infile.fail()) {
    cerr << "Error: Read failed!" << endl;
    infile.close();
    return 1;
}

// Better: read until failure
string line;
while (getline(infile, line)) {
    // line contains valid data
}

infile.close();
```

---

## Text vs. Binary: When to Use

| Aspect | Text | Binary |
|--------|------|--------|
| **Readability** | Human-readable | Not readable |
| **Size** | Larger (strings) | Smaller (compact) |
| **Parsing** | Easy (delimited) | Complex (byte-level) |
| **Compatibility** | Cross-platform | Platform-specific |
| **Use case** | Config, CSV, JSON | Images, executable, database |

```cpp
// TEXT: Easy parsing, larger size
// "123|Ali|3.8\n"

// BINARY: Compact, efficient, not readable
// [struct bytes in raw format]
```

---

## File Pointer Navigation

```cpp
fstream file("data.txt", ios::in | ios::out);

file.seekg(0, ios::beg);         // Go to beginning (input)
file.seekp(0, ios::end);         // Go to end (output)

long pos = file.tellg();         // Current position (input)

// Rewind to start
file.clear();
file.seekg(0, ios::beg);
```

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| **Forgot to close** | File stays locked | Use `file.close()` or RAII |
| **No error check** | Silent failure | Check `if (!file)` |
| **Wrong mode** | Can't write to read-only | Use `ios::out` for write |
| **Buffer issues** | Data not written | Use `file.flush()` or `close()` |
| **Wrong separator** | Parsing fails | Match delimiter with writing |

---

## RAII Pattern for Files

```cpp
class FileGuard {
private:
    ifstream file;
public:
    FileGuard(string filename) {
        file.open(filename);
        if (!file) throw runtime_error("Cannot open file");
    }
    
    ~FileGuard() {
        file.close();            // Automatic cleanup
    }
    
    ifstream& get_stream() { return file; }
};

int main() {
    try {
        FileGuard guard("data.txt");
        string line;
        while (getline(guard.get_stream(), line)) {
            cout << line << endl;
        }
    } catch (exception &e) {
        cout << e.what() << endl;
    }
    // File auto-closes when guard destructs
}
```

---

## Mastery Checklist

- [ ] Open files for reading with `ifstream`
- [ ] Open files for writing with `ofstream`
- [ ] Read text line-by-line with `getline()`
- [ ] Write text with `<<` operator
- [ ] Handle file open errors properly
- [ ] Use binary mode for structured data
- [ ] Write and read binary structs
- [ ] Understand text vs. binary formats
- [ ] Close files explicitly or use RAII
- [ ] Parse delimited text data
