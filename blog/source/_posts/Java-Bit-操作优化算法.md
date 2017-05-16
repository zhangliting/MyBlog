---
title: Java Bit 操作优化算法
tags:
  - 算法
  - bit
categories:
  - 算法
date: 2017-03-27 22:04:09
---
# Java Bit 操作优化算法
---
### 基本操作符
1. ``&`` 或
2. ``|`` 与
3. ``^`` 异或
4. ``!`` 非
5. ``<<`` 左移
6. ``>>`` 右移
7. ``>>>`` 无符号右移

### Examples
---
##### 统计1的bit个数
```java
int count_one(int n) {
    while(n) {
        n = n&(n-1);
        count++;
    }
    return count;
}
```
<!-- more -->
##### 判断是否4的N次方
```java
bool isPowerOfFour(int n) {
    return !(n&(n-1)) && (n&0x55555555);
    //check the 1-bit location;
}
```
##### 计算和
```java
int getSum(int a, int b) {
    return b==0? a:getSum(a^b, (a&b)<<1); //be careful about the terminating condition;
}
```
##### 在0~N的数中找到缺少个一个数字
```java
int missingNumber(vector<int>& nums) {
    int ret = 0;
    for(int i = 0; i < nums.size(); ++i) {
        ret ^= i;
        ret ^= nums[i];
    }
    return ret^=nums.size();
}
```
##### 2的N次方<=M,给定M，找打最大的数
```java
long largest_power(long N) {
    //changing all right side bits to 1.
    N = N | (N>>1);
    N = N | (N>>2);
    N = N | (N>>4);
    N = N | (N>>8);
    N = N | (N>>16);
    return (N+1)>>1;
}
```
##### 反转bit
```java
uint32_t reverseBits(uint32_t n) {
    unsigned int mask = 1<<31, res = 0;
    for(int i = 0; i < 32; ++i) {
        if(n & 1) res |= mask;
        mask >>= 1;
        n >>= 1;
    }
    return res;
}
```
```java
x = ((x & 0xaaaaaaaa) >> 1) | ((x & 0x55555555) << 1);
x = ((x & 0xcccccccc) >> 2) | ((x & 0x33333333) << 2);
x = ((x & 0xf0f0f0f0) >> 4) | ((x & 0x0f0f0f0f) << 4);
x = ((x & 0xff00ff00) >> 8) | ((x & 0x00ff00ff) << 8);
x = ((x & 0xffff0000) >> 16) | ((x & 0x0000ffff) << 16);
```