# Modern C++

Modern C++ is defined by the introduction of significant features in **C++11** and **C++14**, which transformed the language into a more expressive, efficient, and safer tool for developers. C++11 introduced key innovations like **lambda expressions**, **move semantics**, and **smart pointers**, enabling developers to write cleaner, more concise code while enhancing performance and memory safety. It also brought **auto type deduction**, **range-based for loops**, and **constexpr**, pushing C++ closer to other high-level languages in terms of ease of use. C++14 followed up with refinements, such as generalized **lambda captures** and more flexible **constexpr** functions, further improving the developer experience. As Bjarne Stroustrup, the creator of C++, said: "C++11 feels like a new language," and with these enhancements, it’s more powerful and versatile than ever, suited for modern programming challenges.

## Modern Syntactical Features

### Traditional vs. Uniform Initialization

In C++, when creating objects, you can initialize them using either **parentheses (`()`)** or **braces (`{}`)**. The choice between these two initialization syntaxes has significant implications due to the introduction of **uniform initialization** (brace initialization) in C++11. Here's how they differ:

#### 1. **Parentheses Initialization (`()`)**:
   - **Direct Initialization** or **Value Initialization**.
   - It follows the **traditional C++ syntax** for initializing objects.
   - It can invoke constructors, especially when there is ambiguity (e.g., with a constructor that takes a single argument).

   **Example**:
   ```cpp
   class MyClass {
   public:
       MyClass(int x) { /*...*/ }
   };

   MyClass obj(10);  // Direct initialization: Calls constructor MyClass(int)
   ```

##### Key Points for Parentheses Initialization:
- Calls **constructors** directly, including explicit constructors.
- **Can't be used for `std::initializer_list` constructors**, which are called using braces (`{}`).
- Can cause **ambiguity** with single-argument constructors, especially for built-in types.

   **Example of Ambiguity**:
   ```cpp
   MyClass obj();  // This is NOT an object creation but a function declaration!
   ```

- If there is ambiguity (like in the example above), parentheses can be misinterpreted as a function declaration rather than an object declaration, known as **Most Vexing Parse**.

#### 2. **Braces Initialization (`{}`)**:
   - Introduced in **C++11** as **uniform initialization**.
   - It provides a consistent way to initialize objects using **brace initialization** or **list initialization**.
   - It can invoke constructors, including those that accept `std::initializer_list`.

   **Example**:
   ```cpp
   class MyClass {
   public:
       MyClass(int x) { /*...*/ }
       MyClass(std::initializer_list<int> list) { /*...*/ }
   };

   MyClass obj1{10};  // Brace initialization: Calls constructor MyClass(int)
   MyClass obj2{10, 20};  // Calls constructor MyClass(std::initializer_list<int>)
   ```

##### Key Points for Braces Initialization:
- Always **avoids narrowing conversions** (i.e., converting from a larger type to a smaller type or floating point to integer).
- Preferentially calls constructors that take `std::initializer_list` if available.
- Can initialize both aggregate types (like structs and arrays) and non-aggregate types (like user-defined classes).
- **Prevents the Most Vexing Parse problem** since it cannot be misinterpreted as a function declaration.

   **Example**:
   ```cpp
   MyClass obj{};  // Object creation with default constructor
   ```

#### Comparison of `()` vs `{}` Initialization

| **Aspect**                              | **Parentheses (`()`)**             | **Braces (`{}`)**                 |
|-----------------------------------------|------------------------------------|-----------------------------------|
| **Introduced In**                       | C++98 (Traditional Initialization) | C++11 (Uniform Initialization)    |
| **Preferred for**                       | Direct Initialization, Value Initialization | Uniform and List Initialization   |
| **`std::initializer_list` Support**     | No                                 | Yes                               |
| **Narrowing Conversions**               | Allowed                            | Disallowed                        |
| **Ambiguity (Most Vexing Parse)**       | Possible                           | Avoids this issue                 |
| **Aggregate Initialization**            | No                                 | Yes (for structs, arrays, etc.)   |
| **Constructor Call Priority**           | Calls constructors directly        | Prefers `std::initializer_list` if available |

#### Examples to Illustrate the Difference

##### Example 1: Calling Constructors
```cpp
class MyClass {
public:
    MyClass(int x) { std::cout << "int constructor\n"; }
    MyClass(std::initializer_list<int> list) { std::cout << "initializer_list constructor\n"; }
};

MyClass obj1(10);    // Calls MyClass(int)
MyClass obj2{10};    // Calls MyClass(std::initializer_list<int>)
```

- **Explanation**: Using parentheses calls the `int` constructor, while using braces prioritizes the `std::initializer_list<int>` constructor.

##### Example 2: Preventing Narrowing Conversions
```cpp
int x = 10.5;     // Allowed: Narrowing conversion from double to int
int y{10.5};      // Error: Narrowing conversion not allowed
```

- **Explanation**: Brace initialization prevents the narrowing conversion from `double` to `int`, ensuring type safety.

##### Example 3: Aggregate Initialization
```cpp
struct Point {
    int x, y;
};

// Brace initialization for aggregates:
Point p1{10, 20};   // Initializes the aggregate Point with values 10 and 20

// Parentheses can't be used for aggregates:
Point p2(10, 20);   // Error: No constructor in Point
```

