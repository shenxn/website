---
title: '[BZOJ1056][HAOI2008]排名系统（SPLAY + TRIE / HASH）'
tags:
  - BZOJ
  - HAOI
  - HASH
  - SPLAY
  - TRIE
id: 66
categories:
  - OI
date: 2014-05-15 16:55:41
---

只是一道平衡树水题，其实TREAP就可以了，只是有个操作我觉得还是SPLAY写起来舒服。一开始用的TRIE维护名字，后来又用HASH写了，稍微快了一点点。。

<!--more-->

TRIE版：

``` cpp
//HAOI2008 rank.cpp
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

//#define LOCAL
#define READ_FREAD
//#define READ_FILE
//#define VISUAL_STUDIO

#ifdef LOCAL
#define LOCAL_TIME
//#define LOCAL_DEBUG
//#define STD_DEBUG
#endif

const int MAXN = 250010;
const int MAXTRIE = 500000;

#ifdef VISUAL_STUDIO
#pragma warning(disable: 4996)
#endif

#ifdef LOCAL_TIME
#include<ctime>
#endif

#ifdef READ_FREAD
char fread_point;

inline int get_int()
{
    int ret = 0;
    while ((fread_point < '0') || (fread_point > '9'))
    {
        fread_point = getchar();
    }
    while ((fread_point >= '0') && (fread_point <= '9'))
    {
        ret = ret * 10 + fread_point - '0';
        fread_point = getchar();
    }
    return ret;
}

inline char get_char()
{
    while ((fread_point == ' ') || (fread_point == 'n'))
    {
        fread_point = getchar();
    }
    char ret = fread_point;
    fread_point = getchar();
    return ret;
}

inline void get_string(char *input)
{
    memset(input, 0, 12);
    while ((fread_point == ' ') || (fread_point == 'n'))
    {
        fread_point = getchar();
    }
    int loop = 0;
    while ((fread_point != ' ') && (fread_point != 'n') && (fread_point != EOF))
    {
        input[loop++] = fread_point;
        fread_point = getchar();
    }
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

inline void read(char *a)
{
#ifdef READ_FREAD
    get_string(a);
#else
    scanf("%s", a);
#endif
}

typedef long long ll;

inline int string_to_int(char *a)
{
    int ret = 0;
    for (int i = 0; a[i] != 0; i++)
    {
        ret = ret * 10 + a[i] - '0';
    }
    return ret;
}

struct splay_node
{
    int val;
    char name[12];
    splay_node *parent, *l_child, *r_child;
    int size;
    inline splay_node *init()
    {
        parent = l_child = r_child = NULL;
        size = 1;
        return this;
    }
};

splay_node recover_splay[MAXN];
splay_node *stack_splay[MAXN];
int top_stack_splay = MAXN;

inline void init_splay()
{
    for (int i = 0; i < MAXN; i++)
    {
        stack_splay[i] = recover_splay + i;
    }
}

inline splay_node *new_node()
{
    return stack_splay[--top_stack_splay]->init();
}

inline void delete_node(splay_node *del_node)
{
    stack_splay[top_stack_splay++] = del_node;
}

splay_node *head_splay;

inline int splay_size(splay_node *query_node)
{
    if (query_node == NULL)
    {
        return 0;
    }
    return query_node->size;
}

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

inline void zig(splay_node *axis)
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
    axis->size = splay_size(axis->l_child) + splay_size(axis->r_child) + 1;
}

inline void zag(splay_node *axis)
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
    axis->size = splay_size(axis->l_child) + splay_size(axis->r_child) + 1;
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
                zig(grand_parent);
                zig(parent);
            }
            else
            {
                zig(parent);
                zag(grand_parent);
            }
        }
        else
        {
            if (grand_parent->r_child == parent)
            {
                zag(grand_parent);
                zag(parent);
            }
            else
            {
                zag(parent);
                zig(grand_parent);
            }
        }
    }
    if ((parent = v->parent) != NULL)
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
    return v;
}

inline splay_node *splay_find(const int val, const int add_size = 0)
{
    splay_node *last_visit = NULL;
    for (splay_node *tmp = head_splay; tmp != NULL;)
    {
        last_visit = tmp;
        tmp->size += add_size;
        if (tmp->val >= val)
        {
            tmp = tmp->r_child;
        }
        else
        {
            tmp = tmp->l_child;
        }
    }
    return last_visit;
}

inline splay_node *splay_find_by_rank(splay_node *v, const int k)
{
    int rank = 0;
    int left_size;
    while (v != NULL)
    {
        left_size = splay_size(v->l_child) + 1;
        if (rank + left_size == k)
        {
            return v;
        }
        else if (rank + left_size > k)
        {
            v = v->l_child;
        }
        else
        {
            rank += left_size;
            v = v->r_child;
        }
    }
    return NULL;
}

inline splay_node *splay_last(splay_node *v)
{
    while (v->r_child != NULL)
    {
        v = v->r_child;
    }
    return v;
}

void splay_print(splay_node *root)
{
    if (root == NULL)
    {
        return;
    }
    splay_print(root->l_child);
    printf(" %s", root->name);
    splay_print(root->r_child);
}

inline splay_node *splay_insert(const int val, char *name, splay_node *new_pt = NULL)
{
    if (new_pt == NULL)
    {
        new_pt = new_node();
        for (int i = 0; i < 10; i++)
        {
            new_pt->name[i] = name[i];
        }
    }
    else
    {
        new_pt->init();
    }
    new_pt->val = val;
    if (head_splay == NULL)
    {
        head_splay = new_pt;
    }
    else
    {
        splay_node *insert_pt = splay_find(val, 1);
        if (insert_pt->val >= val)
        {
            attach_as_r_child(insert_pt, new_pt);
        }
        else
        {
            attach_as_l_child(insert_pt, new_pt);
        }
        head_splay = splay_route(new_pt);
    }
    return new_pt;
}

inline void splay_delete(splay_node *del_node)
{
    splay_route(del_node);
    if (del_node->l_child != NULL)
    {
        del_node->l_child->parent = NULL;
    }
    if (del_node->r_child != NULL)
    {
        del_node->r_child->parent = NULL;
    }
    if (del_node->l_child == NULL)
    {
        head_splay = del_node->r_child;
    }
    else
    {
        head_splay = splay_route(splay_last(del_node->l_child));
        attach_as_r_child(head_splay, del_node->r_child);
        head_splay->size += splay_size(del_node->r_child);
    }
}

inline void splay_change(splay_node *change_node, const int val)
{
    splay_delete(change_node);
    splay_insert(val, NULL, change_node);
}

inline void splay_query_users(const int k)
{
    head_splay = splay_route(splay_find_by_rank(head_splay, k));
    printf("%s", head_splay->name);
    if (splay_size(head_splay->r_child) <= 9)
    {
        splay_print(head_splay->r_child);
    }
    else
    {
        head_splay->r_child->parent = NULL;
        attach_as_r_child(head_splay, splay_route(splay_find_by_rank(head_splay->r_child, 10)));
        splay_print(head_splay->r_child->l_child);
    }
    printf("n");
}

inline int splay_query_rank(splay_node *v)
{
    int ret = splay_size(v->l_child) + 1;
    while (v->parent != NULL)
    {
        if (v->parent->r_child == v)
        {
            ret += splay_size(v->parent->l_child) + 1;
        }
        v = v->parent;
    }
    return ret;
}

struct trie_node
{
    trie_node *next[26];
    splay_node *val;
    inline trie_node *init()
    {
        for (int i = 0; i < 26; i++)
        {
            next[i] = NULL;
        }
        val = NULL;
        return this;
    }
};

trie_node recover_trie[MAXTRIE];
trie_node *stack_trie[MAXTRIE];
int top_stack_trie = MAXTRIE;

inline void init_trie()
{
    for (int i = 0; i < MAXTRIE; i++)
    {
        stack_trie[i] = recover_trie + i;
    }
}

inline trie_node *new_trie()
{
    return stack_trie[--top_stack_trie]->init();
}

inline void delete_trie(trie_node *del_node)
{
    stack_trie[top_stack_trie++] = del_node;
}

trie_node *head_trie;

inline splay_node *trie_find(char *find_string)
{
    trie_node *tmp = head_trie;
    for (int i = 0; find_string[i] != 0; i++)
    {
        if (tmp->next[find_string[i] - 'A'] == NULL)
        {
            return NULL;
        }
        else
        {
            tmp = tmp->next[find_string[i] - 'A'];
        }
    }
    return tmp->val;
}

inline void trie_insert(char *insert_string, splay_node *val)
{
    trie_node *tmp = head_trie;
    for (int i = 0; insert_string[i] != 0; i++)
    {
        if (tmp->next[insert_string[i] - 'A'] == NULL)
        {
            tmp->next[insert_string[i] - 'A'] = new_trie();
        }
        tmp = tmp->next[insert_string[i] - 'A'];
    }
    tmp->val = val;
}

int n;
char op_char;
char op_name[12];
int k;
splay_node *found;

int main()
{
#ifdef LOCAL_TIME
    ll start_time_ = clock();
#endif
#ifdef READ_FILE
    freopen("rank.in", "r", stdin);
#ifndef STD_DEBUG
    freopen("rank.out", "w", stdout);
#endif
#endif
    fread_point = getchar();
    read(n);
    init_splay();
    init_trie();
    head_trie = new_trie();
    for (int i = 0; i < n; i++)
    {
        read(op_char);
        read(op_name);
#ifdef LOCAL_DEBUG
        printf("%d %c %s ", i, op_char, op_name);
#endif
        if (op_char == '+')
        {
            read(k);
#ifdef LOCAL_DEBUG
            printf("%dn", k);
#endif
            if ((found = trie_find(op_name)) != NULL)
            {
                splay_change(found, k);
            }
            else
            {
                trie_insert(op_name, splay_insert(k, op_name));
            }
        }
        else if (op_char == '?')
        {
#ifdef LOCAL_DEBUG
            printf("n");
#endif
            if ((op_name[0] >= '0') && (op_name[0] <= '9'))
            {
                k = string_to_int(op_name);
                splay_query_users(k);
            }
            else
            {
                printf("%dn", splay_query_rank(trie_find(op_name)));
            }
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

HASH版：

``` cpp
**************************************************************
    Problem: 1056
    User: shenxn
    Language: C++
    Result: Accepted
    Time:1276 ms
    Memory:16320 kb
