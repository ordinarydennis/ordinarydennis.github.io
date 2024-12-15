---
title: C++17 Structured Binding
author:
  name: Dennis
categories: [C++]
tags: [C++]
pin: true
use_math: true
---


Today, I am going to talk about structured binding which was first introduced in C++17.

When you want to return multiple values from a function, how do you usually do it? Do you use std::vector? Let me show you how to create a function that returns multiple values using a vector.

```cpp
#include <vector>

std::vector<int> GetNumbersAandB(int a, int b)
{
	std::vector<int> numbers;
	
	for (int n = a; n <= b; n++)
	{
		numbers.push_back(n);
	}

	return numbers;
}

int main()
{
	std::vector<int> v;

	auto numbers = GetNumbersAandB(1, 10);
}
```
Great! Now we can get all the integers from a to b. However, there’s still a problem: What if we want to return multiple values of different types.

```cpp
#include <string>

struct MyInfo
{
	int			age;
	std::string name;
	float		weight;
};

MyInfo MakeMyInfo(int age, std::string name, float weight)
{
	return MyInfo{ age, name, weight };
}

int main()
{
	auto myInfo = MakeMyInfo(10, "Dennis", 60.19f);
}
```
We retrieved values of multiple types using a structure in the code above. If we need to add more information, we can simply add members to the structure without modifying the function.<br>
However, this method still has a problem. Every time we want to retrieve values of different types, we have to create a new structure or class, which can be quite tedious.
<br><br>

<h2> std::tuple </h2>

std::tuple was first introduced in C++11. std::tuple is a class that bundles together different types.

```cpp
template <class _This, class... _Rest>
class tuple<_This, _Rest...> : private tuple<_Rest...> { // recursive tuple definition
public:
    using _This_type = _This;
    using _Mybase    = tuple<_Rest...>;
```

```cpp
#include <tuple>
#include <string>
#include <iostream>

std::tuple<int, std::string, float> MakeMyInfoByTuple(int age, std::string name, float weight)
{
	return { age, name, weight };
}

int main()
{
	auto t = MakeMyInfoByTuple(10, "Dennis", 60.19f);

	std::cout << std::get<0>(t) << std::endl;
	std::cout << std::get<1>(t) << std::endl;
	std::cout << std::get<2>(t) << std::endl;
}
```
 In the code above, We created the function that returns multiple types without using a structure. After the introduction of std::tuple, We no longer need to use a structure or class in order to return multiple types.

 **The limitations of std::tuple**<br>
 Even std::tuple still has its limitations: it is inconvenient to use in source code. We must use get<>() function to retrieve a value of each element in a std::tuple, which also reduces code readability.  In particular, when dealing with tuples that contain many elements or involve complex types, it becomes difficult to determine which data is stored at which position."

```cpp
std::cout << std::get<0>(t) << std::endl; // It is difficult to intuitively understand what is being retrieved.
std::cout << std::get<1>(t) << std::endl;
std::cout << std::get<2>(t) << std::endl;

```
<br>

<h2>What Is Structured Binding?</h2>
Structured binding, introduced in C++17, is a feature that decomposes data structures like tuples, structs, and arrays, allowing their components to be directly bound to variables.
This improve code readability and arrow for writing more intuitive code. The main purpose is to make the code clean and easy to understand

**Binding process**<br>
A structured binding declaration then performs the binding in one of three possible ways, depending on E:

```
When a structured binding is declared, the compiler internally creates temporary variables to store or reference the original data of the binding. This temporary variable is what we refer to as 'e'.
```

