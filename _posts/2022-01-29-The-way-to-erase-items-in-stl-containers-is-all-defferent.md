---
title: The Way to Erase Items in STL Containers is All Different.
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
There are two way that you can use. One is easy to code but inefficient.
Another is efficient.
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
You can avoid this cost by removing the elements from original container directly.
However, Because associative container is not support remove function like remove_if(), You have to write loop to iterate over the elements of the container.
The code is simple like this:
```cpp
bool isOdd(int num)
{
	return num % 2 != 0;
}

int main()
{
	std::multiset<int> oc = { 1, 2, 3, 111, 4, 5, 111, 6 };

	for (auto i = oc.begin(); i != oc.end(); ++i)
	{
		if (isOdd(*i))
		{
			oc.erase(i);
		}
	}

	for_each(oc.begin(), oc.end(), [](int e) { std::cout << e << " "; });

	std::cout << std::endl;

	return 0;
}
```
Will the code work well? No,  The 'i' has been invalidated after the erase function returns.
so, When the invalidated 'i' is increased(++i) in the loop, a runtime error occurs.
To avoid this problem, You make 'i' point to next element before calling erase function.
The easiest way to do that is to use postfix increment on i.
```cpp
for (auto i = oc.begin(); i != oc.end();)
{
	if (isOdd(*i))
	{
		oc.erase(i++);
	}
	else
	{
		++i;
	}
}
```
At this time, the increment statement of the loop must be deleted. <br>
Memory contiguous containers also have the above problem.
Suppose we erase element satisfying specific condition while looping the container.
```cpp
int main()
{
	std::vector<int> oc = { 1, 2, 3, 111, 4, 5, 111, 6 };

	for (auto i = oc.begin(); i != oc.end(); ++i)
	{
		if (isOdd(*i))
		{
			oc.erase(i);
		}
	}

	for_each(oc.begin(), oc.end(), [](int e) { std::cout << e << " "; });

	std::cout << std::endl;

	return 0;
}
```
This is the code to pass an iterator to the erase function if the elements of the vector are odd.
Will the above code work well? No, The above code ocurrs a runtime error.
In memory contiguouse container, Invoking erase not only invalidates all iterators pointing to the erased element, it also invalidates all iterators beyond the erased element. It doesn't matter if we write i++, ++i.

To solve this problem in memory contiguous container, need to use return value of erase.
```cpp
for (auto i = oc.begin(); i != oc.end();)
{
	if (isOdd(*i))
	{
		i = oc.erase(i);
	}
	else
	{
		++i;
	}
}
```
If the element satisfies the condition, passes the iterator to erase function and overwrites it with return value of erase function. Otherwise junst increases the iterator.
<br><br>

## Conclusion
To summarize<br>
1. If you want to erase specific value in a container :<br>
	- If the container is memory contiguous container, Use erase-remove idiom.
	- If the container is list, Use remove member function.
	- If the container is associative container, Use erase member function.

2. If you want to erase all elements satisfying specific predicate :<br>
	- If the container is memory contiguous container, Use erase-remove_if idiom.
	- If the container is list, Use remove_if member function.
	- If the container is associative container, Use the way to swap after using remove_copy_if() or write loop to iterate over the container.

<br><br>

Referred to [Effective STL](https://www.amazon.com/Effective-STL-Addison-Wesley-Professional-Computing-ebook/dp/B004V4432W)