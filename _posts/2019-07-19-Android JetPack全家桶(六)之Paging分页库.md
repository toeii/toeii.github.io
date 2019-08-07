---
layout:     post
title:      Android JetPack全家桶(六)之Paging分页库
subtitle:    ""
date:       2019-07-19
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

Paging分页库可用于一次性数据和显示小块数据的加载优化，按需加载部分数据减少网络带宽和系统资源的使用。它可以优化Sql数据的加载、也可以优化Network数据的加载。实际应用中，通常是结合RecyclerView使用。

#### 数据分块加载

Paging分页库的关键组件是PagedList类，它是一个可以分块加载的数据集。在需要更多数据时，可以将其分页到现有的PagedList对象中。
如果任何加载的数据发生更改，则会从LiveData或基于RxJava2的对象向可观察数据持有者发出新的PagedList实例。
PagedList实例在被重新生成后，通过Lifecycle，UI也会随着数据的更新而更新。

#### 数据源

DataSource是数据流入PagedList对象的桥。在构建DataSource中，如果数据源是Room库的话，非常简单的利用DataSource.Factory<Int, T>就可以了。而如果不使用Room库而采用其他的第三方ORM库，则需要自定义DataSource，利用PageKeyedDataSource/ItemKeyedDataSource/PositionalDataSource三者之一即可。

在这里，我使用PageKeyedDataSource提供一个BaseDataSource示例：

```java

class BookRepository private constructor(private val bookDao: BookDao) {

    fun getPageBooks(startIndex:Long,endIndex:Long):List<Book> = BookDao.findBooksByIndexRange(startIndex,endIndex)

}

/**
 * 自定义PageKeyedDataSource
 */
class CustomPageDataSource(private val bookRepository: BookRepository) : PageKeyedDataSource<Int, Book>() {

    private val TAG: String by lazy {
        this::class.java.simpleName
    }

    override fun loadInitial(params: LoadInitialParams<Int>, callback: LoadInitialCallback<Int, Book>) {
        val startIndex = 0L
        val endIndex: Long = 0L + params.requestedLoadSize
        val books = bookRepository.getPageBooks(startIndex, endIndex)

        callback.onResult(books, null, 2)
    }

    // 每次分页加载的时候调用
    override fun loadAfter(params: LoadParams<Int>, callback: LoadCallback<Int, Book>) {

        val startPage = params.key
        val startIndex = ((startPage - 1) * BaseConstant.SINGLE_PAGE_SIZE).toLong() + 1
        val endIndex = startIndex + params.requestedLoadSize - 1
        val books = bookRepository.getPageBooks(startIndex, endIndex)

        callback.onResult(books, params.key + 1)
    }

    override fun loadBefore(params: LoadParams<Int>, callback: LoadCallback<Int, Book>) {
       // ... 省略 类似loadAfter
    }
}

```

#### UI

在RecyclerView中设置PagedListAdapter和PagedList类，这些类一起工作以在加载内容时获取和显示内容，预取视图内容并动画内容更改。

## 结合使用

#### 添加依赖

```java

// room
def room_version = '2.2.0-alpha01'
implementation "androidx.room:room-runtime:$room_version"
implementation "androidx.room:room-ktx:$room_version"
kapt "androidx.room:room-compiler:$room_version"
annotationProcessor "android.arch.persistence.room:compiler:$room_version"
kapt "android.arch.persistence.room:compiler:$room_version"

// paging
def paging_version = "2.1.0"
def paging_rxjava_version = "1.0.1"
implementation "android.arch.paging:runtime:$paging_version"
implementation "androidx.paging:paging-runtime-ktx:$paging_version"
implementation "android.arch.paging:rxjava2:$paging_rxjava_version"

// lifecycle
def lifecycle_version = "2.2.0-alpha02"
implementation "androidx.lifecycle:lifecycle-extensions:$lifecycle_version"
annotationProcessor "android.arch.lifecycle:compiler:$lifecycle_version"
kapt "android.arch.lifecycle:compiler:$lifecycle_version"

```

#### 构建数据源

##### 1，创建Database

```java

@Database(entities = [BookBean::class], version = 1,exportSchema = false)
abstract class AppDatabase : RoomDatabase() {
    companion object{
        val CHEESE_DATA = arrayListOf(
            //data ...
        )
        private var instance: AppDatabase? = null

        @Synchronized
        fun get(context: Context):AppDatabase{
            if(null == instance){
                instance = Room.databaseBuilder(context,AppDatabase::class.java,"db_book").build()
            }
            return instance!!
        }
    }

    abstract fun bookDao(): BookDao
    //...Other Dao

}

```

