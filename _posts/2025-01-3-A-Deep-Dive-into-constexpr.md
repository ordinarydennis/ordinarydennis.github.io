---
title: A Deep Dive into constexpr
author:
  name: Dennis
categories: [C++]
tags: [C++]
pin: true
use_math: true
---


As you develop software, you may often come across the `const` and `constexpr` keywords. At first glance, both seem to represent "constants," but in reality, their subtle differences can significantly impact code performance and execution.
"Why should I use `constexpr`?", "Isn't `const` sufficient everywhere?"
If you've ever had these questions, this article is for you. Today, we'll dive deep into the core concepts of `constexpr`, explore its practical applications, and illustrate its power with engaging examples.

<br>


<h1>const</h1>
First, let's take a brief look at const.
As developers, we often need variables that do not change.

```cpp
#include <iostream>
#include <string>

int main()
{
	std::string programVersion = "1.0.0";

	std::string userName;
	std::cin >> userName;

	//...some source code
	
	//This is coding mistake, original intention is 
	//std::string programUserName = userName;
	programVersion = userName;

	std::cout << programVersion << std::endl; //output wrong information 

	return 0;
}

```

Program version information should not change while the program is running.
Imagine accidentally storing a user-inputted name in the program version variable. In this case, the version information could become corrupted. This is a more common mistake than you might think.

```cpp
const std::string programVersion = "1.0.0";
```
The solution is simple. Just change the programVersion variable to a constant.

```cpp
{
...
		programVersion = userName; // The error occures bacause programVersion is const variable.
...
}
```

```
Error	C2678	binary '=': no operator found which takes a left-hand operand of type 'const std::string' (or there is no acceptable conversion)	test	C:\Users\Dennis\Desktop\study\test\test\constexpr.cpp	12	
```
Since a constant cannot be modified, an error message like the one above will appear if you attempt to change it.

<br>

<h1>constexpr</h1>

Now that we have briefly covered the `const` keyword, let's dive into the `constexpr` keyword in detail.

The constexpr specifier declares that it is possible to evaluate the value of the entities at compile time. Such entities can then be used where only compile time constant expressions are allowed (provided that appropriate function arguments are given).


<h2> constexpr as a constant </h2>

A constexpr specifier used in an object declaration or non-static member function(until C++14) implies const.

```cpp
class Point {
public:
    constexpr Point(double x, double y) : x_(x), y_(y) {}

    constexpr double getX() const { return x_; } // implies const in C++11, C++14
    constexpr double getY() const { return y_; } // implies const in C++11, C++14

private:
    double x_;
    double y_;
};

constexpr Point p(1.0, 2.0);
```

Before C++14, constexpr enforced that objects and non-static member functions had const characteristics. This ensured that the function or object remained unchanged throughout the program's execution. After C++14, constexpr functions gained more flexibility, but the fundamental concept related to const remains the same.


<h2> constexpr as a inline </h2>

A constexpr specifier used in the first declaration of a function or static data member(since C++17) implies inline. If any declaration of a function or function template has a constexpr specifier, then every declaration must contain that specifier.

```cpp
class MathConstants
{
public:
    static constexpr double PI = 3.14159; // implies inline
};

int main()
{
    double circleArea = MathConstants::PI * 10 * 10;
    return 0;
}
```
In the example above, the declaration of static constexpr double PI = 3.14159; in the MathConstants class ensures that PI is a static data member initialized at compile time. Since constexpr implies inline starting from C++17, PI is accessible across different translation units. This means that the constant can be used in any file within the program.
(One of the primary purposes of the inline specifier in C++ is to resolve the issue of multiple definitions of the same function or variable across different translation units. This helps prevent linker errors and improves code reusability.)

```cpp
struct Calculator
{
    static constexpr int add(int a, int b)
    {
        return a + b;
    }
};

int main()
{
    int sum = Calculator::add(5, 3); // This function call can be evaluated at compile time.
    return 0;
}
```

In this case, the add function of the Calculator struct is declared as static constexpr, allowing it to be evaluated at compile time. Additionally, since C++17, constexpr functions are automatically treated as inline, meaning this function can also be accessed across different translation units. By being inlined, the function call is replaced with the actual code, reducing execution time.
<br><br><br>

