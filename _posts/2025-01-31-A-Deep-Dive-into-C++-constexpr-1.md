---
title: A Deep Dive into C++ constexpr(1/2)
author:
  name: Dennis
categories: [C++]
tags: [C++]
pin: true
use_math: true
---


As you develop software, you may often come across the `const` and `constexpr` keywords. 
At first glance, both seem to represent "constants," but in reality, their subtle differences can significantly impact code performance and execution.
"Why should I use `constexpr`?", "Isn't `const` sufficient everywhere?" If you've ever had these questions, this article is for you. 
Today, we'll dive deep into the core concepts of `constexpr`, explore its practical applications, and illustrate its power with engaging examples.


<h1> const </h1>

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
Imagine accidentally storing a user-inputted name in the program version variable. 
In this case, the version information could become corrupted. This is a more common mistake than you might think.

The solution is simple. Just change the programVersion variable to a constant.
```cpp
const std::string programVersion = "1.0.0";

...
programVersion = userName; // The error occures bacause programVersion is const variable.
...

```
```
Error C2678 binary '=': no operator found which takes a left-hand operand of type 'const std::string' (or there is no acceptable conversion)
```
Since a constant cannot be modified, an error message like the one above will appear if you attempt to change it.

<br>

<h1>constexpr</h1>

Now that we have briefly covered the `const` keyword, let's dive into the `constexpr` keyword in detail.
The constexpr specifier declares that it is possible to evaluate the value of the entities at compile time. 
Such entities can then be used where only compile time constant expressions are allowed (provided that appropriate function arguments are given).

<h2> constexpr as a constant </h2>

A constexpr specifier used in an object declaration or non-static member function(until C++14) implies const.

```cpp
class Point
{
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

Before C++14, constexpr enforced that objects and non-static member functions had const characteristics. 
This ensured that the function or object remained unchanged throughout the program's execution. 
*After C++14, constexpr functions gained more flexibility, but the fundamental concept related to const remains the same.*


<h2> constexpr as a inline </h2>

A constexpr specifier used in the first declaration of a function or static data member(since C++17) **implies inline**. 
If any declaration of a function or function template has a constexpr specifier, then every declaration must contain that specifier.

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
In the example above, the declaration of static constexpr double PI = 3.14159; in the MathConstants class ensures that PI is a static data member initialized at compile time. *Since constexpr implies inline starting from C++17, PI is accessible across different translation units*. This means that the constant can be used in any file within the program.
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

<br>

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

*`constexpr` variables are useful for storing values that need to be computed at compile time to optimize program performance.* This helps minimize execution time and memory usage by performing complex calculations at compile time, reducing runtime overhead. A variable can be declared as constexpr if it meets the following conditions.

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

**Primitive Types**: Most primitive data types are literal types. For example, int, char, float, double, and bool are all considered literal types. These types can be used as constants at compile time.
    ```cpp
    // constexpr for primitive type
    constexpr int n = 1;
    constexpr char c = 'A';
    constexpr float f = 1.0f;
    constexpr double d = 10.0;
    ```
**Literal Type Structures and Classes:** For a structure or class to be considered a literal type, it must satisfy the following conditions:
  - All data members must be **literal types**.
  - The class must have at least one **`constexpr` constructor**, which must be usable for object initialization.
  - It must have a **non-virtual destructor**.
  - It must not contain a **non-literal base class** or **virtual functions** 
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

**Enumeration Types**: All enumeration types (enum) are also literal types.

```cpp
enum Color { Red, Green, Blue };
constexpr Color favoriteColor = Green;   
```

**Array Types**: An array whose elements are literal types is also considered a literal type.
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


[A Deep Dive into constexpr(2/2)]({{ "/posts/A-Deep-Dive-into-C++-constexpr-2/" | relative_url }})

<br>

<h2> References </h2>

[constexpr specifier (since C++11) - cppreference.com](https://en.cppreference.com/w/cpp/language/constexpr)