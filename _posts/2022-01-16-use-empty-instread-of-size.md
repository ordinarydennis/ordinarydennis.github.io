---
title: When Checking whether Container is Empty, Use empty() instead of Comparing size() with zero 
author:
  name: Dennis
categories: [C++]
tags: [C++, STL]
pin: true
---

## Introduction
Sometimes, programmers would wonder whether use empty() or size() when checking container is empty.  
Today, I am going to talk about why we should use empty() instead of size().
<br><br>

## 1. Easy to Understand and Type.
```cpp
  std::vector<int> v = {1,2,3};

  if(0 < v.size())
  {
    //someting..
  }
```
```cpp
  std::vector<int> v = {1,2,3};

  if(!v.empty())
  {
    //someting..
  }
```
There are two kind of code. Which code is more easy to understand and type?
empty() function does not use any comparison operators. It is possible to prevent bugs due to incorrect use of comparison operators.(such as If had to use '>' but used '<')
Also the code using empty() is easy to understand than size() because it is simpler.
Finally, the biggest advantage of using empty() is that it is easy to type.
<br><br>

## 2. empty() is More Faster than size() 
empty() is implemented as an inline function. also empty() is a constant-time operation for all standard cotainers.
whereas some implementations of size() is O(n) time complexity such as list::size().
The reason that list::size() has linear time is bacause list container offer splice().
in order that size() has constent time complexity, size must be updated whenever every member function operate. 
It is the same when calling splice(). for updating size, need to count the number of elements to be spliced.
So if list::size() must have constent time complexity, splice() time complexity become linear. 
However, if list::size() doesn't need to be constent time(doesn't need to update size), splice() can be a constant time.
<br><br>

## Conclusion
The reason that use empty() instead of size() is simple. Because empty() is more easier and faster than size(). So call empty whenever you need to know whether a container has zero elements.
<br><br>

Referred to [Effective STL](https://www.amazon.com/Effective-STL-Addison-Wesley-Professional-Computing-ebook/dp/B004V4432W), [vector::empty() and vector::size() in C++ STL](https://www.geeksforgeeks.org/vectorempty-vectorsize-c-stl/)