- **Explanation**: Braces allow initializing aggregates (like `struct` and arrays), while parentheses can't.


- **Parentheses (`()`)** are used for **direct initialization** and can invoke regular constructors. They are the traditional initialization method but are prone to certain ambiguities like the Most Vexing Parse.
- **Braces (`{}`)**, introduced in C++11, provide a more consistent and safer initialization mechanism called **uniform initialization**, which prevents narrowing conversions and supports `std::initializer_list` constructors. It also helps with aggregate initialization and avoids parsing ambiguities.

The choice between the two depends on whether you need to avoid narrowing conversions, use aggregate initialization, or explicitly call an initializer list constructor.

### Prefer alias declarations to `typedef`

In C++, both **`typedef`** and **alias declarations** (introduced in C++11 with the `using` keyword) are used to define type aliases, which can make code easier to read and maintain. However, **alias declarations** are generally preferred over `typedefs` for several reasons, especially when working with **templates**.

1. **Basic Syntax**
- **`typedef`** syntax:
  ```cpp
  typedef existing_type alias_name;
  ```
  
- **Alias declaration** (using `using`):
  ```cpp
  using alias_name = existing_type;
  ```

2. **Example**
- **With `typedef`**:
  ```cpp
  typedef unsigned long ulong;
  ```

- **With `using`**:
  ```cpp
  using ulong = unsigned long;
  ```

Both are equivalent in this simple example, but `using` becomes much more advantageous with complex types and templates.

3. **Alias Declarations are More Readable**
- The `using` syntax places the alias name before the type, making it easier to read from left to right. This resembles the way variable declarations are written, making the code more intuitive.

  ```cpp
  using my_type = int*;   // Alias for a pointer to an int
  typedef int* my_type;   // Same with typedef, but the alias is less readable
  ```

4. **Alias Declarations Work Better with Templates**
- When using **templates**, `using` is far more versatile and easier to work with than `typedef`. `typedef` doesn't handle templates as cleanly and can become cumbersome.

- **Template example with `typedef`**:
  ```cpp
  template<typename T>
  typedef std::vector<T> vec;  // ERROR: typedef can't work with templates.
  ```

  **Template example with `using`**:
  ```cpp
  template<typename T>
  using vec = std::vector<T>;  // This works with using.
  ```

  This is one of the main reasons `using` is preferred — it simplifies template metaprogramming and eliminates some limitations of `typedef`.

5. **Alias Templates with `using`**
- **Alias templates** allow you to create a type alias that depends on a template parameter. This is only possible with `using` and not with `typedef`.

  **Example with `using`**:
  ```cpp
  template<typename T>
  using vec = std::vector<T>;

  vec<int> myVec;  // Equivalent to std::vector<int>
  ```

  This isn't possible with `typedef`, making `using` much more powerful and flexible for modern C++ code.

6. **Consistency and Modern Style**
- The `using` keyword is part of **modern C++** style, introduced with C++11, which emphasizes readability, simplicity, and the efficient use of templates. Consistently using `using` instead of `typedef` aligns your code with modern C++ conventions.

---

#### **Summary: Why Prefer Alias Declarations (`using`) over `typedef`**

1. **Easier to read**: `using` follows a more natural syntax (`alias = type`).
2. **Template-friendly**: `using` works with templates, while `typedef` doesn't support template aliases.
3. **Consistent with modern C++**: Aligns with the idioms of modern C++ (post-C++11).
4. **More versatile**: `using` can handle complex types and templates more effectively than `typedef`.

#### **Best Practice**

- **Use `using` for type aliases** unless you are maintaining legacy code that uses `typedef`. The versatility, clarity, and compatibility with templates make it the preferred choice in modern C++.

### Prefer scoped `enum` to unscoped `enum`

In C++, enumerations (enums) are used to define a set of named integral constants. C++11 introduced **scoped enums** (also called **`enum class`**) to address some of the limitations of traditional (unscoped) enums. In general, **scoped enums** are preferred over unscoped enums due to their type safety, better scoping, and other modern C++ advantages.

Here’s a breakdown of why you should prefer **scoped enums** over **unscoped enums**.

---

1. **Scoped vs. Unscoped Enum Syntax**
   - **Unscoped enums** (traditional C++ enums) don't have a scope, and their enumerator names can be accessed globally.
   
   ```cpp
   enum Color { Red, Green, Blue };  // Unscoped enum
   ```

   - **Scoped enums** (introduced in C++11) are declared with the `enum class` keyword. Enumerators are scoped to the enum type and don’t leak into the global scope.
   
   ```cpp
   enum class Color { Red, Green, Blue };  // Scoped enum
   ```

2. **Improved Name Scoping**
   - In unscoped enums, the enumerator names are **global** and can clash with other names.
   
   ```cpp
   enum Fruit { Apple, Orange };
   enum Color { Red, Green, Blue };   // No scope, risk of naming conflicts

   int fruit = Apple;                 // No need for enum qualifier
   ```

   - In scoped enums, the enumerator names are **scoped to the enum type**, meaning you must qualify them with the enum name, which avoids name clashes.
   
   ```cpp
   enum class Fruit { Apple, Orange };
   enum class Color { Red, Green, Blue };

   Fruit f = Fruit::Apple;            // Scoped, must use 'Fruit::'
   Color c = Color::Red;              // Scoped, must use 'Color::'
   ```

