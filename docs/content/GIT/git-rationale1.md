## 说明

本系列博客主要记录工具 GIT 的一些命令，及其引起的仓库文件内容的变化

## 初始化仓库

在一个空文件夹执行 `git init` 命令，可以将当前文件夹设定为一个 GIT 仓库，此时文件夹内会生成一个 .git 文件夹，文件结构如下

```shell
root@localhost:~/git-demo# git init
Initialized empty Git repository in /root/git-demo/.git/
root@localhost:~/git-demo# tree -al
.
└── .git/
    ├── HEAD
    ├── config
    ├── description
    ├── hooks/
    │   ├── applypatch-msg.sample
    │   ├── commit-msg.sample
    │   ├── fsmonitor-watchman.sample
    │   ├── post-update.sample
    │   ├── pre-applypatch.sample
    │   ├── pre-commit.sample
    │   ├── pre-merge-commit.sample
    │   ├── pre-push.sample
    │   ├── pre-rebase.sample
    │   ├── pre-receive.sample
    │   ├── prepare-commit-msg.sample
    │   ├── push-to-checkout.sample
    │   └── update.sample
    ├── info/
    │   └── exclude
    ├── objects/
    │   ├── info/
    │   └── pack/
    └── refs/
        ├── heads/
        └── tags/

9 directories, 17 files
root@localhost:~/git-demo#
```

包含 .git 目录在内，`init` 命令共创建了 9 个文件夹和 17 个文件，为了便于后续理解，暂时将不会使用到的 hooks 文件夹内的 *.sample 文件清空。

```shell
root@localhost:~/git-demo# rm -rf .git/hooks/*.sample
root@localhost:~/git-demo# tree -al
.
└── .git/
    ├── HEAD
    ├── config
    ├── description
    ├── hooks/
    ├── info/
    │   └── exclude
    ├── objects/
    │   ├── info/
    │   └── pack/
    └── refs/
        ├── heads/
        └── tags/

9 directories, 4 files
root@localhost:~/git-demo#
```

首先了解一下 .git/config 文件。

```shell
root@localhost:~/git-demo# cat .git/config
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
root@localhost:~/git-demo#
```

.git/config 文件存储的是当前 git 仓库的一些配置信息，是区别于全局（global）配置文件的配置信息。全局配置文件一般处于用户家目录下的 .gitconfig 文件，一般通过 `git config --global user.email <my@email.com>` 等命令指定。

```shell
root@localhost:~/git-demo# git config --global user.name "aynimor"
root@localhost:~/git-demo# git config --global user.email "aynimor@outlook.com"
root@localhost:~/git-demo# cat ~/.gitconfig
[init]
        defaultBranch = main
[user]
        email = aynimor@outlook.com
        name = aynimor
root@localhost:~/git-demo#
```

如果使用 `config` 命令时，未指定 `--global` 参数，则配置信息会记录到当前仓库的 config 文件，此时 git 会优先使用仓库本地的配置信息

```shell
root@localhost:~/git-demo# git config user.name "demo-user"
root@localhost:~/git-demo# git config user.email "demo@email.com"
root@localhost:~/git-demo# cat .git/config
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
[user]
        name = demo-user
        email = demo@email.com
root@localhost:~/git-demo#
```

## 1. `git add` 命令

首先通过命令 `echo "hello git" > first.txt` 在当前目录下新增一个 first.txt 文件。

执行 `tree -al` 命令会发现，.git 目录本身没有发生任何改变，只是在当前文件夹下，多出了一个 first.txt 文件名的文件。当前目录与 .git 目录同级的文件夹或文件所在的位置被称为工作区。工作区内文件的变化，并不会影响到 git 仓库本身，只有在执行某些命令后，`git` 才会把工作区里的内容和 git 仓库本身去做比对，然后做出相应的处理。

执行命令 `git status` 可以查看当前仓库的信息

```shell
root@localhost:~/git-demo# git status
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        first.txt

nothing added to commit but untracked files present (use "git add" to track)
root@localhost:~/git-demo#
```

