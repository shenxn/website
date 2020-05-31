---
title: '[BZOJ1500][NOI2005]维修数列（SPLAY）'
tags:
  - BZOJ
  - NOI
  - SPLAY
id: 34
categories:
  - OI
date: 2014-05-02 19:34:56
---

好像已经写了好多SPLAY了，维修数列说是最重口味的SPLAY题目了，好吧我也调了将近一上午，主要是最大子段和的地方自己YY错了，少考虑了一种情况。。

<!--more-->

其他操作都很水，然后最大子段和就是要维护max_l, max_r, max_sum分别表示左起最大子段和、右起最大子段和、最大子段和。在叶节点上显然max_l = max_r = max_sum = value。UPDATE的公式是：
max_l = max(l_child-&gt;max_l, l_child-&gt;sum + value, l_child-&gt;sum + value + r_child-&gt;l_child);
max_r = max(r_child-&gt;max_r, r_child-&gt;sum + value,  r_child-&gt;sum + value + l_child-&gt;r_child);
max_sum = max(max_l, max_r, l_child-&gt;max_sum, r_child-&gt;max_sum, max(l_child-&gt;max_r, 0) + value + max(r_child-&gt;max_l, 0));

剩下的就是代码了（下面还有个暴力不要介意。。）

``` cpp
//NOI2005 DAY1 sequence.cpp splay
#include<algorithm>
#include<iostream>
#include<cstring>
#include<cstdio>
#include<string>
#include<cmath>
#include<queue>
#include<stack>

//#define VISUAL_STUDIO
#define READ_FREAD
//#define READ_FILE
//#define LOCAL

#ifdef VISUAL_STUDIO
#pragma warning(disable:4996)
#endif

#ifdef LOCAL
//#define LOCAL_DEBUG
#define LOCAL_TIME
//#define DEBUG_STD
#endif

#ifdef LOCAL_DEBUG
//#define HARD_WORK
#endif

#ifndef HARD_WORK

#ifdef LOCAL_TIME
#include<ctime>
#endif

#ifndef NULL
#define NULL 0
#endif

#ifdef READ_FREAD
const int MAXS = 13 * 1024 * 1024;

char fread_string[MAXS];
char *fread_point = fread_string;

inline int get_int()
{
    int ret = 0;
    int sign = 1;
    while ((*fread_point < '0') || (*fread_point > '9'))
    {
        sign = 1;
        if (*fread_point == '-')
        {
            sign = -1;
        }
        fread_point++;
    }
    while ((*fread_point >= '0') && (*fread_point <= '9'))
    {
        ret = ret * 10 + *(fread_point++) - '0';
    }
    return sign * ret;
}

inline void get_string(char *input_string)
{
#ifdef LOCAL_DEBUG
    memset(input_string, 0, 10);
#endif
    while ((*fread_point == ' ') || (*fread_point == 'n'))
    {
        fread_point++;
    }
    int length = 0;
    while ((*fread_point != ' ') && (*fread_point != 'n'))
    {
        input_string[length++] = *(fread_point++);
    }
}
#endif

inline void read(int *read_num)
{
#ifdef READ_FREAD
    *read_num = get_int();
#else
    scanf("%d", read_num);
#endif
}

inline void read(char *read_string)
{
#ifdef READ_FREAD
    get_string(read_string);
#else
    scanf("%s", read_string);
#endif
}

typedef long long ll;

struct splay_node
{
    int val;
    splay_node *parent, *l_child, *r_child;
    int sum, max_l, max_r, max_sum, size;
    bool down_swap, down_change;
    int change_num;
    inline void init()
    {
        val = sum = max_l = max_r = max_sum = size = change_num = 0;
        parent = l_child = r_child = NULL;
        down_swap = down_change = false;
    }
    inline splay_node()
    {
        init();
    }
    inline ~splay_node()
    {
    }
};

/*---recover memory---*/
const int MAXN = 500000 + 10;

splay_node node_memory[MAXN];
splay_node *memory_stack[MAXN];
int top_stack;

inline void init_memory()
{
    for (int i = 0; i < MAXN; i++)
    {
        memory_stack[i] = node_memory + i;
    }
    top_stack = MAXN;
}

inline splay_node *new_node()
{
    return memory_stack[--top_stack];
}

inline void delete_node(splay_node *del_node)
{
    memory_stack[top_stack++] = del_node;
    del_node->init();
}
/*---recover memory---*/

int n, m;
int insert_num[MAXN];

splay_node *head = NULL;

/*---splay---*/
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

inline void splay_update_max(splay_node *update_node)
{
    if ((update_node->l_child != NULL) && (update_node->r_child != NULL))
    {
        update_node->max_l = std::max(update_node->l_child->max_l, update_node->l_child->sum + update_node->val + (update_node->r_child->max_l > 0 ? update_node->r_child->max_l : 0));
        update_node->max_r = std::max(update_node->r_child->max_r, update_node->r_child->sum + update_node->val + (update_node->l_child->max_r > 0 ? update_node->l_child->max_r : 0));
        update_node->max_sum = std::max(update_node->max_l, update_node->max_r);
        update_node->max_sum = std::max(update_node->max_sum, update_node->l_child->max_sum);
        update_node->max_sum = std::max(update_node->max_sum, update_node->r_child->max_sum);
        update_node->max_sum = std::max(update_node->max_sum, (update_node->l_child->max_r > 0 ? update_node->l_child->max_r : 0) + update_node->val + (update_node->r_child->max_l > 0 ? update_node->r_child->max_l : 0));
    }
    else if (update_node->l_child != NULL)
    {
        update_node->max_l = std::max(update_node->l_child->max_l, update_node->l_child->sum + update_node->val);
        update_node->max_r = std::max(update_node->val, update_node->l_child->max_r + update_node->val);
        update_node->max_sum = std::max(update_node->max_l, update_node->max_r);
        update_node->max_sum = std::max(update_node->max_sum, update_node->l_child->max_sum);
        update_node->max_sum = std::max(update_node->max_sum, update_node->val);
    }
    else if (update_node->r_child != NULL)
    {
        update_node->max_l = std::max(update_node->val, update_node->r_child->max_l + update_node->val);
        update_node->max_r = std::max(update_node->r_child->max_r, update_node->r_child->sum + update_node->val);
        update_node->max_sum = std::max(update_node->max_l, update_node->max_r);
        update_node->max_sum = std::max(update_node->max_sum, update_node->r_child->max_sum);
        update_node->max_sum = std::max(update_node->max_sum, update_node->val);
    }
    else
    {
        update_node->max_l = update_node->max_r = update_node->max_sum = update_node->val;
    }
}

inline void zig(splay_node *axis)
{
    splay_node *left_child = axis->l_child;
    splay_node *parent = axis->parent;
    attach_as_l_child(axis, left_child->r_child);
    attach_as_r_child(left_child, axis);
    left_child->sum = axis->sum;
    axis->sum = (axis->l_child != NULL ? axis->l_child->sum : 0) + (axis->r_child != NULL ? axis->r_child->sum : 0) + axis->val;
    left_child->size = axis->size;
    axis->size = (axis->l_child != NULL ? axis->l_child->size : 0) + (axis->r_child != NULL ? axis->r_child->size : 0) + 1;
    splay_update_max(axis);
    splay_update_max(left_child);
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

inline void zag(splay_node *axis)
{
    splay_node *right_child = axis->r_child;
    splay_node *parent = axis->parent;
    attach_as_r_child(axis, right_child->l_child);
    attach_as_l_child(right_child, axis);
    right_child->sum = axis->sum;
    axis->sum = (axis->l_child != NULL ? axis->l_child->sum : 0) + (axis->r_child != NULL ? axis->r_child->sum : 0) + axis->val;
    right_child->size = axis->size;
    axis->size = (axis->l_child != NULL ? axis->l_child->size : 0) + (axis->r_child != NULL ? axis->r_child->size : 0) + 1;
    splay_update_max(axis);
    splay_update_max(right_child);
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

inline void zig_zig(splay_node *axis)
{
    zig(axis->l_child);
    zig(axis);
}

inline void zig_zag(splay_node *axis)
{
    zig(axis->r_child);
    zag(axis);
}

inline void zag_zag(splay_node *axis)
{
    zag(axis->r_child);
    zag(axis);
}

inline void zag_zig(splay_node *axis)
{
    zag(axis->l_child);
    zig(axis);
}

inline void splay_swap(splay_node *swap_node)
{
    if (swap_node == NULL)
    {
        return;
    }
    std::swap(swap_node->l_child, swap_node->r_child);
    std::swap(swap_node->max_l, swap_node->max_r);
    swap_node->down_swap = !swap_node->down_swap;
}

inline void splay_down_swap(splay_node *root)
{
    if (root->down_swap)
    {
        splay_swap(root->l_child);
        splay_swap(root->r_child);
        root->down_swap = !root->down_swap;
    }
}

inline void splay_change(splay_node *change_node, const int chan_num)
{
    if (change_node == NULL)
    {
        return;
    }
    change_node->val = chan_num;
    change_node->sum = change_node->size * chan_num;
    change_node->max_l = change_node->max_r = change_node->max_sum = (chan_num < 0 ? chan_num : change_node->sum);
    change_node->down_change = true;
    change_node->change_num = chan_num;
}

inline void splay_down_change(splay_node *root)
{
    if (root->down_change)
    {
        splay_change(root->l_child, root->change_num);
        splay_change(root->r_child, root->change_num);
        root->down_change = false;
    }
}

inline splay_node *splay_route(splay_node *v)
{
    splay_node *parent, *grand_parent;
    while (((parent = v->parent) != NULL) && ((grand_parent = parent->parent) != NULL))
    {
        splay_down_swap(grand_parent);
        splay_down_swap(parent);
        //splay_down_swap(v);
        splay_down_change(grand_parent);
        splay_down_change(parent);
        //splay_down_change(v);
        if (grand_parent->l_child == parent)
        {
            if (parent->l_child == v)
            {
                zig_zig(grand_parent);
            }
            else
            {
                zag_zig(grand_parent);
            }
        }
        else
        {
            if (parent->r_child == v)
            {
                zag_zag(grand_parent);
            }
            else
            {
                zig_zag(grand_parent);
            }
        }
    }
    if ((parent = v->parent) != NULL)
    {
        splay_down_swap(parent);
        //splay_down_swap(v);
        splay_down_change(parent);
        //splay_down_change(v);
        if (parent->l_child == v)
        {
            zig(parent);
        }
        else
        {
            zag(parent);
        }
    }
    //splay_down_swap(v);
    //splay_down_change(v);
    return v;
}

inline splay_node *splay_top(splay_node *root = head)
{
    for (splay_node *tmp = root; tmp != NULL; tmp = tmp->l_child)
    {
        splay_down_swap(tmp);
        splay_down_change(tmp);
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
        splay_down_swap(tmp);
        splay_down_change(tmp);
        if (tmp->r_child == NULL)
        {
            return tmp;
        }
    }
    return NULL;
}

inline splay_node *splay_find(const int length, splay_node *root = head)
{
    int size = 0;
    for (splay_node *tmp = root; tmp != NULL;)
    {
        splay_down_swap(tmp);
        splay_down_change(tmp);
        if (size + (tmp->l_child != NULL ? tmp->l_child->size : 0) + 1 == length)
        {
            return tmp;
        }
        if (size + (tmp->l_child != NULL ? tmp->l_child->size : 0) + 1 > length)
        {
            tmp = tmp->l_child;
        }
        else
        {
            size += (tmp->l_child != NULL ? tmp->l_child->size : 0) + 1;
            tmp = tmp->r_child;
        }
    }
    return NULL;
}

splay_node *splay_build(const int l, const int r)
{
    int mid = (l + r) >> 1;
    splay_node *tmp = new_node();
    tmp->val = insert_num[mid];
    if (l + 1 == r)
    {
        tmp->sum = tmp->max_l = tmp->max_r = tmp->max_sum = tmp->val;
        tmp->size = 1;
    }
    else
    {
        int sum = tmp->val, size = 1;
        if (l < mid)
        {
            attach_as_l_child(tmp, splay_build(l, mid));
            sum += tmp->l_child->sum;
            size += tmp->l_child->size;
        }
        if (mid + 1 < r)
        {
            attach_as_r_child(tmp, splay_build(mid + 1, r));
            sum += tmp->r_child->sum;
            size += tmp->r_child->size;
        }
        tmp->sum = sum;
        tmp->size = size;
        splay_update_max(tmp);
    }
    return tmp;
}

void splay_remove(splay_node *remove_node)
{
    if (remove_node->l_child != NULL)
    {
        splay_remove(remove_node->l_child);
    }
    if (remove_node->r_child != NULL)
    {
        splay_remove(remove_node->r_child);
    }
    delete_node(remove_node);
}

inline void splay_insert(const int cur, const int length)
{
    splay_node *root = splay_build(0, length);
    if (head == NULL)
    {
        head = root;
    }
    else
    {
        if (cur == 0)
        {
            head = splay_route(splay_top());
            attach_as_l_child(head, root);
            splay_update_max(head);
            head->size += root->size;
            head->sum += root->sum;
        }
        else
        {
            head = splay_route(splay_find(cur));
            if (head->r_child != NULL)
            {
                head->r_child->parent = NULL;
                splay_node *tmp = splay_route(splay_top(head->r_child));
                attach_as_r_child(head, tmp);
                attach_as_l_child(tmp, root);
                splay_update_max(tmp);
                tmp->size += root->size;
                tmp->sum += root->sum;
                splay_update_max(head);
                head->size += root->size;
                head->sum += root->sum;
            }
            else
            {
                attach_as_r_child(head, root);
                head->size += root->size;
                head->sum += root->sum;
            }
        }
    }
}

inline void splay_delete(const int cur, const int length)
{
    if (length == 0)
    {
        return;
    }
    if (cur == 1)
    {
        if (head->size > length)
        {
            head = splay_route(splay_find(length + 1));
            splay_remove(head->l_child);
            head->l_child = NULL;
            head->size = head->r_child->size + 1;
            head->sum = head->r_child->sum + head->val;
            splay_update_max(head);
        }
        else
        {
            splay_remove(head);
            head = NULL;
        }
    }
    else
    {
        head = splay_route(splay_find(cur - 1));
        if (head->r_child->size > length)
        {
            head->r_child->parent = NULL;
            splay_node *tmp = splay_route(splay_find(length + 1, head->r_child));
            attach_as_r_child(head, tmp);
            tmp->size -= tmp->l_child->size;
            tmp->sum -= tmp->l_child->sum;
            head->size -= tmp->l_child->size;
            head->sum -= tmp->l_child->sum;
            splay_remove(tmp->l_child);
            tmp->l_child = NULL;
            splay_update_max(tmp);
            splay_update_max(head);
        }
        else
        {
            head->size -= head->r_child->size;
            head->sum -= head->r_child->sum;
            splay_remove(head->r_child);
            head->r_child = NULL;
            splay_update_max(head);
        }
    }
}

inline void splay_make(const int cur, const int length, const int change_num)
{
    if (length == 0)
    {
        return;
    }
    if (cur == 1)
    {
        if (head->size > length)
        {
            head = splay_route(splay_find(length + 1));
            splay_change(head->l_child, change_num);
            head->sum = head->l_child->sum + head->val + (head->r_child != NULL ? head->r_child->sum : 0);
            splay_update_max(head);
        }
        else
        {
            splay_change(head, change_num);
        }
    }
    else
    {
        head = splay_route(splay_find(cur - 1));
        if (head->r_child->size > length)
        {
            head->r_child->parent = NULL;
            splay_node *tmp = splay_route(splay_find(length + 1, head->r_child));
            attach_as_r_child(head, tmp);
            splay_change(tmp->l_child, change_num);
            tmp->sum = tmp->l_child->sum + tmp->val + (tmp->r_child != NULL ? tmp->r_child->sum : 0);
            splay_update_max(tmp);
            head->sum = (head->l_child != NULL ? head->l_child->sum : 0) + head->val + tmp->sum;
            splay_update_max(head);
        }
        else
        {
            splay_change(head->r_child, change_num);
            head->sum = (head->l_child != NULL ? head->l_child->sum : 0) + head->val + head->r_child->sum;
            splay_update_max(head);
        }
    }
}

inline void splay_reverse(const int cur, const int length)
{
    if (length == 0)
    {
        return;
    }
    if (cur == 1)
    {
        if (head->size > length)
        {
            head = splay_route(splay_find(length + 1));
            splay_swap(head->l_child);
            splay_update_max(head);
        }
        else
        {
            splay_swap(head);
        }
    }
    else
    {
        head = splay_route(splay_find(cur - 1));
        if (head->r_child->size > length)
        {
            head->r_child->parent = NULL;
            splay_node *tmp = splay_route(splay_find(length + 1, head->r_child));
            attach_as_r_child(head, tmp);
            splay_swap(tmp->l_child);
            splay_update_max(tmp);
            splay_update_max(head);
        }
        else
        {
            splay_swap(head->r_child);
            splay_update_max(head);
        }
    }
}

inline int splay_sum(const int cur, const int length)
{
    if (length == 0)
    {
        return 0;
    }
    if (cur == 1)
    {
        if (head->size > length)
        {
            head = splay_route(splay_find(length + 1));
            return head->l_child->sum;
        }
        else
        {
            return head->sum;
        }
    }
    else
    {
        head = splay_route(splay_find(cur - 1));
        if (head->r_child->size > length)
        {
            head->r_child->parent = NULL;
            splay_node *tmp = splay_route(splay_find(length + 1, head->r_child));
            attach_as_r_child(head, tmp);
            return tmp->l_child->sum;
        }
        else
        {
            return head->r_child->sum;
        }
    }
}

inline int splay_max()
{
    return head->max_sum;
}

#ifdef LOCAL_DEBUG
void splay_check(splay_node *root = head, const bool down_change = false, const int change_num = 0, const bool down_swap = false)
{
    if (root == NULL)
    {
        return;
    }
    int size = 1, sum = down_change ? change_num : root->val;
    if ((root->l_child != NULL) && (!down_swap))
    {
        if (root->l_child->parent != root)
        {
            printf("ERROR ");
        }
        splay_check(root->l_child, root->down_change || down_change, down_change ? change_num : root->change_num, down_swap ^ root->down_swap);
        size += root->l_child->size;
        if (!root->down_change)
        {
            sum += root->l_child->sum;
        }
        else
        {
            sum += root->l_child->size * root->change_num;
        }
    }
    if ((root->r_child != NULL) && down_swap)
    {
        if (root->r_child->parent != root)
        {
            printf("ERROR ");
        }
        splay_check(root->r_child, root->down_change || down_change, down_change ? change_num : root->change_num, down_swap ^ root->down_swap);
        size += root->r_child->size;
        if (!root->down_change)
        {
            sum += root->r_child->sum;
        }
        else
        {
            sum += root->r_child->size * root->change_num;
        }
    }
    printf("%d ", down_change ? change_num : root->val);
    if ((root->r_child != NULL) && (!down_swap))
    {
        if (root->r_child->parent != root)
        {
            printf("ERROR ");
        }
        splay_check(root->r_child, root->down_change || down_change, down_change ? change_num : root->change_num, down_swap ^ root->down_swap);
        size += root->r_child->size;
        if (!root->down_change)
        {
            sum += root->r_child->sum;
        }
        else
        {
            sum += root->r_child->size * root->change_num;
        }
    }
    if ((root->l_child != NULL) && down_swap)
    {
        if (root->l_child->parent != root)
        {
            printf("ERROR ");
            exit(0);
        }
        splay_check(root->l_child, root->down_change || down_change, down_change ? change_num : root->change_num, down_swap ^ root->down_swap);
        size += root->l_child->size;
        if (!root->down_change)
        {
            sum += root->l_child->sum;
        }
        else
        {
            sum += root->l_child->size * root->change_num;
        }
    }
    if ((!down_change) && (sum != root->sum))
    {
        printf("ERROR_SUM ");
        exit(0);
    }
    if (size != root->size)
    {
        printf("ERROR_SIZE ");
        exit(0);
    }
}
#endif
/*---splay---*/

char operator_string[10];
int cur, length, change_num;

#ifdef LOCAL_DEBUG
int sum = 0;
#endif

int main()
{
#ifdef LOCAL_TIME
    ll start_time_ = clock();
#endif
#ifdef READ_FILE
    freopen("sequence.in", "r", stdin);
#ifndef DEBUG_STD
    freopen("sequence.out", "w", stdout);
#endif
#endif
#ifdef READ_FREAD
    fread(fread_string, 1, MAXS, stdin);
#endif
    init_memory();
    read(&n);
    read(&m);
    for (int i = 0; i < n; i++)
    {
        read(insert_num + i);
    }
    splay_insert(0, n);
#ifdef LOCAL_DEBUG
    splay_check();
    sum = n;
    printf("n");
#endif
    for (int i = 0; i < m; i++)
    {
        read(operator_string);
#ifdef LOCAL_DEBUG
        if (i == 31)
        {
            printf("DEBUGn");
        }
        printf("%d SUM:%d %s ", i, sum, operator_string);
        if (sum != head->size)
        {
            printf("nERROR_SUMn");
            exit(0);
        }
#endif
        if (operator_string[2] == 'S')
        {
            read(&cur);
            read(&length);
            for (int j = 0; j < length; j++)
            {
                read(insert_num + j);
            }
#ifdef LOCAL_DEBUG
            printf("%d %dn", cur, length);
            sum += length;
#endif
            splay_insert(cur, length);
        }
        else if (operator_string[2] == 'L')
        {
            read(&cur);
            read(&length);
#ifdef LOCAL_DEBUG
            printf("%d %dn", cur, length);
            sum -= length;
#endif
            splay_delete(cur, length);
        }
        else if (operator_string[2] == 'K')
        {
            read(&cur);
            read(&length);
            read(&change_num);
#ifdef LOCAL_DEBUG
            printf("%d %d %dn", cur, length, change_num);
#endif
            splay_make(cur, length, change_num);
        }
        else if (operator_string[2] == 'V')
        {
            read(&cur);
            read(&length);
#ifdef LOCAL_DEBUG
            printf("%d %dn", cur, length);
#endif
            splay_reverse(cur, length);
        }
        else if (operator_string[2] == 'T')
        {
            read(&cur);
            read(&length);
#ifdef LOCAL_DEBUG
            printf("%d %dn", cur, length);
#endif
            printf("%dn", splay_sum(cur, length));
        }
        else if (operator_string[2] == 'X')
        {
#ifdef LOCAL_DEBUG
            printf("n");
#endif
            printf("%dn", splay_max());
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
#ifndef DEBUG_STD
    fclose(stdout);
#endif
#endif
    return 0;
}

#else

int n, m;
int num_line[1000], tmp[1000];
int length, cur, len, k;
char operator_string[10];

inline void printline()
{
    for (int i = 1; i <= length; i++)
    {
        printf("%d ", num_line[i]);
    }
    printf("n");
}

int main()
{
    freopen("sequence2.in", "r", stdin);
    freopen("std.out", "w", stdout);
    scanf("%d %d", &n, &m);
    length = n;
    for (int i = 1; i <= length; i++)
    {
        scanf("%d", num_line + i);
    }
    printline();
    for (int i = 0; i < m; i++)
    {
        scanf("%s", operator_string);
        printf("%d SUM:%d %s ", i, length, operator_string);
        if (operator_string[2] == 'S')
        {
            scanf("%d %d", &cur, &len);
            printf("%d %dn", cur, len);
            length += len;
            for (int j = length; j > cur + len; j--)
            {
                num_line[j] = num_line[j - len];
            }
            for (int j = cur + 1; j <= cur + len; j++)
            {
                scanf("%d", num_line + j);
            }
        }
        else if (operator_string[2] == 'L')
        {
            scanf("%d %d", &cur, &len);
            printf("%d %dn", cur, len);
            length -= len;
            for (int j = cur; j <= length; j++)
            {
                num_line[j] = num_line[j + len];
            }
        }
        else if (operator_string[2] == 'K')
        {
            scanf("%d %d %d", &cur, &len, &k);
            printf("%d %d %dn", cur, len, k);
            for (int j = cur; j < cur + len; j++)
            {
                num_line[j] = k;
            }
        }
        else if (operator_string[2] == 'V')
        {
            scanf("%d %d", &cur, &len);
            printf("%d %dn", cur, len);
            for (int j = cur, k = 1; j < cur + len; j++, k++)
            {
                tmp[k] = num_line[j];
            }
            for (int j = cur, k = len; j < cur + len; j++, k--)
            {
                num_line[j] = tmp[k];
            }
        }
        else if (operator_string[2] == 'T')
        {
            scanf("%d %d", &cur, &len);
            printf("%d %dn", cur, len);
            int sum = 0;
            for (int j = cur; j < cur + len; j++)
            {
                sum += num_line[j];
            }
            printf("%dn", sum);
        }
        else if (operator_string[2] == 'X')
        {
            printf("n");
            printf("0n");
        }
        printline();
    }
}

#endif
```