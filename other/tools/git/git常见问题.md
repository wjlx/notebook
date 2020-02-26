# git常见问题

[TOC]

## 一、提交问题

### 1. 撤销 git commit —amend 操作

```md
    git reflog  # 查看提交
    git reset --hard {id} # 撤回到上个提交
    git reset --soft {id} # 撤回到上一个提交，并保留缓冲区数据（amend 之前未提交数据）
```

### 2. 提交到了错误的分支上的处理

方法一：

```md
    # 取消最新的提交，然后保留现场原状
    git reset HEAD~ --soft
    git stash
    # 切换到正确的分支
    git checkout name-of-the-correct-branch
    git stash pop
    git add .    # 或添加特定文件
    git commit -m "你的提交说明"
    # 现在你已经提交到正确的分支上了
```

方法二：

```md
    git checkout name-of-the-correct-branch
    # 把主分支上的最新提交摘过来，嘻嘻～～
    git cherry-pick master
    # 再删掉主分支上的最新提交
    git checkout master
    git reset HEAD~ --hard
```

### 3. 删除所有提交中的文件

```md
    git filter-branch --tree-filter 'rm -f README.md' HEAD
```

## 二、版本问题

### 1. 本地回退

#### 1.1 checkout

对单个文件进行回退。不会修改当前的HEAD指针的位置，也就是提交并未回退
可以是某次提交的hash值，或者HEAD（缺省即默认）

```md
    git checkout <commit> -- <filename>
```

#### 1.2 reset

回退到的指定提交以后的提交都会从提交日志上消失.
**注意：**工作区和暂存区的内容都会被重置到指定提交的时候，如果不加--hard则只移动HEAD的指针，不影响工作区和暂存区的内容。
这个方法可以对你的回退操作进行回退，因为这时候git log命令已经找不到历史提交的hash值了。

```md
    git reset --hard <commit>
    git reflog # 找到某次操作前的提交hash值
    git reset <commit>
```

### 1.3 revert

这个方法是最温和，最受推荐的，因为本质上不是修改过去的版本历史，而是将回退版本历史作为一次新的提交，所以不会改变版本历史，在push到远程仓库的时候也不会影响到团队其他人。

```md
git revert <commit>
```

## 分支管理

### 1. 删除不存在对应远程分支的本地分支

```md
git remote prune origin
```

更简单的方法是使用这个命令，它在fetch之后删除掉没有与远程分支对应的本地分支：

```md
git fetch -p
```
