# Multithreading in C++


## Introduction to Threads in C++

**Threads** allow for concurrent execution of tasks within a program. In C++, a thread represents a single sequence of execution and is created to perform a specific task in parallel with other threads. The `std::thread` class from the C++11 standard allows programmers to create and manage threads easily. While every C++ program starts with a single thread running the `main()` function, additional threads can be spawned to execute other functions concurrently.

`std::thread`: https://en.cppreference.com/w/cpp/thread/thread

---

### Key Concepts:
1. **Thread**: A unit of execution that can run concurrently with others. It shares the same process memory space but has its own execution stack.
2. **Parallelism**: Threads allow for the division of tasks, making it possible to run multiple tasks in parallel, which can boost performance, especially on multi-core systems.
3. **Multithreading**: Creating multiple threads within a single program to execute different tasks at the same time.

### Thread Creation Methods in C++:

- **Function Pointers**: Pass a function pointer to the thread constructor.
- **Lambda Functions**: Pass a lambda expression (anonymous function).
- **Functors**: Use an object that defines the `operator()`.
- **Member Functions**: Call a non-static member function on an object.
- **Static Member Functions**: Call a static member function without an object.

---

### Thread Example in C++ : Addition of All Odd and Even Numbers Using Multithreading

Here’s a simple example that shows how to create and join threads in C++ using different approaches.

```cpp
#include <iostream>
#include <thread>
#include <vector>

// Function for adding odd numbers
void add_odd(long long start, long long end, long long& result) {
    for (long long i = start; i <= end; ++i) {
        if (i % 2 != 0) {
            result += i;
        }
    }
}

// Function for adding even numbers
void add_even(long long start, long long end, long long& result) {
    for (long long i = start; i <= end; ++i) {
        if (i % 2 == 0) {
            result += i;
        }
    }
}

int main() {
    long long odd_sum = 0;
    long long even_sum = 0;

    long long start = 1;
    long long end = 1900000000;

    // Create threads for odd and even number additions
    std::thread odd_thread(add_odd, start, end, std::ref(odd_sum));
    std::thread even_thread(add_even, start, end, std::ref(even_sum));

    // Wait for both threads to finish
    odd_thread.join();
    even_thread.join();

    std::cout << "Sum of odd numbers: " << odd_sum << std::endl;
    std::cout << "Sum of even numbers: " << even_sum << std::endl;

    return 0;
}
```

### Explanation:

1. **Thread Creation**: 
   - Two threads, `odd_thread` and `even_thread`, are created using function pointers. The `std::thread` constructor takes a function pointer (`add_odd` and `add_even`), along with the arguments that should be passed to the function.
   - `std::ref(odd_sum)` is used to ensure that the thread works with a reference to the result variable, not a copy.
   
2. **Thread Joining**:
   - The `join()` method blocks the main thread until the thread calling `join()` has finished execution.
   - Both threads (`odd_thread` and `even_thread`) must complete their task before the program can print the results.

### Advantages of Using Threads:

1. **Improved Performance**: If the machine has multiple cores, tasks can be executed in parallel, making better use of CPU resources.
2. **Responsiveness**: Threads allow GUI programs (like browsers, text editors) to remain responsive by offloading intensive tasks like rendering or spell-checking to other threads.
3. **Efficiency**: Tasks that are independent of each other can be done simultaneously instead of waiting for one to finish before starting the next.

---

### Nuances:

1. **Race Conditions**: Threads that access shared memory or resources need synchronization mechanisms like `std::mutex` to avoid race conditions where multiple threads modify shared data concurrently, leading to inconsistent results.
2. **Thread Safety**: Access to shared resources must be handled carefully using mutexes or other synchronization techniques to ensure thread safety.
3. **Overhead**: While threads can improve performance, they also introduce overhead due to context switching and synchronization mechanisms, which must be managed efficiently.


## Different Ways of Thread Creation and Calling in C++

C++ offers several ways to create and manage threads depending on the callable object you wish to pass. Let's explore five different methods for creating threads and calling them using various callable objects.

---

### 1. **Using Function Pointer**

This is the most basic form of creating threads, where a **function pointer** is passed to the thread. The function performs the task in a separate thread.

```cpp
#include <iostream>
#include <thread>

void fun(int x) {
    while (x-- > 0) {
        std::cout << x << std::endl;
    }
}

int main() {
    std::thread t(fun, 10);  // Create a thread that runs the function 'fun' with argument 10
    t.join();  // Wait for the thread to finish
    return 0;
}
```

- **Explanation**: A thread `t` is created to run the `fun()` function, where `x` starts at 10 and decreases until it reaches 0.
- **`t.join()`** ensures that the main thread waits for the new thread `t` to complete.

---

### 2. **Using Lambda Function**

In C++11 and later, lambda expressions can be used to create inline, anonymous functions. These can be passed directly into a thread.

```cpp
#include <iostream>
#include <thread>

int main() {
    auto fun = [](int x) {
        while (x-- > 0) {
            std::cout << x << std::endl;
        }
    };

    std::thread t(fun, 10);  // Create a thread that runs the lambda function with argument 10
    t.join();  // Wait for the thread to finish
    return 0;
}
```

- **Explanation**: A lambda function is defined inline, and a thread is created to execute it. This is a more modern and convenient method, especially for short, one-off tasks.
  
---

### 3. **Using Functor (Function Object)**

A functor (function object) is a class or struct that overloads the `()` operator, allowing instances of the class to behave like functions. Functors can be passed to threads.

```cpp
#include <iostream>
#include <thread>

class Base {
public:
    void operator()(int x) {
        while (x-- > 0) {
            std::cout << x << std::endl;
        }
    }
};

int main() {
    std::thread t((Base()), 10);  // Create a thread with a functor (Base()) and pass argument 10
    t.join();  // Wait for the thread to finish
    return 0;
}
```

- **Explanation**: The class `Base` overloads the `()` operator, making its instances callable like a function. The functor is then passed to the thread to perform the task.

---

### 4. **Using Non-Static Member Function**

To run a member function on a separate thread, you pass the function pointer along with the class object on which it should be called. The function must be non-static, meaning it requires an instance of the class.

