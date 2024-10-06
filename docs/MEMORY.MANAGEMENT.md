# Memory Management

Efficient memory management is crucial in C++ for writing safe, performant, and bug-free code. Here’s a summary of best practices to help you handle memory effectively:

1. **Prefer Automatic Storage (Stack) over Dynamic Allocation (Heap)**
- **Use local variables** whenever possible. Stack-allocated memory is automatically managed and faster than heap memory.
  
  ```cpp
  int x = 10;  // Stack allocation, no need to manually manage memory.
  ```

- **Heap allocations** (`new`/`delete`, `malloc`/`free`) should be minimized since they incur performance overhead and increase the risk of memory leaks.

2. **Use RAII (Resource Acquisition Is Initialization)**
- **RAII** ensures that resources (e.g., memory, file handles) are acquired in a constructor and released in a destructor. This ties resource management to the object's lifetime, preventing leaks.
  
  - Example with smart pointers (which are RAII-compliant):
    ```cpp
    std::unique_ptr<int> ptr = std::make_unique<int>(42);  // Automatically freed when out of scope
    ```

- RAII classes such as **smart pointers**, **std::vector**, and **std::string** automatically manage resources, freeing you from manual memory management.

3. **Use Smart Pointers for Dynamic Memory**
- **Smart pointers** (`std::unique_ptr`, `std::shared_ptr`, `std::weak_ptr`) manage dynamic memory automatically, preventing memory leaks and handling ownership semantics.

  - **`std::unique_ptr`**: Exclusive ownership of memory (cannot be copied).
    ```cpp
    std::unique_ptr<int> ptr = std::make_unique<int>(10);
    ```
  
  - **`std::shared_ptr`**: Shared ownership; memory is deallocated when all owners are destroyed.
    ```cpp
    std::shared_ptr<int> ptr1 = std::make_shared<int>(10);
    std::shared_ptr<int> ptr2 = ptr1;  // Shared ownership
    ```

  - **`std::weak_ptr`**: Weak reference, used to break cyclic references when combined with `std::shared_ptr`.

4. **Avoid Manual Memory Management (`new`/`delete`)**
- Prefer smart pointers or standard containers over raw pointers.
  
  - **Avoid**:
    ```cpp
    int* ptr = new int(10);  // Must manually delete
    delete ptr;              // Risk of forgetting to delete
    ```

  - **Use smart pointers**:
    ```cpp
    std::unique_ptr<int> ptr = std::make_unique<int>(10);  // Automatic cleanup
    ```

5. **Use Standard Containers**
- **Containers** like `std::vector`, `std::map`, `std::string`, etc., manage memory for you, ensuring dynamic memory is handled correctly.
  
  - **Example**:
    ```cpp
    std::vector<int> vec = {1, 2, 3};  // Automatically handles memory for elements
    ```

- Standard containers grow as needed and free memory when they go out of scope.

6. **Avoid Memory Leaks**
- **Memory leaks** occur when dynamically allocated memory is not freed, leading to wasted memory over time.

  - Use tools like **Valgrind** or **AddressSanitizer** to detect leaks.

  - **Smart pointers** eliminate the risk of leaks by handling deallocation automatically.

7. **Move Semantics for Efficient Resource Management**
- **Move semantics** (`std::move`, move constructors, move assignment operators) allow efficient resource transfer instead of costly copies, particularly useful for large objects or containers.
  
  ```cpp
  std::vector<int> v1 = {1, 2, 3};
  std::vector<int> v2 = std::move(v1);  // Transfer resources from v1 to v2
  ```

- **Prefer `emplace` over `insert`** for container insertion to avoid unnecessary copies.

8. **Use `nullptr` Instead of `NULL`**
- **`nullptr`** is a type-safe null pointer introduced in C++11. It eliminates ambiguity and ensures type safety in comparisons and function overloading.
  
  ```cpp
  int* ptr = nullptr;  // Clear and type-safe
  ```

  > ... the literal `0` is an `int`, not a pointer. If C++ finds itself looking at `0` in a context where only a pointer can be used, it’ll grudgingly interpret `0` as a null pointer, but that’s a fallback position. C++’s primary policy is that `0` is an `int`, not a pointer.
  > 
  > Practically speaking, the same is true of `NULL`. There is some uncertainty in the details in `NULL`’s case, because implementations are allowed to give `NULL` an integral type other than `int` (e.g., `long`). That’s not common, but it doesn’t really matter, because the issue here isn’t the exact type of `NULL`, it’s that neither `0` nor `NULL` has a pointer type.

  ```cpp
  void f(int);
  void f(bool);
  void f(void*);

  f(0); // calls f(int), not f(void*)

  f(NULL); // might not compile, but typically calls f(int). Never calls f(void*)
  ```

  > `nullptr`’s advantage is that it doesn’t have an integral type. To be honest, it doesn’t have a pointer type, either, but you can think of it as a pointer of _all_ types. ...
  > 
  > Calling the overloaded function `f` with `nullptr` calls the void* overload (i.e., the pointer overload), because `nullptr` can’t be viewed as anything integral:

  ```cpp
  f(nullptr); // calls f(void*) overload
  ```

9. **Avoid Dangling Pointers**
- **Dangling pointers** occur when a pointer refers to memory that has been deallocated.

  - After `delete`, always set the pointer to `nullptr` to prevent accessing freed memory:
    ```cpp
    int* ptr = new int(10);
    delete ptr;
    ptr = nullptr;  // Prevent dangling pointer
    ```

10. **Minimize Use of Global and Static Variables**
- Global and static variables persist throughout the program’s lifetime, making it harder to manage their lifetimes and avoid resource leaks.
- Prefer local or member variables with well-defined lifetimes managed by RAII.

11. **Avoid `new[]` and `delete[]` for Arrays**
- Prefer **`std::vector`** or other containers for dynamic arrays instead of manually allocating and deallocating memory using `new[]` and `delete[]`.

  - **Avoid**:
    ```cpp
    int* arr = new int[10];
    delete[] arr;
    ```

  - **Use**:
    ```cpp
    std::vector<int> arr(10);  // Manages memory automatically
    ```

12. **Handle Exceptions Carefully**
- **Memory leaks can occur** if an exception is thrown between a `new` and the corresponding `delete`. Use RAII and smart pointers to ensure memory is cleaned up even in case of exceptions.

  ```cpp
  void myFunction() {
      std::unique_ptr<int> ptr = std::make_unique<int>(42);  // No memory leak even if exception occurs
      throw std::runtime_error("Error");
  }
  ```

13. **Be Careful with `reinterpret_cast` and Pointer Arithmetic**
- **Pointer casts and arithmetic** can lead to undefined behavior if misused. Ensure you're casting only when necessary and with full understanding of the memory layout.
- Avoid manual pointer arithmetic unless absolutely necessary (e.g., in low-level code).

14. **Monitor Performance and Resource Usage**
- Tools like **Valgrind**, **AddressSanitizer**, and **profilers** can help identify memory leaks, fragmentation, and performance bottlenecks related to memory management.

---

- **Prefer automatic memory management** with local variables, smart pointers, and standard containers.
- Use **RAII** to tie resource management to object lifetimes.
- **Avoid manual memory management** (`new`/`delete`), and instead use **smart pointers** (`std::unique_ptr`, `std::shared_ptr`).
- **Use move semantics** for efficient resource transfer and avoid unnecessary copying.
- Always ensure memory is properly freed and avoid **dangling pointers** and **memory leaks**.
- Use **modern C++ features** like `nullptr` and `std::move` for safer and more efficient code.

