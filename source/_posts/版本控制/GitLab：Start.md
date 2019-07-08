---
title: GitLab：start
type: tags
tags:
  - null
date: 2019-07-02 12:36:17
categories:
description:
---

# 分支策略

## Gitflow及其缺陷

- 永久dev，容易与master引起不同步
- 开发分支不够灵活，不能支持多条线并行开发
  - 所有代码要回到dev上，一旦回到dev就回到了master上，可能带上了一些不能发布的代码

## one-flow

### 分支

- feature
  - 每次从最新的master、realease分支上拉出，在该分支上开发。合并时回到对应的realease
  - 分支合并的目标为release
  - 对应dev环境
  - bug fix
  - feature尽量是独立的，合并时不产生冲突。
- realease
  - 管理员从master上拉出，是该次迭代的起点
  - 不允许Push代码，只能merge
  - 在发布完成后回到master
  - 对应fat、uat、pt、pro
  - bug fix
- master
  - 永久分支

### 测试

- fat/uat阶段，从最新master拉出单独的release分支，组合需要发布的feature进行测试
- 测试阶段的所有bugfix都必须在feature上进行，dev/fat/uat不允许提交代码，只接受merge request
  - 不能直接push
- 如果存在不确定发布的feature，建议单独建一条release分支，不应该在同一条release分支中进行发布，避免后期剥离
- 如果必须在同一条分支发布，建议增加额外的开关，在生产阶段关闭该功能，直到允许发布
- 发布生产后，由发布系统控制，将发布到pro环境的release分支合并到master
- 如果有多条release，发布生产后，发布系统会自动阻止另一条release分支发布到生产
- 其他release分支，由管理员重新rebase到master，重新发布测试，重点测试是否不兼容



# 项目

## 初始化

## 创建feature

- git fetch
  - 因为代码与其他人没有交集，不用git pull
- git checkout -b feature/xxx origin/master （或release/xxx）
- git add
- git commit -m "xxx"
- git push origin feature/xxx -u（第一次必须带-u）

## merge request

- 打开页面，选择创建合并请求
- 选择分支为刚刚推送的分支，目标分支为release/xxx
- 对于正在开发的分支，打上WIP：标记，创建merge request但不合并
  - 管理员不会合并这样的代码
- 不急于合并代码，开发完毕后再决定是否合并
- 对于多人review的合并请求，在评论中@指定人，选择开始讨论
- 忘记merge命令，尤其是本地的merge操作，不要反向合并master，或release分支代码带feature分支
  - 在我们的代码建立在别人的基础上，而不是将别人的合并进来
  - 处理冲突也不会涉及其他人

## 修正merge request

三种方法

- 新commit，使用git push
- git commit -amend，使用git push -force
- git rebase -i

## MR落后无法合并

- git checkout feature/xxxx
- git fetch
- git rebase origin/release/sprintw26
- 如果存在冲突，解决后执⾏行行 git rebase —continue
- git push origin feature/xxxx —force

## MR合并

- 必须经过code review
- 如果有sql，必须通过dba review
- 必须CI测试通过
- 不应该主动合并标记为WIP的代码

25分钟

# 参考 #

1. 