```cpp
#include <iostream>
#include <thread>

class Base {
public:
    void run(int x) {
        while (x-- > 0) {
            std::cout << x << std::endl;
        }
    }
};

int main() {
    Base b;
    std::thread t(&Base::run, &b, 10);  // Call non-static member function 'run' on object 'b'
    t.join();  // Wait for the thread to finish
    return 0;
}
```

- **Explanation**: The `run()` function is a member function of the `Base` class. When creating the thread, we pass a pointer to the function `&Base::run` and a reference to the object `&b` to call it on.

---

### 5. **Using Static Member Function**

Static member functions do not require an object instance, so you can directly pass the static function pointer to the thread without needing a class instance.

```cpp
#include <iostream>
#include <thread>

class Base {
public:
    static void run(int x) {
        while (x-- > 0) {
            std::cout << x << std::endl;
        }
    }
};

int main() {
    std::thread t(&Base::run, 10);  // Call static member function 'run' directly
    t.join();  // Wait for the thread to finish
    return 0;
}
```

- **Explanation**: Since `run()` is a static function, it can be called without an object instance. You only need the function pointer `&Base::run` to start the thread.

---


### 1. **`join()`**

The `join()` function is used to wait for a thread to complete. When you call `join()` on a thread, the main thread (or the calling thread) pauses its execution and waits for the specified thread to finish its execution. 

#### Key Points:
- **One-Time Use**: You can only call `join()` on a thread once. A second call will result in a runtime error (`std::system_error`).
- **Blocking**: The calling thread (main or parent) is blocked until the thread being joined completes.
- **Joinable Check**: Before calling `join()`, you should check whether the thread is **joinable** using the `joinable()` function.

#### Example:

```cpp
#include <iostream>
#include <thread>

void fun() {
    for (int i = 0; i < 5; ++i) {
        std::cout << "Thread: " << i << std::endl;
    }
}

int main() {
    std::thread t(fun);
    
    if (t.joinable()) {
        t.join();  // Wait for the thread to finish
    }
    
    return 0;
}
```

- **Explanation**: The thread `t` runs the function `fun()`, and the main thread waits for `t` to complete by calling `join()`. We first check if the thread is joinable using `joinable()`.

---

### 2. **`detach()`**

The `detach()` function is used to **detach** a thread from the main thread. Once detached, the thread executes independently from the main thread and runs in the background. The main thread does not wait for the detached thread to finish.

#### Key Points:
- **Background Execution**: The detached thread runs independently. The parent thread doesn’t wait for it.
- **Cannot Join After Detach**: Once a thread is detached, you cannot call `join()` on it.
- **Thread Lifecycle**: If the main function returns before the detached thread finishes, the program may terminate abruptly, and the detached thread may not finish its task.
- **Double Detach Issue**: Just like `join()`, you cannot call `detach()` twice on the same thread object.

#### Example:

```cpp
#include <iostream>
#include <thread>

void fun() {
    for (int i = 0; i < 5; ++i) {
        std::cout << "Thread: " << i << std::endl;
    }
}

int main() {
    std::thread t(fun);
    
    if (t.joinable()) {
        t.detach();  // Detach the thread to allow it to run independently
    }
    
    // The main thread may finish execution before the detached thread completes
    std::cout << "Main thread finished." << std::endl;
    
    return 0;
}
```

- **Explanation**: The thread `t` is detached, so the main thread does not wait for it to finish. The detached thread runs in the background, but if the main function exits before the detached thread completes, the program might terminate abruptly, causing the detached thread to be suspended.

---

### 3. **`joinable()`**

The `joinable()` function checks if a thread can be joined or detached. A thread is **joinable** if it has been created but not yet joined or detached.

#### Key Points:
- **Ensures Safe Operations**: Always check if a thread is joinable before calling `join()` or `detach()` to avoid runtime errors.
- **Thread Lifecycle**: A thread becomes non-joinable after it has been either joined or detached.


---

### Important Notes on Thread Management:
- **Destructor Behavior**: If neither `join()` nor `detach()` is called on a thread object before the thread object is destroyed (i.e., when it goes out of scope), the destructor of the thread object will terminate the program (`std::terminate()` will be called). This is because the destructor checks if the thread is still joinable and terminates the program if it is.
  
  ```cpp
  std::terminate();  // Happens if the thread is still joinable at destruction
  ```

- **One-Time Use**: You must call either `join()` or `detach()` on every thread object before its destructor is invoked, and you can call either of them only once per thread.

### Example Illustrating Join, Detach, and Joinable:

```cpp
#include <iostream>
#include <thread>

void fun() {
    for (int i = 0; i < 5; ++i) {
        std::cout << "Thread: " << i << std::endl;
    }
}

int main() {
    std::thread t(fun);
    
    if (t.joinable()) {
        // You can choose either join or detach, but not both.
        t.join();  // Option 1: Join the thread and wait for it to finish
        // t.detach();  // Option 2: Detach the thread to run independently
    }
    
    // Checking if the thread is still joinable after calling join/detach
    if (!t.joinable()) {
        std::cout << "Thread is no longer joinable." << std::endl;
    }
    
    return 0;
}
```

- **Explanation**: The thread `t` runs the `fun()` function. We check if it’s joinable before calling `join()`. After joining, the thread is no longer joinable, and we confirm this with another `joinable()` check.

---


## Mutex in C++ Threading

A **mutex** (short for mutual exclusion) is a synchronization primitive used to protect shared data from being simultaneously accessed or modified by multiple threads. When multiple threads attempt to access shared resources concurrently, race conditions can occur, which can lead to inconsistent or erroneous results. Mutexes provide a way to ensure that only one thread can access the critical section of code at a time.

`std::mutex`: https://en.cppreference.com/w/cpp/thread/mutex

---

### Race Condition

A **race condition** occurs when two or more threads access shared data at the same time, and the final outcome depends on the timing of their execution. This can lead to unpredictable and inconsistent results, especially when multiple threads are modifying the same data.

#### Example of Race Condition:

```cpp
#include <iostream>
#include <thread>

int counter = 0;

void increment() {
    for (int i = 0; i < 100000; ++i) {
        ++counter;  // Shared resource
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);

    t1.join();
    t2.join();

    std::cout << "Final counter value: " << counter << std::endl;
    return 0;
}
```

- **Explanation**: In this example, two threads `t1` and `t2` increment a shared variable `counter` 100,000 times each. The final value of `counter` should be 200,000, but due to the **race condition**, the actual value may vary, because both threads may read and write to `counter` at the same time, corrupting the final result.

