## 分支

在 GIT [官方文档](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%AE%80%E4%BB%8B) 中，对分支的定义为：

> Git 的分支，其实本质上仅仅是指向提交对象的可变指针。

其中提交对象就是前文所说的 `commit 对象`，它保存有某次提交的信息，包含 `tree 对象`，`blob 对象`，其中 `tree 对象`存储了项目的目录结构信息。

每一次提交都会生成一个 `commit 对象`，而分支所指的指针也会随之指向最新的一次提交

![分支及其提交历史。](https://git-scm.com/book/en/v2/images/branch-and-history.png)

### 创建分支

一个初始化项目的文件目录如下：

```shell
root@localhost:~/git-branch$ tree -al
.
└── .git
    ├── HEAD
    ├── branches
    ├── config
    ├── description
    ├── hooks
    ├── info
    │   └── exclude
    ├── objects
    │   ├── info
    │   └── pack
    └── refs
        ├── heads
        └── tags

10 directories, 4 files
root@localhost:~/git-branch$ cat .git/HEAD
ref: refs/heads/master
```

前文说过 `.git/HEAD` 文件永远指向当前工作分支，通过查看其内容发现它指向了master 分支，但是我们发现，分支文件夹 `.git/refs/heads` 下并没有分支文件，这便是 `master` 作为默认分支的特殊之处。

首先生成一次提交，将 master 分支更新到最新一次提交的 commit object

```shell
root@localhost:~/git-branch$ tree -al
.
├── .git
│   ├── COMMIT_EDITMSG
│   ├── HEAD
│   ├── branches
│   ├── config
│   ├── description
│   ├── hooks
│   ├── index
│   ├── info
│   │   └── exclude
│   ├── logs
│   │   ├── HEAD
│   │   └── refs
│   │       └── heads
│   │           └── master
│   ├── objects
│   │   ├── 18
│   │   │   └── e4c60dd046e18c448b90543fc72d6662227f2a
│   │   ├── ba
│   │   │   └── 07affb996e18887b3613513423112a146dffb2
│   │   ├── fd
│   │   │   └── 43b3fdb5bb9218c56cf1a7375256002eadd605
│   │   ├── info
│   │   └── pack
│   └── refs
│       ├── heads
│       │   └── master
│       └── tags
└── first.txt

16 directories, 13 files

root@localhost:~/git-branch$ cat .git/HEAD
ref: refs/heads/master

root@localhost:~/git-branch$ cat .git/refs/heads/master
ba07affb996e18887b3613513423112a146dffb2

root@localhost:~/git-branch$ git cat-file -t ba07affb
commit
root@localhost:~/git-branch$ git cat-file -p ba07
tree fd43b3fdb5bb9218c56cf1a7375256002eadd605
author aynimor <aynimor@outlook.com> 1656512769 +0800
committer aynimor <aynimor@outlook.com> 1656512769 +0800

1st branch commit

root@localhost:~/git-branch$ git cat-file -p fd43
100644 blob 18e4c60dd046e18c448b90543fc72d6662227f2a    first.txt
```

查看 git 仓库文件可以发现，HEAD 文件指向了 master 分支，而 master 分支又指向了最新的 commit 对象，commit 对象保存了本次提交的 tree 对象，而 tree 对象又记录了项目内所有的文件。

这便是 GIT 仓库记录一个分支的方式，这也是为什么官方文档会说：

> Git 处理分支的方式可谓是难以置信的轻量，创建新分支这一操作几乎能在瞬间完成，并且在不同分支之间的切换操作也是一样便捷。

我们通过命令 `git branch <branch-name>` 新建一个分支来验证这一描述

``` shell
root@localhost:~/git-branch$ git branch new-branch
root@localhost:~/git-branch$ tree -al
.
├── .git
│   ├── HEAD
│   ├── branches
│   ├── config
│   ├── description
│   ├── hooks
│   ├── index
│   ├── info
│   │   └── exclude
│   ├── logs
│   │   ├── HEAD
│   │   └── refs
│   │       └── heads
│   │           ├── master
│   │           └── new-branch
│   ├── objects
│   │   ├── 18
│   │   │   └── e4c60dd046e18c448b90543fc72d6662227f2a
│   │   ├── ba
│   │   │   └── 07affb996e18887b3613513423112a146dffb2
│   │   ├── fd
│   │   │   └── 43b3fdb5bb9218c56cf1a7375256002eadd605
│   │   ├── info
│   │   └── pack
│   └── refs
│       ├── heads
│       │   ├── master
│       │   └── new-branch
│       └── tags
└── first.txt

16 directories, 15 files
root@localhost:~/git-branch$ cat .git/refs/heads/new-branch
ba07affb996e18887b3613513423112a146dffb2
root@localhost:~/git-branch$ cat .git/refs/heads/master
ba07affb996e18887b3613513423112a146dffb2
commit ba07affb996e18887b3613513423112a146dffb2 (HEAD -> master, new-branch)
Author: aynimor <aynimor@outlook.com>
Date:   Wed Jun 29 22:26:09 2022 +0800

    1st branch commit
root@localhost:~/git-branch$
```

可以发现，git 仓库只多出了两个文件，其中一个为日志信息，真正的 分支文件只有 `.git/refs/heads/new-branch`，其记录一次最新的提交与 master 分支保持一致，`.git/HEAD` 同时指向了这两个分支。

通过 `git checkout new-branch` 切换到新的分支并为当前分支新增一次提交，观察内容变化

``` shell
root@localhost:~/git-branch$ git checkout new-branch
Switched to branch 'new-branch'
root@localhost:~/git-branch$ cat .git/HEAD
ref: refs/heads/new-branch
root@localhost:~/git-branch$ echo "new-branch first commit" > new-branch-first.txt
root@localhost:~/git-branch$ git add new-branch-first.txt
root@localhost:~/git-branch$ git commit -m "new branch first commit"
[new-branch 5a5a720] new branch first commit
 1 file changed, 1 insertion(+)
 create mode 100644 new-branch-first.txt
root@localhost:~/git-branch$ tree -al
.
├── .git
│   ├── HEAD
│   ├── objects
│   │   ├── 14
│   │   │   └── 5050ada9fbf64db4c0e8cae685526d61e424e5
│   │   ├── 18
│   │   │   └── e4c60dd046e18c448b90543fc72d6662227f2a
│   │   ├── 47
│   │   │   └── e851e0487feb599434dfb7e24c1ee5cea1ebae
│   │   ├── 6d
│   │   │   └── c6463f884d71e9cbbbdc75eea419487d0d6259
│   │   ├── ba
│   │   │   └── 07affb996e18887b3613513423112a146dffb2
│   │   ├── fd
│   │   │   └── 43b3fdb5bb9218c56cf1a7375256002eadd605
│   └── refs
│       ├── heads
│       │   ├── master
│       │   └── new-branch
│       └── tags
├── first.txt
└── new-branch-first.txt

19 directories, 19 files
root@localhost:~/git-branch$ cat .git/refs/heads/new-branch
ba07affb996e18887b3613513423112a146dffb2

root@localhost:~/git-branch$ git cat-file -t ba07
commit

root@localhost:~/git-branch$ git cat-file -p ba07
tree fd43b3fdb5bb9218c56cf1a7375256002eadd605
author aynimor <aynimor@outlook.com> 1656512769 +0800
committer aynimor <aynimor@outlook.com> 1656512769 +0800

1st branch commit

root@localhost:~/git-branch$ cat .git/refs/heads/master
47e851e0487feb599434dfb7e24c1ee5cea1ebae

root@localhost:~/git-branch$ git log
commit 5a5a7209702b78f3464e2cc73d5bd2005bc25dd5 (HEAD -> new-branch)
Author: aynimor <aynimor@outlook.com>
Date:   Wed Jun 29 22:46:55 2022 +0800

    new branch first commit

commit 3350f9bde39686e736959f5c69cc5aa437032617 (master)
Author: aynimor <aynimor@outlook.com>
Date:   Wed Jun 29 22:44:49 2022 +0800

    1st branch commit
```

可以发现 `.git/HEAD` 指向内容变为了 `new-branch` 分支，`bew-branch` 分支指向内容也变为了最新一次提交。

#### detached HEAD

`.git/HEAD` 指针指向分支，而分支又指向了 commit。GIT 仓库提供了一种方式 `git checkout <commit hash>`  可以让我们切换到任何一次提交

``` shell
root@localhost:~/git-branch$ echo "second new-branch msg" > new-branch-second.txt
root@localhost:~/git-branch$ git add new-branch-second.txt
root@localhost:~/git-branch$ git commit -m "2nd new-branch commit"
[new-branch 52b1d5b] 2nd new-branch commit
 1 file changed, 1 insertion(+)
 create mode 100644 new-branch-second.txt
root@localhost:~/git-branch$ git checkout 5a5a72
Note: switching to '5a5a72'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 5a5a720 new branch first commit
root@localhost:~/git-branch$ ls
first.txt  new-branch-first.txt
root@localhost:~/git-branch$ cat .git/HEAD
5a5a7209702b78f3464e2cc73d5bd2005bc25dd5
root@localhost:~/git-branch$ git status
HEAD detached at 5a5a720
nothing to commit, working tree clean
root@localhost:~/git-branch$ git log
commit 5a5a7209702b78f3464e2cc73d5bd2005bc25dd5 (HEAD)
Author: aynimor <aynimor@outlook.com>
Date:   Wed Jun 29 22:46:55 2022 +0800

    new branch first commit

commit 3350f9bde39686e736959f5c69cc5aa437032617 (master)
Author: aynimor <aynimor@outlook.com>
Date:   Wed Jun 29 22:44:49 2022 +0800

    1st branch commit
root@localhost:~/git-branch$
```

HEAD 指针这次指向了某次具体的提交，而不再是某个分支。如果有需要在该提交下修改内容并提交，可以通过以下命令将当前游离的分支变为一个具体的提交

```shell
git switch -c <new-branch-name>
git checkout -b <new-branch-name>
```

这种方式有一个较为具体的用法：

如果一个分支被删除，但是这个分支的内容并没有被合并到主分支，这时我们就可以 checkou 到那个分支的最后一次提交的 commit 对象，然后 通过上述命令将该分支恢复

``` shell
root@localhost:~/git-branch$ git checkout master
root@localhost:~/git-branch$ git branch -D new-branch
root@localhost:~/git-branch$ git log
commit 3350f9bde39686e736959f5c69cc5aa437032617 (HEAD -> master)
Author: aynimor <aynimor@outlook.com>
Date:   Wed Jun 29 22:44:49 2022 +0800

    1st branch commit
root@localhost:~/git-branch$ git checkout 52b1d5bc6
Note: switching to '52b1d5bc6'.
Omit...
root@localhost:~/git-branch$ git checkout -b new-branch-recover
Switched to a new branch 'new-branch-recover'
root@localhost:~/git-branch$ git log
commit 52b1d5bc65c6764867fbfb092980eb07d795eb37 (HEAD -> new-branch-recover)
Author: aynimor <aynimor@outlook.com>
Date:   Wed Jun 29 22:58:32 2022 +0800

    2nd new-branch commit

commit 5a5a7209702b78f3464e2cc73d5bd2005bc25dd5
Author: aynimor <aynimor@outlook.com>
Date:   Wed Jun 29 22:46:55 2022 +0800

    new branch first commit

commit 3350f9bde39686e736959f5c69cc5aa437032617 (master)
Author: aynimor <aynimor@outlook.com>
Date:   Wed Jun 29 22:44:49 2022 +0800

    1st branch commit
root@localhost:~/git-branch$
```

