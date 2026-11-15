# DSA01 - Assignment 3: Constructors, Destructors and Operator Overloading

**Course:** Object Oriented Programming with C++ / Data Structures and Algorithms (CO3)  
**Student Name:** Nirmal S  
**Register Number:** 192511249  
**Nature of Assignment:** Individual  
**Total Marks:** 50 (Bloom's Level: K5 – Evaluate)  

---

## 1. System and Compiler Information

- **Operating System:** Linux (Ubuntu 24.04 LTS x86_64 via WSL)
- **Compiler:** `g++` (Ubuntu 15.2.0-16ubuntu1) 15.2.0
- **C++ Standard:** C++17 (`-std=c++17`)
- **Sanitizers & Flags:** `-std=c++17 -Wall -Wextra -fsanitize=address`

---

## 2. Directory Structure

```text
DSA01-192511249-Assignment3/
├── .gitignore
├── README.md
├── REPORT.md
├── VIVA_NOTES.md
├── Task01/
│   ├── task01.cpp
│   └── output.txt
├── Task02/
│   ├── task02.cpp
│   └── output.txt
├── Task03/
│   ├── task03.cpp
│   └── output.txt
├── Task04/
│   ├── task04.cpp
│   └── output.txt
├── Task05/
│   ├── task05.cpp
│   └── output.txt
├── Task06/
│   ├── task06.cpp
│   └── output.txt
└── Task07/
    ├── task07.cpp
    └── output.txt
```

---

## 3. Compilation and Execution Commands

Run the following commands from repository root:

### Task 01: Constructors and Constructor Overloading
```bash
g++ -std=c++17 -Wall -Wextra Task01/task01.cpp -o Task01/task01
./Task01/task01
```

### Task 02: Copy Constructors: Shallow and Deep Copies
```bash
g++ -std=c++17 -fsanitize=address -Wall -Wextra Task02/task02.cpp -o Task02/task02
./Task02/task02
```

### Task 03: Dynamic Constructors and Destructors
```bash
g++ -std=c++17 -fsanitize=address -Wall -Wextra Task03/task03.cpp -o Task03/task03
./Task03/task03
```

### Task 04: Overloading Arithmetic and Stream Operators
```bash
g++ -std=c++17 -Wall -Wextra Task04/task04.cpp -o Task04/task04
./Task04/task04
```

### Task 05: Overloading Increment, Comparison and Subscript Operators
```bash
g++ -std=c++17 -Wall -Wextra Task05/task05.cpp -o Task05/task05
./Task05/task05
```

### Task 06: The Rules of Operator Overloading
```bash
g++ -std=c++17 -fsanitize=address -Wall -Wextra Task06/task06.cpp -o Task06/task06
./Task06/task06
```

### Task 07: Design Challenge - AC Circuit Impedance Calculator
```bash
g++ -std=c++17 -Wall -Wextra Task07/task07.cpp -o Task07/task07
./Task07/task07
```

---

## 4. Summary of Tasks and Key Findings

| Task | Topic | Key Findings & Evaluation |
| :--- | :--- | :--- |
| **Task 01** | Constructor Overloading | Used constructor delegation and member initializer lists for 3 book creation scenarios; validated non-negative inventory copies. |
| **Task 02** | Deep Copies & Rule of Three | Implemented copy constructor, copy assignment operator, and destructor for dynamic `Prescription` array; verified zero leaks under AddressSanitizer. |
| **Task 03** | Dynamic Construction & RAII | Managed 2D dynamic array for `Auditorium` seat maps; confirmed strict reverse destruction ordering and stack/heap lifecycle. |
| **Task 04** | Arithmetic & Stream Operators | Built `Money` class handling integer paise arithmetic, carrying paise into rupees, and stream insertion (`<<`). |
| **Task 05** | Increment, Comparison, Subscript | Overloaded prefix vs postfix `++` (benchmarking the copy penalty of postfix), bounds-checked `operator[]`, and clock roll-overs. |
| **Task 06** | Rules of Operator Overloading | Built dynamic `Text` class for SMS gateway, analyzed un-overloadable operators (`::`, `.`, `.*`, `?:`, `sizeof`), and verified self-assignment safety. |
| **Task 07** | Design Challenge | Developed full `Complex` number arithmetic from first principles for series/parallel RLC circuits with safe zero-impedance division handling. |