---

### Critical Section

A **critical section** is a part of the code that accesses shared resources (e.g., variables, data structures) that must not be accessed by more than one thread at a time. The goal is to **protect** the critical section from concurrent access by multiple threads to avoid race conditions.

To protect the critical section, we can use a **mutex**.

---

### Mutex: Mutual Exclusion

A **mutex** is used to enforce mutual exclusion and prevent race conditions. It ensures that only one thread can execute the critical section at a time. The basic operations for a mutex are:

- **lock()**: A thread locks the mutex before entering the critical section, ensuring no other thread can access the critical section until the mutex is unlocked.
- **unlock()**: After the thread finishes executing the critical section, it unlocks the mutex, allowing other threads to acquire the lock and enter the critical section.

#### Example Using Mutex to Prevent Race Condition:

```cpp
#include <iostream>
#include <thread>
#include <mutex>

int counter = 0;
std::mutex mtx;

void increment() {
    for (int i = 0; i < 100000; ++i) {
        mtx.lock();  // lock the mutex
        ++counter;
        mtx.unlock();  // unlock the mutex
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);

    t1.join();
    t2.join();

    std::cout << "Final counter value: " << counter << std::endl;
    return 0;
}
```

- **Explanation**: In this example, we  call `mtx.lock()` before the critical section and `mtx.unlock()` afterward to make sure only one thread can access the critical section at a time.

---


### `std::mutex::try_lock()` in C++11 Threading

The `try_lock()` function in C++11 allows a thread to attempt to lock a mutex without blocking. This is particularly useful in situations where you want to perform some other work if the mutex cannot be immediately locked, rather than waiting for it to be available.

#### Key Points:
1. **Non-blocking**: If the mutex is already locked by another thread, `try_lock()` returns immediately without blocking the current thread.
2. **Return Value**: If `try_lock()` successfully locks the mutex, it returns `true`; otherwise, it returns `false`.
3. **Undefined Behavior**: If the same thread that owns the mutex calls `try_lock()` again, the behavior is undefined. This can result in a deadlock. For this kind of scenario, `std::recursive_mutex` should be used.

#### Example of `std::mutex::try_lock()`:

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;

void tryLockExample(int id) {
    if (mtx.try_lock()) {
        std::cout << "Thread " << id << " acquired the lock." << std::endl;
        std::this_thread::sleep_for(std::chrono::seconds(1));
        mtx.unlock();
    } else {
        std::cout << "Thread " << id << " could not acquire the lock." << std::endl;
    }
}

int main() {
    std::thread t1(tryLockExample, 1);
    std::thread t2(tryLockExample, 2);

    t1.join();
    t2.join();

    return 0;
}
```

- **Explanation**: In this example, two threads (`t1` and `t2`) try to lock the same mutex using `try_lock()`. The first thread to successfully lock the mutex holds it for one second, then unlocks it. If the second thread fails to acquire the lock, it immediately returns without blocking.
- **Non-blocking Behavior**: `try_lock()` allows the thread to immediately continue if the mutex is not available, making it useful for concurrent programming where tasks can be interleaved.

---

### `std::try_lock()` for Multiple Mutexes

The `std::try_lock()` function allows you to attempt to lock multiple mutexes at once. It tries to lock each mutex in the order they are passed as arguments. If it fails to lock any of the mutexes, it will unlock all previously locked mutexes and return the index of the mutex that could not be locked.

`std::try_lock()`: https://en.cppreference.com/w/cpp/thread/try_lock

#### Key Points:
1. **Locks Multiple Mutexes**: It attempts to lock all mutexes passed to it.
2. **Return Value**: It returns `-1` on success, meaning all mutexes were successfully locked. Otherwise, it returns the index (starting from 0) of the mutex it failed to lock.
3. **Exception Safety**: If `std::try_lock()` fails to lock a mutex or an exception is thrown during the lock operation, it will unlock any previously locked mutexes, ensuring no resources are leaked.

#### Example of `std::try_lock()`:

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx1, mtx2;

void task(int id) {
    if (std::try_lock(mtx1, mtx2) == -1) {
        std::cout << "Thread " << id << " acquired both locks." << std::endl;
        std::this_thread::sleep_for(std::chrono::seconds(1));
        mtx1.unlock();
        mtx2.unlock();
    } else {
        std::cout << "Thread " << id << " could not acquire both locks." << std::endl;
    }
}

int main() {
    std::thread t1(task, 1);
    std::thread t2(task, 2);

    t1.join();
    t2.join();

    return 0;
}
```

- **Explanation**: In this example, both threads `t1` and `t2` attempt to acquire two mutexes (`mtx1` and `mtx2`) using `std::try_lock()`. If successful, both mutexes are locked, and the thread holds them for one second before unlocking. If any mutex cannot be locked, `std::try_lock()` releases any previously locked mutexes and returns the index of the failed lock.

#### Example with Failure Handling:

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx1, mtx2, mtx3;

void task(int id) {
    int result = std::try_lock(mtx1, mtx2, mtx3);  // Try locking all three mutexes
    if (result == -1) {
        std::cout << "Thread " << id << " acquired all locks." << std::endl;
        std::this_thread::sleep_for(std::chrono::seconds(1));
        mtx1.unlock();
        mtx2.unlock();
        mtx3.unlock();
    } else {
        std::cout << "Thread " << id << " could not acquire lock at index: " << result << std::endl;
    }
}

