---
title: '[BZOJ1507][NOI2003]文本编辑器（SPLAY）'
tags:
  - BZOJ
  - NOI
  - SPLAY
id: 27
categories:
  - OI
date: 2014-05-02 19:16:08
---

这题我一开始用块链写的，后来也许是memcpy上的问题本地AC了八中上死活A不掉，后来也就没去改。。几天后学了SPLAY，那就用SPLAY水掉了。

然后是代码（这个不是蛋疼模板了，我重写的SPLAY）

<!--more-->

``` cpp
//NOI2003 DAY1 editor.cpp splay
#include<algorithm>
#include<iostream>
#include<cstring>
#include<cstdio>
#include<string>
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
//#define LOCAL_DEBUG
//#define STD_DEBUG
#endif

#ifdef LOCAL_TIME
#include<ctime>
#endif

#ifdef READ_FREAD
const int MAXS = 6 * 1024 * 1024;
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

inline void get_string(char *input_string)
{
#ifdef LOCAL_DEBUG
    memset(input_string, 0, 7);
#endif
    while ((*fread_point == ' ') || (*fread_point == 'n'))
    {
        fread_point++;
    }
    int leng = 0;
    while ((*fread_point != ' ') && (*fread_point != 'n'))
    {
        input_string[leng++] = *(fread_point++);
    }
}

inline void get_insert(char *input_string, const int length)
{
    for (int i = 0; i < length; i++)
    {
        while (*fread_point == 'n')
        {
            fread_point++;
        }
        input_string[i] = *(fread_point++);
    }
}

#endif

typedef long long ll;

const int MAXN = 2 * 1024 * 1024 + 10;

/*---recover memory---*/
struct splay_node
{
    char val;
    splay_node *parent, *l_child, *r_child;
    int size;
    void init()
    {
        parent = l_child = r_child = NULL;
    }
    splay_node()
    {
        init();
    }
    ~splay_node()
    {
    }
};

splay_node splay_memory[MAXN];
splay_node *memory_stack[MAXN];
int stack_top;

void init_memory()
{
    for (int i = 0; i < MAXN; i++)
    {
        memory_stack[i] = splay_memory + i;
    }
    stack_top = MAXN;
}

inline splay_node *new_node()
{
    return memory_stack[--stack_top];
}

inline void delete_node(splay_node *del_node)
{
    memory_stack[stack_top++] = del_node;
    del_node->init();
}
/*---recover memory---*/

char insert_string[MAXN];

splay_node *head = NULL, *cur = NULL;

/*--splay---*/
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

inline splay_node *build_splay(const int l, const int r, int *ret_size)
{
    int mid = (l + r) >> 1;
    splay_node *tmp = new_node();
    tmp->val = insert_string[mid];
    *ret_size += 1;
    if (l + 1 == r)
    {
        tmp->size = 1;
    }
    else
    {
        int size = 0;
        if (l < mid)
        {
            attach_as_l_child(tmp, build_splay(l, mid, &size));
        }
        if (mid + 1 < r)
        {
            attach_as_r_child(tmp, tmp->r_child = build_splay(mid + 1, r, &size));
        }
        tmp->size = size + 1;
        *ret_size += size;
    }
    return tmp;
}

inline splay_node *splay_top(splay_node *root = head)
{
    for (splay_node *tmp = root; tmp != NULL; tmp = tmp->l_child)
    {
        if (tmp->l_child == NULL)
        {
            return tmp;
        }
    }
    return NULL;
}

inline splay_node *splay_bottom(splay_node *root = head)
{
    for (splay_node *tmp = root; tmp != NULL; tmp = tmp->r_child)
    {
        if (tmp->r_child == NULL)
        {
            return tmp;
        }
    }
    return NULL;
}

inline void zig(splay_node *axis)
{
    splay_node *left_child = axis->l_child;
    splay_node *parent = axis->parent;
    left_child->size = axis->size;
    axis->size = (left_child->r_child != NULL ? left_child->r_child->size : 0) + (axis->r_child != NULL ? axis->r_child->size : 0) + 1;
    attach_as_l_child(axis, left_child->r_child);
    attach_as_r_child(left_child, axis);
    left_child->parent = parent;
    if (parent != NULL)
    {
        if (parent->l_child == axis)
        {
            attach_as_l_child(parent, left_child);
        }
        else
        {
            attach_as_r_child(parent, left_child);
        }
    }
}

inline void zag(splay_node *axis)
{
    splay_node *right_child = axis->r_child;
    splay_node *parent = axis->parent;
    right_child->size = axis->size;
    axis->size = (right_child->l_child != NULL ? right_child->l_child->size : 0) + (axis->l_child != NULL ? axis->l_child->size : 0) + 1;
    attach_as_r_child(axis, right_child->l_child);
    attach_as_l_child(right_child, axis);
    right_child->parent = parent;
    if (parent != NULL)
    {
        if (parent->l_child == axis)
        {
            attach_as_l_child(parent, right_child);
        }
        else
        {
            attach_as_r_child(parent, right_child);
        }
    }
}

inline void zig_zig(splay_node *axis)
{
    zig(axis->l_child);
    zig(axis);
}

inline void zag_zag(splay_node *axis)
{
    zag(axis->r_child);
    zag(axis);
}

inline void zig_zag(splay_node *axis)
{
    zig(axis->r_child);
    zag(axis);
}

inline void zag_zig(splay_node *axis)
{
    zag(axis->l_child);
    zig(axis);
}

inline splay_node *splay_route(splay_node *v)
{
    splay_node *parent = NULL, *grandparent = NULL;
    while (((parent = v->parent) != NULL) && ((grandparent = parent->parent) != NULL))
    {
        if (parent->l_child == v)
        {
            if (grandparent->l_child == parent)
            {
                zig_zig(grandparent);
            }
            else
            {
                zig_zag(grandparent);
            }
        }
        else
        {
            if (grandparent->r_child == parent)
            {
                zag_zag(grandparent);
            }
            else
            {
                zag_zig(grandparent);
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

inline void splay_insert(const int length)
{
    get_insert(insert_string, length);
    int size = 0;
    splay_node *root = build_splay(0, length, &size);
    if (head == NULL)
    {
        head = root;
    }
    else
    {
        if (cur == NULL)
        {
            head = splay_route(splay_top());
            attach_as_l_child(head, root);
            head->size += root->size;
        }
        else
        {
            head = splay_route(cur);
            if (head->r_child != NULL)
            {
                head->r_child->parent = NULL;
                splay_node *tmp = splay_route(splay_top(head->r_child));
                attach_as_r_child(head, tmp);
                attach_as_l_child(tmp, root);
                tmp->size += root->size;
                head->size += root->size;
            }
            else
            {
                attach_as_r_child(head, root);
                head->size += root->size;
            }
        }
    }
}

inline splay_node *splay_move(const int move_id, splay_node *root = head)
{
    if (move_id == 0)
    {
        return NULL;
    }
    int total_size = 0;
    for (splay_node *tmp = root; tmp != NULL;)
    {
        if (total_size + (tmp->l_child != NULL ? tmp->l_child->size : 0) + 1 == move_id)
        {
            return tmp;
            break;
        }
        if (total_size + (tmp->l_child != NULL ? tmp->l_child->size : 0) + 1 < move_id)
        {
            total_size += (tmp->l_child != NULL ? tmp->l_child->size : 0) + 1;
            tmp = tmp->r_child;
        }
        else
        {
            tmp = tmp->l_child;
        }
    }
    return NULL;
}

void splay_remove(splay_node * remove_root)
{
    if (remove_root == NULL)
    {
        return;
    }
    if (remove_root->l_child != NULL)
    {
        splay_remove(remove_root->l_child);
    }
    if (remove_root->r_child != NULL)
    {
        splay_remove(remove_root->r_child);
    }
    delete_node(remove_root);
}

inline void splay_delete(const int length)
{
    if (cur != NULL)
    {
        head = splay_route(cur);
        if (head->r_child->size > length)
        {
            head->r_child->parent = NULL;
            splay_node *tmp = splay_route(splay_move(length + 1, head->r_child));
            attach_as_r_child(head, tmp);
            tmp->size -= tmp->l_child->size;
            head->size -= tmp->l_child->size;
            tmp->l_child = NULL;
            splay_remove(tmp->l_child);
        }
        else
        {
            head->size -= length;
            splay_remove(head->r_child);
            head->r_child = NULL;
        }
    }
    else
    {
        if (head->size > length)
        {
            head = splay_route(splay_move(length + 1));
            head->size -= length;
            splay_remove(head->l_child);
            head->l_child = NULL;
        }
        else
        {
            splay_remove(head);
            head = NULL;
        }
    }
}

void splay_print(splay_node *root)
{
    if (root == NULL)
    {
        return;
    }
    if (root->l_child != NULL)
    {
        splay_print(root->l_child);
    }
    printf("%c", root->val);
    if (root->r_child != NULL)
    {
        splay_print(root->r_child);
    }
}

inline void splay_get(const int length)
{
    if (cur != NULL)
    {
        head = splay_route(cur);
        if (head->r_child->size > length)
        {
            head->r_child->parent = NULL;
            splay_node *tmp = splay_route(splay_move(length + 1, head->r_child));
            attach_as_r_child(head, tmp);
            splay_print(tmp->l_child);
        }
        else
        {
            splay_print(head->r_child);
        }
    }
    else
    {
        if (head->size > length)
        {
            head = splay_route(splay_move(length + 1));
            splay_print(head->l_child);
        }
        else
        {
            splay_print(head);
        }
    }
    printf("n");
}

inline void splay_prev()
{
    head = splay_route(cur);
    cur = splay_bottom(head->l_child);
}

inline void splay_next()
{
    if (cur == NULL)
    {
        cur = head = splay_route(splay_top());
    }
    else
    {
        head = splay_route(cur);
        cur = splay_top(head->r_child);
    }
}

#ifdef LOCAL_DEBUG
void splay_check(splay_node *parent = NULL)
{
    if (parent == NULL)
    {
        parent = head;
    }
    if (parent == NULL)
    {
        return;
    }
    int size = 1;
    if (parent->l_child != NULL)
    {
        if (parent->l_child->parent != parent)
        {
            printf("ERROR ");
        }
        size += parent->l_child->size;
        splay_check(parent->l_child);
    }
    printf("%c", parent->val);
    if (parent->r_child != NULL)
    {
        if (parent->r_child->parent != parent)
        {
            printf("ERROR ");
        }
        size += parent->r_child->size;
        splay_check(parent->r_child);
    }
    if (size != parent->size)
    {
        printf("ERROR_SIZE ");
    }
}
#endif
/*---splay---*/

int n;
char operator_string[7];

int main()
{
#ifdef LOCAL_TIME
    ll start_time_ = clock();
#endif
#ifdef READ_FILE
    freopen("editor.in", "r", stdin);
#ifndef STD_DEBUG
    freopen("editor.out", "w", stdout);
#endif
#endif
    fread(fread_string, 1, MAXS, stdin);
    init_memory();
    n = get_int();
    for (int i = 0; i < n; i++)
    {
        get_string(operator_string);
#ifdef LOCAL_DEBUG
        printf("%sn", operator_string);
#endif
        if (operator_string[0] == 'I')
        {
            int m = get_int();
            splay_insert(m);
        }
        else if (operator_string[0] == 'M')
        {
            int k = get_int();
            cur = splay_move(k);
        }
        else if (operator_string[0] == 'D')
        {
            int m = get_int();
            splay_delete(m);
        }
        else if (operator_string[0] == 'G')
        {
            int m = get_int();
            splay_get(m);
        }
        else if (operator_string[0] == 'P')
        {
            splay_prev();
        }
        else if (operator_string[0] == 'N')
        {
            splay_next();
        }
#ifdef LOCAL_DEBUG
        splay_check();
        printf("n");
#endif
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