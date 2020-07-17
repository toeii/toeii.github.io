---
layout:     post
title:      Android Launcher3定制开发之创建小部件
subtitle:    ""
date:       2020-05-08
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Android
---



## 前言

为了加深对Launcher3的整体印象，记录在工作上所遇到和解决的关于Launcher3定制开发的一些知识点，并且归纳总结成本系列文章。同时，也能起到帮助他人解决一些关于Launcher3相关定制的需求开发。

- [Android Launcher3定制开发之结构](https://toeii.github.io/2020/05/06/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E7%BB%93%E6%9E%84/)<br />
- [Android Launcher3定制开发之改造横向化](https://toeii.github.io/2020/05/07/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E6%94%B9%E9%80%A0%E6%A8%AA%E5%90%91%E5%8C%96/)<br />
- [Android Launcher3定制开发之创建小部件](https://toeii.github.io/2020/05/08/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E5%88%9B%E5%BB%BA%E5%B0%8F%E9%83%A8%E4%BB%B6/)<br />
- [Android Launcher3定制开发之预置快捷方式](https://toeii.github.io/2020/05/09/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E9%A2%84%E7%BD%AE%E5%BF%AB%E6%8D%B7%E6%96%B9%E5%BC%8F/)<br />
- [Android Launcher3定制开发之去除搜索和开机引导](https://toeii.github.io/2020/05/11/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E5%8E%BB%E9%99%A4%E6%90%9C%E7%B4%A2/)<br />
- [Android Launcher3定制开发之修改快捷方式图标](https://toeii.github.io/2020/05/12/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E4%BF%AE%E6%94%B9%E5%BF%AB%E6%8D%B7%E6%96%B9%E5%BC%8F%E5%9B%BE%E6%A0%87/)<br />
- [Android Launcher3定制开发之添加负一屏](https://toeii.github.io/2020/05/14/Android-Launcher3%E5%AE%9A%E5%88%B6%E5%BC%80%E5%8F%91%E4%B9%8B%E6%B7%BB%E5%8A%A0%E8%B4%9F%E4%B8%80%E5%B1%8F/)<br />


## 简介

小部件在官方文档里也叫微件，通常是显示在launcher上的窗口view。例如，时间，天气，搜索栏，音乐视频等媒体快捷操作栏。

## 效果图

![时间小部件](/img/toeii/launcher_apptimewidget.jpg)

## 如何创建

### 声明小部件

首先，声明应用小部件，拿到AppWidgetProviderInfo和AppWidgetProvider类对象。下面是官方文档对它们的描述：

- AppWidgetProviderInfo：描述应用小部件的元数据，如应用小部件的布局、更新频率和 AppWidgetProvider 类。此对象应在 XML 中定义。

- AppWidgetProvider：描述应用小部件的元数据，如应用小部件的布局、更新频率和 AppWidgetProvider 类。此对象应在 XML 中定义。

```xml

     <receiver
            android:name="com.android.launcher3.widget.AppTimeWidgetProvider"
            android:label="lxm_launcher_widget" >
            <intent-filter>
                <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
            </intent-filter>
            <meta-data
                android:resource="@xml/default_appwidget"
                android:name="android.appwidget.provider">
            </meta-data>
    </receiver>

```

receiver元素需要android:name属性，该属性指定应用小部件使用的AppWidgetProvider路径。而label就是别名，方便之后找到它。

接着，添加AppTimeWidgetProvider元数据。

```xml

    <appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
        android:initialLayout="@layout/widget"
        android:minHeight="84dp"
        android:minWidth="84dp"
        android:widgetCategory="home_screen"
        android:updatePeriodMillis="0" >
    </appwidget-provider>

```

以下是官方文档对appwidget-provider属性的解释：<br />
1. minWidth 和 minHeight 属性的值指定应用小部件默认情况下占用的最小空间。默认的主屏幕根据定义了高度和宽度的单元格的网格在其窗口中放置应用小部件。如果应用小部件的最小宽度或高度的值与单元格的尺寸不匹配，则应用小部件的尺寸会向上舍入到最接近的单元格大小。<br />
2. minResizeWidth 和 minResizeHeight 属性指定应用小部件的绝对最小大小。这些值应指定应用小部件的大小低于多大就会难以辨认或无法使用。使用这些属性，用户可以将小部件的大小调整为可能小于由 minWidth 和 minHeight 属性定义的默认小部件大小。这些属性是在 Android 3.1 中引入的。<br />
3. updatePeriodMillis 属性定义应用小部件框架通过调用 onUpdate() 回调方法来从 AppWidgetProvider 请求更新的频率应该是多大。不能保证实际更新按此值正好准时发生，我们建议尽可能降低更新频率 - 或许不超过每小时一次，以节省电池电量。您也可以允许用户在配置中调整频率 - 有些人可能希望股票行情自动收录器每 15 分钟更新一次，另有一些人也可能希望它一天只更新 4 次。<br />
4. initialLayout 属性指向用于定义应用小部件布局的布局资源。
configure 属性定义要在用户添加应用小部件时启动以便用户配置应用小部件属性的 Activity。这是可选的（请阅读下文的创建应用小部件配置 Activity）。<br />
5. previewImage 属性指定预览来描绘应用小部件经过配置后是什么样子的，用户在选择应用小部件时会看到该预览。如果未提供，则用户会看到应用的启动器图标。此字段对应于 AndroidManifest.xml 文件的 <receiver> 元素中的 android:previewImage 属性。如需详细了解如何使用 previewImage，请参阅设置预览图片。此属性是在 Android 3.0 中引入的。<br />
6. autoAdvanceViewId 属性指定应由应用小部件的托管应用自动跳转的应用小部件子视图的视图 ID。此属性是在 Android 3.0 中引入的。<br />
7. resizeMode 属性指定可以按什么规则来调整小部件的大小。您可以使用此属性来让主屏幕小部件在横轴上可调整大小、在纵轴上可调整大小，或者在这两个轴上均可调整大小。用户可轻触并按住小部件以显示其大小调整手柄，然后拖动水平和/或垂直手柄以更改布局网格上的大小。resizeMode 属性的值包括“horizontal”、“vertical”和“none”。<br />
8. minResizeHeight 属性指定可将小部件大小调整到的最小高度（以 dp 为单位）。如果此字段的值大于 minHeight 或未启用垂直大小调整（请参阅 resizeMode），则此字段不起作用。此属性是在 Android 4.0 中引入的。<br />
9. minResizeWidth 属性指定可将小部件大小调整到的最小宽度（以 dp 为单位）。如果此字段的值大于 minWidth 或未启用水平大小调整（请参阅 resizeMode），则此字段不起作用。此属性是在 Android 4.0 中引入的。<br />
10. widgetCategory 属性声明应用小部件是否可以显示在主屏幕 (home_screen) 和/或锁定屏幕 (keyguard) 上。只有低于 5.0 的 Android 版本才支持锁定屏幕小部件。对于 Android 5.0 及更高版本，只有 home_screen 有效。<br />

之后，创建小部件Widget Layout布局文件

```xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:gravity="center"
    >

    <TextClock
        android:id="@+id/tcTime"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:format24Hour="HH:mm"
        android:format12Hour="hh:mm"
        android:gravity="center"
        android:textColor="#FFF"
        android:textSize="60sp"
        android:text="123"
        />

    <TextClock
        android:id="@+id/tcDate"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:textColor="#FFF"
        android:textSize="15sp"
        android:format24Hour="MM月dd日 EEEE"
        android:format12Hour="MM月dd日 EEEE" />

</LinearLayout>

```

然后创建AppTimeWidgetProvider类。

```java

public class AppTimeWidgetProvider extends AppWidgetProvider {

    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if (action != null) {
            if(action.equals("android.appwidget.action.APPWIDGET_UPDATE")){
                updateWidget(context);
            } else {
                super.onReceive(context, intent);
            }
        }
    }

    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        updateWidget(context);
    }

    private void updateWidget(Context context) {
        RemoteViews remoteViews = new RemoteViews(context.getPackageName(), R.layout.widget);
        AppWidgetManager appwidgetManager = AppWidgetManager.getInstance(context);
        ComponentName componentname = new ComponentName(context, AppTimeWidgetProvider.class);
        appwidgetManager.updateAppWidget(componentname, remoteViews);
    }

}

```

其中，onUpdate方法可以按 AppWidgetProviderInfo 中的 updatePeriodMillis 属性定义的时间间隔来更新小部件。

### 在Launcher上主动显示小部件

大致思路如下，需要先拿到AppWidgetManager实例，然后检测AppWidgetHostView中小部件是否存在，接着取出所声明的所有小部件，接着添加在CellLayout中。见下列代码和注释：

```java

    //拿到AppWidgetManager
    AppWidgetManagerCompat wm = AppWidgetManagerCompat.getInstance(this);
    //检测AppWidgetHostView中小部件是否存在，存在则清空
    if(null != mQsb){
        reinflateQSBIfNecessary();
    }
    //取出所声明的所有小部件
    LauncherAppWidgetProviderInfo info;
    List<AppWidgetProviderInfo> widgets = wm.getAllProviders();
    for (AppWidgetProviderInfo pInfo : widgets) {
        info = LauncherAppWidgetProviderInfo.fromProviderInfo(this, pInfo);
        //匹配之前声明的receiver别名
        if(info.label.equals("lxm_launcher_widget")){
            mPendingAddInfo.screenId = 1; //指定在第一屏
            mPendingAddInfo.container = -100; 
            mPendingAddInfo.spanX = 4; //占位4X
            mPendingAddInfo.spanY = 2; //占位2Y
            mPendingAddInfo.user =  wm.getUser(info);
            mPendingAddWidgetInfo = info;
            Intent intent = new Intent(AppWidgetManager.ACTION_APPWIDGET_BIND);
            intent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, mPendingAddInfo.id);
            intent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_PROVIDER, info.provider);
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                mPendingAddInfo.user.addToIntent(intent, AppWidgetManager.EXTRA_APPWIDGET_PROVIDER_PROFILE);
            }
            startActivityForResult(intent, REQUEST_ADDED_APPWIDGET);//申请创建（需用户授权）
        }
    }

    ...
 
    private void handleActivityResult(
            final int requestCode, final int resultCode, final Intent data) {

        setWaitingForResult(false);
        final int pendingAddWidgetId = mPendingAddWidgetId;
        mPendingAddWidgetId = -1;

        Runnable exitSpringLoaded = new Runnable() {
            @Override
            public void run() {
                exitSpringLoadedDragModeDelayed((resultCode != RESULT_CANCELED),
                        EXIT_SPRINGLOADED_MODE_SHORT_TIMEOUT, null);
            }
        };

        ...

        // 将创建好的小部件添加至CellLayout
        if (requestCode == REQUEST_ADDED_APPWIDGET) {
            final int appWidgetId = data != null ? data.getIntExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, 9527) : 9527;
            addAppWidgetImpl(appWidgetId, mPendingAddInfo, null,
                    mPendingAddWidgetInfo, ON_ACTIVITY_RESULT_ANIMATION_DELAY);
            getOrCreateQsbBar();
            return;
        } 

        ...

    }

```

## 结语

到这里，Launcher中的小部件应用场景基本讲诉完毕，如果还有不懂的可以查看本系列文章相关代码，通过点击[这里](https://github.com/toeii/Launcher3)找到。希望对大家学习和了解Launcher开发有所帮助。