By adhering to these best practices, you can write robust, efficient, and maintainable C++ programs while avoiding common pitfalls related to memory management.

## Copy Constructor and Move Constructor in C++

In C++, **copy constructors** and **move constructors** are special member functions used to initialize objects using another object of the same type. Understanding how and when these constructors are invoked is crucial for writing efficient C++ programs, particularly when dealing with resource management (e.g., memory, file handles) or large objects.

#### Copy Constructor

The **copy constructor** creates a new object by copying the contents of an existing object. It is invoked when:
- A new object is initialized from an existing object.
- An object is passed by value to a function.
- An object is returned by value from a function (though modern compilers can optimize this with **Return Value Optimization (RVO)**).

##### Default Copy Constructor
If you don't define a copy constructor, the compiler provides one automatically. This default copy constructor performs a **shallow copy**, meaning it copies the member variables from one object to another.

##### Syntax of Copy Constructor
```cpp
class MyClass {
public:
    MyClass(const MyClass& other);  // Copy constructor
};
```

##### Example
```cpp
class MyClass {
public:
    int x;
    MyClass(int val) : x(val) {}

    // Copy constructor
    MyClass(const MyClass& other) : x(other.x) {
        std::cout << "Copy constructor called\n";
    }
};

int main() {
    MyClass obj1(10);
    MyClass obj2 = obj1;  // Copy constructor is invoked
}
```

**Output**:
```
Copy constructor called
```

##### When to Use Copy Constructor
- When you want to make a **copy of an object**, ensuring that all its data members are duplicated.
- If your class manages resources like dynamic memory, you might need a custom copy constructor to perform **deep copying** (i.e., duplicating the resource, not just copying pointers).

#### Move Constructor

The **move constructor** is introduced in C++11 to improve performance by **moving** resources from one object to another, instead of copying them. It is invoked when:
- An object is **initialized** from a temporary (rvalue) object (e.g., `std::move()`).
- An object is returned from a function and **cannot be optimized** by Return Value Optimization (RVO).
  
Move constructors are especially useful when your class manages expensive resources (like dynamic memory, file handles, etc.) because they can **steal** the resource from the original object rather than copying it.

##### Syntax of Move Constructor
```cpp
class MyClass {
public:
    MyClass(MyClass&& other);  // Move constructor
};
```

##### Example
```cpp
class MyClass {
public:
    int* data;

    // Constructor
    MyClass(int val) {
        data = new int(val);
        std::cout << "Constructor called\n";
    }

    // Move constructor
    MyClass(MyClass&& other) noexcept : data(other.data) {
        other.data = nullptr;  // Nullify the source object's pointer
        std::cout << "Move constructor called\n";
    }

    // Destructor
    ~MyClass() {
        delete data;
    }
};

int main() {
    MyClass obj1(10);
    MyClass obj2 = std::move(obj1);  // Move constructor is invoked
}
```

**Output**:
```
Constructor called
Move constructor called
```

**Explanation**:
- The move constructor takes an rvalue reference (`MyClass&&`) to the object, allowing it to "steal" the resource (i.e., the pointer `data`) from `obj1`. 
- After the move, `obj1`'s pointer is set to `nullptr`, leaving it in a **valid but empty state**.

##### When to Use Move Constructor
- When an object is **temporary** or when you want to avoid expensive deep copies, the move constructor can transfer ownership of resources from one object to another.
- Ideal for classes that manage **dynamic memory** or other system resources like file handles, sockets, etc.

#### **Key Differences: Copy Constructor vs Move Constructor**

| **Feature**                    | **Copy Constructor**                        | **Move Constructor**                           |
|---------------------------------|---------------------------------------------|------------------------------------------------|
| **Purpose**                     | Creates a copy of an object.                | Transfers resources from one object to another.|
| **Invocation**                  | Called for lvalues (regular objects).       | Called for rvalues (temporary objects).        |
| **Performance**                 | Can be expensive (creates a deep copy).     | Efficient (avoids copying, transfers ownership).|
| **Use Case**                    | When you need a duplicate object.           | When you want to avoid copying temporary objects.|
| **Parameter Type**              | `const MyClass&`                            | `MyClass&&` (rvalue reference)                 |
| **Action**                      | Copies resources.                          | Moves (transfers) resources, leaving the source object in a valid but empty state. |

#### Example Scenario: Copy vs Move

Consider a class `Buffer` that manages a large array of data.

```cpp
class Buffer {
    int* data;
    size_t size;
public:
    // Constructor
    Buffer(size_t size) : size(size) {
        data = new int[size];
        std::cout << "Constructor: Allocating " << size << " integers\n";
    }

    // Copy constructor
    Buffer(const Buffer& other) : size(other.size) {
        data = new int[other.size];  // Allocating new memory
        std::copy(other.data, other.data + size, data);
        std::cout << "Copy constructor: Copying data\n";
    }

    // Move constructor
    Buffer(Buffer&& other) noexcept : data(other.data), size(other.size) {
        other.data = nullptr;  // Transfer ownership of data
        other.size = 0;
        std::cout << "Move constructor: Moving data\n";
    }

    ~Buffer() {
        delete[] data;
    }
};

int main() {
    Buffer buf1(1000);

    // Copying buf1
    Buffer buf2 = buf1;  // Invokes copy constructor

    // Moving buf1
    Buffer buf3 = std::move(buf1);  // Invokes move constructor
}
```

**Output**:
```
Constructor: Allocating 1000 integers
Copy constructor: Copying data
Move constructor: Moving data
```

**Explanation**:
- The copy constructor duplicates the data, which can be costly for large arrays.
- The move constructor transfers ownership of the data, making it much more efficient since no new memory is allocated or copied.

#### When to Define Copy and Move Constructors

- **Copy Constructor**: Define when your class manages resources (like dynamic memory or handles) and needs a custom strategy for copying them. For example, if the default shallow copy would result in **double deletion** of pointers.
- **Move Constructor**: Define when your class holds **expensive resources** and you want to allow the transfer of ownership to avoid deep copies. It is especially important for **RAII** types or when dealing with large containers, strings, or arrays.

#### Best Practices

1. **Rule of Three/Five/Zero**:
   - **Rule of Three**: If you define a custom **destructor**, **copy constructor**, or **copy assignment operator**, you likely need all three.
   - **Rule of Five**: If your class uses **move semantics**, you may need to define a custom **move constructor** and **move assignment operator** as well.
   - **Rule of Zero**: If possible, avoid manual resource management by relying on **smart pointers** or other RAII wrappers, eliminating the need to define any of these constructors manually.

2. **Use `std::move`**: To explicitly move objects and invoke the move constructor, use `std::move`. This casts an object to an rvalue reference, allowing it to be "moved."

3. **Prefer move over copy**: Use move semantics when dealing with **temporary objects** or **large resources** that should not be duplicated unnecessarily.

### **Summary**

- **Copy constructor**: Creates a copy of an object, usually used when passing objects by value or returning them.
- **Move constructor**: Transfers resources from one object to another, eliminating unnecessary copies, especially for temporary objects.
- Move semantics provide a significant performance boost by allowing **ownership of resources** to be transferred rather than copied.

Understanding these constructors allows you to manage resources efficiently and write more performant C++ code, especially when dealing with large or complex objects.

## Passing by Value for Copyable Parameters That Are Cheap to Move and Always Copied