此时 git 发现当前新增的文件并没有在仓库中（如何发现的会在了解 index 文件时说明），执行命令 `git add first.txt` 将 first.txt 文件添加到 git 仓库

```shell
root@localhost:~/git-demo# git add first.txt
root@localhost:~/git-demo# git status
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   first.txt

root@localhost:~/git-demo# git add first.txt
root@localhost:~/git-demo#
```

然后查看 git 仓库的变化

```shell
root@localhost:~/git-demo# tree -al
.
├── .git
│   ├── HEAD
│   ├── config
│   ├── description
│   ├── hooks
│   ├── index
│   ├── info
│   │   └── exclude
│   ├── objects
│   │   ├── 8d
│   │   │   └── 0e41234f24b6da002d962a26c2495ea16a425f
│   │   ├── info
│   │   └── pack
│   └── refs
│       ├── heads
│       └── tags
└── first.txt

10 directories, 7 files
root@localhost:~/git-demo#
```

在 .git 文件夹下，多出来了一个文件夹 .git/objects/8d 和两个文件 .git/index、.git/objects/8d/0e41234f24b6da002d962a26c2495ea16a425f。

GIT 提供了命令 `git cat-file` 来查看 objects 文件夹下的文件信息。

```shell
root@localhost:~/git-demo# git cat-file -t 8d0e41
blob
root@localhost:~/git-demo#
```

控制台输出了 blob 字符，这是 git 几种文件对象类型中的 `blob` 类型，是 git 用于存储文件内容和文件类型等其他信息的一种对象，后续将会介绍其他几种。 命令中的 `-t` 指明要查看文件对象的类型，`8d0e41234` 为 git 文件对象的哈希码，既 8d 文件夹及 0e412... 文件名，不过一般不用输入全称，只用指定前几个字符，能定位到唯一文件对象即可。

执行 `git cat-file -p 8d0e41` 查看文件的内容

```shell
root@localhost:~/git-demo# git cat-file -p 8d0e41
hello git
root@localhost:~/git-demo#
```

`blob` 类型并不存储文件的名称，可以通过以下方式作为验证，相同的文件内容，并不会导致仓库内新增 blob 文件

```shell
root@localhost:~/git-demo# echo "hello git" > tmp.txt
root@localhost:~/git-demo# git add tmp.txt
root@localhost:~/git-demo# tree -al
.
├── .git
│   ├── HEAD
│   ├── config
│   ├── description
│   ├── hooks
│   ├── index
│   ├── info
│   │   └── exclude
│   ├── objects
│   │   ├── 8d
│   │   │   └── 0e41234f24b6da002d962a26c2495ea16a425f
│   │   ├── info
│   │   └── pack
│   └── refs
│       ├── heads
│       └── tags
├── first.txt
└── tmp.txt

10 directories, 8 files
root@localhost:~/git-demo#
```

git 生成文件对象的方式是通过对内容 `blob <len>\0<content>` 拼接的方式来生成的，`<len>` 为当前工作区文件的长度，`<content>` 为当前工作区文件的内容，然后通过 `sha1` 的加密方式生成哈希来命名文件对象。

**python 代码示例**

```python
import hashlib

with open("~/git-demo/first.txt", "r") as f:
    data = f.read()

sha1 = hashlib.sha1()
sha1.update("blob {length}\0".format(length=len(data)).encode())
sha1.update(data.encode())

assert h.hexdigest() == "8d0e41234f24b6da002d962a26c2495ea16a425f"
```

既然 blob 文件对象不存储文件名，则 git 必然是通过其他的方式来获取文件是否被添加到索引区的信息的，那就是前面提到过的 .git/index 文件，一般被称为索引区（暂存区）。

通过命令 `git ls-files` 可以查看索引区的文件列表，使用 `-s` 参数可以列出明细

