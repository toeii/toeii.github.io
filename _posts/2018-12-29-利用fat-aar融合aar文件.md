---
layout:     post
title:      利用fat-aar融合aar文件
subtitle:    ""
date:       2018-12-29
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Android
---

## 前言
因为之前公司的聚合支付SDK分离了支付业务，而这次又要统一包，所以把之前的微信支付宝聚合的aar与银联的aar进行融合。在这里记录分享一下。注：文章代码并非支付业务代码，而是之后我重新写过的demo代码。

fat-aar的项目地址[点我](https://github.com/adwiv/android-fat-aar)

## 介绍
aar文件是一种Android归档包，这种归档包是由Gradle构建库的Android Library插件产出的，它本质是一个压缩包。

合并前需要先确定到底需要合并哪些子模块。我们通过定义一个dependency configuration: emeded来标记需要被合并的模块，然后在gradle构建的afterEvaluate阶段收集被emeded依赖的模块信息。
```java
afterEvaluate {
    def dependencies = new ArrayList(configurations.embedded.resolvedConfiguration.firstLevelModuleDependencies)
    dependencies.reverseEach {
        ...
        it.moduleArtifacts.each {
            artifact ->
                if (artifact.type == 'aar') {
                    if (!embeddedAarFiles.contains(artifact)) {
                        //要合并的AAR文件
                        embeddedAarFiles.add(artifact)
                    }
                    if (!embeddedAarDirs.contains(modulePath)) {
                        ...
                        //每个AAR的解压目录
                        embeddedAarDirs.add(modulePath)
                    }
                } else if (artifact.type == 'jar') {
                    ...
                    //要合并的JAR文件
                    embeddedJars.add(artifactPath)
                } 
                ...
        }
    }
}
```

可以看出，这里收集了三个集合：

1，要合并的aar文件
2，每个aar的解压目录
3，要合并的jar文件

aar和jar分开收集是因为合并这两种文件的操作不同，jar只需纳入将其Class文件，而aar需要合并更多内容。

## 过程

### 导入fat-aar.gradle

1，将fat-aar.gradle文件放入sdk/目录下

```XML
sdk
    > build
        libs
    > src
        .gitignore
        build.gradle
        fat-aar.gradle
        ...
```

2，修改sdk/build.gradle脚本如下

```XML
apply plugin: 'com.android.library'
apply from: 'fat-aar.gradle'

android {
    ...
}

dependencies {
    if (rootProject.ext.debug) {
        compile project(':base')
    } else {
        embedded project(':base')
    }
    ...
}
```

3，修改工程下build.gradle脚本添加依赖于:sdk:build的main task

```XML
buildscript {
    ext {
        debug = false
        ...
    }
    ...
}
...
task main(dependsOn: ':sdk:build') {

}
```

4，执行gradle命令合并aar包

```XML
gradle clean main
```

5，命令执行成功后，将sdk/build/outputs/aar/sdk-release.aar文件复制到app/libs目录

```XML
app
    > build
        libs
        sdk-release.aar
...
```

6，修改app/build.gradle脚本

```XML
apply plugin: 'com.android.application'

android {
    ...
    repositories {
        flatDir {
            dirs 'libs'
        }
    }
}

dependencies {
    if (rootProject.ext.debug) {
        compile project(':sdk')
    } else {
        compile(name: 'sdk-release', ext: 'aar')
    }
    ...
}
```

最后，运行app查看结果。没有报错切功能已经集成完成，代表融合成功。

## 总结

这里的aar融合方式只支持单渠道的SDK，如果业务需要多渠道则不推荐这样的方式进行融合操作。
