In C++, when deciding how to pass parameters into functions, there are several options: **passing by value**, **passing by reference**, or **passing by rvalue reference**. Choosing the most efficient option depends on the type of parameter, how it's used in the function, and whether the parameter is copied, moved, or modified.

**Item 41** of *Effective Modern C++* discusses the guideline of **passing by value** for **copyable parameters** that are **cheap to move and always copied**. This can lead to simpler code, and sometimes, even better performance.

### Key Points

1. **When to Pass by Value**:
   - You should pass by value when:
     - The object is **always copied** within the function.
     - The type of the object is **cheap to move** (i.e., the move constructor is inexpensive).
     - The type has an **efficient move constructor** (e.g., `std::string`, `std::vector`).

   **Example**:
   ```cpp
   void process(std::string str) {
       // str is passed by value, no extra copy is needed
       // if passed an rvalue, it will be moved
       str += " processed";
   }
   ```

   Here, if `process` is called with an rvalue, the string will be **moved** rather than copied. If it's called with an lvalue, the string will still only be **copied once**, since it's passed by value.

2. **Advantages of Passing by Value**:
   - **Single Copy/Move**: Passing by value enables the compiler to either move or copy the parameter based on whether it's an lvalue or rvalue. You avoid unnecessary copies by directly moving rvalues.
   - **Simpler Code**: No need to worry about distinguishing between lvalue and rvalue references. The function takes ownership of the argument.
   - **Compiler Optimizations**: Modern compilers often optimize the code by eliminating unnecessary copies (e.g., **Return Value Optimization (RVO)** and **Copy Elision**).

3. **Considerations for Cheap-to-Move Types**:
   - **Cheap-to-move types** (like `std::string`, `std::vector`, and other types with efficient move constructors) are ideal candidates for passing by value. For these types, a move operation is usually faster than copying, and passing by value simplifies function signatures.

   **Example**:
   ```cpp
   void process(std::vector<int> vec) {
       vec.push_back(42);  // vec can be moved into the function if passed an rvalue
   }
   ```

   In this example, if `process` is called with an rvalue (e.g., `process(std::vector<int>{1, 2, 3})`), the vector is moved into the function, and no copies are made.

4. **Comparison with Passing by Reference**:
   - **Pass by const reference** (`const T&`): This avoids copying or moving the object but is less flexible when the function always needs to copy or modify the object.
     - If a copy is needed, you’ll need to explicitly copy the reference.
   - **Pass by value** (`T`): The parameter is copied or moved directly into the function, which can be more efficient if you always copy or modify the object anyway.

   **Example**:
   ```cpp
   void process(const std::string& str) {
       std::string copy = str;  // Explicit copy is required
   }
   ```

   In this case, if the function always makes a copy, it's more efficient to pass the argument by value directly.

5. **Avoiding Redundant Copies**:
   - When you pass by value, the function takes ownership of the parameter. If the caller passes an rvalue, it is moved, not copied, preventing redundant copies.
   - If you pass by const reference and then copy inside the function, the object will be copied every time, even if the caller passed an rvalue that could have been moved.

6. **Passing by Value Is Ideal for Small and Move-Efficient Types**:
   - **Small types** (like `int`, `char`, etc.) are naturally cheap to copy, so passing them by value is almost always preferred.
   - **Move-efficient types** (like `std::unique_ptr`, `std::shared_ptr`, `std::string`, etc.) benefit from passing by value when they’re always copied in the function. This allows the function to move the parameter when possible.

### Guidelines for Using Pass by Value

1. **Use pass by value when:**
   - The type is **cheap to move** (e.g., types with efficient move constructors like `std::string` or `std::vector`).
   - The function **always makes a copy** of the parameter, whether to modify it or pass it to another function.
   - You want the function to take ownership of the argument and potentially move it into local variables.

2. **Use pass by const reference when:**
   - You want to **avoid copying** and the function doesn’t modify the parameter.
   - The parameter is **expensive to copy** (e.g., large containers or custom types without move semantics).
   - You don’t need to take ownership of the argument or create a local copy.

3. **Use pass by rvalue reference (`T&&`) when:**
   - You specifically want to handle **rvalues** differently from lvalues (e.g., by moving them but copying lvalues).
   - You are implementing **move semantics** in your function.

### Examples for Comparison

**Pass by Value (Efficient for Cheap-to-Move Types)**:
```cpp
void process(std::string str) {
    // str is passed by value, no need for explicit copying
    str += " processed";
}
```

**Pass by Const Reference (Requires Manual Copy)**:
```cpp
void process(const std::string& str) {
    std::string copy = str;  // Copy needed, even if str is an rvalue
    copy += " processed";
}
```

**Pass by Rvalue Reference (Explicit Move for Rvalues)**:
```cpp
void process(std::string&& str) {
    str += " processed";  // Only accepts rvalues
}
```

### Summary

- **Pass by value** is ideal for parameters that are always copied and are **cheap to move**. This approach ensures efficiency, especially when rvalues are passed.
- It avoids redundant copying, takes advantage of move semantics, and leads to cleaner code by not requiring explicit copies for const references.
- Always consider the **cost of moving** vs. **copying**, and choose pass by value when the object is cheap to move and will always be copied within the function.

By using pass by value for **copyable** and **cheap-to-move** parameters, you can write functions that are both efficient and easy to understand.

## Use `std::unique_ptr` for exclusive-ownership resource management

**`std::unique_ptr`** is designed for **exclusive ownership** of a resource.

### **What is `std::unique_ptr`?**

- **Exclusive Ownership**: A `std::unique_ptr` holds a pointer to a dynamically allocated object and ensures that only one `std::unique_ptr` instance can own the resource at any time. Once a `std::unique_ptr` is destroyed or reset, the resource is automatically released.
  
- **Automatic Resource Management**: When a `std::unique_ptr` goes out of scope or is explicitly deleted, it automatically releases (deletes) the owned resource. This helps prevent memory leaks and simplifies resource management using RAII (Resource Acquisition Is Initialization).

---

### **Why Use `std::unique_ptr`?**

1. **Automatic Resource Deallocation**:
   - A `std::unique_ptr` takes care of deleting the resource when it goes out of scope, preventing memory leaks.
   - **No need for manual `delete`**: You don’t need to call `delete` explicitly, as the `std::unique_ptr` automatically handles it.

2. **Exclusive Ownership**:
   - Only one `std::unique_ptr` can own a particular resource at any time, which helps avoid complex ownership issues and simplifies reasoning about ownership.
   - If you want to transfer ownership, it must be done explicitly (using `std::move`), making ownership transfer clear and intentional.

3. **Non-Copyable**:
   - `std::unique_ptr` is **non-copyable** (by design) to ensure exclusive ownership. You can’t accidentally share ownership or copy the pointer. This prevents double-deletion and dangling pointer issues.

4. **Safe for Exception Handling**:
   - When an exception occurs, `std::unique_ptr` ensures that the resource it manages will still be properly cleaned up, thanks to RAII.

---

### **How to Use `std::unique_ptr`**

#### **1. Basic Creation**
You create a `std::unique_ptr` using `std::make_unique`, which is the recommended and safe way to construct it. 

