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

上篇介绍了[Paging](https://toeii.github.io/2019/07/19/Android-JetPack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%85%AD)%E4%B9%8BPaging%E5%88%86%E9%A1%B5%E5%BA%93/)的用法和使用场景，这篇接着说说Jetpack提供的新的延迟异步方案WorkManager。

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

        val requestPeriodicWork = PeriodicWorkRequestBuilder<DownloadWork>(60, TimeUnit.SECONDS)//
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