****************************************************************/

//HAOI2008 rank.cpp
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

//#define LOCAL
#define READ_FREAD
//#define READ_FILE
//#define VISUAL_STUDIO

#ifdef LOCAL
#define LOCAL_TIME
//#define LOCAL_DEBUG
//#define STD_DEBUG
#endif

const int MAXN = 250010;
const int MAXTRIE = 500000;
const int MAXHASH = 250000;
const int MAXHASHLIST = 100000;

#ifdef VISUAL_STUDIO
#pragma warning(disable: 4996)
#endif

#ifdef LOCAL_TIME
#include<ctime>
#endif

#ifdef READ_FREAD
char fread_point;

inline int get_int()
{
    int ret = 0;
    while ((fread_point < '0') || (fread_point > '9'))
    {
        fread_point = getchar();
    }
    while ((fread_point >= '0') && (fread_point <= '9'))
    {
        ret = ret * 10 + fread_point - '0';
        fread_point = getchar();
    }
    return ret;
}

inline char get_char()
{
    while ((fread_point == ' ') || (fread_point == 'n'))
    {
        fread_point = getchar();
    }
    char ret = fread_point;
    fread_point = getchar();
    return ret;
}

inline void get_string(char *input)
{
    memset(input, 0, 12);
    while ((fread_point == ' ') || (fread_point == 'n'))
    {
        fread_point = getchar();
    }
    int loop = 0;
    while ((fread_point != ' ') && (fread_point != 'n') && (fread_point != EOF))
    {
        input[loop++] = fread_point;
        fread_point = getchar();
    }
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

inline void read(char *a)
{
#ifdef READ_FREAD
    get_string(a);
#else
    scanf("%s", a);
#endif
}

typedef long long ll;

inline int string_to_int(char *a)
{
    int ret = 0;
    for (int i = 0; a[i] != 0; i++)
    {
        ret = ret * 10 + a[i] - '0';
    }
    return ret;
}

struct splay_node
{
    int val;
    char name[12];
    splay_node *parent, *l_child, *r_child;
    int size;
    inline splay_node *init()
    {
        parent = l_child = r_child = NULL;
        size = 1;
        return this;
    }
};

splay_node recover_splay[MAXN];
splay_node *stack_splay[MAXN];
int top_stack_splay = MAXN;

inline void init_splay()
{
    for (int i = 0; i < MAXN; i++)
    {
        stack_splay[i] = recover_splay + i;
    }
}

inline splay_node *new_node()
{
    return stack_splay[--top_stack_splay]->init();
}

inline void delete_node(splay_node *del_node)
{
    stack_splay[top_stack_splay++] = del_node;
}

splay_node *head_splay;

inline int splay_size(splay_node *query_node)
{
    if (query_node == NULL)
    {
        return 0;
    }
    return query_node->size;
}

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

inline void zig(splay_node *axis)
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
    axis->size = splay_size(axis->l_child) + splay_size(axis->r_child) + 1;
}

