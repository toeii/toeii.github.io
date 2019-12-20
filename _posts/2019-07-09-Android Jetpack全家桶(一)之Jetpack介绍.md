---
layout:     post
title:      Android Jetpack全家桶(一)之Jetpack介绍
subtitle:    "入门配置"
date:       2019-07-09
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Android
---



## 前言

Android JetPack是Google在18年IO大会上推荐的一整套组件库，它的出现填补了之前Android中自带的一些缺陷，例如Handler的内存泄露、Camera的不易用性、后台调度难以管理等等。所以我打算把整个架构组件系统性的学习一下，在这里和大家分享，希望能帮助到其他学习者。本系列文章包含十篇：

- [Android Jetpack全家桶(一)之Jetpack介绍](https://toeii.github.io/2019/07/09/Android-Jetpack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B8%80)%E4%B9%8BJetpack%E4%BB%8B%E7%BB%8D/)<br />
- [Android Jetpack全家桶(二)之Lifecycle生命周期感知](https://toeii.github.io/2019/07/09/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%BA%8C)%E4%B9%8BLifecycle%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E6%84%9F%E7%9F%A5/)<br />
- [Android Jetpack全家桶(三)之ViewModel控制器](https://toeii.github.io/2019/07/10/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B8%89)%E4%B9%8BViewModel%E6%8E%A7%E5%88%B6%E5%99%A8/)<br />
- [Android Jetpack全家桶(四)之LiveData数据维持](https://toeii.github.io/2019/07/12/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%9B%9B)%E4%B9%8BLiveData%E6%95%B0%E6%8D%AE%E7%BB%B4%E6%8C%81/)<br />
- [Android Jetpack全家桶(五)之Room ORM库](https://toeii.github.io/2019/07/17/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%BA%94)%E4%B9%8BRoom-ORM%E5%BA%93/)<br />
- [Android Jetpack全家桶(六)之Paging分页库](https://toeii.github.io/2019/07/19/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%85%AD)%E4%B9%8BPaging%E5%88%86%E9%A1%B5%E5%BA%93/)<br />
- [Android Jetpack全家桶(七)之WorkManager工作管理](https://toeii.github.io/2019/08/01/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B8%83)%E4%B9%8BWorkManager%E5%B7%A5%E4%BD%9C%E7%AE%A1%E7%90%86/)<br />
- [Android Jetpack全家桶(八)之Navigation导航](https://toeii.github.io/2019/08/06/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%85%AB)%E4%B9%8BNavigation%E5%AF%BC%E8%88%AA/)<br />
- [Android Jetpack全家桶(九)之DataBinding数据绑定](https://toeii.github.io/2019/08/07/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B9%9D)%E4%B9%8BDataBinding%E6%95%B0%E6%8D%AE%E7%BB%91%E5%AE%9A/)<br />
- [Android Jetpack全家桶(十)之从0到1写一个Jetpack项目](https://toeii.github.io/2019/11/20/Android-Jetpack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%8D%81)%E4%B9%8B%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AAJetPack%E9%A1%B9%E7%9B%AE/)<br />


## 简介

谷歌在 2018 I/O 大会上发布了一系列辅助android开发者的实用工具，这套工具就是Jetpack，它是一套库、工具和指南的合集，可以帮助开发者更轻松地编写和构建出色的 Android 应用程序。
Jetpack中的有些组件并不是第一次推出，其中LifeCycle、LiveData、ViewModel、Room等组件早在 Google I/O 2017年大会上就随着 Android Architecture Component（AAC）一起推出了，但是推广效果一般。时隔一年后谷歌在AAC的基础之上发布了Jetpack，并发布了其他工具以解决Android技术选型乱以及开发不规范等问题。

Jetpack有以下特点：

- 加速开发：组件可以单独采用（不过这些组件是为协同工作而构建的），同时利用 Kotlin 语言功能帮助您提高工作效率。

- 消除样板代码：Jetpack 可管理繁琐的 Activity（如后台任务、导航和生命周期管理）。

- 构建高质量的强大应用：Jetpack 组件围绕现代化设计实践构建而成，具有向后兼容性，可以减少崩溃和内存泄漏。

## Jetpack分类

Android Jetpack组件共分为四大类，Foundation、Architecture、Behavior和UI。

### Foundation(基础组件)：

基础组件提供了横向功能，例如向后兼容性、测试以及Kotlin语言的支持。它包含如下组件库：

- Android KTX：Android KTX 是一组 Kotlin 扩展程序，它优化了供Kotlin使用的Jetpack和Android平台的API。以更简洁、更愉悦、更惯用的方式使用Kotlin进行Android开发。

- AppCompat：提供了一系列以AppCompat开头的API，以便兼容低版本的Android开发。

- Cars(Auto)：有助于开发 Android Auto 应用的组件，无需担心特定于车辆的硬件差异（如屏幕分辨率、软件界面、旋钮和触摸式控件）。

- Benchmark(检测)：从 Android Studio 中快速对基于 Kotlin 或 Java 的代码进行基准化分析。衡量代码性能，并将基准化分析结果输出到 Android Studio 控制台。

- Multidex(多Dex处理)：为方法数超过 64K 的应用启用多 dex 文件。

- Security(安全)：按照安全最佳做法读写加密文件和共享偏好设置。

- Test(测试)：用于单元和运行时界面测试的 Android 测试框架。

- TV：构建可让用户在大屏幕上体验沉浸式内容的应用。

- Wear OS：有助于开发 Wear 应用的组件。

### Architecture(架构组件)：

架构组件可帮助开发者设计稳健、可测试且易维护的应用。它包含如下组件库：

- Data Binding(数据绑定)：数据绑定库是一种支持库，借助该库，可以使用声明式将布局中的界面组件绑定到应用中的数据源。

- Lifecycles：方便管理 Activity 和 Fragment 生命周期，帮助开发者书写更轻量、易于维护的代码。

- LiveData：是一个可观察的数据持有者类。与常规observable不同，LiveData是有生命周期感知的。

- Navigation：处理应用内导航所需的一切。

- Paging：帮助开发者一次加载和显示小块数据。按需加载部分数据可减少网络带宽和系统资源的使用。

- Room：Room持久性库在SQLite上提供了一个抽象层，帮助开发者更友好、流畅的访问SQLite数据库。

- ViewModel：以生命周期感知的方式存储和管理与UI相关的数据。

- WorkManager：即使应用程序退出或设备重新启动，也可以轻松地调度预期将要运行的可延迟异步任务。

### Behavior(行为)：

行为组件可帮助开发者的应用与标准 Android 服务（如通知、权限、分享和 Google 助理）相集成。它包含如下组件库：

- CameraX：帮助开发者简化相机应用的开发工作。它提供一致且易于使用的 API 界面，适用于大多数 Android 设备，并可向后兼容至 Android 5.0（API 级别 21）。

- DownloadManager(下载管理器)：可处理长时间运行的HTTP下载，并在出现故障或在连接更改和系统重新启动后重试下载。

- Media & playback(媒体&播放)：用于媒体播放和路由（包括 Google Cast）的向后兼容 API。

- Notifications(通知)：提供向后兼容的通知 API，支持 Wear 和 Auto。

- Permissions(权限)：用于检查和请求应用权限的兼容性 API。

- Preferences(偏好设置)：提供了用户能够改变应用的功能和行为能力。

- Sharing(共享)：提供适合应用操作栏的共享操作。

- Slices(切片)：创建可在应用外部显示应用数据的灵活界面元素。

### UI(界面组件)：

界面组件可提供各类view和辅助程序，让应用不仅简单易用，还能带来愉悦体验。它包含如下组件库：

- Animation & Transitions(动画&过度)：提供各类内置动画，也可以自定义动画效果。

- Emoji(表情符号)：使用户在未更新系统版本的情况下也可以使用表情符号。

- Fragment：组件化界面的基本单位。

- Layout(布局)：xml书写的界面布局或者使用Compose完成的界面。

- Palette(调色板)：从调色板中提取出有用的信息。

## 使用Jetpack

所有 Jetpack 组件都可在 Google Maven 代码库中找到。

打开你的项目的 build.gradle 文件并添加 google() 代码库，如下所示

```java

    allprojects {
        repositories {
            google()
            jcenter()
        }
    }

```

然后配置App.Gradle：

```java

def lifecycle_version = '2.2.0-alpha02'
def paging_version = '2.1.0-alpha01'
def room_version = '2.2.0'
def work_version = '2.1.0'
def navigation_version = '2.2.0-alpha01'

dependencies {

    ...

    // liveData and viewMode
    implementation "androidx.lifecycle:lifecycle-extensions:$rootProject.lifecycle_version"
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$rootProject.lifecycle_version"
    // paging
    implementation "android.arch.paging:runtime:$rootProject.paging_version"
    implementation "androidx.paging:paging-runtime-ktx:$rootProject.paging_version"
    // room
    kapt "androidx.room:room-compiler:$rootProject.room_version"
    implementation "androidx.room:room-runtime:$rootProject.room_version"
    annotationProcessor "androidx.room:room-compiler:$rootProject.room_version"
    // work
    implementation "androidx.work:work-runtime:$rootProject.work_version"
    implementation "androidx.work:work-runtime-ktx:$rootProject.work_version"
    // navigation
    implementation "androidx.navigation:navigation-ui-ktx:$rootProject.navigation_version"
    implementation "androidx.navigation:navigation-fragment-ktx:$rootProject.navigation_version"
    implementation "androidx.navigation:navigation-runtime-ktx:$rootProject.navigation_version"

}

```

## 写在最后

以上配置是对Jetpack系列的组件进行统一依赖，后续文章中由于针对性的学习只会配置单一依赖。欢迎阅读接下来的文章对Jetpack进行更多了解。



