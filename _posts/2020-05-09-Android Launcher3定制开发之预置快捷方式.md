---
layout:     post
title:      Android Launcher3定制开发之预置应用
subtitle:    ""
date:       2020-05-09
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

大部分Launcher定制都是需要将某些主要的业务应用放在第一屏或者在Hotseat摆放默认应用，本篇文章将教大家如何直接实现预置应用。

## 找到入口

InvariantDeviceProfile类是Launcher的默认配置加载类，通过InvariantDeviceProfile方法可以看出，CellLayout显示的应用行数和列数可以通过findClosestDeviceProfiles查询XML配置来读取配参。

那么，我们先将适合自己业务的XML文件先声明。我这里定义的是一个5X4的布局(关于布局排列如何自定义后面会说)，同时第一屏预置了微信
QQ、相册、设置，同时默认将电话、短信、相机、浏览器预置在Hotseat上显示。详细代码和注释见下：

default_workspace.xml

```xml

<?xml version="1.0" encoding="utf-8"?>
<favorites xmlns:launcher="http://schemas.android.com/apk/res-auto/com.android.launcher3">

    <!-- 微信 -->
    <resolve
        launcher:screen="1" //显示在第几屏
        launcher:x="0" //显示在x轴第几个
        launcher:y="4"> // 显示在y轴第几个
        <favorite launcher:packageName="com.tencent.mm" launcher:className="com.tencent.mm.ui.LauncherUI" /> //通过包名和启动类名找到应用主路由地址（Tips：可配置多个，当第一个失效则默认往下筛选，多数时候为了兼容性可以多写几个）
    </resolve>

    <!-- QQ -->
    <resolve
        launcher:screen="1"
        launcher:x="1"
        launcher:y="4">
        <favorite launcher:packageName="com.tencent.mobileqq" launcher:className="com.tencent.mobileqq.activity.SplashActivity" />
    </resolve>

    <!-- 相册 -->
    <resolve
        launcher:screen="1"
        launcher:x="2"
        launcher:y="4">
        <favorite launcher:uri="#Intent;action=android.intent.action.MAIN;category=android.intent.category.APP_GALLERY;end" />
        <favorite launcher:uri="#Intent;type=images/*;end" />
        <favorite launcher:packageName="com.miui.gallery"  launcher:className="com.miui.gallery.activity.HomePageActivity" />
        <favorite launcher:packageName="com.android.gallery3d" launcher:className="com.huawei.gallery.app.GalleryMain" />
        <favorite launcher:packageName="com.coloros.gallery3d" launcher:className="com.oppo.gallery3d.app.Gallery" />
        <favorite launcher:packageName="com.vivo.gallery" launcher:className="com.android.gallery3d.vivo.GalleryTabActivity" />
    </resolve>

    <!-- 设置 -->
    <resolve
        launcher:screen="1"
        launcher:x="3"
        launcher:y="4">
        <favorite launcher:uri="#Intent;action=android.settings.SETTINGS;end" />
        <favorite  launcher:packageName="com.android.settings" launcher:className="com.android.settings.Settings" />
        <favorite  launcher:packageName="com.android.settings" launcher:className="com.android.settings.MainSettings" />
        <favorite  launcher:packageName="com.android.settings" launcher:className="com.android.settings.HWSettings" />
        <favorite  launcher:packageName="com.android.settings" launcher:className="com.oppo.settings.SettingsActivity" />
    </resolve>

    <!-- 电话 -->
    <resolve
        launcher:container="-101"
        launcher:screen="0"
        launcher:x="0"
        launcher:y="0" >
        <favorite launcher:uri="#Intent;action=android.intent.action.DIAL;end" />
        <favorite launcher:uri="tel:123" />
        <favorite launcher:uri="#Intent;action=android.intent.action.CALL_BUTTON;end" />
    </resolve>

    <!-- 短信 -->
    <resolve
        launcher:container="-101"
        launcher:screen="1"
        launcher:x="1"
        launcher:y="0" >
        <favorite launcher:uri="#Intent;action=android.intent.action.MAIN;category=android.intent.category.APP_MESSAGING;end" />
        <favorite launcher:uri="sms:" />
        <favorite launcher:uri="smsto:" />
        <favorite launcher:uri="mms:" />
        <favorite launcher:uri="mmsto:" />
    </resolve>

    <!-- 相机 -->
    <resolve
        launcher:container="-101"
        launcher:screen="2"
        launcher:x="2"
        launcher:y="0" >
        <favorite launcher:uri="#Intent;action=android.media.action.STILL_IMAGE_CAMERA;end" />
        <favorite launcher:uri="#Intent;action=android.intent.action.CAMERA_BUTTON;end" />
    </resolve>

    <!-- 浏览器 -->
    <resolve
        launcher:container="-101"
        launcher:screen="3"
        launcher:x="3"
        launcher:y="0" >
        <favorite launcher:uri="#Intent;action=android.intent.action.MAIN;category=android.intent.category.APP_BROWSER;end" />
        <favorite launcher:uri="http://www.example.com/" />
        <favorite launcher:packageName="com.android.browser" launcher:className="com.android.browser.BrowserActivity" />
        <favorite launcher:packageName="com.huawei.browser" launcher:className="com.huawei.browser.Main2" />
    </resolve>

</favorites>

```

