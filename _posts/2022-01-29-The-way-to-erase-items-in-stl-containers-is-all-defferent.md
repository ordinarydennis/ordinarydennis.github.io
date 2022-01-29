---
title: The way to erase items in STL containers is all different.
author:
  name: Dennis
categories: [C++]
tags: [C++, STL]
pin: true
---

## Introduction
Becuase the way to erase items in STL containers is all different, Often, it can cause mistakes. Today, Let's find out how to erase elements for each container and situation.
Before we start, it is necessary to divide containers into three categories.

|contiguous memory container|std::list|associative container|
|---|---|---|
|vector, string, deque | std::list | map, set, multimap..

Each container has a different way to erase the elements.
<br><br>

## 1. When you use contiguous memory container, use erase-remove idiom.
Suppose we have a container c with an int type element.

```cpp
std::vector<int> c = {1,2,3,111,4,5,111,6};
```
If you want to erase value which is 111, You can use the erase-remove idiom like this:

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main()
{
	std::vector<int> c = { 1, 2, 3, 111, 4, 5, 111, 6 };

	c.erase(remove(c.begin(), c.end(), 111), c.end());

	for_each(c.begin(), c.end(), [](int e) { std::cout << e << " "; }); 
	
	std::cout << std::endl;

	return 0;
}
```

Because string is vector which has char type, The same goes for string. 

```cpp
#include <string>
int main()
{
	std::string c = { 'a','b','c','d','e','f','c','g' };

	c.erase(remove(c.begin(), c.end(), 'c'), c.end());

	for_each(c.begin(), c.end(), [](char e) { std::cout << e << " "; });

	std::cout << std::endl;

	return 0;
}
```
<br><br>

## 2. When you use std::list, use remove member function.
In fact, std::list can use erase-remove idiom, too.
But std::list has a more efficient way.

```cpp
#include <list>
int main()
{
	std::list<int> c = { 1, 2, 3, 111, 4, 5, 111, 6 };

	c.remove(111);

	for_each(c.begin(), c.end(), [](int e) { std::cout << e << " "; });

	std::cout << std::endl;

	return 0;
}
```
<br><br>

## 3. When you use assosiative container, use erase member function.
```cpp
#include <set>
int main()
{
	std::multiset<int> c = { 1, 2, 3, 111, 4, 5, 111, 6 };

	c.erase(111);

	for_each(c.begin(), c.end(), [](int e) { std::cout << e << " "; });

	std::cout << std::endl;

	return 0;
}
```
The associated containers do not have any function named remove. But you can use erase member function instead. If you use remove algorithm to erase element, It will corrupt the container.
Erase in associative container is a more efficient than remove in sequence container.
Erase take logarithmic time and remove task linear time.
<br><br>

## 4. You must be careful, When you erase element satisfying predicate in associative container.
Erasing elements satisfying specific predicate in sequence container(vector, string, list) is simple. All you need to do is replace remove with remove_if.
```cpp
bool isOdd(int num)
{
	return num % 2 != 0;
}

int main()
{
	std::vector<int> c = { 1,2,3,111,4,5,111,6 };

	c.erase(remove_if(c.begin(), c.end(), isOdd), c.end());

	for_each(c.begin(), c.end(), [](int e) { std::cout << e << " "; }); 
	
	std::cout << std::endl;

	return 0;
}
```

But It is not simple in associative container.
There are two way that you can use. one is easy to code but inefficient.
another is efficient.
The way which is easy to code is using remove_copy_if().
It is add element not satisfying condition to new container then swap original container with new container.

```cpp
bool isOdd(int num)
{
	return num % 2 != 0;
}

int main()
{
	std::multiset<int> oc = { 1, 2, 3, 111, 4, 5, 111, 6 };

	std::multiset<int> nc;

	remove_copy_if(oc.begin(), oc.end(),
		inserter(nc, nc.end()),
		isOdd
	);
	
	oc.swap(nc);

	for_each(oc.begin(), oc.end(), [](int e) { std::cout << e << " "; });

	std::cout << std::endl;

	return 0;
}
```
The drawback to this approach is that it involves copying all the elements that aren't being removed.


<br><br>

## Conclusion

<br><br>

Referred to [Effective STL](https://www.amazon.com/Effective-STL-Addison-Wesley-Professional-Computing-ebook/dp/B004V4432W)