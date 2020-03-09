---
title: git命令
comments: true
date: 2020-03-09 22:04:42
categories: Git
tags: Git
description:
---

[TOC]
<!--more-->

### 分支操作


- [x] git branch 查看本地当前所有分支;
- [x] git branch -a 查看本地与远程的所有分支;
- [x] git branch -d <branchName> 删除本地分支，此分支已经被合并;
- [x] git branch -D <branchName> 强制删除本地分支;
- [x] git push origin --delete <branchName> 删除远程分支
- [x] git merge <branchName> 合并指定分支到当前分支;
- [x] git merge --no-ff <branchName> 合并指定分支到当前分支,生成一个新的提交;
- [x] git merge  --squash <branchName> 合并指定分支(不包含 commit 记录)到当前分支；
- [x] git checkout <branchName> 切换分支
- [x] git checkout -b <branchName> 创建并切换分支
- [x] git branch remotes/origin/<branchName> git checkout -b <branchName>切换远程分支
- [x] git branch -vv 查看本地分支与远程分支映射关系
- [x] git branch -u origin/<bracnName> 或 git branch --set-upstream-to origin/<branchName> 关联当前分支与远程分支
- [x] git branch --unset-upstream 取消当前分支与远程分支关联
### 保存操作

- [x] git stash 保存暂缓区和工作区
- [x] git stash pop 释放保存到暂缓区和工作区
- [x] git stash list 显示保存列表
- [x] git stash pop stash@{1} 释放指定指定序列
- [x] git stash drop stash@{1} 删除指定序列
- [x] git stash clear 删除所有

#### 回滚操作
- [x] git log 查看提交记录
- [x] git reset --soft <commit id> 只回滚提交记录,代码依然为当前代码
- [x] git reset --hard <commit id> 回滚到指定提交记录
- [x] git reflog 看查看操作记录
- [x] git reset --soft HEAD@{number} 只回滚操作记录，代码依然为当前代码 
- [x] git reset --hard HEAD@{number} 回滚到指定操作步骤，代码依然回滚

### 提交操作
- [x]  git status 查看当前状态
- [x]  git add -A 添加至暂缓区
- [x]  git commit -m "message" 提交至本地
- [x]  git commit --amend 以补丁形式提交至上一个分支
- [x]  git push origin HEAD:refs/for/<branchName> 提交至远程进行review
- [x]  git push 提交至远程分支
- [x]  git pull 拉取并合并到当前分支
- [x]  git pull --rebase  拉取并进行rebase
- [x]  git rebase --continue 继续当前余下的补丁
- [x]  git reabse --abort 终止当前 rebase