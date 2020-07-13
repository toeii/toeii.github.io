---
layout:     post
title:      Flutter状态管理之Flutter_Provide
subtitle:    ""
date:       2019-06-25
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Flutter
---

## Provider介绍

在Flutter开发中，状态管理是一个永恒的话题。一般的原则是：如果状态是组件私有的，则应该由组件自己管理；如果状态要跨组件共享，则该状态应该由各个组件共同的父元素来管理。对于组件私有的状态管理很好理解，但对于跨组件共享的状态，管理的方式就比较多了，如使用全局事件总线EventBus（将在下一章中介绍），它是一个观察者模式的实现，通过它就可以实现跨组件状态同步：状态持有方（发布者）负责更新、发布状态，状态使用方（观察者）监听状态改变事件来执行一些操作。

下面我们看一个登陆状态同步的简单示例：

定义事件：

```java

enum Event{
  login,
  ... //省略其它事件
}

```

登录页代码大致如下：

```java

// 登录状态改变后发布状态改变事件
bus.emit(Event.login);

```

依赖登录状态的页面：

```java

void onLoginChanged(e){
  //登录状态变化处理逻辑
}

@override
void initState() {
  //订阅登录状态改变事件
  bus.on(Event.login,onLogin);
  super.initState();
}

@override
void dispose() {
  //取消订阅
  bus.off(Event.login,onLogin);
  super.dispose();
}

```

我们可以发现，通过观察者模式来实现跨组件状态共享有一些明显的缺点：

    必须显式定义各种事件，不好管理
    
    订阅者必须需显式注册状态改变回调，也必须在组件销毁时手动去解绑回调以避免内存泄露。

在Flutter当中有没有更好的跨组件状态管理方式了呢？答案是肯定的，那怎么做的？我们想想前面介绍的InheritedWidget，它的天生特性就是能绑定InheritedWidget与依赖它的子孙组件的依赖关系，并且当InheritedWidget数据发生变化时，可以自动更新依赖的子孙组件！利用这个特性，我们可以将需要跨组件共享的状态保存在InheritedWidget中，然后在子组件中引用InheritedWidget即可，Flutter社区著名的Provider包正是基于这个思想实现的一套跨组件状态共享解决方案，接下来我们便详细介绍一下Provider的用法及原理


## Provider用法及原理

我们通过上面描述的通过InheritedWidget实现的思路来一步一步地实现一个最基础的Provider。

首先，我们需要一个保存需要共享的数据InheritedWidget，由于具体业务数据类型不可预期，为了通用性，我们使用泛型，定义一个通用的InheritedProvider类，它继承自InheritedWidget：

```java

// 一个通用的InheritedWidget，保存任需要跨组件共享的状态
class InheritedProvider<T> extends InheritedWidget {
  InheritedProvider({@required this.data, Widget child}) : super(child: child);

  //共享状态使用泛型
  final T data;

  @override
  bool updateShouldNotify(InheritedProvider<T> old) {
    //在此简单返回true，则每次更新都会调用依赖其的子孙节点的`didChangeDependencies`。
    return true;
  }
}

```

数据保存的地方有了，那么接下来我们需要做的就是在数据发生变化的时候来重新构建InheritedProvider，那么现在就面临两个问题：

    数据发生变化怎么通知？

    谁来重新构建InheritedProvider？

第一个问题其实很好解决，我们当然可以使用之前介绍的eventBus来进行事件通知，但是为了更贴近Flutter开发，我们使用Flutter SDK中提供的ChangeNotifier类 ，它继承自Listenable，也实现了一个Flutter风格的发布者-订阅者模式，ChangeNotifier定义大致如下：

```java

class ChangeNotifier implements Listenable {
  List listeners=[];
  @override
  void addListener(VoidCallback listener) {
     //添加监听器
     listeners.add(listener);
  }
  @override
  void removeListener(VoidCallback listener) {
    //移除监听器
    listeners.remove(listener);
  }

  void notifyListeners() {
    //通知所有监听器，触发监听器回调 
    listeners.forEach((item)=>item());
  }

  ... //省略无关代码
}

```

我们可以通过调用addListener()和removeListener()来添加、移除监听器（订阅者）；通过调用notifyListeners() 可以触发所有监听器回调。

现在，我们将要共享的状态放到一个Model类中，然后让它继承自ChangeNotifier，这样当共享的状态改变时，我们只需要调用notifyListeners() 来通知订阅者，然后由订阅者来重新构建InheritedProvider，这也是第二个问题的答案！接下来我们便实现这个订阅者类：

```java

class ChangeNotifierProvider<T extends ChangeNotifier> extends StatefulWidget {
  ChangeNotifierProvider({
    Key key,
    this.data,
    this.child,
  });

  final Widget child;
  final T data;

  //定义一个便捷方法，方便子树中的widget获取共享数据
  static T of<T>(BuildContext context) {
    final type = _typeOf<InheritedProvider<T>>();
    final provider =  context.dependOnInheritedWidgetOfExactType<InheritedProvider<T>>();
    return provider.data;
  }

  @override
  _ChangeNotifierProviderState<T> createState() => _ChangeNotifierProviderState<T>();
}

```

该类继承StatefulWidget，然后定义了一个of()静态方法供子类方便获取Widget树中的InheritedProvider中保存的共享状态(model)，下面我们实现该类对应的_ChangeNotifierProviderState类：

