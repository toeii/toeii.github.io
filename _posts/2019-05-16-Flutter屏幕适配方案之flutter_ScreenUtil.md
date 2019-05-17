---
layout:     post
title:      Flutter屏幕适配方案之flutter_ScreenUtil
subtitle:    ""
date:       2019-05-16
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Flutter
---

## 前言
利用[flutter_screenutil](https://github.com/OpenFlutter/flutter_screenutil)是目前Flutter屏幕适配用得最多得一种，其实原理也很简单。

比如我们设计师设计的UI是根据iPhone6来做的，我们知道 iPhone6的分辨率是750 * 1334(px)，
又或者是根据hdpi的设备来设计的UI，我们知道hdpi的 Android设备是 (240 dpi),像素密度是1.5，即hdpi设备的分辨率宽度是320px，总之，无论设计稿的单位是px，或者是dp，我们都能够转换成px。
那么我们如果根据px来适配，ios和 android 就都可以兼容了。
假设，我们的设计稿手机是1080 * 1920 px，设计稿上有一个540 * 960 的组件，即宽度和宽度是手机的一半。如果我们直接写的时候组件的尺寸这么定义，在其他尺寸的设备上未必是一半，或多，或少。但是我们可以按比例来看，即我们要实现的宽度是实际设备的一半。
那么假设我们设备的宽度是deviceWidth和deviceHeight，我们要写的组件大小为: 宽:(540/1080) * deviceWidth，高度: (960/1920) * deviceHeight。
通过这个公式我们可以发现，我们要写的组件宽度就是设计稿上的尺寸width * (deviceWdith/原型设备宽度)。那么每次我们写ui的时候，只要直接拿设计稿的尺寸 * (deviceWdith/设备原型)宽度即可。


## 使用方法:

### 安装依赖：

安装之前请查看最新版本
```
dependencies:
  flutter:
    sdk: flutter
  # 添加依赖
  flutter_screenutil: ^0.5.2
```

### 在每个使用的地方导入包：
```
import 'package:flutter_screenutil/flutter_screenutil.dart';
```

### 属性

|属性|类型|默认值|描述|
|:---|:---|:---|:---| 
|width|double|1080px|设计稿中设备的宽度,单位px|
|height|double|1920px|设计稿中设备的高度,单位px|
|allowFontScaling|bool|false|设置字体大小是否根据系统的“字体大小”辅助选项来进行缩放|

### 初始化并设置适配尺寸及字体大小是否根据系统的“字体大小”辅助选项来进行缩放
在使用之前请设置好设计稿的宽度和高度，传入设计稿的宽度和高度(单位px)
一定在MaterialApp的home中的页面设置(即入口文件，只需设置一次),以保证在每次使用之前设置好了适配尺寸:

```
//填入设计稿中设备的屏幕尺寸

//默认 width : 1080px , height:1920px , allowFontScaling:false
ScreenUtil.instance = ScreenUtil.getInstance()..init(context);

//假如设计稿是按iPhone6的尺寸设计的(iPhone6 750*1334) 
ScreenUtil.instance = ScreenUtil(width: 750, height: 1334)..init(context);

//设置字体大小根据系统的“字体大小”辅助选项来进行缩放,默认为false
ScreenUtil.instance = ScreenUtil(width: 750, height: 1334, allowFontScaling: true)..init(context);
    
```

### 使用：

#### 适配尺寸： 

传入设计稿的px尺寸：

根据屏幕宽度适配 `width: ScreenUtil.getInstance().setWidth(540)`,

根据屏幕高度适配 `height: ScreenUtil.getInstance().setHeight(200)`,

也可以使用 `ScreenUtil()` 替代 `ScreenUtil.getInstance()`,
例如：`ScreenUtil().setHeight(200)`

**注意**

高度也根据setWidth来做适配可以保证不变形(当你想要一个正方形的时候)

setHeight方法主要是在高度上进行适配, 在你想控制UI上一屏的高度与实际中显示一样时使用.

例如:

```
//长方形:
Container(
           width: ScreenUtil.getInstance().setWidth(375),
           height: ScreenUtil.getInstance().setHeight(200),
            ),
            
//如果你想显示一个正方形:
Container(
           width: ScreenUtil.getInstance().setWidth(300),
           height: ScreenUtil.getInstance().setWidth(300),
            ),
```

#### 适配字体:
传入设计稿的px尺寸：

``` 
//传入字体大小，默认不根据系统的“字体大小”辅助选项来进行缩放(可在初始化ScreenUtil时设置allowFontScaling)
ScreenUtil.getInstance().setSp(28)         
 
//传入字体大小，根据系统的“字体大小”辅助选项来进行缩放(如果某个地方不遵循全局的allowFontScaling设置)     
ScreenUtil(allowFontScaling: true).setSp(28)        
     
//for example:

Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                Text('我的文字大小在设计稿上是25px，不会随着系统的文字缩放比例变化',
                    style: TextStyle(
                        color: Colors.black, fontSize: ScreenUtil.getInstance().setSp(24))),
                Text('我的文字大小在设计稿上是25px，会随着系统的文字缩放比例变化',
                    style: TextStyle(
                        color: Colors.black, fontSize: ScreenUtil(allowFontScaling: true).setSp(24))),
              ],
            )


```

#### 其他相关api：
```
    ScreenUtil.pixelRatio       //设备的像素密度
    ScreenUtil.screenWidth      //设备宽度
    ScreenUtil.screenHeight     //设备高度
    ScreenUtil.bottomBarHeight  //底部安全区距离，适用于全面屏下面有按键的
    ScreenUtil.statusBarHeight  //状态栏高度 刘海屏会更高  单位px
    ScreenUtil.textScaleFactory //系统字体缩放比例
    
    ScreenUtil.getInstance().scaleWidth  // 实际宽度的dp与设计稿px的比例
    ScreenUtil.getInstance().scaleHeight // 实际高度的dp与设计稿px的比例
    
```

```
//导入
import 'package:flutter_screenutil/flutter_screenutil.dart';

...

  @override
  Widget build(BuildContext context) {
    //设置适配尺寸 (填入设计稿中设备的屏幕尺寸) 假如设计稿是按iPhone6的尺寸设计的(iPhone6 750*1334)
    ScreenUtil.instance = ScreenUtil(width: 750, height: 1334)..init(context);
    
    print('设备宽度:${ScreenUtil.screenWidth}'); //Device width
    print('设备高度:${ScreenUtil.screenHeight}'); //Device height
    print('设备的像素密度:${ScreenUtil.pixelRatio}'); //Device pixel density
    print(
        '底部安全区距离:${ScreenUtil.bottomBarHeight}'); //Bottom safe zone distance，suitable for buttons with full screen
    print(
        '状态栏高度:${ScreenUtil.statusBarHeight}px'); //Status bar height , Notch will be higher Unit px

    print('实际宽度的dp与设计稿px的比例:${ScreenUtil.getInstance().scaleWidth}');
    print('实际高度的dp与设计稿px的比例:${ScreenUtil.getInstance().scaleHeight}');

    print(
        '宽度和字体相对于设计稿放大的比例:${ScreenUtil.getInstance().scaleWidth * ScreenUtil.pixelRatio}'); 
    print(
        '高度相对于设计稿放大的比例:${ScreenUtil.getInstance().scaleHeight * ScreenUtil.pixelRatio}'); 
    print('系统的字体缩放比例:${ScreenUtil.textScaleFactory}');

    return new Scaffold(
      appBar: new AppBar(
        title: new Text(widget.title),
      ),
      body: new Center(
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.center,
          children: <Widget>[
            Row(
              children: <Widget>[
                Container(
                  width: ScreenUtil.getInstance().setWidth(375),
                  height: ScreenUtil.getInstance().setHeight(200),
                  color: Colors.red,
                  child: Text(
                    '我的宽度:${ScreenUtil.getInstance().setWidth(375)}dp',
                    style: TextStyle(
                      color: Colors.white,
                      fontSize: ScreenUtil.getInstance().setSp(12),
                    ),
                  ),
                ),
                Container(
                  width: ScreenUtil.getInstance().setWidth(375),
                  height: ScreenUtil.getInstance().setHeight(200),
                  color: Colors.blue,
                  child: Text('我的宽度:${ScreenUtil.getInstance().setWidth(375)}dp',
                      style: TextStyle(
                        color: Colors.white,
                        fontSize: ScreenUtil.getInstance().setSp(12),
                      )),
                ),
              ],
            ),
            Text('设备宽度:${ScreenUtil.screenWidth}px'),
            Text('设备高度:${ScreenUtil.screenHeight}px'),
            Text('设备的像素密度:${ScreenUtil.pixelRatio}'),
            Text('底部安全区距离:${ScreenUtil.bottomBarHeight}px'),
            Text('状态栏高度:${ScreenUtil.statusBarHeight}px'),
            Text(
              '实际高度的dp与设计稿px的比例:${ScreenUtil.getInstance().scaleHeight}',
              textAlign: TextAlign.center,
            ),
            Text(
              '实际高度的dp与设计稿px的比例:${ScreenUtil.getInstance().scaleHeight}',
              textAlign: TextAlign.center,
            ),
            Text(
              '宽度和字体相对于设计稿放大的比例:${ScreenUtil.getInstance().scaleWidth * ScreenUtil.pixelRatio}',
              textAlign: TextAlign.center,
            ),
            Text(
              '高度相对于设计稿放大的比例:${ScreenUtil.getInstance().scaleHeight * ScreenUtil.pixelRatio}',
              textAlign: TextAlign.center,
            ),
            SizedBox(
              height: ScreenUtil.getInstance().setHeight(100),
            ),
            Text('系统的字体缩放比例:${ScreenUtil.textScaleFactory}'),
            Column(
                         crossAxisAlignment: CrossAxisAlignment.start,
                         children: <Widget>[
                           Text('我的文字大小在设计稿上是25px，不会随着系统的文字缩放比例变化',
                               style: TextStyle(
                                   color: Colors.black, fontSize: ScreenUtil.getInstance().setSp(24))),
                           Text('我的文字大小在设计稿上是25px，会随着系统的文字缩放比例变化',
                               style: TextStyle(
                                   color: Colors.black, fontSize: ScreenUtil(allowFontScaling: true).setSp(24))),
                         ],
                       )
          ],
        ),
      ),
    );
  }
```

### 使用示例:

[example demo](https://github.com/OpenFlutter/flutter_ScreenUtil/blob/master/example/lib/main_zh.dart)
 
效果:

![效果](/img/toeii/flutter_screenutil_demo.png)





部分内容摘自：[__white的掘金文章](https://juejin.im/post/5bc5a56a5188255c352d88fe)

