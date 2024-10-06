# Move Semantics

> When you first learn about them, move semantics and perfect forwarding seem pretty straightforward:
> 
> - **Move Semantics** makes it possible for compilers to replace expensive copying operations with less expensive moves. In the same way that copy constructors and copy assignment operators give you control over what it means to copy objects, move constructors and move assignment operators offer control over the semantics of moving. Move semantics also enables the creation of move-only types, such as `std::unique_ptr`, `std::future`, and `std::thread`.
> 
> - **Perfect forwarding** makes it possible to write function templates that take arbitrary arguments and forward them to other functions such that the target functions receive exactly the same arguments as were passed to the forwarding functions.
> 
> Rvalue references are the glue that ties these two rather disparate features together. They’re the underlying language mechanism that makes both move semantics and perfect forwarding possible.[^1]

## Understand `std::move` and `std::forward`

In modern C++, both `std::move` and `std::forward` are essential tools for efficient memory and resource management, especially when working with **move semantics** and **perfect forwarding**. Here's an explanation of each:

---

### **1. `std::move`**

`std::move` is a cast that **converts an object into an rvalue reference**, meaning it indicates that the object can be "moved from" rather than copied. Moving an object transfers resources (like memory, file handles, etc.) from one object to another without the overhead of copying them.

However, it’s important to note that `std::move` **doesn’t actually move** anything; it only casts the object to an rvalue reference, which allows move semantics to be used if appropriate (e.g., invoking a move constructor or move assignment operator).

#### **When to Use `std::move`**
- When you want to transfer ownership of resources from one object to another (e.g., to avoid unnecessary copying).
- When you need to pass a resource that can be reused or discarded after it's moved.

#### **Example**:
```cpp
#include <iostream>
#include <vector>
#include <utility> // For std::move

int main() {
    std::vector<int> vec1 = {1, 2, 3, 4, 5};
    std::vector<int> vec2 = std::move(vec1);  // vec1's contents are moved to vec2

    std::cout << "vec1 size: " << vec1.size() << std::endl;  // vec1 is now empty
    std::cout << "vec2 size: " << vec2.size() << std::endl;  // vec2 now contains the data
    return 0;
}
```

- **Explanation**: In this example, `vec1` is moved into `vec2`, which transfers its contents to `vec2` and leaves `vec1` in a valid but empty state.

---

### **2. `std::forward`**

`std::forward` is used in **template functions** to **preserve the value category** (i.e., whether an argument is an lvalue or rvalue). It enables **perfect forwarding**, which allows a function template to forward arguments exactly as they were passed in, whether they are lvalues or rvalues.

The goal of `std::forward` is to avoid unnecessary copies or moves when forwarding arguments, while maintaining the argument's original value category.

#### **When to Use `std::forward`**
- Inside **function templates** to forward arguments to another function while preserving their lvalue or rvalue nature.
- To implement **perfect forwarding**, which avoids unnecessary copies and enables move semantics when appropriate.

#### **Example**:
```cpp
#include <iostream>
#include <utility> // For std::forward

void process(int& x) {
    std::cout << "Lvalue reference called: " << x << std::endl;
}

void process(int&& x) {
    std::cout << "Rvalue reference called: " << x << std::endl;
}

template<typename T>
void forwardExample(T&& arg) {
    process(std::forward<T>(arg));  // Perfect forwarding
}

int main() {
    int a = 10;
    forwardExample(a);        // Pass lvalue (calls lvalue version of process)
    forwardExample(20);       // Pass rvalue (calls rvalue version of process)
    return 0;
}
```

- **Explanation**: The `forwardExample` function uses `std::forward` to forward `arg` to the `process` function. If `arg` is an lvalue, `process(int&)` is called; if `arg` is an rvalue, `process(int&&)` is called. This is how perfect forwarding works.

---

### **Key Differences Between `std::move` and `std::forward`**

| Feature                 | `std::move`                           | `std::forward`                                    |
|-------------------------|---------------------------------------|---------------------------------------------------|
| **Purpose**              | Converts an object to an rvalue reference. | Perfect forwarding: preserves lvalue/rvalue nature. |
| **Use Case**             | When you explicitly want to transfer ownership (move). | In function templates to forward arguments while preserving their value category. |
| **Value Category**       | Converts to an rvalue reference (moveable object). | Forwards the original value category (lvalue/rvalue). |
| **Typical Use**          | To enable move semantics in general functions or classes. | To implement perfect forwarding in function templates. |