3. **Strongly Typed (Type Safety)**
   - **Unscoped enums** are essentially integers, meaning you can implicitly convert between enum values and integers, which can lead to unintended behavior.
   
   ```cpp
   enum Color { Red, Green, Blue };

   int color = Red;  // Implicit conversion to int
   if (color == 1) {  // Compiles, but not clear if 1 is a valid color
         // Ambiguity in comparisons
   }
   ```

   - **Scoped enums** are **strongly typed**, which means you can’t implicitly convert them to integers. This provides better type safety and prevents unintended conversions.
   
   ```cpp
   enum class Color { Red, Green, Blue };

   int color = Color::Red;  // ERROR: No implicit conversion to int
   if (color == 1) {        // ERROR: Can't compare enum with int
         // More explicit and clear comparisons
   }
   ```

4. **Specifying the Underlying Type**
   - In **unscoped enums**, the underlying type is implementation-defined (usually `int`), and you have no control over it unless you explicitly cast values.
   
   ```cpp
   enum Color { Red, Green, Blue };  // Typically backed by int
   ```

   - **Scoped enums** allow you to explicitly specify the underlying type, improving memory usage and clarity about the size of the enum.
   
   ```cpp
   enum class Color : char { Red, Green, Blue };  // Specify underlying type as char
   ```

   This can be useful for saving memory in certain situations or when interfacing with external systems that expect specific types.

5. **Forward Declaration Support**
   - **Unscoped enums** don't allow forward declarations unless you know the underlying type.
   
   ```cpp
   enum Color;  // ERROR: Cannot forward-declare without size information
   ```

   - **Scoped enums** can be forward-declared because you can specify the underlying type.
   
   ```cpp
   enum class Color : int;  // Scoped enums can be forward-declared
   ```

6. **No Accidental Implicit Conversions**
   - **Unscoped enums** allow implicit conversion from enumerator values to integers, which can result in accidental misuse.
   
   ```cpp
   enum Fruit { Apple, Orange };
   int fruit = Apple;  // Implicit conversion to int, risk of misuse
   ```

   - **Scoped enums** don’t allow this, so you must explicitly cast if you need an integer value. This reduces errors and makes your intent explicit.
   
   ```cpp
   enum class Fruit { Apple, Orange };
   int fruit = static_cast<int>(Fruit::Apple);  // Explicit cast needed
   ```

7. **Interoperability with Bitwise Operations**
   - **Unscoped enums** are often used in bitwise operations, but without explicit control over the underlying type, this can cause issues if the size of the enum isn't well-defined.
   
   ```cpp
   enum Flags { Read = 1, Write = 2, Execute = 4 };
   int permissions = Read | Write;  // Works, but permissions is still an int
   ```

   - **Scoped enums** can still be used for bitwise operations, but you need to specify the underlying type. This makes your intentions clearer, but you will need to use explicit casting for the operations.

   ```cpp
   enum class Flags : int { Read = 1, Write = 2, Execute = 4 };
   int permissions = static_cast<int>(Flags::Read) | static_cast<int>(Flags::Write);
   ```

#### **Summary: Why Prefer Scoped Enums (`enum class`)**
1. **Scoped names**: Enumerators don’t pollute the global namespace, preventing naming conflicts.
2. **Type safety**: Scoped enums are strongly typed, disallowing implicit conversions to integers.
3. **Explicit underlying type**: Scoped enums allow you to specify the underlying type, improving memory control and forward declaration support.
4. **Better clarity**: With scoped enums, you must qualify enumerators with the enum type, making the code more readable and explicit.
5. **No implicit integer conversions**: Scoped enums prevent accidental misuse of enum values in numeric contexts, leading to safer code.

#### **Best Practice**
- **Use scoped enums (`enum class`)** whenever possible, as they offer better scoping, type safety, and flexibility. Only use unscoped enums for legacy code or where implicit conversions are necessary for compatibility reasons.

### Prefer Deleted Functions to Private Undefined Ones

In C++, there is an old idiom used to prevent certain member functions from being called: declaring them `private` but **not providing a definition**. However, this practice is now outdated, and since **C++11**, it's better to use **deleted functions** instead. Deleted functions explicitly mark functions as unusable and provide clearer error messages at compile time.

Let’s break down the reasons why **deleted functions** are preferred over the old idiom of using **private undefined functions**.

---

1. **Deleted Functions (`= delete`)**

   - **Deleted functions** allow you to specify that a function cannot be used by marking it with `= delete`.
   
   - Example:
      ```cpp
      class MyClass {
      public:
         MyClass() = default;
         MyClass(const MyClass&) = delete;   // Copy constructor is deleted
         MyClass& operator=(const MyClass&) = delete;  // Copy assignment is deleted
      };
      
      MyClass obj1;
      MyClass obj2 = obj1;  // ERROR: Copy constructor is deleted
      ```

   - **Benefits**:
   - **Clear intent**: You explicitly indicate that a function is disabled.
   - **Early detection**: The compiler generates an error as soon as the deleted function is used, preventing misuse.
   - **Better error messages**: Modern compilers provide clearer diagnostics that tell you exactly why the function cannot be used.

