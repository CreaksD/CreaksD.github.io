---
title: "Git Fork 完全指南：从零理解开源协作的核心工作流"
date: 2026-08-18T10:00:00+08:00
draft: false
tags: ["Git", "GitHub", "开源", "教程"]
---

> 本文全程讨论的是 Git 中的 fork，和 Codex 的 fork 没有关系。

## 1. Fork 的作用是什么？

最典型场景是：我想修改别人 GitHub 上的项目，但是我没有原仓库的写权限。

比如：

```
Spring Project
       ↓
GitHub 原仓库
       ↓
你 Fork
       ↓
你的 GitHub/project
```

然后你可以：

```bash
git clone https://github.com/yourname/project.git
```

接下来：

```
你的电脑
   ↓
你的 GitHub Fork
   ↓
提交代码
   ↓
创建 Pull Request
   ↓
请求合并到原仓库
```

这就是开源项目最经典的工作流。

## 2. Fork 的时候代码是什么状态？

假设原仓库现在：

```
A → B → C → D
```

其中：A = v1、B = v2、C = v3、D = v4。

你在 D 这个时候 Fork，那么你的 Fork 初始状态也是：

```
A → B → C → D
              ↑
           Fork 时
```

但这**不是**「Git 把你的工作目录回退到了 D」，而是：GitHub 给你的账号创建了一个**独立的远程仓库副本**，初始内容和历史基于原仓库当前状态。

## 3. Fork 之后原仓库继续变化怎么办？

这才是 Fork 最重要的地方。假设原仓库继续开发：

```
原仓库：A → B → C → D → E → F
你的 Fork：A → B → C → D
```

你的 Fork **不会自动**变成 E、F——这就是为什么 GitHub 上经常出现提示：

> This branch is 2 commits behind upstream

意思就是：你的 Fork 比原仓库落后两个 commit。

## 4. 那如果我想把原仓库的新代码同步过来？

可以。通常把原仓库叫 **upstream**，自己的 Fork 叫 **origin**：

```
upstream   company/project
origin     yourname/project
```

然后：

```bash
git fetch upstream
```

再根据你的分支情况：

```bash
git merge upstream/main
```

或者：

```bash
git rebase upstream/main
```

于是原来：

```
upstream  A → B → C → D → E → F
origin    A → B → C → D
```

同步之后：

```
origin    A → B → C → D → E → F
```

## 5. 「回到 Fork 的那个版本」是什么意思？

这个才涉及「回退到 Fork 之前的版本」的问题——**不是 Fork 本身完成这个操作**。

假设 Fork 时是 D，你 Fork 完之后自己又开发：

```
A → B → C → D → E → F
```

如果你现在想「回到 Fork 当时的 D」，那么这是 `git checkout / reset / revert / restore` 等版本控制操作的问题：

```bash
git log            # 找到 D 的 commit
git checkout <D的commit>
```

这时候才是工作目录切换到 D 当时的代码状态。

## 6. 而且 Fork 不会删除历史

假设 Fork 时是 A → B → C → D，你的 Fork 会保留这些 Git 历史，你后来提交变成 A → B → C → D → E → F。

所以你随时可以 `git checkout D` 看到当时的代码。但这不是 Fork 帮你保存了一个「快照目录」，而是 **Git 本身保存了 commit 历史**。

## 7. 一个非常关键的概念：Fork 和 Branch 不是一回事

**Branch** 是在同一个仓库里面开一条开发线：

```
project
│
├── main
│
└── feature-login
```

通常是：同一个仓库、不同开发分支。

**Fork** 是在另一个 GitHub 账号下复制一个远程仓库：

```
company/project
       │
       └── Fork
              ↓
        yourname/project
```

通常是：不同账号、不同远程仓库。

所以开源贡献经常是：

```
原项目 company/project
       ↓
     Fork
       ↓
你的项目 yourname/project
       ↓
   创建 branch
       ↓
feature/fix-login
       ↓
      Push
       ↓
Pull Request
       ↓
company/project
```

## 8. 把整个过程想象成这样

```
                GitHub

        company/project
              │
              │ Fork
              ↓
        yourname/project
              │
              │ clone
              ↓
          本地工作区
              │
              ↓
        feature branch
              │
              ↓
          修改代码
              │
              ↓
            commit
              │
              ↓
            push
              │
              ↓
        yourname/project
              │
              ↓
        Pull Request
              │
              ↓
        company/project
```

所以 Fork 的核心目的其实是：**获得一个自己可以自由修改的远程副本，并能够通过 PR 向上游贡献代码**。

## 最后：对应你的问题逐个回答

| 问题 | 回答 |
|------|------|
| 「Fork 是基于当前工作目录吗？」 | ❌ 不是。Fork 是基于远程仓库创建一个属于你的远程副本。 |
| 「Fork 会把地址版本回退到之前吗？」 | ❌ 不会。Fork 不负责回退版本。 |
| 「代码会同步回滚到 Fork 当时吗？」 | ❌ 不会。Fork 之后你的代码可以继续向前发展。 |
| 「那 Fork 当时的代码还能找回来吗？」 | ✅ 可以。Fork 时对应的 commit 仍然存在于 Git 历史中，你可以通过 commit 找回当时的代码。 |
| 「原仓库后来更新会不会自动影响我的 Fork？」 | ❌ 默认不会。需要你主动从 upstream 同步。 |

一句话总结：**Fork 是「复制仓库关系」，Commit 是「保存版本历史」，Branch 是「创建开发分支」，Reset/Checkout/Revert 才是「回到/撤销某个版本」**。