int main() {
    std::thread t1(task, 1);
    std::thread t2(task, 2);

    t1.join();
    t2.join();

    return 0;
}
```

- **Explanation**: Here, `std::try_lock()` attempts to lock three mutexes (`mtx1`, `mtx2`, and `mtx3`). If it fails to lock any of the mutexes, it releases any that were successfully locked and prints the index of the mutex it could not lock. This ensures that no thread holds a partial set of locks, preventing potential deadlocks.

---


### Summary:

- **`std::mutex::try_lock()`**: Tries to acquire a lock on a mutex without blocking the thread. If the mutex is already locked, it returns `false` immediately.
  - Useful in non-blocking scenarios where a thread should perform some other task if the mutex is not available.
  - Undefined behavior if the same thread calls `try_lock()` on the same mutex more than once.
  
- **`std::try_lock()`**: Tries to acquire locks on multiple mutexes in a non-blocking way.
  - Returns `-1` if successful (all mutexes locked); otherwise, returns the index of the first mutex it failed to lock.
  - Ensures that all previously locked mutexes are unlocked if it fails to acquire any mutex.

---
### Timed Mutex in C++ (std::timed_mutex)

A **timed mutex** is a synchronization primitive that adds timing capabilities to a regular `std::mutex`. In addition to the regular locking and unlocking mechanisms, `std::timed_mutex` offers time-based attempts to acquire a lock. This is useful when a thread should only wait for a lock for a certain period before giving up and doing something else.

`std::timed_mutex`: https://en.cppreference.com/w/cpp/thread/timed_mutex

#### Key Features:
1. **Blocking**: Like a regular mutex, a timed mutex blocks the thread until it acquires the lock or the specified timeout expires.
2. **Timeout Mechanism**: If the timeout period expires and the lock has not been acquired, it returns `false`; otherwise, it returns `true`.
3. **Functions**:
   - `lock()` (inherited from `std::mutex`): Blocks until the lock is acquired.
   - `try_lock()` (inherited from `std::mutex`): Attempts to lock and returns immediately (`true` if successful, `false` otherwise).
   - **`try_lock_for(duration)`**: Tries to lock the mutex for a specified duration.
   - **`try_lock_until(time_point)`**: Tries to lock the mutex until a specified time point.
   - `unlock()` (inherited from `std::mutex`): Unlocks the mutex.

---

#### Example: Using `try_lock_for`

The `try_lock_for()` function attempts to lock the mutex for a specified duration. If it successfully locks the mutex within that duration, it returns `true`. If the duration expires before the mutex can be locked, it returns `false`.

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <chrono>

std::timed_mutex tmtx;

void task(int id) {
    if (tmtx.try_lock_for(std::chrono::milliseconds(500))) {
        std::cout << "Thread " << id << " acquired the lock.\n";
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));  // Simulate some work
        tmtx.unlock();
    } else {
        std::cout << "Thread " << id << " could not acquire the lock.\n";
    }
}

int main() {
    std::thread t1(task, 1);
    std::thread t2(task, 2);

    t1.join();
    t2.join();

    return 0;
}
```

- **Explanation**: Thread `t1` attempts to lock the mutex for 500 milliseconds. If it successfully locks the mutex, it holds it for 1 second. Meanwhile, `t2` also tries to lock the mutex for 500 milliseconds. Depending on timing, `t2` might fail to acquire the lock if `t1` is still holding it.

---

#### Example: Using `try_lock_until`

The `try_lock_until()` function attempts to lock the mutex until a specific **time point** is reached. If it successfully locks the mutex before the time point, it returns `true`. Otherwise, it returns `false` if the time point is reached without acquiring the lock.

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <chrono>

std::timed_mutex tmtx;

void task(int id) {
    auto timeout_time = std::chrono::steady_clock::now() + std::chrono::milliseconds(500);
    if (tmtx.try_lock_until(timeout_time)) {
        std::cout << "Thread " << id << " acquired the lock.\n";
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));  // Simulate some work
        tmtx.unlock();
    } else {
        std::cout << "Thread " << id << " could not acquire the lock.\n";
    }
}

int main() {
    std::thread t1(task, 1);
    std::thread t2(task, 2);

    t1.join();
    t2.join();

    return 0;
}
```

- **Explanation**: Thread `t1` attempts to lock the mutex until a specific time point (500 milliseconds from the current time). `t2` also tries to lock the mutex until the same time point. Depending on the state of `t1`, `t2` might fail to acquire the lock if the time runs out.

---

### Recursive Mutex in C++ (`std::recursive_mutex`)

A **recursive mutex** allows the same thread to lock the mutex multiple times without causing a deadlock. This is particularly useful when a function needs to call itself recursively or when a function calls another function that tries to lock the same mutex.

`std::recursive_mutex`: https://en.cppreference.com/w/cpp/thread/recursive_mutex

#### Key Features:
1. **Same Thread Can Lock Multiple Times**: A recursive mutex allows the same thread to acquire the lock multiple times. However, it must release the lock the same number of times.
2. **Lock Count**: A recursive mutex keeps track of how many times it has been locked by the same thread. The thread must call `unlock()` the same number of times to completely release the lock.
3. **Undefined Maximum Locks**: There is no predefined limit on how many times the same thread can lock a recursive mutex, but the system may throw a `std::system_error` if the lock limit is reached.

---

#### Example 1: Recursive Mutex with Recursion

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::recursive_mutex rmtx;

void recursiveTask(int count) {
    if (count > 0) {
        rmtx.lock();
        std::cout << "Locked, count = " << count << std::endl;
        recursiveTask(count - 1);
        rmtx.unlock();
        std::cout << "Unlocked, count = " << count << std::endl;
    }
}

int main() {
    std::thread t1(recursiveTask, 5);
    t1.join();

    return 0;
}
```

- **Explanation**: The `recursiveTask()` function locks the recursive mutex and calls itself recursively. The same thread locks the mutex multiple times, but must also unlock it the same number of times to fully release the mutex. Without a recursive mutex, this would result in a deadlock.

---

#### Example 2: Recursive Mutex in a Loop

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::recursive_mutex rmtx;

void loopTask() {
    for (int i = 0; i < 5; ++i) {
        rmtx.lock();
        std::cout << "Locked, iteration = " << i << std::endl;
    }

    for (int i = 0; i < 5; ++i) {
        rmtx.unlock();
        std::cout << "Unlocked, iteration = " << i << std::endl;
    }
}

int main() {
    std::thread t1(loopTask);
    t1.join();

    return 0;
}
```

- **Explanation**: In this example, the same thread locks the recursive mutex 5 times in a loop, and then unlocks it 5 times. Since `std::recursive_mutex` allows the same thread to lock the mutex multiple times, this works without deadlocking.

---

### `std::lock_guard` in C++

The `std::lock_guard` is a lightweight wrapper for acquiring and releasing a mutex. It is primarily used to manage mutexes in a **scoped manner**, ensuring that the mutex is always released when the scope is exited, either through normal completion or an exception.

`std::lock_guard`: https://en.cppreference.com/w/cpp/thread/lock_guard

#### Key Features of `std::lock_guard`:
1. **Scope-based Mutex Ownership**: The mutex is locked when the `lock_guard` object is created and automatically unlocked when the `lock_guard` goes out of scope.
2. **No Manual Unlocking**: There is no need (or possibility) to explicitly unlock the mutex, as it is automatically handled by `lock_guard`'s destructor.
3. **Non-copyable**: `std::lock_guard` cannot be copied, which prevents accidental copying of the guard, ensuring that only one `lock_guard` owns the mutex at any time.

#### Example of `std::lock_guard`:

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;

void printMessage(const std::string& message) {
    std::lock_guard<std::mutex> lock(mtx);  // Lock mutex for the scope of this function
    std::cout << message << std::endl;
}

int main() {
    std::thread t1(printMessage, "Hello from thread 1");
    std::thread t2(printMessage, "Hello from thread 2");

    t1.join();
    t2.join();

    return 0;
}
```