2. **Private Undefined Functions (Old Idiom)**

   - In older versions of C++, the practice of disabling functions involved declaring them `private` but **not defining** them. This was commonly done for things like preventing copying or assignment.

   - Example:
      ```cpp
      class MyClass {
      private:
         MyClass(const MyClass&);            // Copy constructor (not defined)
         MyClass& operator=(const MyClass&); // Copy assignment (not defined)
      };
      
      MyClass obj1;
      MyClass obj2 = obj1;  // ERROR, but the error may be cryptic or delayed
      ```

   - **Drawbacks**:
   - **Late error detection**: The error might not appear until **link time** because the function is not defined, which can be more difficult to debug.
   - **Less clear intent**: It’s not immediately obvious why a function is declared private and without a definition.
   - **Harder to maintain**: Other programmers might expect the function to be defined later or may accidentally define it.

3. **Compile-Time Errors vs. Link-Time Errors**

   - **Deleted functions** cause **compile-time errors** as soon as the compiler encounters a call to the deleted function. This leads to:
   - **Faster detection of issues**.
   - **Clearer diagnostic messages** that explain exactly which function was deleted and why it cannot be used.

   - **Private undefined functions** often result in **link-time errors** (because the function is declared but never defined), which are:
   - **Harder to debug** since the error only occurs after linking.
   - **Potentially misleading** because the error message might just say the function is missing, without explaining why.

4. **Better Semantics with Deleted Functions**

   - Using `= delete` is part of modern C++ semantics and explicitly conveys the idea that a function is unavailable. This is much clearer to anyone reading the code than the older idiom of using private declarations.
   
   - Example: Deleting the copy constructor and assignment operator indicates that the class **cannot be copied**.
   
      ```cpp
      class NonCopyable {
      public:
         NonCopyable() = default;
         NonCopyable(const NonCopyable&) = delete;  // Cannot copy
         NonCopyable& operator=(const NonCopyable&) = delete;  // Cannot assign
      };
      ```

5. **Deleted Functions Work with All Access Specifiers**

   - Unlike the old idiom, which required you to declare the function in the **private** section, deleted functions can be marked `public` or `protected` depending on your needs, and still result in a compile-time error when used.
   
   - Example of deleting a `public` function:
      ```cpp
      class MyClass {
      public:
         void foo() = delete;  // Cannot call foo(), but it's in the public interface
      };
      
      MyClass obj;
      obj.foo();  // ERROR: foo() is deleted
      ```

6. **Deleted Special Member Functions**

   In modern C++, you often use **deleted special member functions** to prevent undesirable behaviors. These functions include:

   - **Copy constructor**: `ClassName(const ClassName&) = delete;`
   - **Copy assignment operator**: `ClassName& operator=(const ClassName&) = delete;`
   - **Move constructor** (optional): `ClassName(ClassName&&) = delete;`
   - **Move assignment operator** (optional): `ClassName& operator=(ClassName&&) = delete;`

   This ensures your objects behave as intended, e.g., making them non-copyable or non-movable.

---

##### **Summary: Why Prefer Deleted Functions**

1. **Clearer intent**: `= delete` clearly indicates that a function is not allowed, making the code easier to understand.
2. **Compile-time errors**: Deleted functions produce errors at **compile time**, leading to faster detection of issues and better diagnostic messages.
3. **More flexible**: Deleted functions can be placed in any access specifier (`public`, `protected`, or `private`), whereas the old idiom required the function to be `private`.
4. **Modern best practice**: Using `= delete` aligns with **modern C++** practices introduced in C++11, promoting cleaner, more maintainable code.

##### **Best Practice**

- **Use deleted functions (`= delete`)** to prevent function calls when you want to disable specific operations, such as copy construction or assignment. Avoid using the old idiom of private, undefined functions as it is less clear and can lead to harder-to-debug errors.

### Declare Overriding Functions with `override`

In C++, when a derived class **overrides** a virtual function from its base class, it’s highly recommended to explicitly mark that function with the **`override`** keyword. This helps ensure correctness, improves code readability, and prevents subtle bugs. The `override` specifier was introduced in **C++11** and is now considered a best practice.

Let’s explore why using `override` is important and how it improves your C++ code.

---

1. **What Does `override` Do?**

   The `override` keyword explicitly indicates that a function in a derived class **overrides** a virtual function from a base class. It helps the compiler check that you're correctly overriding a function and not making mistakes such as wrong function signatures or misnaming the function.

   - **Example Without `override`**:
      ```cpp
      class Base {
      public:
            virtual void foo() const {
               // Base class implementation
            }
      };

      class Derived : public Base {
      public:
            void foo() {  // This does NOT override, it's a new function due to missing 'const'
               // Derived class implementation
            }
      };
      ```

   Here, you might think you're overriding the `foo` function, but since the base class function is `const` and the derived class version isn’t, **it does not override**. Instead, this creates a new function in the derived class, which can lead to confusing behavior.

   - **Example With `override`**:
      ```cpp
      class Base {
      public:
            virtual void foo() const {
               // Base class implementation
            }
      };

      class Derived : public Base {
      public:
            void foo() const override {  // Correctly overrides
               // Derived class implementation
            }
      };
      ```

   Now, the derived class properly overrides the `foo` function, and the compiler will enforce that.

