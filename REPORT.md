# ASSIGNMENT 3 REPORT: CONSTRUCTORS, DESTRUCTORS AND OPERATOR OVERLOADING

**Course:** Object Oriented Programming with C++ (DSA01 - CO3)  
**Assignment Title:** Construct C++ Programs Using Constructors, Destructors and Operator Overloading  
**Student Name:** Nirmal S  
**Register Number:** 192511249  
**Nature of Assignment:** Individual  
**Total Marks:** 50  
**GitHub Repository Link:** https://github.com/nirmal2308/DSA01-192511249-Assignment3  
**Bloom’s Level Assessed:** K5 – Evaluate  
**Compiler Used:** `g++ (Ubuntu 15.2.0-16ubuntu1) 15.2.0`  
**Standard & Compilation Flags:** `g++ -std=c++17 -Wall -Wextra -fsanitize=address`  

---

## EXECUTIVE SUMMARY

Assignment 3 provides a comprehensive, rigorous evaluation of object lifecycle management and syntactic operator extensibility in modern C++. Through 7 targeted engineering tasks, we investigate:
1. Object creation mechanics (default, parameterised, overloaded, and delegating constructors with member initializer lists).
2. Deep copy semantics vs shallow copy pitfalls under the **Rule of Three**.
3. Dynamic multidimensional resource management and deterministic **RAII** cleanup.
4. Value-class arithmetic encapsulation (`Money` class with stream injection/extraction).
5. Unary increment semantics (prefix reference vs postfix copy) and checked subscript indexing.
6. Foundational language rules governing operator overloading.
7. Architectural synthesis in a full Complex-Number class engineered from first principles for AC circuit impedance calculations.

All implementations were compiled with `-fsanitize=address` to verify zero memory leaks and absence of double-deletion faults.

---

## Q1 – CONSTRUCTORS AND CONSTRUCTOR OVERLOADING (6 MARKS)

### 1.1 Problem Context – Library Book Records
A library catalogs items across three scenarios:
- A new catalog entry with full bibliographic data (`title`, `author`, `copies`).
- A donated volume with only a `title` (author default: "Anonymous Donor", copies: 1).
- An uninitialized placeholder record (`title` = "Untitled", `author` = "None", `copies` = 0).
Legacy records contained corrupted negative copies.

### 1.2 Implementation & Constructor Delegation
Implemented in `Task01/task01.cpp` using modern constructor delegation (C++11):
```cpp
// Target parameterised constructor
Book(const std::string& t, const std::string& a, int c)
    : title(t), author(a), copies(c < 0 ? 0 : c) {}

// Delegating constructor for donated book
explicit Book(const std::string& t) : Book(t, "Anonymous Donor", 1) {}

// Delegating default constructor
Book() : Book("Untitled", "None", 0) {}
```
- **Constructor Delegation Advantage:** Centralizes domain invariant validation in exactly one constructor body, preventing duplicate validation code.

### 1.3 Design & Analysis
- **When Compiler Supplies a Default Constructor:** The compiler generates a synthetic default constructor *only* if the class defines no user-declared constructors. The moment any parameterised constructor is introduced, the compiler suppresses the automatic default constructor.
- **Why Member Initializer Lists Are Preferred:** 
  - Assignment inside the constructor body (`{ member = val; }`) forces a two-step process: the member is first default-constructed, and then assigned a new value via `operator=`.
  - Member initializer lists directly invoke the member's copy/move constructor, eliminating redundant default construction overhead.
  - Constant (`const`) and reference (`&`) members **must** be initialized via member initializer lists, as they cannot be reassigned in the body.
- **Member Initialization Order:** Members are initialized strictly in the **order of their declaration inside the class**, completely independent of the order written in the constructor initializer list. Listing them out of order prompts compiler warning: `-Wreorder`.
- **The `explicit` Keyword:** Prevents unexpected implicit type conversions. Marking `explicit Book(const std::string& t)` prevents accidental assignments like `Book b = std::string("C++ Guide");` from compiling.

