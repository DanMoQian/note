# git

添加到暂存区

```git
git add 
```

提交

```git
git commit -m 'XXXX'
```

日志

```git
git log
```

查看工作区与暂存区的不同

```git
git diff fileName
```

查看暂存区与版本库的不同

```git
git diff --cached fileName
```

版本穿梭

```git
 git reset --hard 2ba98
```

要关联一个远程库，使用命令

```git
git remote add origin https://github.com/自己的github账户名/learn-git.git
```

关联后，使用命令

```
git push -u origin master
```

第一次推送master分支的所有内容；

此后，每次本地提交后，只要有必要，就可以使用命令

```git
git push origin master
```

推送最新修改；

## 分支

Git鼓励大量使用分支：

查看分支：`git branch`

创建分支：`git branch <name>`

切换分支：`git checkout <name>`或者`git switch <name>`

创建+切换分支：`git checkout -b <name>`或者`git switch -c <name>`

合并某分支到当前分支：`git merge <name>`

删除分支：`git branch -d <name>`

##  bug

修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除；

当手头工作没有完成时，先把工作现场`git stash`一下，然后去修复bug，修复后，再`git stash pop`，回到工作现场；

在master分支上修复的bug，想要合并到当前dev分支，可以用`git cherry-pick <commit>`命令，把bug提交的修改“复制”到当前分支，避免重复劳动。

## 多人协作

- 查看远程库信息，使用`git remote -v`；
- 本地新建的分支如果不推送到远程，对其他人就是不可见的；
- 从本地推送分支，使用`git push origin branch-name`，如果推送失败，先用`git pull`抓取远程的新提交；
- 在本地创建和远程分支对应的分支，使用`git checkout -b branch-name origin/branch-name`，本地和远程分支的名称最好一致；
- 建立本地分支和远程分支的关联，使用`git branch --set-upstream branch-name origin/branch-name`；
- 从远程抓取分支，使用`git pull`，如果有冲突，要先处理冲突。