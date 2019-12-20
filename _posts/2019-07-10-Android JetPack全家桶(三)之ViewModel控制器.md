---
layout:     post
title:      Android Jetpack全家桶(三)之ViewModel控制器
subtitle:    ""
date:       2019-07-10
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Android
---



## 前言

Android Jetpack是Google在18年IO大会上推荐的一整套组件库，它的出现填补了之前Android中自带的一些缺陷，例如Handler的内存泄露、Camera的不易用性、后台调度难以管理等等。所以我打算把整个架构组件系统性的学习一下，在这里和大家分享，希望能帮助到其他学习者。本系列文章包含十篇：

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


## 介绍

ViewModel的出现是为了解决数据因Android UI控制器在生命周期活动中造成数据丢失的问题。

在一般情况下，页面数据丢失（转屏、闪退等生命周期重建现象）我们都会通过onSaveInstanceState()方法并从onCreate()中的包中恢复其数据。但此方法仅适用于可以序列化然后反序列化的少量数据，而不适用于潜在的大量数据像用户列表或位图。
另一个问题是UI控制器经常需要进行可能需要一些时间才能返回的异步调用。UI控制器需要管理这些调用并确保系统在销毁后清理它们以避免潜在的内存泄漏。此管理需要大量维护，并且在为配置更改重新创建对象的情况下，这会浪费资源，因为对象可能必须重新发出已经进行的调用。

诸如活动和片段之类的UI控制器主要用于显示UI数据，对用户操作作出反应或处理操作系统通信，例如许可请求。要求UI控制器也负责从数据库或网络加载数据，这会给类增加膨胀。为UI控制器分配过多的责任可能导致单个类尝试自己处理应用程序的所有工作，而不是将工作委托给其他类。以这种方式为UI控制器分配过多的责任也会使测试变得更加困难。
而ViewModel将视图数据所有权与UI控制器逻辑分离起来更容易，更有效的解决这一系列问题。

## 如何使用？

#### 添加依赖

```java

// lifecycle
def lifecycle_version = "2.2.0-alpha02"
implementation "androidx.lifecycle:lifecycle-extensions:$lifecycle_version"
annotationProcessor "android.arch.lifecycle:compiler:$lifecycle_version"
kapt "android.arch.lifecycle:compiler:$lifecycle_version"

```

#### 构建ViewModel

```java

class BaseModel: ViewModel() {

    var userTag = ""

}

```

```java

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        val tvContent = findViewById<TextView>(R.id.tv_content)

        val baseModel = ViewModelProviders.of(this)[BaseModel::class.java]

        if(baseModel.userTag.isNotEmpty()){
            tvContent.text = baseModel.userTag
        }

        tvContent.setOnClickListener {
            baseModel.userTag = "Hello BaseModel"
            tvContent.text = baseModel.userTag
        }

    }

}

```

## 分析ViewModel

#### ViewModel生命周期

![图片介绍](/img/toeii/icon_android_viewmodel_lifecycle.png)

获取ViewModel时，ViewModel对象的范围限定为传递给ViewModelProvider的生命周期。
ViewModel保留在内存中，直到它的作用域生命周期永久消失：在活动的情况下，当它完成时，在片段的情况下，当它被分离时。
图中显示了活动经历轮换然后结束时的各种生命周期状态。该图还显示了关联活动生命周期旁边的ViewModel的生命周期。
此特定图表说明了活动的状态。相同的基本状态适用于片段的生命周期。

通常在系统第一次调用活动对象的onCreate()方法时请求ViewModel。
系统可以在活动的整个生命周期中多次调用onCreate()，例如在旋转设备屏幕时。
而ViewModel从第一次请求ViewModel到活动完成并销毁之时就存在。

#### ViewModel加载原理

ViewModel将UI控制器与数据加载操作分开，这意味着类之间的强引用较少，ViewModel内部是与Room和LiveData一起使用以达到替换加载的效果。
当数据发生变化时，Room存储层会通知到LiveData，然后LiveData会使用修改后的数据更新UI，这样确保数据在设备配置更改后仍然存在。

关于LiveData，它是一个数据维持类，底层由Map实现。下篇文章会讲到，这里略知一二即可。

关于Room，它是一个ORM库。后续文章会讲到，这里略知一二即可。


## 结语

最后，贴上[demo地址](https://github.com/toeii/ViewModelSimpleExample)



