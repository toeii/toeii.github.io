---
layout:     post
title:      RecyclerView中需要优化的地方
subtitle:    ""
date:       2018-11-15
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Android
---

## 前言
毫不夸张的说，RecyclerView占应用UI的60%，所以RecyclerView的性能优化其实还是挺重要的。
以下是我对RecyclerView性能优化的一点知识总结，在这里记录和分享一下。

## Diffutil
DIffUtils是Support-v7:24:2.0中，更新的工具类，它主要是为了配合 RecyclerView 使用，通过比对新、旧两个数据集的差异，生成旧数据到新数据的最小变动，然后对有变动的数据项，进行局部刷新。

DIffUtils必须有两个数据集（这是它的弊端），用法如下：

首先实现一个DiffCallBack

```java
private class DiffCallBack extends DiffUtil.Callback {

    @Override
    public int getOldListSize() {
        return data.size();
    }

    @Override
    public int getNewListSize() {
        return newData.size();
    }

    @Override
    public boolean areItemsTheSame(int oldItemPosition, int newItemPosition) {
        return data.get(oldItemPosition).getType() == newData.get(newItemPosition).getType();
    }

    @Override
    public boolean areContentsTheSame(int oldItemPosition, int newItemPosition) {
        //这里需要对比数据，如果业务复杂，则不推荐使用DIffUtils
        String oldStr = (String) DiffUtilDemoActivity.this.data.get(oldItemPosition).getData();
        String newStr = (String) DiffUtilDemoActivity.this.newData.get(newItemPosition).getData();
        return oldStr.equals(newStr);
    }
}
```

接着调用

```java
    DiffUtil.DiffResult diffResult = DiffUtil.calculateDiff(new DiffCallBack(oldDatas, newDatas), true);
    diffResult.dispatchUpdatesTo(mAdapter);
```

dispatchUpdatesTo()就是将这个数据集差异的结果，通过Adapter更新到RecyclerView上面。

## Prefetch
Prefetch即预取功能，这个功能在Support-v7:25+是默认开启的，我们可以不用关心它如何调用，但是原理还是值得探究一番的，如图所示：

![](/img/toeii/recyclerview_prefetch.png)

虽说预取是默认开启不需要我们开发者操心的事情，但是明白原理还是能加深该功能的理解。下面就说下自己在看预取源码时的一点理解。实现预取功能的一个关键类就是gapworker，可以直接在recyclerView源码中找到该类

```java
GapWorker mGapWorker;
```

通过在ontouchevent中触发预取的判断逻辑，在手指执行move操作的代码末尾有这么段代码

```java
case MotionEvent.ACTION_MOVE: {
    ......
        if (mGapWorker != null && (dx != 0 || dy != 0)) {
            mGapWorker.postFromTraversal(this, dx, dy);
        }
    }
} break;
```

通过每次move操作来判断是否预取下一个可能要显示的item数据，判断的依据就是通过传入的dx和dy得到手指接下来可能要移动的方向，如果dx或者dy的偏移量会导致下一个item要被显示出来则预取出来，但是并不是说预取下一个可能要显示的item一定都是成功的，其实每次recyclerView取出要显示的一个item本质上就是取出一个viewholder，根据viewholder上关联的itemview来展示这个item。而取出viewholder最核心的方法就是

```java
    tryGetViewHolderForPositionByDeadline(int position,boolean dryRun, long deadlineNs)
```

看方法的参数也能找到和预取有关的信息,deadlineNs的一般取值有两种，一种是为了兼容版本25之前没有预取机制的情况，兼容25之前的参数为

```java
static final long FOREVER_NS = Long.MAX_VALUE;
```

另一种就是实际的deadline数值，超过这个deadline则表示预取失败，这个其实也好理解，预取机制的主要目的就是提高recyclerView整体滑动的流畅性，如果要预取的viewholder会造成下一帧显示卡顿强行预取的话那就有点本末倒置了。
关于预取成功的条件通过调用

```java
boolean willCreateInTime(int viewType, long approxCurrentNs, long deadlineNs) {
            long expectedDurationNs = getScrapDataForType(viewType).mCreateRunningAverageNs;
            return expectedDurationNs == 0 || (approxCurrentNs + expectedDurationNs < deadlineNs);
}
```

来进行判断，approxCurrentNs的值为

```java
long start = getNanoTime();
if (deadlineNs != FOREVER_NS && !mRecyclerPool.willCreateInTime(type, start, deadlineNs)) {
        return null;
}
```