### 1.4 Alternative Concept & Complexity
- If a single constructor with default arguments were used: `Book(std::string t = "Untitled", std::string a = "None", int c = 0)`.
- Time and space are $O(1)$ in both designs. However, overloaded delegating constructors provide clearer API intent and separate donor donations from placeholder allocations without allowing bizarre combinations (e.g., providing title and copies but omitting author).

---

## Q2 – COPY CONSTRUCTORS: SHALLOW AND DEEP COPIES (7 MARKS)

### 2.1 Problem Context – Hospital Prescription Records
A `Prescription` holds a dynamically allocated array `std::string* medicines`. When copied naively via compiler-generated memberwise copy:
1. Both instances point to the same heap address (shallow copy).
2. Changes to prescription $B$ alter prescription $A$.
3. At block termination, both destructors invoke `delete[]` on the identical pointer, triggering a catastrophic **double-free runtime abort**.

### 2.2 Deep Copy Implementation & The Rule of Three
Implemented in `Task02/task02.cpp`:
1. **Copy Constructor:** Allocates a separate heap buffer `new std::string[other.n]` and copies elements individually.
2. **Copy Assignment Operator (`operator=`):** Guards against self-assignment (`if (this != &other)`), allocates replacement buffer, copies elements, deallocates old buffer, updates pointer, and returns `*this`.
3. **Destructor:** Releases dynamic array via `delete[] medicines`.

### 2.3 Verification & Expected Output
Executing `Task02/task02.cpp` under `-fsanitize=address`:
```text
Ibuprofen ORS | Paracetamol Cetirizine | Paracetamol ORS
```
- Modifying prescription $p$ to `"Ibuprofen"` did not affect clone $q$ (which has `"Paracetamol"`).
- Self-assignment `p = p` executed safely with zero buffer corruption.
- Pass-by-value invoked the deep copy constructor, whereas pass-by-const-reference copied 0 heap elements.
- AddressSanitizer verified: **0 byte leaks, 0 heap use-after-free, 0 double frees**.

### 2.4 Alternative Concept: Rule of Zero with `std::vector`
If `std::string* medicines` is replaced by `std::vector<std::string>`:
- The class adheres to the **Rule of Zero**: no custom copy constructor, assignment operator, or destructor is required. The compiler-generated defaults correctly deep-copy the vector automatically.
- Time to copy $n$ medicines is $O(n)$ in both designs, but `std::vector` eliminates manual memory management bugs.

---

## Q3 – DYNAMIC CONSTRUCTORS AND DESTRUCTORS (7 MARKS)

### 3.1 Problem Context – Auditorium Seat Booking
An auditorium seat map dimensions ($R \times C$) vary per event. Memory must be allocated dynamically when an event opens and released cleanly on closure.

### 3.2 Dynamic 2D Array Lifecycle
Allocated via an array of row pointers:
```cpp
seatMap = new char*[rows];
for (int i = 0; i < rows; ++i) seatMap[i] = new char[cols];
```
Released via strict inverse deallocation:
```cpp
for (int i = 0; i < rows; ++i) delete[] seatMap[i];
delete[] seatMap;
```

### 3.3 Predict the Output: Logger Lifecycle Analysis
```cpp
struct Logger {
    std::string n;
    Logger(const std::string& s) : n(s) { std::cout << '+' << n << ' '; }
    Logger(const Logger& o) : n(o.n + "c") { std::cout << "copy" << n << ' '; }
    ~Logger() { std::cout << '-' << n << ' '; }
};
void use(Logger) { std::cout << "use "; }
```