2. **Benefits of Using `override`**

   **a. Compile-Time Checking**

   - When you mark a function with `override`, the compiler checks that the function is indeed overriding a **virtual** function in the base class. If it doesn't, the compiler will throw an error. This helps catch mistakes early.

   - Example:
      ```cpp
      class Base {
      public:
         virtual void bar(int) {
               // Base class implementation
         }
      };

      class Derived : public Base {
      public:
         void bar(double) override {  // ERROR: No virtual function with this signature
               // Derived class implementation
         }
      };
      ```

   Here, the compiler will catch the fact that `bar(double)` does not match any virtual function in the base class, helping you correct the mistake.

   **b. Preventing Mistakes**

   - Without `override`, if you slightly change the function signature or misspell the function name, the compiler will not treat it as an override, leading to the function being **shadowed** rather than overridden. This can cause subtle bugs in the code that are hard to debug.

   - Example Without `override`:
      ```cpp
      class Base {
      public:
         virtual void print() {
               // Base class implementation
         }
      };

      class Derived : public Base {
      public:
         void prnt() {  // Mistyped 'print', this doesn't override
               // Derived class implementation
         }
      };
      ```

   - Example With `override`:
      ```cpp
      class Derived : public Base {
      public:
         void print() override {  // Properly overrides, no chance for mistakes
               // Derived class implementation
         }
      };
      ```

   With `override`, such errors are caught during compilation.

   **c. Clearer Intent**

   - Marking a function with `override` makes your **intent explicit**. Anyone reading the code can immediately see that this function is meant to override a virtual function in the base class. This improves code readability and maintainability, making it easier for others (or yourself) to understand the design.

3. **Best Practice: Always Use `override`**

   - Whenever you override a virtual function from a base class, always use `override` in the derived class. This prevents subtle bugs and ensures the compiler checks your intent.

   ```cpp
   class Base {
   public:
         virtual void draw() {
            // Base draw implementation
         }
   };

   class Derived : public Base {
   public:
         void draw() override {  // Always use 'override'
            // Derived draw implementation
         }
   };
   ```

4. **Avoid Using `virtual` in Derived Classes**

   - When using `override`, it's a common best practice **not to repeat the `virtual` keyword** in the derived class. The `override` specifier already implies that the function is virtual, so adding `virtual` is redundant and unnecessary.

   ```cpp
   class Derived : public Base {
   public:
         void draw() override {  // No need for 'virtual'
            // Derived draw implementation
         }
   };
   ```

5. **Combination with `final`**

   - You can combine `override` with `final` to explicitly indicate that a function both overrides a base class function and **cannot be overridden further** in any derived classes.
   
   - Example:
      ```cpp
      class Base {
      public:
         virtual void foo() {
               // Base implementation
         }
      };

      class Derived : public Base {
      public:
         void foo() override final {  // Overrides and prevents further overrides
               // Derived implementation
         }
      };
      ```

6. **When Not to Use `override`**

   - You should not use `override` if the function in the derived class is not intended to override any virtual function from the base class. If the base class doesn’t declare the function as `virtual`, using `override` would lead to a compilation error.

---

#### **Summary: Why Use `override`**

1. **Error prevention**: The compiler checks that your function truly overrides a base class virtual function, preventing mistakes with function signatures or names.
2. **Clear intent**: It makes it explicitly clear to other programmers (and to your future self) that the function is meant to override a base class function.
3. **Improved diagnostics**: The compiler can give more meaningful error messages if something goes wrong, making it easier to debug.
4. **Consistent with modern C++**: The `override` specifier is part of modern C++ best practices, introduced in C++11, and helps improve code quality and maintainability.

---

#### **Best Practice**

- **Always use `override`** when you intend to override a virtual function from a base class. It ensures correctness, improves readability, and aligns with modern C++ best practices.

### Prefer `const_iterator`s to `iterator`s

Using **`const_iterator`s** instead of regular **`iterator`s** in C++ is a best practice that enhances code safety and clarity, especially when working with containers from the Standard Template Library (STL). Here are some key reasons to prefer `const_iterator`s:

1. **Immutability**:
   - A `const_iterator` ensures that the elements being accessed cannot be modified. This guarantees that the underlying data remains unchanged, which is particularly important in functions where you only need to read data.

2. **Safer Code**:
   - By using `const_iterator`s, you reduce the risk of accidental modifications to the container's elements. This helps prevent bugs related to unintended side effects, making your code more robust.

3. **Clearer Intent**:
   - Choosing `const_iterator` signals to other developers (and yourself) that the intention of the code is to read data, not to modify it. This enhances code readability and maintainability.

4. **Compatibility with Range-Based Loops**:
   - When using range-based for loops, iterators can automatically deduce to `const_iterator` when iterating over `const` containers or when the loop variable is declared as `const`. This simplifies syntax and reinforces immutability.
   ```cpp
   std::vector<int> numbers = {1, 2, 3, 4, 5};
   for (const auto& num : numbers) { // num is a const reference
       std::cout << num << " ";      // Safe, can't modify num
   }
   ```