```shell
root@localhost:~/git-demo# git ls-files -s
100644 8d0e41234f24b6da002d962a26c2495ea16a425f 0       first.txt
100644 8d0e41234f24b6da002d962a26c2495ea16a425f 0       tmp.txt
root@localhost:~/git-demo#
```

可以看到展示了一些信息，第一个数 `100644` 是该文件的权限，`8d0e41234f24b6da002d962a26c2495ea16a425f` 是该文件所对应的 `blob` 对象的名称，`0` 是暂存号（这个暂时没搞懂），最后一个就是文件名称（包含路径信息）。

```shell
root@localhost:~/git-demo$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   first.txt
        new file:   tmp.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        second.txt

root@localhost:~/git-demo$ git add .
root@localhost:~/git-demo$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   first.txt
        new file:   second.txt
        new file:   tmp.txt

root@localhost:~/git-demo$ git ls-files -s
100644 8d0e41234f24b6da002d962a26c2495ea16a425f 0       first.txt
100644 e019be006cf33489e2d0177a3837a2384eddebc5 0       second.txt
100644 8d0e41234f24b6da002d962a26c2495ea16a425f 0       tmp.txt
root@localhost:~/git-demo$ tree -al
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
│   ├── objects
│   │   ├── 8d
│   │   │   └── 0e41234f24b6da002d962a26c2495ea16a425f
│   │   ├── e0
│   │   │   └── 19be006cf33489e2d0177a3837a2384eddebc5
│   │   ├── info
│   │   └── pack
│   └── refs
│       ├── heads
│       └── tags
├── first.txt
├── second.txt
└── tmp.txt

12 directories, 10 files
```

这也是 GIT 能够了解到当前文件是 `modified` 或者 `Untracked files` 等其他状态的原因。

> git add 命令会将当前工作区内的文件添加到索引区（暂存区）中

## 2. `git commit` 命令

作为 `git add` 的后续命令，`git commit` 的作用就是将索引区中的文件添加到仓库区。

``` shell
root@localhost:~/git-demo$ git commit -m "1st commit"
[master (root-commit) 8aa7a23] 1st commit
 3 files changed, 3 insertions(+)
 create mode 100644 first.txt
 create mode 100644 second.txt
 create mode 100644 tmp.txt
root@localhost:~/git-demo$ tree -al
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
│   │   ├── 39
│   │   │   └── b241055a572589082e0b3513c43b8f466d6926
│   │   ├── 8a
│   │   │   └── a7a236bd999eb0e2337abe6ba927bd64d6bf34
│   │   ├── 8d
│   │   │   └── 0e41234f24b6da002d962a26c2495ea16a425f
│   │   ├── e0
│   │   │   └── 19be006cf33489e2d0177a3837a2384eddebc5
│   │   ├── info
│   │   └── pack
│   └── refs
│       ├── heads
│       │   └── master
│       └── tags
├── first.txt
├── second.txt
└── tmp.txt

17 directories, 16 files
root@localhost:~/git-demo$
```

执行命令后仓库中多出了较大的变化，其中 objects 中新增了两个对象，新增了一个 logs 文件夹，其中多出了两个文件夹和两个文件，refs中也多出了一个文件，最外层还多出了一个 COMMIT_EDITMSG 的文件

``` SHELL
root@localhost:~/git-demo$ cat .git/COMMIT_EDITMSG
1st commit
root@localhost:~/git-demo$
```

COMMIT_EDITMSG 内容是 `commit` 时填写的提交信息