#### Prediction vs Actual:
- **Predicted Output:** `+A +B -B copyAc use -Ac +P -P end -A `
- **Actual Output:** `+A +B -B copyAc use -Ac +P -P end -A `
- **Evaluation:**
  - `a` constructed: `+A `
  - Scoped block creates and immediately destroys `b`: `+B -B `
  - `use(a)` receives copy: `copyAc `, executes body `use `, then temporary is destroyed at end of expression: `-Ac `
  - Dynamic heap allocation: `+P `, explicit `delete p`: `-P `
  - Output `end `
  - At program termination, `a` is destroyed: `-A `

### 3.4 Why Destructors Cannot Take Arguments or Be Overloaded
Destructors are invoked implicitly by the runtime during automatic stack unwinding, block exit, or heap deletion. Because the programmer does not write the invocation call, no arguments can ever be passed. Hence, overloading destructors is syntactically impossible and prohibited.

---

## Q4 – OVERLOADING ARITHMETIC AND STREAM OPERATORS (7 MARKS)

### 4.1 Problem Context – Money Arithmetic for Billing Counter
Direct floating-point money representations cause precision errors. The `Money` class stores currency strictly in 64-bit integer `paise`.

### 4.2 Member vs Friend Operator Decisions
- **`operator+` as Member:** Declared `Money operator+(const Money& other) const;`. Evaluated as `a.operator+(b)`.
- **`operator+=` as Member:** Modifies left operand in place and returns `Money&` (lvalue reference to `*this`).
- **`operator<<` as Friend:** The left operand of `std::cout << m` is an instance of `std::ostream&`, not `Money`. Member operators require the left operand to be of the host class. Therefore, stream insertion **must** be implemented as a non-member friend function:
  ```cpp
  friend std::ostream& operator<<(std::ostream& os, const Money& m);
  ```

### 4.3 Output Verification
Executing `Task04/task04.cpp`:
```text
Rs 200.00
true
```
- Paise overflow into rupees ($75\text{p} + 50\text{p} = \text{Rs } 1.25$) verified.
- Subtraction yielding negative values handled cleanly: $\text{Rs } 50.00 - \text{Rs } 75.50 = -\text{Rs } 25.50$.

### 4.4 Engineering Decision: Money Multiplication
- $\text{Money} \times \text{Money}$ is **dimensionally meaningless** (units of $\text{rupees}^2$). It must be forbidden.
- $\text{Money} \times \text{double}$ or $\text{Money} \times \text{int}$ represents quantity or tax rate scaling and is semantically valid.

---

## Q5 – OVERLOADING INCREMENT, COMPARISON AND SUBSCRIPT OPERATORS (6 MARKS)

### 5.1 Prefix vs Postfix Increment Mechanics
- **Prefix (`++t`):** Increments the state and returns an lvalue reference to `*this` (`TrackTime&`). No temporary copy is created.
- **Postfix (`t++`):** Captures old state into a local copy, increments `*this`, and returns the old copy by value (`TrackTime`). Takes an unused dummy `int` parameter to allow compiler overload disambiguation.

### 5.2 Predict the Output: Counter Verification
- Prediction: `5 7 7`
- Actual Output: `5 7 7`
- *Explanation:* `c` is 5. `c++` returns old value (5) into `a`, incrementing `c` to 6. `++c` increments `c` to 7 and returns 7 into `b`.

### 5.3 Benchmarking Prefix vs Postfix (10,000,000 Iterations)
From `Task05/task05.cpp`:
- **Prefix (`++c`):** `23 ms`
- **Postfix (`c++`):** `33 ms`
- **Evaluation:** Postfix increment suffers a **$43\%$ execution penalty** due to creating, copying, and returning redundant temporary objects on each iteration.

### 5.4 Subscript Operator `operator[]` Constness
Two overloads are provided for `Playlist`:
1. `std::string& operator[](int index)`: Allows reading and in-place updating of tracks (`pl[0] = "New Song"`).
2. `const std::string& operator[](int index) const`: Enables safe inspection of constant playlists without allowing element mutation.

---