而mCreateRunningAverageNs就是创建同type的holder的平均时间，感兴趣的可以去看下这个值如何得到，不难理解就不贴代码了。关于预取就说到这里，想知道更多可以查看该[文章](https://juejin.im/entry/58a3f4f62f301e0069908d8f)

## setHasFixedSize
关于setHasFixedSize没太多好说的，记住一点就行：如果Item高度是固定的话，可以使用 RecyclerView.setHasFixedSize(true); 来避免requestLayout浪费资源。

如果想知道更多可以查看该[文章](https://www.jianshu.com/p/79c9c70f6502) 

## 四级缓存
RecycleView的四级缓存是由三个类共同作用完成的，Recycler、recyclerViewPool和ViewCacheExtension。

Recycler用于管理已经废弃或者与RecyclerView分离的ViewHolder，这里面有两个重要的成员分别是屏幕内缓存和屏幕外缓存。
屏幕内缓存指在屏幕中显示的ViewHolder，这些ViewHolder会缓存在mAttachedScrap、mChangedScrap中。
mChangedScrap表示数据已经改变的viewHolder列表，mAttachedScrap未与RecyclerView分离的ViewHolder列表。
屏幕外缓存指当列表滑动出了屏幕时，ViewHolder会被缓存在 mCachedViews ，其大小由mViewCacheMax决定，默认DEFAULT_CACHE_SIZE为2，可通过Recyclerview.setItemViewCacheSize()动态设置。

RecyclerViewPool是用来缓存ViewHolder，如果多个RecyclerView之间用setRecyclerViewPool(RecyclerViewPool)设置同一个RecyclerViewPool，他们就可以共享ViewHolder。

ViewCacheExtension是开发者可自定义的一层缓存，是虚拟类ViewCacheExtension的一个实例，开发者可实现方法getViewForPositionAndType(Recycler recycler, int position, int type)来实现自己的缓存。

一张图理解

![](/img/toeii/recyclerview_for_cache.png)

## RecyclerViewPool的复用
如果多个 RecyclerView 的 Adapter 是一样的，比如嵌套的 RecyclerView 中存在一样的 Adapter，可以通过设置 RecyclerView.setRecyclerViewPool(pool); 来共用一个 RecyclerViewPool。

## 布局优化
1，减少过渡绘制
    减少布局层级，可以考虑使用自定义 View 来减少层级，或者更合理地设置布局来减少层级，不推荐在 RecyclerView 中使用 ConstraintLayout，有很多开发者已经反映了使用它效果更差。推荐使用RelativeLayout进行单层排列。

2，减少xml文件inflate时间
    这里的xml文件不仅包括layout的xml，还包括drawable的xml，xml文件inflate出ItemView是通过耗时的IO操作，尤其当Item的复用几率很低的情况下，随着Type的增多，这种Inflate带来的损耗是相当大的，此时我们可以用代码去生成布局，即 new View()的方式，只要搞清楚xml中每个节点的属性对应的API即可。

3，减少 View 对象的创建
    一个稍微复杂的Item会包含大量的View，而大量的View的创建也会消耗大量时间，所以要尽可能简化ItemView；设计ItemType时，对多ViewType能够共用的部分尽量设计成自定义View，减少View的构造和嵌套。

## 其他的细节优化
1，itemanimator不必要的时候可以取消，调用((SimpleItemAnimator) rv.getItemAnimator()).setSupportsChangeAnimations(false)。

2，getAdapterPosition和getLayoutPosition，最好使用getAdapterPosition。

3，removeview和detachview其实差不多，因为removeview的内部会调用detachview，考虑业务理解层面removeview更适合。

4，通过RecycleView.setItemViewCacheSize(size)保存嵌套RecyclerView的滑动状态。

5，去除冗余的setItemclick事件，建议公用一个Listener，根据ID来进行不同的操作，优化了对象的频繁创建带来的资源消耗。

6，通过 getExtraLayoutSpace 来增加 RecyclerView 预留的额外空间（显示范围之外，应该额外缓存的空间）。

```java
new LinearLayoutManager(this) {
    @Override
    protected int getExtraLayoutSpace(RecyclerView.State state) {
        return size;
    }
};
```

7，通过重写 RecyclerView.onViewRecycled(holder) 来回收资源。

8，设置 RecyclerView.addOnScrollListener(listener); 来对滑动过程中停止加载的操作。

9，对TextView使用String.toUpperCase来替代android:textAllCaps="true"。

10，对TextView使用StaticLayout或者DynamicLayout的自定义View来代替它。

## 总结
RecyclerView的优化点很多，需要日常开发中酌情处理。

## 感谢
本文部分内容参考自[RecyclerView性能优化](https://www.jianshu.com/p/aedb2842de30)












