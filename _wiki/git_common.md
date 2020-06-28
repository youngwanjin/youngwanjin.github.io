---
layout: wiki
title: git 常用命令
categories: [wiki]
description: git 工作中常用命令
keywords: git
---

### 1. git 推荐流程

1. `git  pull`  <url>   -- 拉取代码
2. `git checkout -b  dev `  -- 切换开发分支(在自己的本地分支开发代码，开发结束)
3. `git add `  -- 将本地文件添加到git仓库
4. `git commit `  -- 提交本地代码
5.  `git checkout master`   -- 切换到本地master分支
6. `git pull ` -- 拉取最新代码
7. `git checkout dev ` -- 切换到本地开发分支
8. `git rebase master `-- 将开发分支dev合并到master分支
9. `git review ` -- 将代码推到git review上；git push  -- 将代码推送到远端仓库

> **注：**
>
> 如果在第8步遇到冲突此时需要解决冲突，解决完冲突执行如下步骤：
>
> 9. `git add ` -- 将本地修改添加到仓库（解决冲突之后只用执行 **add**，不用执行 **commit**）
> 10. `git rebase -- continue ` -- 继续合并分支
> 11. `git review `-- 将代码推到git review上；git push  -- 将代码推送到远端仓库



### 2. git 常用命令

1. `git log` 	-- 查看提交日志
2. `git reflog` 	-- 查看操作日志
3.  `git reset HEAD^ --soft/--hard ` -- soft 会保留本地修改 -- hard 不会保留本地修改
4. `git branch` -- 查看本地分支， `-a`查看所有分支 ，`-d `删除本地已合入分支，`-D`删除本地未合入分支  ，`<brance-name>` 新建分支
5. `git tag`  -- 代码打tag 
6. `git stash` --暂存工作区，执行完就是干净的工作取，`list` 查看stash列表（一般开发到一半需要切换分支工作会使用）
7. `git pop` --恢复工作区，继续开发



[Ref1]( https://www.jianshu.com/p/bf880bc2c4b3 )

[Ref2]( https://www.jianshu.com/p/c86778e33793 )