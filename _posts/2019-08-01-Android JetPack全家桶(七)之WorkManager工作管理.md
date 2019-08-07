---
layout:     post
title:      Android JetPack全家桶(七)之WorkManager工作管理
subtitle:    ""
date:       2019-08-01
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Android
---



## 前言

Android JetPack组件库是Google18年IO大会上推荐的一整套开发方案，它的出现填补了之前Android中自带的一些缺陷，例如Handler的内存泄露、Camera的不易用性、后台调度难以管理等等。所以我打算把整个架构组件系统性的学习一下，在这里和大家分享，希望能帮助到其他学习者。本系列文章包含十篇：

- [Android JetPack全家桶(一)之JetPack配置](https://toeii.github.io/2019/07/09/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B8%80)%E4%B9%8BJetPack%E9%85%8D%E7%BD%AE/)<br />
- [Android JetPack全家桶(二)之Lifecycle生命周期感知](https://toeii.github.io/2019/07/09/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%BA%8C)%E4%B9%8BLifecycle%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E6%84%9F%E7%9F%A5/)<br />
- [Android JetPack全家桶(三)之ViewModel控制器](https://toeii.github.io/2019/07/10/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B8%89)%E4%B9%8BViewModel%E6%8E%A7%E5%88%B6%E5%99%A8/)<br />
- [Android JetPack全家桶(四)之LiveData数据维持](https://toeii.github.io/2019/07/12/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%9B%9B)%E4%B9%8BLiveData%E6%95%B0%E6%8D%AE%E7%BB%B4%E6%8C%81/)<br />
- [Android JetPack全家桶(五)之Room ORM库](https://toeii.github.io/2019/07/17/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%BA%94)%E4%B9%8BRoom-ORM%E5%BA%93/)<br />
- [Android JetPack全家桶(六)之Paging分页库](https://toeii.github.io/2019/07/19/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%85%AD)%E4%B9%8BPaging%E5%88%86%E9%A1%B5%E5%BA%93/)<br />
- [Android JetPack全家桶(七)之WorkManager工作管理](https://toeii.github.io/2019/08/01/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B8%83)%E4%B9%8BWorkManager%E5%B7%A5%E4%BD%9C%E7%AE%A1%E7%90%86/)<br />
- [Android JetPack全家桶(八)之Navigation导航](https://toeii.github.io/2019/08/06/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%85%AB)%E4%B9%8BNavigation%E5%AF%BC%E8%88%AA/)<br />
- [Android JetPack全家桶(九)之DataBinding数据绑定](https://toeii.github.io/2019/08/07/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B9%9D)%E4%B9%8BDataBinding%E6%95%B0%E6%8D%AE%E7%BB%91%E5%AE%9A/)<br />
- [Android JetPack全家桶(十)之从0到1写一个JetPack小项目](https://toeii.github.io/2019/08/07/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%8D%81)%E4%B9%8B%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AAJetPack%E5%B0%8F%E9%A1%B9%E7%9B%AE/)<br />



## 介绍

WorkManager是Jetpack组件库中提供的一种新的延迟异步方案，它可以实现应用程序退出或设备重新启动的可延迟(也可以不延迟)异步任务。例如：在程序退出时将日志上传到服务器等数据采集操作。

官方建议，WorkManager不适用于在应用程序进程被kill后仍需执行的后台任务，也不适用于需要立即执行的任务。


## 如何使用？

#### 添加依赖

app.build.gradle

```java

dependencies {
     
    ...

    def work_version = '2.1.0'
    implementation "androidx.work:work-runtime:$work_version"
    implementation "androidx.work:work-runtime-ktx:$work_version"
}

```

#### 配置WorkManagerConfiguration

单例创建WorkManagerConfiguration，之后可以便于Worker配置。

```java

internal class MyApplication : Application(), Configuration.Provider {
    override fun getWorkManagerConfiguration() =
        Configuration.Builder()
            .setMinimumLoggingLevel(Log.VERBOSE)
            .build()

    override fun onCreate() {
        super.onCreate()
    }

}

```

#### 新建Worker

新建一个下载Worker，执行Worker之后通过doWork()实现业务逻辑并回调任务Result，之后可以在onStopped()中监听到Worker结束。

```java

class DownloadWork(var context: Context, private var workerParams: WorkerParameters) : Worker(context, workerParams) {


    override fun doWork(): Result {
        //worker execute...
         var result = false

         context.runOnUiThread {
             result = asyncDoWork()
         }

        return if(result){
            Result.success()
        }else{
            Result.failure()
        }

    }

    private fun asyncDoWork(): Boolean {
        ...
    }


    override fun onStopped() {
        super.onStopped()
        //worker task stop ...
    }

}

```

#### 建立Worker约束

针对使用场景建立worker调用约束

```java

    val myConstraints = Constraints.Builder()
//                .setRequiresDeviceIdle(true)//指定{@link WorkRequest}运行时设备是否为空闲
//                .setRequiresCharging(true)//指定要运行的{@link WorkRequest}是否应该插入设备
//                .setRequiredNetworkType(NetworkType.NOT_ROAMING)
//                .setRequiresBatteryNotLow(true)//指定设备电池是否不应低于临界阈值
//                .setRequiresCharging(true)//网络状态
//                .setRequiresDeviceIdle(true)//指定{@link WorkRequest}运行时设备是否为空闲
//                .setRequiresStorageNotLow(true)//指定设备可用存储是否不应低于临界阈值
//                .addContentUriTrigger(Uri.parse(uri),false)//指定内容{@link android.net.Uri}时是否应该运行{@link WorkRequest}更新
    .build()

```

#### 初始化Worker和常用的Work操作

运行单次Work。enqueue()是执行work操作，cancelWorkById()可以通过所绑定的OneTimeWorkRequestBuilder.id取消相应的work(已执行的)操作。

```java

    val requestOneTimeWork = OneTimeWorkRequestBuilder<DownloadWork>()
        .setInputData(data)
        .setConstraints(myConstraints)
        .build()

    // added enqueue one time work
    WorkManager.getInstance(this@MainActivity).enqueue(requestOneTimeWork)
    // cancel work from id
//  WorkManager.getInstance(this@MainActivity).cancelWorkById(requestOneTimeWork.id)

```

运行定期Work，每60秒运行一次。

```java

        val requestPeriodicWork = PeriodicWorkRequestBuilder<DownloadWork>(60, TimeUnit.SECONDS)
                .setInputData(data)
                .setConstraints(myConstraints)
                .build()

        WorkManager.getInstance(this@MainActivity).enqueue(requestPeriodicWork)

```

查看Work运行结果。

```java

    val liveData: LiveData<WorkInfo> = WorkManager.getInstance(this@MainActivity).getWorkInfoByIdLiveData(requestOneTimeWork.id)
        liveData.observe(this, Observer {
            if (it?.state!!.isFinished) {
                if(liveData.value?.state == WorkInfo.State.SUCCEEDED){
                    //success
                    println("download success")
                }else{
                    //other
                    println("download error")
                }
            }
    })

```

#### Worker的进阶操作

链式任务：依次有序的执行workA、workB、workC

```java 

WorkManager.getInstance()
    .beginWith(workA)
    .then(workB)   
    .then(workC)
    .enqueue()

```

并行任务：workA、workB、workC同时执行

```java 

WorkManager.getInstance()
    .beginWith(workA, workB, workC)  
    .enqueue()

```

多链式任务：让workA、workB和workC、workD两组work集并行，两组work集并行运行结束后再执行workE

```java 

val configA_B = WorkManager.getInstance()
     .beginWith(workA)
     .then(workB)
 
val configC_D = WorkManager.getInstance()
     .beginWith(workC)
     .then(workD)
 
WorkContinuation.combine(configA_B,configC_D)
     .then(workE)
     .enqueue()

```

标记任务：让有标记工作序列的work执行，被标记的work不能多次放入任务链执行

```java

WorkManager.getInstance()
    .beginUniqueWork("work tag", ExistingWorkPolicy.APPEND, workRequest)

```

## 总结

![摘自@TeaOf的掘金文章](/img/toeii/android_jetpack_worker_zozal.png)

## 结语

最后贴出文章中的[demo](https://github.com/toeii/WorkManagerSimpleExample)全部代码，以便参考学习。