- **Explanation**: The `std::lock_guard` object locks the mutex `mtx` when created. The mutex will be unlocked when `lock` goes out of scope at the end of the function, ensuring safe access to the shared resource (the `std::cout` stream).

---

### `std::unique_lock` in C++

`std::unique_lock` is a more flexible version of `std::lock_guard`. It provides additional features for controlling the locking behavior of mutexes. It can be used in situations where more control is required, such as timed locks, deferred locking, or transferring ownership of the lock.

`std::unique_lock`: https://en.cppreference.com/w/cpp/thread/unique_lock

#### Key Features of `std::unique_lock`:
1. **Flexible Locking**: Unlike `lock_guard`, `unique_lock` allows for more flexible locking strategies, including deferred locking (locking later), trying to lock without blocking, or adopting an already acquired lock.
2. **Timed Locking**: `unique_lock` supports time-constrained attempts at locking, such as `try_lock_for()` and `try_lock_until()`.
3. **Manual Unlocking**: With `unique_lock`, you have the ability to explicitly unlock the mutex, which allows greater control when needed.
4. **Transfer Ownership**: `unique_lock` can transfer the ownership of the mutex (via move semantics), which means the lock can be moved between different scopes or functions, unlike `lock_guard` which cannot be transferred.

#### Locking Strategies for `std::unique_lock`:
- **defer_lock**: Does not acquire the mutex when the `unique_lock` object is created, allowing you to lock it later.
- **try_to_lock**: Attempts to lock the mutex without blocking, returning immediately if the lock cannot be acquired.
- **adopt_lock**: Assumes the mutex is already locked and takes ownership without trying to lock it again.

#### Example of `std::unique_lock`:

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <chrono>

std::mutex mtx;

void task() {
    std::unique_lock<std::mutex> lock(mtx, std::defer_lock);  // Defer locking
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    
    if (lock.try_lock()) {  // Try to acquire the lock without blocking
        std::cout << "Lock acquired by task\n";
    } else {
        std::cout << "Could not acquire lock by task\n";
    }
}

int main() {
    std::thread t1(task);
    std::thread t2(task);

    t1.join();
    t2.join();

    return 0;
}
```

- **Explanation**: In this example, the `std::unique_lock` is created with the `defer_lock` option, which means the mutex is not locked immediately. The program then tries to acquire the lock using `try_lock()`, allowing the thread to attempt locking the mutex without blocking.

---

### Example: `std::unique_lock` with Multiple Features

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <chrono>

std::mutex mtx;

void task1() {
    std::unique_lock<std::mutex> lock(mtx);  // Immediate lock
    std::cout << "Task 1 has acquired the lock\n";
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << "Task 1 is releasing the lock\n";
}

void task2() {
    std::unique_lock<std::mutex> lock(mtx, std::try_to_lock);  // Try to lock
    if (lock.owns_lock()) {
        std::cout << "Task 2 acquired the lock\n";
    } else {
        std::cout << "Task 2 could not acquire the lock\n";
    }
}

int main() {
    std::thread t1(task1);
    std::this_thread::sleep_for(std::chrono::milliseconds(500));  // Small delay to let task1 acquire lock
    std::thread t2(task2);

    t1.join();
    t2.join();

    return 0;
}
```

- **Explanation**: 
  - `task1()` acquires the lock immediately and holds it for 2 seconds.
  - `task2()` tries to acquire the lock using `std::try_to_lock`. If `task1()` is still holding the lock, `task2()` will fail to acquire it.
  - This demonstrates the flexibility of `std::unique_lock` in handling different locking strategies.

---

### Comparison between `std::lock_guard` and `std::unique_lock`


| **Feature**              | **`std::lock_guard`**                       | **`std::unique_lock`**                      |
|--------------------------|---------------------------------------------|---------------------------------------------|
| **Purpose**               | Simple and lightweight mutex management     | Flexible mutex management with more control |
| **Mutex Locking**         | Locks the mutex immediately upon creation   | Can lock immediately, defer, try, or adopt  |
| **Locking Strategies**    | Only immediate locking                      | Supports `defer_lock`, `try_to_lock`, `adopt_lock` |
| **Lock Acquisition**      | Always locks upon construction              | Can lock upon construction or later         |
| **Explicit Unlocking**    | Not allowed, unlocks when the object goes out of scope | Can explicitly unlock the mutex            |
| **Ownership Transfer**    | Not supported, non-movable                  | Supported via move semantics (moveable but not copyable) |
| **Timed Locking**         | Not supported                              | Supports `try_lock_for()` and `try_lock_until()` |
| **Exception Safety**      | Automatic unlock when going out of scope (RAII) | Automatic unlock, but allows more control in error handling |
| **Use Case**              | Best for simple, scoped, automatic mutex locking | Best for more complex scenarios where more control is needed over the mutex lifecycle |
| **Overhead**              | Minimal, lightweight                        | Slightly higher overhead due to flexibility |
| **Scope-based**           | Yes (RAII pattern)                         | Yes (RAII pattern), but can manually control unlocking and locking |
| **Copyable**              | No                                          | No                                          |
| **Recursive Locking**     | No                                          | Supports recursive locking if used with `std::recursive_mutex` |


#### When to Use:
- **`std::lock_guard`** is the best choice when you need simple, reliable, and automatic locking and unlocking of a mutex within a block of code. It's lightweight and ensures the mutex is locked immediately and released automatically when it goes out of scope.
  
- **`std::unique_lock`** is ideal for more advanced scenarios where you need additional control over the mutex, such as deferred locking, timed locking, or manually unlocking and re-locking. It is also useful when transferring ownership of the mutex lock is required or when you need to work with condition variables.


