---
title: '[BZOJ1207][HNOI2005]虚拟内存（HASH + SPLAY）'
tags:
  - BZOJ
  - HASH
  - HNOI
  - SPLAY
id: 155
categories:
  - OI
date: 2014-05-17 16:15:33
---

我原本妄图做一道HASH乱搞题，本以为这道题可以HASH + 优先队列，后来发现好像不行，然后又蛋疼地敲平衡树了。一开始敲了个FANHQ_TREAP，结果被卡，一个点退化了，只好改SPLAY。虽然不是很快，不过完全不知道这道题开50S时限是什么心态。

<!--more-->

这题就用一个SPLAY根据题目的优先级维护内存中的所有点，再开HASH将内存中的内存编号映射到SPLAY的节点上。细节上再处理一下就好了。

``` cpp
//HNOI2005 DAY2 memory.cpp
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

const int MAXN = 10000;
const int MAX_HASH_LIST = 100000;
const int HASH_BASE = 7;

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

int n, m, k, ans;
int splay_size;

struct memory_node
{
    int val;
    memory_node *next;
    int times, add_time;
    memory_node *parent, *l_child, *r_child;

    inline memory_node *init()
    {
        next = NULL;
        parent = l_child = r_child = NULL;
        times = 0;
        return this;
    }

    inline bool operator < (memory_node &b) const
    {
        if (times != b.times)
        {
            return times < b.times;
        }
        return add_time < b.add_time;
    }
};

memory_node recover_memory_node[MAXN];
memory_node *stack_memory_node[MAXN];
int top_stack_memory_node = MAXN;

inline void init_recover_memory()
{
    for (int i = 0; i < MAXN; i++)
    {
        stack_memory_node[i] = recover_memory_node + i;
        //stack_memory_node[i]->rank = get_rank();
    }
}

inline memory_node *new_node()
{
    return stack_memory_node[--top_stack_memory_node]->init();
}

inline void delete_node(memory_node *del_node)
{
    stack_memory_node[top_stack_memory_node++] = del_node;
}

memory_node *hash_list[MAX_HASH_LIST];

char hash_tmp[11];
inline int hash(const int k)
{
    int ret = 0;
    sprintf(hash_tmp, "%d", k);
    int length = strlen(hash_tmp);
    for (int i = 0; i < length; i++)
    {
        ret = ret * HASH_BASE + hash_tmp[i] - '0';
        if (ret >= MAX_HASH_LIST)
        {
            ret %= MAX_HASH_LIST;
        }
    }
    return ret;
}

inline memory_node *hash_find(const int k)
{
    memory_node *tmp = hash_list[hash(k)];
    while (tmp != NULL)
    {
        if (tmp->val == k)
        {
            tmp->times++;
            return tmp;
        }
        tmp = tmp->next;
    }
    return NULL;
}

inline memory_node *hash_insert(const int k, const int i = -1)
{
    memory_node *new_pt = new_node();
    new_pt->val = k;
    if (i != -1)
    {
        new_pt->add_time = i;
    }
    int hash_tmp = hash(k);
    new_pt->next = hash_list[hash_tmp];
    hash_list[hash_tmp] = new_pt;
    return new_pt;
}

inline void hash_delete(memory_node *del_node)
{
    int tmp_hash = hash(del_node->val);
    if (hash_list[tmp_hash] == del_node)
    {
        hash_list[tmp_hash] = del_node->next;
    }
    else
    {
        memory_node *tmp = hash_list[tmp_hash];
        while (tmp->next != del_node)
        {
            tmp = tmp->next;
        }
        tmp->next = del_node->next;
    }
    delete_node(del_node);
}

memory_node *splay_head = NULL;

inline void attach_as_l_child(memory_node *parent, memory_node *child)
{
    parent->l_child = child;
    if (child != NULL)
    {
        child->parent = parent;
    }
}

inline void attach_as_r_child(memory_node *parent, memory_node *child)
{
    parent->r_child = child;
    if (child != NULL)
    {
        child->parent = parent;
    }
}

inline void splay_zig(memory_node *axis)
{
    memory_node *left_child = axis->l_child;
    memory_node *parent = axis->parent;
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
}

inline void splay_zag(memory_node *axis)
{
    memory_node *right_child = axis->r_child;
    memory_node *parent = axis->parent;
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
}

inline memory_node *splay_route(memory_node *v)
{
    memory_node *parent, *grand_parent;
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

inline void splay_insert(memory_node *insert_node)
{
    if (splay_head == NULL)
    {
        splay_head = insert_node;
    }
    else
    {
        memory_node *tmp = splay_head;
        memory_node *last_visit = NULL;
        while (tmp != NULL)
        {
            last_visit = tmp;
            if (*tmp < *insert_node)
            {
                tmp = tmp->r_child;
            }
            else
            {
                tmp = tmp->l_child;
            }
        }
        if (*last_visit < *insert_node)
        {
            attach_as_r_child(last_visit, insert_node);
        }
        else
        {
            attach_as_l_child(last_visit, insert_node);
        }
        splay_head = splay_route(insert_node);
    }
}

inline memory_node *splay_top()
{
    for (memory_node *tmp = splay_head; tmp != NULL; tmp = tmp->l_child)
    {
        if (tmp->l_child == NULL)
        {
            return tmp;
        }
    }
    return NULL;
}

inline memory_node *splay_bottom(memory_node *root = splay_head)
{
    for (memory_node *tmp = root; tmp != NULL; tmp = tmp->r_child)
    {
        if (tmp->r_child == NULL)
        {
            return tmp;
        }
    }
    return NULL;
}

inline void splay_delete(memory_node *del_node)
{
    splay_head = splay_route(del_node);
    if (splay_head->l_child == NULL)
    {
        splay_head = splay_head->r_child;
        if (splay_head != NULL)
        {
            splay_head->parent = NULL;
        }
    }
    else
    {
        memory_node *left_root = splay_route(splay_bottom(splay_head->l_child));
        attach_as_r_child(left_root, splay_head->r_child);
        splay_head = left_root;
    }
    del_node->l_child = del_node->r_child = del_node->parent = NULL;
}

inline void memory_read(const int k, const int i)
{
    memory_node *tmp = hash_find(k);
    if (tmp == NULL)
    {
        if (splay_size < n)
        {
            tmp = hash_insert(k, i);
            splay_insert(tmp);
            splay_size++;
        }
        else
        {
            tmp = splay_top();
            splay_delete(tmp);
            hash_delete(tmp);
            tmp = hash_insert(k, i);
            splay_insert(tmp);
        }
    }
    else
    {
        ans++;
        splay_delete(tmp);
        tmp->times++;
        splay_insert(tmp);
    }
}

#ifdef LOCAL_DEBUG
int max_deepen;
int splay_checker(const int deepen, memory_node *root = splay_head)
{
    if (root == NULL)
    {
        return 0;
    }
    max_deepen = std::max(max_deepen, deepen);
    int ret = 1;
    ret += splay_checker(deepen + 1, root->l_child);
    ret += splay_checker(deepen + 1, root->r_child);
    return ret;
}
#endif

int main()
{
#ifdef LOCAL_TIME
    long long start_time_ = clock();
#endif
#ifdef READ_FILE
    freopen("memory.in", "r", stdin);
#ifndef STD_DEBUG
    freopen("memory.out", "w", stdout);
#endif
#endif
#ifdef READ_FREAD
    fread_init();
#endif
    init_recover_memory();
    read(n);
    read(m);
    for (int i = 0; i < m; i++)
    {
        read(k);
#ifdef LOCAL_DEBUG
        printf("%d %dn", i, k);
        if (i == 2)
        {
            printf("");
        }
#endif
        memory_read(k, i);
#ifdef LOCAL_DEBUG
        max_deepen = -1;
        if (splay_checker(1) != splay_size)
        {
            printf("SIZE_ERRORn");
        }
        //printf("%dn", max_deepen);
#endif
    }
    printf("%d", ans);
#ifdef LOCAL_TIME
    printf("nrun time: %lld ms", clock() - start_time_);
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