#### **Example Comparison: `const_iterator` vs. `iterator`**

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};

    // Using const_iterator
    for (std::vector<int>::const_iterator it = numbers.cbegin(); it != numbers.cend(); ++it) {
        std::cout << *it << " ";  // Safe, can't modify *it
    }
    std::cout << std::endl;

    // Using iterator (less preferred)
    for (std::vector<int>::iterator it = numbers.begin(); it != numbers.end(); ++it) {
        std::cout << *it << " ";  // Can modify *it (potentially unsafe)
    }
    std::cout << std::endl;

    return 0;
}
```

#### Conclusions

By preferring **`const_iterator`s** over regular **`iterator`s**,** you write safer, clearer, and more maintainable code. This practice ensures that the integrity of your data is preserved while conveying your intent to future readers of your code. Whenever you're not modifying the container's elements, using `const_iterator` is the way to go.

### Declare functions `noexcept` if they won’t emit exceptions

Declaring a function with the `noexcept` specifier communicates that the function is guaranteed not to throw any exceptions. This practice enhances both performance and code safety. Here’s why you should use `noexcept` when appropriate:

1. **Performance Optimization**:
   - Functions marked as `noexcept` enable the compiler to perform optimizations. The absence of exception handling can lead to reduced overhead, particularly in situations where the function is frequently called or is part of a performance-critical path.

2. **Improved Code Clarity**:
   - Using `noexcept` makes it clear to other developers (and to yourself) that the function is intended to be exception-free. This improves code readability and maintainability, as it explicitly states the function's behavior.

3. **Enhanced Safety**:
   - When a function throws an exception, and it is called in a context that cannot handle exceptions (like destructors), it can lead to program termination. Declaring a function as `noexcept` prevents this scenario, providing an extra layer of safety.

4. **Better Error Handling**:
   - If a `noexcept` function does throw an exception, `std::terminate` is called, which avoids potentially unsafe program states. This behavior ensures that errors are handled predictably.

#### **Example of Using `noexcept`**

Here's an example of how to declare functions with `noexcept`:

```cpp
#include <iostream>
#include <vector>

void processData(std::vector<int>& data) noexcept {
    // This function will not throw any exceptions
    for (auto& value : data) {
        value *= 2; // Example operation
    }
}

void mayThrow() {
    throw std::runtime_error("This function may throw!");
}

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};

    processData(numbers); // Safe, guaranteed not to throw

    try {
        mayThrow(); // This may throw an exception
    } catch (const std::exception& e) {
        std::cout << "Caught an exception: " << e.what() << std::endl;
    }

    return 0;
}
```

#### **Conclusion**

Declaring functions as `noexcept` when they are guaranteed not to throw exceptions is a best practice in C++. It enhances performance, improves code clarity, and increases safety by ensuring that exceptions are handled predictably. Whenever you implement a function that will not emit exceptions, consider using the `noexcept` specifier to convey this guarantee clearly.

### Use `constexpr` whenever possible

In C++, the `constexpr` specifier allows you to declare that a variable, function, or constructor can be evaluated at compile time. Using `constexpr` whenever possible is a best practice that offers several advantages:

1. **Compile-Time Evaluation**:
   - Functions and variables declared as `constexpr` can be evaluated at compile time, which can lead to performance improvements. This reduces runtime computation and can help in generating more efficient code.

2. **Enhanced Readability**:
   - Using `constexpr` indicates to readers of the code that the value or computation can be determined at compile time, making the code easier to understand and maintain.

3. **Type Safety**:
   - `constexpr` enforces stricter rules on the function, ensuring it can only use features that are valid in a constant expression. This helps catch errors during compilation rather than at runtime.

4. **Facilitates Optimization**:
   - By allowing more expressions to be evaluated at compile time, the compiler can make better optimization decisions, potentially leading to reduced binary sizes and faster execution.

5. **Support for Constant Expressions**:
   - You can use `constexpr` to define constants that can be used in contexts that require compile-time constants, such as template parameters or array sizes.

#### **Example of Using `constexpr`**

Here’s a simple example demonstrating the use of `constexpr`:

```cpp
#include <iostream>

// A constexpr function to compute the factorial
constexpr int factorial(int n) {
    return n <= 1 ? 1 : n * factorial(n - 1);
}