---

### **Conclusion**

- Use **`std::move`** to cast an object to an rvalue reference, enabling move semantics and resource transfers to avoid unnecessary copies.
- Use **`std::forward`** in **template functions** to forward arguments while preserving their original value category, allowing for **perfect forwarding**.

Understanding and applying `std::move` and `std::forward` properly can greatly enhance performance by reducing unnecessary copies and leveraging move semantics in C++ programs.

## Distinguish universal references from rvalue references

In modern C++, the terms **universal reference** and **rvalue reference** often cause confusion, but they serve different purposes. Here's a clear distinction between the two:

---

### **1. Rvalue References (`T&&`)**

Rvalue references are a specific kind of reference that can **only bind to rvalues**, which are temporary objects or literals that **do not have a name** and are eligible for **move semantics**. The primary purpose of rvalue references is to enable **move semantics**, allowing efficient resource transfers.

- **Syntax**: `T&&` (where `T` is a concrete type)
- **Binds to**: Only rvalues.
- **Use Case**: 
  - Move constructors and move assignment operators.
  - Functions that take rvalue arguments for efficiency (to avoid copying).
  
#### **Example**:
```cpp
void process(int&& x) {
    // x is an rvalue reference
    std::cout << "Rvalue reference: " << x << std::endl;
}

int main() {
    process(5);  // OK: 5 is an rvalue
    // process(a); // Error: lvalue can't bind to rvalue reference
}
```

In the above example, `5` is an rvalue, so it binds to `int&&`. An lvalue (like a named variable) cannot bind to `int&&` unless explicitly casted with `std::move`.

---

### **2. Universal References (`T&&` in Templates)**

A **universal reference** is a special kind of reference that can bind to **both lvalues and rvalues**. It only exists in **template functions** or **auto-deduced contexts**, where **type deduction** occurs. A universal reference is written as `T&&`, but its behavior depends on how it is **instantiated**:

- **Syntax**: `T&&` in a **template** or with **`auto&&`**.
- **Binds to**: Both lvalues and rvalues, depending on how the template is instantiated.
  - If the argument is an **lvalue**, `T` will be deduced as an lvalue reference, making `T&&` become `T& &` (which collapses to `T&`, an lvalue reference).
  - If the argument is an **rvalue**, `T` is deduced as a non-reference type, and `T&&` remains an rvalue reference.
- **Use Case**: Universal references are essential in **perfect forwarding**, where you want to forward arguments to another function, preserving their value category (whether they are lvalues or rvalues).

#### **Example**:
```cpp
template <typename T>
void process(T&& x) {  // x is a universal reference
    std::cout << "Value: " << x << std::endl;
}

int main() {
    int a = 10;
    process(a);       // Binds to lvalue: T = int&, T&& = int&
    process(20);      // Binds to rvalue: T = int, T&& = int&&
}
```

In this example:
- When `process(a)` is called, `a` is an lvalue, so `T` is deduced as `int&`, and the argument `x` is treated as an lvalue reference (`int&`).
- When `process(20)` is called, `20` is an rvalue, so `T` is deduced as `int`, and the argument `x` is treated as an rvalue reference (`int&&`).

---

### **Key Differences Between Universal and Rvalue References**

| **Aspect**               | **Rvalue Reference (`T&&`)**                           | **Universal Reference (`T&&` in Templates)**                |
|--------------------------|--------------------------------------------------------|-------------------------------------------------------------|
| **Context**               | Used outside of templates (e.g., in concrete functions or classes). | Only exists in template functions or `auto&&` deduced contexts. |
| **Binds To**              | Only rvalues (temporary objects or literals).          | Binds to both lvalues and rvalues (depending on type deduction). |
| **Type Deduction**        | Does not involve type deduction; always binds to rvalues. | Type deduction determines whether it binds to an lvalue or rvalue. |
| **Primary Use**           | For move semantics (e.g., move constructors, move assignments). | Perfect forwarding in template functions.                    |

---

### **Conclusion**

- **Rvalue references** (`T&&`) are used when you want to work specifically with **rvalues** (typically for move semantics), and they cannot bind to lvalues.
- **Universal references** (`T&&` in template contexts) are more flexible, as they can bind to **both lvalues and rvalues** depending on the type deduction, making them essential for **perfect forwarding**.

Understanding this distinction is crucial for writing efficient and flexible modern C++ code, especially when dealing with templates and move semantics.

## Using `std::move` on rvalue references and `std::forward` on universal references

