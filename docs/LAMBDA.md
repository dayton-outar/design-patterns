# Lambda Expressions

**Lambda expressions** are a feature in C++ that allow you to define **anonymous functions** (i.e., functions without a name) directly in your code. They can capture variables from their enclosing scope, making them useful for short, inline functionality.

---

### **Syntax**

```cpp
[capture](parameters) -> return_type {
    // Function body
};
```

- **Capture**: Defines how variables from the enclosing scope are captured (e.g., by value or reference).
- **Parameters**: Optional, just like in regular functions.
- **Return Type**: Optional; it’s inferred if omitted.
- **Function Body**: The code to be executed.

---

### **Key Elements**

1. **Capture Clause**:
   - **`[]`**: No variables are captured.
   - **`[=]`**: Capture all variables by **value**.
   - **`[&]`**: Capture all variables by **reference**.
   - **`[x]`**: Capture variable `x` by value.
   - **`[&x]`**: Capture variable `x` by reference.
   - You can mix modes: `[x, &y]` captures `x` by value and `y` by reference.

2. **Return Type**:
   - Can be omitted if the return type is obvious, or you can specify it explicitly using `->`.

3. **Mutable**:
   - By default, value captures are **const**. Use `mutable` to modify them inside the lambda:
     ```cpp
     [x]() mutable { x += 1; };
     ```

---

### **Examples**

1. **Simple Lambda** (No captures, no return type):
   ```cpp
   auto lambda = []() { return 42; };
   ```

2. **Capturing by Value**:
   ```cpp
   int x = 10;
   auto lambda = [x]() { return x + 1; };  // Captures `x` by value
   ```

3. **Capturing by Reference**:
   ```cpp
   int x = 10;
   auto lambda = [&x]() { x += 1; };  // Captures `x` by reference, modifies `x`
   ```

4. **Lambda with Parameters**:
   ```cpp
   auto lambda = [](int a, int b) { return a + b; };
   ```

5. **Explicit Return Type**:
   ```cpp
   auto lambda = [](int a, int b) -> double { return a / static_cast<double>(b); };
   ```

---

### **Use Cases**

- **Inline functionality**: Passing functions as arguments to algorithms like `std::sort`:
  ```cpp
  std::vector<int> v = {3, 1, 4};
  std::sort(v.begin(), v.end(), [](int a, int b) { return a < b; });
  ```

- **Event handling**: Using lambdas for callback functions in UI or other event-driven code.

- **Threading**: Using lambdas with `std::thread` for lightweight thread creation:
  ```cpp
  std::thread t([] { std::cout << "Hello from thread!" << std::endl; });
  t.join();
  ```

---

### **Key Tips**

- **Avoid default capture modes** (`[=]`, `[&]`) for clarity and safety.
- **Use `std::move`** for capturing move-only types, like `std::unique_ptr`.
- Prefer capturing only necessary variables to avoid performance or correctness issues.

---

### **Summary**

Lambda expressions are concise, inline functions that capture variables from their surrounding scope. They offer flexibility for use in algorithms, threading, and event handling, improving code readability and modularity. By controlling capture modes and using mutable when needed, lambdas can be both powerful and safe.

## Avoid default capture modes

**Lambdas** allow you to define **inline, anonymous functions**. These lambdas can capture variables from their enclosing scope, and C++ provides **default capture modes** to make capturing easier. However, it’s a good practice to **avoid default capture modes** for better clarity, control, and to prevent subtle bugs.

---

### **Default Capture Modes Recap**

1. **`[=]` (Capture by Value)**:
   - Captures all variables **by value** (creates a copy of the variables).
   - Example:
     ```cpp
     int x = 10;
     auto lambda = [=]() { return x + 1; }; // x is captured by value (copy)
     ```

2. **`[&]` (Capture by Reference)**:
   - Captures all variables **by reference** (uses the original variable).
   - Example:
     ```cpp
     int x = 10;
     auto lambda = [&]() { x += 1; }; // x is captured by reference (modifies the original)
     ```

While default capture modes (`[=]` and `[&]`) are convenient, they can lead to **ambiguity** and **unintended side effects**. It's often better to explicitly specify which variables you want to capture.

