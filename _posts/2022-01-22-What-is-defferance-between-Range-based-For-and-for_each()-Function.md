---
title: What is defferance between Range-based For and std::for_each() Function
author:
  name: Dennis
categories: [C++]
tags: [C++, STL]
pin: true
---

## Introduction
When the C++ has introduced at first, many features were added. 
remarkable one thing among the features was two kind of for loop.
One thing is Range-based for loop, another is for_each() functions.
They are new techniques for iterating through the elements of a sequence container like vector, array, and string.
Nowadays, many programmers have used above two features to iterate containers. 
Today, I'm going to talk about the proper use of the two loop statement above.
<br><br>

## 1. Range-based For Loop is familiar.
Existing for and while loops need to define variables to repeat logic like i, j, k.

```C++
int main()
{
  for(int i = 0; i < 10; i++)
  {
    //someting...
  }

  return 0;
}
```
But Range-based for loop to iterate sequence container don't need to define any variables.
You just define variable to access to element of container.

```C++
int main() 
{
    std::vector<int> v{ 1,2,3,4,5 };
  
    for (const int& item : v)
    {
        cout << item << " ";
    }

    return 0;
}
```
This can iterates container and access to all elements in container sequentially.
Also, when you want to exit or continue inside loop, you can use continue, break statements like a normal loop statements(for, while).
How to use Range-based for loop is not strange compare to normal loop. so more convenient to write.
<br><br>

## 2. std::for_each() can generalize.
std::for_each() is template function which take two parameters(begin() and end()) and 
one funtion object which will use elements of the sequence container.
The prototype of the function looks like

```C++
template <class InputItr, class Functor>
Functor for_each(InputItr begin, InputItr end, Functor fnobj);
```
Where InputIter is a type which meets the requirement of an InputIterator concept and Functor is a type of callable object like std::function, lamda expression or a functor. Functor should be moveconstructible and match the signature of the following functions:

```C++
void fun(const Type &a); 
or
void fun(Type a); 
```
Where the type Type should be such that an object of InputItr can be dereferenced and implicitly converted to Type.

Let's look at the follow example code:

```C++
#include <iostream>
#include <vector>
#include <algorithm>

int main()
{
	std::vector<int> vec = {1,2,3,4,5};
	int sum = 0;
	
	std::for_each(vec.begin(), vec.end(),
		[&sum](int e) {

			sum += e;

		}
	);

	std::cout << sum << std::endl;
	return 0;
}
```
This code is simple example code calculating sum of all elements in container with std::for_each().
As you can see, std::for_each() tasks two iterators and one function object(here lamda).
You already take a hint, std::for_each can generalize to any situation.
If you want to sum some set of containers, All you need to change start and last. 
If you want to multiply all elements in container, All you need to change lamda logic. 
You can reuse source code as much as you want.
<br><br>

## Use It Appropriately Depending on Where You Use It.
If you want to access all elements of your container simply, It is better to use Range-based for loop than std::for_each().
Also Because Range-based for loop has similiar usage to normal for loop, Source code is more clearer than when you use std::for_each().
But It can't generalize. If you want to access some set of container? It can't be.
If you want to access of container in reverse order? It can't be.
<br><br>

## Conclusion
They all work like a normal loop internally, hence performance is same.
It doesn't matter which for loop you use. Important thing is to use It appropriately depending on where you use ut.
<br><br>

Referred to [C++ 11: Range based for loop and std::for_each() function](https://www.go4expert.com/articles/cpp-11-range-based-loop-stdforeach-t34604/),
[Why You Should Use std::for_each over Range-based For Loops](https://www.fluentcpp.com/2019/02/07/why-you-should-use-stdfor_each-over-range-based-for-loops/)