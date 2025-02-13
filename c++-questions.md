# C++ Questions

### C++ Basics

1. **Difference between references and pointers:**
   - **References** are aliases for an existing variable, and they must be initialized when declared. They cannot be null or reassigned to another object.
   - **Pointers** hold the memory address of another variable, can be null, and can be reassigned to point to different objects.

2. **Difference between memory allocation in stack and on heap:**
   - **Stack:** Allocates memory in a last-in, first-out manner. It is fast and automatically managed (memory is freed when the function returns).
   - **Heap:** Allocates memory dynamically at runtime using `new` or `malloc`. The programmer is responsible for freeing heap memory using `delete` or `free`.

3. **What kinds of smart pointers exist?**
   - **`std::unique_ptr`:** A smart pointer that maintains sole ownership of an object. It cannot be copied, only moved.
   - **`std::shared_ptr`:** A smart pointer that shares ownership of an object. The object is destroyed when the last `shared_ptr` referencing it is destroyed.
   - **`std::weak_ptr`:** A non-owning smart pointer that holds a reference to an object managed by `std::shared_ptr`, without affecting its reference count.

4. **How is `unique_ptr` implemented? How is sole ownership enforced?**
   - `unique_ptr` uses move semantics to transfer ownership. Copying is disabled by deleting the copy constructor and assignment operator. The pointer can only be moved, meaning only one `unique_ptr` can own the object at any given time.

5. **How does `shared_ptr` work? How is the reference counter synchronized between objects?**
   - `shared_ptr` maintains a **reference counter** in a control block. Every time a new `shared_ptr` is created from an existing one, the counter is incremented. When a `shared_ptr` is destroyed, the counter is decremented. Once the counter reaches zero, the managed object and control block are both destroyed. Synchronization is typically handled with atomic operations to ensure thread safety.

6. **Can we copy `unique_ptr` or pass it from one object to another?**
   - `unique_ptr` cannot be copied due to its exclusive ownership. However, it can be **moved** using `std::move`, which transfers ownership to another `unique_ptr`.

7. **What are rvalue and lvalue?**
   - **lvalue:** Refers to an object that persists beyond a single expression (has an identifiable location in memory).
   - **rvalue:** Refers to a temporary object that only exists during an expression (e.g., the result of an arithmetic operation or a literal value).

8. **What are `std::move` and `std::forward`?**
   - **`std::move`:** Casts an lvalue to an rvalue, allowing resources to be moved rather than copied.
   - **`std::forward`:** Preserves the value category (lvalue or rvalue) of an argument passed to a function, used primarily in template forwarding.

### OOPs

1. **Ways to access private fields of a class:**
   - **Friend functions or classes**: Can access private members.
   - **Getters and setters**: Provide controlled access.
   - **Reflection (in certain environments)**: Not native to C++ but possible via external libraries.
   
2. **Can a class inherit multiple classes?**
   - Yes, C++ supports **multiple inheritance**, allowing a class to inherit from more than one class.

3. **Is a static field initialized in the class constructor?**
   - No, **static fields** are initialized **outside** of the class, usually in a **global scope** or in the class definition if initialized inline.

4. **Can an exception be thrown in a constructor or destructor? How to prevent that?**
   - Yes, exceptions can be thrown in both constructors and destructors. However, throwing an exception in a destructor is discouraged as it could lead to **terminate()** being called if another exception is already active. To prevent this, exceptions in destructors should be caught and handled internally.

5. **What are virtual methods?**
   - **Virtual methods** are member functions that can be overridden in derived classes. They enable **runtime polymorphism**, where the function that is called depends on the actual object type, not the type of the pointer/reference.

6. **Why do we need a virtual destructor?**
   - A virtual destructor ensures that when an object is deleted through a base class pointer, the **destructor of the derived class** is called, avoiding resource leaks.

7. **Difference between abstract class and interface:**
   - In C++, an **abstract class** is a class with at least one pure virtual function (`virtual void func() = 0;`). C++ does not have a distinct interface concept, but an abstract class with only pure virtual functions can be used as an interface.

8. **Can a constructor be virtual?**
   - No, **constructors cannot be virtual**. This is because constructors are not inherited and are called based on the object's type.

9. **How is the `const` keyword used for class methods?**
   - A method marked with `const` guarantees that it will not modify any member variables of the class (except those marked as `mutable`).

   Example:
   ```cpp
   int getValue() const { return value; }
   ```

10. **How to protect an object from copying?**
    - You can delete the copy constructor and copy assignment operator:
    ```cpp
    MyClass(const MyClass&) = delete;
    MyClass& operator=(const MyClass&) = delete;
    ```

### STL Containers

1. **Difference between vector and list:**
   - **Vector**: A dynamic array that provides random access and contiguous memory allocation. Insertion and deletion at the end are efficient, but inserting or deleting in the middle is costly.
   - **List**: A doubly linked list that provides efficient insertion and deletion at any position but has no random access (must traverse the list sequentially).

2. **Difference between map and unordered_map:**
   - **map**: A balanced binary search tree (usually implemented as a red-black tree) that keeps keys sorted. Operations like insert and lookup have logarithmic time complexity.
   - **unordered_map**: A hash table-based container. It provides average constant-time complexity for insert and lookup, but the keys are not stored in any particular order.

3. **Does calling `push_back()` invalidate iterators in a vector?**
   - Yes, if the vector needs to **resize** its internal array, all iterators, references, and pointers to elements are invalidated. If no resizing occurs, only the end iterator is invalidated.

4. **How to modify your class to use with `map` and `unordered_map`:**
   - For `map`, you must define a **strict weak ordering** by providing `operator<` or a custom comparator.
   - For `unordered_map`, you need to provide a **hash function** and an **equality function** by overloading `std::hash<T>` and `operator==`.

---

### Why `make_shared` and `make_unique` Are Necessary:

`std::make_shared` and `std::make_unique` offer several advantages over directly using `new` with smart pointers.

1. **Exception Safety:**
   - Before C++17, if you wrote `std::shared_ptr<int> sptr(new int(5));` and an exception occurred while initializing the `shared_ptr` (e.g., due to bad memory allocation), the `new int(5)` memory would leak since there was no smart pointer managing it yet. With `make_shared` or `make_unique`, memory allocation and object construction happen in a single atomic step, ensuring that the object is properly managed by the smart pointer even in case of an exception.

   With the guaranteed order of evaluation introduced in C++17, this issue is mitigated for functions where multiple arguments are being passed and may throw exceptions. However, the atomicity of allocation and construction remains an advantage that still applies even post-C++17.

2. **Fewer Allocations:**
   - With `std::shared_ptr<int>(new int(5))`, two heap allocations occur:
     1. One for the `int` object.
     2. One for the control block that holds the reference count and deleter.

   - With `std::make_shared<int>(5)`, only **one** heap allocation is made because the `int` object and the control block are allocated together in one contiguous block of memory. This reduces the number of allocations, improving performance, especially in high-performance environments like HFT.

`std::make_unique`, on the other hand, simplifies code and guarantees exception safety in the same way, although there is no control block in `unique_ptr`, so no advantage in terms of heap allocations.

---

### Difference Between `std::make_shared<int>(5)` and `std::shared_ptr<int>(new int(5))`:

1. **`std::make_shared<int>(5)`**:
   - **Number of Allocations:** One heap allocation (combines control block and `int` in a single allocation).
   - **Performance:** More efficient because of the reduced number of allocations.

2. **`std::shared_ptr<int>(new int(5))`**:
   - **Number of Allocations:** Two heap allocations (one for the `int` and one for the control block).
   - **Performance:** Less efficient due to two separate allocations.

---

### Templated Function to Remove Duplicates From a Vector of `T`:

#### Why Simply Iterating and Removing Duplicates Doesn’t Work:
When iterating through a `std::vector` and removing elements on the fly, it can cause **iterator invalidation**. Removing an element shifts all subsequent elements down, potentially invalidating existing iterators, making continued iteration unsafe.

#### What Does an Invalid Iterator Mean?
An **invalid iterator** is one that no longer refers to a valid element in the container. This happens when the underlying structure of the container changes (like when an element is removed or the capacity of a vector is resized).

#### Why Are Iterators Invalidated?
- In a `std::vector`, when an element is erased, the rest of the elements after it must be shifted down, causing all iterators after the erased element to become invalid.
- In the case of reallocation (e.g., when `push_back` causes the vector to grow beyond its current capacity), all iterators become invalid because the underlying memory changes.

#### How `std::erase_if` Works:
`std::erase_if` is a standard utility in C++20 that removes elements from a container based on a predicate. Internally, it works by:
1. **Partitioning** the container: It uses `std::remove_if`, which moves elements that should not be removed to the front of the container, while leaving the unwanted elements in the remaining space.
2. **Erasing**: It then calls `erase` on the remaining elements, effectively removing them.

