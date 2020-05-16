---
title: git (二) git标签、Stash管理
categories: 笔记
tags: [git]
date: 2020-01-19
---



## 标签

- 打tag：

  1. 附注标签：git tag -a v1.4 -m "my version 1.4"
  2. 轻量标签：git tag  v1.4
  3. 后期打标签：git tag -a v1.4 <提交校验和>（如9fceb02）

- 列出标签 ： git tag

- 共享标签:

  1. git push origin v1.5 共享某个分支到服务器
  2. git push origin --tags 一次性推送很多标签
- 删除标签：git tag -d v1.4
- 检出标签：git checkout 会使你的库处于“分离头指针”的状态。大部分情况下不适用。
  
  

### Stash

- git stash save "save message" ：执行储存时，添加备注，方便查找
- git stash list ：查看stash了那些储存
- git stash show <stash num>：显示做了那些改动，如不加stash num，默认显示第一个
- git stash apply <stash num>：应用某个储存,默认使用第一个
- git stash pop <stash num>：命令恢复之前缓存的工作目录。将缓存栈堆中的对应stash删除
- git stash drop <stash num>：删除某个stash
- git stash clear ：删除所有缓存的stash



### 账户管理

- 全局：

  1. git config --global user.name "tongziqi"

  2. git config --global user.email "tongziqi@qq.com"

- 某个仓库内

  1. git config user.name "tongziqi"

  2. git config user.email "tongziqi@qq.com"

- 查看配置信息: git config --list 

  

### "HEAD" 是什么？

分支，本质上仅仅是指向commit对象可变指针。但怎么知道你当前在哪个分支上工作？

它保存着一个HEAD的特别指针，HEAD指向你当前的工作。