int main() {
    // Using constexpr to define a constant value at compile time
    constexpr int value = factorial(5); // Evaluated at compile time

    std::cout << "Factorial of 5 is: " << value << std::endl; // Output: 120

    return 0;
}
```

#### **Key Points to Remember**:
- **Usage**: Use `constexpr` for functions, variables, and constructors that can be evaluated at compile time.
- **Limitations**: Ensure that the function only uses `constexpr`-compatible expressions. Avoid using features like dynamic memory allocation or exceptions in `constexpr` functions.
- **Performance**: Utilizing `constexpr` effectively can lead to significant performance improvements, especially in compute-intensive applications.

#### **Conclusion**

Using `constexpr` whenever possible is a fundamental practice in modern C++. It allows for compile-time evaluation, enhances code readability, and enables optimizations that can improve performance. By declaring functions and variables as `constexpr`, you can make your code more efficient and easier to understand.

### Make `const` member functions thread safe

**`const` member functions** promise not to modify the object on which they are called, which implies thread-safety when accessing the same object across multiple threads. However, this is not always guaranteed, especially if the function accesses mutable state or external resources. To ensure **thread safety** in `const` member functions, you need to follow certain best practices and consider some key factors:

#### Key Considerations for Thread Safety:
1. **Data Races**:
   - Even though a `const` function guarantees that it won’t modify the object’s state, it doesn’t mean that other non-const functions won't modify it concurrently. Ensure that shared data accessed by multiple threads is protected.

2. **Mutable State**:
   - Be cautious of `mutable` members or internal caches, as they can be modified by `const` functions. Use proper synchronization mechanisms to guard such modifications.

3. **External Resources**:
   - Accessing external resources like I/O operations or global variables from a `const` function may not be thread-safe. In these cases, thread synchronization is necessary.

#### Strategies for Ensuring Thread Safety

1. **Use `std::mutex` to Protect Shared Data**:
   - Use a **mutex** (`std::mutex`) to prevent multiple threads from concurrently accessing shared mutable data within `const` member functions.

   Example:
   ```cpp
   #include <iostream>
   #include <mutex>
   #include <thread>

   class SafeClass {
   private:
       mutable std::mutex mtx; // Mutex to protect shared data
       int value;

   public:
       SafeClass(int v) : value(v) {}

       // Thread-safe const member function
       int getValue() const {
           std::lock_guard<std::mutex> lock(mtx); // Lock the mutex
           return value;
       }
   };

   int main() {
       SafeClass obj(42);

       // Start multiple threads accessing getValue
       std::thread t1([&obj] { std::cout << obj.getValue() << std::endl; });
       std::thread t2([&obj] { std::cout << obj.getValue() << std::endl; });

       t1.join();
       t2.join();

       return 0;
   }
   ```

   In this example, the **mutex** ensures that concurrent access to `getValue()` is safe, even though multiple threads call it simultaneously.

2. **Atomic Operations for Simple Types**:
   - For simple types like integers, you can use `std::atomic<T>` instead of `std::mutex` to guarantee atomic access without needing explicit locking.

   Example:
   ```cpp
   #include <atomic>
   #include <iostream>
   #include <thread>

   class AtomicClass {
   private:
       mutable std::atomic<int> value; // Use atomic for thread-safe access

   public:
       AtomicClass(int v) : value(v) {}

       // Thread-safe const member function
       int getValue() const {
           return value.load(); // Safe atomic access
       }
   };

   int main() {
       AtomicClass obj(42);

       // Start multiple threads accessing getValue
       std::thread t1([&obj] { std::cout << obj.getValue() << std::endl; });
       std::thread t2([&obj] { std::cout << obj.getValue() << std::endl; });

       t1.join();
       t2.join();

       return 0;
   }
   ```

   Here, `std::atomic<int>` ensures that access to `value` is thread-safe without explicit locks.

3. **Avoid Mutable Members When Possible**:
   - Mutable members allow modification inside `const` functions, which can break thread safety if not carefully managed. If you must use mutable members, protect them with synchronization mechanisms like mutexes.

#### Conclusion

To ensure that `const` member functions are thread-safe in C++, you should carefully manage access to shared data using synchronization primitives like **`std::mutex`** or **`std::atomic`**. Avoiding mutable members where possible and safeguarding external resources will further strengthen thread safety. By following these strategies, you can ensure that even `const` functions can be safely used in multithreaded environments.

### Understanding Special Member Function Generation

In C++, the compiler can automatically generate certain functions for managing objects, known as **special member functions**. These functions handle important tasks like object copying, moving, and destruction. The special member functions are:

1. **Default Constructor** (`ClassName()`)
   - Initializes an object without any arguments. If no constructor is defined, the compiler generates a default constructor.

2. **Destructor** (`~ClassName()`)
   - Cleans up resources when an object is destroyed. The compiler generates a default destructor if none is defined.

3. **Copy Constructor** (`ClassName(const ClassName&)`)
   - Creates a new object as a copy of an existing one. The compiler generates a default copy constructor that performs a member-wise copy.

4. **Copy Assignment Operator** (`ClassName& operator=(const ClassName&)`)
   - Assigns the contents of one object to another that already exists. The compiler generates a default copy assignment operator, which performs a member-wise copy.

5. **Move Constructor** (`ClassName(ClassName&&)`)
   - Transfers resources from a temporary object (rvalue) to a new object, avoiding unnecessary copies. If your class defines resource-heavy operations (like dynamic memory allocation), you should consider defining a move constructor.

6. **Move Assignment Operator** (`ClassName& operator=(ClassName&&)`)
   - Transfers resources from one object to another that already exists, again avoiding unnecessary copies. This is especially useful for optimizing performance when handling temporary objects.

---

#### **When Special Member Functions are Generated**

- **Rule of Zero**: If your class does not manage resources (e.g., raw pointers or dynamic memory), it's often best not to define any of the special member functions manually and let the compiler generate them.
  
- **Rule of Three**: If your class manages resources (e.g., dynamically allocated memory), you should explicitly define the copy constructor, copy assignment operator, and destructor to ensure proper resource handling.
  
- **Rule of Five**: In modern C++ (C++11 and beyond), if your class handles resources and needs custom copy behavior, you should also define the move constructor and move assignment operator for efficiency.

---

#### **Example: Generated vs. User-Defined Special Member Functions**

Here’s an example showing default special member function generation and custom implementations:

```cpp
#include <iostream>
#include <string>