---

### **Why Avoid Default Capture Modes?**

1. **Implicit Behavior**:
   - When you use a default capture mode, it’s unclear which variables are captured and how. This reduces code readability and may confuse others (or yourself) when reading the code later.

   - **Example** (using `[=]`):
     ```cpp
     int x = 5;
     int y = 10;
     auto lambda = [=]() { return x + y; };  // Captures both x and y by value
     ```
     It’s not obvious that **both `x` and `y`** are being captured unless you carefully inspect the lambda’s body.

2. **Accidental Copying**:
   - When using `[=]` to capture by value, you may inadvertently create copies of large objects, leading to unnecessary performance overhead.
   - **Example**:
     ```cpp
     std::vector<int> largeVector(1000000);
     auto lambda = [=]() { return largeVector.size(); };  // largeVector is copied
     ```

3. **Unintentional Side Effects**:
   - When using `[&]` to capture by reference, you may unintentionally modify variables outside the lambda, causing unintended side effects.
   - **Example**:
     ```cpp
     int x = 5;
     auto lambda = [&]() { x += 10; };  // x is modified in the enclosing scope
     lambda();  // x is now 15
     ```

4. **Potential Dangling References**:
   - Capturing by reference can lead to **dangling references** if the lambda outlives the scope of the captured variable.
   - **Example**:
     ```cpp
     auto lambda;
     {
         int x = 10;
         lambda = [&]() { return x; };  // Capture by reference
     }  // x goes out of scope here
     // lambda now has a dangling reference to x, leading to undefined behavior if called
     ```

---

### **Best Practice: Explicit Capture Lists**

Instead of using default capture modes, explicitly specify the variables you want to capture. This improves code readability, clarity, and avoids accidental performance penalties or bugs.

#### **Example: Explicitly Capturing Variables**

Instead of `[=]` or `[&]`, explicitly capture only the necessary variables:
```cpp
int x = 5;
int y = 10;

// Explicit capture by value for x and by reference for y
auto lambda = [x, &y]() {
    return x + y;  // x is captured by value, y is captured by reference
};
```

- **Benefits**:
  - **Clarity**: It’s immediately clear which variables are captured and how.
  - **Avoids Accidental Capture**: You won’t accidentally capture variables you don’t need.
  - **Fine-Grained Control**: You can mix **value capture** and **reference capture** for different variables.

#### **When Capturing `this`**

When capturing `this` in member functions, **explicitly capture the necessary member variables** instead of capturing the entire object by reference or value:
```cpp
class MyClass {
    int data = 42;

    void myFunction() {
        auto lambda = [this]() {
            return data;  // Captures the entire object by reference
        };
    }
};
```
In the above example, capturing `this` can be dangerous if the lambda outlives the class instance. A better approach is to capture only what’s needed:
```cpp
auto lambda = [data = this->data]() { return data; };  // Capture only `data` by value
```

---

### **Additional Recommendations for Safe Capture**:

- **Use `std::move` for Move-Only Types**: If you have move-only types (e.g., `std::unique_ptr`), capture them by value and use `std::move` to transfer ownership inside the lambda.
  
  ```cpp
  std::unique_ptr<int> ptr = std::make_unique<int>(10);
  auto lambda = [p = std::move(ptr)]() { return *p; };
  ```

- **Consider `mutable` for Value Captures**: If you need to modify a value-captured variable inside the lambda, mark the lambda as `mutable`.
  
  ```cpp
  int x = 5;
  auto lambda = [x]() mutable { x += 10; return x; };  // x is modified inside the lambda
  ```

---

### **Summary**

- **Avoid default capture modes** (`[=]` and `[&]`) because they obscure which variables are captured and how, leading to potential bugs or performance issues.
- **Explicitly capture variables** with a clear capture list to improve readability and control.
- **Be cautious** with reference captures, especially if the lambda may outlive the captured variables.
- Use **value captures** when you want to ensure copies are made, and prefer to **explicitly mix capture modes** when needed.

By following these guidelines, you'll write safer, more maintainable, and clearer lambda expressions in C++.

