---
layout:     post
title:      Android Launcher3定制开发之改造横向化
subtitle:    ""
date:       2020-05-07
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

因为国内用户大多数习惯桌面横向滑动，所以很多时候需要把原生luancher从抽屉形态改造成横向形态。我发现这方面的资料不多，所以自己研究了一下，并写下这篇博客记录。

## 思路

- 修改Hotseat中的allAppsButton相关业务
- 将AllAppsContainerView中的所有应用放在LoadTask加载
- 检测替换APP时并更新Workspace
- 改造Workspace的长按呼出菜单

一句话概括就是“屏蔽原有抽屉化业务代码，将数据全放入第一层桌面，然后新增更新第一层桌面的相关业务代码”

## 具体实现

### 修改Hotseat中的allAppsButton相关业务

这里主要是为了去除Hotseat中的allAppsButton加载，并去除上滑打开抽屉桌面相关代码。

因此，我直接屏蔽了Hotseat中的addViewToCellLayout关键代码。

```java

   void resetLayout() {
        mContent.removeAllViewsInLayout();
       /* 
            ...

            mContent.addViewToCellLayout(allAppsButtonallAppsButton, -1, allAppsButton.getId(), lp, true);

            ...
        */
    }

```

接着，处理在LauncherModel屏蔽数据集加载

```java

       private boolean checkItemPlacement(LongArrayMap<ItemInfo[][]> occupied, ItemInfo item,
                   ArrayList<Long> workspaceScreens) {
            LauncherAppState app = LauncherAppState.getInstance();
            InvariantDeviceProfile profile = app.getInvariantDeviceProfile();
            final int countX = profile.numColumns;
            final int countY = profile.numRows;

            long containerIndex = item.screenId;
            if (item.container == LauncherSettings.Favorites.CONTAINER_HOTSEAT) {
                //TODO 过滤Main HotSeat处理
                //    if (mCallbacks == null ||
                //            mCallbacks.get().isAllAppsButtonRank((int) item.screenId)) {
                //        Log.e(TAG, "Error loading shortcut into hotseat " + item
                //                + " into position (" + item.screenId + ":" + item.cellX + ","
                //                + item.cellY + ") occupied by all apps");
                //        return false;
                //    }
            }
            ...
        }

```

然后在PackageUpdatedTask类中屏蔽掉多余的数据初始化，避免不必要的操作。

```java

final Callbacks callbacks = getCallback();

if (callbacks == null) {
    return;
}

final HashMap<ComponentName, AppInfo> addedOrUpdatedApps =
        new HashMap<ComponentName, AppInfo>();

if (added != null) {

//  addAppsToAllApps(context, added);

    final ArrayList<ItemInfo> addedInfos = new ArrayList<ItemInfo>(added);
    addAndBindAddedWorkspaceItems(context, addedInfos);

    for (AppInfo ai : added) {
        addedOrUpdatedApps.put(ai.componentName, ai);
    }

}


```

### 将AllAppsContainerView中的所有应用放在LoadTask加载

找到LoaderTask.Run方法，然后将桌面数据重新添加到第一层桌面上显示。

```java

keep_running: {

    loadAndBindWorkspace();

    if (mStopped) {
        break keep_running;
    }

    waitForIdle();

    loadAndBindAllApps();

    //加载第一层桌面数据
    loadVerifyApplications();

}

private void loadVerifyApplications() {
    final Context context = mApp.getContext();

    ArrayList<ItemInfo> tmpInfos;
    ArrayList<ItemInfo> added = new ArrayList<ItemInfo>();
    synchronized (sBgLock) {
        for (AppInfo app : mBgAllAppsList.data) {
            tmpInfos = getItemInfoForComponentName(app.componentName, app.user);
            if (tmpInfos.isEmpty()) {
                added.add(app);
            }
        }
    }
    if (!added.isEmpty()) {
        addAndBindAddedWorkspaceItems(context, added);
    }
}

```

### 检测替换APP时并更新Workspace

修改PackageUpdatedTask类中execute方法，添加第一层横向桌面的数据更新逻辑。

``` java

public void execute(LauncherAppState app, BgDataModel dataModel, AllAppsList appsList) {

    ....

     final ArrayList<AppInfo> addedOrModified = new ArrayList<>();
    addedOrModified.addAll(appsList.added);
        
    //更新workspace
    if (LauncherAppState.isDisableAllApps()) {
        updateToWorkSpace(context, app, appsList);
    }
    
    ...
}

public void updateToWorkSpace(Context context, LauncherAppState app , AllAppsList appsList){
         ArrayList<Pair<ItemInfo, Object>> installQueue = new ArrayList<>();

    final List<UserHandle> profiles = UserManagerCompat.getInstance(context).getUserProfiles();
    
    ArrayList<InstallShortcutReceiver.PendingInstallShortcutInfo> added 
    = new ArrayList<InstallShortcutReceiver.PendingInstallShortcutInfo>();
    
    for (UserHandle user : profiles) {
        final List<LauncherActivityInfo> apps = LauncherAppsCompat.getInstance(context).getActivityList(null, user);
        synchronized (this) {
            for (LauncherActivityInfo info : apps) {
                for (AppInfo appInfo : appsList.added) {
                    if(info.getComponentName().equals(appInfo.componentName)){
                        InstallShortcutReceiver.PendingInstallShortcutInfo mPendingInstallShortcutInfo 
                        =  new InstallShortcutReceiver.PendingInstallShortcutInfo(info,context);
                        added.add(mPendingInstallShortcutInfo);
                        installQueue.add(mPendingInstallShortcutInfo.getItemInfo());
                    }
                }
            }
        }
    }

    if (!added.isEmpty()) {
        app.getModel().addAndBindAddedWorkspaceItems(installQueue);
    }

}

```

### 改造Workspace的长按呼出菜单

屏蔽Workspace长按删除操作，该删除会导致第一层桌面数据丢失。

在DeleteDropTarget类supportsDrop方法中，屏蔽菜单键的显示。

```java

public static boolean supportsDrop(Object info) {

    //    TODO 屏蔽长按删除
    //    return (info instanceof ShortcutInfo)
    //            || (info instanceof LauncherAppWidgetInfo)
    //            || (info instanceof FolderInfo);

    if (info instanceof ShortcutInfo) {
        ShortcutInfo item = (ShortcutInfo) info;
        return item.itemType != LauncherSettings.BaseLauncherColumns.ITEM_TYPE_APPLICATION;
    }
    return info instanceof LauncherAppWidgetInfo;

}

```

## 结语

Launcher横向化到此也改造完毕。
本系列文章相关代码，可以通过点击[这里](https://github.com/toeii/Launcher3)找到。希望对大家学习和了解Launcher开发有所帮助。



