---
layout: post
title: "WebView中载入Resource里的资源"
description: "Load resources from Resources flod in webview"
category: Snippet
tags: [webview, uikit]
imagefeature: codeplace.jpg
comments: false
mathjax: null
featured: false
published: true
---

需求：在html中载入Resources或者Boundle中的资源。

原理很简单，让html和要使用的声音、图片等资源放在同目录（或者相对目录），然后以一个公共目录为参考，找到他们。
比如把html文件和picture.png都放在Resource文件下，在html中如下写就可以访问到picture：
{% highlight html %}
<img src="picture.png"/>
{% endhighlight %}
具体实现如下：
{% highlight objective-c %}
NSString *filePath=[[NSBundle mainBundle] pathForResource:@"index_page" ofType:@"html"];
NSData *htmlData=[NSData dataWithContentsOfFile:filePath];
if (htmlData){
    [webView loadData:htmlData MIMEType:@"text/html" textEncodingName:@"UTF-8" baseURL:[[NSBundle mainBundle]resourceURL]];
}
{% endhighlight %}
