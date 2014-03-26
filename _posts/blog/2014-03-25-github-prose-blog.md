---
category: blog
published: false
layout: post
title: "天作之和: github pages + prose.io"
description: github jekyll静态页面自动生成，沉浸式的书写体验，git版本控制，夫复何求？
---

## Why github pages + prose.io？

一句话：**github jekyll静态页面自动生成，沉浸式的书写体验，git版本控制，夫复何求？**

写Blog是一个很随意的行为，通常人们对写得要求也不高，所以从WordPress到轻博客再到微博，写的是越来越轻，越来越容易。

但是维护Blog并不容易。

对于向我这样的人来说，一个可以维护的Blog系统需要

1. 用简单的数据格式存储
> 想要找到某片Blog的文章时，无需翻箱倒柜将后台数据库大卸八块。Come on，blog本来不应该就是些txt文件么？Grep一把为什么就不行？

2. 有一个沉浸式的编写工具
> 这个很重要。写东西的时候总是需要没有干扰，左右两边不要不停的有“齐x短裙热卖9元起”这样的东西飞来飞去。

3. 能打草稿，能版本控制。
> 人就是这么懒，想起来一点写一点，但是如果有个工具激励，事情或许会不同一点。

所以github pages是个貌似理想的选择：存储就是文本文件，加上一点简单Markdown；版本控制用git，没有更好的了。

但是github pages缺少一个简单的编写工具。或者说，没有一个简单的可以在Web上使用的工具。clone到本地，动用jekyll server再加vim，显得太重量，适合做一些比较有创造性的事情，不适合随便写写。

于是prose.io弥补了这个空缺，提供了一个很好的在线github pages编辑工具，问题解决了。

## 看起来怎样

以你现在看到的这个blog为例吧，看起来是这样的

![LI, Yu from 1981](/images/github-prose-start.png)

点击左上角的Edit之后，进入在线编辑器，是这样的

![Editing _posts-blog-2014-03-25-github-prose-blog.md](/images/github-prose-edit1.png)

完成编写，提交是这样的
![Editing _posts-blog-2014-03-25-github-prose-blog.md](/images/github-prose-edit2.png)

点击完毕提交，boom，commit到github，github pages通过hook就自动生成了静态页面，就这么简单。

当然，因为它是github pages，所以clone，vim随便写一个很复杂的演示什么的，自然是无比强大。想备份，git clone，想观察更改记录，github的web界面就可以用。

总之，如果你是程序猿，没有比这个更加合适的了。

## 怎样弄一个

## Prose.io使用上的注意事项
