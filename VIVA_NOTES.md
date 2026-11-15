# VIVA & CONCEPT GUIDE: CONSTRUCTORS AND OPERATOR OVERLOADING
**Student:** Nirmal S (Reg No: 192511249)  
**Perspective:** Python Master to C++ Beginner Bridge  
**Goal:** Crack the viva and confidently explain/modify any part of Assignment 3 on the spot.

---

## 1. PYTHON VS C++ "CHEAT SHEET" (YOUR MENTAL BRIDGE)

| Concept | Python Mental Model | C++ Reality in Assignment 3 | Why Faculty Cares |
| :--- | :--- | :--- | :--- |
| **Constructors** | Single `__init__(self, ...)` with default parameters. | Multiple overloaded constructors (`Book()`, `Book(title)`, `Book(title, author, copies)`). | Task 01: Constructor delegation (`: Book(...)`) prevents duplicate logic. |
| **Member Initializer List** | `self.title = t` executed inside `__init__`. | `: title(t), author(a)` runs *before* constructor body. Initializes members in declaration order. | Eliminates double-construction. Mandatory for `const` and reference members. |
| **Copy Semantics** | `b = a` creates an alias. `copy.deepcopy()` copies everything. | `b = a` calls copy constructor. If default memberwise copy is used on raw pointers, it's a **shallow copy** causing double-free crashes! | Core of Task 02: Why the Rule of Three is critical. |
| **Rule of Three** | Garbage collector cleans up memory automatically. | If a class manages raw resources, you MUST define all three: **Destructor, Copy Constructor, Copy Assignment Operator**. | Task 02: Hospital prescription memory safety. |
| **Operator Overloading** | Special dunder methods: `__add__`, `__eq__`, `__getitem__`. | `operator+`, `operator==`, `operator[]`. Must follow C++ arity and precedence rules. | Tasks 04, 05, 06. |
| **Prefix vs Postfix ++** | Python doesn't have `++i` (evaluates as positive `+(+i)`). | `++i` modifies and returns reference (`T&`). `i++` makes a copy, modifies, and returns copy (`T`). | Task 05: Why `++i` is faster than `i++`. |
| **Stream Insertion `<<`** | `__str__` or `__repr__` returns string. | Non-member friend function taking `std::ostream&` as left operand and returning `std::ostream&`. | Task 04 & 07: Displaying `Money` and `Complex` objects. |

---

## 2. TASK-BY-TASK VIVA CHEAT SHEET

### TASK 01: Constructor Overloading (`Task01/task01.cpp`)
- **What it does:** Library book record constructor delegation and negative copy validation.
- **Viva Answer:** *"Sir, constructor delegation allows one constructor to invoke another via the member initializer list, centralizing validation. Member initializer lists are preferred because they avoid redundant default-construction and assignment."*
- **Trap Question:** *"In what order are members initialized?"*  
  $\rightarrow$ *"In the exact order they are declared in the class definition, regardless of the order in the initializer list. Listing them out of order triggers `-Wreorder`."*

---

### TASK 02: Copy Constructors & Rule of Three (`Task02/task02.cpp`)
- **What it does:** Deep copy constructor, copy assignment operator, and destructor for dynamic `Prescription` array.
- **Viva Answer:** *"A shallow copy copies only pointer addresses. When both destructors run, they delete the same memory address, causing a double-free crash. A deep copy allocates a separate heap array and copies each element individually."*
- **Self-Assignment Guard:** *"Why do we check `if (this != &other)` in `operator=`?"*  
  $\rightarrow$ *"If an object assigns to itself (`p = p`) without this check, deleting the existing array destroys the source data before it can be copied!"*

---

### TASK 03: Dynamic Constructors & RAII (`Task03/task03.cpp`)
- **What it does:** Manages dynamic 2D seat map with `new` and `delete[]`. Verifies `Logger` output prediction `+A +B -B copyAc use -Ac +P -P end -A `.
- **Viva Answer:** *"Dynamic constructors allocate heap memory during object initialization, while the destructor guarantees automatic deallocation when the object leaves scope, embodying RAII (Resource Acquisition Is Initialization)."*

---

### TASK 04: Money Operator Overloading (`Task04/task04.cpp`)
- **What it does:** Overloads `+`, `-`, `+=`, `==`, `<`, `<<` for currency stored in 64-bit paise.
- **Viva Answer:** *"We store money strictly in integer paise to eliminate floating-point precision inaccuracies. `operator<<` must be a friend function because the left-hand operand is `std::ostream&`, not `Money`."*

---

### TASK 05: Increment, Comparison, Subscript (`Task05/task05.cpp`)
- **What it does:** Prefix/postfix `++` for `TrackTime` with roll-over, bounds-checked `operator[]` for `Playlist`.
- **Viva Answer:** *"The compiler distinguishes prefix from postfix using an unused dummy `int` parameter (`operator++(int)`). Postfix must create and return a copy of the old state, making it slower than prefix increment."*

---

### TASK 06: Rules of Operator Overloading (`Task06/task06.cpp`)
- **What it does:** SMS `Text` class with dynamic buffer concatenation.
- **Viva Answer:** *"Operators `::`, `.`, `.*`, `?:`, and `sizeof` cannot be overloaded in C++ to preserve fundamental language syntax and type safety. Operator precedence and arity cannot be changed, and at least one operand must be a user-defined type."*

---

### TASK 07: AC Circuit Complex Number Calculator (`Task07/task07.cpp`)
- **What it does:** AC circuit calculator computing series and parallel RLC impedances with safe zero division.
- **Viva Answer:** *"Impedance is represented as a complex number $R + jX$. Series impedances add directly ($Z_1 + Z_2$), while parallel branches combine as $(Z_1 \times Z_2) / (Z_1 + Z_2)$. We implemented robust division handling to safeguard against division by zero impedance."*