## Use init capture to move objects into closures

**Init-capture** (introduced in C++14) allows you to initialize and capture variables directly inside the lambda’s capture clause. This is especially useful when you want to **move** objects (like `std::unique_ptr`) into a lambda, avoiding unnecessary copies.

---

### **Why Use Init-Capture for Moving Objects?**

1. **Move Semantics**: When dealing with move-only types (like `std::unique_ptr`), you can't simply capture by value because it would try to make a copy, which is not allowed for move-only types.
2. **Resource Ownership**: You might want the lambda to take ownership of an object (like a dynamically allocated resource) by moving it.

---

### **Syntax**

The syntax for init-capture is as follows:

```cpp
[variable = std::move(object)](parameters) {
    // lambda body
};
```

- **`variable`**: The name inside the lambda.
- **`std::move(object)`**: Moves the object into the lambda, transferring ownership.

---

### **Example: Moving `std::unique_ptr` into a Lambda**

Here’s how to move a `std::unique_ptr` into a lambda using init-capture:

```cpp
#include <iostream>
#include <memory>

int main() {
    std::unique_ptr<int> ptr = std::make_unique<int>(42);

    // Move the unique_ptr into the lambda using init-capture
    auto lambda = [ptr = std::move(ptr)]() {
        std::cout << "Value: " << *ptr << std::endl;
    };

    // Invoke the lambda
    lambda();

    // ptr is now nullptr as ownership has been transferred to the lambda
    return 0;
}
```

In this example:
- **`ptr = std::move(ptr)`** moves the `std::unique_ptr` into the lambda.
- Inside the lambda, `ptr` can be used safely, and the resource will be automatically released when the lambda goes out of scope.

---

### **Another Example: Moving a Complex Object**

You can also move larger, non-copyable objects into the lambda for later use:

```cpp
#include <vector>
#include <algorithm>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    auto lambda = [vec = std::move(vec)]() {
        for (int v : vec) {
            std::cout << v << " ";
        }
        std::cout << std::endl;
    };

    // The lambda now owns `vec`
    lambda();  // Outputs: 1 2 3 4 5

    return 0;
}
```

Here, the **`vec`** is moved into the lambda, and the lambda takes ownership, preventing further use of `vec` in the outer scope.

---

### **Key Points**

- **Init-capture** allows you to move objects into lambdas.
- Useful for **move-only types** (like `std::unique_ptr`) and large objects that you want to transfer ownership of.
- It avoids unnecessary copies and allows more efficient memory management.

---

### **Summary**

Use **init-capture** in lambda expressions when you need to **move objects** into the lambda's scope. This technique is particularly valuable when working with **move-only types** or when optimizing performance by transferring ownership and avoiding copying large objects.

## Use `decltype` on `auto&&` parameters to `std::forward` them

Forwarding references (`auto&&`) are commonly used in **generic functions** to preserve the value category of the arguments. You can use `decltype` in conjunction with `std::forward` to ensure that parameters are forwarded as **lvalues** or **rvalues**, depending on how they were passed to the function.

---

### **Why Use `decltype` with `auto&&`?**

- `auto&&` deduces the value category of a parameter.
  - If an **lvalue** is passed, `auto&&` deduces to an **lvalue reference**.
  - If an **rvalue** is passed, `auto&&` deduces to an **rvalue reference**.
- To ensure the correct value category is passed to subsequent function calls, we use `std::forward`, which relies on `decltype` to deduce the correct type.

---

### **How `std::forward` Works with `decltype`**

- `std::forward` requires the **exact type** of the argument to forward it correctly. `decltype` helps deduce this type.
- For **rvalues**, `std::forward<T>` will cast the argument back to an rvalue.
- For **lvalues**, it will cast back to an lvalue.

---

### **Example: Forwarding Function Arguments**

Here’s how to use `decltype` to ensure correct forwarding of function parameters:

