---
title: '[BZOJ1861][ZJOI2006]书架（SPLAY）'
tags:
  - BZOJ
  - SPLAY
  - ZJOI
id: 19
categories:
  - OI
date: 2014-05-02 18:24:46
---

听说标程是树状数组，听说树状数组跑得可快了。。好吧拿这题来学SPLAY的确有点做死不过也还算裸

SPLAY要维护的东西很清楚，就是要单独开个数组指向SPLAY上每个节点，以方便根据编号找节点。

<!--more-->

然后就是贴代码了，一如既往的长
``` cpp
//test splay ADT
#include<algorithm>
#include<iostream>
#include<cstring>
#include<cstdio>
#include<string>
#include<cmath>
#include<queue>
#include<stack>

#define MAXN 80010

//#define LOCAL
//#define VISUAL_STUDIO
//#define READ_FILE

#ifdef LOCAL
#define LOCAL_TIME
#define LOCAL_DEBUG
//#define DEBUG_STD
#endif

#ifdef VISUAL_STUDIO
#pragma warning(disable:4996)
#endif

#ifdef LOCAL_TIME
#include<ctime>
#endif

typedef long long ll;

int n, m, id[MAXN], k;
char operation_string[7];

//ADT splay

#ifndef NULL
#define NULL 0
#endif

template <typename SPLAY_VAL>
class splay_node
{
public:
    SPLAY_VAL _value;
    splay_node *_parent, *_l_child, *_r_child;
    int _size;
    void init()
    {
        _parent = _l_child = _r_child = NULL;
        _size = 0;
    }
    splay_node()
    {
        init();
    }

    ~splay_node()
    {
        _parent = _l_child = _r_child = NULL;
    }
};

splay_node<int> _mem_node[MAXN], *_mem_stack[MAXN];
splay_node<int> *_id[MAXN];
int _stack_top;

template <typename SPLAY_VAL>
class splay
{
private:
    splay_node<int> *_head;
    void _init_mem_stack()
    {
        for (int i = 0; i < MAXN; i++)
        {
            _mem_stack[i] = _mem_node + i;
        }
        _stack_top = MAXN;
    }
    splay_node<int> *_new_node()
    {
        return _mem_stack[--_stack_top];
    }
    void _delete_node(splay_node<int> *delete_node)
    {
        _mem_stack[_stack_top++] = delete_node;
        delete_node->init();
    }
    void attach_as_l_child(splay_node<int> *parent, splay_node<int> *child)
    {
        parent->_l_child = child;
        if (child != NULL)
        {
            child->_parent = parent;
        }
    }
    void attach_as_r_child(splay_node<int> *parent, splay_node<int> *child)
    {
        parent->_r_child = child;
        if (child != NULL)
        {
            child->_parent = parent;
        }
    }
    void zig(splay_node<int> *axis)
    {
        splay_node<int> *left_child = axis->_l_child;
        splay_node<int> *parent = axis->_parent;
        left_child->_size = axis->_size;
        axis->_size = (axis->_r_child != NULL ? axis->_r_child->_size : 0) + (left_child->_r_child != NULL ? left_child->_r_child->_size : 0) + 1;
        attach_as_l_child(axis, left_child->_r_child);
        attach_as_r_child(left_child, axis);
        if (parent == NULL)
        {
            left_child->_parent = NULL;
        }
        else
        {
            if (parent->_l_child == axis)
            {
                attach_as_l_child(parent, left_child);
            }
            else
            {
                attach_as_r_child(parent, left_child);
            }
        }
    }
    void zag(splay_node<int> *axis)
    {
        splay_node<int> *right_child = axis->_r_child;
        splay_node<int> *parent = axis->_parent;
        right_child->_size = axis->_size;
        axis->_size = (axis->_l_child != NULL ? axis->_l_child->_size : 0) + (right_child->_l_child != NULL ? right_child->_l_child->_size : 0) + 1;
        attach_as_r_child(axis, right_child->_l_child);
        attach_as_l_child(right_child, axis);
        if (parent == NULL)
        {
            right_child->_parent = NULL;
        }
        else
        {
            if (parent->_l_child == axis)
            {
                attach_as_l_child(parent, right_child);
            }
            else
            {
                attach_as_r_child(parent, right_child);
            }
        }
    }
    void zig_zig(splay_node<int> *axis)
    {
        zig(axis->_l_child);
        zig(axis);
    }
    void zag_zag(splay_node<int> *axis)
    {
        zag(axis->_r_child);
        zag(axis);
    }
    void zig_zag(splay_node<int> *axis)
    {
        zig(axis->_r_child);
        zag(axis);
    }
    void zag_zig(splay_node<int> *axis)
    {
        zag(axis->_l_child);
        zig(axis);
    }
    splay_node<int> *find_top(splay_node<int> *root = NULL)
    {
        for (splay_node<int> *tmp = (root != NULL ? root : _head); tmp != NULL; tmp = tmp->_l_child)
        {
            if (tmp->_l_child == NULL)
            {
                return tmp;
            }
        }
        return NULL;
    }
    splay_node<int> *find_bottom(splay_node<int> *root = NULL)
    {
        for (splay_node<int> *tmp = (root != NULL ? root : _head); tmp != NULL; tmp = tmp->_r_child)
        {
            if (tmp->_r_child == NULL)
            {
                return tmp;
            }
        }
        return NULL;
    }
    splay_node<int> *splay_route(splay_node<int> *v)
    {
        if (v == NULL)
        {
            return NULL;
        }
        splay_node<int> *parent = NULL, *grandparent = NULL;
        while (((parent = v->_parent) != NULL) && ((grandparent = parent->_parent) != NULL))
        {
            if (parent->_l_child == v)
            {
                if (grandparent->_l_child == parent)
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
                if (grandparent->_r_child == parent)
                {
                    zag_zag(grandparent);
                }
                else
                {
                    zag_zig(grandparent);
                }
            }
        }
        if ((parent = v->_parent) != NULL)
        {
            if (parent->_l_child == v)
            {
                zig(parent);
            }
            else
            {
                zag(parent);
            }
        }
        v->_parent = NULL;
        return v;
    }
    splay_node<int> *search_node(SPLAY_VAL value, splay_node<int> *root = NULL)
    {
        if (_head == NULL)
        {
            return NULL;
        }
        splay_node<int> *tmp = (root != NULL ? root : _head);
        splay_node<int> *last_visit = tmp;
        while ((tmp != NULL) && ((tmp->_value < value) || (value < tmp->_value)))
        {
            last_visit = tmp;
            if (tmp->_value < value)
            {
                tmp = tmp->_r_child;
            }
            else
            {
                tmp = tmp->_l_child;
            }
        }
        if (tmp == NULL)
        {
            tmp = last_visit;
        }
        return last_visit;
    }
    splay_node<int> *init_build(const int l, const int r, int* return_size)
    {
        int mid = (l + r) >> 1;
        int size = 1;
        splay_node<int> *tmp = _new_node();
        tmp->_value = id[mid];
        _id[id[mid]] = tmp;
        if (l + 1 == r)
        {
            *return_size += 1;
            tmp->_size = 1;
        }
        else
        {
            if (l < mid)
            {
                attach_as_l_child(tmp, init_build(l, mid, &size));
            }
            if (mid + 1 < r)
            {
                attach_as_r_child(tmp, init_build(mid + 1, r, &size));
            }
            *return_size += size;
            tmp->_size = size;
        }
        return tmp;
    }
    splay_node<int> *splay_remove(splay_node<int> *remove_node)
    {
        for (splay_node<int> *loop = remove_node->_parent; loop != NULL; loop = loop->_parent)
        {
            loop->_size--;
        }
        if (remove_node->_l_child == NULL)
        {
            if (remove_node->_parent == NULL)
            {
                _head = remove_node->_r_child;
                if (_head != NULL)
                {
                    _head->_parent = NULL;
                }
            }
            else
            {
                if (remove_node->_parent->_l_child == remove_node)
                {
                    attach_as_l_child(remove_node->_parent, remove_node->_r_child);
                }
                else
                {
                    attach_as_r_child(remove_node->_parent, remove_node->_r_child);
                }
            }
        }
        else
        {
            remove_node->_l_child->_parent = NULL;
            splay_node<int> *tmp = splay_route(find_bottom(remove_node->_l_child));
            tmp->_size += (remove_node->_r_child != NULL ? remove_node->_r_child->_size : 0);
            attach_as_r_child(tmp, remove_node->_r_child);
            if (remove_node->_parent == NULL)
            {
                _head = tmp;
                tmp->_parent = NULL;
            }
            else
            {
                if (remove_node->_parent->_l_child == remove_node)
                {
                    attach_as_l_child(remove_node->_parent, tmp);
                }
                else
                {
                    attach_as_r_child(remove_node->_parent, tmp);
                }
            }
        }
        remove_node->_l_child = remove_node->_r_child = NULL;
        remove_node->_size = 1;
        return remove_node;
    }
    void splay_swap(splay_node<int> *swap_first, splay_node<int> *swap_second)
    {
        _id[swap_first->_value] = swap_second;
        _id[swap_second->_value] = swap_first;
        int tmp = swap_first->_value;
        swap_first->_value = swap_second->_value;
        swap_second->_value = tmp;
    }
public:
    splay()
    {
        _head = NULL;
        _init_mem_stack();
    }
    ~splay()
    {
        _head = NULL;
    }
    void splay_build()
    {
        int size = 0;
        _head = init_build(0, n, &size);
        _head->_size = size;
    }
    int splay_query(const int id)
    {
        int size = 0;
        splay_node<int> *tmp = _head;
        for (; tmp != NULL;)
        {
            if (size + (tmp->_l_child != NULL ? tmp->_l_child->_size : 0) + 1 == id)
            {
                return tmp->_value;
            }
            if (size + (tmp->_l_child != NULL ? tmp->_l_child->_size : 0) >= id)
            {
                tmp = tmp->_l_child;
            }
            else
            {
                size += (tmp->_l_child != NULL ? tmp->_l_child->_size : 0) + 1;
                tmp = tmp->_r_child;
            }
        }
        if (tmp != NULL)
        {
            _head = splay_route(tmp);
        }
        return 0;
    }
    void splay_top(const int id)
    {
        splay_node<int> *tmp = splay_remove(_id[id]);
        splay_node<int> *top = find_top();
        attach_as_l_child(top, tmp);
        for (splay_node<int> *loop = top; loop != NULL; loop = loop->_parent)
        {
            loop->_size++;
        }
        _head = splay_route(tmp);
    }
    void splay_bottom(const int id)
    {
        splay_node<int> *tmp = splay_remove(_id[id]);
        splay_node<int> *bottom = find_bottom();
        attach_as_r_child(bottom, tmp);
        for (splay_node<int> *loop = bottom; loop != NULL; loop = loop->_parent)
        {
            loop->_size++;
        }
        _head = splay_route(tmp);
    }
    void splay_move(const int id, const int shift)
    {
        if (shift == 0)
        {
            return;
        }
        splay_node<int> *tmp = _id[id];
        _head = splay_route(tmp);
        if (shift == 1)
        {
            splay_swap(tmp, find_top(tmp->_r_child));
        }
        else
        {
            splay_swap(tmp, find_bottom(tmp->_l_child));
        }
    }
    int splay_ask(const int id)
    {
        splay_node<int> *tmp = splay_route(_id[id]);
        _head = tmp;
        return (tmp->_l_child == NULL ? 0 : tmp->_l_child->_size);
    }
#ifdef LOCAL_DEBUG
    void debug_size()
    {
        printf("HEAD_SIZE: %d n", _head->_size);
    }
    void splay_check(splay_node<int> *parent = NULL)
    {
        if (parent == NULL)
        {
            parent = _head;
        }
        if ((parent->_l_child != NULL) && (parent->_l_child->_parent != parent))
        {
            printf("ERROR");
        }
        if (parent->_l_child != NULL)
        {
            splay_check(parent->_l_child);
        }
        printf("%d ", parent->_value);
        if ((parent->_r_child != NULL) && (parent->_r_child->_parent != parent))
        {
            printf("ERROR");
        }
        if (parent->_r_child != NULL)
        {
            splay_check(parent->_r_child);
        }
        if ((parent->_l_child != NULL ? parent->_l_child->_size : 0) + (parent->_r_child != NULL ? parent->_r_child->_size : 0) + 1 != parent->_size)
        {
            printf("SIZE_ERROR");
            exit(0);
        }
    }
#endif
};

int main()
{
#ifdef LOCAL_TIME
    ll start_time_ = clock();
#endif
#ifdef READ_FILE
    freopen("book.in", "r", stdin);
#ifndef DEBUG_STD
    freopen("book.out", "w", stdout);
#endif
#endif
    splay<int> sp;
    scanf("%d %d", &n, &m);
    for (int i = 0; i < n; i++)
    {
        scanf("%d", id + i);
    }
    sp.splay_build();
#ifdef LOCAL_DEBUG
    sp.debug_size();
#endif
    for (int i = 0; i < m; i++)
    {
        scanf("%s %d", operation_string, &k);
#ifdef LOCAL_DEBUG
        printf("%sn", operation_string);
#endif
        if (operation_string[0] == 'Q')
        {
            printf("%dn", sp.splay_query(k));
        }
        else if (operation_string[0] == 'T')
        {
            sp.splay_top(k);
        }
        else if (operation_string[0] == 'B')
        {
            sp.splay_bottom(k);
        }
        else if (operation_string[0] == 'I')
        {
            int shift;
            scanf("%d", &shift);
            sp.splay_move(k, shift);
        }
        else if (operation_string[0] == 'A')
        {
            printf("%dn", sp.splay_ask(k));
        }
#ifdef LOCAL_DEBUG
        sp.splay_check();
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
```