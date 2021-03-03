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

# Git游戏-Learn Git Branching学习笔记

> [Learn Git Branching](https://learngitbranching.js.org/)来自同学张大神安利的一款关于学习Git的h5游戏
>
> ```
> 游戏TIPS
> reset --forSolution 刷新             
> show solution 答案   
> 复制代码
> ```

# 本地仓库的操作（branch、merge、rebase 等等）

```
git Branch name                 创建一个分支
git checkout -b name            创建并切换到当前分支
git merge name                  合并分支
git rebase name                 合并分支
git log                         来查看提交记录的哈希值
git branch -f master HEAD~3     移动分支,将 master 分支强制指向 HEAD 的第 3 级父提交。
git branch -f master 哈希值     移动分支到某一次提交环境下
git checkout HEAD~4             用 HEAD~<num> 一次后退四步。
复制代码
git checkout bugFix;
git merge master  
复制代码
```

因为 master 继承自 bugFix ，Git 什么都不用做，只是简单地把 bugFix 移动到 master 所指向的那个提交记录。 现在所有提交记录的颜色都一样了，这表明每一个分支都包含了代码库的所有修改

> 使用 `HEAD^` 向上移动 1 个提交记录
> 使用 `HEAD~<num>` 向上移动多个提交记录，如 ~3





## 撤销变更 [git reset/revert 哈希值]

C0->C1->C2 ==> C0->C1

- [在reset后， C2所做的变更还在，但是本地代码库处于未加入暂存区状态，对大家一起使用的远程分支是无效的]

C0->C1->C2 ==> C0->C1->C2->C2'

- [在revert后，C2' 的状态与 C1 是相同的，revert 之后就可以把你的更改推送到远程仓库与别人分享。]

```
git reset 哈希值       本地分支修改
git revert 哈希值      远程分支修改
复制代码
```

## 整理提交记录

- git cherry-pick <哈希值>...
  将一些提交复制到当前所在的位置（HEAD）下面
- 交互式的 rebase [git rebase -i HEAD~num]
  从一系列的提交记录中找到想要的记录
  交互式 rebase 指的是使用带参数 --interactive 的 rebase 命令, 简写为 -i

## 本地栈式提交

```
git rebase -i
git cherry-pick
复制代码
```

### git rebase -i

> 提取分支最后一次调试结果合并到主分支下

我们可以通过下面的方法来克服困难：

- 先用 git rebase -i 将提交重新排序，然后把我们想要修改的提交记录挪到最前
- 然后用 commit --amend 来进行一些小修改
- 接着再用 git rebase -i 来将他们调回原来的顺序
- 最后我们把 master 移到修改的最前端（用你自己喜欢的方法），就大功告成啦！

> 当然完成这个任务的方法不止上面提到的一种（我知道你在看cherry-pick啦），之后我们会多点关注这些技巧啦，但现在暂时只专注上面这种方法。最后有必要说明一下目标状态中的那几个'——我们把这个提交移动了两次，每移动一次会产生一个'；而C2上多出来的那个是我们在使用了amend参数提交时产生的，所以最终结果就是这样了。
> 也就是说，我在对比结果的时候只会对比提交树的结构，对于 ' 的数量上的不同，并不纳入对比范围内。只要你的 master分支结构与目标结构相同，我就算你通过。

### git cherry-pick



## git tags name 哈希值

#### 哪某些状况下使用？

> 分支很容易被人为移动，并且当有新的提交时，它也会移动。分支很容易被改变，大部分分支还只是临时的，并且还一直在变。
> 你可能会问了：有没有什么可以永远指向某个提交记录的标识呢，比如软件发布新的大版本，或者是修正一些重要的 Bug 或是增加了某些新特性，有没有比分支更好的可以永远指向这些提交的方法呢？

#### 作用

> Git 的 tag 就是干这个用的啊，它们可以（在某种程度上 —— 因为标签可以被删除后重新在另外一个位置创建同名的标签）永久地将某个特定的提交命名为里程碑，然后就可以像分支一样引用了。
> 更难得的是，它们并不会随着新的提交而移动。你也不能检出到某个标签上面进行修改提交，它就像是提交树上的一个锚点，标识了某个特定的位置。不指定提交记录，Git 会用 HEAD 所指向的位置。





## git describe

#### 哪某些状况下使用？

> 由于标签在代码库中起着“锚点”的作用，Git还为此专门设计了一个命令用来描述离你最近的锚点（也就是标签），它就是 git describe！
> Git Describe 能帮你在提交历史中移动了多次以后找到方向；当你用 git bisect（一个查找产生 Bug 的提交记录的指令）找到某个提交记录时，或者是当你坐在你那刚刚度假回来的同事的电脑前时， 可能会用到这个命令。

#### git describe 的语法是：

> `git describe <ref>`
> `<ref>` 可以是任何能被 Git 识别成提交记录的引用，如果你没有指定的话，Git 会以你目前所检出的位置（HEAD）。
> 它输出的结果是这样的：`<tag>_<numCommits>_g<hash>`
> tag 表示的是离 ref 最近的标签， numCommits 是表示这个 ref 与 tag 相差有多少个提交记录， hash 表示的是你所给定的 ref 所表示的提交记录哈希值的前几位。当 ref 提交记录上有某个标签时，则只输出标签名称





## 选择父提交记录

> 操作符 ^ 与 ~ 符一样，后面也可以跟一个数字。
> 但是该操作符后面的数字与~后面的不同，并不是用来指定向上返回几代， 而是指定合并提交记录的某个父提交。 还记得前面提到过的一个合并提交有两个父提交吧，所以遇到这样的节点时该选择哪条路径就不是很清晰了。 Git 默认选择合并提交的“第一个”父提交，在操作符 ^ 后跟一个数字可以改变这一默认行为。

```
git checkout Head~^2~2
git checkout master^2
复制代码
```





## 纠缠不清的分支

> 现在我们的 master 分支是比 one、two 和 three 要多几个提交。出于某种原因，我们需要把 master 分支上最近的几次提交做不同的调整后，分别添加到各个的分支上。
> one 需要重新排序并删除 C5，two 仅需要重排排序，而 three 只需要提交一次。
> 慢慢来，你会找到答案的 —— 记得通关之后用 show solution 看看我们的答案哦。

```
git checkout one
git cherry-pick C4 C3 C2
git checkout two
git cherry-pick C5 C4 C3 C2
git branch -f three C2
复制代码
git branch -f three C2
git checkout one
git rebase -i C1 C4
git branch -f one C2'
git checkout two
git rebase -i two C5
git branch -f two C2''
复制代码
```





# 远程仓库

> 本地仓库多了一个名为 origin/master 的分支, 这种类型的分支就叫远程分支。由于远程分支的特性导致其在你检出时自动进入分离 HEAD 状态。
> 远程分得有一个特别的属性，在你检出时自动进入分离 HEAD 状态。Git 这么做是出于不能直接在这些分支上进行操作的原因, 你必须在别的地方完成你的工作, （更新了远程分支之后）再用远程分享你的工作成果。 远程分支有一个命名规范 —— 它们的格式是:`<remote name>/<branch name>`
> 例如一个名为 origin/master 的分支，那么这个分支就叫 master，远程仓库的名称就是 o
> Git 远程仓库的操作实际可以归纳为两点：向远程仓库传输数据以及从远程仓库获取数据。





## git fetch —— 从远程仓库获取数据

你会看到当我们从远程仓库获取数据时,远程分支也会更新以反映最新的远程仓库。

#### git fetch 做了些什么

git fetch 完成了仅有的但是很重要的两步:

- 从远程仓库下载本地仓库中缺失的提交记录
- 更新远程分支指针(如 origin/master)

git fetch 实际上将本地仓库中的远程分支更新成了远程仓库相应分支最新的状态。远程分支反映了远程仓库在你最后一次与它通信时的状态，git fetch 就是你与远程仓库通信的方式了！git fetch 通常通过互联网（使用 http:// 或 git:// 协议) 与远程仓库通信。

#### git fetch 不会做的事

> git fetch 并不会改变你本地仓库的状态。它不会更新你的 master 分支，也不会修改你磁盘上的文件。





## git pull —— 从远程仓库更新本地仓库

> 当远程分支中有新的提交时，你可以像合并本地分支的那样来合并远程分支,执行以下命令:

```
git cherry-pick origin/master
git rebase origin/master
git merge origin/master
复制代码
```

git pull 就是 git fetch 和 git merge 的缩写

```
git fetch
git merge origin/master
复制代码
```

> 假设你周一克隆了一个仓库，然后开始研发某个新功能。到周五时，你新功能开发测试完毕，可以发布了。但是——天啊！你的同事这周写了一堆代码，还改了许多你的功能中使用的
> API，这些变动会导致你新开发的功能变得不可用。但是他们已经将那些提交推送到远程仓库了，因此你的工作就变成了基于项目旧版的代码，与远程仓库最新的代码不匹配了。
> 这种情况下, git push 就不知道该如何操作了。如果你执行 git push，Git 应该让远程仓库回到星期一那天的状态吗？还是直接在新代码的基础上添加你的代码，异或由于你的提交已经过时而直接忽略你的提交？
> 因为这情况（历史偏离）有许多的不确定性，Git 是不会允许你 push 变更的。实际上它会强制你先合并远程最新的代码，然后才能分享你的工作。

```
git fetch
git rebase origin/master
git push
复制代码
```

git fetch 更新了本地仓库中的远程分支，然后用rebase将工们的工作移动到最新的提交记录下

```
git fetch
git merge origin/master
git push
复制代码
```

git fetch 更新了本地仓库中的远程分支，然后合并了新变更到我们的本地分支（为了包含远程仓库的变更），最后我们用 git push 把工作推送到远程仓库

- git pull 就是 fetch 和 merge 的简写，类似的 git pull --rebase 就是 fetch 和 rebase 的简写！





## 合并特性分支

如何快速的更新 master 分支并推送到远程

```
git pull --rebase       将我们的工作 rebase 到远程分支的最新提交记录
git push

创建远程分支
git push --set-upstream origin alice
复制代码
```

在开发社区里，有许多关于 merge 与 rebase 的讨论。以下是关于 rebase 的优缺点：

- 优点:Rebase 使你的提交树变得很干净, 所有的提交都在一条线上
- 缺点:Rebase 修改了提交树的历史比如, 提交 C1 可以被 rebase 到 C3 之后。这看起来 C1 中的工作是在 C3 之后进行的，但实际上是在 C3 之前。

一些开发人员喜欢保留提交历史，因此更偏爱merge。而其他人可能更喜欢干净的提交树，于是偏爱 rebase





## 设置远程追踪分支的方法

```
git checkout -b foo origin/master
git branch -u origin/master foo
复制代码
```

> 如果当前就在 foo 分支上, 还可以省略 foo：git branch -u o/master

```
git checkout -b side o/master
git commit 
git pull --rebase
git push
复制代码
git commit
git checkout -b side o/master
git pull
git cherry-pick C3
git branch -f master C1
git push
复制代码
```

#### push 指定参数，语法是：`git push <remote> <place>`

> `git push origin master` 把这个命令翻译过来就是：
> 切到本地仓库中的“master”分支，获取所有的提交，再到远程仓库“origin”中找到“master”分支，将远程仓库中没有的提交记录都添加上去，搞定之后告诉我。
> 我们通过“place”参数来告诉 Git 提交记录来自于 master, 要推送到远程仓库中的 master。它实际就是要同步的两个仓库的位置。
> 需要注意的是，因为我们通过指定参数告诉了 Git 所有它需要的信息, 所以它就忽略了我们所检出的分支的属性！

```
git checkout C0
git push origin master   //ok
git push    //error
复制代码
```

要同时为源和目的地指定 `<place>` 的话，只需要用冒号 `:`将二者连起来就可以了:
`git push origin <source>:<destination>`
这个参数实际的值是个 `refspec`，“refspec” 是一个自造的词，意思是 Git 能识别的位置（比如分支 foo 或者 HEAD~1） 一旦你指定了独立的来源和目的地，就可以组织出言简意赅的远程操作命令了

```
git push origin foo^:master  //foo^->参照位置 ： master->需要移动的远程分支
git push origin master:newBranch
复制代码
```

#### git fetch `<place>` 参数

> 如果你像如下命令这样为 git fetch 设置 `<place>` 的话：git fetch origin foo
> Git 会到远程仓库的 foo 分支上，然后获取所有本地不存在的提交，放到本地的 o/foo 上。

> `git fetch origin <source>:<destination>`
> source 现在指的是远程仓库中的位置，而 `<destination>` 才是要放置提交的本地仓库的位置。它与 git push 刚好相反

```
git fetch origin foo
git fetch origin foo~1:bar
复制代码
```

> 跟 git push 一样，Git 会在 fetch 前自己创建立本地分支, 就像是 Git 在 push 时，如果远程仓库中不存在目标分支，会自己在建立一样。





## 古怪的 `<source>`

> Git 有两种关于 `<source>` 的用法是比较诡异的，即你可以在 git push 或 git fetch 时不指定任何 source，方法就是仅保留冒号和 destination 部分，source 部分留空。
> git push origin :side
> git fetch origin :bugFix

###### 如果 push 空 `<source>` 到远程仓库会如何呢？

> 它会删除远程仓库中的分支！

###### 如果 fetch 空 `<source>` 到本地

> 会在本地创建一个新分支。

```
git push origin :foo
git fetch origin :bar
复制代码
```

> 以下命令在 Git 中是等效的:

```
git pull origin foo     
//      相当于：
git fetch origin foo; git merge o/foo

git pull origin bar~1:bugFix    
//      相当于：
git fetch origin bar~1:bugFix; git merge bugFix
复制代码
git pull origin bar:foo

git pull origin master:side
```