## Condition Variable in C++ Threading

A **condition variable** in C++ is used to synchronize threads by allowing one or more threads to wait for a condition and another thread to notify them when the condition is met. Condition variables are particularly useful in solving problems where one thread needs to wait for another thread to produce or consume data, such as in the **Producer-Consumer Problem**.

https://en.cppreference.com/w/cpp/thread/condition_variable

#### Key Features:
1. **Notifications**: Threads can signal each other using:
   - `notify_one()`: Notifies one waiting thread.
   - `notify_all()`: Notifies all waiting threads.
   
2. **Mutex Requirement**: A condition variable must always be used in conjunction with a mutex to protect the shared data.

3. **Waiting and Notification**: 
   - A thread can wait for a condition to be met by calling `wait()` on the condition variable.
   - Another thread can notify the waiting threads that the condition is met using `notify_one()` or `notify_all()`.

4. **Producer-Consumer Problem**: One of the classic examples where condition variables are used is the Producer-Consumer problem. In this scenario, the producer generates data and the consumer consumes it. Synchronization is necessary to ensure that the consumer waits until the producer has generated data.

---

### Producer-Consumer (Bounded Buffer Problem) Using Condition Variables

The **Producer-Consumer Problem** involves two types of threads:
- The **Producer** thread generates data and places it into a buffer.
- The **Consumer** thread removes data from the buffer for processing.

#### Problem Constraints:
- The producer must wait if the buffer is full.
- The consumer must wait if the buffer is empty.
- The threads need to communicate with each other about when data is available or when space is available in the buffer, which is achieved through condition variables.

#### Example Implementation:

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>
#include <chrono>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> buffer;
const unsigned int maxBufferSize = 10;

bool done = false;

// Producer thread function
void producer(int producerID) {
    int item = 0;
    while (!done) {
        std::this_thread::sleep_for(std::chrono::milliseconds(100));  // Simulate time to produce an item
        std::unique_lock<std::mutex> lock(mtx);  // Lock the mutex
        
        // Wait until the buffer is not full
        cv.wait(lock, [] { return buffer.size() < maxBufferSize; });

        // Produce an item and add it to the buffer
        std::cout << "Producer " << producerID << " produced: " << item << std::endl;
        buffer.push(item++);
        
        // Notify the consumer that an item has been produced
        cv.notify_all();
    }
}

// Consumer thread function
void consumer(int consumerID) {
    while (!done) {
        std::this_thread::sleep_for(std::chrono::milliseconds(150));  // Simulate time to consume an item
        std::unique_lock<std::mutex> lock(mtx);  // Lock the mutex

        // Wait until the buffer is not empty
        cv.wait(lock, [] { return !buffer.empty(); });

        // Consume the item from the buffer
        int item = buffer.front();
        buffer.pop();
        std::cout << "Consumer " << consumerID << " consumed: " << item << std::endl;

        // Notify the producer that an item has been consumed
        cv.notify_all();
    }
}

int main() {
    // Create producer and consumer threads
    std::thread producerThread1(producer, 1);
    std::thread producerThread2(producer, 2);
    std::thread consumerThread1(consumer, 1);
    std::thread consumerThread2(consumer, 2);

    // Let the threads run for some time
    std::this_thread::sleep_for(std::chrono::seconds(5));

    // Set done to true to signal termination
    done = true;

    // Notify all threads to wake up and exit
    cv.notify_all();

    // Join the threads
    producerThread1.join();
    producerThread2.join();
    consumerThread1.join();
    consumerThread2.join();

    return 0;
}
```

### Explanation of the Code:

#### Producer Thread Steps:
1. **Lock the mutex**: The producer acquires the mutex to ensure exclusive access to the shared buffer.
2. **Check buffer state**: If the buffer is full, the producer waits using `cv.wait(lock, condition)` until the buffer has space.
3. **Produce an item**: Once the buffer has space, the producer generates an item and adds it to the buffer.
4. **Unlock the mutex**: The producer releases the mutex when done.
5. **Notify the consumer**: The producer uses `cv.notify_all()` to notify the consumer that data is available.

#### Consumer Thread Steps:
1. **Lock the mutex**: The consumer acquires the mutex to ensure exclusive access to the shared buffer.
2. **Check buffer state**: If the buffer is empty, the consumer waits using `cv.wait(lock, condition)` until data is available.
3. **Consume an item**: Once the buffer has data, the consumer removes an item from the buffer.
4. **Unlock the mutex**: The consumer releases the mutex when done.
5. **Notify the producer**: The consumer uses `cv.notify_all()` to notify the producer that space is available in the buffer.

---

### Key Concepts of Condition Variables in Producer-Consumer Problem:

1. **Mutex**: Used to protect the shared resource (buffer) so that only one thread can access it at a time.
2. **Condition Variable**: Used to signal between producer and consumer threads.
   - **`wait()`**: Blocks the thread until a certain condition is met.
   - **`notify_one()` or `notify_all()`**: Wakes up one or all threads waiting on the condition variable when the condition changes.
3. **Producer-Consumer Synchronization**:
   - The producer adds items to the buffer and notifies the consumer.
   - The consumer waits for items to be available, consumes them, and notifies the producer if space is available.

### Condition Variables: Key Methods

- **`cv.wait(lock, condition)`**: 
   - The waiting thread will block until the condition evaluates to true.
   - This is always done while holding a lock (mutex), so the shared resource is protected during the check.
- **`cv.notify_one()`**: 
   - Wakes up one waiting thread.
- **`cv.notify_all()`**: 
   - Wakes up all threads that are waiting on the condition variable.

---


## Deadlock in Multithreading

#### What is Deadlock?

A **deadlock** occurs in multithreading when two or more threads are unable to proceed because each thread is waiting for the other to release a resource, creating a cycle of dependency that halts execution. In a deadlock, none of the threads can make progress because they are all blocked waiting on each other.

#### Conditions for Deadlock:
1. **Mutual Exclusion**: At least one resource must be held in a non-shareable mode (only one thread can use the resource at a time).
2. **Hold and Wait**: A thread holds at least one resource and is waiting to acquire additional resources held by other threads.
3. **No Preemption**: Resources cannot be forcibly taken away from a thread; they must be released voluntarily.
4. **Circular Wait**: A set of threads must be waiting for each other in a circular chain, where each thread is holding a resource and waiting for the next thread in the chain to release another resource.

#### Ways to Deal with Deadlock:
1. **Deadlock Prevention** (by avoiding one or more of the deadlock conditions).
2. **Deadlock Avoidance** (use techniques like the Banker's Algorithm to avoid unsafe states).
3. **Deadlock Detection and Recovery** (detect deadlock and take corrective actions such as terminating threads or preempting resources).
4. **Using Timeout** (set a timeout for resource requests to break the cycle).
5. **Order Locking** (acquire resources in a predefined order to prevent circular wait).
6. **Resource Hierarchy** (define a strict ordering of resources and ensure threads lock resources in the same order).

---

### `std::lock()` in C++11

The `std::lock()` function in C++11 is used to lock multiple mutexes simultaneously, preventing deadlock by ensuring no circular waiting occurs. It provides a safer way to lock multiple mutexes at once.

https://en.cppreference.com/w/cpp/thread/lock

#### Key Features:
1. **Locks Multiple Mutexes**: It locks all provided mutexes via a sequence of calls to `lock()`, `try_lock()`, or `unlock()`.
2. **Order of Locking Not Defined**: It locks the mutexes in any arbitrary order but ensures no deadlock occurs.
3. **Blocking Call**: The function blocks until all mutexes have been successfully locked.

#### Example:

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx1, mtx2;

void task1() {
    std::lock(mtx1, mtx2);  // Lock both mutexes at once
    std::cout << "Task 1 has locked both mutexes.\n";
    
    // Unlock the mutexes
    mtx1.unlock();
    mtx2.unlock();
}

void task2() {
    std::lock(mtx1, mtx2);  // Lock both mutexes at once
    std::cout << "Task 2 has locked both mutexes.\n";

    // Unlock the mutexes
    mtx1.unlock();
    mtx2.unlock();
}

int main() {
    std::thread t1(task1);
    std::thread t2(task2);

    t1.join();
    t2.join();

    return 0;
}
```

