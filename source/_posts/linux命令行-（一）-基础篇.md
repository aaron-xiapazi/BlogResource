---
title: linux命令行 （一） 基础篇
date: 2020-04-11 21:37:38
tags:
categories:
---

掌握本文中出现的命令行，将可以处理绝大多数的linux命令操作。

本文中出现的命令一定要熟练掌握，这将是下一篇linux系列的基础

## 最常用

- 自言自语 echo
  - echo xiapazi
  - xiapazi
  - echo $PWD
  - /root

- 我在哪个目录下 pwd (Print Working Directory)
  - pwd
  - /root
- 切换目录 cd
  - cd /Python-3.7.7
  - (无返回值)
- 看看当前目录都有什么 ls
  - ls
  - Python-3.7.7 Python-3.7.7.tgz(当前目录下的文件)
  - ls - l(列表模式)
  - ls  -la （列表模式+隐藏文件）
  - ls  -lh (带文件大小)

## 寻求帮助

- 查看用户手册或帮助 man，通常man更全一些
  - man -pwd
- 帮助 -h 或 --help
  - pip -h
  - pip --help

## 文件内容

- 打印文件内容 cat(Concatenate and print files)
  - cat a.txt
  - cat a.txt b.txt
- 打印前n行 head
  - head a.txt
  - (默认打印前10行)
  - head -n 5 
  - 打印前五行

- 打印后n行 tail
  - tail a.txt
  - (默认打印前10行)
  - tail -n 5 a.txt
  - 打印后五行
  - tail -f a.txt
  - （打印后10行且跟踪这个文件，如果有新内容就会打印出来） 

- 交互浏览 less
  - less a.txt
  - 可以理解为进入到了一个只读模式的vi（vi就是linux下的一种编辑器）

- 内容查找
  - 在交互模式下 /
    - 进入交互模式后 / 之后输入你想要查找的内容
  - 在命令行中 | grep
    - cat a.txt | grep -n 8
    - (会显示文件中所有含有8的内容，加了-n之后会显示处于第几行)

- 单词统计 | wc（Word，line and byte count）
  - cat a.txt | wc
  - 100 100 292(100行，100个单词，292个字节)

## 重定向和管道

- 重定向
  - 将输入重定向到文件 >
    - echo hello > hello.txt
  - 将输入重定向追加到文件 >>
    - echo world >> hello.txt
  - 将文件内容重定向至命令行打印出来 <
    - cat <hello.txt
- 管道
  - 将前一个的输出作为下一个的输入，形成一个工作流
    - man less | grep sim
    - (进入交互模式且查找sim)
    - man less |grep -n sim | grep That > that.txt
    - (进入交互模式查找sim并显示行号，后在查找包含That，再之后重定向输入到that.txt文件中)





# 概念补充：Unix哲学

- 每个程序只做一个事情，并且把这个事情做好
- 一个程序的输出可以作为另一个程序的输入（管道），不要输出无关的内容
- 编写处理字符流的程序，因为那是通用接口
- 编写可以互相协作的程序（因为每个程序只做一件事情，多个程序组合就可以实现复杂的操作）