<h1> constexpr variable </h1>

A `constexpr` variable in C++ is used to declare a variable whose value is determined at compile time and can be used as a constant throughout the program.

**The variable of a primitive type.**
```cpp
constexpr int max_size = 100;  // constexpr variable of primitive type
constexpr double pi = 3.14159; // constexpr variable of primitive type

int main()
{
    int myArray[max_size];  // An array whose size is determined at compile time
    double circumference = 2 * pi * 10; // An expression that can be evaluated at compile time
    return 0;
}
```
In this example, max_size and pi are determined at compile time and remain constant throughout the program's execution. The size of myArray is also determined at compile time using max_size.

**The variable of a class type.**
```cpp
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

constexpr Point origin(0.0, 0.0); // constexpr variable of classtype

int main()
{
    double x = origin.getX(); // can be evaluated at compile time
    double y = origin.getY(); // can be evaluated at compile time
    return 0;
}
```

The `Point` class has a `constexpr` constructor, and the origin object is initialized at compile time. The member functions `getX()` and `getY()` of origin are also declared as `constexpr`, allowing their values to be retrieved at compile time.



<h2> Rules for constexpr Variables </h2>

`constexpr` variables are useful for storing values that need to be computed at compile time to optimize program performance. This helps minimize execution time and memory usage by performing complex calculations at compile time, reducing runtime overhead. A variable can be declared as constexpr if it meets the following conditions.

