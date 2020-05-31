---
title: '[BZOJ1002][FJOI2007]轮状病毒（DP + 高精度）'
tags:
  - BZOJ
  - DP
  - FJOI
  - 数学
  - 高精度
id: 78
categories:
  - OI
date: 2014-05-16 15:25:47
---

这题的DP实在是太可怕，证明了半天，其实就是排列组合。最后证明出来的式子是

\(f_n = \begin{cases} 1 &amp; (n = 1) \\ 2 &amp; (n = 2) \\ 3 * f_{n-1} - f_{n-2} + 2 &amp; (n \geq 3) \end{cases}\)

<!--more-->

下面开始证明了（LaTeX真是个好东西）：

首先我们设最后要得到的答案是\(f_n\)，设i个成链状的外围点和中心点组成的图的生成树总数为\(g_i\)。

先考虑\(g_i\)。显然\(g_0 = g_1 = 1\)（此时只有一个中心点，或只有一个中心点和一个外围点）。

然后对于\(i \geq 2\)，这种情况可以由\(g_{i-j} (1 \leq j &lt; i)\)的情况加上i-j个点得到。这里我们只考虑这j-i个点连成一条链，且有一个点与中心点相连的情况，并且仅考虑这i-j个点接在j个点后。稍加考虑即可知道其他的情况都是重复的。而这i-j个点共有i-j种组合（每个点都有可能连接中心点）。于是就可以得到：

\(\begin{eqnarray*}g_i &amp; = &amp; \sum\limits_{j=1}^{i-1} (i - j) * (g_j) &amp; (1)\\g_{i-1} &amp; = &amp; \sum\limits_{j=1}^{i-2} (i - j - 1) * (g_j) &amp; (2)\end{eqnarray*}\)

(1) - (2)可得

\(g_i - g_{i-1} = [i - (i - 1)] * g_{i-1} + \sum\limits_{j=1}^{i-2} g_j\)

所以

\(\begin{eqnarray*}g_i &amp; = &amp; g_{i-1} + \sum\limits_{j=1}^{i-1} g_j &amp; (3)\\g_{i-1} &amp; = &amp; g_{i-2} + \sum\limits_{j=1}^{i-2} g_j &amp; (4)\end{eqnarray*}\)

(3) - (4)可得

\(g_i - g_{i-1} = g_{i-1} - g_{i-2} + \sum\limits_{j=1}^{i-1} g_j - \sum\limits_{j=1}^{i-2} g_j\)

所以

\(g_i = 3 * g_{i-1} - g_{i-2} (i \geq 2)\)

综上所述

\(g_i = \begin{cases} 1 &amp; (i = 0, 1) \\ 3 &amp; (i = 2) \\ 3 * g_{i-1} - g_{i-2} &amp; (i \geq 3) \end{cases}\)

