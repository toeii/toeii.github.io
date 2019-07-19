---
layout:     post
title:      Android JetPack全家桶(二)之Lifecycle生命周期感知
subtitle:    ""
date:       2019-07-09
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Android
---


## 前言

Jetpack全家桶我打算从外至内的学习，所以本篇也主要介绍的是Lifecycle组件。

在开始学习之前，你可以看看如何[配置](https://toeii.github.io/2019/07/09/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B8%80)%E4%B9%8BJetPack%E9%85%8D%E7%BD%AE/)JetPack。

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