解释一下，resolve就在favorites中ViewGroup.addView。现在国产自研ROM分化严重，所以favorite兼容性上处理起来会麻烦一点，需要分不同品牌应用。

## 修改行数和列数

这样，我们已经把预置配置文件创建好了，接下来回到InvariantDeviceProfile.findClosestDeviceProfiles。

可以看到，findClosestDeviceProfiles在源码上根据不同的分辨率设备，分别读取了不同的机型，有的是4X4，有的是4X5，有的是5X6。这里面不符合我们现在的业务需求，我们需要将行数和列数改成5X4并且保证兼容性。所以，我对代码做了如下修改，将所有分辨率规格改为了5X4，同时不同分辨率加载不同大小的快捷图标，见下文代码和注释：

```java

    InvariantDeviceProfile(Context context) {
        WindowManager wm = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
        Display display = wm.getDefaultDisplay();
        DisplayMetrics dm = new DisplayMetrics();
        display.getMetrics(dm);

        Point smallestSize = new Point();
        Point largestSize = new Point();
        display.getCurrentSizeRange(smallestSize, largestSize);

        minWidthDps = Utilities.dpiFromPx(Math.min(smallestSize.x, smallestSize.y), dm);
        minHeightDps = Utilities.dpiFromPx(Math.min(largestSize.x, largestSize.y), dm);

        //图标排列兼容处理
        ArrayList<InvariantDeviceProfile> closestProfiles =
                findClosestDeviceProfiles(minWidthDps, minHeightDps,
                        getPredefinedDeviceProfiles(R.xml.default_workspace));

        InvariantDeviceProfile interpolatedDeviceProfileOut =
                invDistWeightedInterpolate(minWidthDps,  minHeightDps, closestProfiles);

        InvariantDeviceProfile closestProfile = closestProfiles.get(0);
        numRows = closestProfile.numRows;
        numColumns = closestProfile.numColumns;
        numHotseatIcons = closestProfile.numHotseatIcons;
        hotseatAllAppsRank = (int) (numHotseatIcons / 2);
        defaultLayoutId = closestProfile.defaultLayoutId;
        defaultNoAllAppsLayoutId = closestProfile.defaultNoAllAppsLayoutId;
        numFolderRows = closestProfile.numFolderRows;
        numFolderColumns = closestProfile.numFolderColumns;
        minAllAppsPredictionColumns = closestProfile.minAllAppsPredictionColumns;
        iconSize = interpolatedDeviceProfileOut.iconSize;
        iconBitmapSize = Utilities.pxFromDp(iconSize, dm);
        iconTextSize = interpolatedDeviceProfileOut.iconTextSize;
        hotseatIconSize = interpolatedDeviceProfileOut.hotseatIconSize;
        fillResIconDpi = getLauncherIconDensity(iconBitmapSize);

        applyPartnerDeviceProfileOverrides(context, dm);

        Point realSize = new Point();
        display.getRealSize(realSize);
        int smallSide = Math.min(realSize.x, realSize.y);
        int largeSide = Math.max(realSize.x, realSize.y);

        landscapeProfile = new DeviceProfile(context, this, smallestSize, largestSize,
                largeSide, smallSide, true /* isLandscape */);
        portraitProfile = new DeviceProfile(context, this, smallestSize, largestSize,
                smallSide, largeSide, false /* isLandscape */);
    }

    /**
    * 不同分辨率统一处理5X4
    */
    ArrayList<InvariantDeviceProfile> getPredefinedDeviceProfiles(int xmlId) {
        ArrayList<InvariantDeviceProfile> predefinedDeviceProfiles = new ArrayList<>();
        // width, height, #rows(行数), #columns(列数), #folder rows, #folder columns,
        // iconSize, iconTextSize, #hotseat, #hotseatIconSize, defaultLayoutId.
        // TODO 这里可以根据业务定义常量，规范参数。
        predefinedDeviceProfiles.add(new InvariantDeviceProfile("Super Short Stubby",
                255, 300,     5, 4, 4, 4, 4, SMALL_ICON_SIZE_DP, 13, 4, 48,xmlId));
        predefinedDeviceProfiles.add(new InvariantDeviceProfile("Shorter Stubby",
                255, 400,     5, 4, 4, 4, 4, SMALL_ICON_SIZE_DP, 13, 4, 48, xmlId));
        predefinedDeviceProfiles.add(new InvariantDeviceProfile("Short Stubby",
                275, 420,     5, 4, 4, 4, 4, SMALL_ICON_SIZE_DP, 13, 4, 48, xmlId));
        predefinedDeviceProfiles.add(new InvariantDeviceProfile("Stubby",
                255, 450,     5, 4, 4, 4, 4, SMALL_ICON_SIZE_DP, 13, 4, 48, xmlId));
        predefinedDeviceProfiles.add(new InvariantDeviceProfile("Nexus S",
                296, 491.33f, 5, 4, 4, 4, 4, DEFAULT_ICON_SIZE_DP, 13, 4, 48, xmlId));
        predefinedDeviceProfiles.add(new InvariantDeviceProfile("Nexus 4",
                335, 567,     5, 4, 4, 4, 4, DEFAULT_ICON_SIZE_DP, 13, 4, 56, xmlId));
        predefinedDeviceProfiles.add(new InvariantDeviceProfile("Nexus 5",
                359, 567,     5, 4, 4, 4, 4, DEFAULT_ICON_SIZE_DP, 13, 4, 56, xmlId));
        predefinedDeviceProfiles.add(new InvariantDeviceProfile("Large Phone",
                406, 694,     5, 4, 4, 4, 4, DEFAULT_ICON_SIZE_DP, 14.4f,  4, 56, xmlId));
        predefinedDeviceProfiles.add(new InvariantDeviceProfile("Nexus 7",
                575, 904,     5, 4, 4, 4, 4, BIG_ICON_SIZE_DP, 14.4f,  4, 60, xmlId));
        predefinedDeviceProfiles.add(new InvariantDeviceProfile("Nexus 10",
                727, 1207,    5, 4, 4, 4, 4, BIG_ICON_SIZE_DP, 14.4f,  4, 64, xmlId));
        predefinedDeviceProfiles.add(new InvariantDeviceProfile("20-inch Tablet",
                1527, 2527,   5, 4, 4, 4, 4, 100, 20,  4, 72, xmlId));
        return predefinedDeviceProfiles;
    }

```

通过以上配置，就可以自定义自己想要的行数和列数以及预置应用了。

## 结语

如果读完本文还有不懂的地方，可以结合代码加深理解，代码通过点击[这里](https://github.com/toeii/Launcher3)找到。希望对大家学习和了解Launcher开发有所帮助。