- **Example**:
  ```cpp
  #include <memory>  // for std::unique_ptr and std::make_unique
  #include <iostream>

  class Widget {
  public:
      Widget() { std::cout << "Widget Created\n"; }
      ~Widget() { std::cout << "Widget Destroyed\n"; }
  };

  int main() {
      // Creating a unique_ptr using std::make_unique
      std::unique_ptr<Widget> ptr = std::make_unique<Widget>();

      // Widget gets automatically destroyed when 'ptr' goes out of scope
      return 0;
  }
  ```

  In this example, `ptr` owns the dynamically allocated `Widget`. When `ptr` goes out of scope at the end of the `main` function, the `Widget` is automatically destroyed.

#### **2. Transferring Ownership (Using `std::move`)**

Since `std::unique_ptr` cannot be copied, transferring ownership from one `std::unique_ptr` to another must be done using `std::move`.

- **Example**:
  ```cpp
  std::unique_ptr<Widget> ptr1 = std::make_unique<Widget>();

  // Transferring ownership from ptr1 to ptr2 using std::move
  std::unique_ptr<Widget> ptr2 = std::move(ptr1);

  if (!ptr1) {
      std::cout << "ptr1 is empty after move\n";
  }
  ```

  After `std::move(ptr1)`, ownership of the resource is transferred to `ptr2`, and `ptr1` becomes null. This prevents double ownership of the same resource.

#### **3. Custom Deleters**

You can provide a custom deleter for `std::unique_ptr`, which can be useful when you need to clean up resources other than memory (e.g., closing a file or releasing network resources).

- **Example**:
  ```cpp
  auto fileDeleter = [](FILE* file) {
      if (file) {
          std::cout << "Closing file\n";
          fclose(file);
      }
  };

  std::unique_ptr<FILE, decltype(fileDeleter)> filePtr(fopen("example.txt", "r"), fileDeleter);
  ```

  In this case, when `filePtr` goes out of scope, the custom deleter will ensure that the file is closed properly.

#### **4. Arrays with `std::unique_ptr`**

`std::unique_ptr` can also be used to manage arrays. However, you should specify that it is managing an array explicitly, and use `std::make_unique<T[]>` to allocate memory.

- **Example**:
  ```cpp
  std::unique_ptr<int[]> arr = std::make_unique<int[]>(10);
  arr[0] = 42;  // Access array elements
  ```

  Here, `std::unique_ptr` automatically calls `delete[]` to release the array when `arr` goes out of scope.

---

### **When to Use `std::unique_ptr`**

1. **Exclusive Ownership**: Use `std::unique_ptr` when you want to enforce single ownership of a resource. This guarantees that only one pointer is responsible for the resource, simplifying ownership and memory management.

2. **Automatic Cleanup**: Use `std::unique_ptr` when you need automatic cleanup of a resource, especially in complex code with multiple exit paths or exceptions.

3. **Non-Copyable Resources**: For resources that cannot or should not be shared (e.g., file handles, network connections, hardware resources), `std::unique_ptr` is ideal because it prevents accidental sharing or copying.

4. **When You Don’t Need Shared Ownership**: If you don’t need multiple pointers to access the same resource (i.e., no shared ownership is required), `std::unique_ptr` is the best choice. If shared ownership is required, consider `std::shared_ptr`.

---

### **Avoid Using `std::unique_ptr` When...**

1. **You Need Shared Ownership**: If multiple parts of your program need to share ownership of a resource, use `std::shared_ptr` instead.
  
2. **Manual Memory Management is Unnecessary**: If you don’t need to manage a dynamic resource (like a stack-allocated object or a static resource), using a `std::unique_ptr` is overkill and unnecessary.

---

> When C++11 introduced `std::unique_ptr`, a smart pointer type that expresses exclusive ownership of a dynamically allocated object and deletes the object when the `unique_ptr` goes out of scope, our style guide initially disallowed usage. The behavior of the `unique_ptr` was unfamiliar to most engineers, and the related move semantics that the language introduced were very new and, to most engineers, very confusing. Preventing the introduction of std::unique_ptr in the codebase seemed the safer choice. We updated our tooling to catch references to the disallowed type and kept our existing guidance recommending other types of existing smart pointers.
> 
> Time passed. Engineers had a chance to adjust to the implications of move semantics and we became increasingly convinced that using std::unique_ptr was directly in line with the goals of our style guidance. The information regarding object ownership that a `std::unique_ptr` facilitates at a function call site makes it far easier for a reader to understand that code. The added complexity of introducing this new type, and the novel move semantics that come with it, was still a strong concern, but the significant improvement in the long-term overall state of the codebase made the adoption of `std::unique_ptr` a worthwhile trade-off.[^1]

### **Summary**

- **`std::unique_ptr`** is a smart pointer for managing **exclusive ownership** of a dynamically allocated resource.
- It **automatically cleans up** the resource when the pointer goes out of scope, making it ideal for preventing memory leaks.
- **Ownership transfer** must be done explicitly using `std::move`, and `std::unique_ptr` cannot be copied to ensure single ownership.
- It is perfect for **non-shared, dynamically allocated resources**, ensuring safe and efficient memory management in modern C++.

Using `std::unique_ptr` not only simplifies resource management but also leads to safer, more maintainable, and exception-safe code.

## Use `std::shared_ptr` for shared-ownership resource management

**`std::shared_ptr`** is designed for **shared ownership**, allowing multiple pointers to manage the same resource.

---

### **What is `std::shared_ptr`?**

- **Shared Ownership**: A `std::shared_ptr` allows multiple pointers to share ownership of the same dynamically allocated resource. The resource is only freed when the last `std::shared_ptr` that owns the resource is destroyed.
  
- **Reference Counting**: `std::shared_ptr` internally uses a **reference count** to track how many `std::shared_ptr` instances are pointing to the same resource. When the reference count reaches zero (i.e., no more pointers refer to the resource), the resource is automatically deallocated.

- **Automatic Resource Management**: Like `std::unique_ptr`, `std::shared_ptr` automatically manages the resource’s lifetime, ensuring that it is deallocated when no longer needed. However, it is specifically designed for scenarios where **multiple owners** need access to the same resource.

---

### **Why Use `std::shared_ptr`?**

1. **Shared Ownership**:
   - `std::shared_ptr` enables multiple parts of your program to share ownership of a resource, making it ideal for scenarios where the resource needs to be accessed by different objects or threads.
   
2. **Automatic Resource Deallocation**:
   - The resource is automatically deleted when the last `std::shared_ptr` goes out of scope. This ensures proper cleanup and prevents memory leaks.

