---
layout: post  
mathjax: true  
comments: true  
title: Recursion  
tags: [Algorithm]  
---  

Small post about recursion basics.  

Recursion is a fundamental concept in computer science and programming. It can be a very elegant way to solve a problem, but understanding it eludes a great many folks. In order to understand recursion, we have to take a step back and understand a different fundamental concept first --> *The Stack*.  

## Call Stack  
The stack is a simple data structure that encompasses two basic functions.  
1. Push -> add a new item to the top  
2. Pop -> remove the top-most item and read it  

Computers implement this data structure in the form of the *Call Stack*. The call stack acts as a sort of record in memory for a function call. When a function contains a nested function, the top-level function is paused in a partially completed state until the inner function returns. Only when all inner functions have been satisfied can the top-level function return. Each inner function gets **stacked** on the base function (hence the name) and **popped** once they return.  

Now that we've defined what a stack is, let's demonstrate both of these concepts with a popular example, [Euclid's Algorithm](https://en.wikipedia.org/wiki/Euclidean_algorithm). Euclid's Algorithm is a well-known way for computing the greatest common divisor of two numbers, and it is an excellent use-case for recursion.  

## Euclid's Algorithm  
Euclid's algorithm is an elegant solution to finding the greates common divisor of two numbers. The algorithm takes the following steps:  
1. Check if the smaller number can be evenly divided into the larger number -> if it can, the GCD is the smaller number.
2. If the smaller number can't be divided into the larger number, take the difference between the two and compare to the smaller number again.  

This turns out to be a very simple, yet powerful approach, and it lends itself nicely to recursion. Recursion can loosely be defined as a function that calls itself. It is made up of two cases:  
* Base Case -> the termination criteria  
* Recursive Case -> Criteria to repeat the function call until the base case is met  

Recursion uses the call stack, and it can become very memory intensive if you're not careful. Each recursive call is added to call stack until the base case is met. If the recursive function requires many repeated calls, the call stack will grow quite large.  

Now that we have a better understanding of recursion and the call stack, let's write out the algorithm.


```python
def euclids_algo(x: int, y: int) -> int:
    # Base Case
    if (x == 0) or (y == 0):
        return 0
    # Base Case Alt.
    elif x == y:
        return x
    # Recursive Case (calls itself)
    else:
        return euclids_algo(abs(x - y), min(x, y))
```

We can now test this recursive function out on some simple examples to see how it works. We'll start with a simple example (the numbers 50 and 15). We should end up with the number 5. I chose these numbers as they're easy to intuit.


```python
euclids_algo(50, 15)
```




    5



That was easy! Let's try it with a more complicated pair of numbers that isn't easy to inuit.


```python
euclids_algo(1147, 899)
```




    31



How do we know if this is correct? Let's walk through the algo by hand:

899 can't be divided into 1147  
-> 1147 - 899 = 248 (new pair is 248 and 899)  
-> 248 can't be divided into 899  
-> 899 - 248 = 651 (new pair is 651 and 248)  
-> 248 can't be divided into 651  
-> 651 - 248 = 403 (new pair is 403, 248)  
-> 248 can't be divided into 403  
-> 403 - 248 = 155 (new pair is 248 and 155)  
-> 155 can't be divided into 248  
-> 248 - 155 = 93 (new pair is 155 and 93)  
-> 93 can't be divided into 155  
-> 155 - 93 = 62 (new pair is 93 and 62)  
-> 62 can't be divided into 93  
-> 93 - 62 = 31 (new pair is 62 and 31)  
-> Base Case Reached! 31 can be evenly divided into 62!  

This flow is very similar to how the call stack will look for the recursive call. Each successive call will add to the top of the stack until the base case is met. Once it's met, the calls will return until the stack reaches the original function call (in this case it was `euclids_algo(1147, 899)`).  

## Summary  
To summarise:  
* Recursion is a when a function calls itself.  
* The stack is a LIFO data structure (Last in First Out).  
* Recursive calls can become memory intensive if not careful.  
* Euclid's algorithm is an elegant solution that can be implemented recursively.  

### Reference
[1] Bhargava, A. Y. (2016). Grokking algorithms: An illustrated guide for programmers and other curious people.
