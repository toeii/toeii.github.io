---
layout:     post
title:      浅析SnapHelper
subtitle:    "RecyclerView的好帮手"
date:       2018-06-06
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Android
---

## 前言
今天在项目中需要实现一个RecyclerView滑动选择器，但是由于需要考虑到滑动阻尼和双重Item定位等问题，就考虑使用SnapHelper这个辅助类。出于对控件的好奇心，查看了一下源码，在此记录一下。

## 介绍
SnapHelper是一个抽象类，官方提供了一个LinearSnapHelper的子类，可以让RecyclerView滚动停止时相应的Item停留中间位置。25.1.0版本中官方又提供了一个PagerSnapHelper的子类，可以使RecyclerView像ViewPager一样的效果，一次只能滑一页，而且居中显示。
这两个子类使用方式也很简单，只需要创建对象之后调用attachToRecyclerView()附着到对应的RecyclerView对象上就可以了。

```java
new LinearSnapHelper().attachToRecyclerView(mRecyclerView);
//或者
new PagerSnapHelper().attachToRecyclerView(mRecyclerView);
```

## 分析
### 1.Scroll和Fling

SnapHelper总结一下调用栈就是：
```java
SnapHelper
onFling ---> snapFromFling 
```
上面得到最终位置targetPosition,把位置给RecyclerView.SmoothScroller, 然后就开始滑动了：
```java
RecyclerView.SmoothScroller
start --> onAnimation
```
在滑动过程中如果targetPosition对应的targetView已经layout出来了，就会回调SnapHelper，然后计算得到到当前位置到targetView的距离dx,dy
```java
SnapHelper
onTargetFound ---> calculateDistanceToFinalSnap
```
然后把距离dx,dy更新给RecyclerView.Action:
```java
RecyclerView.Action
update --> runIfNecessary --> recyclerView.mViewFlinger.smoothScrollBy
```
最后调用RecyclerView.ViewFlinger, 然后又回到onAnimation
```java
class ViewFlinger implements Runnable{
        public void smoothScrollBy(int dx, int dy, int duration, Interpolator interpolator) {
            if (mInterpolator != interpolator) {
                mInterpolator = interpolator;
                mScroller = new OverScroller(getContext(), interpolator);
            }
            setScrollState(SCROLL_STATE_SETTLING);
            mLastFlingX = mLastFlingY = 0;
            mScroller.startScroll(0, 0, dx, dy, duration);
            postOnAnimation();
        }
}
```
### 2.SnapHelper源码分析
上面其实已经接触到部分的SnapHelper源码, SnapHelper其实是一个抽象类，有三个抽象方法：
```java
    /**
     * Override to provide a particular adapter target position for snapping.
     *
     * @param layoutManager the {@link RecyclerView.LayoutManager} associated with the attached
     *                      {@link RecyclerView}
     * @param velocityX fling velocity on the horizontal axis
     * @param velocityY fling velocity on the vertical axis
     *
     * @return the target adapter position to you want to snap or {@link RecyclerView#NO_POSITION}
     *         if no snapping should happen
     */
    public abstract int findTargetSnapPosition(LayoutManager layoutManager, int velocityX,
            int velocityY);

    /**
     * Override this method to snap to a particular point within the target view or the container
     * view on any axis.
     * <p>
     * This method is called when the {@link SnapHelper} has intercepted a fling and it needs
     * to know the exact distance required to scroll by in order to snap to the target view.
     *
     * @param layoutManager the {@link RecyclerView.LayoutManager} associated with the attached
     *                      {@link RecyclerView}
     * @param targetView the target view that is chosen as the view to snap
     *
     * @return the output coordinates the put the result into. out[0] is the distance
     * on horizontal axis and out[1] is the distance on vertical axis.
     */
    @SuppressWarnings("WeakerAccess")
    @Nullable
    public abstract int[] calculateDistanceToFinalSnap(@NonNull LayoutManager layoutManager,
            @NonNull View targetView);

    /**
     * Override this method to provide a particular target view for snapping.
     * <p>
     * This method is called when the {@link SnapHelper} is ready to start snapping and requires
     * a target view to snap to. It will be explicitly called when the scroll state becomes idle
     * after a scroll. It will also be called when the {@link SnapHelper} is preparing to snap
     * after a fling and requires a reference view from the current set


    
 of child views.
     * <p>
     * If this method returns {@code null}, SnapHelper will not snap to any view.
     *
     * @param layoutManager the {@link RecyclerView.LayoutManager} associated with the attached
     *                      {@link RecyclerView}
     *
     * @return the target view to which to snap on fling or end of scroll
     */
    @SuppressWarnings("WeakerAccess")
    @Nullable
    public abstract View findSnapView(LayoutManager layoutManager);
```
上面三个方法就是我们重写SnapHelper需要实现的，很重要，简单介绍下它们的作用和调用时机：