```java

class _ChangeNotifierProviderState<T extends ChangeNotifier> extends State<ChangeNotifierProvider<T>> {
  void update() {
    //如果数据发生变化（model类调用了notifyListeners），重新构建InheritedProvider
    setState(() => {});
  }

  @override
  void didUpdateWidget(ChangeNotifierProvider<T> oldWidget) {
    //当Provider更新时，如果新旧数据不"=="，则解绑旧数据监听，同时添加新数据监听
    if (widget.data != oldWidget.data) {
      oldWidget.data.removeListener(update);
      widget.data.addListener(update);
    }
    super.didUpdateWidget(oldWidget);
  }

  @override
  void initState() {
    // 给model添加监听器
    widget.data.addListener(update);
    super.initState();
  }

  @override
  void dispose() {
    // 移除model的监听器
    widget.data.removeListener(update);
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return InheritedProvider<T>(
      data: widget.data,
      child: widget.child,
    );
  }
}

```

可以看到_ChangeNotifierProviderState类的主要作用就是监听到共享状态（model）改变时重新构建Widget树。注意，在_ChangeNotifierProviderState类中调用setState()方法，widget.child始终是同一个，所以执行build时，InheritedProvider的child引用的始终是同一个子widget，所以widget.child并不会重新build，这也就相当于对child进行了缓存！当然如果ChangeNotifierProvider父级Widget重新build时，则其传入的child便有可能会发生变化。

## Provider Demo示例

下面由一个购物车案例带领大家入门。

我们需要实现一个显示购物车中所有商品总价的功能，向购物车中添加新商品时总价更新，定义一个Item类，用于表示商品信息。

```java

class Item {
  Item(this.price, this.count);
  double price; //商品单价
  int count; // 商品份数
  //... 省略其它属性
}

```

定义一个保存购物车内商品数据的CartModel类:

```java

class CartModel extends ChangeNotifier {
  // 用于保存购物车中商品列表
  final List<Item> _items = [];

  // 禁止改变购物车里的商品信息
  UnmodifiableListView<Item> get items => UnmodifiableListView(_items);

  // 购物车中商品的总价
  double get totalPrice =>
      _items.fold(0, (value, item) => value + item.count * item.price);

  // 将 [item] 添加到购物车。这是唯一一种能从外部改变购物车的方法。
  void add(Item item) {
    _items.add(item);
    // 通知监听器（订阅者），重新构建InheritedProvider， 更新状态。
    notifyListeners();
  }
}

```

CartModel即要跨组件共享的model类。最后我们构建示例页面：

```java

class ProviderRoute extends StatefulWidget {
  @override
  _ProviderRouteState createState() => _ProviderRouteState();
}

class _ProviderRouteState extends State<ProviderRoute> {
  @override
  Widget build(BuildContext context) {
    return Center(
      child: ChangeNotifierProvider<CartModel>(
        data: CartModel(),
        child: Builder(builder: (context) {
          return Column(
            children: <Widget>[
              Builder(builder: (context){
                var cart=ChangeNotifierProvider.of<CartModel>(context);
                return Text("总价: ${cart.totalPrice}");
              }),
              Builder(builder: (context){
                print("RaisedButton build"); //在后面优化部分会用到
                return RaisedButton(
                  child: Text("添加商品"),
                  onPressed: () {
                    //给购物车中添加商品，添加后总价会更新
                    ChangeNotifierProvider.of<CartModel>(context).add(Item(20.0, 1));
                  },
                );
              }),
            ],
          );
        }),
      ),
    );
  }
}

```

其实，就这个例子来看，只是更新同一个路由页中的一个状态，我们使用ChangeNotifierProvider的优势并不明显，但是如果我们是做一个购物APP呢？由于购物车数据是通常是会在整个APP中共享的，比如会跨路由共享。如果我们将ChangeNotifierProvider放在整个应用的Widget树的根上，那么整个APP就可以共享购物车的数据了，这时ChangeNotifierProvider的优势将会非常明显。

Model变化后会自动通知ChangeNotifierProvider（订阅者），ChangeNotifierProvider内部会重新构建InheritedWidget，而依赖该InheritedWidget的子孙Widget就会更新。

我们可以发现使用Provider，将会带来如下收益：

    我们的业务代码更关注数据了，只要更新Model，则UI会自动更新，而不用在状态改变后再去手动调用setState()来显式更新页面。

    数据改变的消息传递被屏蔽了，我们无需手动去处理状态改变事件的发布和订阅了，这一切都被封装在Provider中了。这真的很棒，帮我们省掉了大量的工作！

    在大型复杂应用中，尤其是需要全局共享的状态非常多时，使用Provider将会大大简化我们的代码逻辑，降低出错的概率，提高开发效率。

## 写在最后

通过介绍事件总线在跨组件共享中的一些缺点引出了通过InheritedWidget来实现状态的共享的思想，然后基于该思想实现了一个简单的Provider，在实现的过程中也更深入的探索了InheritedWidget与其依赖项的注册机制和更新机制。通过本节的学习，读者应该达到两个目标，首先是对InheritedWidget彻底吃透，其次是Provider的设计思想。

InheritedWidget是Flutter中非常重要的一个Widget，举一反三，像国际化、主题等都可以通过它来实现。