3. **Thread Safety**:
   - **Reference counting** in `std::shared_ptr` is thread-safe. Multiple threads can safely share and manage the same resource without worrying about race conditions in reference counting (though the resource itself may need additional synchronization if it's mutable).

4. **Flexibility**:
   - You can easily pass `std::shared_ptr` objects around, assign them to new variables, or return them from functions without worrying about manual memory management.

---

### **How to Use `std::shared_ptr`**

#### **1. Basic Creation**

You create a `std::shared_ptr` using `std::make_shared`, which is the recommended way to allocate a resource for a shared pointer. This is efficient and reduces the chances of memory leaks.

- **Example**:
  ```cpp
  #include <memory>  // for std::shared_ptr and std::make_shared
  #include <iostream>

  class Widget {
  public:
      Widget() { std::cout << "Widget Created\n"; }
      ~Widget() { std::cout << "Widget Destroyed\n"; }
  };

  int main() {
      // Create a shared_ptr using std::make_shared
      std::shared_ptr<Widget> ptr1 = std::make_shared<Widget>();

      {
          // Create another shared_ptr that shares ownership
          std::shared_ptr<Widget> ptr2 = ptr1;
          std::cout << "Shared ownership count: " << ptr1.use_count() << "\n";
      }

      // ptr2 goes out of scope, but ptr1 still owns the resource
      std::cout << "Shared ownership count: " << ptr1.use_count() << "\n";

      // Resource is destroyed when the last shared_ptr (ptr1) is destroyed
      return 0;
  }
  ```

- **Output**:
  ```
  Widget Created
  Shared ownership count: 2
  Shared ownership count: 1
  Widget Destroyed
  ```

In this example, both `ptr1` and `ptr2` share ownership of the `Widget`. When `ptr2` goes out of scope, `ptr1` still owns the resource. The `Widget` is only destroyed when `ptr1` also goes out of scope and the reference count drops to zero.

#### **2. Sharing Ownership**

Multiple `std::shared_ptr` instances can point to the same resource. This is done by copying or assigning the `std::shared_ptr`.

- **Example**:
  ```cpp
  std::shared_ptr<Widget> ptr1 = std::make_shared<Widget>();

  // Share ownership
  std::shared_ptr<Widget> ptr2 = ptr1;
  ```

Both `ptr1` and `ptr2` share ownership of the `Widget`, and the reference count will be 2.

#### **3. Resetting and Releasing Resources**

You can reset a `std::shared_ptr`, which decreases its reference count. When the reference count reaches zero, the resource is released.

- **Example**:
  ```cpp
  std::shared_ptr<Widget> ptr1 = std::make_shared<Widget>();

  // Reset ptr1, decreasing the reference count
  ptr1.reset();  // If ptr1 was the last owner, Widget is destroyed
  ```

If `ptr1` was the only owner of the resource, calling `reset` will destroy the `Widget`. If other `std::shared_ptr` instances share ownership, the resource will not be released until all owners are reset or go out of scope.

#### **4. Weak Pointers and Avoiding Cycles**

One problem with `std::shared_ptr` is **cyclic references**, where two or more `std::shared_ptr` instances refer to each other, causing a memory leak (since their reference counts never drop to zero). To avoid this, **`std::weak_ptr`** is used as a non-owning reference to a `std::shared_ptr`-managed resource.

- **Example**:
  ```cpp
  std::shared_ptr<Widget> ptr1 = std::make_shared<Widget>();
  std::weak_ptr<Widget> weakPtr = ptr1;  // weakPtr does not affect the reference count

  if (std::shared_ptr<Widget> tempPtr = weakPtr.lock()) {
      // Resource is still valid
  } else {
      // Resource has been deleted
  }
  ```

`std::weak_ptr` is useful in breaking **cyclic dependencies** (e.g., in graphs or bidirectional relationships between objects).

---

### **When to Use `std::shared_ptr`**

1. **Shared Ownership**: Use `std::shared_ptr` when multiple parts of your program need to **share ownership** of a resource. Each `std::shared_ptr` increases the reference count, and the resource will be deallocated when the last `std::shared_ptr` is destroyed.

2. **Thread-Safe Resource Sharing**: Use `std::shared_ptr` in multi-threaded environments where multiple threads need to share and manage the same resource. The reference counting mechanism is thread-safe, making it ideal for concurrent programs.

3. **When Copying is Required**: If you need to copy a pointer to a resource and maintain shared ownership between different parts of the program, `std::shared_ptr` is the best solution.

4. **Avoiding Manual Memory Management**: Like `std::unique_ptr`, `std::shared_ptr` helps avoid manual `new`/`delete` calls and simplifies resource management.

---

### **When Not to Use `std::shared_ptr`**

1. **Single Ownership**: If only one part of the program should own the resource, use `std::unique_ptr` instead. `std::shared_ptr` has more overhead due to its reference counting, so it's unnecessary when shared ownership is not needed.

2. **Performance-Sensitive Code**: `std::shared_ptr` introduces overhead due to its reference counting mechanism. In performance-critical applications, this overhead may be undesirable.

3. **Cyclic Dependencies**: Be cautious of **cyclic references**. If two `std::shared_ptr` objects reference each other, they can form a cycle, preventing proper resource deallocation. To solve this, use `std::weak_ptr` to break the cycle.

---

### **Custom Deleters in `std::shared_ptr`**

Like `std::unique_ptr`, you can specify custom deleters in `std::shared_ptr`. This is useful for managing resources other than memory (e.g., closing file handles or releasing network connections).

- **Example**:
  ```cpp
  auto fileDeleter = [](FILE* file) {
      if (file) {
          std::cout << "Closing file\n";
          fclose(file);
      }
  };

  std::shared_ptr<FILE> filePtr(fopen("example.txt", "r"), fileDeleter);
  ```

---

### **Summary**

- **`std::shared_ptr`** provides **shared ownership** of a resource, allowing multiple pointers to manage the same resource through **reference counting**.
- The resource is automatically cleaned up when the last `std::shared_ptr` that owns it goes out of scope or is reset.
- It is ideal when multiple parts of a program or multiple threads need to share access to the same resource.
- **Watch out for cyclic dependencies**; use `std::weak_ptr` to break cycles and avoid memory leaks.
  
In conclusion, `std::shared_ptr` is a powerful tool for managing shared ownership of resources in C++ while simplifying memory management and ensuring safe, automatic cleanup. However, it should be used judiciously to avoid unnecessary overhead and potential issues with cyclic references.

## Use `std::weak_ptr` for `std::shared_ptr`-like pointers that can dangle

`std::weak_ptr` works as a non-owning smart pointer that **points to a resource managed by a `std::shared_ptr`**, but it doesn't affect the **reference count** of the shared resource. This makes `std::weak_ptr` useful in situations where you want to avoid **cyclic dependencies** and safely manage resources that may no longer exist.

---

### **What is `std::weak_ptr`?**

- **Non-Owning Smart Pointer**: Unlike `std::shared_ptr`, a `std::weak_ptr` does not **own** the resource it points to. Instead, it simply observes a resource managed by one or more `std::shared_ptr`s.

- **No Reference Counting**: `std::weak_ptr` does not participate in the reference counting mechanism of `std::shared_ptr`. It doesn't increase or decrease the reference count of the shared object.

- **Dangling Pointers**: When the last `std::shared_ptr` managing a resource is destroyed, the resource is freed, but any `std::weak_ptr` pointing to that resource becomes **dangling**. `std::weak_ptr` handles this safely by allowing you to check if the resource still exists before accessing it.

---

### **Why Use `std::weak_ptr`?**

1. **Avoid Cyclic References**:
   - If two `std::shared_ptr`s reference each other, they form a **cycle** where neither can be destroyed, even if they go out of scope. This causes a **memory leak**. `std::weak_ptr` breaks such cycles by pointing to a `std::shared_ptr`-managed object **without participating in reference counting**.

2. **Safe Access to Potentially Expired Objects**:
   - With `std::weak_ptr`, you can **safely check** if the resource still exists before accessing it. If the resource has been freed, the `std::weak_ptr` can’t access it, preventing dangling pointer errors.

3. **Efficient Observer Pattern**:
   - In the **observer pattern**, observers may want to reference a shared object without keeping it alive. Using `std::weak_ptr` allows them to reference the object without extending its lifetime.

---

### **How to Use `std::weak_ptr`**

#### **1. Creating a `std::weak_ptr`**

A `std::weak_ptr` is typically created from an existing `std::shared_ptr`. It does not change the reference count of the object.

- **Example**:
  ```cpp
  #include <iostream>
  #include <memory>

  class Widget {
  public:
      Widget() { std::cout << "Widget Created\n"; }
      ~Widget() { std::cout << "Widget Destroyed\n"; }
  };

  int main() {
      std::shared_ptr<Widget> sharedPtr = std::make_shared<Widget>();

      // Create a weak_ptr from sharedPtr
      std::weak_ptr<Widget> weakPtr = sharedPtr;

      // weakPtr does not affect the reference count
      std::cout << "Shared ownership count: " << sharedPtr.use_count() << "\n"; // 1

      return 0;  // Resource will be destroyed when sharedPtr goes out of scope
  }
  ```

- **Output**:
  ```
  Widget Created
  Shared ownership count: 1
  Widget Destroyed
  ```

In this example, `weakPtr` observes the resource without incrementing the reference count. The `Widget` is destroyed when `sharedPtr` goes out of scope, even though `weakPtr` still exists.

#### **2. Accessing the Resource Safely with `lock()`**

To access the resource managed by a `std::weak_ptr`, you need to **lock** it. Locking a `std::weak_ptr` returns a `std::shared_ptr` to the resource, allowing you to use it safely if the resource still exists.

- **Example**:
  ```cpp
  std::shared_ptr<Widget> sharedPtr = std::make_shared<Widget>();
  std::weak_ptr<Widget> weakPtr = sharedPtr;

  // Check if the resource still exists before using it
  if (std::shared_ptr<Widget> tempPtr = weakPtr.lock()) {
      // Safe to use tempPtr
      std::cout << "Resource is still valid\n";
  } else {
      // Resource has been deleted
      std::cout << "Resource is no longer available\n";
  }
  ```

The `lock()` function attempts to create a `std::shared_ptr` from the `std::weak_ptr`. If the original resource has already been freed, `lock()` returns `nullptr`, ensuring you don’t access a dangling pointer.

#### **3. Handling Cyclic Dependencies**

One of the most common uses of `std::weak_ptr` is to break **cyclic dependencies**. In scenarios where two or more objects hold `std::shared_ptr`s to each other, the reference count never reaches zero, leading to a memory leak.

- **Example of Cyclic Dependency**:
  ```cpp
  class B;

  class A {
  public:
      std::shared_ptr<B> ptrB;
      ~A() { std::cout << "A Destroyed\n"; }
  };

  class B {
  public:
      std::shared_ptr<A> ptrA;  // This causes a cyclic dependency
      ~B() { std::cout << "B Destroyed\n"; }
  };

  int main() {
      std::shared_ptr<A> a = std::make_shared<A>();
      std::shared_ptr<B> b = std::make_shared<B>();

      a->ptrB = b;
      b->ptrA = a;  // Now we have a cycle

      // Neither A nor B will be destroyed, resulting in a memory leak
      return 0;
  }
  ```

- **Fixing Cyclic Dependency with `std::weak_ptr`**:
  ```cpp
  class B;

  class A {
  public:
      std::shared_ptr<B> ptrB;
      ~A() { std::cout << "A Destroyed\n"; }
  };

  class B {
  public:
      std::weak_ptr<A> ptrA;  // Break the cycle with std::weak_ptr
      ~B() { std::cout << "B Destroyed\n"; }
  };

  int main() {
      std::shared_ptr<A> a = std::make_shared<A>();
      std::shared_ptr<B> b = std::make_shared<B>();

      a->ptrB = b;
      b->ptrA = a;  // Now the cycle is broken

      return 0;
  }
  ```

By changing `B::ptrA` to a `std::weak_ptr`, the cyclic reference is broken, and both `A` and `B` will be correctly destroyed when their `std::shared_ptr`s go out of scope.

#### **4. Checking Validity with `expired()`**

`std::weak_ptr` provides a member function `expired()` to check if the object it points to has been destroyed.

- **Example**:
  ```cpp
  std::shared_ptr<Widget> sharedPtr = std::make_shared<Widget>();
  std::weak_ptr<Widget> weakPtr = sharedPtr;

  // Destroy sharedPtr
  sharedPtr.reset();

  if (weakPtr.expired()) {
      std::cout << "Resource is no longer valid\n";
  }
  ```

Here, `expired()` returns `true` because `sharedPtr` has been reset, and the resource has been destroyed.

---

### **When to Use `std::weak_ptr`**

1. **Breaking Cyclic References**:
   - In cases where two or more objects may reference each other, use `std::weak_ptr` for one of the references to avoid cyclic dependencies and memory leaks.

2. **Non-Owning Observers**:
   - If a class or function needs to observe a shared resource without affecting its lifetime, use `std::weak_ptr`. This is common in observer patterns or callbacks.

3. **Temporary or Conditional Access**:
   - When you only need to temporarily access a resource managed by a `std::shared_ptr`, and you’re not concerned if the resource has been deallocated, `std::weak_ptr` is a good choice.

---

### **When Not to Use `std::weak_ptr`**

1. **When Ownership is Required**:
   - If a part of your code needs to extend the lifetime of a resource, use `std::shared_ptr` instead. `std::weak_ptr` does not keep the resource alive.

2. **Overhead for Simple Cases**:
   - `std::weak_ptr` incurs a small additional overhead due to the need for managing the weak reference. If the resource does not need shared access or is easily managed manually, `std::weak_ptr` may be unnecessary.

---

### **Summary**

- **`std::weak_ptr`** is a **non-owning smart pointer** that allows you to safely reference resources managed by `std::shared_ptr`.
- It avoids **cyclic dependencies** and can be used to safely access potentially expired resources.
- It is commonly used in observer patterns and in scenarios where shared resources need to be weakly referenced.

`std::weak_ptr` helps solve critical problems in C++ resource management, providing a safe mechanism to observe shared resources without extending their lifetimes or creating memory leaks through cyclic dependencies.

## Prefer `std::make_unique` and `std::make_shared` to direct use of `new`

It is considered best practice to use factory functions like `std::make_unique` and `std::make_shared` rather than directly allocating objects with `new`. These functions provide several benefits, including better memory safety, efficiency, and cleaner code.

---

### **Why Avoid Direct Use of `new`?**

When you use `new`, you must manually manage memory allocation and deallocation. This makes your code more error-prone and can lead to issues such as **memory leaks** or **double deletions** if not properly handled with `delete`. 

For example:
```cpp
MyClass* obj = new MyClass();  // Allocates memory
delete obj;                    // Deallocates memory
```

This pattern is outdated in modern C++ because it doesn't take full advantage of **smart pointers** that automatically manage the lifecycle of the object and ensure memory is cleaned up correctly.

---

### **Using `std::make_unique` and `std::make_shared`**

#### **1. `std::make_unique` for Exclusive Ownership**

`std::make_unique` is used to create a **unique pointer** (`std::unique_ptr`), which enforces **exclusive ownership** of a resource. The resource will be automatically destroyed when the `std::unique_ptr` goes out of scope.

- **Example**:
  ```cpp
  #include <memory>
  
  std::unique_ptr<MyClass> obj = std::make_unique<MyClass>();
  ```

  This is preferable over:
  ```cpp
  std::unique_ptr<MyClass> obj(new MyClass());
  ```

- **Benefits**:
  - **Exception safety**: `std::make_unique` guarantees exception safety. If an exception is thrown during object creation, no memory leak occurs because the object will not be created unless the entire construction process completes successfully.
  - **Code simplicity**: Cleaner, more concise code that eliminates the need for `new`.

#### **2. `std::make_shared` for Shared Ownership**

`std::make_shared` creates a **shared pointer** (`std::shared_ptr`) that allows multiple entities to share ownership of the same resource. The resource is deallocated only when the last `std::shared_ptr` referencing it is destroyed.

- **Example**:
  ```cpp
  #include <memory>

  std::shared_ptr<MyClass> obj = std::make_shared<MyClass>();
  ```

  This is preferable over:
  ```cpp
  std::shared_ptr<MyClass> obj(new MyClass());
  ```

- **Benefits**:
  - **Efficiency**: `std::make_shared` performs a **single memory allocation** for both the object and the reference counting structure, which is more efficient than creating them separately with `new` and `std::shared_ptr`.
  - **Exception safety**: Just like `std::make_unique`, it ensures no memory is leaked if an exception is thrown during object creation.
  
---

### **Detailed Benefits of `std::make_unique` and `std::make_shared`**

#### **1. Exception Safety**
When using `new`, if an exception occurs after the allocation but before the object is fully initialized, you may end up with a **memory leak**. With `std::make_unique` and `std::make_shared`, the responsibility for memory management is handled automatically, and no memory leak occurs even in case of an exception.

#### **2. Memory Efficiency**
`std::make_shared` is more memory-efficient than using `new` with `std::shared_ptr` because it performs only one memory allocation. With `new`, two separate allocations are needed: one for the object and one for the control block that manages reference counting. `std::make_shared` combines these into a single allocation, reducing overhead.

#### **3. Cleaner and More Concise Code**
Using `std::make_unique` and `std::make_shared` makes code shorter and easier to read:
- You don’t need to worry about manually calling `new` or `delete`.
- The factory functions clearly indicate the type of smart pointer being created, improving code clarity.

#### **4. Best Practice for Modern C++**
Since C++11, the use of `std::make_unique` and `std::make_shared` has been strongly recommended by the C++ community, and many modern codebases avoid raw use of `new` altogether, favoring these functions for memory management.

---

### **When Not to Use `std::make_shared`**

There are a few rare cases where `std::make_shared` may not be appropriate:
1. **Custom Deleters**: If you need a custom deleter, you can’t use `std::make_shared`. In such cases, you should still manually allocate the object using `new` or provide a custom deleter for the `std::shared_ptr`.
   - Example:
     ```cpp
     std::shared_ptr<MyClass> obj(new MyClass(), CustomDeleter());
     ```

2. **Weak Ownership**: If you're working in scenarios where you're concerned about **weak pointers** (`std::weak_ptr`), you'll want to manage the lifetime of the object separately from the reference counting structure.

---

### **Summary**

- **Use `std::make_unique`** when you need **exclusive ownership** of an object, as it’s safer and cleaner than using `new`.
- **Use `std::make_shared`** when you need **shared ownership** of an object, as it improves memory efficiency and ensures exception safety.
- Both of these factory functions are **modern C++ best practices** that reduce the risk of memory leaks and simplify resource management, making code more robust and easier to maintain.

### Pimpl Idiom and Special Member Functions

The **Pimpl Idiom** (Pointer to Implementation) is a common C++ design pattern that helps to **hide implementation details** from a class’s public interface. This can improve **encapsulation, compilation time, and ABI stability**. It works by moving the implementation of a class into a separate class (the "impl" or implementation class) and only exposing a pointer to that class in the header.

---

### **Why Define Special Member Functions in the Implementation File?**

When using the Pimpl idiom, the special member functions (e.g., constructor, destructor, copy constructor, move constructor, assignment operators) should be **defined** in the **implementation file (.cpp)**. This is important for several reasons:

1. **Avoiding Inline Compilation Dependencies**:
   - By moving the definitions of special member functions into the `.cpp` file, you reduce the need to include the full definition of the implementation class in the header file. This helps avoid unnecessary **re-compilation** of translation units that depend on the header, speeding up build times.

2. **Preserving Binary Compatibility (ABI Stability)**:
   - The Pimpl idiom helps ensure **ABI stability** by preventing changes in the class’s implementation details from affecting the class’s public interface. Defining special member functions in the implementation file further enforces this, as changes to the implementation won’t require recompilation of code that depends on the class.

3. **Simplifying Forward Declarations**:
   - By keeping the special member functions out of the header file, you can forward-declare the implementation class (`Impl`) without needing to include its full definition. This helps keep the header file **lightweight**.

4. **Cleaner Interface**:
   - The header file is cleaner and free from implementation details, making it easier to work with and maintain. Clients using the class only see the high-level interface without being exposed to private members or unnecessary includes.

---

### **Example: Pimpl Idiom with Special Member Functions**

#### **Header File (`MyClass.h`)**

```cpp
// MyClass.h
#ifndef MYCLASS_H
#define MYCLASS_H

#include <memory>  // for std::unique_ptr

class MyClass {
public:
    MyClass();                            // Constructor
    ~MyClass();                           // Destructor
    MyClass(const MyClass& other);        // Copy Constructor
    MyClass& operator=(const MyClass& other);  // Copy Assignment
    MyClass(MyClass&& other) noexcept;    // Move Constructor
    MyClass& operator=(MyClass&& other) noexcept;  // Move Assignment

    void someMethod();

private:
    class Impl;                           // Forward declaration of implementation class
    std::unique_ptr<Impl> pImpl;          // Pointer to implementation
};

#endif // MYCLASS_H
```

#### **Implementation File (`MyClass.cpp`)**

```cpp
// MyClass.cpp
#include "MyClass.h"
#include <iostream>

class MyClass::Impl {
public:
    void someMethod() {
        std::cout << "Doing something in the Impl class." << std::endl;
    }
};

// Constructor
MyClass::MyClass() : pImpl(std::make_unique<Impl>()) {}

// Destructor
MyClass::~MyClass() = default;

// Copy Constructor
MyClass::MyClass(const MyClass& other) : pImpl(std::make_unique<Impl>(*other.pImpl)) {}

// Copy Assignment
MyClass& MyClass::operator=(const MyClass& other) {
    if (this == &other) return *this;
    *pImpl = *other.pImpl;
    return *this;
}

// Move Constructor
MyClass::MyClass(MyClass&& other) noexcept = default;

// Move Assignment
MyClass& MyClass::operator=(MyClass&& other) noexcept = default;

// Method definition
void MyClass::someMethod() {
    pImpl->someMethod();
}
```

---

### **Key Points to Note in the Example:**

1. **Forward Declaration of `Impl` in Header**:
   - In the header file (`MyClass.h`), the implementation class `Impl` is **forward-declared**. Only a pointer to `Impl` is stored as `std::unique_ptr<Impl>`, keeping the implementation hidden.

2. **Special Member Functions Defined in `.cpp`**:
   - All special member functions (constructor, destructor, copy/move constructors, and assignment operators) are defined in the `.cpp` file. This ensures that the full definition of `Impl` is not required in the header, reducing dependencies.

3. **Use of `std::unique_ptr` for Pimpl**:
   - `std::unique_ptr` is used to manage the lifetime of the implementation object (`Impl`). This automatically handles resource deallocation when the `MyClass` object is destroyed.

4. **Default Move Operations**:
   - The move constructor and move assignment operator are **defaulted** because `std::unique_ptr` naturally supports efficient move semantics. Defaulting these operations ensures correct ownership transfer without needing to manually define them.

---

### **Benefits of Defining Special Member Functions in the `.cpp` File**

- **Encapsulation**: The implementation is hidden from the header file, which means changes to `Impl` don’t affect clients using `MyClass`.
- **Reduced Coupling**: Since `Impl` is not included in the header file, other files that include `MyClass.h` don’t need to know about the details of `Impl`, reducing coupling and improving compilation times.
- **Cleaner and More Modular Code**: The public interface is separated from implementation details, leading to a cleaner and more maintainable codebase.

---

### **Summary**

- In the Pimpl idiom, the special member functions (constructors, destructors, copy/move constructors, and assignment operators) should be defined in the **implementation file** to promote **encapsulation**, **reduce dependencies**, and **improve compilation times**.
- By defining them in the `.cpp` file, you can avoid exposing implementation details and **maintain ABI stability**, especially in large codebases.

## Using `emplace` Instead of `insert` in C++

In C++, the STL containers provide two primary ways of adding elements: **insertion (`insert`)** and **emplacement (`emplace`)**. While they might seem similar at first glance, they have important differences in how they handle objects during insertion. In general, **`emplace`** is more efficient and flexible than `insert` in many scenarios, especially when dealing with complex types.

### **Key Differences Between `emplace` and `insert`**

1. **`insert`**:
   - **`insert` requires a fully constructed object** to be passed.
   - The object is **copied or moved** into the container. Even when moving, the move constructor is invoked.
   - Used when you already have an object ready for insertion.

   **Example**:
   ```cpp
   std::vector<std::string> vec;
   std::string s = "hello";
   vec.insert(vec.end(), s);   // Copy `s` into the vector
   vec.insert(vec.end(), std::move(s));  // Move `s` into the vector
   ```

   In this case, you are inserting an already constructed object into the container. If you use `move`, it avoids the copy, but the move constructor still needs to be called.

2. **`emplace`**:
   - **`emplace` constructs the object in-place**, directly within the container.
   - Instead of taking a pre-constructed object, `emplace` accepts the **constructor arguments** for the object and forwards them.
   - It avoids unnecessary moves or copies, making it more efficient for complex types.

   **Example**:
   ```cpp
   std::vector<std::string> vec;
   vec.emplace_back("hello");  // Constructs the string "hello" directly in the vector
   ```

   Here, `emplace` constructs the string in place using the constructor that takes a `const char*`. There's no extra copy or move, as would happen with `insert`.

### **Why Prefer `emplace` over `insert`?**

1. **Avoids Extra Copy/Move Operations**:
   - When you use `insert`, an object needs to be created before it is inserted into the container. Even if you move the object, it still needs to be constructed first, and then moved into the container.
   - With `emplace`, the object is constructed directly inside the container, skipping any copy or move operations.

   **Example of `insert` and `emplace`:**
   ```cpp
   std::vector<std::pair<int, std::string>> vec;

   // insert requires a pair to be constructed first
   vec.insert(vec.end(), std::pair<int, std::string>(1, "one"));

   // emplace constructs the pair directly in-place
   vec.emplace_back(1, "one");
   ```

   **Explanation**: `insert` creates the `std::pair` object outside the container and then copies or moves it into the vector. `emplace_back` constructs the `std::pair` directly in the vector.

2. **Efficiency with Complex Types**:
   - When dealing with user-defined types that have expensive copy or move constructors, `emplace` can significantly improve performance because it eliminates unnecessary temporary object creation.
   - This is especially important for containers like `std::map` or `std::set`, where the inserted objects are often complex key-value pairs.

   **Example**:
   ```cpp
   std::map<int, std::string> m;
   
   // insert requires a pair to be created first
   m.insert(std::make_pair(1, "one"));

   // emplace constructs the pair directly in the map
   m.emplace(1, "one");
   ```

3. **Perfect Forwarding**:
   - **`emplace` uses perfect forwarding**, which means it forwards the arguments to the constructor of the object being emplaced. This allows you to avoid temporary objects and make efficient use of the arguments.
   - In contrast, `insert` always operates on fully constructed objects, which might result in extra temporary objects if you are not careful.

   **Example**:
   ```cpp
   class Widget {
   public:
       Widget(int a, double b) { /*...*/ }
   };

   std::vector<Widget> vec;
   vec.emplace_back(42, 3.14);  // Forwards arguments to Widget's constructor
   ```

   Here, `emplace_back` directly constructs a `Widget` using the arguments `42` and `3.14`, which is more efficient than constructing the `Widget` outside the vector and inserting it.

### **When to Use `emplace`**

1. **Prefer `emplace` when:**
   - You are adding elements to a container, and **you don't already have a fully constructed object**.
   - The object being added is **expensive to move or copy**, such as complex classes or large objects.
   - You want to **construct the object in place** with minimal overhead.
   - You want to leverage **perfect forwarding** for constructor arguments.

2. **Use `insert` when:**
   - You already have a fully constructed object, and you simply want to move or copy it into the container.
   - The object type is **trivial** (e.g., primitive types like `int`, `double`), where the overhead of `emplace` and `insert` is negligible.

### Examples to Illustrate the Difference

#### Example 1: `insert` vs `emplace` in a Vector

```cpp
#include <vector>
#include <iostream>

class Widget {
public:
    Widget(int x, double y) {
        std::cout << "Widget constructed\n";
    }
};

int main() {
    std::vector<Widget> vec;

    // Insert: Requires the Widget to be fully constructed first
    Widget w(42, 3.14);
    vec.insert(vec.end(), w);  // Copying Widget

    // Emplace: Constructs the Widget in place
    vec.emplace_back(42, 3.14);  // No copy or move
}
```

**Output**:
```
Widget constructed
Widget constructed  // Insert causes two constructions
Widget constructed  // Emplace constructs only once
```

**Explanation**: When using `insert`, the `Widget` object is first created outside the vector and then copied into the vector, resulting in two constructions. With `emplace`, the object is constructed directly inside the vector, avoiding the extra copy.

#### Example 2: Using `emplace` in a `std::map`

```cpp
#include <map>
#include <iostream>

int main() {
    std::map<int, std::string> m;

    // Insert: Requires a std::pair to be constructed first
    m.insert(std::make_pair(1, "one"));

    // Emplace: Constructs the pair in place
    m.emplace(2, "two");

    for (const auto& p : m) {
        std::cout << p.first << ": " << p.second << '\n';
    }
}
```

**Output**:
```
1: one
2: two
```

**Explanation**: With `emplace`, the `std::pair` object is constructed directly in the map, avoiding the creation of a temporary object as done in the `insert` case.

### **Best Practices and Summary**

- **Use `emplace`** when you want to construct objects directly inside a container and avoid extra copies or moves.
- **Prefer `insert`** when you already have an object and you want to explicitly move or copy it into a container.
- **Complex types** or **types with expensive move/copy operations** benefit the most from `emplace`.
- **For trivial types** like `int` or `double`, the difference between `insert` and `emplace` is negligible, and either can be used.

By using `emplace`, you can reduce overhead, improve performance, and simplify code when adding objects to containers.

## Videos

1. [Smart Pointers in C++](https://www.youtube.com/watch?v=UOB7-B2MfwA&pp=ygUOc21hcnQgcG9pbnRlcnM%3D)

[^1]: Chapter 8, _Style Guides and Rules_. Software Engineering at Google. Curated by Titus Winters, Tom Manshreck & Hyrum Wright