findTargetSnapPosition用来找到最终的目标位置，在fling操作刚触发的时候会根据速度计算一个最终目标位置，然后开始fling操作 calculateDistanceToFinalSnap 这个用来计算滑动到最终位置还需要滑动的距离，在一开始attachToRecyclerView或者targetView layout的时候会调用 findSnapView用来找到上面的targetView，就是需要对其的view，在calculateDistanceToFinalSnap调用之前会调用该方法。

我们看下SnapHelper怎么用的，其实就一行代码：
```java
this.snapHelper.attachToRecyclerView(view);
```
SnapHelper正是通过该方法附着到RecyclerView上，从而实现辅助RecyclerView滚动对齐操作,那我们就从上面的attachToRecyclerView开始入手：
```java
    public void attachToRecyclerView(@Nullable RecyclerView recyclerView)
            throws IllegalStateException {
        if (mRecyclerView == recyclerView) {
            return; // nothing to do
        }
        if (mRecyclerView != null) {
            destroyCallbacks();
        }
        mRecyclerView = recyclerView;
        if (mRecyclerView != null) {
            setupCallbacks();
            mGravityScroller = new Scroller(mRecyclerView.getContext(),
                    new DecelerateInterpolator());
            snapToTargetExistingView();
        }
    }
```
在attachToRecyclerView()方法中会清掉SnapHelper之前保存的RecyclerView对象的回调(如果有的话)，对新设置进来的RecyclerView对象设置回调,然后初始化一个Scroller对象,最后调用snapToTargetExistingView()方法对SnapView进行对齐调整。

#####snapToTargetExistingView()

该方法的作用是对SnapView进行滚动调整，以使得SnapView达到对齐效果。

看下源码：
```java
    void snapToTargetExistingView() {
        if (mRecyclerView == null) {
            return;
        }
        LayoutManager layoutManager = mRecyclerView.getLayoutManager();
        if (layoutManager == null) {
            return;
        }
        View snapView = findSnapView(layoutManager);
        if (snapView == null) {
            return;
        }
        int[] snapDistance = calculateDistanceToFinalSnap(layoutManager, snapView);
        if (snapDistance[0] != 0 || snapDistance[1] != 0) {
            mRecyclerView.smoothScrollBy(snapDistance[0], snapDistance[1]);
        }
    }
```
snapToTargetExistingView()方法就是先找到SnapView，然后计算SnapView当前坐标到目的坐标之间的距离，然后调用RecyclerView.smoothScrollBy()方法实现对RecyclerView内容的平滑滚动，从而将SnapView移到目标位置，达到对齐效果。

其实这个时候RecyclerView还没进行layout，一般findSnapView会返回null，不需要对齐。

### 3.回调
SnapHelper要有对齐功能，肯定需要知道RecyclerView的滚动scroll和fling过程的，这个就是通过回调接口实现。再看下attachToRecyclerView的源码：
```java
    public void attachToRecyclerView(@Nullable RecyclerView recyclerView)
            throws IllegalStateException {
        if (mRecyclerView == recyclerView) {
            return; // nothing to do
        }
        if (mRecyclerView != null) {
            destroyCallbacks();
        }
        mRecyclerView = recyclerView;
        if (mRecyclerView != null) {
            setupCallbacks();
            mGravityScroller = new Scroller(mRecyclerView.getContext(),
                    new DecelerateInterpolator());
            snapToTargetExistingView();
        }
    }
```
一开始会先清空之前的回调接口然后再注册接口，先看下destroyCallbacks:
```java
    /**
     * Called when the instance of a {@link RecyclerView} is detached.
     */
    private void destroyCallbacks() {
        mRecyclerView.removeOnScrollListener(mScrollListener);
        mRecyclerView.setOnFlingListener(null);
    }
```
可以看出SnapHelper对RecyclerView设置了两个回调，一个是OnScrollListener对象mScrollListener，另外一个就是OnFlingListener对象。

再看下setupCallbacks:
```java
    /**
     * Called when an instance of a {@link RecyclerView} is attached.
     */
    private void setupCallbacks() throws IllegalStateException {
        if (mRecyclerView.getOnFlingListener() != null) {
            throw new IllegalStateException("An instance of OnFlingListener already set.");
        }
        mRecyclerView.addOnScrollListener(mScrollListener);
        mRecyclerView.setOnFlingListener(this);
    }
```
SnapHelper实现了RecyclerView.OnFlingListener接口，所以OnFlingListener就是SnapHelper自身。

