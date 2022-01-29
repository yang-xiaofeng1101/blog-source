---
uuid: c976d01a-99a6-f131-cf6c-e66c97779db2
title: Functors
tags:
  - C++
  - thrust
  - cuda
categories:
  - 编程
toc: false
date: 2020-03-12 00:40:09
---

# 1. C++ Functors
### Alright, what is a functor?
A C++ functor is a "function-object". In other words, it is an object that can be called and treated just like a regular function. Let's see an example.
```c++
class Doubler
{
public:
    void operator()(int &val) const
    {
        val *= 2;
    }
};

Doubler d1, d2; // two instances of the Doubler class
int i = 20, j = 30;

d1(i); // both instances can be called as
d2(j); //   if they were regular functions
// now i=40 and j=60
```
### OK, so what do functors give us? **LOTS**!

- We can *maintain per-instance state*.
- We can *pass functors around just like variables*.
- Compiler optimizations (vs. function pointers)
```c++
class AddX
{
public:
    AddX(int x) : _x(x) {}
    int operator()(const int &val) const
    {
        return val + _x;
    }

private:
    const int _x;
};
AddX add5(5), add10(10);            // two instances
std::cout << add5(3) << std::endl;  // prints 8
std::cout << add10(3) << std::endl; // prints 13
```
### We can also pass functor instances into the STL algorithms.
```c++
void doubleIt(int &val)
{
    val *= 2;
}

class Doubler
{
public:
    void operator()(int &val) const
    {
        val *= 2;
    }
};

std::vector<int> myset(10);
Doubler myDoubler;

// the next two lines are functionally equivalent
std::for_each(myset.begin(), myset.end(), doubleIt);
std::for_each(myset.begin(), myset.end(), myDoubler);
```
# 2. Thrust - Functors
OK, so Thrust provides some vectors and generic algorithms for me. But what if I want to do more than just sort, count, and sum my data?

Well, look at the title of the slide! - You can make your own functors and pass these into Thrust's generic algorithms.
```c++
// the next two lines are functionally equivalent
std::for_each(myset.begin(), myset.end(), doubleIt);
std::for_each(myset.begin(), myset.end(), myDoubler);

// calculate result[] = (a * x[]) + y[]
struct saxpy
{
    const float _a;
    saxpy(int a) : _a(a) {}

    __host__ __device__ float operator()(const float &x, const float &y) const
    {
        return a * x + y;
    }
};

thrust::device_vector<float> x, y, result;
// ... fill up x & y vectors ...
thrust::transform(x.begin(), x.end(), y.begin(),
                  result.begin(), saxpy(a));
```

```c++
struct saxpy
{
    const float _a;
    saxpy(int a) : _a(a) {}

    __host__ __device__ float operator()(const float &x, const float &y) const
    {
        return a * x + y;
    }
};
```
Let's look more at that. The operator() function is prefixed with __host__ __device__. This is CUDA compiler notation, but to Thrust it means that it can be called with a host_vector OR device_vector.

Also notice that in this form of programming you don't need to worry about threadIdx and blockIdx index calculations in the kernel code. You can just focus on what needs to happen to each element.

There are many excellent Thrust examples (including this saxpy one) in the installed distribution and online.