In modern C++, both `std::move` and `std::forward` play crucial roles in **efficient resource management** and **perfect forwarding**. Here's a concise guide on when and how to use them effectively:

---

### **1. Using `std::move` on Rvalue References**

When you have an **rvalue reference** (e.g., `T&&` that only binds to rvalues), you can use `std::move` to explicitly indicate that the object’s resources can be transferred (moved) to another location. **`std::move` does not actually move the object**, but casts it to an rvalue, allowing the **move constructor** or **move assignment operator** to be invoked, thereby enabling move semantics.

- **Use Case**: When you know that the object is an rvalue (temporary or no longer needed) and you want to avoid copying it.

#### **Example**:
```cpp
#include <iostream>
#include <utility>  // for std::move

class Resource {
public:
    Resource() { std::cout << "Resource acquired\n"; }
    ~Resource() { std::cout << "Resource destroyed\n"; }
    Resource(Resource&&) { std::cout << "Resource moved\n"; }
    Resource& operator=(Resource&&) { std::cout << "Resource moved via assignment\n"; return *this; }
};

void processResource(Resource&& res) {
    Resource newRes = std::move(res);  // Use std::move to cast rvalue reference
}

int main() {
    Resource res;
    processResource(std::move(res));  // Cast to rvalue reference using std::move
}
```

- **Explanation**: In this example, the `processResource` function takes an rvalue reference to `Resource`. We use `std::move` on the rvalue reference to cast it and enable the move constructor of `Resource` to be called.

---

### **2. Using `std::forward` on Universal References**

