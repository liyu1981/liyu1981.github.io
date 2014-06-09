---
layout: post
title: 说一下DogeOS, SmartOS和illumos
description: 一个真正面向云计算的操作系统以及其来龙去脉
category: blog
published: false
---

## DogeOS, SmartOS, illumos，这些都是什么？

都是操作系统。
都是云计算操作系统。
都是先进的云计算操作系统。
都是小众的先进的云计算操作系统。

~~讲完收工~~

当然不是这么简单 :)

但是也不希望写的太长。以下就尽量言简意垓的把来龙去脉，历史纠纷，以及革命先进性都说清楚。

## 从illumos开始

什么是[illumos](http://www.illumos.org)？简单的讲，illumos是OpenSolaris的后续版本。在万恶的Oracle收购了Sun之后，原有OpenSolaris的开发者基本上都离开了Oracle，组成了illumos社区，继续对OpenSolaris先进的内核进行维护，更新，以及添加nb的新功能。这种事情在Oracle的收购史上似乎一再出现，想想MySQL，OpenOffice，Java，强烈的既视感。

当然，OpenSolaris不是完蛋了，其实还[存活着](http://solaris.java.net/)。不过因为核心开发者的出走，都在illumos那边玩，很多[先进特性](http://www.slideshare.net/ahl0003/illumos-innovations-that-will-never-be-in-oracle-solaris)（尤以存储和虚拟化方面为甚）都不再回流。

illumos继承了所有OpenSolaris的先进特性，例如ZFS，Zone，Crossbow和Dtrace。简单的总结

* ZFS可以说是终极文件系统，提供了卷管理，快照，和所有想得到的nb东西
> 在linux里面，这个功能由LVM以及一堆不同的文件系统完成
* Zone是OS Container，即操作系统隔离，或者叫做轻量级虚拟机
> linux在Zone出现5年之后，也模仿开发了LXC功能
* Crossbow是网络虚拟化，说白了就是将网卡硬件和软件设备分离，一块硬件对应好多个软件设备，并且提供了nb的虚拟switch支持
> linux这部分的成果也基本上进入了内核，在几年之后
* Dtrace是系统管理员的终极武器，无痛在线勘察系统瓶颈的手术刀
> MacOSX借用FreeBSD的成果移植了Dtrace，构成了Xcode中nb闪闪的instruments功能（乔老爷子亲自宣布的），linux这边限于授权协议问题，则由IBM联合RedHat克隆了一份，名为SystemTap

然后，在joyent公司里面[两位神人的努力下](http://www.slideshare.net/bcantrill/experiences-porting-kvm-to-smartos)，KVM功能被移植到了illumos内核环境中，于是illumos也拥有了通常所说的Type 2虚拟化技术。

于是illumos = ZFS + Zone + Crossbow + Dtrace + KVM。所有云计算的关键服务在这里集合，形成了一个为云计算而生的系统内核。

但是需要注意的是illumos只是一个内核（正确的说是，内核OS加网络层(Network)，即ON），在geek的手里，它已经完备。需要放在生产环境，则还欠缺一个包装。

## 然后是SmartOS

怎么把illumos包装成通常意义上的OS，这是接下来需要考虑的问题，也是最好的商业化的契机。

于是就诞生了一批社区和公司围绕着illumos做包装工作，谓之发行版。

第一个明显的方向，即是让GNU，也就是通常意义上我们以为的linux部分（Gnome，KDE，X11等等）跑在illumos内核上，组成一个可以天天使用和开发的平台。这方面诞生了[OpenIndiana](http://www.openindiana.org)，安装使用这个系统，在感官层次，基本上与linux也区别不大。

但是illumos的优势毕竟不是在桌面系统，而是在服务器。所以第二个方向，就是打包成一个可以给数据中心使用的云操作系统。这方面的公司，就有[nexenta](http://www.nexenta.com)，[delphix](http://www.delphix.com)和[joyent](http://www.joyent.com)三家公司。前两者集中火力发挥ZFS存储的威力，对准EMC开炮。joyent则走向了云计算，不仅运营自己的公有云，也提供私有云解决方案。

[SmartOS](http://www.smartos.org)即是joyent在包装illumos方面的答案。joyent将illumos内核和必要的组件和命令行工具，最终打包成为了一个300MB左右的LiveCD，实现了“启动即拥有云计算能力”这个目标。

joyent公司其实更加为人熟知的，莫过于其[nodejs](http://www.nodejs.org)这个平台，所以SmartOS里面的关键命令行工具和组件，都是用nodejs来实现的。用javascript来实现严肃认真的服务器功能，或许这很让人觉得有些不可思议。但是joyent就是这样做的，而且用事实证明，他们干的很不错，实现的效率并不比编译好的二进制程序差。

因为KVM技术是由joyent第一次整合进入illumos，所以SmartOS理所当然成为第一个能提供KVM服务（即运行CentOS，Ubuntu和Windows）的illumos发行版。当然鉴于illumos社区的良好风气，[KVM on illumos](https://github.com/joyent/illumos-kvm)从一开始就是开源项目，最后也进入illumos内核。

SmartOS的特点总结如下

* 继承了illumos所有的特点（ZFS + Zone + Crossbow + Dtrace + KVM）
> linux近年来，在KVM和LXC进入内核之后，也逐步具有了以上功能，但是网络虚拟化和Dtrace部分，则始终差一点
* 集成了joyent所开发的vmadm，imgadm等工具软件，使得虚拟化部分具有了可操作性
> linux系统则是由libvirt+一系列小工具来完成这些功能
* 秉承LiveOS的理念，强调无需安装，将几乎全部的服务器资源留给了租户
> linux方面，这方面的进步则不明显，但是有商业公司进行探讨

## Project FiFo

SmartOS是云计算操作系统的代表，出世之后即引起了大家的注意，开始应用。但是熟知数据中心维护的管理员都知道，安装好方才是梦魇的开始。接下来，怎么维护系统？怎么管理系统的各项资源？怎么进行日常的操作？怎么监控系统？怎么处理系统的故障？怎么升级系统？回答这些问题，方才是系统管理员最终选择一个操作系统的真正考察点。

SmartOS只可以部分的回答上述问题。

joyent是一个商业公司，因此它并没有把所有的东西都放在SmartOS里面让大家免费使用。准确的说，joyent将回答所有上述问题的能力，无私的都放在了SmartOS中。但是体现这些能力的工具，或者以更为熟知的术语，运维管理系统，并没有放出来。joyent的运维管理工具，名为SmartDataCenter，是一个商业产品，需要不菲的授权费用才可以使用。

于是使用SmartOS运营数据中心，就变成如下两种常见的方式：

1. 强者是不需要任何运维工具的，因为既然能力都已经具备，那么实际上所有事情都可以通过一些基本管理工具，以及API的调用完成。换言之，强者会开发自己的运维系统。
2. 不具备开发自己运维系统的，要么就是小规模的使用SmartOS，生产虚拟机之后，即很长时间不去动弹它（SmartOS的稳定性基本可以保证终身不怎么出问题），要么就是去获取SmartDataCetner的授权。

如果有一个开源的运维管理系统就好了？通常我们都会这么想。

运维管理系统的开发，理论上来讲，并不是一个很难的事情（这可能是illumos内核那帮家伙不屑的:)），只是需要付出大量的时间和开发精力。另一方面，这套系统又是运维的核心部分，天然的具有巨大的商业价值，因此不排除有些运营SmartOS的商业公司，开发了这套系统，但是并没有将之开源。

直到[Project FiFo](http://www.project-fifo.net)的出现。

简言之Project FiFo贡献了一套开源SmartOS的运维管理系统，它的开发者主要是德国人[Heinz N. Gies](https://github.com/Licenser)。FiFo不仅仅是一套运维系统，实际上它还做了更多，完成了镜像管理，高可用等多方面的任务。简言之，FiFo解决了前面提出的大多数的问题。

让我们感谢Heinz先。

## 最后是DogeOS

Project FiFo作为一套运维系统，补全了SmartOS，但是它并不是SmartOS的一部分。

简单的说，用户仍然需要先安装SmartOS，再安装好FiFo。注意，SmartOS理论上是没有安装这种概念的，但是FiFo作为管理系统，显然是需要安装以解决持久化的问题的。

于是用户再一次可能陷入混乱。

我作为一个SmartOS的爱好者，本来我是准备克隆一套SmartDataCenter来扬名立万的:)，但是发现Heinz把这件事情做的这么好，我就放弃了自己开发，转而投入到粉Heinz的事业中去。

直到我发现了FiFo还并不是SmartOS一部分这个问题，于是我便开始解决这个问题，将SmartOS拆散重组，整合FiFo进去，形成了一个可能稍微好用一点点的系统。

这个系统被我叫做[DogeOS](http://www.dogeos.net)。

一个理想中的，用做管理数据中心的云计算操作系统，终于可能成为现实。

这让我为之兴奋不已。
