---
title: topK
tags:
  - 排序
originContent: ''
categories:
  - 编程
toc: false
date: 2020-06-20 21:42:09
---

## 描述

在数组中找到第 k 大的元素。

你可以交换数组中的元素的位置
  
### 样例
样例 1：

输入：
n = 1, nums = [1,3,4,2]
输出：
4
样例 2：

输入：
n = 3, nums = [9,3,2,4,8]
输出：
4
### 挑战
要求时间复杂度为O(n)，空间复杂度为O(1)。
### 源码
```cpp
class Solution {
public:
    /**
     * @param n: An integer
     * @param nums: An array
     * @return: the Kth largest element
     */
     static bool cmp(int a, int b) {
	        return a > b;
    }
void quicksort(vector<int>& a, int left, int right, int n)
{
	int temp = a[left];


	int i, j;
	i = left;
	j = right;

	while (i < j)
	{
		while (i < j && a[j] <= temp)
			j--;
		a[i] = a[j];
		while (i < j && a[i] >= temp)
			i++;
		a[j] = a[i];

		
	}
	a[i] = temp;

	if (i < n)
	{
		quicksort(a, i+1, right, n);
	}
	else if (i > n)
	{
		quicksort(a, left, i - 1, n);
	}
	else
	{

		sort(a.begin()+left, a.begin()+i+1, cmp);
	}
}

int kthLargestElement(int n, vector<int>& nums)
{

	int l, r;

	l = 0;
	r = nums.size() - 1;
    n--;
	quicksort(nums, l, r, n);

	return nums[n];
}
};
```
