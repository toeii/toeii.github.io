---
layout:     post
title:      Flutter与Android混合开发之项目集成
subtitle:    "跨平台是未来的趋势啊..."
date:       2019-01-12
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Flutter
---

## 前言
大多数公司的Android项目无法向Flutter整体迁移，这样代价太大。所以相对于单独开发Flutter应用，混合开发对于线上项目更具有实际意义，可以把风险控制到最低，也便于进行业务迭代上线。

## 思考
目前流行的混合方案有几种：
    1，[官方推荐方案](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps) 
    2，[闲鱼团队使用的方案](https://yq.aliyun.com/articles/618599?spm=a2c4e.11153959.0.0.4f29616b9f6OWs)
    3，[头条团队使用的方案](https://mp.weixin.qq.com/s/wdbVVzZJFseX2GmEbuAdfA)

由于考虑到一般公司的开发团队大小和项目体量，我个人认为更好的混合方案是官方推荐方案。所以，以下针对官方推荐方案进行混合集成。

## 集成过程
1，在已有项目根目录运行命令

```XML
    flutter create -t module my_flutter
```

2，在项目的settings.gradle 文件添加如下代码

```java
// MyApp/settings.gradle
include ':app'                                    
setBinding(new Binding([gradle: this]))                                 
evaluate(new File(                                                    
    settingsDir.parentFile,                                            
    'my_flutter/.android/include_flutter.groovy'                       
)) 
```

3，在运行模块app的build.gradle添加依赖

```java
// MyApp/app/build.gradle
dependencies {
    implementation project(':flutter')
}
```

4，在原生Android端创建并添加flutterView

```java
val flutterView = Flutter.createView(this, lifecycle, "route1")
val layoutParams = FrameLayout.LayoutParams(-1, -1)
addContentView(flutterView, layoutParams)
```

5，在Flutter端入口操作

```java
void main() => runApp(_widgetForRoute(window.defaultRouteName));

Widget _widgetForRoute(String route) {
    switch (route) {
        case 'route1':
        return SomeWidget(...);
        default:
        return Center(
            child: Text('Unknown route: $route', textDirection: TextDirection.ltr),
        );
    }
}
```

到这里，集成已经完成。





