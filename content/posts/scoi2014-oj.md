---
title: '[BZOJ3595][SCOI2014]方伯伯的OJ（MAP + SPLAY）'
tags:
  - BZOJ
  - MAP
  - SCOI
  - SPLAY
  - STL
id: 290
categories:
  - OI
date: 2014-05-19 18:50:52
---

这道题实在是坑爹，比赛的时候一度以为要两个MAP两棵SPLAY，然后感觉写着蛋疼就写了一个SPLAY 40分滚粗了。后来重新看题，发现其实AC算法也只要一个SPLAY就好了，还有一个SPLAY也可以用MAP来代替（在此BS一下PASCAL，问PASCAL党做这种题该怎么办）。

<!--more-->

首先我们要考虑第一个操作，我们肯定是不能在SPLAY里改编号的，所以说需要两个MAP来维护用户编号的修改，一个MAP将初始编号映射到修改后的编号，另一个将修改后的编号映射到初始编号，这样每次操作的时候就转成初始编号，输出的时候再转成修改后的编号即可。

然后我们来考虑排名的维护。因为N有(10^8)空间根本开不下，所以时空复杂度一定是关于M的。事实上，我们不需要维护所有的用户，因为只有M个操作，所以至多只有M个用户会被操作到，那么剩下的用户就可以合并起来了。但是这道题强制在线，所以我们不能一开始就分好组，只能在操作的过程中分。初始情况下SPLAY中只有一个节点包含[1, n]整个区间，当我们需要移动一个编号为i用户时，我们就将这个节点拆分成[1, i-1]、[i, i]、[i+1, n]三个节点，然后就可以进行操作了。这样做的话排名已经能很好地维护了，但是我没有办法通过用户的编号找到用户所在的SPLAY节点，所以原来我考虑再开一个相似的SPLAY，只是根据节点编号来维护，每个节点再映射到原SPLAY上相应的节点。后来我发现每个SPLAY节点区间的右端点r是唯一的，而MAP是可以做二分查找的，所以只要再开一个节点维护区间右端点r到SPLAY节点的映射，每次操作时用lower_bound查找（lower_bound(x)返回大于等于x的迭代器，而uper_bound(x)返回大于x的迭代器，很显然这里需要用lower_bound）。这样就做完了。然后一开始我开了10W + 10个节点结果RE了，我还以为写渣，后来开到14W就A了，因为最多有10W个节点要被操作，那么最多会有20W + 1个区间，然后再除去一些查询操作，总之10W一定是不够的。

最后莫名其妙还拿了个RANK1（虽然就那么几个人A了）。

