---
layout: post
title: "数组和指针的差异"
description: "c"
category: tech
tags: [c]
---
{% include JB/setup %}


数组和指针的差异
----------------

先在c trap and pitfalls中看到过这个问题，一者没整理.
今天在看c 专家编程 和csapp 链接章节时，又遇到了这个问题。

先说现象:

两个文件

foo.c
    #include <stdio.h>
    char buf[] = "abcd";
    void print_foo()
    {
        printf("foo:%p\n", buf);
    }
bar.c
    #include <stdio.h>
    extern char *buf;
    int main(int argc, const char *argv[])
    {
        print_foo();
        printf("bar:%p\n", buf);
        printf("bar:%p\n", &buf);
        return 0;
    }
编译运行:
    gcc -o test  foo.c bar.c 
    ./test
输出:
    foo:0x804a020
    bar:0x64636261
    bar:0x804a020

怎么回事，第二个输出怎么不一样， 不是应该时数组的地址吗？第三个输入又时什么呢?

我们知道数组名，表示是一个常量，不能修改，在上述例子中就是"abcd"的首地址。

指针是一个占用4个字节(32位操作系统)的变量.

链接器在链接的时候，在链接bar.o时, 会对引用的符号变量, 进行符号查找确定地址.
这就找到了buf在foo.o中的定义的符号buf, 其地址是0x804a020, 就然后就作为了bar.o 中的buf地址.
这解释了第三个的输出.

但是第二输出呢?

在bar中打印buf,本意时打印buf数组的首地址,但是由于buf被声明为一个指针.
访问buf就是访问从0x804a020四个字节的内容，就是"abcd", 16进制位:0x61626364.
我的机器字节序是小端存储，因此访问的内容是0x64636261.

上面这段代码，指出了数组的指针的一个不同，数组名时一个常量地址, 不占用内存，指针确实一个变量占用内存.

也告诫我们在外部变量进行声明的时候,类型应完全一致.

各位看官,为什么foo.c 总buf被当作了数组名,而不是指针呢?

类型只在高级语言中才有意义，在生成汇编代码的时，会利用类型信息产生合适的指令，如根据类型信息确定操作数的大小，产生movl或movw或movb传送指令.

嗯，是编译时用到了类型信息，所以在对foo.c 编译的时候，就确定了buf的类型.

ps: 上述的链接过程，描述的比较糙，具体去看csapp吧，这本书强烈推荐.
