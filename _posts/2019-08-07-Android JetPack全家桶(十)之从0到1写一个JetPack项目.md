---
layout:     post
title:      Android JetPack全家桶(十)之从0到1写一个JetPack项目
subtitle:    ""
date:       2019-08-07
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Android
---


## 前言

Android JetPack是Google在18年IO大会上推荐的一整套组件库，它的出现填补了之前Android中自带的一些缺陷，例如Handler的内存泄露、Camera的不易用性、后台调度难以管理等等。所以我打算把整个架构组件系统性的学习一下，在这里和大家分享，希望能帮助到其他学习者。本系列文章包含十篇：

- [Android JetPack全家桶(一)之JetPack配置](https://toeii.github.io/2019/07/09/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B8%80)%E4%B9%8BJetPack%E9%85%8D%E7%BD%AE/)<br />
- [Android JetPack全家桶(二)之Lifecycle生命周期感知](https://toeii.github.io/2019/07/09/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%BA%8C)%E4%B9%8BLifecycle%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E6%84%9F%E7%9F%A5/)<br />
- [Android JetPack全家桶(三)之ViewModel控制器](https://toeii.github.io/2019/07/10/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B8%89)%E4%B9%8BViewModel%E6%8E%A7%E5%88%B6%E5%99%A8/)<br />
- [Android JetPack全家桶(四)之LiveData数据维持](https://toeii.github.io/2019/07/12/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%9B%9B)%E4%B9%8BLiveData%E6%95%B0%E6%8D%AE%E7%BB%B4%E6%8C%81/)<br />
- [Android JetPack全家桶(五)之Room ORM库](https://toeii.github.io/2019/07/17/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%BA%94)%E4%B9%8BRoom-ORM%E5%BA%93/)<br />
- [Android JetPack全家桶(六)之Paging分页库](https://toeii.github.io/2019/07/19/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%85%AD)%E4%B9%8BPaging%E5%88%86%E9%A1%B5%E5%BA%93/)<br />
- [Android JetPack全家桶(七)之WorkManager工作管理](https://toeii.github.io/2019/08/01/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B8%83)%E4%B9%8BWorkManager%E5%B7%A5%E4%BD%9C%E7%AE%A1%E7%90%86/)<br />
- [Android JetPack全家桶(八)之Navigation导航](https://toeii.github.io/2019/08/06/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%85%AB)%E4%B9%8BNavigation%E5%AF%BC%E8%88%AA/)<br />
- [Android JetPack全家桶(九)之DataBinding数据绑定](https://toeii.github.io/2019/08/07/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B9%9D)%E4%B9%8BDataBinding%E6%95%B0%E6%8D%AE%E7%BB%91%E5%AE%9A/)<br />
- [Android JetPack全家桶(十)之从0到1写一个JetPack项目](https://toeii.github.io/2019/08/07/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%8D%81)%E4%B9%8B%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AAJetPack%E5%B0%8F%E9%A1%B9%E7%9B%AE/)<br />


## 正文

项目正在开发中，待更新，先占坑。