``` cpp
//SCOI2014 oj.cpp
#include<algorithm>
#include<iostream>
#include<cstring>
#include<cstdlib>
#include<string>
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

const int MAXM = 140000;

#ifdef LOCAL
#define LOCAL_TIME
#define LOCAL_DEBUG
//#define STD_DEBUG
#endif

#ifdef LOCAL_TIME
#include<ctime>
#endif

#ifdef VISUAL_STUDIO
#pragma warning(disable: 4996)
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
int a;
int op;
int x, y, k;
std::map <int, int> id_to_num;
std::map <int, int> num_to_id;
std::map <int, int> ::iterator tmp;
std::map <int, int> interval_to_splay;

struct splay_node
{
    int l, r;
    splay_node *l_child, *r_child, *parent;
    int size;
    inline splay_node *init()
    {
        l_child = r_child = parent = NULL;
        return this;
    }
};

splay_node recover_splay_node[MAXM];
splay_node *stack_recover_splay_node[MAXM];
int top_stack_recover_splay_node = MAXM;

inline void recover_splay_init()
{
    for (int i = 0; i < MAXM; i++)
    {
        stack_recover_splay_node[i] = recover_splay_node + i;
    }
}

inline splay_node *new_node()
{
    return stack_recover_splay_node[--top_stack_recover_splay_node]->init();
}

inline void delete_node(splay_node *del_node)
{
    stack_recover_splay_node[top_stack_recover_splay_node++] = del_node;
}

splay_node *splay_head;

inline void attach_as_l_child(splay_node *parent, splay_node *child)
{
    parent->l_child = child;
    if (child != NULL)
    {
        child->parent = parent;
    }
}

inline void attach_as_r_child(splay_node *parent, splay_node *child)
{
    parent->r_child = child;
    if (child != NULL)
    {
        child->parent = parent;
    }
}

inline int splay_size(splay_node *query_node)
{
    if (query_node == NULL)
    {
        return 0;
    }
    return query_node->size;
}

inline void splay_update_size(splay_node *update_node)
{
    update_node->size = splay_size(update_node->l_child) + splay_size(update_node->r_child) + update_node->r - update_node->l + 1;
}

inline void splay_zig(splay_node *axis)
{
    splay_node *left_child = axis->l_child;
    splay_node *parent = axis->parent;
    attach_as_l_child(axis, left_child->r_child);
    attach_as_r_child(left_child, axis);
    left_child->parent = parent;
    if (parent != NULL)
    {
        if (parent->l_child == axis)
        {
            parent->l_child = left_child;
        }
        else
        {
            parent->r_child = left_child;
        }
    }
    left_child->size = axis->size;
    splay_update_size(axis);
}

inline void splay_zag(splay_node *axis)
{
    splay_node *right_child = axis->r_child;
    splay_node *parent = axis->parent;
    attach_as_r_child(axis, right_child->l_child);
    attach_as_l_child(right_child, axis);
    right_child->parent = parent;
    if (parent != NULL)
    {
        if (parent->l_child == axis)
        {
            parent->l_child = right_child;
        }
        else
        {
            parent->r_child = right_child;
        }
    }
    right_child->size = axis->size;
    splay_update_size(axis);
}

inline splay_node *splay_route(splay_node *v)
{
    splay_node *parent, *grand_parent;
    while (((parent = v->parent) != NULL) && ((grand_parent = parent->parent) != NULL))
    {
        if (parent->l_child == v)
        {
            if (grand_parent->l_child == parent)
            {
                splay_zig(grand_parent);
                splay_zig(parent);
            }
            else
            {
                splay_zig(parent);
                splay_zag(grand_parent);
            }
        }
        else
        {
            if (grand_parent->r_child == parent)
            {
                splay_zag(grand_parent);
                splay_zag(parent);
            }
            else
            {
                splay_zag(parent);
                splay_zig(grand_parent);
            }
        }
    }
    if ((parent = v->parent) != NULL)
    {
        if (parent->l_child == v)
        {
            splay_zig(parent);
        }
        else
        {
            splay_zag(parent);
        }
    }
    return v;
}

inline splay_node *splay_bottom(splay_node *v = splay_head)
{
    while (v != NULL)
    {
        if (v->r_child == NULL)
        {
            return v;
        }
        else
        {
            v = v->r_child;
        }
    }
    return NULL;
}

inline splay_node *splay_top(splay_node *v = splay_head)
{
    while (v != NULL)
    {
        if (v->l_child == NULL)
        {
            return v;
        }
        else
        {
            v = v->l_child;
        }
    }
    return NULL;
}

inline void splay_init()
{
    splay_head = new_node();
    splay_head->l = 1;
    splay_head->r = n;
    splay_head->size = splay_head->r - splay_head->l + 1;
    interval_to_splay.insert(std::pair <int, int>(n, splay_head - recover_splay_node));
}

inline int get_num_by_id(const int id)
{
    tmp = id_to_num.find(id);
    if (tmp == id_to_num.end())
    {
        return id;
    }
    return tmp->second;
}

inline int get_id_by_num(const int num)
{
    tmp = num_to_id.find(num);
    if (tmp == num_to_id.end())
    {
        return num;
    }
    return tmp->second;
}

inline splay_node *get_splay_by_num(const int num)
{
    tmp = interval_to_splay.lower_bound(num);
    return recover_splay_node + tmp->second;
}

inline int splay_get_rank(const int id)
{
    int num = get_num_by_id(id);
    splay_head = splay_route(get_splay_by_num(num));
    return splay_size(splay_head->l_child) + num - splay_head->l + 1;
}

inline void map_change(int x, const int y)
{
    tmp = id_to_num.find(x);
    if (tmp != id_to_num.end())
    {
        x = tmp->second;
        id_to_num.erase(tmp);
        tmp = num_to_id.find(x);
        num_to_id.erase(tmp);
    }
    id_to_num.insert(std::pair <int, int>(y, x));
    num_to_id.insert(std::pair <int, int>(x, y));
}

inline int splay_move_to_top(const int id)
{
    int num = get_num_by_id(id);
    tmp = interval_to_splay.lower_bound(num);
    splay_head = splay_route(recover_splay_node + tmp->second);
    int ret = splay_size(splay_head->l_child) + num - splay_head->l + 1;
    splay_node *move_node = NULL;
    if (splay_head->l == splay_head->r)
    {
        move_node = splay_head;
        if (splay_head->l_child == NULL)
        {
            splay_head->r_child->parent = NULL;
            splay_head = splay_head->r_child;
        }
        else
        {
            splay_head->l_child->parent = NULL;
            splay_node *new_root = splay_route(splay_bottom(splay_head->l_child));
            attach_as_r_child(new_root, splay_head->r_child);
            splay_update_size(new_root);
            splay_head = new_root;
        }
        move_node->l_child = move_node->r_child = NULL;
        move_node->size = 1;
    }
    else if (splay_head->l == num)
    {
        move_node = new_node();
        move_node->l = move_node->r = num;
        move_node->size = 1;
        interval_to_splay.insert(std::pair <int, int>(num, move_node - recover_splay_node));
        splay_head->l = num + 1;
        splay_head->size--;
    }
    else if (splay_head->r == num)
    {
        move_node = new_node();
        move_node->l = move_node->r = num;
        move_node->size = 1;
        interval_to_splay.erase(tmp);
        interval_to_splay.insert(std::pair <int, int>(num - 1, splay_head - recover_splay_node));
        interval_to_splay.insert(std::pair <int, int>(num, move_node - recover_splay_node));
        splay_head->r = num - 1;
        splay_head->size--;
    }
    else
    {
        move_node = new_node();
        move_node->l = move_node->r = num;
        move_node->size = 1;
        interval_to_splay.erase(tmp);
        interval_to_splay.insert(std::pair <int, int>(num, move_node - recover_splay_node));
        splay_node *splite_right = new_node();
        splite_right->l = num + 1;
        splite_right->r = splay_head->r;
        interval_to_splay.insert(std::pair <int, int>(splite_right->r, splite_right - recover_splay_node));
        interval_to_splay.insert(std::pair <int, int>(num - 1, splay_head - recover_splay_node));
        splay_head->r = num - 1;
        splay_head->size--;
        attach_as_r_child(splite_right, splay_head->r_child);
        attach_as_r_child(splay_head, splite_right);
        splay_update_size(splite_right);
    }
    splay_head = splay_route(splay_top());
    attach_as_l_child(splay_head, move_node);
    splay_head->size++;
    return ret;
}

inline int splay_move_to_bottom(const int id)
{
    int num = get_num_by_id(id);
    tmp = interval_to_splay.lower_bound(num);
    splay_head = splay_route(recover_splay_node + tmp->second);
    int ret = splay_size(splay_head->l_child) + num - splay_head->l + 1;
    splay_node *move_node = NULL;
    if (splay_head->l == splay_head->r)
    {
        move_node = splay_head;
        if (splay_head->l_child == NULL)
        {
            splay_head->r_child->parent = NULL;
            splay_head = splay_head->r_child;
        }
        else
        {
            splay_head->l_child->parent = NULL;
            splay_node *new_root = splay_route(splay_bottom(splay_head->l_child));
            attach_as_r_child(new_root, splay_head->r_child);
            splay_update_size(new_root);
            splay_head = new_root;
        }
        move_node->l_child = move_node->r_child = NULL;
    }
    else if (splay_head->l == num)
    {
        move_node = new_node();
        move_node->l = move_node->r = num;
        move_node->size = 1;
        interval_to_splay.insert(std::pair <int, int>(num, move_node - recover_splay_node));
        splay_head->l = num + 1;
        splay_head->size--;
    }
    else if (splay_head->r == num)
    {
        move_node = new_node();
        move_node->l = move_node->r = num;
        move_node->size = 1;
        interval_to_splay.erase(tmp);
        interval_to_splay.insert(std::pair <int, int>(num - 1, splay_head - recover_splay_node));
        interval_to_splay.insert(std::pair <int, int>(num, move_node - recover_splay_node));
        splay_head->r = num - 1;
        splay_head->size--;
    }
    else
    {
        move_node = new_node();
        move_node->l = move_node->r = num;
        move_node->size = 1;
        interval_to_splay.erase(tmp);
        interval_to_splay.insert(std::pair <int, int>(num, move_node - recover_splay_node));
        splay_node *splite_right = new_node();
        splite_right->l = num + 1;
        splite_right->r = splay_head->r;
        interval_to_splay.insert(std::pair <int, int>(splite_right->r, splite_right - recover_splay_node));
        interval_to_splay.insert(std::pair <int, int>(num - 1, splay_head - recover_splay_node));
        splay_head->r = num - 1;
        splay_head->size--;
        attach_as_r_child(splite_right, splay_head->r_child);
        attach_as_r_child(splay_head, splite_right);
        splay_update_size(splite_right);
    }
    splay_head = splay_route(splay_bottom());
    attach_as_r_child(splay_head, move_node);
    splay_head->size++;
    return ret;
}

inline int splay_get_num(int k)
{
    splay_node *v = splay_head;
    int left_length;
    while (v != NULL)
    {
        left_length = splay_size(v->l_child);
        if (left_length < k)
        {
            if (left_length + v->r - v->l + 1 >= k)
            {
                return get_id_by_num(v->l + k - left_length - 1);
            }
            else
            {
                k -= left_length + v->r - v->l + 1;
                v = v->r_child;
            }
        }
        else
        {
            v = v->l_child;
        }
    }
    return -1;
}

int main()
{
#ifdef LOCAL_TIME
    long long start_time_ = clock();
#endif
#ifdef READ_FILE
    freopen("oj.in", "r", stdin);
#ifndef STD_DEBUG
    freopen("oj.out", "w", stdout);
#endif
#endif
    fread_init();
    recover_splay_init();
    read(n);
    read(m);
    splay_init();
    for (int i = 0; i < m; i++)
    {
#ifdef LOCAL_DEBUG
        if (i == 5)
        {
            printf("");
        }
#endif
        read(op);
        if (op == 1)
        {
            read(x);
            read(y);
            x -= a;
            y -= a;
            map_change(x, y);
            printf("%dn", a = splay_get_rank(y));
        }
        else if (op == 2)
        {
            read(x);
            x -= a;
            printf("%dn", a = splay_move_to_top(x));
        }
        else if (op == 3)
        {
            read(x);
            x -= a;
            printf("%dn", a = splay_move_to_bottom(x));
        }
        else if (op == 4)
        {
            read(k);
            k -= a;
            printf("%dn", a = splay_get_num(k));
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