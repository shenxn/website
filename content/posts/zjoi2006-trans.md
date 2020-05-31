---
title: '[BZOJ1003][ZJOI2006]物流运输（DP + DIJKSTRA）'
tags:
  - BZOJ
  - DIJKSTRA
  - DP
  - ZJOI
id: 150
categories:
  - OI
date: 2014-05-17 07:39:32
---

好吧这就是一道大水题，很显然的DP思路。DP状态转移方程就是(dp_i = text{min}(f_{1,i}, dp_j + k + f_{j+1,i}) (1 leq j &lt; i))，其中(f_{i,j})表示从i时刻到j时刻（包括i和j）走同一条路所需的总成本。而由于n和m都很小，这个直接暴力标记不能走的点然后做一次DIJKSTRA就可以了，同时把已经计算过的(f_{i,j})存下来，以便下次使用。这样的话总体的时间复杂度大概就是（原谅我不会算，随便乱估计的）(text{O}(n^2 log_2 m))。

<!--more-->

``` cpp
//ZJOI 2006 day1 trans.cpp
#include<algorithm>
#include<iostream>
#include<cstring>
#include<cstdlib>
#include<cstdio>
#include<string>
#include<cmath>
#include<queue>
#include<stack>
#include<set>
#include<map>

//#define LOCAL
//#define READ_FILE
//#define VISUAL_STUDIO

const int MAXM = 22;
const int MAXD = 100;
const int MAXN = 110;

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

struct edge_node
{
    int to;
    int val;
    edge_node *next;
    inline edge_node()
    {
        next = NULL;
    }
};

struct unable_node
{
    int node, begin, end;
};

struct queue_node
{
    int node;
    int val;
    inline queue_node()
    {

    }
    inline queue_node(const int node_, const int val_)
    {
        node = node_;
        val = val_;
    }
    inline bool operator < (const queue_node &b) const
    {
        if (val == -1)
        {
            return true;
        }
        else if (b.val == -1)
        {
            return false;
        }
        return val > b.val;
    }
};

std::priority_queue <queue_node> Q;
queue_node tmp;
int val[MAXN];
bool visit[MAXN];
int n, m, k, e, d;
edge_node edge[MAXM * MAXM];
int edge_top;
edge_node* edge_head[MAXM];
int l, r, v;
unable_node unable[MAXD];
bool sign_able[MAXN];
int f[MAXN][MAXN];
int dp[MAXN];
int ans;

inline void add_edge(const int l, const int r, const int val)
{
    edge[edge_top].to = r;
    edge[edge_top].val = val;
    edge[edge_top].next = edge_head[l];
    edge_head[l] = edge + edge_top++;
}

inline int min(const int a, const int b)
{
    if (a == -1)
    {
        return b;
    }
    else if (b == -1)
    {
        return a;
    }
    return a < b ? a : b;
}

inline int dijkstra(const int begin, const int end)
{
    memset(val, -1, sizeof(val));
    memset(visit, 0, sizeof(visit));
    val[begin] = 0;
    Q.push(queue_node(begin, 0));
    while (!Q.empty())
    {
        tmp = Q.top();
        Q.pop();
        if (visit[tmp.node])
        {
            continue;
        }
        visit[tmp.node] = true;
        for (edge_node *e = edge_head[tmp.node]; e != NULL; e = e->next)
        {
            if (sign_able[e->to])
            {
                if ((val[e->to] == -1) || (val[e->to] > val[tmp.node] + e->val))
                {
                    val[e->to] = val[tmp.node] + e->val;
                    visit[e->to] = false;
                    Q.push(queue_node(e->to, val[e->to]));
                }
            }
        }
    }
    return val[end];
}

inline int min_val(const int begin, const int end)
{
    if (f[begin][end] != -1)
    {
        return f[begin][end];
    }
    memset(sign_able, 1, sizeof(sign_able));
    for (int i = 0; i < d; i++)
    {
        if ((unable[i].begin <= end) && (unable[i].end >= begin))
        {
            sign_able[unable[i].node] = false;
        }
    }
#ifdef LOCAL_DEBUG
    printf("DIJ: %d %d %dn", begin, end, dijkstra(1, m));
#endif
    return f[begin][end] = (end - begin + 1) * dijkstra(1, m);
}

int main()
{
#ifdef LOCAL_TIME
    long long start_time_ = clock();
#endif
#ifdef READ_FILE
    freopen("1003.in", "r", stdin);
    freopen("1003.out", "w", stdout);
#endif
    scanf("%d %d %d %d", &n, &m, &k, &e);
    for (int i = 0; i < e; i++)
    {
        scanf("%d %d %d", &l, &r, &v);
        add_edge(l, r, v);
        add_edge(r, l, v);
    }
    scanf("%d", &d);
    for (int i = 0; i < d; i++)
    {
        scanf("%d %d %d", &unable[i].node, &unable[i].begin, &unable[i].end);
    }
    memset(dp, -1, sizeof(dp));
    memset(f, -1, sizeof(f));
    dp[0] = 0;
    for (int i = 1; i <= n; i++)
    {
        dp[i] = min_val(1, i);
        if (dp[i] <= -1)
        {
            dp[i] = -1;
        }
        for (int j = 1; j < i; j++)
        {
            if ((min_val(j + 1, i) > -1) && (dp[j] > -1))
            {
                dp[i] = min(dp[i], dp[j] + k + min_val(j + 1, i));
            }
        }
    }
    printf("%d", dp[n]);
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