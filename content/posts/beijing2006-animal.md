---
title: '[BZOJ1001][BEIJING2006]狼抓兔子（最小割 + DIJKSTRA）'
tags:
  - BEIJING
  - BZOJ
  - DIJKSTRA
  - 最小割
id: 181
categories:
  - OI
date: 2014-05-19 11:20:50
---

这道题很裸的最小割，只是数据规模大，目测会T（WJS大爷说在BZ上能过，真是跪烂了），其实最小割可以转成最短路的做法。一种做法是转成对偶图然后求最短路，当然我不会，就只好去墙角画圈圈了。不过有一天没事乱翻LRJ的白书突然就翻到了这道题（我真的只是乱翻），然后就观摩了一下题解。转什么对偶图啊。。对于这道题，由于这个图样子比较特殊，可以知道割线必然是一条从图的左边界或下边界的边出发，经过若干条边到达右边界或上边界的边，需要的狼的数量就是经过的所有边的权值和。

<!--more-->

下面就是建图了，把原图中所有的边建成点，将每个小直角三角形内的三条边两两相连（实际实现中不需要连起来，只要做DIJKSTRA时枚举所有相连的边即可），再建一个超级源连接所有左边界和下边界的边，一个超级汇连接所有上边界和右边界的边（实际实现中不需要建，只需将左边界和下边界的边的d都初始化为边的权值，最终的答案即为上边界和右边界的边的d的最小值），对图做最短路就行了。一开始DIJKSTRA打错一个变量名MLE了（为什么坑爹的是MLE），害得我还去改了SPFA，莫名其妙的16S AC，后来发现了错改回DIJKSTRA就4.7S了。

