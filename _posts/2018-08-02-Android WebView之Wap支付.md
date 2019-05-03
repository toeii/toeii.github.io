---
layout:     post
title:      Android WebView之Wap支付
subtitle:    "前些时间公司有个聚合支付SDK，顺带分享一下其中的知识点"
date:       2018-08-02
author:     Toeii
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - Android
---

## 前言
做过支付SDK的都知道，费率最划算接入方式最多的就是wap支付了，其实实现起来很简单，只需要在WebView做相关处理就行，这里记录分享一下。
 
## 创建WebView及其简单配置
相信你已经了解过下面的webview创建方式：
```XML
 <LinearLayout
        android:id="@+id/layout_webview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
         />
```
```java
private LinearLayout webViewLayout;
private WebView mWebView;
    @Override
    public void initViews() {
        LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(LinearLayout.LayoutParams.MATCH_PARENT, LinearLayout.LayoutParams.MATCH_PARENT);
        mWebView = new WebView(getActivity());
        mWebView.setLayoutParams(params);
        webViewLayout.addView(mWebView);
    }
```
WebView相关配置：
```java
 private void initWebSettings() {
        webSettings = mWebView.getSettings();
        webSettings.setLoadWithOverviewMode(true);
        webSettings.setJavaScriptEnabled(true);
        //是否阻塞图片下载
        //webSettings.setBlockNetworkImage(false);
        //支持js打开窗口
        webSettings.setJavaScriptCanOpenWindowsAutomatically(true);
        // 设置允许访问文件数据
       //  webSettings.setAllowFileAccess(true);
        //支持dom解析
        webSettings.setDomStorageEnabled(true);
        webSettings.setAllowContentAccess(true);
        //设置字体保持原比例，不受用户设置系统字体大小影响
       // webSettings.setTextZoom(100);
        //支持缩放，默认为true。
        webSettings.setSupportZoom(true);
        //设置内置的缩放控件。若为false，则该WebView不可缩放
        webSettings.setBuiltInZoomControls(true);
        webSettings.setDisplayZoomControls(false);
        webSettings.setLayoutAlgorithm(WebSettings.LayoutAlgorithm.NORMAL);
        //在LOLLIPOP 版本开始需要开启图片混合加载(http 和 https 不开启会导致图片无法加载)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            webSettings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
        }
        //移除常见JS漏洞
        removeJavascriptInterfaces(mWebView);
        mWebView.setWebViewClient(new MyWebViewClient());
        mWebView.loadUrl(url);//url:包含H5支付的网页地址
    }
```
这是WebView一些基础配置，具体配置需要看具体业务需求。具体WebView优化就不过多介绍了。

WebViewClient配置：
```java
 class MyWebViewClient extends WebViewClient {

        @Override
        public boolean shouldOverrideUrlLoading(WebView view, String url) {
            startPay(url);
            return super.shouldOverrideUrlLoading(view, url);
        }

        @TargetApi(Build.VERSION_CODES.LOLLIPOP)
        @Override
        public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
            return shouldOverrideUrlLoading(view, request.getUrl().toString());
        }


        @Override
        public void onPageFinished(WebView view, String url) {
            super.onPageFinished(view, url);
            mWebView.setLayerType(View.LAYER_TYPE_HARDWARE, null);
        }

        @Override
        public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) {
            super.onReceivedError(view, errorCode, description, failingUrl);
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                return;
            }
           //根据业务场景自定义错误处理
        }

        @TargetApi(Build.VERSION_CODES.M)
        @Override
        public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
            super.onReceivedError(view, request, error);
            if (request.isForMainFrame()) {
                i//根据业务场景自定义错误处理
            }
        }

        @Override
        public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
            handler.proceed();//处理https证书问题}
    }
```
当然不能忘了处理WebView的生命周期
```java
    @Override
    public void onResume() {
        super.onResume();
        if (null != mWebView) {
            mWebView.resumeTimers();
            mWebView.onResume();
        }
        if(null != webSettings){
            webSettings.setJavaScriptEnabled(true);
        }
    }

    @Override
    public void onPause() {
        super.onPause();
        if (null != mWebView) {
            mWebView.pauseTimers();
            mWebView.onPause();
        }
    }

    @Override
    public void onStop() {
        super.onStop();
        if(null != webSettings){
            webSettings.setJavaScriptEnabled(false);
        }
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        if (mWebView != null) {
            mWebView.clearCache(true);
            if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.LOLLIPOP) {
                if (webViewLayout != null) {
                    webViewLayout.removeView(mWebView);
                }
                mWebView.removeAllViews();
                mWebView.destroy();
            } else {
                mWebView.removeAllViews();
                mWebView.destroy();
                if (webViewLayout != null) {
                    webViewLayout.removeView(mWebView);
                }
            }
            mWebView = null;
        }
    }
```
wap支付核心逻辑
```java
    private boolean startPay(String url) {
        try {
            if (url.startsWith("weixin://")) {
                Intent intent = new Intent();
                intent.setAction(Intent.ACTION_VIEW);
                intent.setData(Uri.parse(url));
                startActivity(intent);
                return true;
            } else if (parseScheme(url)) {
                Intent intent;
                intent = Intent.parseUri(url, Intent.URI_INTENT_SCHEME);
                intent.addCategory("android.intent.category.BROWSABLE");
                intent.setComponent(null);
                startActivity(intent);
                return true;
            }
        } catch (Exception e) {

        }
        return false;
    }

    public boolean parseScheme(String url) {
        if (url.contains("platformapi/startApp") || url.contains("platformapi/startapp")) {
            return true;
        } else if ((Build.VERSION.SDK_INT > Build.VERSION_CODES.M) && (url.contains("platformapi") && (url.contains("startApp") || url.contains("startpp")))) {
            return true;
        } else {
            return false;
        }
    }
```
相信你在其他文章中看到的是这个样子的：
```java
    public boolean parseScheme(String url) {
        if (url.contains("platformapi/startApp"))) {
            return true;
        } else if ((Build.VERSION.SDK_INT > Build.VERSION_CODES.M) && (url.contains("platformapi") && url.contains("startApp"))) {
            return true;
        } else {
            return false;
        }
    }
```
但是前不久支付宝改了wap支付的方式，之前是直接能在APP中调起支付界面的，现在调整为进入支付宝APP才能掉起支付界面，而且相应的URL格式发生变化，从startApp 变为了startapp，为了适应新的变化，只能两个都加上判断了（今后的版本可能还会有其他变化，可以考虑接入热更新保持SDK的稳定）

##总结

wap支付虽然没有什么高深的技术，但是需要一直花费精力去更新维护，保障业务闭环的稳定性。




