---
title: MyBatis的SQL执行结果和客户端相同SQL执行结果不一致问题
date: 2025/4/9
author: 花木凋零成兰
tags: 
  - Java
  - 后端  
  - Mybatis
cover: https://img.upyun.ytazwc.top/blog/20250409172647.png
hiddenCover: true
---

# MyBatis的SQL执行结果和客户端相同SQL执行结果不一致问题

在开发时候，我们使用MyBatis进行数据库操作，但是发现SQL执行结果和客户端执行结果不一致，导致出现一些问题。

## 问题环境

- Oracle 数据库

## 原因描述

先描述出现Mybatis的SQL执行结果与客户端相同SQL执行结果不一致问题的原因, 是因为笔者在调试接口的时候, 参数是从某软件**直接复制**过来在 Apifox 粘贴并发送的;

问题就出在从别的软件直接复制过来上, 复制的时候携带了一个隐藏字符 [ZWNJ]; 这个字符的 Unicode 编码为 `U+200C` 是不可见的字符;

当参数携带了一个隐藏字符, 在Mybatis中构造SQL语句时,  where 条件自然是无法成立的, 然后就会导致 SQL 的执行出现问题, 自然会出现与客户端SQL执行结果不一致的情况;

## 问题发现

经过多次调试和Debug, 笔者终于在文章: [MyBatis的SQL执行结果和客户端执行结果不一致问题排查](https://blog.csdn.net/modelmd/article/details/128460956) 中找到可能的问题所在;

使用 idea 自带的 Debug 最终发现了隐藏字符;

![](https://img.upyun.ytazwc.top/blog/20250409172647.png)

::: warning 注意
此处为模拟场景
:::

同时可以很显然发现, **虽然 Debug 会将参数值在旁边标出来, 但是不点击打开, 使很难发现隐藏字符的**;

我们可以查看控制台打印的 SQL 语句, 如下所示:

![](https://img.upyun.ytazwc.top/blog/20250409172931.png)

是完全看不出问题所在的, 这个时候一般会将 SQL 复制到客户端中执行来尝试找到问题, Navicat客户端执行结果如下所示:

![](https://img.upyun.ytazwc.top/blog/20250409173408.png)

上述语句更加突显了隐藏字符带来的影响, 直接复制 `%Jack%` 显然是包含了隐藏字符;

## 问题总结

本次能及时发现问题的原因, 是因为接口是 Get 接口, 本身参数不多, 也有先例可以参考; 在 Debug 的过程中, 使用笨方法, 一个一个参数点击开来是存在发现问题的可能性的;

可是不排除参数极多的情况, 在这种情况下, 难道一个一个参数进行点击吗?

尝试借助文本工具软件, 查看是否能显现隐藏字符, 采用软件有 Notepad++ 和 Windows自带的记事本, 结果如下:

![](https://img.upyun.ytazwc.top/blog/20250409173912.png)

可以发现Windows的记事本显现了隐藏字符, 可以很容易看到奇怪的地方, 此外也可以尝试借助项目中的 `application.yml` 文件;

在日常的开发学习中, 我们要做的还是尽量避免问题, **因此对于复制参数不长的情况, 可以尝试自己手打, 尽量将参数先复制到文本工具中, 再复制到 Apifox 中**.

## 参考文章

- [MyBatis的SQL执行结果和客户端执行结果不一致问题排查](https://blog.csdn.net/modelmd/article/details/128460956)