```shell
root@localhost:~/git-demo$ cat .git/refs/heads/master
8aa7a236bd999eb0e2337abe6ba927bd64d6bf34
root@localhost:~/git-demo$ git cat-file -t 8aa7a236bd999eb0e2337abe6ba927bd64d6bf34
commit
root@localhost:~/git-demo$ git cat-file -p 8aa7a236bd999eb0e2337abe6ba927bd64d6bf34
tree 39b241055a572589082e0b3513c43b8f466d6926
author aynimor <aynimor@outlook.com> 1652105605 +0800
committer aynimor <aynimor@outlook.com> 1652105605 +0800

1st commit
root@localhost:~/git-demo$ git cat-file -t 39b241055a572589082e0b3513c43b8f466d6926
tree
root@localhost:~/git-demo$ git cat-file -p 39b241055a572589082e0b3513c43b8f466d6926
100644 blob 8d0e41234f24b6da002d962a26c2495ea16a425f    first.txt
100644 blob e019be006cf33489e2d0177a3837a2384eddebc5    second.txt
100644 blob 8d0e41234f24b6da002d962a26c2495ea16a425f    tmp.txt
```

refs/heads/master 文件内容为一个对象的 sha1 值，我们查看该对象的信息，发现对象名称是 `commit`，内容为本次 `commit` 的一些信息，其中第一行是一个 `tree` 加上一个对象的 sha1 值，我们通过 `git cat-file` 查看，发现是一个名称为 `tree` 的对象，值为一个 `blob` 对象的列表及其名称，与索引区文件 `index` 十分相似。

再次新增文件一个文件并新增一次提交，来观察 `tree` 对象内容

``` shell
root@localhost:~/git-demo$ echo "tree content" >> third.txt
root@localhost:~/git-demo$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        third.txt

nothing added to commit but untracked files present (use "git add" to track)
root@localhost:~/git-demo$ git add third.txt
root@localhost:~/git-demo$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   third.txt

root@localhost:~/git-demo$ git ls-files -s
100644 8d0e41234f24b6da002d962a26c2495ea16a425f 0       first.txt
100644 e019be006cf33489e2d0177a3837a2384eddebc5 0       second.txt
100644 03f227e91ff2bda8627c209f207b6446acd6d2f5 0       third.txt
100644 8d0e41234f24b6da002d962a26c2495ea16a425f 0       tmp.txt
root@localhost:~/git-demo$ git commit -m "2nd commit"
[master 7dbfdcb] 2nd commit
 1 file changed, 1 insertion(+)
 create mode 100644 third.txt
root@localhost:~/git-demo$ tree -al
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
│   │   ├── 03
│   │   │   └── f227e91ff2bda8627c209f207b6446acd6d2f5
│   │   ├── 27
│   │   │   └── 5aa288c5b126d5f07c6e6267affe58110713b5
│   │   ├── 39
│   │   │   └── b241055a572589082e0b3513c43b8f466d6926
│   │   ├── 7d
│   │   │   └── bfdcbd17d4c4608a242256269334f037ee9251
│   │   ├── 8a
│   │   │   └── a7a236bd999eb0e2337abe6ba927bd64d6bf34
│   │   ├── 8d
│   │   │   └── 0e41234f24b6da002d962a26c2495ea16a425f
│   │   ├── e0
│   │   │   └── 19be006cf33489e2d0177a3837a2384eddebc5
│   │   ├── info
│   │   └── pack
│   └── refs
│       ├── heads
│       │   └── master
│       └── tags
├── first.txt
├── second.txt
├── third.txt
└── tmp.txt

20 directories, 20 files
root@localhost:~/git-demo$
```

可以发现，又再次新增三个对象，查看其类型分别是一个 `blob` 对象，一个 `commit` 对象 一个`tree` 对象

```shell
root@localhost:~/git-demo$ git cat-file -t 03f227
blob
root@localhost:~/git-demo$ git cat-file -t 275aa2
tree
root@localhost:~/git-demo$ git cat-file -t 7dbfdc
commit
root@localhost:~/git-demo$ git cat-file -p 7dbfdcbd17d4c4608a242256269334f037ee9251
tree 275aa288c5b126d5f07c6e6267affe58110713b5
parent 8aa7a236bd999eb0e2337abe6ba927bd64d6bf34
author aynimor <aynimor@outlook.com> 1652106658 +0800
committer aynimor <aynimor@outlook.com> 1652106658 +0800

2nd commit
root@localhost:~/git-demo$
```