- **Explanation**: In this example, `std::lock(mtx1, mtx2)` is used to lock both mutexes `mtx1` and `mtx2` simultaneously in both tasks. This avoids deadlock, as `std::lock()` guarantees that both mutexes will be locked without causing a circular wait.


### `std::scoped_lock` in C++17

`std::scoped_lock` is a feature introduced in C++17 to simplify and improve the locking of multiple mutexes. It is designed as a replacement for the combination of `std::lock()` and `std::lock_guard`, offering a more convenient and safer way to lock multiple mutexes.

https://en.cppreference.com/w/cpp/thread/scoped_lock

#### Key Features:
1. **Multiple Mutex Locking**: `std::scoped_lock` can lock multiple mutexes at once, just like `std::lock()`, but it automatically manages unlocking when it goes out of scope.
2. **No Deadlock**: Just like `std::lock()`, it locks the mutexes in a safe order to avoid deadlock.
3. **RAII (Resource Acquisition Is Initialization)**: It automatically releases the mutexes when the `scoped_lock` object goes out of scope, ensuring proper unlocking even if an exception is thrown.
4. **Simpler Syntax**: Compared to the combination of `std::lock()` and `std::lock_guard`, `std::scoped_lock` simplifies the syntax by providing a one-step process for locking multiple mutexes.

#### Example of `std::scoped_lock`:

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx1, mtx2;

void task1() {
    std::scoped_lock lock(mtx1, mtx2);  // Lock both mutexes at once
    std::cout << "Task 1 has locked both mutexes.\n";
    // Mutexes are automatically unlocked when 'lock' goes out of scope
}

void task2() {
    std::scoped_lock lock(mtx1, mtx2);  // Lock both mutexes at once
    std::cout << "Task 2 has locked both mutexes.\n";
    // Mutexes are automatically unlocked when 'lock' goes out of scope
}

int main() {
    std::thread t1(task1);
    std::thread t2(task2);

    t1.join();
    t2.join();

    return 0;
}
```

- **Explanation**: In this example, `std::scoped_lock` locks both `mtx1` and `mtx2` safely and automatically unlocks them when the `scoped_lock` object goes out of scope. It avoids the need to manually unlock each mutex, ensuring exception safety and cleaner code.

---

### Comparison: `std::lock()` vs `std::scoped_lock`

| Feature               | **`std::lock()`**                          | **`std::scoped_lock`**                     |
|-----------------------|--------------------------------------------|--------------------------------------------|
| **Mutex Locking**      | Locks multiple mutexes in a safe order     | Locks multiple mutexes in a safe order     |
| **Manual Unlocking**   | Must use `std::lock_guard` or manual unlock| Automatic unlocking (RAII) when it goes out of scope |
| **RAII Support**       | Requires combination with `std::lock_guard`| Supports RAII directly                     |
| **Exception Safety**   | Must combine with `std::lock_guard` for RAII| Provides automatic exception safety        |
| **Syntax Complexity**  | More verbose (`std::lock()` + `std::lock_guard`) | Simpler, one-step locking                  |
| **Flexibility**        | Provides control over lock and unlock      | Less flexible, but ensures automatic unlocking |
| **Availability**       | C++11                                      | C++17                                      |

---

### Example of `std::lock()` with `std::lock_guard`:

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx1, mtx2;

void task1() {
    std::lock(mtx1, mtx2);  // Lock both mutexes at once
    std::lock_guard<std::mutex> lock1(mtx1, std::adopt_lock);  // Adopt the lock
    std::lock_guard<std::mutex> lock2(mtx2, std::adopt_lock);  // Adopt the lock
    std::cout << "Task 1 has locked both mutexes.\n";
}

void task2() {
    std::lock(mtx1, mtx2);  // Lock both mutexes at once
    std::lock_guard<std::mutex> lock1(mtx1, std::adopt_lock);  // Adopt the lock
    std::lock_guard<std::mutex> lock2(mtx2, std::adopt_lock);  // Adopt the lock
    std::cout << "Task 2 has locked both mutexes.\n";
}

int main() {
    std::thread t1(task1);
    std::thread t2(task2);

    t1.join();
    t2.join();

    return 0;
}
```

- **Explanation**: In this case, `std::lock()` locks both mutexes, and then `std::lock_guard` adopts the locks with `std::adopt_lock`. This ensures that the mutexes are properly locked and unlocked. However, this approach is more verbose than using `std::scoped_lock`.

