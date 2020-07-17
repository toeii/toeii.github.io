---
layout:     post
title:      Android Launcher3定制开发之添加负一屏
subtitle:    ""
date:       2020-05-14
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Android
---



## 前言

为了加深对Launcher3的整体印象，记录在工作上所遇到和解决的关于Launcher3定制开发的一些知识点，并且归纳总结成本系列文章。同时，也能起到帮助他人解决一些关于Launcher3相关定制的需求开发。

- [Android Launcher3定制开发之结构](https://toeii.github.io/2020/05/06/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E7%BB%93%E6%9E%84/)<br />
- [Android Launcher3定制开发之改造横向化](https://toeii.github.io/2020/05/07/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E6%94%B9%E9%80%A0%E6%A8%AA%E5%90%91%E5%8C%96/)<br />
- [Android Launcher3定制开发之创建小部件](https://toeii.github.io/2020/05/08/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E5%88%9B%E5%BB%BA%E5%B0%8F%E9%83%A8%E4%BB%B6/)<br />
- [Android Launcher3定制开发之预置快捷方式](https://toeii.github.io/2020/05/09/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E9%A2%84%E7%BD%AE%E5%BF%AB%E6%8D%B7%E6%96%B9%E5%BC%8F/)<br />
- [Android Launcher3定制开发之预置文件夹](https://toeii.github.io/2020/05/10/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E9%A2%84%E7%BD%AE%E6%96%87%E4%BB%B6%E5%A4%B9/)<br />
- [Android Launcher3定制开发之去除搜索和开机引导](https://toeii.github.io/2020/05/11/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E5%8E%BB%E9%99%A4%E6%90%9C%E7%B4%A2/)<br />
- [Android Launcher3定制开发之修改快捷方式图标](https://toeii.github.io/2020/05/12/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E4%BF%AE%E6%94%B9%E5%BF%AB%E6%8D%B7%E6%96%B9%E5%BC%8F%E5%9B%BE%E6%A0%87/)<br />
- [Android Launcher3定制开发之添加负一屏](https://toeii.github.io/2020/05/14/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E6%B7%BB%E5%8A%A0%E8%B4%9F%E4%B8%80%E5%B1%8F/)<br />


## 简介

Launcher定制开发中，负一屏的定制几乎是所有Launcher应用都需要做的事情。所以，本文将说一下如何添加负一屏，并且如何监听滑动处理负一屏信息加载。

## 添加负一屏

首先，在Launcher.hasCustomContentToLeft中，默认开启显示负一屏View。

```java

    protected boolean hasCustomContentToLeft() {
    //    if (mLauncherCallbacks != null) {
    //        return mLauncherCallbacks.hasCustomContentToLeft();
    //    }
    //    return false;
    
        //TODO 负一屏处理
        return true;
    }

```

接着，在Launcher.populateCustomContentContainer中替换负一屏加载自定义LayoutView

```java

    protected void populateCustomContentContainer() {
    //    if (mLauncherCallbacks != null) {
    //        mLauncherCallbacks.populateCustomContentContainer();
    //    }

        //TODO 负一屏Custom View
        View customView = getLayoutInflater().inflate(R.layout.activity_negative_operation, null);
        addToCustomContentPage(customView,null,"");
    }

```

以上，负一屏就添加完了。

## 桌面滑动监听，并处理负一屏数据加载

负一屏虽然添加完成了，但是多数业务场景都是要对负一屏进行刷新和更新数据的。那么怎么办呢，我们可以写一套观察者模式，采用订阅的方式更换数据集，但是针对场景来说，也可以直接监听是否滑动到了负一屏来进行无感更新(默认取缓存数据再替换服务端返回数据)。那么，我们怎么做呢？其实很简单。Launcher.onPageSwitch就可以接收到滑动回调结果，见下：

```java

    @Override
    public void onPageSwitch(View newPage, int newPageIndex) {
        if (mLauncherCallbacks != null) {
            mLauncherCallbacks.onPageSwitch(newPage, newPageIndex);
        }

        if(newPageIndex == 0){//TODO 负一屏加载
            notifyNegativeData();
        }
    }

```

## 结语

如果读完本文还有不懂的地方，可以结合代码加深理解，代码通过点击[这里](https://github.com/toeii/Launcher3)找到。希望对大家学习和了解Launcher开发有所帮助。


