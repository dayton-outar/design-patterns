# Concurrency

> One of C++11’s great triumphs is the incorporation of concurrency into the language and library. Programmers familiar with other threading APIs (e.g., pthreads or Windows threads) are sometimes surprised at the comparatively Spartan feature set that C++ offers, but that’s because a great deal of C++’s support for concurrency is in the form of constraints on compiler-writers. The resulting language assurances mean that for the first time in C++’s history, programmers can write multithreaded programs with standard behavior across all platforms. This establishes a solid foundation on which expressive libraries can be built, and the concurrency elements of the Standard Library (tasks, futures, threads, mutexes, condition variables, atomic objects, and more) are merely the beginning of what is sure to become an increasingly rich set of tools for the development of concurrent C++ software.[^1]

## Prefer task-based programming to thread-based

Task-based programming is generally preferred over thread-based programming in modern C++ because it abstracts the complexities of direct thread management and allows the runtime to efficiently manage task execution. Here's why **task-based programming** is a better approach:

### **Advantages of Task-Based Programming**

1. **Simplified Code**:
   - In thread-based programming, developers must explicitly manage thread creation, synchronization, and joining. This is complex and error-prone.
   - Task-based programming uses higher-level abstractions like `std::async`, `std::future`, or thread pools, which automatically handle these details.

2. **Improved Resource Management**:
   - Threads are expensive system resources. When directly managing threads, you can easily create too many, which can degrade performance.
   - In task-based programming, tasks are distributed to a pool of worker threads, allowing the system to optimize thread usage and prevent resource overconsumption.

3. **Better Scalability**:
   - In thread-based programming, adding more threads doesn't always improve performance due to overhead like context switching.
   - Task-based programming allows the runtime to optimize task distribution across available CPU cores, improving scalability on multi-core systems.

4. **Concurrency Without Explicit Synchronization**:
   - With thread-based programming, you often need to handle synchronization manually, leading to potential bugs like data races or deadlocks.
   - Task-based programming frameworks handle concurrency internally, so you focus on the task’s logic without worrying about low-level synchronization.

5. **Portability and Flexibility**:
   - Task-based programming abstracts platform-specific threading details. A task-based system can be more portable across different hardware and operating systems.
   - Task-based solutions can easily adapt to different levels of parallelism, whether on single-core, multi-core, or even distributed systems.

---

### **Task-Based Programming in C++**

C++11 introduced task-based features like **`std::async`**, **`std::future`**, and **`std::promise`**, which provide a high-level mechanism for executing tasks concurrently.

#### **Example: Task-Based Programming with `std::async`**

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <vector>

// A function that performs a task
int compute(int x) {
    return x * x;
}

int main() {
    std::vector<std::future<int>> futures;

    // Launch tasks asynchronously
    for (int i = 1; i <= 5; ++i) {
        futures.push_back(std::async(std::launch::async, compute, i));
    }

    // Retrieve and print the results
    for (auto& future : futures) {
        std::cout << "Result: " << future.get() << std::endl;
    }

    return 0;
}
```

In this example:
- `std::async` launches tasks that compute the square of numbers. The system decides how to execute them, either in parallel or sequentially, depending on the available resources.
- We avoid explicit thread management, focusing instead on task logic.

---

### **Thread-Based Programming (for Comparison)**

To understand why task-based programming is better, here’s a thread-based version of the same program:

```cpp
#include <iostream>
#include <thread>
#include <vector>

void compute(int x, int& result) {
    result = x * x;
}