inline void zag(splay_node *axis)
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
    axis->size = splay_size(axis->l_child) + splay_size(axis->r_child) + 1;
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
                zig(grand_parent);
                zig(parent);
            }
            else
            {
                zig(parent);
                zag(grand_parent);
            }
        }
        else
        {
            if (grand_parent->r_child == parent)
            {
                zag(grand_parent);
                zag(parent);
            }
            else
            {
                zag(parent);
                zig(grand_parent);
            }
        }
    }
    if ((parent = v->parent) != NULL)
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
    return v;
}

inline splay_node *splay_find(const int val, const int add_size = 0)
{
    splay_node *last_visit = NULL;
    for (splay_node *tmp = head_splay; tmp != NULL;)
    {
        last_visit = tmp;
        tmp->size += add_size;
        if (tmp->val >= val)
        {
            tmp = tmp->r_child;
        }
        else
        {
            tmp = tmp->l_child;
        }
    }
    return last_visit;
}

inline splay_node *splay_find_by_rank(splay_node *v, const int k)
{
    int rank = 0;
    int left_size;
    while (v != NULL)
    {
        left_size = splay_size(v->l_child) + 1;
        if (rank + left_size == k)
        {
            return v;
        }
        else if (rank + left_size > k)
        {
            v = v->l_child;
        }
        else
        {
            rank += left_size;
            v = v->r_child;
        }
    }
    return NULL;
}

