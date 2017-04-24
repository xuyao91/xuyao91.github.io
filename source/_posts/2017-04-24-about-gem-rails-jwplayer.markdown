---
layout: post
title: "关于rails-jwplayer gem"
date: 2017-01-16 19:54:57 +0800
comments: true
categories: 
---
&emsp;这几天公司有个业务，有一些课程（视频播放）需要统计用户的观看数据，包括用户看了多长时间，在哪些地方快进了，哪些地方暂停了，哪些地方重复看了等一些用户的行为操作，调查了一翻，H5自带的video是不支持这些功能，好在发现国外有个[jwplayer](https://www.jwplayer.com/)的视频播放插件，功能不是一般的强大，支持各种自定义功能，于是看完文档后，自己就集成了一个基于jwplayer的rails gem（[rails-jwplayer](https://github.com/xuyao91/rails-jwplayer)），便于以后自己用。

&emsp;目前功能做的比较简单，就是集成了一些基本的功能，后期会把spec测试补齐，及再做一些深度的集可成。 
具体如何使用，可以到github上看下[rails-jwplayer](https://github.com/xuyao91/rails-jwplayer/blob/master/README.md)
