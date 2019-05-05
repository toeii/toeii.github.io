---
layout:     post
title:      Flutter与Android混合开发之路由通信
subtitle:    "跨平台是未来的趋势啊..."
date:       2019-02-02
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Flutter
---

## 前言
当Android项目集成Flutter项目之后，将两个项目链接起来的核心就是路由通信了。所以在这里记录以下混合开发的路由通信方式，加深印象。

## 正文
Flutter提供 MethodChannel、EventChannel、BasicMessageChannel 三种方式。

    1，类似注册监听，发送的模式 原则
    2，使用顺序：先注册，后发送，否则接收不到。尤其使用 MethodChannel、EventChannel 不符合该原则会抛出异常，BasicMessageChannel方式只是收不到消息

#### MethodChannel
    应用场景：Flutter端 调用 Native端

Flutter端代码：

```java
    var result = await MethodChannel("com.simple.channelflutterandroid/method", 
                      StandardMethodCodec())
                      .invokeMethod("toast", {"msg": msg})
```

Android端代码：

```java
    MethodChannel(flutterView, "com.simple.channelflutterandroid/method",
              StandardMethodCodec.INSTANCE)
             .setMethodCallHandler { methodCall, result ->
        when (methodCall.method) {
             "toast" -> {
                //调用传来的参数"msg"对应的值
                val msg = methodCall.argument<String>("msg")
                //调用本地Toast的方法
                Toast.makeText(cxt, msg, Toast.LENGTH_SHORT).show()
                //回调给客户端
                 result.success("native android toast success")
             }
            "other" -> {
               // ...
             }
             else -> {
               // ...
             }
         }
    }
```

#### EventChannel
    应用场景：Native端 调用 Flutter端

Flutter端代码：

```java
    EventChannel("com.simple.channelflutterandroid/event")
            .receiveBroadcastStream()
            .listen((event){
                    print("It is Flutter -  receiveBroadcastStream $event");
    })
```

Android端代码：

```java
    EventChannel(flutterView, "com.simple.channelflutterandroid/event")
         .setStreamHandler(object : EventChannel.StreamHandler {
               override fun onListen(o: Any?, eventSink: EventChannel.EventSink) {
                     eventSink.success("我是发送Native的消息")
               }

               override fun onCancel(o: Any?) {
                       // ...
               }
    })
```

#### BasicMessageChannel
    应用场景：互相调用
    两种使用方式，创建方式和格式不一样

##### 第一种
Flutter端

```java
_basicMessageChannel = BasicMessageChannel("com.simple.channelflutterandroid/basic", 
stringCodec())
// 发送消息
_basicMessageChannel.send("我是Flutter发送的消息");
// 接收消息
_basicMessageChannel.setMessageHandler((str){
    print("It is Flutter -  receive str");
});
```

Android端

```java
val basicMessageChannel = BasicMessageChannel<String>(flutterView, "com.simple.channelflutterandroid/basic", StringCodec.INSTANCE)
// 发送消息
basicMessageChannel.send("我是Native发送的消息")
// 接收消息
basicMessageChannel.setMessageHandler { o, reply ->
    println("It is Native - setMessageHandler $o")
    reply.reply("It is reply from native")
}
```

    其中StringCodec()，四种方式可选

##### 第二种

Flutter端

```java
static const BASIC_BINARY_NAME = "com.simple.channelflutterandroid/basic/binary";
// 发送消息
val res = await BinaryMessages.send(BASIC_BINARY_NAME, ByteData(256))
// 接收消息
BinaryMessages.setMessageHandler(BASIC_BINARY_NAME, (byteData){
    println("It is Flutter - setMessageHandler $byteData")
});
```

Android端

```java
const val BASIC_BINARY_NAME = "com.simple.channelflutterandroid/basic/binary"
// 发送消息
flutterView.send(BASIC_BINARY_NAME,ByteBuffer.allocate(256));
// 接收消息
flutterView.setMessageHandler(BASIC_BINARY_NAME) { byteBuffer, binaryReply ->
    println("It is Native - setMessageHandler $byteBuffer")
    binaryReply.reply(ByteBuffer.allocate(256))
}
```