int main() {
    std::vector<std::thread> threads;
    std::vector<int> results(5);

    // Create threads to compute the squares
    for (int i = 1; i <= 5; ++i) {
        threads.emplace_back(compute, i, std::ref(results[i - 1]));
    }

    // Join the threads
    for (auto& t : threads) {
        t.join();
    }

    // Print the results
    for (int result : results) {
        std::cout << "Result: " << result << std::endl;
    }

    return 0;
}
```

This thread-based approach:
- Requires explicit management of threads and manual synchronization of results.
- Lacks the flexibility of task-based solutions, where the runtime optimizes concurrency.

---

### **Task-Based Concurrency Libraries**

For larger, more complex systems, libraries like **Intel TBB** (Threading Building Blocks) or **Microsoft's PPL** (Parallel Patterns Library) provide advanced task-based programming tools. These frameworks allow you to express parallelism at a higher level, leaving the task scheduling and thread management to the library.

---

### **Conclusion**

Task-based programming simplifies concurrency by abstracting thread management and letting the runtime system handle the complexities of task scheduling, synchronization, and resource management. It leads to cleaner, more maintainable code and improves scalability and performance. In modern C++, it's recommended to **prefer task-based programming** (via constructs like `std::async`, thread pools, or libraries like TBB) over direct thread-based programming.

## Specify `std::launch::async` if asynchronicity is essential

When using `std::async` in C++, it's important to understand that by default, the function may run asynchronously (in a separate thread) or synchronously (in the same thread). To ensure **asynchronous execution**, explicitly use `std::launch::async`.

### **Default Behavior of `std::async`**

When you call `std::async` without a launch policy, like this:

```cpp
std::async(compute, x);
```

It behaves as if the launch policy is `std::launch::async | std::launch::deferred`. This means that:
1. **Asynchronous** (`std::launch::async`): The task runs on a separate thread immediately.
2. **Deferred** (`std::launch::deferred`): The task is only executed when the result is needed (i.e., when `future.get()` is called).

This can lead to unpredictable behavior if you expect the task to run asynchronously but the runtime defers its execution instead.

### **Forcing Asynchronous Execution**

To guarantee that the task runs asynchronously (on a separate thread), you should explicitly specify `std::launch::async`:

```cpp
std::future<int> result = std::async(std::launch::async, compute, x);
```

This ensures the task begins execution in a new thread immediately, without waiting for the result to be requested.

### **Example: Forcing Asynchronous Execution**

```cpp
#include <iostream>
#include <future>
#include <thread>

int compute(int x) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    return x * x;
}

int main() {
    // Explicitly specify std::launch::async to force asynchronous execution
    std::future<int> result = std::async(std::launch::async, compute, 10);

    std::cout << "Task is running asynchronously...\n";
    
    // Do other work while the task runs in a separate thread
    std::cout << "Doing other work...\n";

    // Retrieve the result (this may block until the async task is finished)
    std::cout << "Result: " << result.get() << std::endl;

    return 0;
}
```

### **Why Specify `std::launch::async`?**

- **Predictable Asynchronous Execution**: By specifying `std::launch::async`, you're ensuring the task is executed immediately on a separate thread. This is crucial when your program relies on the concurrency of multiple tasks running simultaneously.
  
- **Non-blocking Behavior**: If you don't specify the policy, the task might execute in a deferred manner, only running when you call `future.get()`. This can cause blocking and negate the benefits of concurrency.

- **Performance Consideration**: In some situations, deferred execution may hurt performance, especially if you expect parallelism. By forcing asynchronous execution, you take full advantage of multi-threading capabilities.

### **Conclusion**

When using `std::async` in C++, always specify `std::launch::async` if **asynchronous execution** is essential. This avoids deferred execution and guarantees that the task will run in a separate thread immediately, giving you control over concurrency and ensuring non-blocking behavior in your programs.

## Make `std::threads` unjoinable on all paths

In C++, when you create a `std::thread`, it is essential to manage its lifecycle carefully. A `std::thread` must either be **joined** or **detached** before it is destroyed, or else the program will terminate with an error. To prevent this, you should ensure that a thread is **unjoinable** (either joined or detached) in **all code paths**, including error cases and exceptions.

Here’s how you can make a `std::thread` unjoinable in all situations:

### **Key Techniques**

1. **RAII with `std::thread`**:
   - Use RAII (Resource Acquisition Is Initialization) to ensure the thread is properly joined or detached by wrapping it in a **RAII class**.

2. **Exception Safety**:
   - Ensure threads are joined or detached even in the presence of exceptions by using RAII or `try`-`catch` blocks.

### **RAII Class for `std::thread`**

One of the safest ways to manage `std::thread` is to create an RAII-style wrapper class. This ensures the thread will either be joined or detached when the object goes out of scope, even in case of exceptions.

#### **Example: RAII Wrapper for `std::thread`**

```cpp
#include <iostream>
#include <thread>

// RAII class for managing std::thread
class ThreadGuard {
    std::thread& t;

public:
    explicit ThreadGuard(std::thread& t_) : t(t_) {}

    // Destructor joins the thread if joinable
    ~ThreadGuard() {
        if (t.joinable()) {
            t.join();  // Ensure the thread is joined when going out of scope
        }
    }

    // Delete copy constructor and assignment operator
    ThreadGuard(const ThreadGuard&) = delete;
    ThreadGuard& operator=(const ThreadGuard&) = delete;
};

void threadFunction() {
    std::cout << "Thread is running\n";
}

