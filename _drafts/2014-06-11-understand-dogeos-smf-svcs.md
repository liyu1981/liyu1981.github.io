---
published: false
---

## 理解DogeOS的系统服务体系

为什么要理解系统服务体系？

因为作为一个服务器操作系统，实际上管理员与用户感受到的，主要就是这些跑在后台的服务。

（DogeOS目前本质上就是SmartOS，以下的分析也适用于SmartOS。更进一步其中的原理，适用于所有illumos派生的操作系统）。

## 从SMF框架说起

DogeOS继承了illumos/SmartOS的衣钵，系统服务组织上，采用了先进的SMF框架。因此理解系统服务的结构，必须要先理解什么是SMF？
