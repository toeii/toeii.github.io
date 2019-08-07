---
layout:     post
title:      Android JetPack全家桶(九)之DataBinding数据绑定
subtitle:    ""
date:       2019-08-07
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Android
---


## 前言

Android JetPack是Google在18年IO大会上推荐的一整套组件库，它的出现填补了之前Android中自带的一些缺陷，例如Handler的内存泄露、Camera的不易用性、后台调度难以管理等等。所以我打算把整个架构组件系统性的学习一下，在这里和大家分享，希望能帮助到其他学习者。本系列文章包含十篇：

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

#### 什么是Data Binding？
Data Binding，即数据绑定，是Android团队实现MVVM架构的一种方法，使得数据（对象）可以直接绑定到布局的xml中，数据的变化直接反映到View上。
同时，Data Binding也支持双向绑定。

#### 有什么好处

1，省去大量模板代码，比如findViewById()，setOnClickListener()， setText()，等等。<br/>
2，使得View与逻辑彻底解耦（MVVM）成为可能，不像MVC那样逻辑与View操作混在一起难以维护，也不像MVP那样定义大量接口，费时费力。<br/>
3，由于数据（对象）与View进行双向绑定，所以开发时只需要关注数据（对象）即可，无需关心View的各种繁杂操作（如setVisibility()，setText()等）。<br/>
4，功能强大，xml中即可完成简单的逻辑（xml中支持表达式语言，逻辑/数学运算等）。

## 如何使用？

### 添加依赖

app.build.gradle

```xml

android {
    ...
    dataBinding {
        enabled = true
    }
}

```

gradle.properties

```xml

    ...
    android.databinding.enableV2=true

```

### 引入xml布局

```xml

<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

         <!-- 这里可以配置DataBinding -->

    </data>

    <!-- 下面可以写当前的xml布局 -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:gravity="center"
        >

        <TextView
            android:id="@+id/tv_example"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="16sp" />

        <View> ... </View>

    </LinearLayout>

</layout>

```

### 页面绑定

#### 注意事项

**在绑定页面之前，记得先“Mark project”工程。IDEA会自动根据刚才加入<layout>的xml布局视图生成对应的DataBinding映射类。**

上面我们修改了布局后，下面我们就可以在代码中进行数据绑定了。此时工程中会自动生成该布局对应的java绑定文件:ActivityMainBinding。仔细观察就会发现，这个文件名就是将布局的下划线形式转换成java规范的驼峰形式，后面加上Binding。

#### 初始化Binding

```java

 val mBinding = DataBindingUtil.setContentView<ActivityMainBinding>(this@MainActivity,R.layout.activity_main)

 mBinding.tvExample.Text = "Binding Text";

```

此时就可以从mBinding中获取到布局中的所有View了，比如上面的mBinding.tvExample

### 一般数据绑定

#### Data标签

在这个标签中，我们通常用来做下面的事情：

1，定义所绑定的数据的名称（变量名）及对应类型；<br/>
2，引入页面所需的类；

示例如下：

```xml

<data>
    <import type="android.view.View" />
    <import type="android.text.TextUtils" />
    
    <variable name="visible" type="boolean"/>
    <variable name="title" type="String"/>
    <variable name="user" type="cn.examplecode.androiddatabinding.User"/>
</data>

```

其中import标签表示引入一个类，比如上例中引入了View类和一个工具类TextUtils，当然也可以引入你自己的类，比如常量类或者工具类。

下面variable标签定义了本页面所需要的各种数据名称或类型，其类型可以是java中的基础类型，或者自定义的类。

#### 设置数据

上面定义了页面中所需要的数据后，下面就需要通过获取到的Binding对象设置这些数据：

```java

mBinding.setVisible(true);
mBinding.setTitle("用户信息");
User user = new User();//User必须拥有get/set方法
user.setName("Steve Jobs");
mBinding.setUser(user);

```

#### 在布局中使用这些数据

数据设置完毕以后就可以在页面中使用这些数据了，使用起来也非常方便。

```xml

<TextView
    android:id="@+id/tv_title"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{title}"
    android:visibility="@{visible ? View.VISIBLE : View.GONE}"
    />
    
<TextView
    android:id="@+id/tv_username"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{user.name}"
    />

```

#### 根据View获取Binding

经过前面的学习，我们可以很方便地从数据绑定对象(Binding) 中获取到其绑定的View，但是我们会碰到一些应用场景，比如我们在操作View的时候，需要取得其绑定的Binding对象，以获取到当前View中的子View的引用，从而避免类似findViewById()这样的操作，并且也可以获取到当前View所绑定的数据。

那么该如何从View获取当前绑定的Binding对象呢？

其实很简单，DataBindingUtil就提供了这样的方法，举例如下：

```java

val userBinding = DataBindingUtil.getBinding<ViewDataBinding>(view)

```

### RecyclerView数据绑定

#### 获取binding对象