int main() {
    std::thread t(threadFunction);

    // Ensure the thread is joined on all paths using RAII
    ThreadGuard guard(t);

    // Do other work here

    return 0;  // The thread will be joined when guard goes out of scope
}
```

- **How it works**: When the `ThreadGuard` object goes out of scope, its destructor checks if the thread is joinable and joins it if necessary.
- This approach ensures that the thread is always properly handled, even if exceptions occur or the function returns early.

### **Manually Managing Threads with `try`-`catch`**

If you don’t want to use an RAII wrapper, you can manually ensure the thread is unjoinable using `try`-`catch` blocks. However, this approach requires more manual effort and is error-prone.

#### **Example: Manually Handling Threads**

```cpp
#include <iostream>
#include <thread>

void threadFunction() {
    std::cout << "Thread is running\n";
}

int main() {
    std::thread t(threadFunction);

    try {
        // Do some work that might throw an exception
        // ...
    } catch (...) {
        // Ensure the thread is joined even if an exception occurs
        if (t.joinable()) {
            t.join();
        }
        throw;  // Re-throw the exception after joining the thread
    }

    // Ensure the thread is joined in normal execution
    if (t.joinable()) {
        t.join();
    }

    return 0;
}
```

- **How it works**: The thread is joined in the `catch` block if an exception is thrown, and in the normal path after the `try` block.
- **Drawback**: This is more verbose than using an RAII wrapper and can be error-prone if not handled correctly in all paths.

### **Best Practices**

1. **Always Check if `std::thread` is Joinable**: Before calling `join()`, always check if the thread is joinable using `t.joinable()`. This prevents double joins or attempting to join a thread that has already been detached or joined.
  
2. **Use RAII for Safety**: The best way to ensure `std::thread` is properly handled in all paths is to use an RAII-style wrapper class like `ThreadGuard`. This ensures that the thread is joined or detached automatically when the object goes out of scope, including in the presence of exceptions.

3. **Avoid Manual Thread Management if Possible**: Manual thread management using `try`-`catch` blocks works, but it is more error-prone and harder to maintain than using an RAII wrapper.

### **Conclusion**

To make `std::thread` unjoinable on all paths, the best approach is to use an **RAII wrapper** that ensures the thread is joined (or detached) when the object goes out of scope. This avoids resource leaks and ensures safe handling of threads, even in the face of exceptions. If you prefer manual thread management, always ensure that you handle both normal execution paths and exception cases, joining or detaching the thread in both scenarios.

## Be aware of varying thread handle destructor behavior

In C++, the behavior of the destructor of a `std::thread` object can lead to program termination if not handled properly. Specifically, when a `std::thread` object is destroyed and is still **joinable**, the program will terminate (`std::terminate()` is called). Therefore, it's critical to manage the lifecycle of threads carefully to avoid this issue.

Here’s what you need to be aware of:

### **Thread Handle Destructor Behavior**
- **Joinable Threads**: If a `std::thread` is still joinable when its destructor is called, the program will terminate. A thread is joinable if:
  - It has been successfully started (i.e., a function was provided to run in the thread).
  - It has not yet been joined or detached.
  
- **Unjoinable Threads**: If the thread has been joined (`t.join()`) or detached (`t.detach()`), it is considered unjoinable. Destroying an unjoinable `std::thread` does not cause termination, and the destructor behaves as expected.

### **What Happens if a Joinable `std::thread` is Destroyed?**
When the destructor of a joinable `std::thread` is invoked without having been joined or detached, `std::terminate()` is called. This happens because the runtime cannot implicitly decide whether the thread should be joined or detached—it's a design decision left to the programmer.

### **Best Practices for Handling `std::thread` Destructor Behavior**

1. **Always Make Threads Unjoinable Before Destruction**:
   - Ensure that every thread is either **joined** or **detached** before its destructor is called. Failing to do so will result in a program crash.

2. **Check `joinable()` Before Destroying the Thread**:
   - Before destroying a `std::thread`, use `t.joinable()` to check if the thread is still joinable. If it is, join or detach it.

3. **Use RAII for Thread Management**:
   - As explained earlier, using an RAII-style wrapper class like `ThreadGuard` ensures that threads are properly managed and unjoinable before they go out of scope.

4. **Decide Between `join()` and `detach()`**:
   - Use `t.join()` if you want the calling thread to wait for the thread to finish execution.
   - Use `t.detach()` if you want the thread to run independently and don't need to wait for its completion.

5. **Destructor Responsibility**:
   - If you're managing threads manually without an RAII wrapper, you must ensure that `join()` or `detach()` is called in all code paths, including in cases of exceptions or early returns.

### **Example of Proper Thread Destruction Handling**

#### **Using RAII (`ThreadGuard`)**

```cpp
#include <iostream>
#include <thread>

