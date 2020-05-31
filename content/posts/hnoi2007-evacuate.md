---
title: '[BZOJ1189][HNOI2007]紧急疏散evacuate（二分答案 + 最大流）'
tags:
  - BZOJ
  - GAP优化
  - HNOI
  - 二分答案
  - 最大流
  - 最高标号预流推进
id: 234
categories:
  - OI
date: 2014-05-21 18:17:40
---

做了好几题裸的最大流，来一道稍微不裸一点的（不过这种经典题也没什么好说的），首先题目二分答案，然后用最大流验证是否满流。首先要做一次BFS预处理，预处理出所有的空点到所有门的最短路。做最大流时，首先将源点向所有空位置连一条容量为1的边，将所有的门向汇点连一条容量为时间的边（一开始连了N * M的容量，坑了好久。。），再将所有空点向时间内能到达的门连一条容量为1的边。

<!--more-->

``` cpp
//HNOI2007 evacuate.cpp
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
#include<set>
#include<map>

//#define LOCAL
//#define VISUAL_STUDIO
#define READ_FREAD
//#define READ_FILE

const int MAXN = 25;
const int MAXPT = 410;

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

int n, m;
char map[MAXN][MAXN];
int d[MAXPT][MAXPT];
bool graph_in[MAXPT], graph_out[MAXPT];
int l, r, mid, tot_p;
int s, t;

inline int get_node(const int i, const int j)
{
	return (i - 1) * m + j;
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

int flow_graph[MAXPT][MAXPT];
int height[MAXPT], inflow[MAXPT], gap[MAXPT];
bool visit[MAXPT], in_q[MAXPT];
std::queue <int> bfs_q;
std::priority_queue <queue_node> Q;

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
		for (int i = s; i <= t; i++)
		{
			if ((!visit[i]) && (flow_graph[i][tmp] > 0))
			{
				gap[height[i] = height[tmp] + 1]++;
				visit[i] = true;
				bfs_q.push(i);
			}
		}
	}
}

inline void max_flow_init(const int time)
{
	memset(flow_graph, 0, sizeof(flow_graph));
	memset(height, 0, sizeof(height));
	memset(inflow, 0, sizeof(inflow));
	memset(gap, 0, sizeof(gap));
	memset(visit, false, sizeof(visit));
	memset(in_q, false, sizeof(in_q));
	in_q[s] = in_q[t] = true;
	for (int i = s + 1; i < t; i++)
	{
		flow_graph[s][i] = graph_in[i];
		flow_graph[i][t] = graph_out[i] * time;
	}
	for (int i = s + 1; i < t; i++)
	{
		for (int j = s + 1; j < t; j++)
		{
			if (graph_out[j])
			{
				flow_graph[i][j] = (d[i][j] <= time) && (d[i][j] > 0) ? 1 : 0;
			}
		}
	}
	bfs();
}

inline int max_flow(const int time)
{
	max_flow_init(time);
	for (int i = s + 1; i < t; i++)
	{
		if (flow_graph[s][i] > 0)
		{
			flow_graph[i][s] = inflow[i] = flow_graph[s][i];
			flow_graph[s][i] = 0;
			in_q[i] = true;
			Q.push(queue_node(i, height[i]));
		}
	}
	queue_node tmp;
	int push_flow, min_height;
	while (!Q.empty())
	{
		tmp = Q.top();
		Q.pop();
		in_q[tmp.id] = false;
		for (int i = s + 1; (i <= t) && (inflow[tmp.id] > 0); i++)
		{
			if ((flow_graph[tmp.id][i] > 0) && (height[tmp.id] == height[i] + 1))
			{
				push_flow = std::min(inflow[tmp.id], flow_graph[tmp.id][i]);
				inflow[i] += push_flow;
				inflow[tmp.id] -= push_flow;
				flow_graph[tmp.id][i] -= push_flow;
				flow_graph[i][tmp.id] += push_flow;
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
			for (int i = s + 1; i < t; i++)
			{
				if (flow_graph[tmp.id][i] > 0)
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
					Q.push(queue_node(tmp.id, height[tmp.id]));
				}
				in_q[tmp.id] = true;
			}
		}
	}
	return inflow[t];
}

struct map_node
{
	int x, y;
	inline map_node(const int x_ = 0, const int y_ = 0)
	{
		x = x_;
		y = y_;
	}
};

std::queue <map_node> map_q;

inline void do_bfs(const int x, const int y)
{
	map_node tmp;
	while (!map_q.empty())
	{
		tmp = map_q.front();
		map_q.pop();
		if ((map[tmp.x - 1][tmp.y] == '.') && (d[get_node(tmp.x - 1, tmp.y)][get_node(x, y)] == 0))
		{
			d[get_node(tmp.x - 1, tmp.y)][get_node(x, y)] = d[get_node(tmp.x, tmp.y)][get_node(x, y)] + 1;
			map_q.push(map_node(tmp.x - 1, tmp.y));
		}
		if ((map[tmp.x + 1][tmp.y] == '.') && (d[get_node(tmp.x + 1, tmp.y)][get_node(x, y)] == 0))
		{
			d[get_node(tmp.x + 1, tmp.y)][get_node(x, y)] = d[get_node(tmp.x, tmp.y)][get_node(x, y)] + 1;
			map_q.push(map_node(tmp.x + 1, tmp.y));
		}
		if ((map[tmp.x][tmp.y - 1] == '.') && (d[get_node(tmp.x, tmp.y - 1)][get_node(x, y)] == 0))
		{
			d[get_node(tmp.x, tmp.y - 1)][get_node(x, y)] = d[get_node(tmp.x, tmp.y)][get_node(x, y)] + 1;
			map_q.push(map_node(tmp.x, tmp.y - 1));
		}
		if ((map[tmp.x][tmp.y + 1] == '.') && (d[get_node(tmp.x, tmp.y + 1)][get_node(x, y)] == 0))
		{
			d[get_node(tmp.x, tmp.y + 1)][get_node(x, y)] = d[get_node(tmp.x, tmp.y)][get_node(x, y)] + 1;
			map_q.push(map_node(tmp.x, tmp.y + 1));
		}
	}
}

inline void prepare_bfs(const int x, const int y)
{
	if (map[x][y] != 'D')
	{
		return;
	}
	if (x == 1)
	{
		if (map[x + 1][y] == '.')
		{
			map_q.push(map_node(x + 1, y));
			d[get_node(x + 1, y)][get_node(x, y)] = 1;
			do_bfs(x, y);
		}
	}
	else if (x == n)
	{
		if (map[x - 1][y] == '.')
		{
			map_q.push(map_node(x - 1, y));
			d[get_node(x - 1, y)][get_node(x, y)] = 1;
			do_bfs(x, y);
		}
	}
	else if (y == 1)
	{
		if (map[x][y + 1] == '.')
		{
			map_q.push(map_node(x, y + 1));
			d[get_node(x, y + 1)][get_node(x, y)] = 1;
			do_bfs(x, y);
		}
	}
	else if (y == m)
	{
		if (map[x][y - 1] == '.')
		{
			map_q.push(map_node(x, y - 1));
			d[get_node(x, y - 1)][get_node(x, y)] = 1;
			do_bfs(x, y);
		}
	}
#ifdef LOCAL_DEBUG
	for (int i = 2; i < n; i++)
	{
		for (int j = 2; j < m; j++)
		{
			printf("%d ", d[get_node(i, j)][11]);
		}
		printf("n");
	}
#endif
}

inline void bfs_init()
{
	for (int i = 2; i < n; i++)
	{
		prepare_bfs(i, 1);
		prepare_bfs(i, m);
	}
	for (int i = 2; i < m; i++)
	{
		prepare_bfs(1, i);
		prepare_bfs(n, i);
	}
}

int main()
{
#ifdef LOCAL_TIME
	long long start_time_ = clock();
#endif
#ifdef READ_FILE
	freopen("evacuate.in", "r", stdin);
#ifndef STD_DEBUG
	freopen("evacuate.out", "w", stdout);
#endif
#endif
#ifdef READ_FREAD
	fread_init();
#endif
	read(n);
	read(m);
	s = 0;
	t = n * m + 1;
	for (int i = 1; i <= n; i++)
	{
		for (int j = 1; j <= m; j++)
		{
			read(map[i][j]);
			if (map[i][j] == '.')
			{
				tot_p++;
				graph_in[get_node(i, j)] = true;
			}
			else if (map[i][j] == 'D')
			{
				graph_out[get_node(i, j)] = true;
			}
		}
	}
	bfs_init();
	l = 1, r = n * m;
	while (l < r)
	{
		mid = (l + r) >> 1;
#ifdef LOCAL_DEBUG
		int ret = max_flow(mid);
		printf("MAX_FLOW: %d %d %dn", mid, ret, tot_p);
		if (ret >= tot_p)
#else
		if (max_flow(mid) >= tot_p)
#endif
		{
			r = mid;
		}
		else
		{
			l = mid + 1;
		}
	}
	if (l >= n * m)
	{
		printf("impossible");
	}
	else
	{
		printf("%d", l);
	}
#ifdef LOCAL_TIME
	printf("nrun time: %lld msn", (clock() - start_time_) / 1000);
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