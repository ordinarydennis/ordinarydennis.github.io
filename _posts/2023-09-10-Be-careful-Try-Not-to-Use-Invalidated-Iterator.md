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

<h2>What is Iterator Invalidation?</h2>
The Iterator Invalidation is that  container to which iterator point internally change due to an add/delete operation. i.e. when elements are moved from one position to another, and the initial iterator still points to the old invalid location, then it is called Iterator invalidation.<br><br>


<h2>In case of Change Container Size while Traversing the Container. (Reallocation of Container Memory)</h2>

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
![vector_iterator_invalidation]({{site.url}}/assets/img/2023-09-10-Be-careful-Try-Not-to-Use-Invalidated-Iterator/vector_iterator_invalidation.png)

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

![vector_iterator_validation]({{site.url}}/assets/img/2023-09-10-Be-careful-Try-Not-to-Use-Invalidated-Iterator/vector_iterator_validation.png){: width="400" height="400"){: .left}<br><br><br><br><br><br>

<h2>In Case of Add Element at The Middle of Vector When Vector Has Enough Space.</h2>

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

The above code is the code that adds 100 to vec when the value pointed to by the iterator is 20. Does this code work well? The above code is repeated infinitely. When an element is added to the vector, the elements on the right side of added position are moved to the right once. As a result, the iterator encounters an element that meets the same condition again in the next loop, and another element is added. when an element is added until there is no space left in the vector and the vector reallocates memory,
An invalidated (pointing to an old memory) iterator repeats the for statement infinitely until it meets the end of the reassigned vector.
![vector_add_at_the_middle]({{site.url}}/assets/img/2023-09-10-Be-careful-Try-Not-to-Use-Invalidated-Iterator/vector_add_at_the_middle.png){: width="700" height="600"){: .left}<br><br><br><br><br><br><br><br>

<h2>In Case of Deleting an Element of The Vector.</h2>

```cpp

std::vector<int> vec = { 10,20,30,40,50 };

auto it = vec.begin() + 2;

vec.erase(it);

std::cout << *it << std::endl;

```

In the code above, the iterator points to the third element(30) of the vector. What happens when you delete an element that the iterator points to? This is because after the element 30 pointed by the iterator was deleted, the other elements after the deleted element were moved once to the left (the movement of the element occurred). It can be said that this iterator has been invalidated because the element pointed to by the iterator has changed after the element was deleted from the vector.
Let's check why the above code is invalidated with the example below.

```cpp

std::vector<int> vec = { 10,20,30,40,50 };

auto it = vec.begin();

for (; it != vec.end(); ++it) 
{ 
	std::cout << *it << std::endl;

	if (*it == 20) 
	{ 
		vec.erase(it); 
	} 
}

```

What will be the output of the above code? In the code above, while touring the container, elements that meet certain conditions were deleted. Is the output 10,20,30,40,50? no, it is 10,20,40,50.
As 20 was deleted, the elements after 20 moved once to the left.
When the vec.erase(it)  is performed, the element to which iterator point change 20 to 30. This iterator is invalidated because the element that the iterator points to has changed.

![deleting_an_element_of_the_vector]({{site.url}}/assets/img/2023-09-10-Be-careful-Try-Not-to-Use-Invalidated-Iterator/deleting_an_element_of_the_vector.png){: width="700" height="600"){: .left}<br><br><br><br>

After that, The iterator points to 40 because the iterator is increased in the increment expression of the loop.

![deleting_an_element_of_the_vector]({{site.url}}/assets/img/2023-09-10-Be-careful-Try-Not-to-Use-Invalidated-Iterator/deleting_an_element_of_the_vector_2.png){: width="700" height="600"){: .left}<br><br><br><br><br><br>


<h2> Conclusion </h2>
Today, We learned about in case that iterator of vetor is invalidated and problem that occurs when using the iterator which is invalidated.
The point we need to be careful is that the iterator which points to element may be invalidated after insert or erase operation in the loop.