If `std::erase_if` didn’t exist, you could implement this as:
```cpp
template<typename Container, typename Predicate>
void erase_if(Container& container, Predicate pred) {
    container.erase(std::remove_if(container.begin(), container.end(), pred), container.end());
}
```

#### Keeping Track of Seen Elements:

- **Using `std::unordered_set`**:
   If `T` has a hashing function, you can use `std::unordered_set` to keep track of elements seen so far. This gives an **average O(1)** lookup and insertion time, making it the most efficient for large datasets.

- **Using `std::set`**:
   If `T` doesn’t support hashing but does support comparison (i.e., `operator<`), you could use `std::set`. However, this results in **O(log n)** insertion and lookup times, which is slower than `std::unordered_set`.

##### Implications for Expensive-to-Copy Types:
- If `T` is **expensive to copy** (e.g., large objects), storing elements in a `set` or `unordered_set` could be inefficient because every insertion might involve copying. To mitigate this, you could use **move semantics** if `T` supports it, or store **pointers** (e.g., `std::unique_ptr<T>` or `std::shared_ptr<T>`) to avoid expensive copies.

Here’s a possible implementation to remove duplicates using `std::unordered_set`:

```cpp
template<typename T>
void remove_duplicates(std::vector<T>& vec) {
    std::unordered_set<T> seen;
    vec.erase(std::remove_if(vec.begin(), vec.end(),
                             [&seen](const T& value) {
                                 return !seen.insert(value).second;
                             }), vec.end());
}
```

