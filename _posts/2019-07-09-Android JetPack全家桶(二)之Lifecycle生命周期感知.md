---
layout:     post
title:      Android Jetpack全家桶(二)之Lifecycle生命周期感知
subtitle:    ""
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


## 介绍

### Lifecycle介绍

Lifecycle组件主要是在Jetpack体系中起到粘接的作用，感知页面(Activity/Fragment)生命周期，调整活动。其原理其实是利用观察者模式，注册订阅、分发消费。

Lifecycle使用两个主要枚举来跟踪其关联组件的生命周期状态：

  Event：从框架和Lifecycle类调度的生命周期事件，这些事件映射到活动和片段中的回调事件。
  State：跟踪的Lifecycle组件的当前状态。

![图片介绍](/img/toeii/icon_android_lifecycle_states.png)
    

### Lifecycle使用场景

在平时的开发过程中，我们难免有些逻辑的执行是和UI的生命周期相结合的，需要在特定的生命周期中执行相应的方法，我们平时做的可能就是在View中的每个周期调用Presenter中获取数据的方法，然后在调用View的回调接口更新UI，但现在使用Lifecycle可以使用注解和观察的模式自动调用Observe中定义好的方法。

## 如何使用？

#### 添加依赖

```java

// lifecycle
def lifecycle_version = "2.2.0-alpha02"
implementation "androidx.lifecycle:lifecycle-extensions:$lifecycle_version"
annotationProcessor "android.arch.lifecycle:compiler:$lifecycle_version"
kapt "android.arch.lifecycle:compiler:$lifecycle_version"

```

#### 创建Observer

```java

class AppLifecycleListener : LifecycleObserver {

    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    public fun connectOnCreate(){
        System.out.println("connectOnCreate================")
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    public fun connectOnStart(){
        System.out.println("connectOnStart================")
    }


    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public fun connectOnResume(){
        System.out.println("connectOnResume================")
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public fun connectOnPause(){
        System.out.println("connectOnPause================")
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    public fun connectOnStop(){
        System.out.println("connectOnStop================")
    }


    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    public fun connectOnDestroy(){
        System.out.println("connectOnDestroy================")
    }

}

```

#### 绑定Observer

##### 针对单页面场景

```java

class MainActivity : AppCompatActivity(), LifecycleOwner {

    private lateinit var lifecycleRegistry: LifecycleRegistry
    private lateinit var appLifecycleListener: AppLifecycleListener

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        lifecycleRegistry = LifecycleRegistry(this)
        appLifecycleListener = AppLifecycleListener()
        lifecycleRegistry.addObserver(appLifecycleListener)

        lifecycleRegistry.markState(Lifecycle.State.CREATED)

    }

    override fun onDestroy() {
        super.onDestroy()
        lifecycleRegistry.markState(Lifecycle.State.DESTROYED)
    }


    override fun getLifecycle(): Lifecycle {
        return lifecycleRegistry
    }

}

```

##### 针对全局应用场景

```java

class App : Application(){

    private val appLifecycleListener by lazy { AppLifecycleListener() }

    override fun onCreate() {
        super.onCreate()

        //ProcessLifecycleOwner
        ProcessLifecycleOwner.get().lifecycle.addObserver(appLifecycleListener)

    }

}

```

## 分析Lifecycle

#### LifecycleObserver

LifecycleObserver是一个接口类用于Observer的扩展。它还有两个间接的子类，DefaultLifecycleObserver和LifecycleEventObserver。从间接子类的实现可以看出来，它是将类标记为LifecycleObserver，它没有任何方法，而是依赖于OnLifecycleEvent带注释的方法。

#### LifecycleOwner和ProcessLifecycleOwner

LifecycleOwner是一个单一方法接口，表示该类具有生命周期，它只有一个方法getLifecycle（）。它主要用于独立的Activity/Fragment感知场景，Lifecycle.Event将跟随UI调度。
而ProcessLifecycleOwner是LifecycleOwner接口的扩展类，它可以将LifecycleOwner视为所有活动的组合，用于整个应用的生命周期感知。所以Lifecycle.Event.ON_CREATE将调度一次并且Lifecycle.Event.ON_DESTROY永远不会被调度。

#### LifecycleRegistry

LifecycleRegistry继承自Lifecycle，它可以处理单个Lifecycle对应多个观察者的关系。它可以注册和注销Observer，并且分发和消费Lifecycle.Event。

## 结语

对Lifecycle的介绍与使用说明就到这里结束了，如果对此有兴趣深入了解，可以查看源码或者自己尝试写一个Lifecycle机制的框架。

最后，贴上[demo地址](https://github.com/toeii/LifecycleSimpleExample)



