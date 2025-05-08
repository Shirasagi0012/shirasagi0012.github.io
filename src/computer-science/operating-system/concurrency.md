---
title: 并发 Concurrency
date: 2025-04-21
category:
  - 计算机科学
  - 操作系统
tags:
  - 操作系统
  - 计算机科学
---

# 并发 Concurrency

> [!TIP]
>
> 内容来源于《OSTEP》Pt. 2 Concurrency, Ch. 25-34

## 线程 Thread

### 与进程的关系

线程和进程的状态非常相似：

- 多线程程序会有多个执行点（多个程序计数器），而单线程程序只有一个执行点

- 多线程程序中每个线程共享内存地址空间，从而能访问相同的数据。

- 线程和进程都有自己的程序计数器，一组用于计算的寄存器。

  因此线程切换和进程切换一样会发生类似的上下文切换。

  - 进程的上下文保存在进程控制块（Process Control Block）
  - 线程的上下文保存在一个/多个线程控制块（Thread Control Block）

  由于共享内存地址空间，线程切换不需要切换页表。

线程和进程的栈是主要区别之一：

- 单线程应用中，只有一个栈，位于地址空间底部
- 多线程应用中，每个线程都有一个栈。

线程中所有位于栈上的变量、参数、返回值等，被放在有时被称为*线程本地*（thread-local）*存储*的地方，即线程的栈

### 线程的复杂性

- 线程的执行顺序和创建的先后顺序无关。
- 当多线程同时访问共享的数据时每次产生不同的结果。

这是由于*不可控的调度*导致的。调度程序可以自由决定线程的执行顺序，你无法知晓线程何时会执行。

#### 竞态条件

两个或者以上进程或者线程并发执行时，其最终的结果依赖于进程或者线程执行的精确时序。

因此每次执行都会得到不同的结果，是不确定的。

#### 临界区

某段代码被多个线程同时执行可能会导致**竞态条件**，这些代码也被称作**临界区**。临界区是一段访问共享变量（或者其它共享资源）的代码，一定不能由多个线程同时执行。

#### 互斥

保证一个线程在临界区执行时，其它线程被阻止进入临界区

### 原子性

即“作为一个单元”，没有中间态，要么“全部”，要么“没有”。

硬件会提供一些有用的指令，可以在这些指令上构建一个通用的集合，即所谓的同步原语。

## 线程的API

### 创建线程

```c
#include <pthread.h>
int pthread_create(
    pthread_t * 				thread,
    const pthread_attr_t *    attr,
    void *                    (*start_routine)(void*),
    void *                    arg
);
```

参数：

- `thread`: 一个`pthread_t`的指针，`pthread_create()`将会初始化这个`pthread_t`

- `attr`: 设置线程的属性，使用`pthread_attr_init()`来初始化一个属性。

- `start_routine`: 这个线程执行哪个函数（返回值类型和参数类型可以自定义）
- `arg`: `start_routine`函数的参数 （`arg`的类型需与`start_routine`的参数类型相同）

如果你需要多个参数，可以自定义一个结构体。

### 线程完成

```c
int pthread_join(
    pthread_t  thread,
    void   		**retval
);
```

参数：

- `thread`: 要等待的线程
- `retval`: 返回值放哪里。不需要返回值可以填`NULL`

> [!WARNING]
>
> 对于线程的返回值，永远不要从另一个线程返回一个指向那个线程栈内存空间的指针！
>
> 线程结束后，栈已经销毁，所有的值都被释放。

## 锁 Lock

锁可以帮助我们实现对临界区的互斥访问。

```c
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_trylock(pthread_mutex_t *mutex);
int pthread_mutex_timedlock(
    pthread_mutex_t *restrict mutex,
    const struct timespec *abs_timeout
);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

### 初始化

锁（`pthread_mutex_t`）需要使用`PTHREAD_MUTEX_INITIALIZER`来静态初始化，或者使用`pthread_mutex_init`来动态初始化（更常用）。`attr`参数可选，你可以传入一个`NULL`。

```c
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *restrict attr)
```

### 销毁

使用完成后，你也需要把锁销毁：

```c
int pthread_mutex_destroy(pthread_mutex_t *mutex)
```

