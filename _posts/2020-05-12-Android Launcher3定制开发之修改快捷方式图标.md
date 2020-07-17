---
layout:     post
title:      Android Launcher3定制开发之修改快捷方式图标
subtitle:    ""
date:       2020-05-12
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

Launcher中的应用图标有时候因为厂商系统图标的缘故导致大小不一，样式不统一等等。又或者是，业务上需要将某个包名对应的应用改成其他的图标显示，那么这样就需要修改应用的图标了。

## 具体实现

首先我们通过前几篇文章可以知道，应用图标是做了缓存的，那么我们直接看IconCache类。可以看到，IconCache中有一个getTitleAndIcon方法，应用图标、标题等信息就是通过它来获取的。我们再找到关键cacheLocked方法，并通过修改CacheEntry获取参数来自定义自己想要的应用图标。

下面写一个例子以供参考：

```java

private CacheEntry cacheLocked(ComponentName componentName, LauncherActivityInfoCompat info,
            UserHandleCompat user, boolean usePackageIcon, boolean useLowResIcon) {

    CacheEntry entry = mCache.get(componentName);
    if (entry == null) {
        entry = new CacheEntry();
        mCache.put(componentName, entry);
        ComponentName key = LauncherModel.getComponentNameFromResolveInfo(info);
        if (labelCache != null && labelCache.containsKey(key)) {
            entry.title = labelCache.get(key).toString();
        } else {
            entry.title = info.loadLabel(mPackageManager).toString();
            if (labelCache != null) {
                labelCache.put(key, entry.title);
            }
        }
        if (entry.title == null) {
            entry.title = info.activityInfo.name;
        }
        Drawable icon;
        int index = sysIndexOf(componentName.getClassName());
        icon = getFullResIcon(info);
        if (index >= 0) {
            entry.icon = Utilities.createIconBitmap(icon, mContext);
        } else {
            entry.icon = Utilities.createIconBitmap(getFullResIcon(info), mContext);
                    icon, mContext, true);
        }

        // 此处即为替换图标代码
        if("第三方应用的componentName".equals(componentName.toString())){
            entry.icon = BitmapFactory.decodeResource(mContext.getResources(), R.drawable.xxx);
        }
 
    }
    return entry;
}  

```

## 结语

如果读完本文还有不懂的地方，可以结合代码加深理解，代码通过点击[这里](https://github.com/toeii/Launcher3)找到。希望对大家学习和了解Launcher开发有所帮助。