inline splay_node *splay_last(splay_node *v)
{
    while (v->r_child != NULL)
    {
        v = v->r_child;
    }
    return v;
}

void splay_print(splay_node *root)
{
    if (root == NULL)
    {
        return;
    }
    splay_print(root->l_child);
    printf(" %s", root->name);
    splay_print(root->r_child);
}

inline splay_node *splay_insert(const int val, char *name, splay_node *new_pt = NULL)
{
    if (new_pt == NULL)
    {
        new_pt = new_node();
        for (int i = 0; i < 10; i++)
        {
            new_pt->name[i] = name[i];
        }
    }
    else
    {
        new_pt->init();
    }
    new_pt->val = val;
    if (head_splay == NULL)
    {
        head_splay = new_pt;
    }
    else
    {
        splay_node *insert_pt = splay_find(val, 1);
        if (insert_pt->val >= val)
        {
            attach_as_r_child(insert_pt, new_pt);
        }
        else
        {
            attach_as_l_child(insert_pt, new_pt);
        }
        head_splay = splay_route(new_pt);
    }
    return new_pt;
}

inline void splay_delete(splay_node *del_node)
{
    splay_route(del_node);
    if (del_node->l_child != NULL)
    {
        del_node->l_child->parent = NULL;
    }
    if (del_node->r_child != NULL)
    {
        del_node->r_child->parent = NULL;
    }
    if (del_node->l_child == NULL)
    {
        head_splay = del_node->r_child;
    }
    else
    {
        head_splay = splay_route(splay_last(del_node->l_child));
        attach_as_r_child(head_splay, del_node->r_child);
        head_splay->size += splay_size(del_node->r_child);
    }
}

inline void splay_change(splay_node *change_node, const int val)
{
    splay_delete(change_node);
    splay_insert(val, NULL, change_node);
}

inline void splay_query_users(const int k)
{
    head_splay = splay_route(splay_find_by_rank(head_splay, k));
    printf("%s", head_splay->name);
    if (splay_size(head_splay->r_child) <= 9)
    {
        splay_print(head_splay->r_child);
    }
    else
    {
        head_splay->r_child->parent = NULL;
        attach_as_r_child(head_splay, splay_route(splay_find_by_rank(head_splay->r_child, 10)));
        splay_print(head_splay->r_child->l_child);
    }
    printf("n");
}

inline int splay_query_rank(splay_node *v)
{
    int ret = splay_size(v->l_child) + 1;
    while (v->parent != NULL)
    {
        if (v->parent->r_child == v)
        {
            ret += splay_size(v->parent->l_child) + 1;
        }
        v = v->parent;
    }
    return ret;
}