class ThreadGuard {
    std::thread& t;

public:
    explicit ThreadGuard(std::thread& t_) : t(t_) {}

    ~ThreadGuard() {
        if (t.joinable()) {
            t.join();  // Join the thread when the guard goes out of scope
        }
    }

    ThreadGuard(const ThreadGuard&) = delete;
    ThreadGuard& operator=(const ThreadGuard&) = delete;
};

void threadFunction() {
    std::cout << "Thread is running\n";
}

int main() {
    std::thread t(threadFunction);
    ThreadGuard guard(t);  // Ensures thread is joined at the end of scope

    // Other work here

    return 0;  // Thread will be joined automatically
}
```

#### **Manually Handling Threads**

```cpp
#include <iostream>
#include <thread>

void threadFunction() {
    std::cout << "Thread is running\n";
}

int main() {
    std::thread t(threadFunction);

    try {
        // Some work that might throw an exception
        // ...

    } catch (...) {
        // Ensure the thread is joined if an exception occurs
        if (t.joinable()) {
            t.join();
        }
        throw;  // Re-throw the exception
    }

    // Join the thread if no exception occurs
    if (t.joinable()) {
        t.join();
    }

    return 0;
}
```

### **Conclusion**

To avoid issues with `std::thread` destructor behavior:
- **Always make threads unjoinable** by calling `join()` or `detach()` before they go out of scope.
- **Check `joinable()`** before calling these methods to avoid errors.
- Use RAII techniques (like a `ThreadGuard` class) to ensure threads are automatically handled in all paths, including exceptions.

This practice will prevent your program from crashing due to `std::terminate()` being called when a joinable thread's destructor is invoked.

## Consider `void` futures for one-shot event communication

In C++ multithreading, using **`std::future<void>`** can be a clean and efficient way to implement **one-shot event communication** between threads. A `void` future is useful when you want to signal the completion of a task without returning any specific value.

### **What is a `std::future<void>`?**

A `std::future<void>` represents the result of an asynchronous operation that doesn't return a value. It is often used to signal **event completion**, typically with one thread setting the future and another thread waiting for it.

### **Why Use `std::future<void>` for One-Shot Event Communication?**

- **One-Shot Event**: Futures are naturally designed for one-shot events. Once a future is set (via a promise), it can only be queried (via `get()` or `wait()`) once. This makes it perfect for signaling when a task is complete.
  
- **Simplified API**: With a `void` future, you don’t need to deal with any return value, which simplifies the communication model when you're only concerned with task completion.

- **Thread Synchronization**: Futures provide built-in synchronization. When a thread calls `get()` or `wait()` on the future, it blocks until the result (in this case, the event) is ready.

### **Example of Using `std::future<void>` for One-Shot Event Communication**

In the following example, a thread is performing a task, and once the task is complete, it signals another thread using a `std::future<void>`.

```cpp
#include <iostream>
#include <thread>
#include <future>

// Function to perform some work and signal completion
void performWork(std::promise<void>& readyPromise) {
    std::cout << "Performing some work...\n";
    std::this_thread::sleep_for(std::chrono::seconds(2));

    // Signal the event by setting the promise
    readyPromise.set_value();
}

