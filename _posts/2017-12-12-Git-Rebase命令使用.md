---
layout: post
title: 使用git rebase 合并提交
subtitle: 使用git rebase 合并提交
tags: [Git, rebase, 版本控制]
---


## 背景
先看一个github贡献代码的操作流程：

1. fork一个项目，在自己的repo下面产生一个fork过来的项目副本
2. `git clone` 到本地做自己的修改，
3. 将本地的修改 commit到仓库，push到github
4. 发起一个`pull request`请求项目的owner来进行合并
5. owner合并代码到项目，完成代码的贡献

当修改历史比较多的时候，也就是说，提交了n次的commit操作，这个时候，git 的提交历史会很不优雅，可能这些提交都是针对一个功能点做的改动。合并到owner的仓库去，也会污染原始项目的提交历史。这时可以借助rebase来进行提交历史的合并，来改变历史，使得代码提交历史更加优雅。

## git rebase简介
简单来说rebase就是把某个分支上的一部分commit嫁接到另一个commit后面，而在这个过程中这些commit的base（基）变了，所以这个操作叫做『变基』。

git rebase 用法如下

    usage: git rebase [-i] [options] [--exec <cmd>] [--onto <newbase>] [<upstream>] [<branch>]
       or: git rebase [-i] [options] [--exec <cmd>] [--onto <newbase>] --root [<branch>]
       or: git-rebase --continue | --abort | --skip | --edit-todo

    Available options are
        -v, --verbose         display a diffstat of what changed upstream
        -q, --quiet           be quiet. implies --no-stat
        --autostash           automatically stash/stash pop before and after
        --fork-point          use 'merge-base --fork-point' to refine upstream
        --onto ...            rebase onto given branch instead of upstream
        -p, --preserve-merges
                              try to recreate merges instead of ignoring them
        -s, --strategy ...    use the given merge strategy
        --no-ff               cherry-pick all commits, even if unchanged
        -m, --merge           use merging strategies to rebase
        -i, --interactive     let the user edit the list of commits to rebase
        -x, --exec ...        add exec lines after each commit of the editable list
        -k, --keep-empty	     preserve empty commits during rebase
        -f, --force-rebase    force rebase even if branch is up to date
        -X, --strategy-option ...
                              pass the argument through to the merge strategy
        --stat                display a diffstat of what changed upstream
        -n, --no-stat         do not show diffstat of what changed upstream
        --verify              allow pre-rebase hook to run
        --rerere-autoupdate   allow rerere to update index with resolved conflicts
        --root                rebase all reachable commits up to the root(s)
        --autosquash         move commits that begin with squash
                              move commits that begin with squash!/fixup! under -i
        --committer-date-is-author-date
                              passed to 'git am'
        --ignore-date         passed to 'git am'
        --whitespace ...      passed to 'git apply'
        --ignore-whitespace   passed to 'git apply'
        -C ...                passed to 'git apply'
        -S, --gpg-sign[=...]  GPG-sign commits

    Actions:
        --continue            continue
        --abort               abort and check out the original branch
        --skip                skip current patch and continue
        --edit-todo           edit the todo list during an interactive rebase

修改历史 `git rebase -i`

    pick = 要這條 commit ，什麼都不改
    reword = 要這條 commit ，但要改 commit message
    edit = 要這條 commit，但要改 commit 的內容
    squash = 要這條 commit，但要跟前面那條合併，並保留這條的 messages
    fixup = squash + 只使用前面那條 commit 的 message ，捨棄這條 message
    exec = 執行一條指令（但我沒用過）

## 使用

比如我们有如下的提交历史，当前的分支是topic：

         A---B---C topic(HEAD)
        /
    D---E---F---G master

我们执行了如下任何一个命令之后：

    $ git rebase master
    $ git rebase master topic

提交历史将会变成如下这样：

                  A'--B'--C' topic(HEAD)
                 /
    D---E---F---G master

可以看出git把A---B---C这段commit嫁接到了G之后，不过虽然这些新commit的内容是一样的，但是hash值是不同的（A'--B'--C'），原因将在后面解释。

命令完整的形式如下：

    $ git rebase <upstream> <branch>

其中<upstream>是新的base，如果你提供<branch>，那么首先会checkout到这个<branch>，然后再进行rebase操作。所以以下两种方式

    $ git rebase master topic
    $ git rebase master

的区别是第一种形式会首先checkout到topic分支，然后再执行rebase的操作。

那么rebase都做了什么事情呢？

首先，git会对topic分支和<upstream>做一个差集，把不同的commit找出来，类似于执行git log <upstream>..HEAD，对于以上例子来说结果就是A---B---C，然后把这些commit存在一个临时的地方。

其次，git会把当前分支reset到<upstream>上，类似于执行git reset --hard <upstream>命令。对于以上例子来说就是reset到master。

最后，git把第一步中暂存的commit，按照顺序一个一个地应用到分支上，相当于一个一个重复提交，这就是为什么rebase之后commit的hash值变了。

如果中的一个commit进行了某项修改，而当前分支中也存在一个commit，这两个commit的修改的内容一样，那么当前分支中的commit将会被忽略。比如以下的```A```和```A'```就是这样两个commit。

          A---B---C topic
         /
    D---E---A'---F master
执行完git rebase master之后，结果如下：

                   B'---C' topic
                  /
    D---E---A'---F master

如果你想更灵活的进行commit嫁接，那么你需要rebase --onto，命令格式如下：

    $ git rebase --onto <newbase> <upstream> <branch>
假设你有如下的branch tree：

    o---o---o---o---o  master
             \
              o---o---o---o---o  next
                               \
                                o---o---o  topic
你想要得到如下的branch tree：

    o---o---o---o---o  master
            |            \
            |             o'--o'--o'  topic
             \
              o---o---o---o---o  next
那我们需要如下操作：

    $ git rebase --onto master next topic
这个操作会把从next开始的commit嫁接到master上。如果你提供<branch>，那么首先会checkout到这个<branch>，然后再进行rebase操作。

我们再看一个例子，比如我们有如下的branch tree：

    E---F---G---H---I---J  topicA
我们想要删除F---G这两个commit，那么通过rebase --onto就可以实现：

    $ git rebase --onto topicA~5 topicA~3 topicA
执行结果是：

    E---H'---I'---J'  topicA
同样，rebase也会产生冲突，当解决完冲突之后你可以继续rebase的进程：

    $ git rebase --continue
或者取消此次rebase：

    $ git rebase --abort
关于commit修改、顺序调整、合并等操作可以通过rebase -i来完成

    $ git rebase -i <upstream>


## 参考资料

- [https://blog.yorkxin.org/2011/07/29/git-rebase](https://blog.yorkxin.org/2011/07/29/git-rebase)
- [http://www.tuicool.com/articles/q2eURn](http://www.tuicool.com/articles/q2eURn)
- [http://blog.csdn.net/alps1992/article/details/38548107](http://blog.csdn.net/alps1992/article/details/38548107)

