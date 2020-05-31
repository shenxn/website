---
title: '[BZOJ1012][JSOI2008]最大数（树状数组）'
tags:
  - BZOJ
  - JSOI
  - 树状数组
id: 165
categories:
  - OI
date: 2014-05-18 09:11:29
---

树状数组水了一题感觉不熟练就又水了一题。这题主要就是要把整个数列倒过来插，这样就可以把求后L项的最大数转化为求数列前L项的最大数。在树状数组中，维护一个数列大小size，每次插入就插入到第m - size的位置上，每次查询就查询前m - size + l项的最大值（前导零对答案不影响）。

<!--more-->

``` cpp
//JSOI2008 maxnumber.cpp
#include<algorithm>
#include<iostream>
#include<cstring>
#include<cstdlib>
#include<cstdio>
#include<string>
#include<cmath>
#include<stack>
#include<queue>
#include<map>
#include<set>

//#define LOCAL
//#define READ_FILE
//#define VISUAL_STUDIO

#ifdef LOCAL
#define LOCAL_TIME
#define LOCAL_DEBUG
#endif

#ifdef VISUAL_STUDIO
#pragma warning(disable: 4996)
#endif

#ifdef LOCAL_TIME
#include<ctime>
#endif

int m, d;
int size;
char op[2];
int op_num;
int t;
int interval[200010];

inline int lowerbit(const int x)
{
    return x & (-x);
}

inline int interval_query(int x)
{
    int ret = 0;
    for (; x > 0; x -= lowerbit(x))
    {
        ret = std::max(ret, interval[x]);
    }
    return ret;
}

inline void interval_add(int x, const int val)
{
    for (; x <= m; x += lowerbit(x))
    {
        interval[x] = std::max(interval[x], val);
    }
}

int main()
{
#ifdef LOCAL_TIME
    long long start_time_ = clock();
#endif
#ifdef READ_FILE
    freopen("maxnumber.in", "r", stdin);
    freopen("maxnumber.out", "w", stdout);
#endif
    scanf("%d %d", &m, &d);
    for (int i = 0; i < m; i++)
    {
        scanf("%s %d", op, &op_num);
        if (op[0] == 'Q')
        {
            printf("%dn", t = interval_query(m - size + op_num));
        }
        else if (op[0] == 'A')
        {
            op_num = (op_num + t) % d;
            interval_add(m - size, op_num);
            size++;
        }
    }
#ifdef READ_FILE
    fclose(stdin);
    fclose(stdout);
#endif
#ifdef READ_FILE
    fclose(stdin);
    fclose(stdout);
#endif
    return 0;
}
```