先来看下RecyclerView.OnScrollListener对象mScrollListener

RecyclerView.OnScrollListener
先看下mScrollListener是怎么实现的：
```java
    private final RecyclerView.OnScrollListener mScrollListener =
            new RecyclerView.OnScrollListener() {
                boolean mScrolled = false;

                @Override
                public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
                    super.onScrollStateChanged(recyclerView, newState);
                    if (newState == RecyclerView.SCROLL_STATE_IDLE && mScrolled) {
                        mScrolled = false;
                        snapToTargetExistingView();
                    }
                }

                @Override
                public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
                    if (dx != 0 || dy != 0) {
                        mScrolled = true;
                    }
                }
            };
```
mScrolled = true表示之前滚动过，RecyclerView.SCROLL_STATE_IDLE表示滚动停止，这个不清楚的可以看考之前的博客RecyclerView之Scroll和Fling。这个监听器的实现其实很简单，就是在滚动停止的时候调用snapToTargetExistingView对目标View进行滚动调整对齐。

RecyclerView.OnFlingListener
RecyclerView.OnFlingListener接口只有一个方法，这个就是在Fling操作触发的时候会回调，返回true就是已处理，返回false就会交给系统处理。
```java
    /**
     * This class defines the behavior of fling if the developer wishes to handle it.
     * <p>
     * Subclasses of {@link OnFlingListener} can be used to implement custom fling behavior.
     *
     * @see #setOnFlingListener(OnFlingListener)
     */
    public abstract static class OnFlingListener {

        /**
         * Override this to handle a fling given the velocities in both x and y directions.
         * Note that this method will only be called if the associated {@link LayoutManager}
         * supports scrolling and the fling is not handled by nested scrolls first.
         *
         * @param velocityX the fling velocity on the X axis
         * @param velocityY the fling velocity on the Y axis
         *
         * @return true if the fling was handled, false otherwise.
         */
        public abstract boolean onFling(int velocityX, int velocityY);
    }
```
看下SnapHelper怎么实现onFling()方法:
```java
    @Override
    public boolean onFling(int velocityX, int velocityY) {
        LayoutManager layoutManager = mRecyclerView.getLayoutManager();
        if (layoutManager == null) {
            return false;
        }
        RecyclerView.Adapter adapter = mRecyclerView.getAdapter();
        if (adapter == null) {
            return false


    
;
        }
        int minFlingVelocity = mRecyclerView.getMinFlingVelocity();
        return (Math.abs(velocityY) > minFlingVelocity || Math.abs(velocityX) > minFlingVelocity)
                && snapFromFling(layoutManager, velocityX, velocityY);
    }
```
首先会获取mRecyclerView.getMinFlingVelocity()需要进行fling操作的最小速率，只有超过该速率，Item才能在手指离开的时候进行Fling操作。 关键就是调用snapFromFling方法实现平滑滚动。

snapFromFling
看下怎么实现的：
```java
    private boolean snapFromFling(@NonNull LayoutManager layoutManager, int velocityX,
            int velocityY) {
        if (!(layoutManager instanceof ScrollVectorProvider)) {
            return false;
        }

        SmoothScroller smoothScroller = createScroller(layoutManager);
        if (smoothScroller == null) {
            return false;
        }

        int targetPosition = findTargetSnapPosition(layoutManager, velocityX, velocityY);
        if (targetPosition == RecyclerView.NO_POSITION) {
            return false;
        }

        smoothScroller.setTargetPosition(targetPosition);
        layoutManager.startSmoothScroll(smoothScroller);
        return true;
    }
```
首先判断是不是实现了ScrollVectorProvider接口，系统提供的Layoutmanager默认都实现了该接口
创建SmoothScroller对象,默认是LinearSmoothScroller对象，会用LinearInterpolator进行平滑滚动，在目标位置成为Recyclerview的子View时会用DecelerateInterpolator进行减速停止。
通过findTargetSnapPosition()方法，以layoutManager和速率作为参数，找到targetSnapPosition,这个方法就是自定义SnapHelper需要实现的。
把targetSnapPosition设置给平滑滚动器，然后开始进行滚动操作。
很明显重点就是要看下平滑滚动器了。

