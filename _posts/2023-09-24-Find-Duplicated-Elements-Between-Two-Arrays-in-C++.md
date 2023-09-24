---
title: Find Duplicated Elements Between Two Arrays in C++
author:
  name: Dennis
categories: [C++, Algorithm]
tags: [C++, Algorithm]
pin: true
use_math: true
---

Someday I was wondering how to get elements which is duplicated between two arrays. So, Today I am gonna talk to you about this. Let’s see code below
<br>


<h2>Approach 1: Brute Force</h2>

```cpp
//Approach 1: Brute Force
#include <iostream>
#include <array>
#include <vector>

int main()
{
	std::array<int, 5> arr1 = {10, 20, 30, 40, 50};
	std::array<int, 5> arr2 = {100, 200, 30, 400, 50};

	for (auto e1 : arr1)
	{
		for (auto e2 : arr2)
		{
			if (e1 == e2)
			{
				std::cout << e1 << " ";
			}
		}
	}

	return 0;
}
```
Elentment 30, 50 are duplicated in the code above. So  you can see that 30 and 50 are output.

![output]({{site.url}}/assets/img/2023-09-24-Find-Duplicated-Elements-Between-Two-Arrays-in-C++/output.png)


The first approach is brute force. the elements of two arrays is compared using nested loop. this solution is simple and easy to complement but inefficient because the loop is repeated until n *m times (n is arr1 size and m is arr2 size). so time complexity is O(n^2). O(n^2) is very inefficient time complexity. I will optimize above code. second approach is using unordered_set container
<br><br>


<h2>Approach 2: To Use Extra Space</h2>

```cpp
//Approach 2: To Use Extra Space
#include <iostream>
#include <array>
#include <vector>
#include <unordered_set>

int main()
{
	std::array<int, 5> arr1 = { 10, 20, 30, 40, 50 };
	std::array<int, 5> arr2 = { 100, 200, 30, 400, 50 };

	std::unordered_set<int> set;

	for (int i = 0; i < arr1.size(); ++i)
	{
		set.insert(arr1[i]);
	}

	for (int i = 0; i < arr2.size(); ++i)
	{
		if (set.count(arr2[i]))
		{
			std::cout << arr2[i] << " ";
		}
	}

	return 0;
}

```

The second approach is to use the std::unordered_set container. This container is used to check for duplicate elements. First, we add all the elements of arr1 to the container. Then, we traverse the array arr2 to find elements that are duplicates of elements in arr1.

Second approach time complexity is O(n + m) and space complexity is O(n)

1. Time complexity of adding all elements of arr1 to container is O(n) (n represents size of arr1)
2. Time complexity of finding element duplicated to elements of arr2 is O(m) (m represents size of arr2)

Second approach time complexity is more efficient than first approach.

We get to know that we could find elements duplicated efficiently when we use additional memory. But it is not appropriate in the environment can not use additional memory. Is there more efficient way to find elements without additional memory?
<br><br>


<h2>Approach 3: Sort Array</h2>
Yes it is. If we sort given arrays,  we can solve this problem without additional space.

```cpp
//Approach 3: Sort Array
#include <iostream>
#include <array>
#include <vector>
#include <unordered_set>
#include <algorithm>

int main()
{
	std::array<int, 5> arr1 = { 10, 20, 30, 40, 50 };
	std::array<int, 5> arr2 = { 100, 200, 30, 400, 50 };

	std::sort(arr1.begin(), arr1.end());
	std::sort(arr2.begin(), arr2.end());

	int i = 0, j = 0;

	std::vector<int> duplicated;

	while (i < arr1.size() && j < arr2.size())
	{
		if (arr1[i] == arr2[j])
		{
			std::cout << arr1[i] << " ";
			i++;
			j++;
		}
		else if (arr1[i] < arr2[j])
		{
			i++;
		}
		else
		{
			j++;
		}
	}

	return 0;
}

```

First, sort given two arrays. then compare elements starting from smallest element. If two elements are different, Increase the index of smaller.<br>
Third approach time complexity is O(n log n + m log m) and There is no space complexity.

1. The time complexity of std::sort() function is O(n log n).
2. We used sort function 2 times to arr1 and arr2.

This solution do not use extra space but it is not appropriate in case that the array to need to sort.
<br><br>

<h2> Conclusion </h2>
The Second approach affects memory usage because it uses additional memory to create unordered_set. On the other hand, The Third approach uses less memory because it uses sorting, but the two arrays need to be sorted, which can lead to greater time complexity if the array is larger.
Therefore, which method is more efficient depends on the situation and must be chosen considering the size of the input data and memory limitations.

