---
title: '[BZOJ2768][JLOI2010]冠军调查（最大流 最高标号预流推进 + GAP优化）'
tags:
  - BZOJ
  - GAP优化
  - JLOI
  - 最大流
  - 最高标号预流推进
id: 211
categories:
  - OI
date: 2014-05-20 15:10:36
---

这题很显然是一道最小割的题目，然后题目只要求最小割的容量，那么直接跑个最大流（关于最小割-最大流定理请自行谷歌），以前写过DINIC感觉不够高大上，然后果断来一个最高标号预流推进秀优越，事实上这个时间复杂度的确很优秀。一开始我写的没加GAP优化，400+ms，跑得比DINIC还慢，真是不爽，下午看到了一种叫GAP优化的东西，加上之后就28ms了，怒刷到rank2。

<!--more-->
> <span style="color: #38761d;">预流推进算法给每一个顶点一个标号h(v)，表示该点到t的最短路（在残量网络中）。> 
> 第一步hights()过程，就是BFS出初始最短路，计算出每一个顶点的h(v)。</span>
> 
> <span style="color: #38761d;">预流推进算法的特征是运用了预流来加快运算。预流说明图中的节点(除s, t)，仅需要满足流入量 &gt;= 流出量。其中流入量&gt;流出量的接点，我们称之为活动节点。我们的算法就是不断地将活动结点，变为非活动结点，使得预流成为可行流。</span>
> 
> <span style="color: #38761d;">算法过程prepare()，即首先将与s相连的边设为满流，并将这时产生的活动结点加入队列Q。这是算法的开始。> 
> 以后便重复以下过程直到Q为空：> 
> (1).选出Q的一个活动顶点u。并依次判断残量网络G'中每条边(u, v)，若h(u) = h(v) + 1，则顺着这里条边推流，直到Q变成非活动结点（不存在多余流量）。(Push推流过程)> 
> (2).如果u还是活动结点。则需要对u进行重新标号：h(u) = min{h(v) + 1}，其中边(u,v)存在于G' 中。然后再将u加入队列。(reCalc过程)</span>
> 
> <span style="color: #38761d;">可以证明，通过以上算法得到的结果就是最大流。> 
> 如果该算法的Q是标准的FIFO队列，则时间复杂度为(n2m)，如果是优先队列，并且标号最高的点优先的话：我们就得到了最高标号预流推进算法，其时间复杂度仅为(n2sqrt(m))，算是比较快的最大流算法了。</span>
以上内容摘自雅礼中学ZQM的网络流PPT：[网络流算法专题.ppt](http://shenxn-others.qiniudn.com/%E7%BD%91%E7%BB%9C%E6%B5%81%E7%AE%97%E6%B3%95%E4%B8%93%E9%A2%98.ppt)

而关于GAP优化，我们需要一个GAP数组记录每层的节点数，当一个层的节点数变为零时，即分层图中出现了空隙，在GAP以上的所有点的预流已经不可能成为可行流，那就将它们从队列中取出，并将它们的高度修改为size以免再次被加入队列。

``` cpp
//JLOI2010 champion.cpp
#include<algorithm>
#include<iostream>
#include<cstring>
#include<cstdlib>
#include<cstdio>
#include<string>
#include<vector>
#include<cmath>
#include<queue>
#include<stack>
#include<map>
#include<set>

//#define LOCAL
#define READ_FREAD
//#define READ_FILE
//#define VISUAL_STUDIO

const int MAXN = 310;
const int MAXM = 100000;

#ifdef LOCAL
#define LOCAL_TIME
//#define LOCAL_DEBUG
//#define STD_DEBUG
//#define DATA
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

int n, m;
int k;
int s, t;
int x, y;
int map[MAXN][MAXN];

struct node_node
{
    int id, height;
    inline node_node(const int id_ = 0, const int height_ = 0)
    {
        id = id_;
        height = height_;
    }
    inline bool operator < (const node_node &b) const
    {
        return height < b.height;
    }
};

bool visit[MAXN];
int inflow[MAXN], height[MAXN];
std::priority_queue <node_node> Q;

int gap[MAXN];

inline void add_edge(const int u, const int v, const int d, const bool both_way = false)
{
    map[u][v] = d;
    map[v][u] = both_way ? d : 0;
}

inline void bfs()
{
    std::queue <int> bfs_q;
    bfs_q.push(t);
    int tmp;
    gap[0] = 1;
    while (!bfs_q.empty())
    {
        tmp = bfs_q.front();
        bfs_q.pop();
        for (int i = s; i <= t; i++)
        {
            if ((!visit[i]) && (map[i][tmp] > 0))
            {
                height[i] = height[tmp] + 1;
                gap[height[i]]++;
                visit[i] = true;
                bfs_q.push(i);
            }
        }
    }
}

inline int max_flow()
{
    memset(visit, false, sizeof(visit));
    for (int i = s + 1; i < t; i++)
    {
        if (map[s][i] > 0)
        {
            inflow[i] = map[s][i];
            map[i][s] += inflow[i];
            map[s][i] = 0;
            Q.push(node_node(i, height[i]));
            visit[i] = true;
        }
    }
    node_node tmp;
    int push_flow = -1;
    int min_height = -1;
    visit[t] = true;
    while (!Q.empty())
    {
        tmp = Q.top();
        Q.pop();
#ifdef LOCAL_DEBUG
        printf("POP ID HEIGHT INFLOW: %d %d %dn", tmp.id, tmp.height, inflow[tmp.id]);
#endif
        visit[tmp.id] = false;
        for (int i = s + 1; (i <= t) && (inflow[tmp.id] > 0); i++)
        {
            if ((height[tmp.id] == height[i] + 1) && (map[tmp.id][i] > 0))
            {
                push_flow = std::min(inflow[tmp.id], map[tmp.id][i]);
                inflow[tmp.id] -= push_flow;
                inflow[i] += push_flow;
                map[tmp.id][i] -= push_flow;
                map[i][tmp.id] += push_flow;
                if (!visit[i])
                {
                    Q.push(node_node(i, height[i]));
                    visit[i] = true;
                }
            }
        }
        min_height = -1;
        if ((inflow[tmp.id] > 0) && (tmp.id != t))
        {
            for (int i = s + 1; i <= t; i++)
            {
                if (map[tmp.id][i] > 0)
                {
                    if ((min_height == -1) || (height[i] < min_height))
                    {
                        min_height = height[i];
                    }
                }
            }
            if ((min_height != -1) && (min_height < t))
            {
                gap[height[tmp.id]]--;
                if (gap[height[tmp.id]] == 0)
                {
                    for (int i = s; i <= t; i++)
                    {
                        if ((height[i] >= height[tmp.id]) && (height[i] < t))
                        {
                            gap[height[i]]--;
                            height[i] = t;
                            if (visit[i])
                            {
                                Q.pop();
                            }
                        }
                    }
                }
                gap[height[tmp.id] = min_height + 1]++;
                Q.push(node_node(tmp.id, height[tmp.id]));
                visit[tmp.id] = true;
            }
        }
    }
    return inflow[t];
}

#ifndef DATA
int main()
{
#ifdef LOCAL_TIME
    long long start_time_ = clock();
#endif
#ifdef READ_FILE
    freopen("champion.in", "r", stdin);
#ifndef STD_DEBUG
    freopen("champion.out", "w", stdout);
#endif
#endif
    read(n);
    read(m);
    s = 1;
    t = n + 2;
    for (int i = 1; i <= n; i++)
    {
        scanf("%d", &k);
        if (k == 1)
        {
            add_edge(s, i + 1, 1);
        }
        else
        {
            add_edge(i + 1, t, 1);
        }
    }
    for (int i = 0; i < m; i++)
    {
        read(x);
        read(y);
        add_edge(x + 1, y + 1, 1, true);
    }
    bfs();
    printf("%d", max_flow());
#ifdef LOCAL_TIME
    printf("nrun time: %lld msn", clock() - start_time_);
#endif
#ifdef READ_FILE
    fclose(stdin);
#ifndef STD_DEBUG
    fclose(stdout);
#endif
#endif
    return 0;
}
#else
int main()
{
    int N = 300;
    int M = 44850;
    int RAND_BASE = 32;
    freopen("champion.in", "w", stdout);
    srand(RAND_BASE);
    printf("%d %dn", N, M);
    for (int i = 0; i < N; i++)
    {
        printf("%d ", rand() % 2);
    }
    printf("n");
    for (int i = 0; i < M; i++)
    {
        int x = rand() % N + 1;
        int y = rand() % N + 1;
        if (x == y)
        {
            i--;
            continue;
        }
        printf("%d %dn", x, y);
    }
    fclose(stdout);
    return 0;
}
#endif
```