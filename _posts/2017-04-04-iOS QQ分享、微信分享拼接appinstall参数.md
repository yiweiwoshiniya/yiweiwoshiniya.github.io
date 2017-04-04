---
layout: post
title: iOS QQ分享、微信分享拼接appinstall参数
date: 2017-04-04
tags: 博客
---


标签： iOS、QQ、分享、appinstall

前几天开发，测试吐槽iOS分享链接到QQ，在QQ内打不开，当时找安卓同学发现没有问题，经过排查，发现QQ给自己的链接后面默认拼接了一个`appinstall=0`参数，查了腾讯开放平台，这个参数是QQ定向分享来判断是否安装了自己的软件。

但是这并不能解决问题，这个参数我是没有找到如何取消，不让拼接这个参数，但是问题得解决啊。和同事讨论后，从URL来看，下面是iOS NSURL的一些属性。

~~~
@property (nullable, readonly, copy) NSString *host;
@property (nullable, readonly, copy) NSNumber *port;
@property (nullable, readonly, copy) NSString *user;
@property (nullable, readonly, copy) NSString *password;
@property (nullable, readonly, copy) NSString *path;
@property (nullable, readonly, copy) NSString *fragment;
@property (nullable, readonly, copy) NSString *parameterString;
@property (nullable, readonly, copy) NSString *query;
@property (nullable, readonly, copy) NSString *relativePath; // The same as path if baseURL is nil
~~~
我们分享的URL大致是这个样子的：`http://www.baidu.com/?name=test#!/index/color`
分享后：`http://www.baidu.com/?name=test#!/index/color&appinstall=0`。


仔细会发现，我们有一个`name=test`的参数，`#!/index/color`这个是fragment，fragment用来定位跳转到本页面指定位置，例如：`<p id="bottom">` #号后面跟bottom，这个页面加载出来会自动跳转到bottom处。


问题就跟fragment有关，分享前是：`#!/index/color`，分享后：`#!/index/color&appinstall=0`。浏览器默认将#后面的都当成了fragment，所以导致页面加载没有问题，就是定位不到位置，所以显示不出来。


最后问题解决就是让同事在分享前后面拼接一个？。`http://www.baidu.com/?name=test#!/index/color？`。这样QQ如果拼接上，那么浏览器会`appinstall=0`解析成请求参数。

**结尾**

如果有知道好的解决方案，求告知。问题就是链接被拼接了一个参数后解析错误，自己对这方面不是很了解，自己的想法，将fragment放到host后面和请求参数换一个顺序，请求参数放到链接最后面，这样不管怎么拼接参数都没有问题。