---
layout: post
published: true
title: git的存储
---

{{ page.title }}
================

git的存储，基本上就是把所有的东西都存了一遍。

具体的细节可以见这里

http://git-scm.com/book/en/Git-Internals-Git-Objects

懒得看细节的，可以类似如下做一下简单验证

1. 准备工作

```bash
mkdir gittest
cd gittest
cat world >hello.txt
git init
git add .
git commit -a -m "init"
cat world2 >>hello.txt
git commit -a -m "step2"
```

至此我们有了一个简单的repo，两次commit

2. 接下来我们找出来git到底把文件存在哪里了

首先通过git log找到commit的sha

```
[liyu@hk153 ~/DevCamp/gittest]$ git log
commit f5bf4f39d9d0d86164e574850528e43c70f2bb0e      <== commit 2的sha
Author: LI Yu <liyu@clustertech.com>
Date:   Mon Mar 17 12:12:28 2014 +0800

    step 2

commit 5757f1fb992f9b8a13d2b3cbe426b48134777bb1       <== commit 1的sha
Author: LI Yu <liyu@clustertech.com>
Date:   Mon Mar 17 12:12:09 2014 +0800

    init
```

通过commit的sha找到记录commit的文件，看看具体记录了些啥

```
[liyu@hk153 ~/DevCamp/gittest]$ git cat-file -p 5757f1fb992f9b8a13d2b3cbe426b48134777bb1
tree 7f2dbfa479cbe99062de2ef82b713f044c4406d8            <== commit 1的文件记录在这里面
author LI Yu <liyu@clustertech.com> 1395029529 +0800
committer LI Yu <liyu@clustertech.com> 1395029529 +0800

init
[liyu@hk153 ~/DevCamp/gittest]$ git cat-file -p f5bf4f39d9d0d86164e574850528e43c70f2bb0e
tree 82e61d0a3922801e92d6912c009168345c268353            <== commit 2的文件记录在这里面
parent 5757f1fb992f9b8a13d2b3cbe426b48134777bb1
author LI Yu <liyu@clustertech.com> 1395029548 +0800
committer LI Yu <liyu@clustertech.com> 1395029548 +0800

step 2
```

按照commit 1和2的文件记录，继续查找具体文件的sha编码

```
[liyu@hk153 ~/DevCamp/gittest]$ git cat-file -p 7f2dbfa479cbe99062de2ef82b713f044c4406d8
100644 blob cc628ccd10742baea8241c5924df992b5c019f71    hello.txt    <== commit 1存储的hello.txt的 sha编码

[liyu@hk153 ~/DevCamp/gittest]$ git cat-file -p 82e61d0a3922801e92d6912c009168345c268353
100644 blob 5ad310361e95be08361d9ecd032ad66506d4c066    hello.txt    <== commit 2存储的hello.txt的 sha编码
```

3. 按照git存储的指示，把存储的文件打印出来，看看是存的啥

commit 1的文件

```
[liyu@hk153 ~/DevCamp/gittest]$ git cat-file -p cc628ccd10742baea8241c5924df992b5c019f71
world
```

commit 2的文件

```
[liyu@hk153 ~/DevCamp/gittest]$ git cat-file -p 5ad310361e95be08361d9ecd032ad66506d4c066
world
world2
```

可见，git实际上把两份完整的版本都存储下来了。

这样的好处是，逻辑上非常简单，每次commit，只有计算sha和copy文件的过程，没有传统如svn计算diff的过程。

速度快，浪费存储，但是对于git的使用场景，存储通常都不是问题。

这些文件git先存在.git/objects里面，随便找找就知道了

```
[liyu@hk153 ~/DevCamp/gittest]$ find .git/objects/
.git/objects/
.git/objects/info
.git/objects/pack
.git/objects/57
.git/objects/57/57f1fb992f9b8a13d2b3cbe426b48134777bb1
.git/objects/cc
.git/objects/cc/628ccd10742baea8241c5924df992b5c019f71
.git/objects/f5
.git/objects/f5/bf4f39d9d0d86164e574850528e43c70f2bb0e
.git/objects/7f
.git/objects/7f/2dbfa479cbe99062de2ef82b713f044c4406d8
.git/objects/5a
.git/objects/5a/d310361e95be08361d9ecd032ad66506d4c066
.git/objects/82
.git/objects/82/e61d0a3922801e92d6912c009168345c268353
```

两级文件夹结构，则是充分利用的sha编码的独特性，用文件系统就完成了index，再次没有做不必要的事情。

随便vi其中一个，可以发现是经过压缩的。在objects文件夹过大时，最老的文件，会被打包到一起。需要使用 时，再来查找加解压缩。所以其实git是借助了压缩文件来节省存储，不做不必要的事情。

此外，git的这种结构的好处是，任意两个commit的diff，只需要计算一次（把两个文件拿到临时目录做diff即 可）。不像svn，如果commit间隔太长，要累加所有途中的diff，这个过程不仅仅慢，而且有些情况可能不太可能 （例如复杂branch的情况，累加diff会有conflict）。对于linus或者开发者，这个好处就是可以变相的鼓励fork 和开branch，因为不管你改成什么样的乱七八糟，最终我都可以很快和安全的知道你改了什么，以及做可能的 merge（例如github的pull request），这是传统svn在大规模应用时很难做到的（大规模的svn基本上只鼓励一两 个branch）。

BTW: git这个东西，linus本人是觉得甚至比linux kernel做的更好的东西，他自己也比较得意。其编写过程中的 一些思路方法（比如简单优先，不做重复的事情）也值得我们借鉴。大家有时间可以看看这个talk https://www.youtube.com/watch?v=4XpnKHJAok8，linus本人讲git，很有启发。
