---
layout: post
title: "solution for joseph question"
description: "joseph question"
category:
tags: [algorithm]
---

[囧瑟夫问题](https://www.coursera.org/learn/jisuanji-biancheng/programming/nXnUt/shu-ju-cheng-fen-ying-yong-lian-xi):

#### 描述

有ｎ只猴子，按顺时针方向围成一圈选大王（编号从１到ｎ），从第１号开始报数，一直数到ｍ，数到ｍ的猴子退出圈外，剩下的猴子再接着从1开始报数。就这样，直到圈内只剩下一只猴子时，这个猴子就是猴王，编程求输入ｎ，ｍ后，输出最后猴王的编号。

#### 输入
每行是用空格分开的两个整数，第一个是 n, 第二个是 m ( 0 < m,n <=300)。最后一行是：

```
0 0
```

#### 输出

对于每行输入数据（最后一行除外)，输出数据也是一行，即最后猴王的编号

#### 样例输入

```
6 2
12 4
8 3
0 0
```

#### 样例输出

```
5
1
7
```

有用类似链表的实现方法，我没有这么做……

```c++
//for joseph problem
#include <iostream>
using namespace std;

int joseph(int n, int m){
  int flat[300];
  for(int i = 0; i < n; i++)
    flat[i] = 1;

  for(int i = 0, count = n, mod = m - 1; count != 1; i = ((++i) % n)){
      if(flat[i] == 1){
        if(mod == 0){
          flat[i] = 0;
          count--;
        }
        mod = (mod - 1) % m;
      }
  }
  for(int i = 0; i < n; i++){
    if(flat[i] == 1)
      return i + 1;
  }
}

int main(){
  int set_n[100], set_m[100];
  int count = 0;

  while(true){
    int n, m;
    cin >> n >> m;
    if ((n == 0) && (m == 0)){
      break;
    } else {
      set_n[count] = n;
      set_m[count] = m;
      count++;
    }
  }

  for(int i = 0; i < count; i++)
    cout << joseph(set_n[i], set_m[i]) << endl;

  return 0;
}
```
