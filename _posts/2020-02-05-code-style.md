---
layout: post
title: "码农的心结：编程风格"
subtitle: '码农的心结：编程风格'
author: "顾海军"
header-style: text
tags:
  - 杂想
---

你平时定义一个函数的风格可能是这样的：

```
int Foo(bool isBar) {
    if (isBar) {
        bar();
        return 1;
    } else
        return 0;
}
```

然而，你团队中的一个大兄弟的风格是这样的：

```
int Foo(bool isBar)
{
    if (isBar) {
        bar();
        return 1;
    } else
        return 0;
}
```

或者是这样的：

```
int Foo(bool isBar)
{
    if (isBar)
    {
        bar();
        return 1;
    }
    else
        return 0;
}
```

团队开发中，由于每个开发人员的经历和成长的不同，成员中编程风格也各有特点。规定团队代码风格是保证代码可读性可维护行的基本措施之一。本文简单的介绍了一些常见的编程风格，以便参考。


# 常见编程风格

编程风格种类很多，优点缺点每个人有每个人的理解和说辞。存在即合理，这里介绍各编程风格特点，不去讨论编程风格长短问题。

## K&R 及其变体

K&R 是 Kernighan 和 Ritchie 的缩写，即《The C Programming Language》的作者。

这里变体包括：1TBS、mandatory braces、Java、Stroustrup、BSD KNF

K&R 风格，起始大括号跟随在 `array` 、 `struct` 、`enum` 、`statement` 结尾位置，但是对于 `namespace`, `class`, `function` 起始大括号放置于下一行的开头

```
int Foo(bool isBar)
{
    if (isBar) {
        bar();
        return 1;
    } else
        return 0;
}
```

K&R 变体中各编程风格均略有不同或差异

#### Linux Kernel/KNF

不同的缩进要求为 8 个空格。

```
int Foo(bool isBar)
{
        if (isFoo) {
                bar();
                return 1;
        } else
                return 0;
}
```

#### Java

不同的 `function` 中起始大括号跟随在函数名结尾

```
int Foo(bool isBar) {
    if (isBar) {
        bar();
        return 1;
    } else
        return 0;
}
```

#### Stroustrup

不同的所有起始大括号均跟随在行尾。比较特殊的 `if else` 中 `else` 位于新的一行开头。

```
int Foo(bool isBar)
{
    if (isBar) {
        bar();
        return 1;
    }
    else
        return 0;
}
```

#### 1TBS

不同的条件语句中单行语句不得省略大括号。

```
int Foo(bool isBar)
{
    if (isFoo) {
        bar();
        return 1;
    } else {
        return 0;
    }
}
```


## Allman/BSD

所有起始大括号放置于下一行的开头。

```
int Foo(bool isBar)
{
    if (isBar)
    {
        bar();
        return 1;
    }
    else
        return 0;
}
```

## Whitesmiths

起始大括号放置于下一行的开头且缩进，大括号内语句与大括号使用相同缩进。

```
int Foo(bool isBar)
    {
    if (isBar)
        {
        bar();
        return 1;
        }
    else
        return 0;
    }
```

## VTK（Visualization Toolkit）

起始大括号放置于下一行的开头且缩进（其中函数起始大括号不要求缩进），大括号内语句与大括号使用相同缩进。

```
int Foo(bool isBar)
{
    if (isBar)
        {
        bar();
        return 1;
        }
    else
        return 0;
}
```

## Banner

起始大括号均跟随在行尾，结尾大括号与大括号内语句使用相同缩进。

```
int Foo(bool isBar) {
    if (isBar) {
        bar();
        return 1;
        }
    else
        return 0;
    }
```

## GNU

起始大括号放置于下一行的开头，只有函数内部大括号可以缩进，缩进通常为 2 个空格。

```
int Foo(bool isBar)
{
  if (isBar)
    {
      bar();
      return 1;
    }
  else
    return 0;
}
```

## Horstmann

起始大括号放置于下一行的开头，起始大括号后跟首行代码，缩进通常为 3 个空格。

```
int Foo(bool isBar)
{  if (isBar)
   {  bar();
      return 1;
   }
   else
      return 0;
}
```

## Pico

起始大括号放置于下一行的开头，后跟首行代码。结尾大括号跟随代码库最后一行代码，缩进通常为 2 个空格。


```
int Foo(bool isBar)
{  if (isBar)
   {  bar();
      return 1; }
    else
      return 0; }
```

## Lisp

起始大括号均跟随在行尾，结尾大括号跟随代码库最后一行代码。

```
int Foo(bool isBar) {
    if (isBar) {
        bar()
        return 1; }
    else
        return 0; }
```

#参考

[https://en.wikipedia.org/wiki/Indentation_style](https://en.wikipedia.org/wiki/Indentation_style)
[https://www.kernel.org/doc/html/v4.15/translations/zh_CN/coding-style.html](https://www.kernel.org/doc/html/v4.15/translations/zh_CN/coding-style.html)