- The declaration is a [definition](https://en.cppreference.com/w/cpp/language/definition).
- It is of a [literal type](https://en.cppreference.com/w/cpp/language/constant_expression#Literal_type).
- It is initialized (by the declaration).
- The full-expression of its initialization is a constant expression. (until C++26)
- It is [constant-initializable](https://en.cppreference.com/w/cpp/language/constant_expression#Constant-initialized_entities). (since C++26)
- It has constant destruction, which means one of the following conditions needs to be satisfied: (since C++20)
  - It is not of class type nor (possibly multi-dimensional) array thereof
  - It is of a class type with a constexpr destructor or (possibly multi-dimensional) array thereof, and for a hypothetical expression e whose only effect is to destroy the object, e would be a [core constant expression](https://en.cppreference.com/w/cpp/language/constant_expression#Core_constant_expression) if the lifetime of the object and its non-mutable subobjects (but not its mutable subobjects) were considered to start within e.

If a constexpr variable is not [translation-unit-local](https://en.cppreference.com/w/cpp/language/tu_local), it should not be initialized to refer to a translation-unit-local entity that is usable in constant expressions, nor have a subobject that refers to such an entity. Such initialization is disallowed in a [module interface unit](https://en.cppreference.com/w/cpp/language/modules) (outside its [private module fragment](https://en.cppreference.com/w/cpp/language/modules#Private_module_fragment), if any) or a module partition, and is deprecated in any other context.



<h3> 1. The declaration is a definition. </h3>

To use the `constexpr` specifier, the declaration and definition must be done simultaneously.
```cpp
constexpr std::string programVersion = "1.0.0";
```

<h3> 2. It is of a literal type. </h3>

In C++, a literal type refers to a type whose value can be determined at compile time. Since these types must be initialized and used at compile time, they need to satisfy certain conditions.
[constant_expression - cppreference](https://en.cppreference.com/w/cpp/language/constant_expression#Literal_type)

1. **Primitive Types**: Most primitive data types are literal types. For example, int, char, float, double, and bool are all considered literal types. These types can be used as constants at compile time.
    ```cpp
    // constexpr for primitive type
    constexpr int n = 1;
    constexpr char c = 'A';
    constexpr float f = 1.0f;
    constexpr double d = 10.0;
    ```
2. **Literal Type Structures and Classes:** For a structure or class to be considered a literal type, it must satisfy the following conditions:
   - All data members must be **literal types**.
   - The class must have at least one **`constexpr` constructor**, which must be usable for object initialization.
   - It must have a **non-virtual destructor**.
   - It must not contain a **non-literal base class** or **virtual functions**.
    ```cpp
    // A literal class
    class conststr
    {
        const char* p;
        std::size_t sz;
    public:
        template<std::size_t N>
        constexpr conststr(const char(&a)[N]): p(a), sz(N - 1) {}
        ~ conststr() = default;  //
    
        // constexpr functions signal errors by throwing exceptions
        // in C++11, they must do so from the conditional operator ?:
        constexpr char operator[](std::size_t n) const
        {
            return n < sz ? p[n] : throw std::out_of_range("");
        }
    
        constexpr std::size_t size() const { return sz; }
    };
    ```
3. **Enumeration Types**: All enumeration types (enum) are also literal types.
    ```cpp
    enum Color { Red, Green, Blue };
    constexpr Color favoriteColor = Green;   
    ```
4. **Array Types**: An array whose elements are literal types is also considered a literal type.
   ```cpp
   constexpr int size = 5;  // Literal type integer
   constexpr int factorials[size] = {  // An array which has elements literal type
        1, 2, 3, 4, 5 
    };
   ```


<h3> 3. It is initialized (by the declaration). </h3>

This is similar to "1. The declaration is a definition." A constexpr variable must be initialized at the time of its declaration.
```cpp
constexpr std::string programVersion = "1.0.0";
```

<h3> 4. The full-expression of its initialization is a constant expression. (until C++26) </h3>

The expression used to initialize must be completely evaluated at compile time.
```cpp
#include <iostream>

constexpr int square(int x)
{
    return x * x;
}

int main()
{
    constexpr int size = 5;
    constexpr int squares[size] = {square(1), square(2), square(3), square(4), square(5)};

    for (int i = 0; i < size; ++i)
    {
        std::cout << "Square of " << (i + 1) << " is " << squares[i] << std::endl;
    }

    return 0;
}
```
**`square` Function:** Defined as a `constexpr` function, it takes an integer as input and returns its square. Since this function returns a constant expression, it can be invoked at compile time.

**`squares` Array:** This is a `constexpr` array that is initialized at compile time, with each element being initialized using the result of the `square` function. All expressions used for the array's initialization are constant expressions.

<h3> 5. It is constant-initializable. (since C++26) </h3>

This expression means that a variable or object can be initialized as a constant. Here, **"can be initialized as a constant"** means that the variable or object is either declared using the `const` qualifier or can be treated as such in a given context. This may be related to `constexpr`, or it may simply apply to a `const` variable.

The term **"constant-initializable"** can be used when the initial value is either known at compile time or determined at runtime but must remain immutable.
```cpp
constexpr int x = 5 + 3; // Full-expression of its initialization is a constant expression
```

In the example above, x is declared as constexpr, and the initialization expression 5 + 3 is a constant expression that can be evaluated at compile time.
```cpp
const int y = 5 + 3; // It is constant-initializable
```
Here, y is declared as const and initialized with 5 + 3. This initialization can be resolved at compile time, and y cannot be modified during runtime.

**The following are situations where `constant-initialization` is not possible.**

- The case where the value to be initialized is evaluated at runtime.
    ```cpp
    #include <iostream>

    int main()
    {
        const int userValue; // error: A const variable must be initialized at the time of its declaration.
        std::cin >> userValue;
        return 0;
    }
  ```
  This is a case where the initial value of a variable must be determined at runtime and cannot be known at compile time. For example, values obtained from user input or read from a file cannot be known during compilation.
- When the value used for initialization contains a non-constant expression.
    ```cpp
    int a = 10; // non-constant
    const int b = a; // It is not an error, but it does not satisfy the **"constant-initializable"** condition.
    ```
    In the example above, `b` is initialized using the value of `a`, but `a` is not a constant. Therefore, since `b`'s initialization depends on `a`'s value, it is difficult to consider `b` as **"constant-initializable."**
    Even in this case, `b` is declared as `const`, so no compilation error occurs. However, this is not an ideal case of constant initialization.
- When the return value of a function cannot be initialized as a constant.
    ```cpp
    #include <iostream>
    #include <random>

    int getRandomNumber()
    {
        std::random_device rd;
        std::mt19937 gen(rd());
        std::uniform_int_distribution<> dis(1, 100);
        return dis(gen);
    }

    int main() {
        const int randomValue = getRandomNumber(); // It is initialized as constant, but it is not 'constant-initializable'
        return 0;
    }
    ```

<br><br>

<h1> constexpr function </h1>

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


<br>

<h2> 1. It is not a virtual function.  (until C++20) </h2>

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

<br>

<h2> 2. Its return type (if exists) is a literal type. (until C++23) </h2>

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

<br>

<h2> 3. Each of its parameter types is a literal type. (until C++23) </h2>

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

<br>

<h2> 4. It is not a coroutine. (since C++20) </h2>

**Coroutines in C++20 and Their Incompatibility with `constexpr` Functions**

Coroutines, introduced in **C++20**, are a powerful feature that enables seamless implementation of **asynchronous programming, data stream processing, and asynchronous I/O operations**. However, since `constexpr` functions and coroutines have different constraints and execution environments, the **C++ standard explicitly states that a `constexpr` function cannot be a coroutine**.

**Characteristics of Coroutines**

A coroutine is a function that can **pause execution and resume later**, meaning it can maintain state and have multiple entry points. Coroutines are implemented using the **`co_yield`**, **`co_return`**, and **`co_await`** keywords. Thanks to these characteristics, a coroutine can resume execution from the point where it was previously suspended, allowing for efficient asynchronous execution.

**Conflict Between Coroutines and `constexpr`**

One of the key properties of coroutines is their ability to **maintain state, pause, and resume execution dynamically**. This **dynamic nature directly contradicts the static and predictable nature of `constexpr` functions**. Coroutines modify their internal state during execution, can produce different outputs for the same input, and must retain execution context.

Due to these fundamental differences, **a `constexpr` function cannot be a coroutine** in C++.


<br>

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

<br>

<h3> 6.  Its function body is = default, = delete, or a compound statement that(until C++20) does not enclose the following: </h3>

- [goto](https://en.cppreference.com/w/cpp/language/goto) statements (since C++14, until C++23)
- statements with [labels](https://en.cppreference.com/w/cpp/language/statements#Labeled_statements) other than case and default (since C++14, until C++23)
- [try blocks](https://en.cppreference.com/w/cpp/language/try) (until C++20)
- [Inline assembly](https://en.cppreference.com/w/cpp/language/asm) declarations (until C++20)
- Definitions of variables for which [no initialization is performed](https://en.cppreference.com/w/cpp/language/default_initialization) (until C++20)
- Definitions of variables of non-literal types (since C++14, until C++23)
- Definitions of variables of static or thread [storage duration](https://en.cppreference.com/w/cpp/language/storage_duration) (since C++14, until C++23)


<br>

<h4>6-1. goto statements (since C++14, until C++23)</h4>
The goto statement is used to control the program flow by directly jumping to a specific label, altering the execution sequence. Such jumps make it difficult to follow program logic and can create hard-to-understand code, especially within complex functions. The use of goto statements in constexpr functions is prohibited for the following key reasons:

<br>

**Increased Complexity in Compile-Time Evaluation**
<br>
One of the main purposes of `constexpr` functions is to evaluate functions at **compile time**, reducing runtime computations. However, `goto` statements make execution flow **unpredictable**, making it difficult for the compiler to determine the function’s result at compile time. This can **hinder the fundamental purpose** of `constexpr` functions.

<br>

**Reduced Code Readability and Maintainability**
<br>
`goto` statements cause **nonlinear execution flow**, making it harder to understand the program structure. `constexpr` functions should be **as simple and predictable as possible**, and `goto` statements compromise this clarity and simplicity. Using `goto` inside a `constexpr` function requires more time and effort to track and understand the function’s flow, making maintenance and debugging more challenging.

<br>

**Interference with Compiler Optimizations**
<br>
The compiler analyzes execution flow to optimize code efficiently. `goto` statements complicate this flow analysis, **hindering the compiler’s ability to generate efficient code**. Since `constexpr` functions perform **compile-time calculations**, their execution flow must be **clear and predictable**.

Due to these reasons, `goto` statements are not allowed in `constexpr` functions. They introduce **unpredictability**, **reduce maintainability**, and **interfere with compile-time evaluation and optimizations**, all of which go against the purpose of `constexpr` functions in C++.

<br>

<h4>6-2. statements with labels other than case and default (since C++14, until C++23)</h4>

The use of **`goto` statements** and related **labels** inside `constexpr` functions is prohibited. This rule applies from **C++14 to C++23** and is intended to maintain the **purity and predictability** of `constexpr` functions.

This restriction **prohibits all labels** except for `case` and `default` labels used in `switch` statements. Since these labels are typically used in combination with `goto` statements, this rule effectively falls under the same category as restricting the use of `goto` itself.

This rule belongs to the same category as the **"goto statements (since C++14, until C++23)"** restriction. Both rules **prohibit the use of `goto` statements and their supporting labels**, ensuring that `constexpr` functions are evaluated **accurately and consistently** at compile time. These restrictions help maintain **logical and consistent execution flow**, allowing the compiler to effectively evaluate functions during compilation.



<h4> try blocks (until C++20) </h4>

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
**divide Function and Compile-Time Exception Handling in constexpr.**
The divide function is a constexpr function that divides two integers.
Within a try block, it performs the division operation and throws a std::invalid_argument exception if the denominator is 0.
The catch block catches the exception, prints an error message, and returns a default value of 0.

In the main function, the divide function is called with a constexpr variable, ensuring that the computation occurs at compile time.
If the denominator is 0, an exception is thrown and handled within the catch block, causing the error message to be displayed at compile time, and result2 is initialized to 0.

**⚠️ Important Considerations**
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

constexpr int add_asm(int a, int b) {
	int result = 0;
	__asm {				//inline assembly is not available 
		mov eax, a
		add eax, b
		mov result, eax
	}
	return result;
}

int main() {
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


<h1> const vs constexpr </h1>

`const` and `constexpr` are both important keywords in C++, but they differ in **usage and scope**.

The primary difference between them is that **`const`** indicates that a variable **cannot be modified**, whereas **`constexpr`** signifies that a variable or function **can be evaluated at compile time**.

Therefore, the main reason for using `constexpr` is **performance optimization**.

<h3> const </h3>

`const` informs the compiler that a variable **cannot be modified** after it is declared. It is primarily used to **clarify the intent of the program** and **prevent unintended modifications**.

For example, `const` can be used to ensure that a function parameter **cannot be altered within the function**, providing additional safety.

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

<h3> constexpr </h3>

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


<h1> Advantages and Disadvantages of Using constexpr </h1>

<h3> Advantages </h3>

- **Compile-Time Optimization**:
    
    **`constexpr`** ensures that a variable or function is evaluated at **compile time**, reducing the need for runtime calculations. This helps improve **program startup time** and overall **execution performance**.
    
- **Memory Usage Optimization**:
    
    **`constexpr`** variables can be replaced with **literal values** where necessary, potentially reducing **executable file size**. Since their values are determined at compile time, **no additional memory allocation is required at runtime**.
    
- **Improved Security and Stability**:
    
    Since **`constexpr`** functions and variables are evaluated at **compile time**, **unexpected runtime behavior** is minimized. This increases **code stability and predictability**.
    

Therefore, when considering **performance optimization and stability**, it is recommended to use **`constexpr`** whenever possible. By leveraging **`constexpr`**, you can **reduce execution time, optimize memory usage, and handle more logic at compile time**, ultimately improving the **overall efficiency** of the program.


<h3> Disadvantages </h3>

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

The `Product_cxx14` function in the code is a `constexpr` function, so it **cannot be debugged**.

Even if you set a **breakpoint** inside the function, it will **not be triggered**.


<br><br>

<h1> Conclusion </h1>

Today, we took an in-depth look at **`constexpr`**. While `const` and `constexpr` may appear similar in syntax, they serve **different purposes and roles** in C++.

- `const` is used to declare **immutable variables** and can be applied at **runtime**.
- On the other hand, **`constexpr`** ensures that **an expression can be evaluated at compile time**, focusing on **reducing execution time and maximizing optimization**.

Since these two keywords have **distinct usage purposes and scopes**, it is crucial to choose the appropriate one **based on code performance and maintainability**.

C++ continues to evolve, and **since C++20, `constexpr` has been further expanded, increasing its utility**. Therefore, ongoing research and learning are essential to effectively leverage `constexpr` in the latest C++ standards.

I hope this article has helped you understand the **concept and practical applications of `constexpr`**, and I look forward to sharing more in-depth content in the future. 

<h2> References </h2>

[constexpr specifier (since C++11) - cppreference.com](https://en.cppreference.com/w/cpp/language/constexpr)