查看 `commit` 对象的内容发现，比起第一次提交，多出了一行 `parent` 的对象信息，而这就是上一次提交的 `commit` 对象的 sha1 的值。

```shell
root@localhost:~/git-demo$ git cat-file -p 275aa2
100644 blob 8d0e41234f24b6da002d962a26c2495ea16a425f    first.txt
100644 blob e019be006cf33489e2d0177a3837a2384eddebc5    second.txt
100644 blob 03f227e91ff2bda8627c209f207b6446acd6d2f5    third.txt
100644 blob 8d0e41234f24b6da002d962a26c2495ea16a425f    tmp.txt
root@localhost:~/git-demo$
```

查看 `tree` 对象的内容发现，此次提交我们仅添加了一个 third.txt 文件，但是其他的文件依然添加到了 `tree` 对象的内容中。也就是说，GIT 能通过任何一次 `commit` 和 `tree` 得到当时那次提交的时候仓库中所有文件的状态，这也是 GIT 能进行版本管理的原因。

继续新增一次 `commit` 观察变化

```shell
root@localhost:~/git-demo$ mkdir folder1
root@localhost:~/git-demo$ git status
On branch master
nothing to commit, working tree clean
root@localhost:~/git-demo$ echo "folder content" >> folder1/folder_file.txt
root@localhost:~/git-demo$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        folder1/

nothing added to commit but untracked files present (use "git add" to track)
root@localhost:~/git-demo$ git add folder1/folder_file.txt
root@localhost:~/git-demo$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   folder1/folder_file.txt

root@localhost:~/git-demo$ git ls-files -s
100644 8d0e41234f24b6da002d962a26c2495ea16a425f 0       first.txt
100644 b818c8b099ebb9491b61307e80f30f9dcace841b 0       folder1/folder_file.txt
100644 e019be006cf33489e2d0177a3837a2384eddebc5 0       second.txt
100644 03f227e91ff2bda8627c209f207b6446acd6d2f5 0       third.txt
100644 8d0e41234f24b6da002d962a26c2495ea16a425f 0       tmp.txt
root@localhost:~/git-demo$ git commit -m "3rd commit"
[master 877ed95] 3rd commit
 1 file changed, 1 insertion(+)
 create mode 100644 folder1/folder_file.txt
root@localhost:~/git-demo$ tree -al
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
│   │   ├── 03
│   │   │   └── f227e91ff2bda8627c209f207b6446acd6d2f5
│   │   ├── 27
│   │   │   └── 5aa288c5b126d5f07c6e6267affe58110713b5
│   │   ├── 39
│   │   │   └── b241055a572589082e0b3513c43b8f466d6926
│   │   ├── 6e
│   │   │   └── 4b157ff4e99ffb78577ab095f20d55e4b9b3cb
│   │   ├── 7d
│   │   │   └── bfdcbd17d4c4608a242256269334f037ee9251
│   │   ├── 87
│   │   │   ├── 7ed95e881682e000ea23775796e72475991f2a
│   │   │   └── eaa280093b441807518708d97b0bbb60d0b1e8
│   │   ├── 8a
│   │   │   └── a7a236bd999eb0e2337abe6ba927bd64d6bf34
│   │   ├── 8d
│   │   │   └── 0e41234f24b6da002d962a26c2495ea16a425f
│   │   ├── b8
│   │   │   └── 18c8b099ebb9491b61307e80f30f9dcace841b
│   │   ├── e0
│   │   │   └── 19be006cf33489e2d0177a3837a2384eddebc5
│   │   ├── info
│   │   └── pack
│   └── refs
│       ├── heads
│       │   └── master
│       └── tags
├── first.txt
├── folder1
│   └── folder_file.txt
├── second.txt
├── third.txt
└── tmp.txt

24 directories, 25 files
root@localhost:~/git-demo$
```