class MyClass {
public:
    // Default constructor generated by the compiler
    MyClass() = default;

    // Destructor generated by the compiler
    ~MyClass() = default;

    // Compiler-generated copy constructor
    MyClass(const MyClass&) = default;

    // User-defined move constructor
    MyClass(MyClass&& other) noexcept {
        std::cout << "Move constructor called" << std::endl;
    }

    // User-defined copy assignment operator
    MyClass& operator=(const MyClass& other) {
        std::cout << "Copy assignment operator called" << std::endl;
        return *this;
    }

    // User-defined move assignment operator
    MyClass& operator=(MyClass&& other) noexcept {
        std::cout << "Move assignment operator called" << std::endl;
        return *this;
    }
};

int main() {
    MyClass obj1;
    MyClass obj2 = obj1;  // Copy constructor
    MyClass obj3 = std::move(obj1);  // Move constructor
    obj2 = obj3;  // Copy assignment operator
    obj2 = std::move(obj3);  // Move assignment operator
    return 0;
}
```

#### **Conclusion**

Understanding **special member function generation** is essential for writing efficient, safe, and predictable C++ programs. By default, the compiler will generate these functions when necessary, but if your class manages resources or has specific needs (e.g., dynamic memory management), you may need to define them explicitly. In modern C++, always consider the **rule of five** for optimal performance and safety.

## File Extensions

I noticed that C++ files had two (2) different type of extensions: `.cc` and `.cpp`. This got me curious to find out if there were any nuances between the two (2).

<sup><strong>NOTE</strong></sup> C++ files are also saved with the following extensions: `.cp`, `.c++`, `.cxx`.

I found [this](https://stackoverflow.com/questions/18590135/what-is-the-difference-between-cc-and-cpp-file-suffix)[^1]

<blockquote>
<p>Conventions.</p>

Historically, the suffix for a C++ source file was `.C`. This caused a few problems the first time C++ was ported to a system where case wasn't significant in the filename.

Different users adopted different solutions: `.cc`, `.cpp`, `.cxx` and possibly others. Today, outside of the Unix world, it's mostly `.cpp`. Unix seems to use `.cc` more often.

For headers, the situation is even more confusing: for whatever reasons, the earliest C++ authors decided not to distinguish between headers for C and for C++, and used `.h`.

This doesn't cause any problems if there is no C in the project, but when you start having to deal with both, it's usually a good idea to distinguish between the headers which can be used in C (`.h`) and those which cannot (`.hh` or `.hpp`).

In addition, in C++, a lot of users (including myself) prefer keeping the template sources and the inline functions in a separate file. Which, while strictly speaking a header file, tends to get yet another set of conventions (`.inl`, `.tcc` and probably a lot of others).

In the case of headers it makes absolutely no difference to the compiler.

In the case of source files different endings will cause the compiler to assume a different language. But this can normally be overridden, and I used `.cc` with VC++ long before VC++ recognized it as C++.
</blockquote>

Another good explanation was from an entry on Quora.[^2]

<blockquote>
Programming languages, like human languages, grow out of specific communities of people. The same goes for the various tools that work with programming languages.

In this case, to my knowledge, the `.cc` extension comes from the Unix culture in which Bjarne Stroustrup (the original developer of C++) was initially working. I believe the GNU gcc compiler used `.cc` as its default file extension; I imagine Bjarne considered it to be “one more ‘c’ than ‘.c’”.

The `.cpp` extension, I believe, originated at Microsoft, when it started developing its own C++ compiler. Another variation of `.cpp` is `.cxx` (the X’s represent +’s rotated 45 degrees). At Microsoft, there’s a fairly wide use of both `.cpp` and `.cxx` simply because, **1)** The choice doesn’t really matter (the compiler doesn’t care), and **2)** in any situation where a choice doesn’t really matter, pure randomness will ensure that different people will make different choices. (And therefore, **3)** there will be a lot of gratuitous debate about which arbitrary choice is the best!)

But at Microsoft I have almost never seen any C++ source files with a `.cc` extension. Back when the C++ compilers were first being developed, DOS “8.3” file naming (eight-character filename, three-character extension) was still quite common, and “C++” itself has three characters. Hence, I believe, Microsoft’s choice to use three-character extensions based on the name “C++”.

&hyphen; Rob Jellinghaus
</blockquote>

## References

1. [What is the C++ equivalent of python collections.Counter?](https://stackoverflow.com/questions/53055563/what-is-the-c-equivalent-of-python-collections-counter)
2. [Using C++ on Linux in VS Code](https://code.visualstudio.com/docs/cpp/config-linux)

## Videos

1. [Debugging a C++ project in VS Code](https://youtu.be/G9gnSGKYIg4)
2. [why do header files even exist?](https://youtu.be/tOQZlD-0Scc?si=mfygFFgj4WY47wE_)


 [^1]: [What is the difference between .cc and .cpp file suffix?](https://stackoverflow.com/questions/18590135/what-is-the-difference-between-cc-and-cpp-file-suffix)

[^2]: [Why do both .cc and .cpp file extensions exist for C++? What's the history behind this?](https://www.quora.com/Why-do-both-cc-and-cpp-file-extensions-exist-for-C-Whats-the-history-behind-this)