## Q6 – THE RULES OF OPERATOR OVERLOADING (7 MARKS)

### 6.1 SMS Gateway Text Class Implementation
Built dynamic string class in `Task06/task06.cpp` supporting concatenation (`+`), equality (`==`), inequality (`!=`), and self-assignment protection.

### 6.2 Immutable Rules of C++ Operator Overloading
1. **Operators that CANNOT be Overloaded:**
   - `::` (Scope resolution)
   - `.` (Direct member access)
   - `.*` (Pointer-to-member dereference)
   - `?:` (Ternary conditional)
   - `sizeof` (Type size query)
   - *Reason:* Allowing overloading of these fundamental operators would destabilize basic compiler type inference, pointer binding, and memory layout guarantees.
2. **Precedence and Arity are Fixed:** Overloading `+` cannot change its binary nature or make its precedence higher than `*`.
3. **User-Defined Type Requirement:** At least one argument must be a user-defined class/struct/enum. You cannot overload `+` for two primitive `int`s.
4. **Member-Only Operators:** `operator=`, `operator[]`, `operator()`, and `operator->` must be non-static member functions.

---

## Q7 – DESIGN CHALLENGE: AC CIRCUIT IMPEDANCE CALCULATOR (10 MARKS)

### Stage 1 – Problem Interpretation & Requirements
In AC circuit analysis:
- Resistor impedance: $Z_R = R + 0j$
- Inductor impedance: $Z_L = 0 + j\omega L$
- Capacitor impedance: $Z_C = 0 - j\frac{1}{\omega C}$
- Series combination: $Z_{series} = Z_1 + Z_2$
- Parallel combination: $Z_{parallel} = \frac{Z_1 \times Z_2}{Z_1 + Z_2}$

### Stage 2 – Design & Mathematical Formulation
- Class `Complex` with attributes `double real`, `double imag`.
- Multiplication formula:
  $$(a + bj)(c + dj) = (ac - bd) + j(ad + bc)$$
- Division formula:
  $$\frac{a + bj}{c + dj} = \frac{(ac + bd) + j(bc - ad)}{c^2 + d^2}$$
- Safe zero-denominator check: if $c^2 + d^2 == 0$, return `NaN` state safely without crash.

### Stage 3 – Implementation & Hand-Calculation Verification
Using circuit parameters $R = 100\,\Omega$, $L = 0.1\,\text{H}$, $C = 10\,\mu\text{F}$ at $\omega = 1000\,\text{rad/s}$:
- $Z_R = 100.00 + 0.00j$
- $Z_L = 0.00 + 100.00j$
- $Z_C = 0.00 - 100.00j$

**Series Circuit Test:**
$$Z_{series} = (100 + 0j) + (0 + 100j) + (0 - 100j) = 100.00 + 0.00j$$
$$|Z_{series}| = 100.00\,\Omega \quad (\text{Exact resonance verified})$$

**Parallel RL Circuit Test:**
$$Z_{parallel} = \frac{(100 + 0j)(0 + 100j)}{100 + 100j} = \frac{10000j}{100 + 100j} = 50.00 + 50.00j$$
$$|Z_{parallel}| = \sqrt{50^2 + 50^2} = 70.71\,\Omega \quad (\text{Exact hand calculation verified})$$

### Stage 4 – Design Defense
- **Comparison with `std::complex<double>`:** Our class provides explicit domain-specific diagnostics (e.g. division-by-zero alerts) and customized engineering stream formatting (`a + bj` notation standard in electrical engineering vs standard `(a, b)` format).
- **Time/Space Complexity:** All arithmetic operations are $O(1)$ time and $O(1)$ space (two 64-bit IEEE-754 doubles = 16 bytes).

---

## CONCLUSION

Assignment 3 demonstrates mastery over C++ resource acquisition, object lifecycle guarantees under RAII, deep copy semantics under the Rule of Three, and operator overloading rules.
