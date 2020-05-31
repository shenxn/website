---
title: '[BZOJ2743][HEOI2012]采花（离线 + 树状数组）'
tags:
  - BZOJ
  - HEOI
  - 树状数组
  - 离线
id: 160
categories:
  - OI
date: 2014-05-17 23:13:42
---

打算开始学主席树了，然后发现好久没写树状数组，就找了道题练练手，谁知今天脑残不宜写题，WA了半天又T了半天。。

<!--more-->

由于题目里要考虑这个区间后面还有没有这种颜色的花，那就把输入按r排序。首先我们需要一个数组f，在r确定的情况下，(f_i) 表示选择区间[i, r]能采(f_i)朵花，那么对于区间[l, r]的询问就是(f_l)的值。而r增加1变为j'，此时第j'朵花加入了区间，稍加分析就可以得到，(f_{pre_{pre_i}+1})到(f_{pre_i})都需加上1（(pre_i)表示与第i朵花颜色相同，且在第i朵花前的最后一朵花的编号），那么我们需要O(n)的时间预处理pre数组。但每次修改这么多数的时间复杂度是无法接受的，于是我们就用到了树状数组。用到一个数组a，其中：

(a_i = begin{cases} f_i & (i = 1) \ f_i - f_{i-1} & (i > 1) end{cases} )

这时候区间修改已经转化为单点修改，将(f_{pre_{pre_i}+1})到(f_{pre_i})加上1就转化为(a_{pre_{pre_i}+1} + 1)，(a_{pre_i+1} - 1)，而(f_i = sumlimits_{j=1}^{i} a_j)，所以就可以通过树状数组维护a。（这是我这么久以来写过的最短的代码）

``` cpp
//HEOI2012 flower.cpp
#include<algorithm>
#include<iostream>
#include<cstring>
#include<cstdlib>
#include<cstdio>
#include<string>
#include<cmath>
#include<queue>
#include<stack>
#include<map>
#include<set>

//#define LOCAL
//#define READ_FILE
//#define READ_FREAD
//#define VISUAL_STUDIO

const int MAXN = 1000010;

#ifdef LOCAL
#define LOCAL_TIME
#define LOCAL_DEBUG
#endif

#ifdef LOCAL_TIME
#include<ctime>
#endif

#ifdef VISUAL_STUDIO
#pragma warning(disable: 4996)
#endif

struct query_node
{
    int l, r, id;
    inline bool operator < (const query_node &b) const
    {
        return r < b.r;
    }
};

int n, c, m;
int color[MAXN];
query_node query[MAXN];
int pre[MAXN], last_color[MAXN];
int ans[MAXN];

int interval[MAXN];

inline int lowerbit(const int x)
{
    return x & (-x);
}

inline void interval_add(const int l, const int add)
{
    for (int i = l; i <= n; i += lowerbit(i))
    {
        interval[i] += add;
    }
}

inline int interval_sum(int n)
{
    int ret = 0;
    while (n > 0)
    {
        ret += interval[n];
        n -= lowerbit(n);
    }
    return ret;
}

int main()
{
#ifdef LOCAL_TIME
    long long start_time_ = clock();
#endif
#ifdef READ_FILE
    freopen("flower.in", "r", stdin);
    freopen("flower.out", "w", stdout);
#endif
    scanf("%d %d %d", &n, &c, &m);
    for (int i = 1; i <= n; i++)
    {
        scanf("%d", color + i);
    }
    for (int i = 0; i < m; i++)
    {
        scanf("%d %d", &query[i].l, &query[i].r);
        query[i].id = i;
    }
    std::sort(query, query + m);
    for (int i = 1; i <= n; i++)
    {
        pre[i] = last_color[color[i]];
        last_color[color[i]] = i;
    }
    int last_right = 0;
    for (int i = 0; i < m; i++)
    {
        while (query[i].r > last_right)
        {
            last_right++;
            interval_add(pre[pre[last_right]] + 1, 1);
            interval_add(pre[last_right] + 1, -1);
        }
        ans[query[i].id] = interval_sum(query[i].l);
    }
    for (int i = 0; i < m; i++)
    {
        printf("%dn", ans[i]);
    }
#ifdef LOCAL_TIME
    printf("run time: %lld msn", clock() - start_time_);
#endif
#ifdef READ_FILE
    fclose(stdin);
    fclose(stdout);
#endif
    return 0;
}
```