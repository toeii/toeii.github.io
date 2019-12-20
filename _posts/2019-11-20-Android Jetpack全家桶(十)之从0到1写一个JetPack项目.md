---
layout:     post
title:      Android Jetpack全家桶(十)之从0到1写一个JetPack项目
subtitle:    ""
date:       2019-11-20
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Android
---


## 前言

Android JetPack是Google在18年IO大会上推荐的一整套组件库，它的出现填补了之前Android中自带的一些缺陷，例如Handler的内存泄露、Camera的不易用性、后台调度难以管理等等。所以我打算把整个架构组件系统性的学习一下，在这里和大家分享，希望能帮助到其他学习者。本系列文章包含十篇：

- [Android Jetpack全家桶(一)之Jetpack介绍](https://toeii.github.io/2019/07/09/Android-Jetpack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B8%80)%E4%B9%8BJetPack%E9%85%8D%E7%BD%AE/)<br />
- [Android Jetpack全家桶(二)之Lifecycle生命周期感知](https://toeii.github.io/2019/07/09/Android-Jetpack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%BA%8C)%E4%B9%8BLifecycle%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E6%84%9F%E7%9F%A5/)<br />
- [Android Jetpack全家桶(三)之ViewModel控制器](https://toeii.github.io/2019/07/10/Android-Jetpack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B8%89)%E4%B9%8BViewModel%E6%8E%A7%E5%88%B6%E5%99%A8/)<br />
- [Android Jetpack全家桶(四)之LiveData数据维持](https://toeii.github.io/2019/07/12/Android-Jetpack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%9B%9B)%E4%B9%8BLiveData%E6%95%B0%E6%8D%AE%E7%BB%B4%E6%8C%81/)<br />
- [Android Jetpack全家桶(五)之Room ORM库](https://toeii.github.io/2019/07/17/Android-Jetpack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%BA%94)%E4%B9%8BRoom-ORM%E5%BA%93/)<br />
- [Android Jetpack全家桶(六)之Paging分页库](https://toeii.github.io/2019/07/19/Android-Jetpack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%85%AD)%E4%B9%8BPaging%E5%88%86%E9%A1%B5%E5%BA%93/)<br />
- [Android Jetpack全家桶(七)之WorkManager工作管理](https://toeii.github.io/2019/08/01/Android-Jetpack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B8%83)%E4%B9%8BWorkManager%E5%B7%A5%E4%BD%9C%E7%AE%A1%E7%90%86/)<br />
- [Android Jetpack全家桶(八)之Navigation导航](https://toeii.github.io/2019/08/06/Android-Jetpack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%85%AB)%E4%B9%8BNavigation%E5%AF%BC%E8%88%AA/)<br />
- [Android Jetpack全家桶(九)之DataBinding数据绑定](https://toeii.github.io/2019/08/07/Android-Jetpack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E4%B9%9D)%E4%B9%8BDataBinding%E6%95%B0%E6%8D%AE%E7%BB%91%E5%AE%9A/)<br />
- [Android Jetpack全家桶(十)之从0到1写一个JetPack项目](https://toeii.github.io/2019/08/07/Android-Jetpack%E5%85%A8%E5%AE%B6%E6%A1%B6(%E5%8D%81)%E4%B9%8B%E4%BB%8E0%E5%88%B01%E5%86%99%E4%B8%80%E4%B8%AAJetPack%E5%B0%8F%E9%A1%B9%E7%9B%AE/)<br />


## 前言

因为拓意阅读之前有写过flutter版本，所以这次打算结合原有的api和交互效果写一款Jetpack版本，旨在结合博客内容加深学习印象，和大家一起真正的使用Jetpack组件。

该应用数据主要来源于开眼api。采用了QMUI + Jetpack MVVM的架构。使用QMUI也是一种尝试，不过在实际用过之后感觉该UI库不符合预期，并不推荐大家在商业化项目中使用。

## 项目介绍

### 项目架构

架构上采用了官方的做法，以MVVM架构形式打通整套Jetpack体系流程。详见下图：

![架构](/img/toeii/jetpack_extension_read_egg.png)

### 项目结构

结构上简单来说就是功能模块与UI模块划分后，再次在UI模块内做业务性的功能划分，这样的划分形式很清晰展现出了各个模块职能。详见下图：

![结构1](/img/toeii/icon_jetpack_project_shot.png)
![结构2](/img/toeii/icon_jetpack_project_shot2.png)

### 项目展示

以下是拓意阅读Jetpack版项目截图。详见下图：

![展示](/img/toeii/icon_jetpackapp_screenshots.png)

### 项目下载

以下是拓意阅读Jetpack版下载地址。详见下图：

![下载](/img/toeii/jetpckapk_download_code.png)

## 项目说明（以日报模块为例）

主体思路：

    1，视图（生成ViewDataBinding，关联ViewModel）

    2，数据来源（配置Paging组件，利用DataSource控制数据）

    3，交互（PagedList建立关联，填充Adapter数据）

具体实现：

fragment_daily.xml

```XML

<layout>

    <data></data>

    <com.qmuiteam.qmui.widget.QMUIWindowInsetLayout xmlns:android="http://schemas.android.com/apk/res/android"
                                                    android:layout_width="match_parent"
                                                    android:layout_height="match_parent">

        <com.qmuiteam.qmui.widget.QMUIEmptyView
                android:id="@+id/empty_view"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:background="@color/qmui_config_color_white"
                android:fitsSystemWindows="true"/>

        <FrameLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:fitsSystemWindows="true">
            <com.qmuiteam.qmui.widget.pullRefreshLayout.QMUIPullRefreshLayout
                    android:id="@+id/pull_to_refresh"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent">
                <androidx.recyclerview.widget.RecyclerView
                        android:id="@+id/rv_coordinator"
                        android:layout_width="match_parent"
                        android:layout_height="match_parent">
                </androidx.recyclerview.widget.RecyclerView>
            </com.qmuiteam.qmui.widget.pullRefreshLayout.QMUIPullRefreshLayout>
        </FrameLayout>

    </com.qmuiteam.qmui.widget.QMUIWindowInsetLayout>

</layout>

```

DailyFragment.kt

```java

class DailyFragment : BaseFragment<FragmentDailyBinding>(){

    private val mDailyAdapter: DailyAdapter by lazy { DailyAdapter() }

    private val mViewModel: DailyViewModel by lazy(LazyThreadSafetyMode.NONE)  {
        ViewModelProviders.of(this,DailyModelFactory(DailyRepository()))[DailyViewModel::class.java]
    }

    override fun getLayoutId(): Int = R.layout.fragment_daily

    override fun initView(view : View) {

        mBinding.emptyView.show(true)

        mBinding.rvCoordinator.layoutManager = LinearLayoutManager(context)
        mBinding.rvCoordinator.isNestedScrollingEnabled = true
        (mBinding.rvCoordinator.itemAnimator as SimpleItemAnimator).supportsChangeAnimations = false
        mBinding.rvCoordinator.adapter = mDailyAdapter
        mBinding.rvCoordinator.addItemDecoration(SpacesItemDecoration(10))

    }

    override fun initData() {
        mViewModel.fetchResult()
        mViewModel.result?.observe(this, Observer<PagedList<HomeDailyItemListBean>>{
            mDailyAdapter.submitList(it)
        })
    }

    override fun initListener() {
        mDailyAdapter.setOnItemListener(OnItemClickListener { position, view ->
            val data = mDailyAdapter.currentList!![(position-1)]
            if (data != null) {
                openWebView(data.data.content.data.title,data.data.content.data.webUrl.raw)
                val browseRecordBean = BrowseRecordBean(
                    data.id.toString(),
                    data.data.content.data.title, data.data.content.data.description,
                    data.data.content.data.webUrl.raw, data.data.content.data.cover.feed
                )
                doAsync {
                    ERApplication.db.browseRecordDao().insert(browseRecordBean)
                }
            }
        })

        mBinding.pullToRefresh.setOnPullListener(object : QMUIPullRefreshLayout.OnPullListener {
            override fun onMoveTarget(offset: Int) {}

            override fun onMoveRefreshView(offset: Int) {}

            override fun onRefresh() {
                initData()
                mBinding.pullToRefresh.postDelayed( { mBinding.pullToRefresh.finishRefresh() }, 500)
            }
        })

        CoroutineBus.register(this.javaClass.simpleName, UI, EventMessage::class.java) {
            when {
                it.tag == DailyFragment::class.java.name + ERAppConfig.PAGE_DATA_INIT ->
                    mBinding.emptyView.show(false)
                it.tag == DailyFragment::class.java.name + ERAppConfig.PAGE_DATA_LOAD_START ->
                    mDailyAdapter.isLoadMore = 1
                it.tag == DailyFragment::class.java.name + ERAppConfig.PAGE_DATA_LOAD_END ->
                    mDailyAdapter.isLoadMore = -1
            }
            mDailyAdapter.notifyDataSetChanged()
        }

    }

    override fun onDestroy() {
        super.onDestroy()
        CoroutineBus.unregister(this.javaClass.simpleName)
    }

}

```

DailyPaging.kt

```java

class DailyRepository{

    suspend fun getHomeDailyList(page: Int): List<HomeDailyItemListBean>? = withContext(Dispatchers.IO){
        val result = RetrofitManager.apiService.getHomeDailyList(System.currentTimeMillis().toString()).itemList
        val filterData = result.filter {
            it.type == "followCard"
        }
        filterData
    }

}

class DailyDataSource(private val repository: DailyRepository) : PageKeyedDataSource<Int, HomeDailyItemListBean>(), CoroutineScope by MainScope(){


    override fun loadInitial(params: LoadInitialParams<Int>, callback: LoadInitialCallback<Int, HomeDailyItemListBean>) {
        safeLaunch{
            val result = repository.getHomeDailyList(1)
            result?.let {
                callback.onResult(it,1,2)
                CoroutineBus.post(
                    EventMessage(
                        DailyFragment::class.java.name
                                + ERAppConfig.PAGE_DATA_INIT,
                        null
                    )
                )
            }
        }
    }

    override fun loadAfter(params: LoadParams<Int>, callback: LoadCallback<Int, HomeDailyItemListBean>) {
        CoroutineBus.post(
            EventMessage(
                DailyFragment::class.java.name
                        + ERAppConfig.PAGE_DATA_LOAD_END,
                null
            )
        )
    }

    override fun loadBefore(params: LoadParams<Int>, callback: LoadCallback<Int, HomeDailyItemListBean>) {

    }

    override fun invalidate() {
        super.invalidate()
        cancel()
    }

}

class DailyDataSourceFactory(private val repository: DailyRepository) : DataSource.Factory<Int, HomeDailyItemListBean>() {
    override fun create(): DataSource<Int, HomeDailyItemListBean> = DailyDataSource(repository)
}


```

DailyViewModel.kt

```java

class DailyViewModel(private val repository: DailyRepository) : ViewModel() {

    var result: LiveData<PagedList<HomeDailyItemListBean>>? = null

    fun fetchResult() {
        result = LivePagedListBuilder(
            DailyDataSourceFactory(repository),
            PagedList.Config.Builder()
                .setPageSize(ERAppConfig.PAGE_SIZE)
                .setEnablePlaceholders(ERAppConfig.ENABLE_PLACEHOLDERS)
                .setInitialLoadSizeHint(ERAppConfig.PAGE_SIZE_HINT)
                .setPrefetchDistance((ERAppConfig.PAGE_SIZE/2))
                .build()
        ).build()
    }

}


class DailyModelFactory(private val repository: DailyRepository) : ViewModelProvider.NewInstanceFactory() {

    @Suppress("UNCHECKED_CAST")
    override fun <T : ViewModel?> create(modelClass: Class<T>): T = DailyViewModel(repository) as T

}

```

DailyAdapter.kt

```java

class DailyAdapter : PagedListAdapter<HomeDailyItemListBean, RecyclerView.ViewHolder>(diffCallback) {

    internal var isLoadMore = 0
    private lateinit var itemListener: OnItemClickListener

    fun setOnItemListener(listener: OnItemClickListener) {
        this.itemListener = listener
    }

    override fun getItemViewType(position: Int): Int =
        when (position) {
            0 -> ITEM_TYPE_HEADER
            itemCount - 1 -> ITEM_TYPE_FOOTER
            else -> super.getItemViewType(position)
        }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder =
        when (viewType) {
            ITEM_TYPE_HEADER -> HeaderViewHolder(parent)
            ITEM_TYPE_FOOTER -> FooterViewHolder(parent)
            else -> DailyViewHolder(parent)
        }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        when (holder) {
            is HeaderViewHolder -> if(!currentList.isNullOrEmpty()) holder.bindsHeader(currentList!![0])
            is FooterViewHolder -> holder.bindsFooter(isLoadMore)
            is DailyViewHolder -> getDataItem(position)?.let { holder.bindTo(it,itemListener) }
        }
    }

    private fun getDataItem(position: Int): HomeDailyItemListBean? =
        getItem(position-1)

    override fun getItemCount(): Int =
        super.getItemCount() + 2

    override fun registerAdapterDataObserver(observer: RecyclerView.AdapterDataObserver) {
        super.registerAdapterDataObserver(BasePagedListAdapterObserver(observer, 1))
    }

    companion object {
        private val diffCallback = object : DiffUtil.ItemCallback<HomeDailyItemListBean>() {
            override fun areItemsTheSame(oldItem: HomeDailyItemListBean, newItem: HomeDailyItemListBean): Boolean =
                oldItem.id == newItem.id
            override fun areContentsTheSame(oldItem: HomeDailyItemListBean, newItem: HomeDailyItemListBean): Boolean =
                oldItem == newItem
        }

        private const val ITEM_TYPE_HEADER = 99
        private const val ITEM_TYPE_FOOTER = 100

    }
}

class DailyViewHolder(parent: ViewGroup) : RecyclerView.ViewHolder(
    LayoutInflater.from(parent.context).inflate(R.layout.view_list_item_daily, parent, false)) {

    private lateinit var mBinding : ViewListItemDailyBinding

    fun bindTo(data: HomeDailyItemListBean,listener: OnItemClickListener) {

        mBinding = initViewBindingImpl(itemView) as ViewListItemDailyBinding

        if(data.type == "textCard"){
            mBinding.rlDailyLayout.layoutParams.height = 0
        }else{
            mBinding.rlDailyLayout.layoutParams.height = ViewGroup.LayoutParams.WRAP_CONTENT
            data.data.header.iconType = mBinding.root.resources.getString(R.string.str_release)+"："
            data.data.header.issuerName = data.data.content.data.author.name
            data.data.header.icon = data.data.content.data.author.icon
            data.data.header.iconType = data.data.content.data.author.description
        }
        mBinding.item = data

        mBinding.rlDailyLayout.setOnClickListener {
            listener.onItemClick(layoutPosition,it)
        }

    }

}

internal class HeaderViewHolder(parent: ViewGroup) : RecyclerView.ViewHolder(
    LayoutInflater.from(parent.context).inflate(R.layout.view_list_item_header, parent, false)) {

    private lateinit var mHeaderBinding : ViewListItemHeaderBinding

    fun bindsHeader(item: HomeDailyItemListBean?) {
        mHeaderBinding = initViewBindingImpl(itemView) as ViewListItemHeaderBinding
        mHeaderBinding.headerText.visibility = View.GONE
    }

}

internal class FooterViewHolder(parent: ViewGroup) : RecyclerView.ViewHolder(
    LayoutInflater.from(parent.context).inflate(R.layout.view_list_item_footer, parent, false)) {

    private lateinit var mFooterBinding : ViewListItemFooterBinding

    fun bindsFooter(isLoadMore: Int) {
        mFooterBinding = initViewBindingImpl(itemView) as ViewListItemFooterBinding
        when(isLoadMore){
            0 ->  mFooterBinding.tvBanner.text = ""
            1 ->  mFooterBinding.tvBanner.text = mFooterBinding.root.resources.getString(R.string.str_loading)
            -1 ->  mFooterBinding.tvBanner.text = mFooterBinding.root.resources.getString(R.string.str_not_more)
        }
    }

}

private fun initViewBindingImpl(itemView: View): ViewDataBinding? =
    if (null == DataBindingUtil.getBinding(itemView)) {
        DataBindingUtil.bind(itemView)
    } else {
        DataBindingUtil.getBinding(itemView)
    }

```

## 总结

以上是项目模块的实现分析，如果理解不透可以参考学习一下整体项目。[项目地址点我。](https://github.com/toeii/JetPackExampleApp_ExtensionRead)

项目的开发主要为了加深学习印象和与实战相结合。希望对大家学习和了解Jetpack组件有所帮助。








