---
layout: post
title: "pthread_atfork"
description: ""
category: linux
published: false
tags: [linux, muduo]
---
{% include JB/setup %}


最近在看muduo的代码.

因为多处会用到tid, 比如日志.

所以需要获取线程的tid, 基本线程id是不变的, 没必要每次获取都通过系统调用, 可以缓存起来.
但是线程id, 在一种情况下会变化, 就是fork, fork后tid会发生变化, 因此需要更新tid.
每次fork后都要记者去更新, tid, 那就太痛苦了.

pthread_atfork 正好可以用来干这事情.

来看一下muduo中的用法:

摘自 muduo/base/thread.cc:

    void afterFork()
    {
      muduo::CurrentThread::t_cachedTid = 0;
      muduo::CurrentThread::t_threadName = "main";
      CurrentThread::tid();
      // no need to call pthread_atfork(NULL, NULL, &afterFork);
    }

    class ThreadNameInitializer
    {
     public:
      ThreadNameInitializer()
      {
        muduo::CurrentThread::t_threadName = "main";
        CurrentThread::tid();
        pthread_atfork(NULL, NULL, &afterFork);
      }
    };

    ThreadNameInitializer init;

定义了一个类, 一个全局变量, 全局变量的构造函数会在进入main函数之前调用.
初始化主线程的信息, 调用了pthread_atfork.

afterFork中的注释引起了我的注意, 为什么就没必要呢再次调用pthread_atfork呢.

那如果再子进程中调用fork，会调用afterFork吗?

会, 我测试了一下.

原因应该是pthread_at_fork,保存的信息被子进程继承过去了,所以就不用再次调用了.

上面不是重点, 进入正题....

pthread_atfork的原本设计目的
-----------------------------

    假设这么一个环境，在 fork 之前，有一个子线程 lock 了某个锁，获得了对锁的所有权。fork 以后，在子进程中，所有的额外线程都人间蒸发了。而锁却被正常复制了，在子进程看来，这个锁没有主人，所以没有任何人可以对它解锁。

    当子进程想 lock 这个锁时，不再有任何手段可以解开了。程序发生死锁。

我们可以这么做.

1. 在fork之前, 在父进程中获取所有子进程可能用到的锁, 然后在fork之后, 在子进程中释放锁就行了.

2. 为了父进程正常执行, 父进程fork后, 要释放锁.

这就是phtead_atfork, 3个参数的不同作用.

The prepare fork handler is called in the parent before fork creates the child process. 
This fork handler's job is to acquire all locks defined by the parent. 

The parent fork handler is called in the context of the parent after fork has created the child process, but before fork has returned. 
This fork handler's job is to unlock all the locks acquired by the prepare fork handler. 

The child fork handler is called in the context of the child process before returning from fork. 
Like the parent fork handler, the child fork handler too must release all the locks acquired by the prepare fork handler.


参考资料:

1.  APUE
2.  pthread_atfork, man page
3.  http://blog.codingnow.com/2011/01/fork_multi_thread.html