int main() {
    // Create a promise and get the associated future
    std::promise<void> readyPromise;
    std::future<void> readyFuture = readyPromise.get_future();

    // Launch a worker thread
    std::thread worker(performWork, std::ref(readyPromise));

    // Wait for the event (work completion) to be signaled
    std::cout << "Waiting for the work to complete...\n";
    readyFuture.get();  // Blocks until the promise is set

    std::cout << "Work completed!\n";

    // Join the worker thread
    worker.join();

    return 0;
}
```

### **Key Components in This Example**:

1. **`std::promise<void>`**: Used to signal the completion of an event (i.e., the work done by the worker thread).
2. **`std::future<void>`**: Represents the result of the asynchronous task. The `main` thread waits on this future for the completion signal.
3. **Event Communication**: The worker thread performs a task and then calls `set_value()` on the promise to signal completion. The main thread is blocked until it receives this signal by calling `get()` on the future.

### **When to Use `std::future<void>` for Event Communication?**

- **One-Time Event Notification**: It’s ideal when you need to signal the completion of a task, and that task only needs to be signaled once.
  
- **Simplicity**: If you don’t need to return a value, `void` futures simplify the API by eliminating the need for return types.

- **Single Producer, Single Consumer**: The `std::future` and `std::promise` model is built around one producer (the thread setting the promise) and one consumer (the thread waiting on the future). This makes it straightforward for one-shot signaling.

### **Alternatives**:
- **Condition Variables**: If you need more complex synchronization or signaling between multiple threads, condition variables might be more appropriate.
- **`std::shared_future<void>`**: If multiple threads need to wait for the same event, you could use a `std::shared_future` which allows multiple consumers.

### **Conclusion**

Using `std::future<void>` is an effective and simple way to implement one-shot event communication in C++ multithreading. It provides a clean and efficient way to signal task completion between threads, leveraging C++'s modern concurrency features while simplifying your code by avoiding return values.

## Use `std::atomic` for concurrency, `volatile` for special memory

In C++ programming, it's essential to differentiate between `std::atomic` and `volatile`, as they serve different purposes in the context of multithreading and memory access.

#### **1. `std::atomic` for Concurrency:**

`std::atomic` is a template provided by the C++ standard library that enables **safe access to shared variables across multiple threads**. It ensures that operations on the atomic variables are **thread-safe** without requiring explicit locks, which can reduce overhead and improve performance in concurrent applications.

- **Thread-Safe Operations**: All read/write operations on `std::atomic` variables are atomic, meaning that they will not be interrupted by other threads. This ensures **consistent** and **synchronized** access.
- **Avoids Data Races**: Using `std::atomic` prevents data races, which occur when multiple threads simultaneously access shared variables without synchronization, leading to undefined behavior.
- **Memory Ordering**: `std::atomic` provides memory ordering guarantees that help manage how operations are visible to other threads (e.g., `memory_order_relaxed`, `memory_order_acquire`, `memory_order_release`).
  
**Example:**

```cpp
#include <iostream>
#include <atomic>
#include <thread>

std::atomic<int> counter(0);

void incrementCounter() {
    for (int i = 0; i < 1000; ++i) {
        counter.fetch_add(1, std::memory_order_relaxed);  // Thread-safe increment
    }
}

int main() {
    std::thread t1(incrementCounter);
    std::thread t2(incrementCounter);

    t1.join();
    t2.join();

    std::cout << "Final counter value: " << counter.load() << std::endl;
    return 0;
}
```

- **Explanation**: In the above example, two threads increment a shared counter concurrently. Thanks to `std::atomic<int>`, no locks are required, and the final result will always be correct, with no race conditions.

#### **2. `volatile` for Special Memory:**

`volatile` is used to inform the compiler that a variable’s value can change unexpectedly, meaning it should not optimize away reads or writes to that variable. `volatile` does **not** provide any thread-safety guarantees or synchronization.

- **Memory-Mapped Hardware**: `volatile` is typically used for **special memory locations**, like memory-mapped hardware registers, where the value of the variable might change due to external factors outside the program’s control.
- **Prevent Compiler Optimizations**: The compiler will not optimize away accesses to a `volatile` variable, ensuring that the variable is re-read from memory each time it is accessed, which is useful for interacting with hardware or signal handlers.

**Example:**

```cpp
volatile int flag = 0;  // This variable might be changed externally

void checkFlag() {
    while (flag == 0) {
        // Wait until flag changes (e.g., hardware sets it)
    }
    std::cout << "Flag is set!" << std::endl;
}
```

- **Explanation**: In the above example, `flag` is marked `volatile` because it could be changed by external hardware, so the compiler will ensure it checks the variable’s value every time without optimizing the reads away.

### **Key Differences**:

| Feature               | `std::atomic`                               | `volatile`                                 |
|-----------------------|---------------------------------------------|--------------------------------------------|
| **Purpose**            | Thread-safe operations in concurrent code.  | Prevents compiler optimizations for specific memory (e.g., hardware). |
| **Thread Safety**      | Provides thread safety for shared variables. | Does not provide thread safety.            |
| **Usage**              | Multithreading to prevent data races.       | For special memory like hardware registers.|
| **Memory Ordering**    | Supports memory orderings.                  | No memory ordering guarantees.             |

### **Conclusion:**
- **Use `std::atomic`** when working with variables shared between threads to ensure thread-safe access and prevent race conditions.
- **Use `volatile`** when dealing with special memory, like hardware registers, to prevent the compiler from optimizing away critical reads/writes.

In concurrent programming, **`std::atomic`** is the correct choice for managing shared state between threads, while **`volatile`** is suited for specific low-level memory access scenarios, unrelated to multithreading.

[^1]: Chapter 7, The Concurrency API. _Effective Modern C++_ by Scott Meyers.