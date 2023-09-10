---
title: The Way to Erase Items in STL Containers Is All Different
author:
  name: Dennis
categories: [C++]
tags: [C++, STL]
pin: true
---

<h2>Introduction</h2>
Becuase the way to erase items in STL containers is all different, Often, it can cause mistakes. Today, Let's find out how to erase elements for each container and situation.
Before we start, It is necessary to divide containers into main three categories.

|contiguous memory container|std::list|associative container|
|---|---|---|
|vector, string, deque | std::list | map, set, multimap..

Each container has a different way to erase the elements.
<br><br>

<h2>1. When you use contiguous memory container, Use erase-remove idiom.</h2>
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
std::string c = { 'a','b','c','d','e','f','c','g' };
c.erase(remove(c.begin(), c.end(), 'c'), c.end());
```
<br><br>

<h2>2. When you use std::list, Use remove member function.</h2>
In fact, you can use the erase-remove idiom on std::list as well.
However, std::list has a more efficient way.

```cpp
std::list<int> c = { 1, 2, 3, 111, 4, 5, 111, 6 };
c.remove(111);
```
<br><br>

<h2>3. When you use assosiative container, Use erase member function.</h2>

```cpp
std::multiset<int> c = { 1, 2, 3, 111, 4, 5, 111, 6 };
c.erase(111);
```

The associative containers don't have any function named remove. However, you can use erase member function instead. If you use remove algorithm to erase element, it will corrupt the container.
Erase of associative container is more efficient than remove of the sequence container.
Erase take logarithmic time and remove task linear time.
<br><br>

<h2>4. You must be careful, When you erase element satisfying predicate in associative container.</h2>
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

	return 0;
}
```

But it is not simple on associative container.
There are two ways you can use. One is easy to code but it not efficient.
another is efficient.
After adding elements that do not satisfy the condition to the new container, the original container is exchanged for a new container.

```cpp
std::multiset<int> oc = { 1, 2, 3, 111, 4, 5, 111, 6 };

std::multiset<int> nc;

remove_copy_if(oc.begin(), oc.end(),
	inserter(nc, nc.end()),
	isOdd
);

oc.swap(nc);
```
The drawback to this approach is that it involves copying all the elements that aren't being removed.
You can avoid this cost by removing the elements from original container directly.
However, Because associative container don't support remove function like remove_if(), You need to write loop to iterate over the elements of the container.
The code is simple like this:

```cpp
std::multiset<int> oc = { 1, 2, 3, 111, 4, 5, 111, 6 };

for (auto i = oc.begin(); i != oc.end(); ++i)
{
	if (isOdd(*i))
	{
		oc.erase(i);
	}
}
```

Will the above code work well? No, the 'i' was invalidated after the erase function was returned.
Therefore, when the invalidated 'i' increased (++i) in iteration-expression of loop, a runtime error occurs.
To avoid this problem, you make 'i' point to next element before calling erase function.
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
std::vector<int> oc = { 1, 2, 3, 111, 4, 5, 111, 6 };

for (auto i = oc.begin(); i != oc.end(); ++i)
{
	if (isOdd(*i))
	{
		oc.erase(i);
	}
}
```

This is the code to pass an iterator to the erase function if the elements of the vector are odd.
Will the above code work well? No, The above code ocurrs a runtime error.
In memory contiguouse container, Invoking erase not only invalidates all iterators pointing to the erased element, it also invalidates all iterators beyond the erased element. It doesn't matter if we write i++, ++i.
To solve this problem in memory contiguous container, you need to use return value of erase.
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
If the element satisfies the condition, passes the iterator to erase function and overwrites it with return value of erase function. Otherwise just increases the iterator.
<br><br>

<h2>Conclusion</h2>
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