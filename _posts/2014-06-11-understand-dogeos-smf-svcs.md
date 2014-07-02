---
category: blog
published: true
layout: post
title: DogeOS/SmartOS的系统服务体系
description: 理解一个OS，得从它跑了些什么服务开始
---

## 为什么要理解一个服务器OS的系统服务体系？

因为作为一个服务器OS，实际上与管理员与用户有关系的，能感受到的，主要就是这些跑在后台的服务。

打个比方，手机OS不刷不死心人，整天关心的就是后台都跑了多少服务，应该怎么杀。那么对比服务器OS折腾星人，不了解后台到底都是跑的些啥，显然不专业。（当然就不要想着杀了，会杀出问题来。）

所以，想要显摆自己有多精通一个服务器OS，不入门到精通这些后台服务是不行的。

DogeOS目前本质上就是SmartOS，所以以下的内容，也可以用于辅助理解SmartOS或者其它illumos派生的操作系统的系统服务体系。

## 从SMF框架说起

DogeOS/SmartOS继承了illumos的衣钵，系统服务组织上，采用了一种叫做SMF的先进框架。因此我们要先理解什么是SMF？

从维基百科上可以找到[这东西的页面](http://en.wikipedia.org/wiki/Service_Management_Facility)，摘录并翻译如下

> Service Management Facility (SMF) 是Solaris操作系统引入的一项功能，它能创建可维护的，统一的服务与服务管理模型，它取代了init.d。

看到这里我相信大家都有一个呼声：

**能不能说人话？**

好吧说人话。

直白的说就是Solaris的那帮开发者（现在是illumos的开发者）看unix/linux的init.d十分不爽，就另行开发了一个系统取代之。这个系统称之为SMF，这个系统将解决一些在*nix系统服务里面积累多年的问题。

## *nix系统服务到底有啥问题？

老实说，这不是一个很容易让人意识到其重要性的问题。因为大部分用户一辈子，需要处理这东西几率不高。

但是做为一个系统管理员是不可以不理会它的。

Unix系统，以及大部分的linux系统，使用的系统服务框架为[init](http://en.wikipedia.org/wiki/Init)。简单的总结，init是*nix系统在内核启动之后，所启动的第一个应用程序，由它再来负责启动之后的所有的应用程序与服务。所以init做为一个程序，它的运作模式就定义了系统服务框架的机制。

形象的说，*nix系统的启动是这样的

1. 当按下机箱上电源键之后，机器会傻傻的直接执行储存在主板上的应用程序（即BIOS）;
2. BIOS一根筋的检测完硬件之后，去启动主硬盘的引导区的那个引导程序。现在应用的最广泛的引导程序叫做grub（除开MacOS和Windows外）;
3. grub会做一些初始化文件系统的脏活儿，读取一个配置文件，去启动真正想要的内核;
4. 内核在这个时候被载入内存启动了。跟前面步骤的那些应用程序执行完了就把自己杀掉不同，内核不会杀掉自己。它会把你的机器硬件弄成可以共享的形式，欢迎后面的应用程序来共享使用;
5. 然后内核就不直接过问具体的凡间事物（或者说user land），它会启动一个新的应用程序，来负责启动其他的应用程序。

最后那个应用程序就是init。

作为所有后面应用程序的入口，大家应该都意识到了，这是流量的入口啊，怎么管理好这个入口有大学问。比如怎么雁过拔毛，收收买路钱，甚至强迫别人内置几个广告的事情，这事情大家应该都心里有数，我明白的。

但是init是70年代就诞生的，那时候人心还挺淳朴，所以init也特别的头脑简单。

init定下了如下的规则

**所有需要让我启动的程序，排成不同运行等级（run level）的队伍，按照队伍级别高低和名字顺序，一个一个来**

有入口的地方就有江湖，谁先谁后怎么插队一直是人民群众特别关心的问题。在init的简单规则下，想插队，只能调高自己的等级，或者改个能排在前面的名字。

显然这一套能运作30多年就是个奇迹。

想一想，这一套至少会有这些问题

1. 等级划分太简单。*nix一般就6个运行等级，其中level 0和6有特殊含义，level 4被保留了。能用就1，3和5。特权阶级和非特权阶级通通挤在一起，有啥事儿不能听领导的再行动（即依赖关系）。
2. 一个一个来，那万一有人耍赖怎么办？比如网络方面的服务只要排在tcp/ip相关的服务后面即可以，但是文件系统排在更前面，它要是出点问题还要影响网络这边的事儿。
3. 都是用root来执行的。这个对一些关键服务还可以说，但是什么打印机啊之类的有必要么？人家黑了你打印机服务就顺带端掉了整个系统。
4. init启动过一次就不管了，之后这些服务就自生自灭，这可不好，至少得有个自动重启什么的增值服务吧。

## 回到SMF

SMF的发明者将*nix的问题总结了一下，创造了SMF这个init的替代者，它有如下特点

1. 引入依赖关系。一个服务可以依赖另几个服务，满足不了条件就不起动或者暂停。
2. 引入权限控制。打印机服务这种让[linus都抓狂的服务](http://lkml.iu.edu//hypermail/linux/kernel/1306.1/00108.html)，就让它在低权限跑去吧。
3. 引入守护机制。死了帮你重启，重启不成就帮你暂停，以及暂停其他所有依赖这个服务。
4. 用基于xml的配置文件，取代了简单的使用init.rc*文件夹以及文件名来组织各个服务。这个不用解释，因为init的那一套只能说是太原始了。

**扯这么多概念，整点实用的行不行？**

行。

先来三板斧

### 查看系统当前运行的服务

输入`svcs`.

```bash
$ svcs
STATE          STIME    FMRI
...
online         Sep_22   svc:/system/svc/restarter:default
online         Sep_22   svc:/system/filesystem/autofs:default
online         Sep_22   svc:/system/system-log:default
online         Sep_22   svc:/network/smtp:sendmail
online         Sep_22   svc:/system/filesystem/local:default
online         Sep_22   svc:/network/ssh:default
online         Sep_22   svc:/system/dumpadm:default
online         Sep_22   svc:/network/loopback:default
```

类似这样的结果，主要也就是看个状态，`online/offline/maintenance`三种，看字面也知道除了`online`，多半都是不太好的结果，需要果断进行处理。

常用的处理的方式有

* `svcadm disable <FMRI>` 禁用
* `svcadm enable <FMRI>` 启用

### 查看有问题服务的日志文件

具体某个服务的日志文件，一般是位于`/var/svc/log/<FMRI>.log`里面。

例如`svc:/system/filesystem/autofs:default`的日志文件就在`/var/svc/log/svc:/system/filesystem/autofs:default.log`。

出现什么问题的服务，多看看日志文件，才能定位问题。

### 查看服务的依赖与被依赖关系

既然存在依赖关系，那么查询出来总会对定位解决问题有好处，这方面的命令常用的有

* `svcs -d <FMRI>` 查询服务依赖的所有服务
* `svcs -D <FMRI>` 查询依赖服务的所有服务

举个例子

```bash
$ svcs -d milestone/multi-user
STATE          STIME    FMRI
disabled       Sep_22   svc:/network/smtp:sendmail
online         Sep_22   svc:/milestone/name-services:default
online         Sep_22   svc:/milestone/single-user:default
online         Sep_22   svc:/system/filesystem/local:default
online         Sep_22   svc:/network/rpc/bind:default
online         Sep_22   svc:/milestone/sysconfig:default
online         Sep_22   svc:/system/utmp:default
online         Sep_22   svc:/network/inetd:default
online         Sep_22   svc:/network/nfs/client:default
online         Sep_22   svc:/system/system-log:default
```

顺藤摸瓜去查找服务以及所有依赖的服务的问题，就是系统管理员通常要做的事情。

实际上，这方面的还有更多的技巧，这里[有一片文章解释的很清楚](http://www.oreillynet.com/lpt/a/6542)。

## 理解DogeOS的服务体系

明白了DogeOS所用的服务系统的机制，接下来就是具体的看看这个服务体系到底是怎样的。

实际上，在SMF的支撑下，DogeOS以及SmartOS的服务体系发展的异常的强大。支持系统的，不再是一些独立的互不相干的服务，而是一张服务的网。

[DogeOS Smf Browser](images/DogeOS-Smf-Browser.png)

上图即展现了这张网。实际上，它是截图自[这个工具](http://www.dogeos.net/smfgraph/)。它是一个互动的工具，可以用来形象的展现DogeOS的系统服务网络。

例如要查询一下`svc:/milestone/single-user:default`这个服务所依赖的全部服务，只需要在左边的下拉框里面找到这个FMRI就可以了。一目了然，我们就可以知道要保证其成功，需要一些什么样的服务作为先导。

实际上，所有带有milestone关键字的服务，是一种特殊的服务。它们实际上模拟了init系统里面的level的概念。在这个对应关系里，`svc:/milestone/single-user:default`就相当于init的level 1，是单用户模式，而`svc:/milestone/multi-user:default`就相当于init的level 3，是多用户模式。

还有一个问题

### 一个服务具体是如何定义的呢？

它实际上是由一个叫做SMF manifest的XML文件来定义的。这些文件主要位于`/etc/svc/manifest`目录中。这个目录中又按照不同的类别分成了若干个子目录。例如`svc:/milestone/single-user:default`实际上就位于milestone子目录中。透过名字上的联系，相信也很容易猜测到组织这些文件的一些习惯。

找到了定义服务的XML文件，再来解读它，就变得容易了。关于Manifest的具体怎么写，如何读懂它，可以参考这个[规范](http://www.oracle.com/technetwork/server-storage/solaris10/solaris-smf-manifest-wp-167902.pdf)。

Manifest通常会定义一些start/stop/restart的命令，其内容又通常会指向一个shell脚本，顺着这条线，我们就不难搞清楚这个服务到底做了一些什么事情。

## 总结 or TL;DR

DogeOS/SmartOS，以及类illumos的OS，其服务体系采用了SMF机制，是一个具有依赖关系的服务网络。理解这些后台服务，需要从它们之间的依赖关系开始着手，通过查看Manifest定义文件，理解它们，以及试图尝试解决它们的问题。

**BTW:**

Linux的各个发行版，类似于SMF机制也引入了不同的init替代品，这其中最有名的，即是RedHat主导的[systemd](http://en.wikipedia.org/wiki/Systemd)，以及Ubuntu中所采用的[upstrart](http://en.wikipedia.org/wiki/Upstart)。
