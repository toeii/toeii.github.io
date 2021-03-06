---
layout:     post
title:      Android Jetpack全家桶(五)之Room ORM库
subtitle:    ""
date:       2019-07-17
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

Room是基于SQLite实现的，一款具备比较全面的SQLite功能的ORM库。可以理解为它是对SQLite的增强优化版。

平时在日常开发中，ORM库用得比较多的是GreenDao、LitePal、Realm等优秀的开源库，但是它们都是第三方的。而如今Google自己推出了一款ORM库，也就是本篇要介绍的Room。毕竟是亲儿子，我们作为Android开发者找不到不去学习它的理由。接下来我们一起来看看吧。

（Tips：ORM是建立关系对象映射，也就是我们常用到的数据持久）

## 如何使用？

#### 一，添加依赖

App.build.gradle

```java

// room
def room_version = '2.2.0-alpha01'
implementation "androidx.room:room-runtime:$room_version"
implementation "androidx.room:room-ktx:$room_version"
kapt "androidx.room:room-compiler:$room_version"
annotationProcessor "android.arch.persistence.room:compiler:$room_version"
kapt "android.arch.persistence.room:compiler:$room_version"

```

#### 二，初始化DB

```java

class App: Application() {
    companion object{
        var instance: App? = null
    }
    var db: AppDatabase? = null
    override fun onCreate() {
        super.onCreate()
        instance = this
        db = Room.databaseBuilder(
            applicationContext,
            AppDatabase::class.java, "db_book"
        ).build()
    }
}

```

#### 三，建立实体类

```java

@Entity
data class BookBean(
    @PrimaryKey var uid: Int,
    @ColumnInfo(name = "book_name") var bookName: String?,
    @ColumnInfo(name = "is_like") var isLike: Boolean?
)

```

#### 四，建立Dao层

```java

@Dao
interface BookDao {

    @Insert
    fun insertAll(books: List<BookBean>)

    @Delete
    fun delete(book: BookBean)

    @Update
    fun updateBook(book: BookBean)

    @Query("SELECT * FROM bookbean")
    fun getAll(): List<BookBean>

    @Query("SELECT * FROM bookbean WHERE is_like IN (:isLike)")
    fun getAllByLike(isLike: Boolean): List<BookBean>


}

```

```java

@Database(entities = [BookBean::class], version = 1,exportSchema = false)
abstract class AppDatabase : RoomDatabase() {

    abstract fun bookDao(): BookDao

    //...Other Dao

}

```

#### 五，Dao层业务实现

```java

// Room added data
val bookStore = arrayOf("...")
val books: MutableList<BookBean> = mutableListOf()
for ((index, bookName) in bookStore.withIndex()){
    val book = BookBean(index,bookName,false)
    books.add(book)
}
App.instance?.db?.bookDao()?.insertAll(books)

```

```java

doAsync {
    // Room delete data
    val delResult = App.instance?.db?.bookDao()?.delete(datas[position])
    if(null != delResult){
        uiThread {
            datas -= datas[position]
            notifyDataSetChanged()
        }
    }
}

```

```java

doAsync {
    // Room update data
    datas[position].isLike = !datas[position].isLike!!
    App.instance?.db?.bookDao()?.updateBook(datas[position])

    uiThread {
        notifyDataSetChanged()
    }
}

```

```java

doAsync {
    // Query data for Room
    mDatas = App.instance?.db?.bookDao()?.getAll()!!
    if(mDatas.isEmpty()){
        initData()
    }
    uiThread {
        mBookAdapter = BookAdapter(this@BookActivity,mDatas)
        mRecyclerView.adapter = mBookAdapter
        mBookAdapter.notifyDataSetChanged()
    }
}

```


```java

doAsync {
    // Query like data for Room
    mDatas = App.instance?.db?.bookDao()?.getAllByLike(true)!!
    uiThread {
        if(mDatas.isNotEmpty()){
            for(bookBean:BookBean in mDatas){
                mStringBuilder.append(bookBean.bookName)
                mStringBuilder.append(",")
            }
            mTvLike.text = "喜欢的书：$mStringBuilder"
        }else{
            mTvLike.text = "喜欢的书：暂无（请在主页添加）"
        }
    }
}

```

## 分析Room

如图所示：

![图片介绍](/img/toeii/android_jetpack_room_architecture.png)

Room有3个主要部分：

数据库：包含数据库持有者，并作为应用程序持久关系数据的基础连接的主要访问点。

使用@Database注释的类应满足以下条件：
1，是一个扩展RoomDatabase的抽象类。
2，在注释中包括与数据库关联的实体列表。
3，包含一个具有0个参数的抽象方法，并返回使用@Dao注释的类。

在运行时，您可以通过调用Room.databaseBuilder（）或Room.inMemoryDatabaseBuilder（）来获取Database实例。

实体：表示数据库中的表。

DAO：包含用于访问数据库的方法。

该应用程序使用Room数据库来获取与该数据库关联的数据访问对象或DAO。
然后，应用程序使用每个DAO从数据库中获取实体，并将对这些实体的任何更改保存回数据库。
最后，应用程序使用实体来获取和设置与数据库中的表列对应的值。

## 注意事项

实现查询和修改操作时，需要在异步调用Room，如果不在异步调用则会报出以下异常：

```XML

Caused by: java.lang.IllegalStateException: Cannot access database on the main thread since it may potentially lock the UI for a long periods of time.

```

## 了解更多

有关于数据库升级和数据库文件导出，可以查阅[这篇文章](https://www.jianshu.com/p/3e358eb9ac43)

## 结语

最后，希望大家可以结合[demo](https://github.com/toeii/RoomSimpleExample)进行拓展性的学习。




