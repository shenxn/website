---
title: '[BZOJ1066][SCOI2007]蜥蜴（最大流）'
tags:
  - BZOJ
  - GAP优化
  - SCOI
  - 最大流
  - 最高标号预流推进
id: 221
categories:
  - OI
date: 2014-05-20 18:57:37
---

这道题就是最大流，首先对柱子进行拆点，一个入点和一个出点，入点向出点连一条容量为柱子高度的边，从每个柱子的出点，向可以一次跳到的所有柱子的入点连一条容量为当前柱子高度的边，再从每个可以一次跳出的柱子向汇点连一条容量为柱子高度的边，从源点向每个有蜥蜴的柱子的入点连一条容量为1的边，然后做一次最大流即可。当然答案输出的是不能跳出的蜥蜴，我被坑了好久233

<!--more-->

``` cpp
//SCOI2007 lizard.cpp
#include<algorithm>
#include<iostream>
#include<cstring>
#include<cstdlib>
#include<cstdio>
#include<string>
#include<vector>
#include<queue>
#include<stack>
#include<cmath>
#include<set>
#include<map>

//#define LOCAL
#define READ_FREAD
//#define READ_FILE
//#define VISUAL_STUDIO

const int MAXN = 20;
const int MAXPT = 2 * MAXN * MAXN + 2;
const int MAXM = 170000;

#ifdef LOCAL
#define LOCAL_DEBUG
#define LOCAL_TIME
//#define STD_DEBUG
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

inline char get_char()
{
    char ret;
    while ((fread_char == ' ') || (fread_char == 'n'))
    {
        fread_char = getchar();
    }
    ret = fread_char;
    fread_char = getchar();
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

inline void read(char &a)
{
#ifdef READ_FREAD
    a = get_char();
#else
    scanf("%c", &a);
#endif
}

struct pillar_node
{
    int x, y, d;
};

int graph[MAXPT][MAXPT];

inline void add_edge(const int u, const int v, const int d)
{
#ifdef LOCAL_DEBUG
    printf("ADD_EDGE %d %d %dn", u, v, d);
#endif
    graph[u][v] = d;
    graph[v][u] = 0;
}

struct queue_node
{
    int id, height;
    inline queue_node(const int id_ = 0, const int height_ = 0)
    {
        id = id_;
        height = height_;
    }
    inline bool operator < (const queue_node &b) const
    {
        return height < b.height;
    }
};

int r, c, d;
int tot_lizard;
pillar_node pillar[MAXM];
int tot_pillar;
int k;
char tmp_k;
char lizard_pt;
int s, t;
int gap[MAXPT];
int height[MAXPT], inflow[MAXPT];
bool in_q[MAXPT];
bool visit[MAXPT];
std::priority_queue <queue_node> Q;

inline int in_pt(const int x, const int y)
{
    return (x - 1) * c + y;
}

inline int out_pt(const int x, const int y)
{
    return r * c + (x - 1) * c + y;
}

std::queue <int> bfs_q;

inline void bfs()
{
    gap[0] = 1;
    bfs_q.push(t);
    visit[t] = true;
    int tmp;
    while (!bfs_q.empty())
    {
        tmp = bfs_q.front();
        bfs_q.pop();
        for (int i = s; i < t; i++)
        {
            if ((!visit[i]) && (graph[i][tmp] > 0))
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
    in_q[s] = in_q[t] = true;
    for (int i = s + 1; i < t; i++)
    {
        if (graph[s][i] > 0)
        {
            inflow[i] = graph[s][i];
            graph[i][s] = inflow[i];
            graph[s][i] = 0;
            in_q[i] = true;
            Q.push(queue_node(i, height[i]));
        }
    }
    queue_node tmp;
    int push_flow = -1;
    int min_height = -1;
    while (!Q.empty())
    {
        tmp = Q.top();
        Q.pop();
#ifdef LOCAL_DEBUG
        printf("DEBUG ID HEIGHT INFLOW: %d %d %dn", tmp.id, tmp.height, inflow[tmp.id]);
#endif
        in_q[tmp.id] = false;
        for (int i = s + 1; (i <= t) && (inflow[tmp.id] > 0); i++)
        {
            if ((graph[tmp.id][i] > 0) && (height[tmp.id] == height[i] + 1))
            {
                push_flow = std::min(inflow[tmp.id], graph[tmp.id][i]);
                inflow[tmp.id] -= push_flow;
                inflow[i] += push_flow;
                graph[tmp.id][i] -= push_flow;
                graph[i][tmp.id] += push_flow;
                if (!in_q[i])
                {
                    in_q[i] = true;
                    Q.push(queue_node(i, height[i]));
                }
            }
        }
        if (inflow[tmp.id] > 0)
        {
            min_height = -1;
            for (int i = s + 1; i <= t; i++)
            {
                if (graph[tmp.id][i] > 0)
                {
                    if ((min_height == -1) || (min_height > height[i]))
                    {
                        min_height = height[i];
                    }
                }
            }
            if ((min_height > -1) && (min_height < t))
            {
                gap[height[tmp.id]]--;
                if (gap[height[tmp.id]] == 0)
                {
                    for (int i = s + 1; i < t; i++)
                    {
                        if ((height[i] >= height[tmp.id]) && (height[i] < t))
                        {
                            gap[height[i]]--;
                            height[i] = t;
                            if (in_q[i])
                            {
                                Q.pop();
                            }
                        }
                    }
                }
                else
                {
                    gap[height[tmp.id] = min_height + 1]++;
                    in_q[tmp.id] = true;
                    Q.push(queue_node(tmp.id, height[tmp.id]));
                }
            }
        }
    }
    return inflow[t];
}

int main()
{
#ifdef LOCAL_TIME
    long long start_time_ = clock();
#endif
#ifdef READ_FILE
    freopen("lizard.in", "r", stdin);
#ifndef STD_DEBUG
    freopen("lizard.out", "w", stdout);
#endif
#endif
#ifdef READ_FREAD
    fread_init();
#endif
    read(r);
    read(c);
    read(d);
    s = 0;
    t = r * c * 2 + 1;
    for (int i = 1; i <= r; i++)
    {
        for (int j = 1; j <= c; j++)
        {
            read(tmp_k);
            k = tmp_k - '0';
            if (k > 0)
            {
                pillar[tot_pillar].x = i;
                pillar[tot_pillar].y = j;
                pillar[tot_pillar++].d = k;
                add_edge(in_pt(i, j), out_pt(i, j), k);
            }
        }
    }
    for (int i = 1; i <= r; i++)
    {
        for (int j = 1; j <= c; j++)
        {
            read(lizard_pt);
            if (lizard_pt == 'L')
            {
                tot_lizard++;
                add_edge(s, in_pt(i, j), 1);
            }
        }
    }
    for (int i = 0; i < tot_pillar; i++)
    {
        if ((pillar[i].x <= d) || (pillar[i].y <= d) || (r - pillar[i].x + 1 <= d) || (c - pillar[i].y + 1 <= d))
        {
            add_edge(out_pt(pillar[i].x, pillar[i].y), t, pillar[i].d);
        }
        for (int j = 0; j < tot_pillar; j++)
        {
            if (i == j)
            {
                continue;
            }
            if ((pillar[i].x - pillar[j].x) * (pillar[i].x - pillar[j].x) + (pillar[i].y - pillar[j].y) * (pillar[i].y - pillar[j].y) <= d * d)
            {
                add_edge(out_pt(pillar[i].x, pillar[i].y), in_pt(pillar[j].x, pillar[j].y), pillar[i].d);
            }
        }
    }
    bfs();
    printf("%d", tot_lizard - max_flow());
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
```