/*
struct trie_node
{
    trie_node *next[26];
    splay_node *val;
    inline trie_node *init()
    {
        for (int i = 0; i < 26; i++)
        {
            next[i] = NULL;
        }
        val = NULL;
        return this;
    }
};

trie_node recover_trie[MAXTRIE];
trie_node *stack_trie[MAXTRIE];
int top_stack_trie = MAXTRIE;

inline void init_trie()
{
    for (int i = 0; i < MAXTRIE; i++)
    {
        stack_trie[i] = recover_trie + i;
    }
}

inline trie_node *new_trie()
{
    return stack_trie[--top_stack_trie]->init();
}

inline void delete_trie(trie_node *del_node)
{
    stack_trie[top_stack_trie++] = del_node;
}

trie_node *head_trie;

inline splay_node *trie_find(char *find_string)
{
    trie_node *tmp = head_trie;
    for (int i = 0; find_string[i] != 0; i++)
    {
        if (tmp->next[find_string[i] - 'A'] == NULL)
        {
            return NULL;
        }
        else
        {
            tmp = tmp->next[find_string[i] - 'A'];
        }
    }
    return tmp->val;
}

inline void trie_insert(char *insert_string, splay_node *val)
{
    trie_node *tmp = head_trie;
    for (int i = 0; insert_string[i] != 0; i++)
    {
        if (tmp->next[insert_string[i] - 'A'] == NULL)
        {
            tmp->next[insert_string[i] - 'A'] = new_trie();
        }
        tmp = tmp->next[insert_string[i] - 'A'];
    }
    tmp->val = val;
}
*/

struct hash_node
{
    splay_node *val;
    char name[12];
    hash_node *next;
    inline hash_node *init()
    {
        next = NULL;
        memset(name, 0, 12);
        return this;
    }
};

hash_node recover_hash[MAXHASH];
hash_node *stack_hash[MAXHASH];
int top_stack_hash = MAXHASH;

inline void init_hash()
{
    for (int i = 0; i < MAXHASH; i++)
    {
        stack_hash[i] = recover_hash + i;
    }
}

inline hash_node *new_hash()
{
    return stack_hash[--top_stack_hash]->init();
}

inline void delete_hash(hash_node *del_node)
{
    stack_hash[top_stack_hash++] = del_node;
}

hash_node *hash_list[MAXHASHLIST];

inline int hash(char *hash_string)
{
    int ret = 0;
    for (int i = 0; hash_string[i] != 0; i++)
    {
        ret = ret * 37 + hash_string[i] - 'A';
        if (ret >= MAXHASHLIST)
        {
            ret %= MAXHASHLIST;
        }
    }
    return ret;
}

inline bool equal(char *a, char *b)
{
    for (int i = 0; (a[i] != 0) || (b[i] != 0); i++)
    {
        if (a[i] != b[i])
        {
            return false;
        }
    }
    return true;
}

inline splay_node *hash_get(char *get_string)
{
    hash_node *tmp = hash_list[hash(get_string)];
    while (tmp != NULL)
    {
        if (equal(get_string, tmp->name))
        {
            return tmp->val;
        }
        tmp = tmp->next;
    }
    return NULL;
}

inline void hash_insert(char *insert_string, splay_node *val)
{
    hash_node *tmp = hash_list[hash(insert_string)];
    hash_node *new_pt = new_hash();
    new_pt->val = val;
    for (int i = 0; i < 10; i++)
    {
        new_pt->name[i] = insert_string[i];
    }
    if (tmp == NULL)
    {
        hash_list[hash(insert_string)] = new_pt;
    }
    else
    {
        while (tmp->next != NULL)
        {
            tmp = tmp->next;
        }
        tmp->next = new_pt;
    }
}

int n;
char op_char;
char op_name[12];
int k;
splay_node *found;

int main()
{
#ifdef LOCAL_TIME
    ll start_time_ = clock();
#endif
#ifdef READ_FILE
    freopen("rank.in", "r", stdin);
#ifndef STD_DEBUG
    freopen("rank.out", "w", stdout);
#endif
#endif
    fread_point = getchar();
    read(n);
    init_splay();
    init_hash();
    /*
    init_trie();
    head_trie = new_trie();
    */
    for (int i = 0; i < n; i++)
    {
        read(op_char);
        read(op_name);
#ifdef LOCAL_DEBUG
        printf("%d %c %s ", i, op_char, op_name);
#endif
        if (op_char == '+')
        {
            read(k);
#ifdef LOCAL_DEBUG
            printf("%dn", k);
#endif
            if ((found = /*trie_find(op_name)*/ hash_get(op_name)) != NULL)
            {
                splay_change(found, k);
            }
            else
            {
                hash_insert(op_name, splay_insert(k, op_name));
                /*
                trie_insert(op_name, splay_insert(k, op_name));
                */
            }
        }
        else if (op_char == '?')
        {
#ifdef LOCAL_DEBUG
            printf("n");
#endif
            if ((op_name[0] >= '0') && (op_name[0] <= '9'))
            {
                k = string_to_int(op_name);
                splay_query_users(k);
            }
            else
            {
                printf("%dn", splay_query_rank(/*trie_find(op_name)*/ hash_get(op_name)));
            }
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