按照通常的做法，我们在Adapter中会定义一个ViewHolder，在此ViewHolder中取得一些布局View的引用。
使用DataBinding后同样可以简化Adapter中的操作。
比如我们新建布局文件view_list_item.xml:

```xml

<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>
        <variable name="example" type="String"/>
    </data>

    <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:gravity="center"
    >

        <TextView
                android:id="@+id/tv_example"
                android:layout_width="wrap_content"
                android:layout_height="100dp"
                android:gravity="center"
                android:text="@{example}"
                android:textColor="@android:color/black"
                android:textSize="24sp" />

    </LinearLayout>
</layout>

```

新建后我们会发现IDE会为我们生成一个绑定类ViewListItemBinding.java，此时我们就可以把它定义在ViewHolder中：

```java

class ItemsHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
    var binding: ViewListItemBinding? = null
    init {
        binding = DataBindingUtil.bind(itemView)
    }
}

```

接着实现RecyclerView.Adapter：

```java

class RecycleListAdapter(private var context: Context, private var datas: MutableList<Map<String,String>>) : RecyclerView.Adapter<RecycleListAdapter.ItemsHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ItemsHolder {
        val v = LayoutInflater.from(parent.context).inflate(R.layout.view_list_item, parent, false)
        return ItemsHolder(v)
    }


    override fun getItemCount(): Int {
        return datas.size
    }

    override fun onBindViewHolder(holder: ItemsHolder, position: Int) {
        val item = datas[position]
        holder.binding?.example = item["name"]
    }

}

```

上面我们创建好ViewHolder后，我们就可以在这个方法中使用它了，用法就跟之前我们在Activity中和Fragment中一样了。

### 利用自定义Interface实现onClick事件

#### 为什么要使用自定义Interface？

1，我们平常在Android的开发中，比如如果要设置一个View的点击事件，通常通过view.setOnClickListener()来实现的，这种方式略显繁琐，而且要通过findViewById()来获取到此View的引用。使用了Data Binding技术以后，我们无需这样做，可以直接通过在xml布局文件中设置一个Interface的实现来直接调用某个方法，非常方便。<br/>
2，除了上面说的方便之外，当两个Fragment之间需要通信时，Android是强烈不建议两个Fragment之间直接通信的，它们之间的通信只能通过他们所在的Activity来进行中转。使用Data Binding之后，这种情况处理起来就简单了很多，通过将一个Interface的实现设置到两个Fragment的xml布局文件中就可以实现。

#### 定义接口

```java

interface IMainActivity {

    fun clickedBeanGoing()

    fun clickedRecycleListGoing()

}

```

#### 实现并绑定接口

```java

class MainActivity : AppCompatActivity(),IMainActivity {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        //...
        binding.iMainActivity = this
    }

    override fun clickedBeanGoing() {
        startActivity(Intent(this@MainActivity,BeanBindActivity::class.java))
    }

    override fun clickedRecycleListGoing() {
        startActivity(Intent(this@MainActivity,RecycleListActivity::class.java))
    }

}

```

#### 在布局文件的标签中定义该接口的变量

```xml

<data>
        <variable
            name="iMainActivity"
            type="com.toeii.databinding.IMainActivity" 
        />
</data>

```

#### 在布局文件中设置对应的接口回调

```xml

 <Button
    android:id="@+id/btn_bean"
    android:layout_width="200dp"
    android:layout_height="80dp"
    android:text="@{beanText}"
    android:layout_marginTop="20dp"
    android:onClick="@{() -> iMainActivity.clickedBeanGoing()}"
/>

<Button
    android:id="@+id/btn_list"
    android:layout_width="200dp"
    android:layout_height="80dp"
    android:text="@{listText}"
    android:layout_marginTop="20dp"
    android:onClick="@{() -> iMainActivity.clickedRecycleListGoing()}"
/>

```

通过简单的几步就可以在布局中直接调用Activity（或任意对象）中的方法了，以简单的点击事件及简单的事件进行用法的举例，大家可以根据自己的业务应用到更多的场景中。

## DataBinding所暴露的一些问题

@以下内容摘自[我为什么放弃在项目中使用DataBinding](https://blog.csdn.net/maosidiaoxian/article/details/85560206)

1、xml代码耦合度增加，业务逻辑内聚性降低。不利于项目质量的持续发展。<br/>
2、经常需要手动点击编译，影响开发体验。 在布局里新增的Data Binding变量，在Java/Kotlin中要使用的时候需要先点击编译等待完成，否则可能引用不到对应的BR类或该类里的变量。另外，已经删除的变量，如果不执行清理，在BR类里也依然存在，无法如R类一样更新及时。<br/>
3、失去了Kotlin语法糖的优势。 Kotlin扩展函数的特点可以使得代码尽可能的简洁直观易于阅读，而在xml中目前只支持Java语法而不支持Kotlin，所以也失去了使用Kotlin作为开发语言所带来的优势。

## 结语

最后贴出文章中的[demo](https://github.com/toeii/DataBindingSimpleExample)全部代码，以便参考学习。



