---
title: '[BZOJ1433][ZJOI2009]假期的宿舍（最大流）'
tags:
  - BZOJ
  - GAP优化
  - ZJOI
  - 最大流
  - 最高标号预流推进
id: 291
categories:
  - OI
date: 2014-05-20 20:58:14
---

又是一道最大流，建图很简单，源点到所有的在校学生和外校学生建一条容量为1的边，所有在校学生的床（感觉听起来怪怪的）到汇点连一条边，所有在校学生和自己的床连一条边，所有学生到他认识的在校学生的床连一条边。然后做最大流与需要住校的学生总数比较，相等就输出^_^，不相等就输出T_T（这个太逗了）。。

<!--more-->

``` cpp
//ZJOI2009 holiday.cpp
#include<algorithm>
#include<iostream>
#include<cstring>
#include<cstdlib>
#include<cstdio>
#include<vector>
#include<cmath>
#include<queue>
#include<stack>
#include<map>
#include<set>

//#define LOCAL
//#define READ_FILE
#define READ_FREAD
//#define VISUAL_STUDIO

const int MAXN = 50;
const int MAXPT = 110;

#ifdef LOCAL
#define LOCAL_TIME
#define LOCAL_DEBUG
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

int cases, n;
int s, t;
int tot_at_school;
int school_student[MAXN];
int at_home[MAXN];
int k;
int graph[MAXPT][MAXPT];
int height[MAXPT], inflow[MAXPT], gap[MAXPT];
bool visit[MAXPT], in_q[MAXPT];
std::queue <int> bfs_q;
std::priority_queue <queue_node> Q;

inline void add_edge(const int u, const int v)
{
#ifdef LOCAL_DEBUG
	printf("ADDEDGE %d %dn", u, v);
#endif
	graph[u][v] = 1;
}

inline void bfs()
{
	memset(visit, false, sizeof(visit));
	memset(height, 0, sizeof(height));
	memset(gap, 0, sizeof(gap));
	gap[0] = 1;
	bfs_q.push(t);
	int tmp;
	while (!bfs_q.empty())
	{
		tmp = bfs_q.front();
		bfs_q.pop();
		for (int i = s; i < t; i++)
		{
			if ((graph[i][tmp] > 0) && (!visit[i]))
			{
				gap[height[i] = height[tmp] + 1]++;
				visit[i] = true;
				bfs_q.push(i);
			}
		}
	}
}

inline int max_flow()
{
	memset(in_q, false, sizeof(in_q));
	memset(inflow, 0, sizeof(inflow));
	in_q[s] = in_q[t] = true;
	for (int i = 1; i <= n; i++)
	{
		if (graph[s][i] > 0)
		{
			graph[i][s] = inflow[i] = graph[s][i];
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
		printf("POP ID HEIGHT INFLOW: %d %d %dn", tmp.id, tmp.height, inflow[tmp.id]);
#endif
		in_q[tmp.id] = false;
		for (int i = s + 1; (i <= t) && (inflow[tmp.id] > 0); i++)
		{
			if ((graph[tmp.id][i] > 0) && (height[tmp.id] == height[i] + 1))
			{
				push_flow = std::min(graph[tmp.id][i], inflow[tmp.id]);
				graph[tmp.id][i] -= push_flow;
				graph[i][tmp.id] += push_flow;
				inflow[tmp.id] -= push_flow;
				inflow[i] += push_flow;
				if (!in_q[i])
				{
					in_q[i] = true;
					Q.push(queue_node(i, height[i]));
				}
			}
		}
		if ((inflow[tmp.id] > 0) && (tmp.id < t))
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
			if ((min_height > -1) & (min_height < t))
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
							else
							{
								in_q[i] = true;
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
	freopen("holiday.in", "r", stdin);
#ifndef STD_DEBUG
	freopen("holiday.out", "w", stdout);
#endif
#endif
	read(cases);
	while (cases--)
	{
		read(n);
		s = 0;
		t = 2 * n + 1;
		tot_at_school = 0;
		memset(graph, 0, sizeof(graph));
		for (int i = 1; i <= n; i++)
		{
			read(school_student[i]);
			if (school_student[i])
			{
				add_edge(n + i, t);
			}
			else
			{
				add_edge(s, i);
				tot_at_school++;
			}
		}
		for (int i = 1; i <= n; i++)
		{
			read(at_home[i]);
			if (school_student[i] && (!at_home[i]))
			{
				add_edge(s, i);
				tot_at_school++;
			}
		}
		for (int i = 1; i <= n; i++)
		{
			for (int j = 1; j <= n; j++)
			{
				read(k);
				if (i == j)
				{
					if (school_student[i])
					{
						add_edge(i, n + i);
					}
				}
				else if (k)
				{
					if (school_student[i])
					{
						add_edge(j, n + i);
					}
					if (school_student[j])
					{
						add_edge(i, n + j);
					}
				}
			}
		}
		bfs();
#ifdef LOCAL_DEBUG
		int ret = max_flow();
		printf("%d %dn", ret, tot_at_school);
		if (ret == tot_at_school)
#else
		if (max_flow() == tot_at_school)
#endif
		{
			printf("^_^n");
		}
		else
		{
			printf("T_Tn");
		}
	}
#ifdef LOCAL_TIME
	printf("run time: %lld msn", clock() - start_time_);
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