```cpp
#include <iostream>
#include <utility>  // for std::forward

// Template function that forwards its argument
template <typename T>
void forwardFunction(T&& arg) {
    // Forward the argument using decltype to preserve its value category
    anotherFunction(std::forward<decltype(arg)>(arg));
}

// Example target function
void anotherFunction(int& x) {
    std::cout << "Lvalue reference passed: " << x << std::endl;
}

void anotherFunction(int&& x) {
    std::cout << "Rvalue reference passed: " << x << std::endl;
}

int main() {
    int x = 10;

    // Calling with lvalue
    forwardFunction(x);  // Calls the lvalue version of anotherFunction

    // Calling with rvalue
    forwardFunction(20);  // Calls the rvalue version of anotherFunction

    return 0;
}
```

### **Explanation**:

1. **`T&& arg`**: This is a **forwarding reference** (or a **universal reference**) that can bind to both lvalues and rvalues.
2. **`decltype(arg)`**: Deduces whether `arg` is an lvalue or rvalue. 
   - If `arg` is an lvalue, `decltype(arg)` results in `T&`.
   - If `arg` is an rvalue, `decltype(arg)` results in `T&&`.
3. **`std::forward<decltype(arg)>(arg)`**: Uses `decltype` to forward the argument correctly:
   - **Lvalues** are forwarded as lvalues.
   - **Rvalues** are forwarded as rvalues.

---

### **Key Points**

- **`auto&&`** in function templates is a universal reference that can bind to both lvalues and rvalues.
- **`decltype(arg)`** allows deducing whether a parameter is an lvalue or an rvalue.
- **`std::forward<decltype(arg)>(arg)`** ensures that the argument is forwarded to the correct function, preserving its value category.

---

### **Summary**

Using `decltype` with `auto&&` allows you to forward function arguments in a way that preserves their original value category (lvalue or rvalue). This is crucial for writing efficient, generic code that works with both lvalue and rvalue references, ensuring the optimal behavior when passing arguments.

## Prefer lambdas to `std::bind`

**Lambdas** are often preferred over `std::bind` for several reasons, including readability, flexibility, and performance. Here’s an overview of why you might choose lambdas over `std::bind`, along with examples.

---

### **Reasons to Prefer Lambdas**

1. **Readability**:
   - Lambdas provide a clearer and more concise syntax for defining inline functions. This makes code easier to read and understand, especially for those familiar with C++11 and later.

2. **Type Safety**:
   - Lambdas can capture variables from their surrounding scope, and the type deduction is performed at compile time. This leads to fewer issues related to type mismatches that can occur with `std::bind`.

3. **Flexibility**:
   - Lambdas allow for capturing both by value and by reference easily, making them versatile for various use cases.
   - You can directly define the function body within the lambda, making it easier to customize behavior.

4. **Performance**:
   - Lambdas can often lead to better performance because they avoid the overhead of creating a `std::function` or the additional indirection that `std::bind` may introduce.

5. **Modern Syntax**:
   - Using lambdas is more idiomatic in modern C++ and encourages a more functional programming style.

---

### **Example: Using `std::bind` vs. Lambdas**

**Using `std::bind`:**

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <functional>

void print(int value) {
    std::cout << value << std::endl;
}

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};

    // Using std::bind to bind the print function
    auto boundPrint = std::bind(print, std::placeholders::_1);

    std::for_each(numbers.begin(), numbers.end(), boundPrint);

    return 0;
}
```

**Using Lambdas:**

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};

    // Using a lambda to perform the same task
    auto lambdaPrint = [](int value) {
        std::cout << value << std::endl;
    };

    std::for_each(numbers.begin(), numbers.end(), lambdaPrint);

    return 0;
}
```

### **Analysis:**

- **Readability**: The lambda syntax is often clearer. The intention of printing each number is immediately visible.
- **Conciseness**: The lambda approach eliminates the need for `std::placeholders`, making it less verbose.
- **Flexibility**: If you want to modify the behavior, you can easily do so inside the lambda without changing the function signature or creating new binding expressions.

---

### **Conclusion**

In most cases, **lambdas** are the preferred choice over `std::bind` for creating inline functions and callbacks in C++. They enhance code readability, maintain type safety, and provide more flexibility. As C++ continues to evolve, embracing lambdas will often lead to cleaner and more efficient code.