- Case 1: If `E` is an array type, then the names are bound to the array elements.
- Case 2: If `E` is a non-union class type and [std::tuple_size](http://en.cppreference.com/w/cpp/utility/tuple_size)<E> is a complete type with a member named `value` (regardless of the type or accessibility of such member), then the "tuple-like" binding protocol is used.
- Case 3: If `E` is a non-union class type but [std::tuple_size](http://en.cppreference.com/w/cpp/utility/tuple_size)<E> is not a complete type, then the names are bound to the accessible data members of `E`.

[https://en.cppreference.com/w/cpp/language/structured_binding](https://en.cppreference.com/w/cpp/language/structured_binding)

For convenience, let me explain in the order of case 1, case 3, and case 2.

**Case 1** : if E is an array type, then the names are bound to the array elements.

**Binding the elements of an array.**

```cpp
#include <iostream>

int main() {
    
    int arr[3] = {10, 20, 30};

    // bind elements of array using structured biding
    auto [a, b, c] = arr;

    std::cout << "a: " << a << "\n"; // arr[0]
    std::cout << "b: " << b << "\n"; // arr[1]
    std::cout << "c: " << c << "\n"; // arr[2]

    // change value of a
    a = 100; // Even if the value of a is changed, the original array is not affected.    
    std::cout << "Updated a: " << a << "\n";
    std::cout << "Original array arr[0]: " << arr[0] << "\n"; // arr[0] still is 10

    return 0;
}
```

output
```
a: 10
b: 20
c: 30
Updated a: 100
Original array arr[0]: 10
```

```cpp
auto [a, b, c] = arr;
```
In this declaration, structured binding binds the elements of the arr array to a, b, and c (a is bound to `arr[0]`,  b is bound to `arr[1]` and c is bound to `arr[2]`).
The bound variable a, b, and c have copied values, so changing their values does not affect the original arr array.


**The reference binding of the elements of the array**

```cpp
#include <iostream>

int main() {
    int arr[3] = {10, 20, 30};

    // binding elements of the array by reference
    auto& [a, b, c] = arr;

    // modification through reference
    a = 100;

    std::cout << "Updated a: " << a << "\n";
    std::cout << "Original array arr[0]: " << arr[0] << "\n"; // arr[0] changes to 100

    return 0;
}
```
output
```
Updated a: 100
Original array arr[0]: 100
```
In structured binding, whether the elements of the array are copied or referenced depends on the choice of auto or auto&.


**Case 3** : if E is a non-union class type but tuple_size is not a complete type, then the names are bound to the public data members of E.


**Structured Binding for Class Type**
```cpp
#include <iostream>
#include <string>

class MyInfo {
public:
    int			mAge;
    std::string mName;
    float		mWeight;

    MyInfo(int age, std::string name, float weight)
        :mAge(age), mName(name), mWeight(weight)
    {
    }

};

int main() {
    MyInfo myInfo = { 10, "Dennis", 60.19f};

    //Structured Binding for class type 
    auto [age, name, weight] = myInfo;

    std::cout << "age: " << age << std::endl;
    std::cout << "name: " << name << std::endl;
    std::cout << "weight: " << weight << std::endl;

    return 0;
}
```
Structured binding for class types binds to public data members in order. Private or protected members are not bound.

- age → mAge (first member variable)
- name → mName (second member variable)
- weight → mWeight (third member variable)

However, structured binding for class types has limitations. if  private or protected members are added, compile errors occur.

```cpp
#include <iostream>
#include <string>

class MyInfo {
public:
    int			mAge;
    std::string mName;
    float		mWeight;

    MyInfo(int age, std::string name, float weight, std::string country)
        :mAge(age), mName(name), mWeight(weight), mCountry(country)
    {
    }

private:
    std::string mCountry;

};

int main() {
    MyInfo myInfo = { 10, "Dennis", 60.19f, "US"};

    //Structured Binding for class type 
    auto [age, name, weight] = myInfo;

    std::cout << "age: " << age << std::endl;
    std::cout << "name: " << name << std::endl;
    std::cout << "weight: " << weight << std::endl;

    return 0;
}
```

output
```
Error	C3448	the number of identifiers must match the number of array elements or members in a structured binding declaration
```
The private member variable mCountry has been added. A compile error occurs because the number of members of class and the number of structured binding variables are different.

**Case 2** : if E is a non-union class type and tuple_size is a complete type, then the “tuple-like” binding protocol is used. In Case 3 above, structured binding could not be used for the class type. To use structured binding with a class type, you need to understand  tuple_size and tuple-like binding

**tuple_size and tuple-like binding**<br>
What is tuple_size?<br>
`std::tuple_size` is a template class provided by the C++ standard library. It is a meta-information class that provides the number of elements in 'tuple-like types' at compile time.
i.e., `tuple_size` provides the number of elements in 'tuple-like types' at compile time.

**Requirements for the Tuple-Like Binding Protocol**
1. The type `E` must be able to determine the number of elements at compile time through `tuple_size<E>`
2. Each element must be accessible through std::get<N>(E)
   
Structured binding works based on tuple_size and std::get, allowing simple decomposition of std::tuple, std::pair, or user-defined tuple-like types. By defining tuple_size and std::get in a user-defined class, structured binding can also be utilized with custom classes.

```cpp
#include <iostream>
#include <string>
#include <tuple>

class MyInfo
{
public:
    int mAge;
    std::string mName;
    float mWeight;

    MyInfo(int age, std::string name, float weight, std::string country)
        : mAge(age), mName(name), mWeight(weight), mCountry(country) {}

private:
    std::string mCountry;
};

namespace std {
    template <> struct tuple_size<MyInfo> : std::integral_constant<size_t, 3> {};
    template <> struct tuple_element<0, MyInfo> { using type = int; };
    template <> struct tuple_element<1, MyInfo> { using type = std::string; };
    template <> struct tuple_element<2, MyInfo> { using type = float; };

    template <size_t N>
    auto get(const MyInfo& info) {
        if constexpr (N == 0) return info.mAge;
        else if constexpr (N == 1) return info.mName;
        else if constexpr (N == 2) return info.mWeight;
        else static_assert(N < 3, "Index out of bounds");
    }

    template <size_t N>
    auto& get(MyInfo& info) {
        if constexpr (N == 0) return info.mAge;
        else if constexpr (N == 1) return info.mName;
        else if constexpr (N == 2) return info.mWeight;
        else static_assert(N < 3, "Index out of bounds");
    }
}

int main()

{
    MyInfo myInfo(10, "Dennis", 60.19f, "US");

    // 1.copy
    auto [ageCopy, nameCopy, weightCopy] = myInfo;
    ageCopy = 20; // original age is not chagned because it is copy.

    std::cout << "Modified ageCopy: " << ageCopy << std::endl;
    std::cout << "Original myInfo.mAge: " << myInfo.mAge << std::endl;

    // 2. reference
    auto& [ageRef, nameRef, weightRef] = myInfo;
    ageRef = 30; // original variable age is changed because it is reference.

    std::cout << "Modified ageRef: " << ageRef << std::endl;
    std::cout << "Original myInfo.mAge (after modification): " << myInfo.mAge << std::endl;

    return 0;
}
```
<br>

<h2>The Limitations of Structured Binding</h2>
Even structured binding has its limitations.

**1. Cannot use std::ignore**<br>
This means that when you want to ignore a specific member or element in 
structured bindings, you can’t use std::ignore.
For example, When you want to write the following code:
```cpp
#include <array>
#include <iostream>

int main()
{
    std::array<int, 3> arr = {1, 2, 3};

    auto [a, std::ignore, c] = arr;  // ERROR: std::ignore can not be used.

    std::cout << "a: " << a << ", c: " << c << std::endl;

    return 0;
}

```
In the code above, you want to ignore the second element using std::ignore, but std::ignore is not allowed in structured bindings, resulting in a compilation error.

**How to ignore specific members in structured bindings?**<br>
To ignore some elements in structured bindings, you typically assign them to a temporary variable like  `_` .

For example:
```cpp
#include <array>
#include <iostream>

int main()
{
    std::array<int, 3> arr = {1, 2, 3};

    auto [a, _, c] = arr;  // `_` Do not use variable

    std::cout << "a: " << a << ", c: " << c << std::endl;

    return 0;
}
```
This code requires declaring variables for all members in structured bindings. Unused variables can be ignored by assigning them to `_` or another meaningless name.
This method does not fulfill the role of `std::ignore`, but it is used to indicate to the compiler that certain variables will not be used.


**2. Cannot specify qualifiers (e.g., const, reference) individually.**  
Using structured bindings allows you to decompose and bind multiple variables at once, but it is not possible to configure qualifiers individually for each bound variable. In other words, it is not possible to configure the following qualifiers in structured bindings

```cpp
int main()
{
    MyInfo myInfo(10, "Dennis", 60.19f, "US");

    auto [&age,const& name, const weight] = myInfo;

    return 0;
}
```
It is not possible to bind age as a reference, name as a constant reference, and weight as a constant in structured bindings. Such an approach results in a compilation error. In other words, all variables bound in structured bindings must use the same approach, making individual customization impossible.
<br><br><br>

<h2> Conclusion </h2>
Today, we explored structured bindings in C++. Structured bindings, introduced in C++17, are a powerful feature that allows for clearer and more intuitive handling of data structures. By enabling the decomposition of complex data structures and easy access to individual members, this feature significantly improves code readability and maintainability.

However, like any tool, structured bindings have their limitations. Constraints such as the inability to configure individual qualifiers or challenges in handling inheritance relationships are important considerations during development. Understanding these limitations and using structured bindings appropriately can help make your C++ code cleaner and more concise.

Try incorporating structured bindings into your C++ projects. Embracing new features and writing better code is a great way to grow as a developer. 😊
<br><br><br>


<h2> References </h2>

[Structured binding declaration (since C++17) - cppreference.…](https://en.cppreference.com/w/cpp/language/structured_binding)
[Server Developer Owen C++ 구조적 바인딩(Structured Bindings)](https://game-server-developer.tistory.com/4)
[GeeksforGeeks Structured binding in C++ - GeeksforGeeks](https://www.geeksforgeeks.org/structured-binding-c/)
[Seonghyeon Cho C++ 구조적 바인딩(structured bindings)](https://sh-cho.github.io/cpp-structured-binding/)
[nicotina04 C++ Structured Binding(구조적 바인딩)](https://nicotina04.tistory.com/178)
