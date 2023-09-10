---
title: Be careful Try Not to Use Invalidated Iterator
author:
  name: Dennis
categories: [C++]
tags: [C++, STL]
pin: true
use_math: true
---

Recently, there was a case of a bug while running a program using invalidated iterator in the team I work for.
Today, we will find out what the iterator invalidation is and summarize the cases of iterator invalidation.
<br><br>

## What is Iterator Invalidation?
The Iterator Invalidation is that  container to which iterator point internally change due to an add/delete operation. i.e. when elements are moved from one position to another, and the initial iterator still points to the old invalid location, then it is called Iterator invalidation.<br><br>


## In case of Change Container Size while Traversing the Container. (Reallocation of Container Memory)
```cpp
vector<int> v = { 1, 5, 10, 15, 20 };

for (auto it = v.begin(); it != v.end(); it++) 
{ 
    cout << *it <<", "; 
    if ((*it) == 5) 
    { 
        v.push_back(100); 
    } 
}
```
Will the code above work well? if you look at the code simply, it seems to be no problem.
however the iterator is invalidated in above code while traversing the container. Above code causes indeterminate behavior due to invalidating iterator while traversing the container.
![vector_iterator_invalidation]({{site.url}}\assets\img\2023-09-10-Be-careful-Try-Not-to-Use-Invalidated-Iterator/vector_iterator_invalidation.png)
When an element is 5, 100 is added to the vector. At this time, memory reallocation occurs while exceeding the vector's maximum capacity. Allocate memory to a location that is completely different from the original location and move all existing elements to a new space. but the iterator still point to old memory location.<br>
As a result, the iterator performs numerous iterations to reach at the v.end(). If you add or delete elements while traversing the container, you may use an invalidated iterator, you must be careful this.<br><br>
As we saw in the above case. If you add elements to a vector when there is no free space, the memory of vector is reallocated and iterator is invalidated. What if the vector has enough capacity?<br>
If you add elements in a vector when there is free space, memory reallocation and iterator invalidation do not occur. 
```cpp
vector<int> v;
v.reserve(10);
v.push_back(1);
v.push_back(5);
v.push_back(10);
v.push_back(15);
v.push_back(20);

cout << v.size() << endl; // size is 5 
cout << v.capacity() << endl; // capacity is 10 

for (auto it = v.begin(); it != v.end(); it++)
{
	cout << *it << ", ";
	if ((*it) == 5)
	{
		v.push_back(100);
	}
}
```
![vector_iterator_validation]({{site.url}}\assets\img\2023-09-10-Be-careful-Try-Not-to-Use-Invalidated-Iterator/vector_iterator_validation.png){: width="400" height="400"){: .left}<br><br><br><br>
<br><br>

##  In Case of Add Element at The Middle of Vector When Vector Has Enough Space.
What if  add element at the middle of the vector when the vector has enough space?
```cpp
std::vector<int> vec;
vec.reserve(10);
vec.push_back(10);
vec.push_back(20);
vec.push_back(30);

auto it = vec.begin();
for (; it != vec.end(); ++it)
{
	std::cout << *it << ", ";
	if (*it == 20)
	{
		vec.insert(it, 100);
	}
}
```
![vector_add_at_the_middle]({{site.url}}\assets\img\2023-09-10-Be-careful-Try-Not-to-Use-Invalidated-Iterator/vector_add_at_the_middle.png){: width="700" height="600"){: .left}<br><br><br><br>