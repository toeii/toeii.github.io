---
layout:     post
title:      Android Launcher3定制开发之去除搜索和开机引导
subtitle:    ""
date:       2020-05-11
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
- [Android Launcher3定制开发之去除搜索和开机引导](https://toeii.github.io/2020/05/11/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E5%8E%BB%E9%99%A4%E6%90%9C%E7%B4%A2/)<br />
- [Android Launcher3定制开发之修改快捷方式图标](https://toeii.github.io/2020/05/12/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E4%BF%AE%E6%94%B9%E5%BF%AB%E6%8D%B7%E6%96%B9%E5%BC%8F%E5%9B%BE%E6%A0%87/)<br />
- [Android Launcher3定制开发之添加负一屏](https://toeii.github.io/2020/05/14/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E6%B7%BB%E5%8A%A0%E8%B4%9F%E4%B8%80%E5%B1%8F/)<br />


## 简介

用户在开启Launcher的时候，弹出默认的开启引导往往显得有些碍事，我们可以通过去除或者自定义引导完善业务场景。同时，本文也将教大家如何把首屏默认的google搜索栏小部件去除掉。

## 去除桌面引导

Launcher在初始化的时候，会通过shouldShowIntroScreen检查，是否是第一次运行。
```java

    if (shouldShowIntroScreen()) {
        showIntroScreen();
    } else {
        showFirstRunActivity();
        showFirstRunClings();
    }

```
    
可以看到Launcher.showFirstRunClings方法，它处理了显示第一次运行的引导窗。那么，我们直接将代码屏蔽掉。

```java

    @Thunk void showFirstRunClings() {
        //TODO 屏蔽开机引导提示
    //    LauncherClings launcherClings = new LauncherClings(this);
    //    if (launcherClings.shouldShowFirstRunOrMigrationClings()) {
    //        mClings = launcherClings;
    //        if (mModel.canMigrateFromOldLauncherDb(this)) {
    //            launcherClings.showMigrationCling();
    //        } else {
    //            launcherClings.showLongPressCling(true);
    //        }
    //    }
    }

```

## 自定义引导

结合上面的代码分析，可以看到Launcher.showFirstRunActivity方法中，它处理了第一运行的Activity。那么，我们可以在这个方法中启动自己想要替换的Activity，做到引导界面替换。由于这里已经阐述的很明确了，这里就不贴代码了，可以自己修改试一试。

## 去除预置Google搜索栏

在DeviceProfile类中找到layout方法，可以看出搜索栏在这里做了处理，那么我们将它隐藏就可以了。

```java

  public void layout(Launcher launcher) {
        FrameLayout.LayoutParams lp;
        boolean hasVerticalBarLayout = isVerticalBarLayout();
        final boolean isLayoutRtl = Utilities.isRtl(launcher.getResources());
        View searchBar = launcher.getSearchDropTargetBar();
        ...
        searchBar.setLayoutParams(lp);
        //隐藏
        searchBar.setVisibility(View.GONE);
  }

```

## 结语

如果读完本文还有不懂的地方，可以结合代码加深理解，代码通过点击[这里](https://github.com/toeii/Launcher3)找到。希望对大家学习和了解Launcher开发有所帮助。

