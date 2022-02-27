---
title: What Is Time Complextiy.
author:
  name: Dennis
categories: [Algorithm]
tags: [Algorithm]
use_math: true
pin: true
---

## Abstract
These days, I am solving the LeetCode problems one by one a day.
If I solve the problem, it will tell me the CPU usage rate and memory usage rate of my algorithm. One day, I wondered if my algorithm is efficient.
One of the ways to evaluate the efficiency of an algorithm is to calculate the time complexity.
In this post, we will learn about time complexity.
<br><br>

## What is Time Complexity
Time complexity means the time an algorithm takes to solve a problem.
It can be said that the shorter the time to solve the problem, the more efficient the algorithm is. So, How can we calculate time complexity?
Should we calculate the total time it takes for the algorithm to solve the problem? 
That could be a way too. However, the method of calculating the total execution time of an algorithm may have different results depending on the execution environment in which the algorithm is run. Therefore, we must use a method that will produce the same results in any environment.
<br><br>

## Asymtotic Notation
When analyzing the performance of an algorithm to solve a problem, it is difficult to obtain a constant result due to various factors such as the type of data given, the environment in which the experiment is performed, and the performance of the system used for the experiment, and the comparison result may not always be constant. An effective solution of this is asymptotic analysis. It is a method of counting the number of executions of an operation Instead of measuring the execution time. There are three expression methods for asymptotic analysis. <br>

Omega Notation, Ω <br>
The notation Ω(n) is the formal way to express the lower bound of an algorithm's running time. It measures the best case time complexity or the best amount of time an algorithm can possibly take to complete. <br>

Big Oh Notation, Ο <br>
The notation Ο(n) is the formal way to express the upper bound of an algorithm's running time. It measures the worst case time complexity or the longest amount of time an algorithm can possibly take to complete. <br>

Theta Notation, θ <br>
The notation θ(n) is the formal way to express both the lower bound and the upper bound of an algorithm's running time. It is represented as follows −

When expressing time complexity, Big O is mainly used. Why is that?
Algorithm execution time may depend on the given data.
<br><br>

Insertion Sort Algorithm <br>
The Worst Case : If it is sorted in reverse order. <br>
The Best Case : If the data is sorted and sorting is not required any more. <br>

The algorithm performance is always better than worst case, whatever  the data is 
given. In order to always express constant performance regardless of data conditions,
It is most common to judge by the number of executions for the worst case.
Time complexity is calculated by the total number of basic operations performed to determine the exact efficiency of the algorithm. Basic operations mean data input/output(=), arithmetic(+,-,/ ,*), comparison(<,>, <=, >=, ==) and control statement(if, switch).
Time complexity can be expressed as the sum of operations. If the number of operations is expressed as a polynomial, the time complexity is expressed only with the unknown of the highest order term and exponential of the highest order term.
<br><br>


## The Reason Why Only The Unknown and Exponential of The highest Order Term Are Leaved.
Asymptotic notaion considers only the highest order terms excluding the insignificant terms to focus on the significant growth rate at runtime.


<br><br>


## Calculate Time Complexity of Insertion Sort.

```cpp
void insertionSort(std::vector<int>& list)
{
    int n = list.size();
    //outer loop count is n-1 including i++ of last loop
    for (int i = 1; i < n; i++)
    {
        int curNum = list[i]; // (1) n - 1

        int a = i - 1;
        for (; 0 <= a; a--)
        {
            if (list[a] > curNum) // (2) Up to i times
            {
                list[a + 1] = list[a]; // (3) Up to i times
            }
            else
            {
                //If list is already sorted, call break
                break;
            }
        }

        list[a + 1] = curNum;  // (4)  n - 1
    }
}
```
The sum up to i (2)(3) can be expressed as the sum of arithmetic sequences.<br>

$$\sum_{i=1}^{n-1}{i} = \frac{(n-1)(1+n-1)}{2} = \frac{n(n-1)}{2}$$

So, When the data is sorted in reverse order(the worst case), the time complexity of insertion sort is as follows.<br>

$$2\times(n-1) + 2\times \frac{n(n-1)}{2} = n^2 + n - 2 = O(n^2)$$

When all data is already sorted(the best case), In each loop, only list[a] > curNum (2) is checked and the loop is exited. This means that list[a] > curNumber is executed n - 1 times.
So, The time complexity in the best case is as follows.<br>

$$3\times(n-1) = 3n -3 = O(n)$$

<br><br>


## Conclusion


<br><br>

## References
[Time Complexity, Space Complexity](https://yoongrammer.tistory.com/79)
[Asymtotic notation](https://www.tutorialspoint.com/data_structures_algorithms/asymptotic_analysis.htm)
[Effective STL](https://www.amazon.com/Effective-STL-Addison-Wesley-Professional-Computing-ebook/dp/B004V4432W)