这次先新增了一个文件夹，然后在文件中新增文件，在新增文件夹时查看 `git status` 时发现，GIT 并没有感知到新增文件夹，而在添加到索引区后，发现索引区的信息中包含了文件的目录层级，**同时**本次提交一共产生了四个对象，分别是 `blob`、`tree`、`tree`、`commit` 对象，其中一个 `tree` 对象就是新增的文件夹，其内容为文件夹下所有文件的列表及其信息

```shell
root@localhost:~/git-demo$ git cat-file -t 6e4b15
tree
root@localhost:~/git-demo$ git cat-file -t 877ed9
commit
root@localhost:~/git-demo$ git cat-file -t 87eaa2
tree
root@localhost:~/git-demo$ git cat-file -t b818c8
blob
root@localhost:~/git-demo$ git cat-file -p 877ed9
tree 87eaa280093b441807518708d97b0bbb60d0b1e8
parent 7dbfdcbd17d4c4608a242256269334f037ee9251
author aynimor <aynimor@outlook.com> 1652109465 +0800
committer aynimor <aynimor@outlook.com> 1652109465 +0800

3rd commit
root@localhost:~/git-demo$ git cat-file -p 87eaa2
100644 blob 8d0e41234f24b6da002d962a26c2495ea16a425f    first.txt
040000 tree 6e4b157ff4e99ffb78577ab095f20d55e4b9b3cb    folder1
100644 blob e019be006cf33489e2d0177a3837a2384eddebc5    second.txt
100644 blob 03f227e91ff2bda8627c209f207b6446acd6d2f5    third.txt
100644 blob 8d0e41234f24b6da002d962a26c2495ea16a425f    tmp.txt
root@localhost:~/git-demo$ git cat-file -p 6e4b15
100644 blob b818c8b099ebb9491b61307e80f30f9dcace841b    folder_file.txt
root@localhost:~/git-demo$
```

通过以下的图形可以对 `tree`，`commit`，`blob` 三者的关系做一个简单的描述

![image-20220509235208084](/img/image-20220509235208084.png)

查看另一个新增的文件 `refs/heads/master` 可以发现，其内容就是我们最后那次提交新生成的 `commit` 对象的 sha1 的值，同时我们可以发现 `.git/HEAD` 文件的内容也正好是 `refs/heads/master` 文件的路径， `HEAD` 在 GIT 中的含义是分支指针，它永远指向当前工作分支，我们当前工作分支就是在 `master` 分支下，通过 `git branch` 命令可以获取到分支列表，标注 `*` 的即是当前工作的分支。

```shell
root@localhost:~/git-demo$ cat .git/refs/heads/master
7dbfdcbd17d4c4608a242256269334f037ee9251
root@localhost:~/git-demo$ cat .git/HEAD
ref: refs/heads/master
root@localhost:~/git-demo$ git branch
* master
root@localhost:~/git-demo$
```

`./git/logs` 文件夹内的文件记录的是 GIT 仓库变化的一些信息描述

``` shell
root@localhost:~/git-demo$ cat .git/logs/HEAD
0000000000000000000000000000000000000000 8aa7a236bd999eb0e2337abe6ba927bd64d6bf34 aynimor <aynimor@outlook.com> 1652105605 +0800   commit (initial): 1st commit
8aa7a236bd999eb0e2337abe6ba927bd64d6bf34 7dbfdcbd17d4c4608a242256269334f037ee9251 aynimor <aynimor@outlook.com> 1652106658 +0800   commit: 2nd commit
root@localhost:~/git-demo$ cat .git/logs/refs/heads/master
0000000000000000000000000000000000000000 8aa7a236bd999eb0e2337abe6ba927bd64d6bf34 aynimor <aynimor@outlook.com> 1652105605 +0800   commit (initial): 1st commit
8aa7a236bd999eb0e2337abe6ba927bd64d6bf34 7dbfdcbd17d4c4608a242256269334f037ee9251 aynimor <aynimor@outlook.com> 1652106658 +0800   commit: 2nd commit
root@localhost:~/git-demo$
```

