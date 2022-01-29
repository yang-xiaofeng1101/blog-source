---
title: a+b位运算
tags:
  - 位运算
  - 算法
categories:
  - 编程
toc: false
date: 2020-06-15 11:21:12
---

## a+b位运算
#### 核心思想
a^b(不进位加法)。
(a&b)<<1(用于表示进位的位置)。
#### 具体步骤
将上面2结果再次进行按位异或^运算，再求新的异或加法的进位表示，直到没有进位产生也就是(a&b)<<1==0时a^b的结果即为a+b的最终结果
#### 源码实现
```c++
class Solution {
public:
    /**
     * @param a: An integer
     * @param b: An integer
     * @return: The sum of a and b 
     */
    int aplusb(int a, int b) {
        
        int c= a^b;
        int d = (a&b)<<1;
        
        while(d!=0){
            a=c;
            b=d;
            c=a^b;
            d=(a&b)<<1;
        }
        return c;
    }
};
```