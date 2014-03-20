---
layout: post
title: git的存储
description: git的存储，基本上就是把所有的东西都存了一遍。为啥git要这么笨？
category: blog
published: true
---

git的存储，基本上就是把所有的东西都存了一遍。

## 具体的细节

可以见这里 [Git-Internals-Git-Objects](http://git-scm.com/book/en/Git-Internals-Git-Objects)

## 懒得看细节

可以类似如下做一下简单验证

### 准备工作

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

至此我们有了一个简单的repo，两次commit。

### 接下来我们找出来git到底把文件存在哪里了

首先通过git log找到commit的sha

```
[liyu@hk153 ~/DevCamp/gittest]$ git log
commit f5bf4f39d9d0d86164e574850528e43c70f2bb0e
Author: LI Yu <liyu@clustertech.com>
Date:   Mon Mar 17 12:12:28 2014 +0800

    step 2

commit 5757f1fb992f9b8a13d2b3cbe426b48134777bb1
Author: LI Yu <liyu@clustertech.com>
Date:   Mon Mar 17 12:12:09 2014 +0800

    init
```

可见 Commit 1的sha是f5bf4f39d9d0d86164e574850528e43c70f2bb0e， Commit 2的sha是5757f1fb992f9b8a13d2b3cbe426b48134777bb1

再通过commit的sha找到记录commit的文件，看看具体记录了些啥

```bash
[liyu@hk153 ~/DevCamp/gittest]$ git cat-file -p 5757f1fb992f9b8a13d2b3cbe426b48134777bb1
tree 7f2dbfa479cbe99062de2ef82b713f044c4406d8
author LI Yu <liyu@clustertech.com> 1395029529 +0800
committer LI Yu <liyu@clustertech.com> 1395029529 +0800

init
[liyu@hk153 ~/DevCamp/gittest]$ git cat-file -p f5bf4f39d9d0d86164e574850528e43c70f2bb0e
tree 82e61d0a3922801e92d6912c009168345c268353
parent 5757f1fb992f9b8a13d2b3cbe426b48134777bb1
author LI Yu <liyu@clustertech.com> 1395029548 +0800
committer LI Yu <liyu@clustertech.com> 1395029548 +0800

step 2
```

Commit 1的文件记录在sha为7f2dbfa479cbe99062de2ef82b713f044c4406d8文件里，Commit 23的文件记录在sha为82e61d0a3922801e92d6912c009168345c268353文件里。

根据这些信息，继续查找具体文件的sha编码

```bash
[liyu@hk153 ~/DevCamp/gittest]$ git cat-file -p 7f2dbfa479cbe99062de2ef82b713f044c4406d8
100644 blob cc628ccd10742baea8241c5924df992b5c019f71    hello.txt

[liyu@hk153 ~/DevCamp/gittest]$ git cat-file -p 82e61d0a3922801e92d6912c009168345c268353
100644 blob 5ad310361e95be08361d9ecd032ad66506d4c066    hello.txt
```

Commit 1存储的hello.txt的sha编码是cc628ccd10742baea8241c5924df992b5c019f71，Commit 2存储的hello.txt的sha编码是5ad310361e95be08361d9ecd032ad66506d4c066

终于全找到了。

### 把存储的文件打印出来，看看是存的啥

Commit 1的hello.txt文件

```bash
[liyu@hk153 ~/DevCamp/gittest]$ git cat-file -p cc628ccd10742baea8241c5924df992b5c019f71
world
```

Commit 2的hello.txt文件

```bash
[liyu@hk153 ~/DevCamp/gittest]$ git cat-file -p 5ad310361e95be08361d9ecd032ad66506d4c066
world
world2
```

可见，git实际上把两份完整的版本都存储下来了。

## git为啥要这么干？

这样的好处是，逻辑上非常简单，每次commit，只有计算sha和copy文件的过程，没有传统如svn计算diff的过程。速度快，浪费存储。但是对于git的使用场景，存储通常都不是问题。

**原则：简单优先，别做多余的事情。**

git按照sha编码存放文件，具体存在哪里？ 随便翻翻.git/objects/就知道了

```bash
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

两级文件夹结构，并且充分利用的sha编码的独特性（前几位重复的概率很小），用目录系统就完成了index。

**再次：简单优先，别做多余的事情。**

随便vi其中一个文件辨认一下，可以发现文件是经过压缩的。更进一步，在objects文件夹文件过多时，时间上最古老的文件，会被打包到一起（放进pack目录）。需要使用 时，再来查找加解压缩。所以git是借助了压缩文件来节省存储，最终给人的感觉，也没有浪费多少空间。

## Side Effect

此外，git的这种结构的好处是，任意两个commit的diff，只需要计算一次（把两个文件拿到临时目录做diff即 可）。不像svn，如果commit间隔太长，要累加所有途中的diff，这个过程不仅仅慢，而且有些情况可能不太可能 （例如复杂branch的情况，累加diff会有conflict）。对于Linus或者开发者，这个好处就是可以变相的鼓励fork和开branch，实现各种疯狂的不疯狂的想法。因为不管你改成什么样的乱七八糟，最终都可以很快和安全的知道你改了什么，以及做可能的merge（例如github的pull request），这是传统svn在大规模应用时很难做到的。

## 八卦

git这个东西，Linus本人觉得比linux kernel做的更好，他自己相当得意（不服的可以去kernel mail list还是什么地方吐个槽[呼唤他](http://thread.gmane.org/gmane.comp.version-control.git/57643/focus=57918)）。其编写过程中的一些思路方法（比如简单优先，不做重复的事情）也值得我们借鉴。

就算你不用git，有时间也可以看看这个 [Youtube Talk](https://www.youtube.com/watch?v=4XpnKHJAok8) Linus本人讲git，很有启发。
