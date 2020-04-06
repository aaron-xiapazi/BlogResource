---
title: git (一) 基础操作
categories: git
tags: [git]
date: 2020-01-12
---

#GIT 基础

### 基本操作

1. git init 在本地初始代码仓库
2. git clone 克隆远程仓库至本地
3. git pull 拉取远程代码
4. git add 添加文件至暂存区 (git add . 添加全部)
5. git commit -m “提交信息” 提交变更至本地
6. git commit -am “提交信息” 将本地所有改动的文件提交变更至本地
PS：Step 6 在没有增加文件的情况下 ==  git add . + git commit -m ""
7. git push 推送至远程代码仓库
8. git status 查看相关文件的状态
9. git reset --soft HEAD~1 回退上次提交
10. git reset --hard 回退上次所有改动（本地改动的代码也会消失，如果非要执行建议先stash）
11. git revert 撤销一个提交
PS：Revert 撤销一个提交的同时也会重新创建一个提交。这是一个安全的方法，因为它不会重写提交历史。相比git reset，它不会改变现在的提交历史。因此，git revert可以用在公共分支上，git reset应该用在私有分支上。你也可以把git revert当作撤销已经提交的更改，而git reset HEAD用来撤销没有提交的更改。

---

### 分支

1. git branch 查看本地分支
2. git branch -a 查看所有分支
PS: git branch -r 无法获取远程分支，ui可以看见分支但是git 命令无法查看
原因 git branch -a 这条命令并没有每一次都从远程更新仓库信息，我们可以手动更新一下
3. git barnch -r 查看远程分支
4. git checkout 切换分支
5. git checkout -b <分支> 创建并切换分支
6. git checkout -b <分支> origin(远程仓库名字)/DEV(基于的分支名称) 切换远程分支
7. git merge <分支> 将当前分支及目标分支整体当做一次提交，完成后HEAD有两个parent
8. git rebase <分支> HEAD移至当前分支顶端,将目标分支的所有提交记录依次执行到当前分支中，执行完成后目标分支仅作为改动副本,不会产生两个parent


### PS：git切换成默认源npm config set registryhttps://registry.npmjs.org
npm config set registry https://registry.npmjs.org