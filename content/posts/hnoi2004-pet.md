---
title: '[BZOJ1208][HNOI2004]宠物收养所（treap）'
tags:
  - BZOJ
  - HNOI
  - TREAP
id: 289
categories:
  - OI
date: 2014-05-02 16:57:43
---

没什么好说的，一道不能更裸的TREAP，TREAP这随机种子真是蛋疼。。直接上代码了（原谅我的代码风格，这代码真是长。。）

<!--more-->

``` cpp
/**************************************************************
    Problem: 1208
    User: shenxn
    Language: C++
    Result: Accepted
    Time:92 ms
    Memory:2416 kb
****************************************************************/

//HNOI2004 DAY1 pet.cpp
#include<algorithm>
#include<iostream>
#include<cstring>
#include<cstdlib>
#include<string>
#include<cstdio>
#include<cmath>
#include<queue>
#include<stack>

//#define LOCAL
#define READ_FREAD
//#define READ_FILE
//#define VISUAL_STUDIO

#ifdef VISUAL_STUDIO
#pragma warning(disable:4996)
#endif

#ifdef LOCAL
#define LOCAL_TIME
#define LOCAL_DEBUG
//#define STD_DEBUG
#endif

#ifdef LOCAL_TIME
#include<ctime>
#endif

#ifndef NULL
#define NULL 0
#endif

#ifdef READ_FREAD
const int MAXS = 900 * 1024;

char fread_string[MAXS];
char *fread_point = fread_string;

inline int get_int()
{
    int ret = 0;
    while ((*fread_point < '0') || (*fread_point > '9'))
    {
        fread_point++;
    }
    while ((*fread_point >= '0') && (*fread_point <= '9'))
    {
        ret = ret * 10 + *(fread_point++) - '0';
    }
    return ret;
}
#endif

inline void read(int &read_int)
{
#ifdef READ_FREAD
    read_int = get_int();
#else
    scanf("%d", &read_int);
#endif
}

typedef long long ll;

int n, operator_num, special_num;
int total_pet, total_person;

const int OP_PET = 0, OP_PERSON = 1;

struct treap_node
{
    int val, rank;
    treap_node *l_child, *r_child, *parent;
    inline void init()
    {
        val = rank = 0;
        l_child = r_child = parent = NULL;
    }
    inline treap_node()
    {
        init();
    }
};

/*---recover memory---*/
const int MAXN = 10000;

treap_node node_memory[MAXN];
treap_node *memory_stack[MAXN];
int top_stack;

inline void init_memory()
{
    for (int i = 0; i < MAXN; i++)
    {
        memory_stack[i] = node_memory + i;
    }
    top_stack = MAXN;
}

inline treap_node *new_node()
{
    return memory_stack[--top_stack];
}

inline void delete_node(treap_node *del_node)
{
    memory_stack[top_stack++] = del_node;
    del_node->init();
}
/*---recover memory---*/

treap_node *head;

/*---rank---*/
int rank_sum;

const int rank_base = 854;

inline int get_rank(const int val)
{
    if (rank_sum == 0)
    {
        srand(val + rank_base + rand());
    }
    rank_sum++;
    if (rank_sum > 30)
    {
        rank_sum = 0;
    }
    return (rand() << 11) + rand();
}
/*---rank---*/

/*---treap---*/
inline void attach_as_l_child(treap_node *parent, treap_node *child)
{
    parent->l_child = child;
    if (child != NULL)
    {
        child->parent = parent;
    }
}

inline void attach_as_r_child(treap_node *parent, treap_node *child)
{
    parent->r_child = child;
    if (child != NULL)
    {
        child->parent = parent;
    }
}

inline void zig(treap_node *axis)
{
    treap_node *left_child = axis->l_child;
    treap_node *parent = axis->parent;
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
    else
    {
        head = left_child;
    }
}

inline void zag(treap_node *axis)
{
    treap_node *right_child = axis->r_child;
    treap_node *parent = axis->parent;
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
    else
    {
        head = right_child;
    }
}

inline void treap_route(treap_node *v)
{
    treap_node *parent;
    while (((parent = v->parent) != NULL) && (v->rank < parent->rank))
    {
        if (parent->l_child == v)
        {
            zig(parent);
        }
        else
        {
            zag(parent);
        }
    }
}

inline treap_node *cmp_s(treap_node *a, treap_node *b, const int val)
{
    if (b->val >= val)
    {
        return a;
    }
    if (a == NULL)
    {
        return b;
    }
    if (a->val < b->val)
    {
        return b;
    }
    return a;
}

inline treap_node *cmp_b(treap_node *a, treap_node *b, const int val)
{
    if (b->val <= val)
    {
        return a;
    }
    if (a == NULL)
    {
        return b;
    }
    if (a->val > b->val)
    {
        return b;
    }
    return a;
}

inline treap_node *treap_find(const int val, treap_node *(*cmp)(treap_node*, treap_node*, int) = NULL)
{
    treap_node *last_visit = head;
    treap_node *min_max = NULL;
    for (treap_node *tmp = head; tmp != NULL;)
    {
        if (tmp->val == val)
        {
            return tmp;
        }
        if (cmp != NULL)
        {
            min_max = cmp(min_max, tmp, val);
        }
        last_visit = tmp;
        if (tmp->val > val)
        {
            tmp = tmp->l_child;
        }
        else
        {
            tmp = tmp->r_child;
        }
    }
    if (cmp != NULL)
    {
        return min_max;
    }
    return last_visit;
}

inline void treap_insert(const int val)
{
    treap_node *insert_node = new_node();
    insert_node->val = val;
    insert_node->rank = get_rank(val);
    if (head == NULL)
    {
        head = insert_node;
        return;
    }
    treap_node *tmp = treap_find(val);
    if (tmp->val > val)
    {
        attach_as_l_child(tmp, insert_node);
    }
    else
    {
        attach_as_r_child(tmp, insert_node);
    }
    treap_route(insert_node);
}

inline void treap_remove(treap_node *remove_node)
{
    while ((remove_node->l_child != NULL) && (remove_node->r_child != NULL))
    {
        if (remove_node->l_child->val < remove_node->r_child->val)
        {
            zig(remove_node);
        }
        else
        {
            zag(remove_node);
        }
    }
    if (remove_node->parent == NULL)
    {
        if (remove_node->l_child != NULL)
        {
            head = remove_node->l_child;
            remove_node->l_child->parent = NULL;
        }
        else
        {
            head = remove_node->r_child;
            if (remove_node->r_child != NULL)
            {
                remove_node->r_child->parent = NULL;
            }
        }
    }
    else
    {
        if (remove_node->parent->l_child == remove_node)
        {
            if (remove_node->l_child != NULL)
            {
                attach_as_l_child(remove_node->parent, remove_node->l_child);
            }
            else
            {
                attach_as_l_child(remove_node->parent, remove_node->r_child);
            }
        }
        else
        {
            if (remove_node->l_child != NULL)
            {
                attach_as_r_child(remove_node->parent, remove_node->l_child);
            }
            else
            {
                attach_as_r_child(remove_node->parent, remove_node->r_child);
            }
        }
    }
    delete_node(remove_node);
}

inline int treap_delete(const int val)
{
    treap_node *s_node = treap_find(val, cmp_s);
    if ((s_node != NULL) && (s_node->val == val))
    {
        treap_remove(s_node);
        return val;
    }
    treap_node *b_node = treap_find(val, cmp_b);
    if (s_node == NULL)
    {
        int value = b_node->val;
        treap_remove(b_node);
        return value;
    }
    if (b_node == NULL)
    {
        int value = s_node->val;
        treap_remove(s_node);
        return value;
    }
    if (b_node->val - val == val - s_node->val)
    {
        int value = s_node->val;
        treap_remove(s_node);
        return value;
    }
    if (b_node->val - val < val - s_node->val)
    {
        int value = b_node->val;
        treap_remove(b_node);
        return value;
    }
    else
    {
        int value = s_node->val;
        treap_remove(s_node);
        return value;
    }
}
/*---treap---*/

int ans;
const int MOD = 1000000;

int main()
{
#ifdef LOCAL_TIME
    ll start_time_ = clock();
#endif
#ifdef READ_FILE
    freopen("pet.in", "r", stdin);
#ifndef STD_DEBUG
    freopen("pet.out", "w", stdout);
#endif
#endif
#ifdef READ_FREAD
    fread(fread_string, 1, MAXS, stdin);
#endif
    init_memory();
    read(n);
    for (int i = 0; i < n; i++)
    {
        read(operator_num);
        read(special_num);
#ifdef LOCAL_DEBUG
        printf("%d %dn", operator_num, special_num);
#endif
        if (operator_num == OP_PET)
        {
            if (total_person == 0)
            {
                total_pet++;
                treap_insert(special_num);
            }
            else
            {
                total_person--;
                ans += std::abs(treap_delete(special_num) - special_num);
                if (ans >= MOD)
                {
                    ans %= MOD;
                }
            }
        }
        else
        {
            if (total_pet == 0)
            {
                total_person++;
                treap_insert(special_num);
            }
            else
            {
                total_pet--;
                ans += std::abs(treap_delete(special_num) - special_num);
                if (ans >= MOD)
                {
                    ans %= MOD;
                }
            }
        }
    }
    printf("%d", ans);
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