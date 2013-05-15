---
layout: post
title: "计算机是怎样工作的？"
description: "linux课程实验1"
category: lab
tags: [course, linux, lab]
---
{% include JB/setup %}

Linux操作系统分析实验一

Author:李明(164)

**********************


## 1.实验内容及要求
* 实验:请使用example的c代码分别生成.cpp,.s,.o和ELF可执行文件,并加载运行,分析.s汇编代码在CPU上的执行过程
* 实验报告要求:通过实验解释单任务计算机是怎样工作的,并在此基础上讨论分析多任务计算机是怎样工作的。

`example.c`源程序
{% highlight c %}
int g(int x)
{
    return x+3;
}
int f(int x)
{
    return g(x);
}
int main(void)
{
    return f(8)+1;
}

{% endhighlight %}

## 2.实验过程

#### 1. 对`Example.c`的进行预处理
{% highlight console %}
gcc -E -o example.cpp example.c
{% endhighlight %}

经过预处理后得到的`example.cpp`内容如下：
{% highlight c %}
# 1 "example.c"
# 1 "<built-in>"
# 1 "<command-line>"
# 1 "example.c"
int g(int x)
{
    return x+3;
}
int f(int x)
{
    return g(x);
}
int main(void)
{
    return f(8)+1;
}
{% endhighlight %}
预处理会对源文件进行宏替换，include文件的替换及注释的去除等，这里因为源程序中没有定义宏，也没有include文件，没有注释因此没有明显变化，只是在文件开始处添加了４行`linemarker`。

#### 2.编译为汇编代码

* 可以直接通过源程序编译得到汇编代码
{% highlight console %}
gcc –S –o example.s example.c
{% endhighlight %}

* 也可以通过上一步中得到的预处理文件编译得到
{% highlight console %}
gcc –x cpp-output –S –o example.s example.cpp
{% endhighlight %}

得到的汇编代码如下：
{% highlight c-objdump %}
    .file   "example.c"
    .text
    .globl  g
    .type   g, @function
g:
.LFB0:
    .cfi_startproc
    pushl   %ebp
    .cfi_def_cfa_offset 8
    .cfi_offset 5, -8
    movl    %esp, %ebp
    .cfi_def_cfa_register 5
    movl    8(%ebp), %eax
    addl    $3, %eax
    popl    %ebp
    .cfi_def_cfa 4, 4
    .cfi_restore 5
    ret
    .cfi_endproc
.LFE0:
    .size   g, .-g
    .globl  f
    .type   f, @function
f:
.LFB1:
    .cfi_startproc
    pushl   %ebp
    .cfi_def_cfa_offset 8
    .cfi_offset 5, -8
    movl    %esp, %ebp
    .cfi_def_cfa_register 5
    subl    $4, %esp
    movl    8(%ebp), %eax
    movl    %eax, (%esp)
    call    g
    leave
    .cfi_restore 5
    .cfi_def_cfa 4, 4
    ret
    .cfi_endproc
.LFE1:
    .size   f, .-f
    .globl  main
    .type   main, @function
main:
.LFB2:
    .cfi_startproc
    pushl   %ebp
    .cfi_def_cfa_offset 8
    .cfi_offset 5, -8
    movl    %esp, %ebp
    .cfi_def_cfa_register 5
    subl    $4, %esp
    movl    $8, (%esp)
    call    f
    addl    $1, %eax
    leave
    .cfi_restore 5
    .cfi_def_cfa 4, 4
    ret
    .cfi_endproc
.LFE2:
    .size   main, .-main
    .ident  "GCC: (Ubuntu/Linaro 4.6.3-1ubuntu5) 4.6.3"
    .section    .note.GNU-stack,"",@progbits
{% endhighlight %}