``` cpp
//BEIJING2006 animal.cpp
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
//#define VISUAL_STUDIO
#define READ_FREAD
//#define READ_FILE

const int MAXN = 1010;

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

#ifdef READ_FREAD
char fread_char;

inline void fread_init()
{
    fread_char = getchar();
}

inline int get_int()
{
    int ret = 0;
    while ((fread_char < '0') || (fread_char > '9'))
    {
        fread_char = getchar();
    }
    while ((fread_char >= '0') && (fread_char <= '9'))
    {
        ret = ret * 10 + fread_char - '0';
        fread_char = getchar();
    }
    return ret;
}
#endif

inline void read(int &a)
{
#ifdef READ_FREAD
    a = get_int();
#else
    scanf("%d", &a);
#endif
}

struct queue_node
{
    int d;
    int type; //1: row, 2: list, 3:slant
    int x, y;
    inline queue_node()
    {

    }
    inline queue_node(const int d_, const int type_, const int x_, const int y_)
    {
        d = d_;
        type = type_;
        x = x_;
        y = y_;
    }
    inline bool operator < (const queue_node &b) const
    {
        if (d == -1)
        {
            return b.d == -1 ? false : true;
        }
        else if (b.d == -1)
        {
            return false;
        }
        return d > b.d;
    }
};

int n, m;
int row[MAXN][MAXN], list[MAXN][MAXN], slant[MAXN][MAXN];
int d_row[MAXN][MAXN], d_list[MAXN][MAXN], d_slant[MAXN][MAXN];
bool visit_row[MAXN][MAXN], visit_list[MAXN][MAXN], visit_slant[MAXN][MAXN];
std::priority_queue <queue_node> Q;

inline int min(const int a, const int b)
{
    if (a == -1)
    {
        return b;
    }
    if (b == -1)
    {
        return a;
    }
    return a < b ? a : b;
}

inline int dijkstra()
{
    memset(d_row, -1, sizeof(d_row));
    memset(d_list, -1, sizeof(d_list));
    memset(d_slant, -1, sizeof(d_slant));
    for (int i = 1; i < n; i++)
    {
        d_list[i][1] = list[i][1];
        Q.push(queue_node(d_list[1][i], 2, i, 1));
    }
    for (int i = 1; i < m; i++)
    {
        d_row[n][i] = row[n][i];
        Q.push(queue_node(d_row[n][i], 1, n, i));
    }
    queue_node tmp;
    while (!Q.empty())
    {
        tmp = Q.top();
        Q.pop();
        if (tmp.type == 1)
        {
            if (visit_row[tmp.x][tmp.y])
            {
                continue;
            }
            visit_row[tmp.x][tmp.y] = true;
            if (tmp.x > 1)
            {
                if ((d_slant[tmp.x - 1][tmp.y] == -1) || (tmp.d + slant[tmp.x - 1][tmp.y] < d_slant[tmp.x - 1][tmp.y]))
                {
                    d_slant[tmp.x - 1][tmp.y] = tmp.d + slant[tmp.x - 1][tmp.y];
                    Q.push(queue_node(d_slant[tmp.x - 1][tmp.y], 3, tmp.x - 1, tmp.y));
                    visit_slant[tmp.x - 1][tmp.y] = false;
                }
                if ((d_list[tmp.x - 1][tmp.y] == -1) || (tmp.d + list[tmp.x - 1][tmp.y] < d_list[tmp.x - 1][tmp.y]))
                {
                    d_list[tmp.x - 1][tmp.y] = tmp.d + list[tmp.x - 1][tmp.y];
                    Q.push(queue_node(d_list[tmp.x - 1][tmp.y], 2, tmp.x - 1, tmp.y));
                    visit_list[tmp.x - 1][tmp.y] = false;
                }
            }
            if (tmp.x < n)
            {
                if ((d_list[tmp.x][tmp.y + 1] == -1) || (tmp.d + list[tmp.x][tmp.y + 1] < d_list[tmp.x][tmp.y + 1]))
                {
                    d_list[tmp.x][tmp.y + 1] = tmp.d + list[tmp.x][tmp.y + 1];
                    Q.push(queue_node(d_list[tmp.x][tmp.y + 1], 2, tmp.x, tmp.y + 1));
                    visit_list[tmp.x][tmp.y + 1] = false;
                }
                if ((d_slant[tmp.x][tmp.y] == -1) || (tmp.d + slant[tmp.x][tmp.y] < d_slant[tmp.x][tmp.y]))
                {
                    d_slant[tmp.x][tmp.y] = tmp.d + slant[tmp.x][tmp.y];
                    Q.push(queue_node(d_slant[tmp.x][tmp.y], 3, tmp.x, tmp.y));
                    visit_slant[tmp.x][tmp.y] = false;
                }
            }
        }
        else if (tmp.type == 2)
        {
            if (visit_list[tmp.x][tmp.y])
            {
                continue;
            }
            visit_list[tmp.x][tmp.y] = true;
            if (tmp.y > 1)
            {
                if ((d_row[tmp.x][tmp.y - 1] == -1) || (tmp.d + row[tmp.x][tmp.y - 1] < d_row[tmp.x][tmp.y - 1]))
                {
                    d_row[tmp.x][tmp.y - 1] = tmp.d + row[tmp.x][tmp.y - 1];
                    Q.push(queue_node(d_row[tmp.x][tmp.y - 1], 1, tmp.x, tmp.y - 1));
                    visit_row[tmp.x][tmp.y - 1] = false;
                }
                if ((d_slant[tmp.x][tmp.y - 1] == -1) || (tmp.d + slant[tmp.x][tmp.y - 1] < d_slant[tmp.x][tmp.y - 1]))
                {
                    d_slant[tmp.x][tmp.y - 1] = tmp.d + slant[tmp.x][tmp.y - 1];
                    Q.push(queue_node(d_slant[tmp.x][tmp.y - 1], 3, tmp.x, tmp.y - 1));
                    visit_slant[tmp.x][tmp.y - 1] = false;
                }
            }
            if (tmp.y < m)
            {
                if ((d_slant[tmp.x][tmp.y] == -1) || (tmp.d + slant[tmp.x][tmp.y] < d_slant[tmp.x][tmp.y]))
                {
                    d_slant[tmp.x][tmp.y] = tmp.d + slant[tmp.x][tmp.y];
                    Q.push(queue_node(d_slant[tmp.x][tmp.y], 3, tmp.x, tmp.y));
                    visit_slant[tmp.x][tmp.y] = false;
                }
                if ((d_row[tmp.x + 1][tmp.y] == -1) || (tmp.d + row[tmp.x + 1][tmp.y] < d_row[tmp.x + 1][tmp.y]))
                {
                    d_row[tmp.x + 1][tmp.y] = tmp.d + row[tmp.x + 1][tmp.y];
                    Q.push(queue_node(d_row[tmp.x + 1][tmp.y], 1, tmp.x + 1, tmp.y));
                    visit_row[tmp.x + 1][tmp.y] = false;
                }
            }
        }
        else if (tmp.type == 3)
        {
            if (visit_slant[tmp.x][tmp.y])
            {
                continue;
            }
            visit_slant[tmp.x][tmp.y] = true;
            if ((d_row[tmp.x][tmp.y] == -1) || (tmp.d + row[tmp.x][tmp.y] < d_row[tmp.x][tmp.y]))
            {
                d_row[tmp.x][tmp.y] = tmp.d + row[tmp.x][tmp.y];
                Q.push(queue_node(d_row[tmp.x][tmp.y], 1, tmp.x, tmp.y));
                visit_row[tmp.x][tmp.y] = false;
            }
            if ((d_row[tmp.x + 1][tmp.y] == -1) || (tmp.d + row[tmp.x + 1][tmp.y] < d_row[tmp.x + 1][tmp.y]))
            {
                d_row[tmp.x + 1][tmp.y] = tmp.d + row[tmp.x + 1][tmp.y];
                Q.push(queue_node(d_row[tmp.x + 1][tmp.y], 1, tmp.x + 1, tmp.y));
                visit_row[tmp.x + 1][tmp.y] = false;
            }
            if ((d_list[tmp.x][tmp.y] == -1) || (tmp.d + list[tmp.x][tmp.y] < d_list[tmp.x][tmp.y]))
            {
                d_list[tmp.x][tmp.y] = tmp.d + list[tmp.x][tmp.y];
                Q.push(queue_node(d_list[tmp.x][tmp.y], 2, tmp.x, tmp.y));
                visit_list[tmp.x][tmp.y] = false;
            }
            if ((d_list[tmp.x][tmp.y + 1] == -1) || (tmp.d + list[tmp.x][tmp.y + 1] < d_row[tmp.x][tmp.y + 1]))
            {
                d_list[tmp.x][tmp.y + 1] = tmp.d + list[tmp.x][tmp.y + 1];
                Q.push(queue_node(d_list[tmp.x][tmp.y + 1], 2, tmp.x, tmp.y + 1));
                visit_list[tmp.x][tmp.y + 1] = false;
            }
        }
    }
    int min_num = -1;
    for (int i = 1; i < m; i++)
    {
        min_num = min(min_num, d_row[1][i]);
    }
    for (int i = 1; i <= n; i++)
    {
        min_num = min(min_num, d_list[i][m]);
    }
    return min_num;
}

int main()
{
#ifdef LOCAL_TIME
    long long start_time_ = clock();
#endif
#ifdef READ_FILE
    freopen("animal.in", "r", stdin);
    freopen("animal.out", "w", stdout);
#endif
    read(n);
    read(m);
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j < m; j++)
        {
            read(row[i][j]);
        }
    }
    for (int i = 1; i < n; i++)
    {
        for (int j = 1; j <= m; j++)
        {
            read(list[i][j]);
        }
    }
    for (int i = 1; i < n; i++)
    {
        for (int j = 1; j < m; j++)
        {
            read(slant[i][j]);
        }
    }
    printf("%d", dijkstra());
#ifdef LOCAL_TIME
    printf("nrun time: %lld msn", clock() - start_time_);
#endif
#ifdef READ_FILE
    fclose(stdin);
    fclose(stdout);
#endif
    return 0;
}
```