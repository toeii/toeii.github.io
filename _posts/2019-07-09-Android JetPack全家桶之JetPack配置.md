---
layout:     post
title:      Android JetPack全家桶之JetPack配置
subtitle:    "入门配置"
date:       2019-07-09
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Android
---


## 前言

Android Jetpack是Google目前力推的Android开发组件库，它的作用主要是为了填补之前Android中自带的一些缺陷，例如Handler的内存泄露、Camera的不易用性、后台调度难以管理等等。所以，我也跟随Google的脚步，一点点进入对Jetpack的学习使用，在此记录分享。

## 开始配置

所有 Jetpack 组件都可在 Google Maven 代码库中找到。

打开你的项目的 build.gradle 文件并添加 google() 代码库，如下所示：

```XML
allprojects {
    repositories {
        google()
        jcenter()
    }
}
```

然后配置App.Gradle：

```XML
dependencies {
    def lifecycle_version = "2.0.0"
    implementation "androidx.lifecycle:lifecycle-extensions:$lifecycle_version"
    // Optional : Kotlin extension (https://d.android.com/kotlin/ktx)
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
    ...
}
```

## 总结

完成以上配置，就可以开始使用Jetpack组件了。欢迎阅读接下来的文章对Jetpack进行更多了解。



