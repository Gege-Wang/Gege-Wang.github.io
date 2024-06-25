---
layout: single
title:  "git merge and git rebase"
date:  2024-06-11
categories: git
---
> 写这篇文章的目的是为了记录在 git 操作的时候的一些方法，避免自己只会 `git add .`, `git commit -m`, `git push`, 关于 git merge 和 git rebase 的理解经历了很长时间，很不好意思的是，我现在也没有彻底搞清楚，但是已经学会在常见的场合去使用它们了。

### 情况一： 简称偷作业。你和另一个人都从同一个主仓库 fork , 你想用另一个人的某几个提交

1. 首先你需要将对方的仓库添加为你的远程仓库  

`git remote add <remote-name> <remote-url>`  

例如：  

`git remote add upstream https://github.com/username/repo.git`

2. 获取它的远程分支  

`git fetch upstream`  

3. 检查远程分支

`git branch -r`  

4. 合并他的分支到你的分支上

```shell
git checkout <your-branch>
git merge upstream/<their-branch>
```

5. 解决冲突（如果有）：如果在合并过程中有冲突， git 会提示你解决这些冲突，你需要手动编辑冲突文件，执行以下命令。

```shell
git add <resolved-files>
git commit
```

6. 推送修改， 作业偷取到手

```shell
git push origin <your-branch>
```

### 情况二：我提交了，但是我反悔了

#### 1. git reset

1.1 返回上一个提交，但是保留工作目录的更改  

`git reset --soft HEAD~1`

1.2 返回上一个提交，并取消暂存和对文件的更改。此操作不可逆，慎用。

`git reset --hard HEAD~1`

1.3 返回到指定的提交。

`git reset --hard <commit-hash>` or `git reset --hard <commit-name>`
> git reset --hard 是一项危险的操作，因为会永久性的删除工作目录的更改。所以一定要非常非常确定再也不需要这些更改才行。

#### 2. git revert

2.1 撤销某个特定的提交  

`git revert <commit-hash>`

2.2 撤销最后一个提交  

`git revert HEAD`

2.3 撤销多个提交  

`git revert <start_commit>^..<end_commit>`
> git revert 是一种更安全的撤销，因为它不会改变提交历史，而是创建新的提交来撤销指定提交的更改.

### 情况三：提交 PR 时多次进行小更改，需要对这些更改进行合并

1. 进入交互式操作  

`git rebase -i HEAD~N # 显示你要合并的 N 个分支`  

2. 进行合并操作  
  
```shell
pick abc1234 Commit message 1
squash def5678 Commit message 2
pick ghi91011 Commit message 3
```

这个将 squash 的分支合并到 message 1 上面， 操作完之后修改提交的 commit message， 提交记录就只剩下 commit 1 和 commit 3 了。  

3. 强制推动到远程仓库

`git push --force-with-lease origin your-branch-name`