A **universal reference** (e.g., `T&&` in template functions) can bind to both lvalues and rvalues. To maintain the original value category of the argument (i.e., whether it's an lvalue or rvalue), we use `std::forward`. It preserves the value category by conditionally casting the universal reference to an lvalue reference or rvalue reference, enabling **perfect forwarding**.

- **Use Case**: When writing a function template and you want to forward the argument to another function without changing its original value category.

#### **Example**:
```cpp
#include <iostream>
#include <utility>  // for std::forward

void process(int& x) {
    std::cout << "Lvalue reference: " << x << std::endl;
}

void process(int&& x) {
    std::cout << "Rvalue reference: " << x << std::endl;
}

template <typename T>
void forwardExample(T&& arg) {
    process(std::forward<T>(arg));  // Forward argument as lvalue or rvalue
}

int main() {
    int a = 10;
    forwardExample(a);         // Calls lvalue version of process
    forwardExample(20);        // Calls rvalue version of process
}
```

- **Explanation**: In this example, `forwardExample` takes a universal reference `T&&` and forwards it to `process` using `std::forward`. Depending on whether the argument is an lvalue (`a`) or an rvalue (`20`), the appropriate overload of `process` is called. This preserves the original type of the argument and avoids unnecessary copying or moving.

---

### **Key Takeaways**:

1. **Use `std::move` on rvalue references** to cast them to rvalues, enabling **move semantics** (e.g., invoking the move constructor or move assignment).
2. **Use `std::forward` on universal references** to forward them as either lvalues or rvalues, maintaining their original value category. This is essential for **perfect forwarding** in template functions.

By applying `std::move` and `std::forward` correctly, you ensure **efficient resource management** in C++ and avoid unnecessary copying, improving the performance of your programs.

## Avoid overloading on universal references

Overloading on **universal references** (i.e., `T&&` in templates) can lead to **unintended behavior** due to **ambiguous function calls** or **unexpected type deduction**. This happens because universal references are very flexible and can bind to both **lvalues** and **rvalues**, making it difficult for the compiler to decide which overload to call.

Here's a breakdown of why and how to avoid overloading on universal references:

---

### **1. Problem with Overloading on Universal References**

When you overload functions with universal references, the **ambiguity** arises from the fact that a universal reference can bind to both lvalues and rvalues. This can lead to:

- **Ambiguous overload resolution**: The compiler may not be able to clearly decide which overload to choose.
- **Unexpected template instantiations**: Universal references may deduce types in ways that don't match your expectations, leading to surprising behavior.

#### **Example**:
```cpp
#include <iostream>

void process(int&& x) {
    std::cout << "Rvalue reference overload" << std::endl;
}

template <typename T>
void process(T&& x) {
    std::cout << "Universal reference overload" << std::endl;
}

int main() {
    int a = 10;
    process(a);         // Ambiguous: Is this an lvalue or should it go to the template?
    process(20);        // Rvalue: May choose universal or rvalue overload
}
```

In this example:
- The overload for the rvalue reference (`int&&`) and the universal reference template (`T&&`) create ambiguity when `process(a)` is called. The compiler has difficulty deciding whether to treat `a` as an lvalue reference or instantiate the template.

---

### **2. Why to Avoid Overloading on Universal References**

- **Ambiguity**: The compiler might choose the wrong overload or fail to compile due to ambiguity in function resolution.
- **Complexity**: The behavior of universal references can be confusing, especially in complex codebases where it’s unclear which overload will be chosen.
- **Performance pitfalls**: Unnecessary overloads can result in suboptimal performance when you expect perfect forwarding but instead invoke an unintended overload.

---

### **3. Best Practice: SFINAE or `std::enable_if`**

To avoid overloading on universal references, you can use **SFINAE (Substitution Failure Is Not An Error)** or `std::enable_if` to control template instantiation based on the type properties of the arguments. This ensures that only the desired overload is selected.

#### **Example Using `std::enable_if`**:
```cpp
#include <iostream>
#include <type_traits>

template <typename T>
typename std::enable_if<std::is_rvalue_reference<T&&>::value>::type
process(T&& x) {
    std::cout << "Rvalue reference (perfect forwarding)" << std::endl;
}

template <typename T>
typename std::enable_if<!std::is_rvalue_reference<T&&>::value>::type
process(T&& x) {
    std::cout << "Lvalue reference (perfect forwarding)" << std::endl;
}

int main() {
    int a = 10;
    process(a);         // Calls lvalue reference overload
    process(20);        // Calls rvalue reference overload
}
```

In this example, `std::enable_if` is used to differentiate between lvalue and rvalue references explicitly. This avoids ambiguity in the overloads while allowing perfect forwarding to take place.

---

### **4. Key Takeaways**

1. **Avoid overloading** functions with universal references (`T&&`) unless it's absolutely necessary.
2. **Use SFINAE or `std::enable_if`** to control template instantiation and avoid ambiguity in overload resolution.
3. **Test your overloads carefully**, especially when using universal references in templates, to ensure the correct behavior in your program.

By following these best practices, you can avoid the pitfalls of overloading on universal references and ensure clearer, more predictable code.

## Familiarize yourself with alternatives to overloading on universal references

Overloading on **universal references** (i.e., `T&&` in templates) can lead to ambiguous overload resolution and unintended behavior. To avoid these pitfalls, there are several alternatives that can provide more explicit and reliable functionality while preserving flexibility. Here are the most commonly used alternatives:

---

### **1. SFINAE and `std::enable_if`**
**SFINAE** (Substitution Failure Is Not An Error) and **`std::enable_if`** are template techniques used to restrict function templates based on certain conditions, such as whether a type is an lvalue or rvalue, or whether a type has specific properties.

- **How It Helps**: Instead of relying on overloading, you control the instantiation of the template function based on traits or conditions.

#### **Example**:
```cpp
#include <iostream>
#include <type_traits>

template <typename T>
typename std::enable_if<std::is_rvalue_reference<T&&>::value>::type
process(T&& x) {
    std::cout << "Rvalue reference" << std::endl;
}

template <typename T>
typename std::enable_if<!std::is_rvalue_reference<T&&>::value>::type
process(T&& x) {
    std::cout << "Lvalue reference" << std::endl;
}

int main() {
    int a = 10;
    process(a);    // Calls lvalue version
    process(20);   // Calls rvalue version
}
```

- **Benefit**: You can precisely define which version of a template should be instantiated, avoiding overload ambiguity.

---

### **2. Tag Dispatching**

**Tag dispatching** is a technique where you use special marker types to differentiate function overloads at compile-time, allowing you to dispatch the appropriate version of a function based on the traits of the argument.

- **How It Helps**: Tag dispatching allows explicit handling of lvalue vs rvalue cases without ambiguous overloads.

#### **Example**:
```cpp
#include <iostream>
#include <utility>
#include <type_traits>

// Tag structs to distinguish lvalues and rvalues
struct LvalueTag {};
struct RvalueTag {};

// Overload based on tag type
template <typename T>
void process(T&& x, LvalueTag) {
    std::cout << "Lvalue reference" << std::endl;
}

template <typename T>
void process(T&& x, RvalueTag) {
    std::cout << "Rvalue reference" << std::endl;
}

// Main function
template <typename T>
void dispatch(T&& x) {
    if (std::is_lvalue_reference<T>::value)
        process(std::forward<T>(x), LvalueTag{});
    else
        process(std::forward<T>(x), RvalueTag{});
}

int main() {
    int a = 10;
    dispatch(a);     // Lvalue
    dispatch(20);    // Rvalue
}
```

- **Benefit**: This is explicit and avoids overload ambiguity, as the function dispatch is based on traits, clearly separating the logic for lvalues and rvalues.

---

### **3. Concepts (C++20)**

With C++20, **concepts** provide a powerful and cleaner way to constrain templates. You can specify conditions directly in the function signature, reducing complexity compared to `std::enable_if` and making the code more readable.

- **How It Helps**: Concepts let you specify template constraints directly in the function declaration, making it easy to control which function is instantiated.

#### **Example**:
```cpp
#include <iostream>
#include <concepts>

// Define a concept for rvalue references
template <typename T>
concept RvalueReference = std::is_rvalue_reference_v<T&&>;

template <typename T>
concept LvalueReference = std::is_lvalue_reference_v<T>;

// Function for rvalue references
void process(RvalueReference auto&& x) {
    std::cout << "Rvalue reference" << std::endl;
}

// Function for lvalue references
void process(LvalueReference auto&& x) {
    std::cout << "Lvalue reference" << std::endl;
}

int main() {
    int a = 10;
    process(a);     // Lvalue
    process(20);    // Rvalue
}
```

- **Benefit**: Concepts provide a clean, expressive way to write template constraints and avoid overloading issues, making your code easier to understand.

---

### **4. Perfect Forwarding Without Overloading**

Another approach is to avoid overloading entirely and write a single function template that uses **perfect forwarding** and `std::forward`. This approach handles both lvalues and rvalues efficiently without the need for overloads.

- **How It Helps**: By forwarding arguments correctly, you can avoid overloading while still supporting both lvalues and rvalues.

#### **Example**:
```cpp
#include <iostream>
#include <utility>

template <typename T>
void process(T&& x) {
    // Perfectly forward x
    if constexpr (std::is_lvalue_reference_v<T>)
        std::cout << "Lvalue reference" << std::endl;
    else
        std::cout << "Rvalue reference" << std::endl;
}

int main() {
    int a = 10;
    process(a);       // Lvalue
    process(20);      // Rvalue
}
```

- **Benefit**: This eliminates the need for overloads while still handling lvalue and rvalue references efficiently through perfect forwarding.

---

### **5. Using `std::variant` or `std::optional` for Different Types**

If your overloads exist purely to handle different types (instead of value categories like lvalues and rvalues), using a type-safe union like **`std::variant`** or **`std::optional`** can simplify your code. This can eliminate the need for overloaded functions entirely.

- **How It Helps**: Instead of overloading based on types, you can use `std::variant` to hold different types and write a single function that processes all cases.

#### **Example**:
```cpp
#include <iostream>
#include <variant>

void process(std::variant<int, std::string> v) {
    std::visit([](auto&& arg) { std::cout << arg << std::endl; }, v);
}

int main() {
    process(10);          // Works with int
    process(std::string("Hello")); // Works with string
}
```

- **Benefit**: You handle multiple types in one function without the need for overloading or templates.

---

### **Conclusion**

To avoid overloading on universal references, these alternatives provide explicit and type-safe ways to handle lvalues, rvalues, and different types:

1. **SFINAE and `std::enable_if`**: Control template instantiation based on type traits.
2. **Tag dispatching**: Use explicit tag types to guide function calls based on argument properties.
3. **Concepts (C++20)**: Constrain templates in a clean, readable way.
4. **Perfect forwarding**: Handle both lvalues and rvalues in a single function template using `std::forward`.
5. **`std::variant` or `std::optional`**: Use type-safe alternatives to overloading when dealing with multiple types.

By using these techniques, you avoid the ambiguity and complexity that can arise from overloading on universal references.

## Understand reference collapsing

**Reference collapsing** is a set of rules in C++ that determine the result when references to references are formed. It is an important concept when dealing with **template programming** and **perfect forwarding**, as it dictates how reference types combine in various situations.

In C++, you cannot have a reference to a reference. For instance, `int& &` or `int&& &` is not valid. Instead, the language provides rules to simplify these reference combinations into a single valid reference type.

---

### **The Rules of Reference Collapsing**

When combining reference types (typically in template instantiations), C++ follows **reference collapsing rules**:

1. **T& &** collapses to **T&** (lvalue reference remains lvalue reference)
2. **T& &&** collapses to **T&** (lvalue reference + rvalue reference collapses to lvalue reference)
3. **T&& &** collapses to **T&** (rvalue reference + lvalue reference collapses to lvalue reference)
4. **T&& &&** collapses to **T&&** (rvalue reference remains rvalue reference)

These rules ensure that when references are nested or deduced, you always get a valid and consistent type.

---

### **Why Reference Collapsing Happens**

Reference collapsing is primarily used in **template programming** and especially in **perfect forwarding** when working with universal references. Universal references (`T&&`) can bind to both lvalues and rvalues, so the rules of reference collapsing ensure that the correct reference type is chosen when arguments are forwarded.

---

### **Example: Reference Collapsing in Action**

Here's a simple example that shows reference collapsing in a template function:

```cpp
#include <iostream>

// Template function using a universal reference
template <typename T>
void process(T&& x) {
    if constexpr (std::is_lvalue_reference_v<T>) {
        std::cout << "Lvalue reference" << std::endl;
    } else if constexpr (std::is_rvalue_reference_v<T>) {
        std::cout << "Rvalue reference" << std::endl;
    }
}

int main() {
    int a = 10;
    process(a);         // T deduced as int&, collapses to int&
    process(20);        // T deduced as int, collapses to int&&
}
```

In this example:
- When `process(a)` is called, `T` is deduced as `int&`, and according to the rules, `T&&` collapses to `int&` (an lvalue reference).
- When `process(20)` is called, `T` is deduced as `int`, and `T&&` remains `int&&` (an rvalue reference).

---

### **Reference Collapsing with Forwarding**

Reference collapsing is most often encountered in **perfect forwarding** scenarios, where we use `std::forward` to correctly forward arguments based on their value category. This ensures that lvalue references remain lvalues, and rvalue references remain rvalues.

#### **Example**:
```cpp
#include <iostream>
#include <utility>

template <typename T>
void forwarder(T&& arg) {
    process(std::forward<T>(arg));  // std::forward preserves the value category
}

int main() {
    int a = 10;
    forwarder(a);         // T deduced as int&, forward as lvalue
    forwarder(20);        // T deduced as int, forward as rvalue
}
```

In this case:
- `forwarder(a)` deduces `T` as `int&`, and `std::forward` preserves it as an lvalue.
- `forwarder(20)` deduces `T` as `int`, and `std::forward` preserves it as an rvalue.

---

### **Key Takeaways:**

1. **Reference collapsing** simplifies reference-to-reference combinations to either an lvalue or an rvalue reference.
2. It's especially important in **template programming** and **perfect forwarding**, ensuring proper reference behavior.
3. **Universal references (`T&&`)** rely on reference collapsing to determine whether a reference should behave as an lvalue or rvalue reference.
4. **Reference collapsing rules**:
   - `T& &` -> `T&`
   - `T& &&` -> `T&`
   - `T&& &` -> `T&`
   - `T&& &&` -> `T&&`

By understanding reference collapsing, you can better control how your template code handles arguments and perfect forwarding.

## Assume that move operations are not present, not cheap, and not used

When **move operations are not present, not cheap, and not used** in C++, you should fall back on more traditional strategies involving **copy operations** and focus on **efficient memory management** and **manual resource handling**. Here's how you should approach this scenario:

---

### **1. Use Copy Semantics**

Without move operations, objects will need to rely entirely on **copy constructors** and **copy assignment operators** for transferring resources between objects. Make sure your classes implement efficient **copy constructors** and **copy assignment operators** to minimize the cost of copying large resources, like dynamic memory or file handles.

#### **Copy Constructor Example**:
```cpp
class MyClass {
public:
    int* data;
    
    // Copy constructor
    MyClass(const MyClass& other) {
        data = new int(*other.data);  // Deep copy
    }
    
    // Copy assignment operator
    MyClass& operator=(const MyClass& other) {
        if (this == &other) return *this;  // Self-assignment check
        delete data;                       // Free existing resource
        data = new int(*other.data);       // Deep copy
        return *this;
    }
    
    ~MyClass() {
        delete data;  // Clean up
    }
};
```
- **Deep Copying**: Make sure to perform **deep copies** when handling dynamically allocated memory to avoid shallow copying and double-deletion problems.
  
### **2. Use Efficient Data Structures**

Since moves are off the table, it's crucial to avoid frequent **copying of large objects**. Use efficient data structures that minimize copies:
- Use **references** (`T&`) and **pointers** (`T*`) instead of passing large objects by value.
- For large containers, use **`std::vector`** (which allows efficient memory management) or **`std::list`** (which has efficient insertion and deletion).

#### **Pass by Reference Example**:
```cpp
void process(const MyClass& obj);  // Pass by const reference to avoid copying
```

### **3. Manual Resource Management (RAII)**

Without the benefit of move semantics, managing resources (like dynamic memory, file handles, or network connections) becomes more manual. **RAII (Resource Acquisition Is Initialization)** is key here—resources should be acquired and released in constructors and destructors, ensuring no resource leaks occur.

#### **RAII Pattern**:
```cpp
class Resource {
private:
    FILE* file;

public:
    Resource(const char* filename) {
        file = fopen(filename, "r");
        if (!file) throw std::runtime_error("File could not be opened");
    }

    ~Resource() {
        if (file) fclose(file);  // Ensure cleanup in destructor
    }

    // Disable move operations
    Resource(const Resource&) = delete;
    Resource& operator=(const Resource&) = delete;
};
```

### **4. Prefer Pass by Reference for Large Objects**

If copying is expensive and move operations are not used, always prefer **passing by reference** for large objects, especially in function parameters, to avoid the overhead of copying large amounts of data.

#### **Example**:
```cpp
void process(const MyClass& obj) {
    // Function logic here, no copying occurs
}
```

### **5. Optimize Copying Strategies**

If copying cannot be avoided:
- Use **lazy copying** where possible, only copying when absolutely necessary.
- Use **reference counting** (e.g., with `std::shared_ptr`) to manage shared resources without copying.
- Consider **copy-on-write** strategies where objects are shared between multiple owners and only copied when one of them modifies the object.

---

### **Summary**:
- **Copy Semantics**: Ensure your classes have efficient copy constructors and assignment operators.
- **Efficient Data Structures**: Use `std::vector`, references, or pointers to minimize unnecessary copies.
- **RAII**: Properly manage resources using constructors and destructors.
- **Pass by Reference**: Always pass large objects by reference (`T&`) or `const T&` to avoid copying.
- **Lazy Copying & Reference Counting**: Use shared ownership models and copy only when necessary.

In scenarios where move semantics are unavailable, the goal is to reduce the need for unnecessary copying and efficiently manage resources through careful use of copy semantics and memory management.

## Familiarize yourself with perfect forwarding failure cases

**Perfect forwarding** is a powerful C++ feature that allows a function template to forward its arguments to another function while preserving the value category (whether they are lvalues or rvalues). This is typically done using **universal references** (`T&&`) and **`std::forward`**. While it's generally effective, there are some **failure cases** where perfect forwarding does not behave as expected.

Here’s an overview of the most common failure cases:

---

### **1. Ambiguity with Overloaded Functions**

One of the most common failure cases arises when forwarding to an **overloaded function**. The compiler can struggle to select the correct function overload based on the deduced type, particularly when forwarding universal references.

#### **Example**:
```cpp
void foo(int& x) { }      // Lvalue overload
void foo(int&& x) { }     // Rvalue overload

template <typename T>
void wrapper(T&& x) {
    foo(std::forward<T>(x));  // Perfect forwarding attempt
}

int main() {
    int a = 10;
    wrapper(a);   // Works: foo(int&) is called
    wrapper(10);  // Works: foo(int&&) is called

    const int b = 20;
    wrapper(b);   // Error: No matching function for foo(const int&)
}
```

- **Issue**: There's no `foo(const int&)` overload, and `std::forward<T>(b)` forwards `b` as a `const int&`. 
- **Solution**: Ensure that function overloads exist for all possible types (`const`, `volatile`, etc.) when using perfect forwarding.

---

### **2. Forwarding of `0` or `NULL`**

When forwarding `0` or `NULL`, the **type deduction** can fail because the compiler struggles to deduce the correct type. Specifically, `0` can be deduced as `int`, which may not work as intended for functions expecting pointer types.

#### **Example**:
```cpp
template <typename T>
void wrapper(T&& x) {
    bar(std::forward<T>(x));  // Forwarding attempt
}

void bar(int* ptr) { }

int main() {
    wrapper(0);  // Error: 0 deduced as int, not int*
}
```

- **Issue**: `0` is deduced as an `int`, but `bar` expects an `int*`.
- **Solution**: Use `nullptr` instead of `0` when dealing with pointers, as `nullptr` will be correctly deduced as `std::nullptr_t`.

---

### **3. Forwarding to Functions with Non-deduced Contexts**

Perfect forwarding can fail when the target function has **non-deduced contexts**, meaning that certain template arguments cannot be deduced automatically by the compiler. For instance, function pointer or member function pointer arguments are not deducible by the compiler, leading to deduction failures.

#### **Example**:
```cpp
template <typename T>
void wrapper(T&& x) {
    bar(std::forward<T>(x));  // Forwarding attempt
}

template <typename T>
void bar(void (T::*func)()) { }

void foo() { }

int main() {
    wrapper(&foo);  // Error: template argument deduction fails
}
```

- **Issue**: The function pointer `&foo` cannot be deduced correctly in the template argument.
- **Solution**: In these cases, you'll need to explicitly specify the template arguments, which can be cumbersome in perfect forwarding scenarios.

---

### **4. Forwarding Braced Initializer Lists**

Perfect forwarding of **braced initializer lists** (e.g., `{1, 2, 3}`) can fail because initializer lists have special type deduction rules. They do not behave like other types when passed to templates, as they are not "true" types themselves and can't be forwarded as arguments.

#### **Example**:
```cpp
template <typename T>
void wrapper(T&& x) {
    foo(std::forward<T>(x));  // Forwarding attempt
}

void foo(std::initializer_list<int> il) { }

int main() {
    wrapper({1, 2, 3});  // Error: Cannot deduce template argument
}
```

- **Issue**: `{1, 2, 3}` cannot be deduced as an `std::initializer_list<int>`.
- **Solution**: Explicitly pass the `std::initializer_list` to avoid deduction issues.

---

### **5. Reference Collapsing Issues**

In certain cases, reference collapsing (when references to references are formed) can lead to unexpected results in perfect forwarding. If universal references collapse incorrectly, you may end up with unintended types being forwarded, such as lvalues being treated as rvalues.

#### **Example**:
```cpp
template <typename T>
void wrapper(T&& x) {
    bar(std::forward<T>(x));  // Forwarding attempt
}

void bar(int& x) { }
void bar(int&& x) { }

int main() {
    int a = 10;
    wrapper(std::move(a));  // Calls bar(int&&) as expected

    const int b = 20;
    wrapper(std::move(b));  // Error: No matching function for const int&&
}
```

- **Issue**: Forwarding `std::move(b)` collapses to `const int&&`, but there's no matching `bar` overload for `const int&&`.
- **Solution**: Ensure that correct overloads are provided and avoid using `std::move` on const objects.

---

### **6. Forwarding of Overloaded Operators**

Forwarding to functions with **overloaded operators** can also cause issues, as operator overloading may not resolve in the way you expect due to ambiguities or incorrect type deduction.

#### **Example**:
```cpp
#include <iostream>

struct Foo {
    void operator()(int) { std::cout << "int overload\n"; }
    void operator()(double) { std::cout << "double overload\n"; }
};

template <typename T, typename U>
void wrapper(T&& func, U&& arg) {
    func(std::forward<U>(arg));  // Forwarding attempt
}

int main() {
    Foo f;
    wrapper(f, 10);  // Calls the int overload
    wrapper(f, 10.0);  // Calls the double overload
}
```

- **Issue**: If there’s ambiguity in the type being forwarded, the compiler might not select the intended overload.
- **Solution**: Ensure proper overloads exist and avoid ambiguous situations.

---

### **Conclusion:**

When using perfect forwarding, be mindful of:
- **Overload ambiguity** and ensure all relevant overloads exist.
- **Non-deducible contexts** like function pointers.
- Avoid **braced initializer lists** in perfect forwarding.
- Be cautious with **reference collapsing** and avoid unintended results with lvalue/rvalue combinations.
- Use **`nullptr`** instead of `0` for pointer types.

By understanding these failure cases, you can better manage perfect forwarding and avoid common pitfalls in template programming.

## Videos

1. [Re-inventing move semantics in modern C++ in 13 minutes](https://youtu.be/kqQ90R0_GFI?si=ON7AgK1d3fpumxDT)
2. [Move Semantics Explained - Learn Modern C++](https://youtu.be/ruHw7E71zBw?si=Vx8z9v2-Oy8vYwSU)

[^1]: Chapter 5, Rvalue References, Move Semantics, and Perfect Forwarding. _Effective Modern C++_ by Scott Meyers.