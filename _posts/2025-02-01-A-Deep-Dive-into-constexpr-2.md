---
title: A Deep Dive into constexpr(2/2)
author:
  name: Dennis
categories: [C++]
tags: [C++]
pin: true
use_math: true
---

Continuing from the last session, we will explore `constexpr` in more detail.  
This time, we will discuss the rules of `constexpr` functions, the differences between `constexpr` and `const`, and the advantages and disadvantages of `constexpr`.


<h1> constexpr function </h1>

A `constexpr` function is a function that **can be executed at compile time**.  
In other words, the compiler **precomputes the result** and replaces it with a constant value.

<h2> Rules for constexpr Function </h2>

A function or function template can be declared constexpr. The constexpr function should satisfy the following rules.
- It is not a [virtual](https://en.cppreference.com/w/cpp/language/virtual) function.  (until C++20)
- Its return type (if exists) is a [literal type](https://en.cppreference.com/w/cpp/language/constant_expression#Literal_type). (until C++23)
- Each of its parameter types is a literal type. (until C++23)
- It is not a [coroutine](https://en.cppreference.com/w/cpp/language/coroutines). (since C++20)
- Its function body is = default, = delete, or a compound statement [enclosing](https://en.cppreference.com/w/cpp/language/statements#Substatements) only the following: (until C++14)
    - [null statements](https://en.cppreference.com/w/cpp/language/statements#Expression_statements)
    - [`static_assert`](https://en.cppreference.com/w/cpp/language/static_assert) declarations
    - [`typedef`](https://en.cppreference.com/w/cpp/language/typedef) declarations and [alias](https://en.cppreference.com/w/cpp/language/type_alias) declarations that do not define classes or enumerations
    - [using declarations](https://en.cppreference.com/w/cpp/language/namespace#Using-declarations)
    - [using directives](https://en.cppreference.com/w/cpp/language/namespace#Using-directives)
    - exactly one [`return`](https://en.cppreference.com/w/cpp/language/return) statement if the function is not a constructor
- Its function body is = default, = delete, or a compound statement that(until C++20) does **not** [enclose](https://en.cppreference.com/w/cpp/language/statements#Substatements) the following:
    - [goto](https://en.cppreference.com/w/cpp/language/goto) statements (since C++14, until C++23)
    - statements with [labels](https://en.cppreference.com/w/cpp/language/statements#Labeled_statements) other than case and default (since C++14, until C++23)
    - [try blocks](https://en.cppreference.com/w/cpp/language/try) (until C++20)
    - [inline assembly](https://en.cppreference.com/w/cpp/language/asm) declarations (until C++20)
    - definitions of variables for which [no initialization is performed](https://en.cppreference.com/w/cpp/language/default_initialization) (until C++20)
    - definitions of variables of non-literal types (since C++14, until C++23)
    - definitions of variables of static or thread [storage duration](https://en.cppreference.com/w/cpp/language/storage_duration) (since C++14, until C++23)


<h3> 1. It is not a virtual function.  (until C++20) </h3>

In C++, a constexpr function must be evaluable at compile time, and its result must be determinable during compilation. Due to this characteristic, constexpr functions have certain restrictions. One of these restrictions is that a constexpr function cannot be a virtual function. This is because the dynamic binding nature of virtual functions conflicts with the requirement for compile-time evaluation in constexpr.
```cpp
class Base
{
public:
    virtual int getValue() const { return 5; }
};

class Derived : public Base {
public:
    int getValue() const override { return 10; }
};

constexpr int getValue(const Base& obj)
{
    return obj.getValue(); // error: C3615 constexpr function 'getValue' cannot result in a constant expression
}

int main()
{
    Derived d;
    int val = getValue(d);
    return 0;
}
```
Since constexpr functions must be evaluated at compile time, they cannot have the dynamic nature of virtual functions. This is because the function call must be resolved at compile time rather than at runtime. Therefore, a constexpr function must not be a virtual function, and constexpr cannot be used within a class hierarchy that includes virtual functions. This restriction is an important consideration when understanding and applying constexpr functions.

<h3> 2. Its return type (if exists) is a literal type. (until C++23) </h3>

In C++, constexpr functions must follow several important rules. One of these rules is that the return type of the function must be a literal type. This ensures that the return value of a constexpr function can be evaluated at compile time.
```cpp
#include <iostream>

class Point
{
public:
    constexpr Point(double x, double y) : x_(x), y_(y) {}

    constexpr double getX() const { return x_; }
    constexpr double getY() const { return y_; }

private:
    double x_;
    double y_;
};

constexpr Point createPoint(double x, double y)
{
    return Point(x, y);  // Point는 리터럴 타입, 생성자가 constexpr
}

int main()
{
    constexpr Point p = createPoint(3.5, 2.5);
    std::cout << "Point: (" << p.getX() << ", " << p.getY() << ")" << std::endl;
    return 0;
}
```
In this code, the `Point` class is a **literal type** because it has a `constexpr` constructor and a trivial destructor. The `createPoint` function creates and returns such a `Point` object, and since it is also declared as `constexpr`, it can generate and initialize the object at compile time.

The rule that requires the return type of a `constexpr` function to be a **literal type** ensures that the function can be safely used at compile time. This means that all types used with `constexpr` functions must be evaluable at compile time, guaranteeing the accuracy and predictability of `constexpr` expressions.

<h3> 3. Each of its parameter types is a literal type. (until C++23) </h3>

The parameter types passed to a function must all be **literal types**.
Below is a simple example of a `constexpr` function that uses **literal type** parameters:
```cpp
#include <iostream>

constexpr int multiply(int x, int y)
{
    return x * y;
}

int main()
{
    constexpr int result = multiply(5, 4);
    std::cout << "The result is: " << result << std::endl;
    return 0;
}

```
In this example, the **`multiply`** function takes two **`int`** parameters. Since **`int`** is a primitive data type, it qualifies as a **literal type**. The computed result of the function is also declared as **`constexpr`**, ensuring that it is evaluated at compile time.

The requirement that all parameters of a **`constexpr`** function must be **literal types** is a crucial rule that guarantees the function can be evaluated at compile time. This enables **`constexpr`** functions to operate more efficiently by reducing execution time and optimizing resource usage.

By using **literal type** parameters, a **`constexpr`** function can complete all calculations at compile time rather than at runtime, significantly improving program performance.

<h3> 4. It is not a coroutine. (since C++20) </h3>

**Coroutines in C++20 and Their Incompatibility with `constexpr` Functions**<br>
Coroutines, introduced in C++20, are a powerful feature that enables seamless implementation of asynchronous programming, data stream processing, and asynchronous I/O operations. However, since `constexpr` functions and coroutines have different constraints and execution environments, the C++ standard explicitly states that a `constexpr` function cannot be a coroutine.

**Characteristics of Coroutines**<br>
A coroutine is a function that can **pause execution and resume later**, meaning it can maintain state and have multiple entry points. Coroutines are implemented using the **`co_yield`**, **`co_return`**, and **`co_await`** keywords. Thanks to these characteristics, a coroutine can resume execution from the point where it was previously suspended, allowing for efficient asynchronous execution.

**Conflict Between Coroutines and `constexpr`**<br>
One of the key properties of coroutines is their ability to maintain state, pause, and resume execution dynamically. This dynamic nature directly contradicts the static and predictable nature of `constexpr` functions. Coroutines modify their internal state during execution, can produce different outputs for the same input, and must retain execution context.

Due to these fundamental differences, **a `constexpr` function cannot be a coroutine** in C++.

<h3> 5. Its function body is = default, = delete, or a compound statement enclosing only the following: (until C++14) </h3>

- [null statements](https://en.cppreference.com/w/cpp/language/statements#Expression_statements)
- [`static_assert`](https://en.cppreference.com/w/cpp/language/static_assert) declarations
- [`typedef`](https://en.cppreference.com/w/cpp/language/typedef) declarations and [alias](https://en.cppreference.com/w/cpp/language/type_alias) declarations that do not define classes or enumerations
- [using declarations](https://en.cppreference.com/w/cpp/language/namespace#Using-declarations)
- [using directives](https://en.cppreference.com/w/cpp/language/namespace#Using-directives)
- exactly one [`return`](https://en.cppreference.com/w/cpp/language/return) statement if the function is not a constructor
    ```cpp
    #include <iostream>
    #include <type_traits>

    // constexpr 함수 정의
    constexpr int square(int n)
    {
        static_assert(std::is_integral<int>::value, "Input must be an integer");  // static_assert 선언
        using Integer = int;  // using declarations, typedef 또는 alias 선언
        using std::cout;  // using directives

        return n * n;  // 정확히 하나의 return 문
    }

    int main()
    {
        constexpr int result = square(5);  // 컴파일 시간에 계산
        std::cout << "The square of 5 is " << result << std::endl;
        return 0;
    }
    ```


<h3> 6.  Its function body is = default, = delete, or a compound statement that(until C++20) does not enclose the following: </h3>

- [goto](https://en.cppreference.com/w/cpp/language/goto) statements (since C++14, until C++23)
- statements with [labels](https://en.cppreference.com/w/cpp/language/statements#Labeled_statements) other than case and default (since C++14, until C++23)
- [try blocks](https://en.cppreference.com/w/cpp/language/try) (until C++20)
- [Inline assembly](https://en.cppreference.com/w/cpp/language/asm) declarations (until C++20)
- Definitions of variables for which [no initialization is performed](https://en.cppreference.com/w/cpp/language/default_initialization) (until C++20)
- Definitions of variables of non-literal types (since C++14, until C++23)
- Definitions of variables of static or thread [storage duration](https://en.cppreference.com/w/cpp/language/storage_duration) (since C++14, until C++23)



<h4>6-1. goto statements (since C++14, until C++23)</h4>

The goto statement is used to control the program flow by directly jumping to a specific label, altering the execution sequence. Such jumps make it difficult to follow program logic and can create hard-to-understand code, especially within complex functions. The use of goto statements in constexpr functions is prohibited for the following key reasons:

**Increased Complexity in Compile-Time Evaluation**<br>
One of the main purposes of `constexpr` functions is to evaluate functions at compile time, reducing runtime computations. However, `goto` statements make execution flow **unpredictable**, making it difficult for the compiler to determine the function’s result at compile time. This can hinder the fundamental purpose of `constexpr` functions.

**Reduced Code Readability and Maintainability**<br>
`goto` statements cause **nonlinear execution flow**, making it harder to understand the program structure. `constexpr` functions should be as simple and predictable as possible, and `goto` statements compromise this clarity and simplicity. Using `goto` inside a `constexpr` function requires more time and effort to track and understand the function’s flow, making maintenance and debugging more challenging.

**Interference with Compiler Optimizations**<br> 
The compiler analyzes execution flow to optimize code efficiently. `goto` statements complicate this flow analysis, hindering the compiler’s ability to generate efficient code. Since `constexpr` functions perform **compile-time calculations**, their execution flow must be **clear and predictable**.

Due to these reasons, `goto` statements are not allowed in `constexpr` functions. They introduce **unpredictability**, **reduce maintainability**, and **interfere with compile-time evaluation and optimizations**, all of which go against the purpose of `constexpr` functions in C++.

<h4>6-2. statements with labels other than case and default (since C++14, until C++23)</h4>

The use of **`goto` statements** and related **labels** inside `constexpr` functions is prohibited. This rule applies from **C++14 to C++23** and is intended to maintain the **purity and predictability** of `constexpr` functions.

This restriction **prohibits all labels** except for `case` and `default` labels used in `switch` statements. Since these labels are typically used in combination with `goto` statements, this rule effectively falls under the same category as restricting the use of `goto` itself.

This rule belongs to the same category as the **"goto statements (since C++14, until C++23)"** restriction. Both rules **prohibit the use of `goto` statements and their supporting labels**, ensuring that `constexpr` functions are evaluated **accurately and consistently** at compile time. These restrictions help maintain **logical and consistent execution flow**, allowing the compiler to effectively evaluate functions during compilation.

<h4> 6-3. try blocks (until C++20) </h4>

```cpp
#include <iostream>
#include <stdexcept>

constexpr int divide(int a, int b)
{
    try
    {
        if (b == 0)
        {
            throw std::invalid_argument("Division by zero");
        }
        return a / b;
    }
    catch (const std::invalid_argument& e)
    {
        std::cerr << "Error: " << e.what() << std::endl;
        return 0; // Or throw a different exception that can be handled at compile time
    }
}

int main() {
    constexpr int result1 = divide(10, 2); // This calculation can be performed at compile time.
    constexpr int result2 = divide(5, 0); // Error C2131 expression did not evaluate to a constant	

    std::cout << "Result 1: " << result1 << std::endl;
    std::cout << "Result 2: " << result2 << std::endl;

    return 0;
}
```
The divide function is a constexpr function that divides two integers.
Within a try block, it performs the division operation and throws a std::invalid_argument exception if the denominator is 0.
The catch block catches the exception, prints an error message, and returns a default value of 0.

In the main function, the divide function is called with a constexpr variable, ensuring that the computation occurs at compile time.
If the denominator is 0, an exception is thrown and handled within the catch block, causing the error message to be displayed at compile time, and result2 is initialized to 0.

**⚠️ Important Considerations**<br>
Exceptions thrown within a constexpr function must be handled at compile time.
If the catch block throws a new exception, that exception must also be resolvable at compile time.
While using try and catch inside constexpr functions is useful for compile-time exception handling, excessive use may increase code complexity and should be managed carefully.
This example demonstrates how to use try and catch blocks in constexpr functions starting from C++20.
By leveraging compile-time exception handling, it enables writing more robust and reliable code.

In C++20, calling `divide(5, 0)` results in **Error C2131: expression did not evaluate to a constant**.
Before C++20, the above code would not work.
before C++20
```cpp
Error	C3615	constexpr function 'divide' cannot result in a constant expression
```

<h4> 6-4. Inline assembly declarations (until C++20) </h4>

A constexpr function must be composed entirely of constant expressions that can be evaluated at compile time. Since inline assembly cannot be interpreted and executed at compile time, it cannot be used directly within a constexpr function.
```cpp
#include <iostream>

constexpr int add_asm(int a, int b)
 {
    int result = 0;
    __asm {				//inline assembly is not available 
        mov eax, a
        add eax, b
        mov result, eax
    }
    return result;
}

int main()
{
    constexpr int result = add_asm(5, 3);
    std::cout << "Result: " << result << std::endl;
    return 0;
}
```
In the example above, the add function is a constexpr function that adds two integers. However, attempting to perform the addition using inline assembly will result in a compilation error.
```
Error	C2131	expression did not evaluate to a constant	
```


<h4> 6-5. Definitions of variables for which no initialization is performed (until C++20) </h4>

Before C++20, uninitialized variables should be avoided within constexpr functions. Variables had to be explicitly initialized before use or initialized with a constant expression whose value is determined at compile time.

```cpp
#include <iostream>

constexpr int calculate(int a)
{
  int result = 0; // local variable initialized.
  result = a * 2;
  return result;
}

int main()
{
  constexpr int value = calculate(5);
  std::cout << "Value: " << value << std::endl;
  return 0;
}
```
Starting from C++20, the restriction on using uninitialized variables in **`constexpr`** functions has been relaxed. It is now possible to declare a variable within a **`constexpr`** function and assign a value to it later. However, the assigned value must still be a constant expression that can be determined at compile time.


```cpp
#include <iostream>

constexpr int calculate(int a)
{
  int result; // An uninitialized local variable
  if (a > 0)
  {
    result = a * 2; // The value is determined at compile time.
  } 
  else
  {
    result = 0;
  }
  return result;
}

int main()
{
  constexpr int value = calculate(5);
  std::cout << "Value: " << value << std::endl;
  return 0;
}
```

<h4> 6-6. Definitions of variables of non-literal types (since C++14, until C++23) </h4>

All variables declared within a **`constexpr`** function must be of **literal type**. Defining a non-literal type variable inside a **`constexpr`** function is not allowed from **C++14 to C++23**.
A **non-literal type** cannot be safely processed at compile time because it may require **dynamic allocation** or contain **complex user-defined constructors and destructors**.

```cpp
#include <iostream>
#include <string>

class NonLiteral
{
public:
    NonLiteral() { std::cout << "NonLiteral constructor called\n"; }
    ~NonLiteral() { std::cout << "NonLiteral destructor called\n"; }
};

constexpr int useNonLiteral() {
    NonLiteral nl;  // E2660	variable type "NonLiteral" in constexpr function is not a literal typ

    return 42;
}

int main() {
    int result = useNonLiteral();
    std::cout << "Result: " << result << std::endl;
    return 0;
}
```
This code will produce a compilation error. Since the NonLiteral class is a `non-literal` type, it cannot be used inside a `constexpr` function. This class includes user-defined constructors and destructors, which are `non-trivial` and thus disqualify it from being a `literal type`.


<h4> 6-7. Definitions of variables of static or thread storage duration (since C++14, until C++23) </h4>

In C++, constexpr functions are designed to be evaluated at compile time. To meet this requirement, constexpr functions cannot define variables with static or thread-local storage duration. Variables with these storage durations retain their state across multiple function calls, which conflicts with the requirements of constexpr functions that must be evaluated at compile time.

**Static and Thread-Local Storage Duration**

**Static Storage Duration**: Variables with static storage duration persist in memory from the start to the end of program execution and retain their values across multiple function calls.

**Thread-Local Storage Duration**: Variables with thread_local storage duration have unique instances for each thread, and their values are maintained only within the corresponding thread.

The following code will produce a compilation error because the getStaticValue and getThreadLocalValue functions define static and thread_local variables, respectively. Such variables are not allowed in constexpr functions.
```cpp
#include <iostream>

constexpr int getStaticValue()
{
    static int value = 0;  // E2661	variable in constexpr function does not have automatic storage duration	
    value++;
    return value;
}

constexpr int getThreadLocalValue()
{
    thread_local int threadValue = 0;  // E2661	variable in constexpr function does not have automatic storage duration
    threadValue++;
    return threadValue;
}

int main()
{
    int staticVal = getStaticValue();
    int threadVal = getThreadLocalValue();
    std::cout << "Static Value: " << staticVal << std::endl;
    std::cout << "Thread Local Value: " << threadVal << std::endl;
    return 0;
}
```
A constexpr function cannot include variables that retain state across function calls. This requirement ensures that each function call is independent and that all results can be determined at compile time. Variables with static or thread-local storage duration do not meet these requirements, which is why they cannot be used inside constexpr functions.

<br>

<h1> const vs constexpr </h1>

`const` and `constexpr` are both important keywords in C++, but they differ in **usage and scope**.

The primary difference between them is that **`const`** indicates that a variable **cannot be modified**, whereas **`constexpr`** signifies that a variable or function **can be evaluated at compile time**. Therefore, the main reason for using **`constexpr`** is **performance optimization**.

<h2> const </h2>

`const` informs the compiler that a variable cannot be modified after it is declared. It is primarily used to clarify the intent of the program and **prevent unintended modifications**.

For example, `const` can be used to ensure that a function parameter cannot be altered within the function, providing additional safety.

```cpp
void print_array(const int* arr, size_t size)
{
    for (size_t i = 0; i < size; ++i)
    {
        std::cout << arr[i] << " ";
    }
    // arr[i] = 5; // This code occurs compile error.
}
```

<h2> constexpr </h2>

`constexpr` specifies that a value can be evaluated at compile time. This is useful for optimizing a program's execution time and memory usage. For example, declaring frequently used constant values as constexpr allows them to be directly used across multiple parts of the program, improving runtime performance.
```cpp
constexpr int factorial(int n)
{
    return n <= 1 ? 1 : n * factorial(n - 1);
}

int main()
{
    int arr[factorial(5)];  // This expression is evaluated as arr[120] at compile time.
}
```

<br>

<h1> Advantages and Disadvantages of Using constexpr </h1>

<h2> Advantages </h2>

- **Compile-Time Optimization**:
    **`constexpr`** ensures that a variable or function is evaluated at **compile time**, reducing the need for runtime calculations. This helps improve **program startup time** and overall **execution performance**.
    
- **Memory Usage Optimization**:
    **`constexpr`** variables can be replaced with literal values where necessary, potentially reducing **executable file size**. Since their values are determined at compile time, no additional memory allocation is required at runtime.
    
- **Improved Security and Stability**:
    Since **`constexpr`** functions and variables are evaluated at **compile time**, unexpected runtime behavior is minimized. This increases code stability and predictability.
    

Therefore, when considering **performance optimization and stability**, it is recommended to use **`constexpr`** whenever possible. By leveraging **`constexpr`**, you can reduce execution time, optimize memory usage, and handle more logic at compile time, ultimately improving the overall efficiency of the program.


<h2> Disadvantages </h2>

- Difficult to Debug:Since constexpr functions are executed at compile time, debugging them can be difficult or even impossible.
    ```cpp
    #include <iostream>

    constexpr int Product_cxx14(int n)
    {
        if (n % 2 == 0)
        {
            return n * n;
        }
        else
        {
            return n * n * n;
        }
    }

    int main()
    {
        constexpr int n2 = Product_cxx14(2);
        std::cout << n2 << std::endl;
    }
    ```

The `Product_cxx14` function in the code is a `constexpr` function, so it **cannot be debugged**. Even if you set a **breakpoint** inside the function, it will **not be triggered**.

<br>

<h1> Conclusion </h1>

So far, we took an in-depth look at **`constexpr`**. While `const` and `constexpr` may appear similar in syntax, they serve different purposes and roles in C++.

- **`const`** is used to declare **immutable variables** and can be applied at **runtime**.
- **`constexpr`** ensures that **an expression can be evaluated at compile time**, focusing on **reducing execution time and maximizing optimization**.

Since these two keywords have distinct usage purposes and scopes, it is crucial to choose the appropriate one based on code performance and maintainability.

C++ continues to evolve, and since C++20, `constexpr` has been further expanded, increasing its utility. Therefore, ongoing research and learning are essential to effectively leverage `constexpr` in the latest C++ standards.

I hope this article has helped you understand the concept and practical applications of `constexpr`, and I look forward to sharing more in-depth content in the future. 


<br>

<h2> References </h2>

[constexpr specifier (since C++11) - cppreference.com](https://en.cppreference.com/w/cpp/language/constexpr)