到这里\(g_i\)已经推完了，现在要用\(g_i\)来推\(f_n\)。这次我们要枚举外围点中最上面那个点（选其他的点也可以）所在的链的长度(该链有一个点与中心点相连），我们只需考虑其中一个点，因为其他的情况都是重复的。若该点所在的链的长度为i，则该链覆盖的点共有i种可能，对于每种可能，又有i种组合（每个点都可能连接中心点），所以共有\(i^2\)种可能。而对于剩下的n-i个点，即有\(g_{n-i}\)种。所以可以得到：

\(\begin{eqnarray*}f_n &amp; = &amp; \sum\limits_{i=1}^{n} i^2 * g_{n-i}\\ &amp; = &amp; \sum\limits_{i=1}^{n-3} (i^2 * 3 * g_{n-i-1} - i^2 * g_{n-i-2}) + (n - 2)^2 * g_2 + (n - 1)^2 * g_1 + n^2 * g_0 \end{eqnarray*}\)

又

\(\begin{eqnarray*}f_{n-1} &amp; = &amp; \sum\limits_{i=1}^{n-1} i^2 * g_{n-i-1} \\ &amp; = &amp; \sum\limits_{i=1}^{n-3} i^2 * g_{n-i-1} + (n - 2)^2 * g_1 + (n - 1)^2 * g_0 \end{eqnarray*}\)

\(\begin{eqnarray*}f_{n-2} &amp; = &amp; \sum\limits_{i=1}^{n-2} i^2 * g_{n-i-2} \\ &amp; = &amp; \sum\limits_{i=1}^{n-3} i^2 * g_{n-i-2} + (n - 2)^2 * g_0 \end{eqnarray*}\)

则

\(\begin{eqnarray*}3 * f_{n-1} - f_{n-2} &amp; = &amp; 3 * [\sum\limits_{i=1}^{n-3} i^2 * g_{n-i-1} + (n - 2)^2 * g_1 + (n - 1)^2 * g_0] - \sum\limits_{i=1}^{n-3} i^2 * g_{n-i-2} + (n - 2)^2 * g_0 \\ &amp; = &amp; \sum\limits_{i=1}^{n-3} (3 * i^2 * g_{n-i-1} - i^2 * g_{n-i-2}) + 3 * (n - 2)^2 * g_1 + 3 * (n - 1)^2 * g_{0} - (n - 2)^2 * g_0 \\ &amp; = &amp; f_n - (n - 2)^2 * g_2 - (n - 1)^2 * g_1 - n^2 * g_0 + 3 * (n - 2)^2 * g_1 + 3 * (n - 1)^2 * g_0 - (n - 2)^2 * g_0 \\ &amp; = &amp; f_n - (n^2 - 4n - 4) * 3 - (n^2 - 2n + 1) * 1 - n^2 * 1 + 3 * (n^2 - 4n + 4) * 1 + 3 * (n^2 - 2n + 1) * 1 - (n^2 - 4n + 4) * 1 \\ &amp; = &amp; f_n - 2 \end{eqnarray*}\)

所以

\(f_n = 3 * f_{n-1} - f_{n-2} + 2 (n \geq 3)\)

综上所述

\(f_n = \begin{cases} 1 &amp; (n = 1) \\ 2 &amp; (n = 2) \\ 3 * f_{n-1} - f_{n-2} + 2 &amp; (n \geq 3) \end{cases}\)

我总觉得会有更简单的证法，当然还见过什么分奇偶找规律的，还有什么基尔霍夫矩阵（那是什么，没听说过），有空再去想了。

证明完之后，还要一个高精度，然后就是水题了。听说N只有100，直接上代码：

``` cpp
//FJOI 2007 virus.cpp
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
//#define VISUAL_STUDIO

const int MAXBIT = 1000;
const int MAXN = 110;

#ifdef LOCAL
#define LOCAL_TIME
#define LOCAL_DEBUG
#endif

#ifdef VISUAL_STUDIO
#pragma warning(disable:4996)
#endif

#ifdef LOCAL_TIME
#include<ctime>
#endif

struct bign
{
    int val[MAXBIT];
    int length;
    inline bign()
    {
        memset(val, 0, MAXBIT);
        length = 1;
    }
    inline bign(char *a)
    {
        memset(val, 0, MAXBIT);
        length = strlen(a);
        for (int i = 0; i < length; i++)
        {
            val[i] = a[length - i - 1] - '0';
        }
    }
    inline bign(const int a)
    {
        memset(val, 0, MAXBIT);
        char tmp[MAXBIT];
        sprintf(tmp, "%d", a);
        *this = bign(tmp);
    }
    inline bign operator = (const int a)
    {
        return *this = bign(a);
    }
    inline bign operator = (char *a)
    {
        return *this = bign(a);
    }
    inline bign operator * (const int b) const
    {
        int tmp = 0;
        bign c;
        c.length = length;
        for (int i = 0; i < c.length; i++)
        {
            c.val[i] = val[i] * b + tmp;
            tmp = c.val[i] / 10;
            c.val[i] %= 10;
        }
        while (tmp > 0)
        {
            c.val[c.length++] = tmp % 10;
            tmp /= 10;
        }
        return c;
    }
    inline bign operator + (const bign &b) const
    {
        int tmp = 0;
        bign c;
        c.length = std::max(length, b.length);
        for (int i = 0; i < c.length; i++)
        {
            c.val[i] = val[i] + b.val[i] + tmp;
            tmp = 0;
            if (c.val[i] >= 10)
            {
                tmp = 1;
                c.val[i] -= 10;
            }
        }
        if (tmp > 0)
        {
            c.val[c.length++] = tmp;
            tmp = 0;
        }
        return c;
    }
    inline bign operator + (const int b) const
    {
        return *this + bign(b);
    }
    inline bign operator - (const bign &b) const
    {
        bign c;
        c.length = length;
        for (int i = 0; i < length; i++)
        {
            c.val[i] += val[i] - b.val[i];
            if (c.val[i] < 0)
            {
                c.val[i] += 10;
                c.val[i + 1]--;
            }
        }
        while (c.val[c.length - 1] == 0)
        {
            c.length--;
        }
        return c;
    }
    inline void print() const
    {
        for (int i = length - 1; i >= 0; i--)
        {
            printf("%d", val[i]);
        }
    }
};

int n;
bign f[MAXN];

int main()
{
#ifdef LOCAL_TIME
    long long start_time_ = clock();
#endif
#ifdef READ_FILE
    freopen("1002.in", "r", stdin);
    freopen("1002.out", "w", stdout);
#endif
    scanf("%d", &n);
    f[1] = 1;
    f[2] = 5;
    for (int i = 3; i <= n; i++)
    {
        f[i] = f[i - 1] * 3 - f[i - 2] + 2;
    }
    f[n].print();
    /*for (int i = 1; i <= 100; i++)
    {
        printf("%d ", i);
        f[i].print();
        printf("\n");
    }*/
#ifdef LOCAL_TIME
    printf("\nrun time: %lld ms\n", clock() - start_time_);
#endif
#ifdef READ_FILE
    fclose(stdin);
    fclose(stdout);
#endif
    return 0;
}
```

当然咯，N只有100，果断敲个TABLE刷RANK，只是内存懒得优化，排在超级后面。