LinearSmoothScroller
看下系统怎么实现：
```java
    @Nullable
    protected LinearSmoothScroller createSnapScroller(LayoutManager layoutManager) {
        if (!(layoutManager instanceof ScrollVectorProvider)) {
            return null;
        }
        return new LinearSmoothScroller(mRecyclerView.getContext()) {
            @Override
            protected void onTargetFound(View targetView, RecyclerView.State state, Action action) {
                int[] snapDistances = calculateDistanceToFinalSnap(mRecyclerView.getLayoutManager(),
                        targetView);
                final int dx = snapDistances[0];
                final int dy = snapDistances[1];
                final int time = calculateTimeForDeceleration(Math.max(Math.abs(dx), Math.abs(dy)));
                if (time > 0) {
                    action.update(dx, dy, time, mDecelerateInterpolator);
                }
            }

            @Override
            protected float calculateSpeedPerPixel(DisplayMetrics displayMetrics) {
                return MILLISECONDS_PER_INCH / displayMetrics.densityDpi;
            }
        };
    }
```
在通过findTargetSnapPosition()方法找到的targetSnapPosition成为Recyclerview的子View时(根据Recyclerview的缓存机制，这个时候可能该View在屏幕上还看不到),会回调onTargetFound，看下系统定义：
```java
        /**
         * Called when the target position is laid out. This is the last callback SmoothScroller
         * will receive and it should update the provided {@link Action} to define the scroll
         * details towards the target view.
         * @param targetView    The view element which render the target position.
         * @param state         Transient state of RecyclerView
         * @param action        Action instance that you should update to define final scroll action
         *                      towards the targetView
         */
        protected abstract void onTargetFound(View targetView, State state, Action action);
```
传入的第一个参数targetView就是我们希望滚动到的位置对应的View，最后一个参数就是我们可以用来通知滚动器要减速滚动的距离。

其实就是我们要在这个方法里面告诉滚动器在目标子View layout出来后还需要滚动多少距离， 然后通过Action通知滚动器。

第二个方法是计算滚动速率，返回值会影响onTargetFound中的calculateTimeForDeceleration方法，看下源码：
```java
    private final float MILLISECONDS_PER_PX;
    public LinearSmoothScroller(Context context) {
        MILLISECONDS_PER_PX = calculateSpeedPerPixel(context.getResources().getDisplayMetrics());
    }

    /**
     * Calculates the time it should take to scroll the given distance (in pixels)
     *
     * @param dx Distance in pixels that we want to scroll
     * @return Time in milliseconds
     * @see #calculateSpeedPerPixel(android.util.DisplayMetrics)
     */
    protected int calculateTimeForScrolling(int dx) {
        // In a case where dx is very small, rounding may return 0 although dx > 0.
        // To avoid that issue, ceil the result so that if dx > 0, we'll always return positive
        // time.
        return (int) Math.ceil(Math.abs(dx) * MILLISECONDS_PER_PX);
    }

    /**
     * <p>Calculates the time for deceleration so that transition from LinearInterpolator to
     * DecelerateInterpolator looks smooth.</p>
     *
     * @param dx Distance to scroll
     * @return Time for DecelerateInterpolator to smoothly traverse the distance when transitioning
     * from LinearInterpolation
     */
    protected int calculateTimeForDeceleration(int dx) {
        // we want to cover same area with the linear interpolator for the first 10% of the
        // interpolation. After that, deceleration will take control.
        // area under curve (1-(1-x)^2) can be calculated as (1 - x/3) * x * x
        // which gives 0.100028 when x = .3356
        // this is why we divide linear scrolling time with .3356
        return  (int) Math.ceil(calculateTimeForScrolling(dx) / .3356);
    }
```

可以看到，第二个方法返回值越大，需要滚动的时间越长，也就是滚动越慢。

## 总结
到这里，SnapHelper的源码就分析完了，整理下思路，SnapHelper辅助RecyclerView实现滚动对齐就是通过给RecyclerView设置OnScrollerListener和OnFlingListener这两个监听器实现的。 整个过程如下：

在onFling操作触发的时候首先通过findTargetSnapPosition找到最终需要滚动到的位置，然后启动平滑滚动器滚动到指定位置，
在指定位置需要渲染的View -targetView layout出来后，系统会回调onTargetFound,然后调用calculateDistanceToFinalSnap方法计算targetView需要减速滚动的距离，然后通过Action更新给滚动器。
在滚动停止的时候，也就是state变成SCROLL_STATE_IDLE时会调用snapToTargetExistingView，通过findSnapView找到SnapView，然后通过calculateDistanceToFinalSnap计算得到滚动的距离，做最后的对齐调整。


