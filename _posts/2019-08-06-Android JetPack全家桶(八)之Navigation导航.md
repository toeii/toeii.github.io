---
layout:     post
title:      Android JetPack全家桶(八)之Navigation导航
subtitle:    ""
date:       2019-08-06
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

Navigation导航是指允许用户在应用内的不同内容中导航，导入和退出的交互。利用它我们能够更好的实现Fragment的管理，轻松实现单个Activity和多个Fragment的交互模式，这种交互模式其实也是一个APP最“科学”的形式。

## 和第三方框架比较

类似的导航库其实在开源第三方项目中早已存在，例如优秀的[Fragmentation](https://github.com/YoKeyword/Fragmentation/)和[FragmentRigger](https://github.com/JingYeoh/FragmentRigger/)，它们都趋向成熟且有一定规模。那么Navigation和它们相比又有什么优势？

首先，Navigation职责单一，目前来看仅对Fragment做导航管理（导航该页或回退该页）。
二是，Navigation的兼容性更好，能保障向上兼容性。
三是，Navigation的后续扩展性，从代码设计上可以看出，Navigation并非只为Fragment提供导航。

## 如何使用？

#### 添加依赖

app.build.gradle

```java

dependencies {
     
    ...

    def navigation_version = '2.1.0-beta02'
    implementation "androidx.navigation:navigation-runtime-ktx:$navigation_version"
    implementation "androidx.navigation:navigation-fragment-ktx:$navigation_version"
    implementation "androidx.navigation:navigation-ui-ktx:$navigation_version"
}

```

#### 新建fragment

这里是我所写的Demo示例部分代码。

ContactList.kt

```java

class ContactList : Fragment(){

    private lateinit var datas : MutableList<Map<String,String>>

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {

        datas = arrayListOf()

        initData()//...

        val view = inflater.inflate(R.layout.fragment_contact, container, false)

        view.findViewById<RecyclerView>(R.id.rl_contact).adapter = ContactAdapter(activity!!.applicationContext,datas)

        return view

    }

}


class ContactAdapter(private var context: Context,private var datas: MutableList<Map<String,String>>) : RecyclerView.Adapter<ContactAdapter.ContactHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ContactHolder {
        val view = LayoutInflater.from(context).inflate(R.layout.view_contact_list_item,null)
        return ContactHolder(view)
    }

    override fun getItemCount(): Int {
        return datas.size
    }

    override fun onBindViewHolder(holder: ContactHolder, position: Int) {
        val mapData = datas[position]
        holder.itemView.iv_head_icon.setImageResource((mapData["icon"] ?: error("")).toInt())
        holder.itemView.tv_name.text = mapData["name"]
        holder.itemView.rl_layout.setOnClickListener {
            val builder = Bundle()
            builder.putString("phone",mapData["phone"])
            builder.putString("name", mapData["name"])
            Navigation.findNavController(holder.itemView).navigate(R.id.action_contactList_to_contactMessage,builder)
        }
    }

    class ContactHolder(itemView: View): RecyclerView.ViewHolder(itemView)

}

```

ContactMessage.kt

```java

class ContactMessage : Fragment(){

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {

        val view = inflater.inflate(R.layout.fragment_contact_msg, container, false)

        val phone = arguments?.getString("phone") ?: "null"
        val name = arguments?.getString("name") ?: "null"
        view.findViewById<TextView>(R.id.tv_about).text = name

        view.findViewById<Button>(R.id.btn_look).setOnClickListener {
            val builder = Bundle()
            builder.putString("phone",phone)
            builder.putString("name",name)
            Navigation.findNavController(view).navigate(R.id.action_contactMessage_to_about,builder)
        }

        view.findViewById<Button>(R.id.btn_back).setOnClickListener {
            Navigation.findNavController(view).popBackStack()
        }

        return view

    }

}

```


#### 配置Navigation Graph

res目录下新建navigation目录，并创建导航结构图nav_graph.xml

其中，action是导航行为，可以被NavController所指定调用。argument是携带参数，可以单独配置。

```java

<?xml version="1.0" encoding="utf-8"?>
<navigation
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:id="@+id/navigation"
        app:startDestination="@id/contactList">

    <fragment android:id="@+id/contactList" android:name="com.toeii.navigation.contact.ContactList"
              android:label="ContactList">
        <action android:id="@+id/action_contactList_to_contactMessage"
                app:destination="@id/contactMessage"
        />
    </fragment>

     <fragment android:id="@+id/contactMessage" android:name="com.toeii.navigation.contact.ContactMessage"
              android:label="ContactMessage">
        <argument android:name="phone"
                  android:defaultValue="null"/>

        <action android:id="@+id/action_contactMessage_to_contactList"
                app:destination="@id/contactList"
        />
    </fragment>

    //...

</navigation>

```

#### 定义节点NavHost

设置app:defaultNavHost="true"和导航目录加载于指定fragment之上

```java

<?xml version="1.0" encoding="utf-8"?>
<FrameLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        xmlns:app="http://schemas.android.com/apk/res-auto"
>

    <fragment
            android:id="@+id/fl_main"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:name="androidx.navigation.fragment.NavHostFragment"
            app:defaultNavHost="true"
            app:navGraph="@navigation/nav_graph"
    >

    </fragment>

</FrameLayout>

```

#### NavController调用栈

携带Bundle的导航行为

``` java

    val builder = Bundle()
    builder.putString("phone",mapData["phone"])
    builder.putString("name", mapData["name"])
    Navigation.findNavController(view).navigate(R.id.action_contactList_to_contactMessage,builder)

```

导航回退行为

``` java

    Navigation.findNavController(view).popBackStack()

```

## 了解更多

本篇结合Demo介绍了Navigation的基本用法，如果想了解更多进阶知识，可以查阅官方资料。

[深度链接](https://developer.android.google.cn/guide/navigation/navigation-deep-link)、
[条件导航](https://developer.android.google.cn/guide/navigation/navigation-conditional)、
[动画转换](https://developer.android.google.cn/guide/navigation/navigation-animate-transitions)

## 结语

最后贴出文章中的[demo](https://github.com/toeii/NavigationSimpleExample)全部代码，以便参考学习。