##### 2，创建Entity

```java

@Entity
data class BookBean(
    @PrimaryKey(autoGenerate = true) var uid: Int,
    @ColumnInfo(name = "book_name") var bookName: String?,
    @ColumnInfo(name = "is_like") var isLike: Boolean?
)

```

##### 3，创建Dao

```java

@Dao
interface BookDao {

    @Insert
    fun insertAll(books: List<BookBean>)

    @Delete
    fun delete(book: BookBean)

    @Query("SELECT * FROM bookbean")
    fun getAll(): DataSource.Factory<Int,BookBean>

    @Query("SELECT COUNT(*) FROM bookbean ")
    fun getDataCount(): Int

}

```

#### 4，利用ViewModel构建LiveData<PagedList>

```java

class BookViewModel(app: Application): AndroidViewModel(app) {

    private val dao = AppDatabase.get(app).bookDao()

    companion object {
        private const val PAGE_SIZE = 15
        private const val ENABLE_PLACEHOLDERS = false
    }

    val getAllBook = dao.getAll().toLiveData(Config(
            pageSize = PAGE_SIZE,
            enablePlaceholders = ENABLE_PLACEHOLDERS,
            initialLoadSizeHint = PAGE_SIZE
    ))

    fun insertAll(books: List<String>) = ioThread{
        dao.insertAll(books.map {
            BookBean(uid = 0,bookName = it,isLike = false)
        })
    }

    fun getCount():Int{
        var count = 0
        ioThread{
             count = dao.getDataCount()
        }
        return count
    }

    fun delete(book: BookBean) = ioThread {
         dao.delete(book)
    }

}

```

#### 5，创建PagedListAdapter

```java

class BookAdapter(private var context: Context): PagedListAdapter<BookBean,RecyclerView.ViewHolder>(diffCallback) {

    var viewModel: BookViewModel? = null

    companion object {
        private val diffCallback = object : DiffUtil.ItemCallback<BookBean>() {
            override fun areItemsTheSame(oldItem: BookBean, newItem: BookBean): Boolean =
                oldItem.uid == newItem.uid

            override fun areContentsTheSame(oldItem: BookBean, newItem: BookBean): Boolean =
                oldItem == newItem
        }
    }

    @SuppressLint("InflateParams")
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        val view = LayoutInflater.from(context).inflate(R.layout.view_list_book_item,null)
        return BookHolder(view)
    }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        val book= currentList?.get(position)
        if(null != book){
            holder.itemView.tv_book_title.text = book.bookName
            holder.itemView.iv_delete_icon.onClick {
                viewModel?.delete(book)
            }
        }
    }

    class BookHolder(itemView: View): RecyclerView.ViewHolder(itemView)

}

```

#### 6，结合LifecycleOwner设置UI监听与更新

```java

class BookActivity : AppCompatActivity (){

    lateinit var mBaseModel: BookViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_book)

        mBaseModel = ViewModelProviders.of(this)[BookViewModel::class.java]

        if(mBaseModel.getCount() <=0){
            initData()
        }

        val bookAdapter = BookAdapter(this)
        BookRecyclerView.adapter = bookAdapter
        bookAdapter.viewModel = mBaseModel

        mBaseModel.getAllBook.observe(this, Observer { bookAdapter.submitList(it) }) 
        //PS：这里的this对应的是androidx.appcompat:appcompat:1.1.0-alpha01包下的AppCompatActivity，该AppCompatActivity默认实现了LifecycleOwner。如果是其他包下的AppCompatActivity则需要自己实现LifecycleOwner接口。

    }

    private fun initData() {
        mBaseModel.insertAll(AppDatabase.CHEESE_DATA)
    }

}

```

## 结语

Paging的使用场景有很多，该文章介绍了结合RecyclerView和Room的加载场景，这种场景是比较通用的。如果你的使用场景在其他方面，可以通过查阅官方的[Samples](https://github.com/googlesamples/android-architecture-components/tree/master/PagingSample)进行学习。

最后贴出文章中的[demo](https://github.com/toeii/PagingSimpleExample)地址，以便参考学习。