In this example, `insert` returns a pair where `second` is `true` if the element was inserted (i.e., it wasn't a duplicate) and `false` if it was already in the set.

#### Handling Types Without Hashing Support or Expensive Copies:
If `T` doesn't support hashing (needed for `std::unordered_set`) or if copying `T` is too costly, you might switch to a strategy like:

1. **Using `std::set`:**
   For types that support comparison, using `std::set` is feasible, though slower than `unordered_set`.

2. **Using Pointers:**
   For large or expensive-to-copy objects, storing **pointers** (e.g., `std::shared_ptr<T>` or `std::unique_ptr<T>`) instead of the objects directly might be more efficient. This avoids copying large objects but incurs the cost of pointer dereferencing.

3. **Using `std::vector<T*>`:**
   If `T` cannot be copied at all, you could use a `std::vector<T*>` to hold raw pointers, but you must handle memory management manually to avoid leaks, or use smart pointers.

Each strategy depends on the characteristics of the type `T` and the performance considerations in play.

---

These questions test your understanding of memory management, smart pointers, and the implications of working with iterators and data structures under varying constraints, crucial skills in high-performance computing like HFT.


### **Could you explain to me how virtual functions work? Why would you use them?**

**Virtual functions** are a mechanism that supports runtime polymorphism in C++. When a base class declares a function as virtual, derived classes can override that function. At runtime, the appropriate function (either from the base class or a derived class) is called, depending on the type of the object pointed to by the pointer/reference. 

You use virtual functions when you want to allow derived classes to change behavior defined in a base class. It’s essential when you want to implement polymorphism and work with class hierarchies where the behavior of an object should depend on its runtime type.

Example:
```cpp
class Base {
public:
    virtual void print() { std::cout << "Base class"; }
};

class Derived : public Base {
public:
    void print() override { std::cout << "Derived class"; }
};

void printBase(Base* obj) {
    obj->print();  // Will call the correct method at runtime.
}
```

### **What is the overhead of calling a virtual function?**

The overhead of calling a virtual function is generally the cost of dereferencing a pointer (the vtable pointer) and performing an indirect call through the vtable. This is a bit slower than a regular function call, but the difference is often negligible unless it’s in a performance-critical section or called frequently in a tight loop.

### **Are there any situations when this overhead is larger or smaller?**

The overhead can be **larger** if:
- The CPU's branch predictor struggles to predict which function will be called, causing cache misses.
- Virtual calls happen in a performance-critical section like a loop that is called millions of times.

The overhead can be **smaller** if:
- The virtual function call occurs in a well-predicted or infrequent branch.
- Modern CPUs optimize these indirect calls quite well, so in many cases, the overhead is minimal.

### **When should you make a destructor virtual?**

You should make a **destructor virtual** when you have a class that is intended to be inherited from and you expect objects to be deleted through a base class pointer. Without a virtual destructor, only the base class destructor will be called, potentially leading to resource leaks.

Example:
```cpp
class Base {
public:
    virtual ~Base() { std::cout << "Base destructor"; }
};

class Derived : public Base {
public:
    ~Derived() { std::cout << "Derived destructor"; }
};
```

### **Could you explain to me the difference between `static_cast` and `dynamic_cast`?**

- **`static_cast`:** Performs compile-time type checking and can be used to convert between related types (upcast or downcast). It doesn’t do any runtime checks, so it's the programmer's responsibility to ensure the cast is valid.
  
  Example:
  ```cpp
  Base* base = static_cast<Base*>(derivedPtr);  // Compile-time cast
  ```

- **`dynamic_cast`:** Performs runtime type checking and ensures that the cast is valid by checking the actual object type. It's safer for downcasting, especially when working with polymorphism, but incurs a runtime overhead.
  
  Example:
  ```cpp
  Derived* derived = dynamic_cast<Derived*>(basePtr);  // Runtime check
  ```

### **How would you implement a class that opens two files if you could not use exceptions?**

Without exceptions, you can rely on return codes to handle errors:
```cpp
class FileManager {
private:
    FILE* file1;
    FILE* file2;
public:
    bool openFiles(const char* fileName1, const char* fileName2) {
        file1 = fopen(fileName1, "r");
        if (!file1) return false;
        file2 = fopen(fileName2, "r");
        if (!file2) {
            fclose(file1);
            return false;
        }
        return true;
    }

    void closeFiles() {
        if (file1) fclose(file1);
        if (file2) fclose(file2);
    }
};
```

### **How would you implement that class with two files using exceptions?**

With exceptions, you could simplify error handling and rely on RAII:
```cpp
class FileManager {
private:
    std::ifstream file1;
    std::ifstream file2;
public:
    FileManager(const std::string& fileName1, const std::string& fileName2) {
        file1.open(fileName1);
        if (!file1) throw std::runtime_error("Failed to open file 1");
        file2.open(fileName2);
        if (!file2) throw std::runtime_error("Failed to open file 2");
    }

    ~FileManager() {
        if (file1.is_open()) file1.close();
        if (file2.is_open()) file2.close();
    }
};
```

### **How would you handle one of the files failing to open in the case where you have exceptions?**

If one file fails to open, an exception should be thrown. RAII will ensure that the already-opened file is closed during the stack unwinding process.
```cpp
try {
    FileManager manager("file1.txt", "file2.txt");
} catch (const std::exception& e) {
    std::cerr << e.what() << std::endl;
}
```

### **Could your constructor throw an exception after one of the files had been opened and the second one failed? Which destructors would be called in this case?**

Yes, the constructor can throw an exception after opening the first file but failing to open the second. The **destructor of the first file’s RAII object** will be called as part of the stack unwinding, ensuring that resources are released correctly.

### **How would you find the 10 most used words in a large text file?**

You could use an **unordered_map** to count word occurrences and a **min-heap (priority queue)** to keep track of the top 10 words:

```cpp
std::unordered_map<std::string, int> wordCount;
std::priority_queue<std::pair<int, std::string>> topWords;
std::ifstream file("text.txt");

std::string word;
while (file >> word) {
    ++wordCount[word];
}

for (const auto& wc : wordCount) {
    topWords.push({wc.second, wc.first});
    if (topWords.size() > 10) {
        topWords.pop();  // Keep only top 10
    }
}

while (!topWords.empty()) {
    std::cout << topWords.top().second << ": " << topWords.top().first << std::endl;
    topWords.pop();
}
```

### **What is Cache?**

A **cache** is a small, fast memory that sits between the CPU and the main memory (RAM). Its purpose is to store copies of frequently accessed data from main memory so that future requests for that data can be served more quickly. Caches exploit **temporal locality** (accessing the same data repeatedly) and **spatial locality** (accessing data that is near to other data).

#### Cache Hierarchy:
- **L1 Cache**: Closest to the CPU, small in size (tens of KB) but very fast.
- **L2 Cache**: Larger than L1 (hundreds of KB), but slower.
- **L3 Cache**: Shared among CPU cores, larger but slower (MBs).

### **What Are Cache Misses?**

A **cache miss** occurs when the data that the CPU needs is not found in the cache, so the system has to fetch it from the slower main memory. There are three types of cache misses:
- **Cold Miss** (or compulsory miss): Happens when data is accessed for the first time and is not yet in the cache.
- **Conflict Miss**: Occurs when two or more memory addresses map to the same cache line, forcing one to be evicted even though it is still being used.
- **Capacity Miss**: Occurs when the cache is too small to hold all the working data set, and older data must be evicted to make room for new data.

### **How to Design for Minimal Cache Misses:**

To write **cache-friendly code** and minimize cache misses, follow these principles:

- **Data locality**:
  - **Temporal locality**: Access the same data frequently within a short period.
  - **Spatial locality**: Access data that is stored close together in memory (e.g., in arrays).

- **Data structures**:
  - Use **contiguous memory structures** like arrays or `std::vector` instead of linked lists because arrays are better suited for cache due to spatial locality.
  - Avoid excessive pointer chasing (e.g., linked lists, trees) because pointer dereferencing can cause random memory access, leading to cache misses.

- **Data-Oriented Design (DOD)**:
  - Focus on how data is laid out in memory to optimize for cache usage.
  - Instead of focusing solely on objects (object-oriented design), structure your data in a way that reduces cache misses and increases locality. For example, group similar data together rather than scattering it across multiple objects.

Example:
```cpp
struct BadStruct {
    int id;
    float value;
    bool active;
};

// Better: data-oriented design with better spatial locality
struct GoodStruct {
    std::vector<int> ids;
    std::vector<float> values;
    std::vector<bool> activeFlags;
};
```

In the second design, all the `ids`, `values`, and `activeFlags` are stored contiguously, improving cache performance.

### **What is SIMD?**

**SIMD** (Single Instruction, Multiple Data) is a parallel processing technique that allows a single CPU instruction to operate on multiple data points simultaneously. SIMD instructions can significantly speed up operations like processing arrays, matrices, or graphics.

Example: Instead of looping over an array to perform an operation on each element, SIMD allows you to perform the operation on multiple elements at once using special CPU instructions.

### **What is Auto-Vectorization?**

**Auto-vectorization** is a feature of modern compilers that automatically transforms loops and other operations into SIMD instructions when it detects that such a transformation is safe and beneficial for performance. 

### **How to Design for Auto-Vectorization:**
- Write simple, **loop-friendly code**. Auto-vectorization works best with straightforward loops.
- Avoid complex control flow (e.g., branching) inside loops.
- Use contiguous memory structures, as SIMD requires accessing memory in predictable patterns.
- Avoid function calls inside loops. Compilers may not vectorize loops with function calls.
- Use standard algorithms like `std::transform` or `std::for_each` where compilers can apply optimizations more easily.

Example:
```cpp
// Good for vectorization
for (int i = 0; i < n; ++i) {
    data[i] *= 2;
}

// Hard to vectorize (branches and non-contiguous memory)
for (int i = 0; i < n; ++i) {
    if (data[i] % 2 == 0) {
        process(data[i]);
    }
}
```

### **What is Branching?**

**Branching** refers to conditional operations in your code where different instructions are executed based on some condition, typically using `if-else` statements, `switch` statements, or loops with conditionals.

### **What is Branch Prediction?**

**Branch prediction** is a technique used by modern CPUs to guess the outcome of a branch (conditional statement) before it is known, allowing the CPU to preemptively execute instructions to keep the pipeline full. If the prediction is correct, the CPU continues executing efficiently. If the prediction is wrong (branch misprediction), the CPU must discard the work it did after the wrong branch, causing performance penalties.

### **How to Avoid (Excessive) Branching:**

- **Branchless programming**: Use arithmetic and bitwise operations to avoid branches.
  
Example:
```cpp
// With branching
if (x > 0) {
    result = x;
} else {
    result = -x;
}

// Branchless alternative
result = abs(x);
```

- **Minimize unpredictable branches**: Avoid branches that change frequently or are data-dependent. These are harder for branch predictors to handle efficiently.
  
- **Use lookup tables**: For certain problems, you can precompute values and use a lookup table instead of conditionals.

- **Loop unrolling**: Manually or using compiler hints to unroll loops to reduce branches and increase instruction-level parallelism.

### **How to Measure Performance:**

- **Profiling tools**: Use profiling tools to measure performance and identify hotspots in your code. Common tools include:
  - **`perf` (Linux)**: A performance analysis tool.
  - **Valgrind (Linux)**: For detecting cache misses, branch mispredictions, and memory errors.
  - **Intel VTune**: For in-depth analysis of cache usage, branch prediction, and SIMD efficiency.
  - **`gprof` (Linux)**: A simple profiling tool for measuring performance.

- **Cache performance analysis**: Tools like **Valgrind’s cachegrind** or Intel VTune can provide detailed insights into cache hits and misses, and whether your code is cache-friendly.

- **SIMD utilization**: Check if your code is taking advantage of SIMD instructions using compiler reports (e.g., `-fopt-info-vec` in GCC). Some tools, like Intel VTune, can also show SIMD usage.

- **Branch prediction analysis**: Profiling tools can show the rate of branch mispredictions, allowing you to focus on problematic branches and reduce the number of mispredicted branches.

- **Timing measurements**: Simple wall-clock timers (e.g., `std::chrono`) can be used to measure performance before and after applying optimizations. For more precise measurements, use CPU cycle counters (e.g., `rdtsc` on x86).


### Mimic Java's `try-catch-finally` in C++

In C++, you can achieve the same behavior as Java's `try-catch-finally` by using **RAII (Resource Acquisition Is Initialization)** and **smart pointers**. The idea is that resources are acquired when an object is created and released when the object goes out of scope. This ensures that clean-up code is executed regardless of whether an exception is thrown.

The most common C++ technique to replicate `finally` is to use **destructors** of stack-allocated objects, which are guaranteed to run when an object goes out of scope, regardless of whether an exception was thrown.

Here’s a simple example that demonstrates how to use a custom class to mimic the behavior of `finally`:

#### Example: Using RAII for Cleanup

```cpp
#include <iostream>

class Finally {
public:
    Finally(std::function<void()> func) : cleanup(func) {}
    ~Finally() {
        // Destructor will always run, even if an exception is thrown
        cleanup();
    }
private:
    std::function<void()> cleanup;
};

int main() {
    try {
        Finally onExit([] {
            std::cout << "Cleaning up resources." << std::endl;
        });

        std::cout << "Doing some work..." << std::endl;
        
        // Simulate an exception
        throw std::runtime_error("Something went wrong!");

    } catch (const std::exception& e) {
        std::cout << "Caught exception: " << e.what() << std::endl;
    }

    return 0;
}
```

#### Explanation:
- The `Finally` class takes a lambda function or callable object in its constructor. This function is the clean-up code.
- The destructor of the `Finally` object is invoked automatically when the object goes out of scope, ensuring that the clean-up code is executed whether or not an exception was thrown.
- Even if an exception occurs within the `try` block, the `Finally` object’s destructor will still run.

#### Alternatives:
You can also use **standard smart pointers** (like `std::unique_ptr`) with custom deleters for resource management:

```cpp
std::unique_ptr<int, std::function<void(int*)>> ptr(new int(5), [](int* p) {
    std::cout << "Cleanup in custom deleter." << std::endl;
    delete p;
});
```

This leverages smart pointers' destructors to guarantee clean-up.

### Iterator Invalidation

#### Containers with **Iterator Invalidation** on Erasure:

1. **`std::vector`:**
   - **Iterator invalidation on erasure**:
     - When an element is erased using `erase()`, all **iterators and references to elements after the erased element** are invalidated because the remaining elements are shifted to fill the gap.
   - **Iterator invalidation on resizing**:
     - If the `vector` grows beyond its current capacity (e.g., due to `push_back`), **all iterators, references, and pointers** are invalidated because the underlying array is reallocated.

2. **`std::deque`:**
   - **Iterator invalidation on erasure**:
     - If an element is erased, all **iterators, references, and pointers to elements after the erased element** are invalidated, as elements may be shifted.
   - **Iterator invalidation on resizing**:
     - If a reallocation is needed, **all iterators** are invalidated, but **references to elements are preserved** unless an element is erased or inserted.

3. **`std::string`:**
   - **Iterator invalidation on erasure**:
     - Similar to `std::vector`, all **iterators and references after the erased element** are invalidated due to the need to shift elements.
   - **Iterator invalidation on resizing**:
     - Similar to `std::vector`, if a reallocation occurs, **all iterators, references, and pointers** are invalidated.

4. **`std::map`, `std::set`, `std::multimap`, `std::multiset`:**
   - **Iterator invalidation on erasure**:
     - Only the **iterator to the erased element** is invalidated. Iterators to other elements remain valid because the underlying tree structure does not require shifting elements.
   - **Iterator invalidation on insertion**:
     - Insertion does not invalidate iterators, except for the iterator to the newly inserted element if it points to the container's `end()`.

5. **`std::unordered_map`, `std::unordered_set`, `std::unordered_multimap`, `std::unordered_multiset`:**
   - **Iterator invalidation on erasure**:
     - Only the **iterator to the erased element** is invalidated.
   - **Iterator invalidation on rehashing** (e.g., due to `reserve()` or insertion causing a rehash):
     - **All iterators** are invalidated if a rehashing occurs. This happens when the number of elements exceeds the load factor, forcing the container to reallocate buckets.

#### Containers with **No Iterator Invalidation** on Erasure:

1. **`std::list`:**
   - **Iterator invalidation on erasure**:
     - Erasing an element invalidates **only the iterator to the erased element**. All other iterators remain valid because `std::list` is a doubly-linked list, and no element shifting occurs.
   - **Iterator invalidation on insertion**:
     - Inserting elements does not invalidate any iterators, including `end()`.

2. **`std::forward_list`:**
   - **Iterator invalidation on erasure**:
     - Similar to `std::list`, only the **iterator to the erased element** is invalidated.
   - **Iterator invalidation on insertion**:
     - Inserting elements does not invalidate iterators, including `end()`.

#### Summary of Iterator Invalidation:

| Container                 | On Erasure                      | On Insertion/Resizing              |
|---------------------------|----------------------------------|------------------------------------|
| `std::vector`              | Iterators after erased element   | All iterators if reallocation occurs |
| `std::deque`               | Iterators after erased element   | All iterators if reallocation occurs |
| `std::string`              | Iterators after erased element   | All iterators if reallocation occurs |
| `std::list`                | Only the erased iterator         | No invalidation                    |
| `std::forward_list`        | Only the erased iterator         | No invalidation                    |
| `std::map`, `std::set`     | Only the erased iterator         | No invalidation                    |
| `std::unordered_map`, `std::unordered_set` | Only the erased iterator | All iterators if rehashing occurs  |


### **What does `std::move` do? Why is it needed?**

**`std::move`** is a cast that converts an lvalue (an object with a persistent location in memory) into an **rvalue reference**, enabling **move semantics**. It doesn’t actually move anything itself but signals that the object passed can be “moved from,” meaning its resources (e.g., memory) can be transferred rather than copied. This avoids costly deep copying of resources, improving performance.

**Why is it needed?**
- **Move semantics** are essential for avoiding unnecessary deep copies, especially for objects that manage large or non-trivial resources like dynamic memory or file handles.
- When using `std::move`, the compiler knows that it can use the object's resources (such as memory buffers) rather than copying them, thus performing a shallow copy.
  
For example:
```cpp
std::string s1 = "Hello, World!";
std::string s2 = std::move(s1);  // s1 is now in an unspecified but valid state.
```
After the move, `s1` is left in a valid but unspecified state (typically empty), and `s2` has taken ownership of the resources from `s1`.

### **What are Universal References?**

A **universal reference** is a reference that can bind to both lvalues and rvalues. It occurs when a template parameter is a forwarding reference (T&&) and the function is templated. The term was coined by Scott Meyers in his book *Effective Modern C++*.

Example:
```cpp
template <typename T>
void foo(T&& param);  // 'param' is a universal reference
```

If you pass an lvalue to `foo`, `param` becomes an lvalue reference (`T&`), and if you pass an rvalue, `param` becomes an rvalue reference (`T&&`). This allows perfect forwarding of arguments using `std::forward`.

### **What Are Reasons to Mark a Constructor `explicit`?**

Marking a constructor **`explicit`** prevents it from being used for **implicit conversions** and **copy-initialization**. This is important to avoid accidental conversions that can lead to unexpected behavior.

Example:
```cpp
class MyClass {
public:
    explicit MyClass(int x) { /* ... */ }
};

MyClass obj = 5;  // Error! Implicit conversion is not allowed due to 'explicit'
```

**Reasons to mark a constructor explicit**:
- Prevent **unintended implicit conversions**.
- Improve **code clarity**, ensuring constructors are called only when explicitly intended.
- Avoid subtle bugs, especially when defining single-argument constructors that could lead to accidental type conversions.

### **What Is Slicing? How to Prevent It?**

**Object slicing** occurs when a derived class object is assigned to a base class object, causing the **derived part of the object to be "sliced off"**, leaving only the base class portion.

Example:
```cpp
class Base {
public:
    int x;
};

class Derived : public Base {
public:
    int y;
};

Derived d;
Base b = d;  // Slicing occurs: 'b' only contains 'x', not 'y'
```

To prevent slicing:
- **Use pointers or references** to base classes, rather than directly storing objects.
  
Example:
```cpp
Base* b = &d;  // No slicing, as we are storing a pointer to the derived class.
```

Alternatively, you can mark the base class’s copy constructor or assignment operator as **`delete`** to prevent copying altogether.

### **In What Circumstances Is the `auto` Keyword Required?**

`auto` is required or useful in several circumstances:
- When the **type is complex** or difficult to write, such as iterators or function objects returned from STL algorithms:
  ```cpp
  auto it = myContainer.begin();  // Simplifies code
  ```
  
- When dealing with **lambda expressions**, where the exact type of the lambda is unknown:
  ```cpp
  auto lambda = [](int x) { return x * 2; };
  ```

- When using **type deduction for template expressions**:
  ```cpp
  template<typename T, typename U>
  auto add(T a, U b) -> decltype(a + b) {
      return a + b;
  }
  ```

- When you want to let the compiler **deduce the type**, ensuring flexibility and potentially avoiding unnecessary type conversions.

### **Explain the Difference Between Undefined Behavior and Implementation-Defined Behavior.**

- **Undefined Behavior (UB):**
  - Occurs when the C++ standard does not specify what happens for certain conditions, and the result can be unpredictable. The compiler is allowed to do **anything**, including crashing, producing incorrect results, or running as expected.
  - **Example:** Accessing an array out of bounds, dereferencing a null pointer, or modifying a variable multiple times without a sequence point.
  ```cpp
  int x = 5;
  x = ++x + x++;  // Undefined behavior, as the order of evaluation is not specified.
  ```

- **Implementation-Defined Behavior:**
  - The C++ standard allows certain behaviors to be decided by the compiler, but the compiler must document how it handles these situations. It can vary across platforms, but it must be **documented**.
  - **Example:** The size of `int`, or how integer overflow is handled.
  ```cpp
  int a = -5 % 3;  // Implementation-defined behavior (depends on the compiler how negative mod works).
  ```

### **What Happens If You Call a Virtual Function in a Constructor?**

If you call a **virtual function** from a constructor (or destructor), the **version of the function in the base class** will be called, not the overridden version in a derived class. This is because, during construction (and destruction), the object is treated as an instance of the base class, and the virtual table (vtable) is not fully set up to point to derived class functions.

Example:
```cpp
class Base {
public:
    Base() {
        virtualFunction();  // Calls Base::virtualFunction
    }
    virtual void virtualFunction() { std::cout << "Base\n"; }
};

class Derived : public Base {
public:
    void virtualFunction() override { std::cout << "Derived\n"; }
};

int main() {
    Derived d;  // Output will be "Base", not "Derived"
}
```

**Why this happens:**
- When constructing an object, the constructor for the base class runs first. At this point, the derived class’s part of the object has not been fully constructed, so the virtual table still refers to the base class's virtual functions. Calling a virtual function at this stage results in the base class’s version being invoked.


### **Types of Cast Operations in C++:**

C++ provides several types of casting operations that allow you to convert between types, control type safety, and manage polymorphism. These casts are more powerful and safer than the traditional C-style casts, and each serves a specific purpose.

#### a. **`static_cast`**:
   - **Purpose**: Used for compile-time type conversions between related types (e.g., upcasting in inheritance hierarchies, converting numeric types like `int` to `double`, etc.).
   - **Usage**: 
     - Safe when you are certain about the type conversion.
     - Cannot be used for casting from a base class to a derived class if the base class is not polymorphic.
   - **Example**:
     ```cpp
     int a = 10;
     double b = static_cast<double>(a);  // Convert int to double
     ```

#### b. **`dynamic_cast`**:
   - **Purpose**: Used for safe downcasting in polymorphic inheritance hierarchies (classes with virtual functions). It performs a runtime check to ensure the cast is valid and returns `nullptr` if it fails (when casting pointers).
   - **Usage**:
     - Only works with polymorphic types (classes with at least one virtual function).
     - Slower than `static_cast` because of the runtime check.
   - **Example**:
     ```cpp
     class Base {
         virtual void foo() {}
     };
     class Derived : public Base {};
     
     Base* basePtr = new Derived;
     Derived* derivedPtr = dynamic_cast<Derived*>(basePtr);  // Safe downcast
     ```

#### c. **`const_cast`**:
   - **Purpose**: Used to add or remove `const` qualifiers from a variable.
   - **Usage**:
     - Can be used to modify a `const` variable, though modifying a truly constant variable (e.g., `const int x = 5;`) is **undefined behavior**.
   - **Example**:
     ```cpp
     const int a = 10;
     int* b = const_cast<int*>(&a);  // Remove constness
     ```

#### d. **`reinterpret_cast`**:
   - **Purpose**: Used for low-level reinterpreting of bits. It allows casting between unrelated types, such as casting a pointer to an integer or vice versa.
   - **Usage**:
     - Dangerous because it bypasses type safety and can lead to undefined behavior if not used carefully.
   - **Example**:
     ```cpp
     int* p = reinterpret_cast<int*>(0xDEADBEEF);  // Cast a pointer to an int
     ```
   
#### e. **C-style Cast (`(Type)variable`)**:
   - **Purpose**: A legacy cast from C, which combines the functionality of `static_cast`, `const_cast`, and `reinterpret_cast`, but without type safety.
   - **Usage**:
     - Should generally be avoided in modern C++ because it's less safe and harder to identify the kind of cast being performed.
   - **Example**:
     ```cpp
     int a = 10;
     double b = (double)a;  // C-style cast (less safe)
     ```

### **Upcasting and Downcasting**

#### a. **Upcasting**:
   - **Definition**: Converting a pointer or reference from a derived class type to a base class type.
   - **Characteristics**:
     - Always safe.
     - Typically done implicitly.
     - No runtime overhead.
   - **Example**:
     ```cpp
     class Base {};
     class Derived : public Base {};

     Derived d;
     Base* basePtr = &d;  // Implicit upcast (Derived* to Base*)
     ```

#### b. **Downcasting**:
   - **Definition**: Converting a pointer or reference from a base class type to a derived class type.
   - **Characteristics**:
     - Requires a runtime check when using `dynamic_cast` (for polymorphic types).
     - Unsafe if the object is not of the derived type, so use `dynamic_cast` to ensure safety.
   - **Example**:
     ```cpp
     Base* basePtr = new Derived;
     Derived* derivedPtr = dynamic_cast<Derived*>(basePtr);  // Safe downcast
     ```

#### **Examples of Upcasting and Downcasting**

##### **Upcasting**:
Upcasting is typically used when you want to treat an object of a derived class as if it were an object of the base class. Since a derived class contains all members of the base class, this is safe and often implicit.

```cpp
class Animal {
public:
    virtual void speak() { std::cout << "Animal speaks\n"; }
};

class Dog : public Animal {
public:
    void speak() override { std::cout << "Dog barks\n"; }
};

Dog dog;
Animal* animal = &dog;  // Upcasting: implicit and safe
animal->speak();  // Outputs: "Dog barks" because of virtual function
```

##### **Downcasting**:
Downcasting is used when you want to access derived class members using a base class pointer. However, downcasting is not safe unless you are certain that the base class pointer actually points to an object of the derived class. Use `dynamic_cast` to perform this safely.

```cpp
Animal* animal = new Dog();  // Upcast: implicit and safe

Dog* dog = dynamic_cast<Dog*>(animal);  // Downcast: safe because animal is pointing to a Dog
if (dog) {
    dog->speak();  // Outputs: "Dog barks"
}
```

If `animal` were pointing to an `Animal` object that is not a `Dog`, the `dynamic_cast` would return `nullptr`, preventing invalid access.

#### **When to Use Each Cast**

- **`static_cast`**: Use when you need a simple and safe conversion between related types, and you are sure the conversion is valid (e.g., upcasting).
- **`dynamic_cast`**: Use when you need to safely downcast in a polymorphic class hierarchy, and you want a runtime check to ensure the validity of the cast.
- **`const_cast`**: Use when you need to add or remove `const` or `volatile` qualifiers. Be cautious when modifying `const` objects, as this can lead to undefined behavior.
- **`reinterpret_cast`**: Use only for low-level operations, such as when dealing with raw memory or interfacing with hardware. This is the most dangerous and should be used sparingly.
- **C-style cast**: Should be avoided in modern C++ code, as it lacks the type safety provided by C++'s specialized cast operators.

### Polymorphism in detail

Polymorphism in C++ is a key feature of object-oriented programming, allowing objects of different classes to be treated through the same interface (typically via a base class). This is achieved through **virtual functions** and **dynamic binding** (i.e., the decision of which function to call is made at runtime, rather than compile time). Here's a detailed explanation of the complete flow of polymorphism, including how derived class objects are created, how the virtual table (vtable) works, and how virtual function calls are dispatched.

### 1. **Polymorphism Setup: Virtual Functions and Inheritance**

In C++, polymorphism relies on:
- **Inheritance**: A base class defines an interface or common functionality, and derived classes override or extend that functionality.
- **Virtual functions**: Functions marked with the `virtual` keyword can be overridden in derived classes. The decision about which function to call (base or derived) is made at runtime based on the type of the object being pointed to, not the type of the pointer/reference itself.

### 2. **Object Layout and the Virtual Table (vtable)**

#### a. **Virtual Table (vtable):**
The **vtable** (virtual table) is a mechanism used by C++ compilers to support **dynamic dispatch**. It’s an internal table maintained by the compiler for every class that has virtual functions. The vtable contains **pointers to the virtual functions** that can be called on instances of the class.

For each class with virtual functions:
- The compiler creates a **vtable** that holds pointers to the **most derived** version of each virtual function in that class.
- Each object of a class that uses virtual functions has a hidden pointer called a **vptr** (virtual pointer) that points to the class's vtable.

#### b. **Vtable Creation and Object Layout:**
When you create an instance of a class that has virtual functions:
- The compiler sets up the object’s memory layout, which includes the data members and a **vptr** (pointer to the vtable).
- During the **construction of the object**, the vptr is initialized to point to the vtable of the class being constructed.

For example:
```cpp
class Base {
public:
    virtual void speak() { std::cout << "Base speaks\n"; }
    virtual ~Base() {}
};

class Derived : public Base {
public:
    void speak() override { std::cout << "Derived speaks\n"; }
};
```

- **Base** has a vtable with a pointer to `Base::speak()`.
- **Derived** has its own vtable, which points to `Derived::speak()`.
- If you create a `Derived` object, its vptr will point to the **Derived vtable**.

#### c. **Vtable Setup in Constructor:**
- During the **base class constructor** execution, the object’s vptr points to the **Base vtable**.
- As soon as the **Derived class constructor** starts, the object’s vptr is updated to point to the **Derived vtable**.

### 3. **Flow of Virtual Function Calls (Dynamic Dispatch)**

When a virtual function is called, the flow is as follows:
1. **Call via a base class pointer/reference**:
   If a virtual function is called on a base class pointer or reference, the runtime system looks at the **vptr** of the object being pointed to. The vptr leads to the appropriate vtable.
   
   Example:
   ```cpp
   Base* obj = new Derived();
   obj->speak();  // Calls Derived::speak() due to dynamic dispatch
   ```
   
2. **vptr Dereferencing**:
   The runtime looks up the function pointer in the vtable (using the vptr) and calls the correct function based on the actual type of the object, which in this case is `Derived`. So even though the pointer type is `Base*`, it will call `Derived::speak()`.

### 4. **Virtual Destructors and the Vtable**
A **virtual destructor** is necessary when deleting an object of a derived class through a pointer to a base class. Without a virtual destructor, only the base class destructor would be called, potentially leading to resource leaks (e.g., memory or file handles) since the derived class’s destructor would not be invoked.

When a class has a virtual destructor, the vtable also contains a pointer to the **most derived destructor**. This ensures that when `delete` is called, the destructors are called in the correct order:
1. Derived class destructor.
2. Base class destructor.

### 5. **Nuances of Polymorphism and the Vtable**

#### a. **Calling a Virtual Function from a Constructor/Destructor:**
- During object construction, the vptr points to the vtable of the currently constructing class.
- If a virtual function is called from within a **base class constructor**, the base class version of the function is called, **not** the derived class version, because the object is still being constructed and the derived class part has not been initialized yet.

Example:
```cpp
class Base {
public:
    Base() { speak(); }  // Calls Base::speak(), even if Derived constructor is running
    virtual void speak() { std::cout << "Base speaks\n"; }
};

class Derived : public Base {
public:
    Derived() : Base() {}
    void speak() override { std::cout << "Derived speaks\n"; }
};

Derived d;  // Output: "Base speaks"
```
- The same principle applies when calling virtual functions from destructors. The **most derived version of the object** is no longer valid, so base class functions are called.

#### b. **Multiple Inheritance and the Vtable:**
In the case of **multiple inheritance** where a class inherits from more than one base class, each base class will have its own vptr. The object layout becomes more complex, and there may be multiple vtables for each object to handle the different base class pointers.

Example:
```cpp
class A {
public:
    virtual void foo() { std::cout << "A's foo\n"; }
};

class B {
public:
    virtual void foo() { std::cout << "B's foo\n"; }
};

class C : public A, public B {
public:
    void foo() override { std::cout << "C's foo\n"; }
};
```
In this case, class `C` has two vtables—one for `A` and one for `B`—and each vtable has pointers to the appropriate `foo()` function.

#### c. **Performance Overhead of Virtual Functions**:
While virtual functions provide flexibility through polymorphism, they introduce some overhead:
- **Indirect function call**: The function call is made through a pointer (vptr lookup), which is slower than a direct function call.
- **Additional memory**: Each object requires a hidden vptr, and vtables occupy additional memory. However, the vtable is typically shared across all objects of the same class.

### 6. **Summary of Polymorphism Flow**:

1. **Derived class creation**:
   - The base class constructor initializes the base part of the object, setting the vptr to point to the base class’s vtable.
   - When the derived class constructor starts, the vptr is updated to point to the derived class’s vtable.
   
2. **Vtable creation**:
   - The compiler generates a vtable for each class with virtual functions, containing pointers to the appropriate function implementations.

3. **Virtual function call**:
   - At runtime, the object’s vptr points to the correct vtable, and the function is called through the vtable, ensuring that the most derived function is invoked (if the call is made via a base class pointer/reference).
   
4. **Destruction**:
   - If the destructor is virtual, destructors are called in the correct order, starting with the derived class and ending with the base class.



### **Runtime Difference Between Templated Code and Virtualization**

- **Templated Code (Compile-Time Polymorphism)**:
  - **Templates** in C++ are instantiated at **compile-time**, meaning that a unique version of the templated code is generated for each type used. This leads to **zero runtime overhead** because the function calls are resolved during compilation, and there are no indirections (no vtable lookups or virtual dispatch).
  - **Benefits**:
    - No runtime overhead.
    - Better optimization by the compiler (inlining, constant propagation).
  - **Drawbacks**:
    - Code bloat: For each type `T`, a new version of the template code is instantiated, which increases binary size (sometimes significantly).

- **Virtualization (Runtime Polymorphism)**:
  - With **virtual functions**, the decision of which function to call is made at **runtime** via a vtable lookup. This involves dereferencing the vptr (virtual table pointer) to call the appropriate function.
  - **Benefits**:
    - Flexibility: You can use polymorphic behavior dynamically and allow for different implementations of the same interface to be chosen at runtime.
  - **Drawbacks**:
    - **Runtime overhead** due to indirection (vtable lookup and pointer dereferencing).
    - Harder to optimize due to the need for dynamic dispatch.
  
In summary, **templated code** is generally faster at runtime due to **compile-time resolution**, whereas **virtualization** incurs a runtime cost due to the **dynamic dispatch mechanism**.

### **What is Hot/Cold Cache? What is Cache Ping-Pong?**

- **Hot Cache**:
  - Refers to data that is **frequently accessed** and is therefore likely to be present in the cache when needed. This means lower memory latency and higher performance since cache hits (when data is already in the cache) are much faster than fetching data from main memory.

- **Cold Cache**:
  - Refers to data that is **infrequently accessed** or recently evicted from the cache, resulting in a **cache miss**. The CPU must then fetch the data from main memory, which is significantly slower than accessing cached data.

- **Cache Ping-Pong**:
  - This is a performance issue that occurs in **multithreaded programs** when multiple processors or cores keep **invalidating** and **updating the same cache line**. If two threads modify variables that share the same cache line (even if the variables are different), the cache coherence protocol forces one thread to invalidate the other thread’s cache, causing frequent memory transfers between caches.
  - **Effect**: Increased memory traffic and slower performance due to constant invalidation and reloading of cache lines.
  - **Solution**: Use techniques like **padding** or **aligning** variables to different cache lines to prevent this issue.

### **Questions About Data-Oriented Design, Lock-Free Data Structures, and Atomics**

- **Data-Oriented Design (DOD)**:
  - **Data-Oriented Design** focuses on organizing data in memory to optimize **cache efficiency**. Rather than focusing on objects and classes (as in Object-Oriented Design), DOD emphasizes **contiguity of data** (e.g., arrays of structs or struct of arrays) to minimize cache misses.
  - **Why it matters**: Optimizing for how data is accessed (temporal and spatial locality) can significantly improve performance, especially in systems where memory access time is the bottleneck.

- **Lock-Free Data Structures**:
  - **Lock-free data structures** (like lock-free queues, stacks) allow multiple threads to **access and modify shared data** without the use of traditional locks (e.g., mutexes).
  - **Why it matters**: These structures use **atomic operations** to avoid the contention and overhead caused by locks. This can be important in high-performance and real-time systems where thread blocking is unacceptable.
  - **Examples**: Lock-free stacks, queues, and hash maps. These typically rely on atomic instructions like **compare-and-swap (CAS)** or **fetch-and-add**.

- **Atomics**:
  - **Atomics** in C++ (from the `<atomic>` header) allow for operations on variables that are **guaranteed to be atomic** (indivisible). This prevents race conditions in multi-threaded programs without the need for locks.
  - **Types of atomic operations**:
    - **Atomic loads and stores**.
    - **Atomic fetch-add, fetch-subtract, compare-exchange (CAS)**.
  - **Memory ordering**: Atomics come with various memory order guarantees (`memory_order_relaxed`, `memory_order_acquire`, `memory_order_release`) to control visibility and synchronization between threads.

### **Questions About Rvalue Semantics and Perfect Forwarding**

- **Rvalue Semantics**:
  - Rvalue semantics refer to the concept of **temporaries or objects that are about to go out of scope**. Using **move semantics** (introduced in C++11), rvalue references (`T&&`) allow for transferring ownership of resources rather than copying them.
  - **Why it matters**: Move semantics improve performance by **avoiding unnecessary copies** of resources (e.g., dynamic memory or file handles) when objects are passed around or returned from functions.
  
  Example of move semantics:
  ```cpp
  std::string a = "Hello";
  std::string b = std::move(a);  // Move ownership of a's resource to b
  ```

- **Perfect Forwarding**:
  - **Perfect forwarding** is the technique that allows a function template to **forward its arguments exactly as received** (whether they are lvalues, rvalues, const, or non-const) to another function.
  - **How it works**: Perfect forwarding is achieved using **universal references** (`T&&`) in conjunction with `std::forward`.
  
  Example:
  ```cpp
  template<typename T>
  void wrapper(T&& arg) {
      foo(std::forward<T>(arg));  // Perfectly forward the argument
  }
  ```

### **How Would You Implement Lock-Free Data Structures?**

**Lock-free data structures** are used to allow multiple threads to access and modify shared data concurrently without using traditional locks (e.g., `std::mutex`), avoiding the overhead and contention associated with locking. These structures rely on **atomic operations** to ensure synchronization between threads.

#### Steps to Implement Lock-Free Data Structures:

1. **Use Atomic Operations:**
   - Implement lock-free data structures using **atomic operations** such as **`compare-and-swap` (CAS)**, **`fetch-and-add`**, or **`atomic exchange`**. These operations are guaranteed to be executed atomically by the CPU, meaning they cannot be interrupted.

2. **Compare-and-Swap (CAS) Principle:**
   - CAS is the core atomic operation for many lock-free data structures. It compares the value at a memory location to an expected value and, if they are the same, updates the memory location with a new value in a single atomic step.
   - Example:
     ```cpp
     std::atomic<int> value = 0;
     int expected = 0;
     int desired = 1;
     value.compare_exchange_strong(expected, desired);  // Swap if value equals expected
     ```

3. **Memory Fences and Ordering:**
   - Use memory fences (e.g., `std::memory_order_acquire`, `std::memory_order_release`) to ensure correct ordering of operations. This is important for preventing reordering of operations in a multi-threaded environment.

4. **ABA Problem:**
   - In lock-free data structures, the **ABA problem** occurs when a value is changed from `A` to `B` and back to `A` before another thread checks it. The CAS operation would incorrectly succeed because the value appears unchanged. To avoid this, use techniques such as **version tagging** (including a version number in the pointer) to detect when the value has changed.

5. **Example: Lock-Free Stack:**
   - A basic lock-free stack can be implemented using `CAS` to update the top of the stack:
     ```cpp
     struct Node {
         int data;
         Node* next;
     };

     std::atomic<Node*> head(nullptr);

     void push(int value) {
         Node* newNode = new Node{value, nullptr};
         do {
             newNode->next = head.load();
         } while (!head.compare_exchange_weak(newNode->next, newNode));
     }

     bool pop(int& value) {
         Node* oldHead = head.load();
         do {
             if (oldHead == nullptr) return false;
         } while (!head.compare_exchange_weak(oldHead, oldHead->next));
         value = oldHead->data;
         delete oldHead;
         return true;
     }
     ```

### **What is a Union? What is it Used For? Struct vs Union.**

#### **Union:**
A **union** is a data structure in which all members share the same memory location. Only one member can hold a valid value at a time, and writing to one member overwrites the value of the other members.

#### **Use Cases:**
- Unions are used when a variable may hold one of several types, but only one type is needed at any given time. They are useful for **memory-efficient** storage.
- Commonly used in low-level programming, such as in embedded systems, for implementing **variant types** (similar to a `std::variant` in modern C++).

#### **Union vs Struct:**
- **Union**: All members share the same memory location, so only one member can be used at a time. The size of the union is equal to the size of its largest member.
- **Struct**: Each member has its own memory location, so all members can be used simultaneously. The size of a struct is the sum of its members' sizes (with potential padding for alignment).

Example of a union:
```cpp
union Data {
    int i;
    float f;
    char str[20];
};

Data data;
data.i = 42;      // Stores an int
data.f = 3.14;    // Now stores a float, overwriting the int
```

### **RAII in Depth (Resource Acquisition Is Initialization)**

**RAII (Resource Acquisition Is Initialization)** is a programming idiom in C++ that ties the lifecycle of a resource (e.g., memory, file handles, mutexes) to the lifespan of an object. The resource is acquired during object construction and released when the object is destroyed (in its destructor). This ensures that resources are properly cleaned up even if exceptions occur.

#### Key Concepts:
1. **Constructor Acquires Resources**: Resources (e.g., file handles, memory) are acquired during the constructor's execution.
2. **Destructor Releases Resources**: When the object goes out of scope, its destructor is called, releasing the resources automatically.
3. **Exception Safety**: RAII ensures that resources are properly cleaned up even when exceptions are thrown, as destructors are called during stack unwinding.

#### Example: File Handling with RAII:
```cpp
class File {
    FILE* file;
public:
    File(const char* filename, const char* mode) {
        file = fopen(filename, mode);
        if (!file) throw std::runtime_error("Could not open file");
    }
    ~File() {
        if (file) fclose(file);
    }
    // Additional methods for file operations
};

void func() {
    File f("example.txt", "r");
    // File is automatically closed when `f` goes out of scope
}
```

In this example, the file is automatically closed when the `File` object goes out of scope, even if an exception is thrown, ensuring proper resource management.

### **Open a File and Retrieve a Specific Number of Bytes from It**

You can use the **C++ standard library's** file I/O utilities (`std::ifstream`) to open a file and read a specific number of bytes. Here's an example:

```cpp
#include <iostream>
#include <fstream>
#include <vector>

void readBytes(const std::string& fileName, size_t numBytes) {
    std::ifstream file(fileName, std::ios::binary);
    if (!file) {
        std::cerr << "Failed to open file\n";
        return;
    }

    std::vector<char> buffer(numBytes);
    file.read(buffer.data(), numBytes);
    
    if (file) {
        std::cout << "Successfully read " << numBytes << " bytes\n";
    } else {
        std::cout << "Only " << file.gcount() << " bytes could be read\n";
    }
}

int main() {
    readBytes("example.txt", 100);  // Read the first 100 bytes
}
```

This example opens a file in binary mode, reads a specific number of bytes into a buffer, and handles the case where fewer bytes than requested are available.

### **Why Would You Use `std::array` Instead of C-Arrays?**

**`std::array`** is a fixed-size array in C++ that provides several advantages over raw C-style arrays:
1. **Type Safety**: `std::array` is a safer alternative to C arrays because it provides bounds checking in debug mode when accessing elements using `at()`.
2. **STL Compatibility**: `std::array` integrates with the STL algorithms (e.g., `std::sort`, `std::for_each`) since it has iterator support.
3. **Size Information**: `std::array` knows its size at compile-time using the `.size()` method, unlike C-arrays, where size information is lost when passed to a function.
4. **Cleaner Syntax**: Provides a cleaner and more consistent interface with functions and algorithms.

Example:
```cpp
std::array<int, 5> arr = {1, 2, 3, 4, 5};
std::sort(arr.begin(), arr.end());
```

### **How Does an Atomic Instruction Work on a CPU?**

An **atomic instruction** on a CPU is one that is executed as an **indivisible operation**. This means that no other thread can observe or interrupt the operation midway—it either happens fully or not at all.

#### How It Works:
- CPUs provide atomic instructions that work directly on the memory, such as `CAS (Compare-And-Swap)` or `fetch-and-add`. These instructions are implemented at the hardware level, ensuring that no other thread or core can intervene during the operation.
- **Cache Coherency**: Atomic operations ensure that the CPU cache remains coherent across multiple cores, preventing race conditions when multiple threads operate on shared memory.
- **Memory Barriers**: Some atomic operations may involve **memory barriers** (also known as memory fences), which ensure the correct ordering of loads and stores.

### **What Do `override` and `final` Do?**

#### **`override`**:
- `override` is used in a derived class to indicate that a virtual function is intended to **override** a base class virtual function. It ensures that the base class indeed has a matching virtual function with the same signature.
- **Why it's useful**:
  - It helps catch mistakes at compile time, such as incorrect function signatures or missing virtual functions in the base class.

Example:
```cpp
class Base {
    virtual void func() {};
};

class Derived : public Base {
    void func() override {};  // Ensures that `func` is overriding `Base::func`
};
```

#### **`final`**:
- `final` prevents further overriding of a virtual function or inheritance from a class. This ensures that no derived class can override a virtual function or inherit from a class that is marked as `final`.

- **Why it's useful**:
  - It helps catch errors where unintended overriding might occur.
  - It can also enable certain **compiler optimizations**. When a function is marked as `final`, the compiler knows that no further overrides exist, and it can optimize virtual function calls by potentially inlining the function or skipping the vtable lookup.

Example:
```cpp
class Base {
    virtual void func() final { std::cout << "Base::func\n"; }  // Cannot be overridden
};

class Derived final : public Base {
    // Cannot inherit from Derived in future classes
};
```

In this example:
- `Derived::func()` cannot override `Base::func()` because `func` is marked as `final`.
- No further class can inherit from `Derived` because `Derived` itself is marked `final`.


### **Constructors: Should They Throw or Be `noexcept`?**

- **Constructors can throw exceptions**. This is because constructors often perform resource allocations (e.g., dynamic memory, file handles, or network connections), and if these allocations fail (e.g., memory allocation failure), it is reasonable for the constructor to throw an exception to indicate failure.
- If a constructor throws, it prevents the object from being created in an invalid state, and C++ will automatically handle clean-up for any fully constructed subobjects.

#### When to throw:
- Throw exceptions from constructors when it's impossible to create the object in a valid state due to failed resource acquisition or invalid input.
  
Example:
```cpp
class Resource {
public:
    Resource() {
        if (!initialize()) {
            throw std::runtime_error("Failed to initialize resource");
        }
    }
};
```

#### When to mark `noexcept`:
- If a constructor is simple and guaranteed not to throw (e.g., it only initializes primitive types or POD structs), you can mark it as `noexcept`. Marking constructors `noexcept` is beneficial because it can enable certain optimizations (e.g., in `std::vector`, where move operations are often assumed `noexcept`).

Example:
```cpp
class Simple {
public:
    Simple() noexcept = default;
};
```

#### Guidelines:
- **Default constructors** are often marked `noexcept` when they don’t perform any complex operations that could fail.
- If your constructor allocates resources or interacts with external systems, allowing it to throw is reasonable.

### **Destructors: Should They Throw or Be `noexcept`?**

- **Destructors should never throw exceptions**. If a destructor throws an exception during stack unwinding (e.g., while another exception is propagating), the C++ runtime will call `std::terminate()`, causing the program to abruptly terminate.
- By default, destructors are `noexcept` unless explicitly declared otherwise. Marking destructors `noexcept` ensures they follow best practices and prevent exceptions from leaking.

#### When to throw:
- **Never** throw exceptions from a destructor. If a destructor encounters an error while cleaning up resources, log the error or handle it silently instead of throwing an exception.

Example:
```cpp
class SafeResource {
public:
    ~SafeResource() noexcept {
        try {
            // Clean up resources
        } catch (...) {
            // Log the error, but don't throw exceptions
        }
    }
};
```

#### When to mark `noexcept`:
- You should **always mark destructors `noexcept`** (or let them implicitly be `noexcept`).
  
Example:
```cpp
class SimpleResource {
public:
    ~SimpleResource() noexcept {
        // Clean up resources safely
    }
};
```

### **Copy Constructors and Copy Assignment Operators: Should They Throw or Be `noexcept`?**

- **Copy constructors and copy assignment operators** can throw exceptions, especially if they perform deep copies of dynamically allocated resources. For example, if copying an object requires allocating memory and the allocation fails, it’s appropriate to throw an exception.
  
#### When to throw:
- If copying an object involves resource allocation or operations that can fail, it's reasonable for the copy constructor or assignment operator to throw exceptions.

Example:
```cpp
class DeepCopy {
    int* data;
public:
    DeepCopy(const DeepCopy& other) {
        data = new int(*other.data);  // May throw if memory allocation fails
    }

    DeepCopy& operator=(const DeepCopy& other) {
        if (this != &other) {
            delete data;
            data = new int(*other.data);  // May throw
        }
        return *this;
    }
};
```

#### When to mark `noexcept`:
- If your copy constructor or assignment operator does not perform any operations that can throw, you should mark them `noexcept`. Marking them `noexcept` allows standard containers (like `std::vector`, `std::map`, etc.) to take advantage of optimizations.

Example:
```cpp
class TrivialCopy {
public:
    TrivialCopy(const TrivialCopy&) noexcept = default;  // No dynamic memory, so noexcept
    TrivialCopy& operator=(const TrivialCopy&) noexcept = default;
};
```

#### Guidelines:
- If the copy constructor/assignment operator **can throw**, do not mark them `noexcept`.
- If the copy constructor/assignment operator **cannot throw** (e.g., shallow copying of primitive types), mark them `noexcept`.

### **Move Constructors and Move Assignment Operators: Should They Throw or Be `noexcept`?**

- **Move constructors and move assignment operators should be `noexcept` whenever possible**. This is crucial for efficiency, especially in standard containers like `std::vector` or `std::deque`. These containers often assume that moving an object will not throw, and if a move constructor or move assignment operator is not `noexcept`, they will fall back to using the copy constructor, which can be much slower.
  
#### When to throw:
- Ideally, **move constructors and move assignment operators should never throw**. However, if moving resources could fail (e.g., if moving requires dynamic memory allocation), it may throw. But such cases are rare, as the essence of moving is to transfer ownership cheaply.
  
#### When to mark `noexcept`:
- You should **always mark move constructors and move assignment operators `noexcept`** if possible. If a move constructor or assignment operator throws, certain standard containers may lose their strong exception guarantees and fall back to copying.

Example:
```cpp
class Movable {
    int* data;
public:
    Movable(Movable&& other) noexcept : data(other.data) {
        other.data = nullptr;  // Transfer ownership
    }

    Movable& operator=(Movable&& other) noexcept {
        if (this != &other) {
            delete data;
            data = other.data;
            other.data = nullptr;
        }
        return *this;
    }
};
```

#### Guidelines:
- **Move constructors and move assignment operators should be `noexcept`** unless there’s a compelling reason to throw. Not marking them `noexcept` can result in significant performance degradation in standard containers that use move semantics.

### 5. **Summary of Best Practices:**

| Function Type                         | Should Throw Exceptions? | `noexcept`?                                |
|---------------------------------------|--------------------------|--------------------------------------------|
| **Constructors**                      | Can throw (e.g., resource acquisition failures) | Mark `noexcept` if no throwing operations are involved. |
| **Destructors**                       | Should not throw         | Always `noexcept`.                         |
| **Copy Constructors/Assignment**      | Can throw (e.g., resource allocation) | Mark `noexcept` if no throwing operations are involved. |
| **Move Constructors/Assignment**      | Should not throw         | Should always be `noexcept`.               |



###  **`new` vs `malloc`**

Both `new` and `malloc` are used for dynamic memory allocation in C++ and C, respectively, but they differ significantly in terms of functionality and usage. Here are the key differences:

| **Feature**         | **`new`**                                              | **`malloc`**                                |
|---------------------|--------------------------------------------------------|---------------------------------------------|
| **Type Safety**      | `new` automatically calls the constructor and returns a pointer of the correct type. | `malloc` returns a `void*`, requiring explicit casting to the desired type. |
| **Initialization**   | `new` initializes the allocated object (calls the constructor for class types). | `malloc` does **not** initialize the memory (leaves it uninitialized). |
| **Syntax**           | `Type* ptr = new Type(arguments);`                     | `Type* ptr = (Type*) malloc(sizeof(Type));` |
| **Memory Allocation**| Allocates memory from the heap and calls the constructor. | Allocates memory from the heap, but does **not** call constructors. |
| **Deallocation**     | Uses `delete` or `delete[]` to deallocate memory and call the destructor. | Uses `free()` to deallocate memory, but does **not** call destructors. |
| **Overloading**      | The `new` and `delete` operators can be overloaded.    | `malloc` and `free` cannot be overloaded in user code. |
| **Usage in C++**     | Preferred in C++ because it is type-safe, calls constructors, and integrates with object-oriented design. | C-style memory management. Still supported in C++ but less frequently used. |

#### **Example: Using `new` and `malloc`**
```cpp
// Using new
int* p = new int(10);  // Allocates and initializes an int with value 10
delete p;              // Deallocates memory and calls the destructor (if applicable)

// Using malloc
int* q = (int*) malloc(sizeof(int));  // Allocates memory, but no initialization
*q = 10;  // Must manually initialize the value
free(q);  // Frees memory, but does not call destructors
```

### 2. **What is Placement New?**

**Placement `new`** is a variant of the `new` operator that allows you to **construct an object at a specific memory location**. It does not allocate memory; instead, it takes a pointer to pre-allocated memory and constructs an object in that location.

#### Syntax of Placement New:
```cpp
void* memory = ::operator new(sizeof(Type));  // Allocate raw memory
Type* obj = new (memory) Type();  // Construct the object at the allocated memory
```

#### Use Cases:
- **Memory Pools**: Placement `new` is used when you want to allocate a large chunk of memory and then construct objects in that pre-allocated space. This is common in real-time systems or high-performance scenarios to avoid frequent memory allocations.
- **Custom Memory Management**: Sometimes used in embedded systems or games where you need full control over memory layout and allocation.
- **In-place Construction**: You might use placement `new` to construct objects in specific pre-allocated regions, such as shared memory or memory-mapped I/O.

#### Example of Placement `new`:
```cpp
#include <iostream>
#include <new>  // For std::nothrow

class MyClass {
public:
    MyClass() { std::cout << "MyClass constructed\n"; }
    ~MyClass() { std::cout << "MyClass destructed\n"; }
};

int main() {
    // Allocate raw memory
    void* memory = ::operator new(sizeof(MyClass));
    
    // Construct MyClass object at the allocated memory location
    MyClass* obj = new (memory) MyClass();

    // Manually call the destructor
    obj->~MyClass();

    // Free the memory
    ::operator delete(memory);
    
    return 0;
}
```

In this example:
- **Raw memory** is allocated using `::operator new()`.
- An object is **constructed in place** using placement `new`.
- The destructor is called explicitly, and the memory is freed using `::operator delete()`.

### **Can You Overload the `new` Operator?**

Yes, **you can overload the `new` operator** in C++. Overloading the `new` operator allows you to define custom memory allocation behavior. Similarly, you can overload the `delete` operator to define custom memory deallocation behavior.

#### Why Overload the `new` Operator?
- **Custom Memory Management**: You may want to use a custom memory pool, implement memory tracking, or handle memory alignment requirements.
- **Performance Optimizations**: By implementing a more efficient memory allocation strategy (e.g., using memory pools or stack allocation), you can reduce overhead and improve performance.
- **Debugging/Profiling**: Overloaded `new` operators can track memory usage, detect memory leaks, or provide detailed logging of memory allocations.

#### How to Overload the `new` Operator:

Overloading the `new` operator involves defining a custom version of the global or class-specific `operator new` and `operator delete` functions.

#### Example: Global `new` Operator Overload

```cpp
#include <iostream>
#include <cstdlib>  // For malloc/free

void* operator new(std::size_t size) {
    std::cout << "Custom new called, size = " << size << std::endl;
    void* p = std::malloc(size);  // Use malloc for memory allocation
    if (!p) throw std::bad_alloc();  // If allocation fails
    return p;
}

void operator delete(void* p) noexcept {
    std::cout << "Custom delete called" << std::endl;
    std::free(p);  // Use free to deallocate memory
}

int main() {
    int* p = new int(42);  // Custom new operator is called
    delete p;              // Custom delete operator is called
    return 0;
}
```

#### Example: Class-Specific `new` Operator Overload

You can also overload the `new` operator for a specific class.

```cpp
#include <iostream>
#include <cstdlib>

class MyClass {
public:
    void* operator new(std::size_t size) {
        std::cout << "MyClass custom new, size = " << size << std::endl;
        return std::malloc(size);
    }

    void operator delete(void* p) noexcept {
        std::cout << "MyClass custom delete" << std::endl;
        std::free(p);
    }
};

int main() {
    MyClass* obj = new MyClass();  // Calls MyClass custom new
    delete obj;                    // Calls MyClass custom delete
    return 0;
}
```

#### Overloading Placement `new`:
You can also overload **placement `new`**, but this is less common. The syntax for placement `new` overloads allows additional arguments, as seen in the placement `new` example above.

#### Custom `new` with Arguments:
```cpp
void* operator new(std::size_t size, const char* msg) {
    std::cout << "Custom new called with message: " << msg << std::endl;
    return std::malloc(size);
}
```
x
### Summary:

- **`new` vs `malloc`**:
  - `new` is preferred in C++ because it constructs objects, is type-safe, and integrates with destructors via `delete`. `malloc` is a C-style memory allocator that doesn’t call constructors or destructors.
  
- **Placement `new`**:
  - Allows you to construct objects at a specific memory location. It is useful for memory pools, custom memory management, and real-time systems where you need to control memory allocation and object placement.

- **Overloading `new`**:
  - You can overload the `new` and `delete` operators to customize memory allocation and deallocation behavior globally or for specific classes. This is useful for implementing memory optimizations, custom allocators, and memory tracking systems.