---

### Key Differences:
1. **RAII (Automatic Unlocking)**:
   - **`std::lock()`** requires you to manually manage unlocking with `std::lock_guard` or manual calls to `unlock()`. This makes the code more complex and prone to errors.
   - **`std::scoped_lock`** automatically handles unlocking when the scope is exited, reducing the risk of forgetting to release a mutex.
   
2. **Exception Safety**:
   - **`std::lock()`** combined with `std::lock_guard` provides exception safety, but the code becomes more verbose.
   - **`std::scoped_lock`** provides built-in exception safety since it locks and unlocks in the scope automatically.
   
3. **Ease of Use**:
   - **`std::lock()`** is more complex to use because it requires additional calls to either `std::lock_guard` or manual unlocking, making the code more verbose.
   - **`std::scoped_lock`** is simpler, allowing you to lock multiple mutexes with a single object.

4. **Availability**:
   - **`std::lock()`** has been available since C++11.
   - **`std::scoped_lock`** is available starting from C++17, providing a cleaner and more modern syntax.

---

## `std::promise` and `std::future` in C++ Threading

In C++11, `std::promise` and `std::future` are used for communication between threads. They are a part of the C++ Standard Library's concurrency features and help to pass results (or exceptions) from one thread to another.

---

### `std::promise`

A **`std::promise`** is an object that holds a value or exception that can be set by one thread and retrieved by another. It is typically used by a producer thread to pass data or an exception to a consumer thread.

#### Key Features:
1. **Setting Values**: A thread can set a value into the promise using the `set_value()` method.
2. **Setting Exceptions**: A thread can signal an exception to another thread using the `set_exception()` method.

---

### `std::future`

A **`std::future`** is used to retrieve the result of an asynchronous operation. It acts as a placeholder for a value that will eventually be set by a `std::promise`. The thread that holds the `std::future` waits for the result (or an exception) to become available.

#### Key Features:
1. **Get Values**: A thread can retrieve the result from a `std::promise` using the `get()` method on the associated `std::future`.
2. **Check Availability**: The thread can check if the value is ready using `valid()` or `wait_for()`.
3. **Blocking Call**: When the `get()` method is called on `std::future`, it blocks until the promised value is available or an exception is set.

---

### Relationship Between `std::promise` and `std::future`

- **Producer (Promise)**: The `std::promise` is responsible for producing the value (or exception) and passing it to the `std::future`.
- **Consumer (Future)**: The `std::future` is responsible for consuming the value when it is ready and waits for the result.

When you create a `std::promise`, you can obtain a `std::future` that is linked to that promise using the `get_future()` method. This `std::future` can be passed to another thread, which will wait for the result from the promise.

---

### Example: `std::promise` and `std::future`

```cpp
#include <iostream>
#include <thread>
#include <future>

// Function to set the value in the promise (Producer)
void produceData(std::promise<int>& prom) {
    std::this_thread::sleep_for(std::chrono::seconds(2));  // Simulate work
    prom.set_value(10);  // Set the value in the promise
}

// Function to get the value from the future (Consumer)
void consumeData(std::future<int>& fut) {
    std::cout << "Waiting for the value..." << std::endl;
    int value = fut.get();  // Get the value (blocks until value is ready)
    std::cout << "Value received: " << value << std::endl;
}

int main() {
    // Create a promise and obtain the associated future
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();

    // Create threads for producer and consumer
    std::thread producer(produceData, std::ref(prom));
    std::thread consumer(consumeData, std::ref(fut));

    // Wait for threads to finish
    producer.join();
    consumer.join();

    return 0;
}
```

#### Explanation:
1. **Producer Thread**: The `produceData()` function simulates work (with a delay of 2 seconds) and then sets a value (`10`) into the `std::promise`.
2. **Consumer Thread**: The `consumeData()` function waits for the value using `fut.get()`, which blocks the consumer thread until the value is available in the associated `std::promise`.
3. **Synchronization**: The consumer thread is automatically synchronized with the producer thread because `std::future::get()` waits for the result.

---

### Example: Handling Exceptions with `std::promise` and `std::future`

You can also use `std::promise` and `std::future` to pass exceptions between threads.

```cpp
#include <iostream>
#include <thread>
#include <future>
#include <stdexcept>

// Function to set an exception in the promise (Producer)
void produceException(std::promise<int>& prom) {
    try {
        throw std::runtime_error("Error: Something went wrong!");  // Simulate an error
    } catch (...) {
        prom.set_exception(std::current_exception());  // Pass the exception to the promise
    }
}

// Function to get the result or exception from the future (Consumer)
void consumeData(std::future<int>& fut) {
    try {
        int value = fut.get();  // Get the value or exception
        std::cout << "Value received: " << value << std::endl;
    } catch (const std::exception& e) {
        std::cout << "Exception caught: " << e.what() << std::endl;
    }
}

int main() {
    // Create a promise and obtain the associated future
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();

    // Create threads for producer and consumer
    std::thread producer(produceException, std::ref(prom));
    std::thread consumer(consumeData, std::ref(fut));

    // Wait for threads to finish
    producer.join();
    consumer.join();

    return 0;
}
```

#### Explanation:
1. **Producer Thread**: The `produceException()` function throws an exception, and the exception is caught and passed to the `std::promise` using `set_exception()`.
2. **Consumer Thread**: The `consumeData()` function tries to retrieve the value from the `std::future`. If an exception is stored in the `std::promise`, it is rethrown when `get()` is called, and the consumer catches and handles the exception.

---

### Key Methods of `std::promise` and `std::future`

#### `std::promise`
- **`set_value(T value)`**: Sets the value of the promise that will be passed to the future.
- **`set_exception(std::exception_ptr)`**: Passes an exception to the future.
- **`get_future()`**: Returns the `std::future` associated with the promise.

#### `std::future`
- **`get()`**: Retrieves the value from the future, blocking the calling thread if the value is not yet available. If an exception is set in the promise, it rethrows the exception.
- **`valid()`**: Returns whether the future is valid and associated with a promise.
- **`wait()`**: Blocks until the value is available.
- **`wait_for(duration)`**: Waits for a specific duration for the value to be available.
- **`wait_until(time_point)`**: Waits until a specific time point for the value to be available.

---


