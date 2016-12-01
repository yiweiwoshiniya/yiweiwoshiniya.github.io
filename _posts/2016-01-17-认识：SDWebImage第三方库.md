---
layout: post
title: 认识：SDWebImage第三方库
date: 2016-01-17
tags: 博客
---

iOS 第三方库

**新浪博客迁移过来的文章**
---

这个类库提供一个UIImageView类别以支持加载来自网络的远程图片。具有缓存管理、异步下载、同一个URL下载次数控制和优化等特征。ios9.0出来以后，不知道SDWebImage支持了NSUrlSession了没有，应该是没有，我在git上下载的最新的仍然用的是NSURLConnection。目前AFNetworking已经支持。[github链接](https://github.com/rs/SDWebImage)

### 先看代码:

~~~
#import "UIImageView+WebCache.h"
~~~

~~~
UIImageView *imageView1 = [[UIImageView alloc]initWithFrame:self.view.bounds];
    [self.view addSubview:imageView1];
    
    NSURL *url = [NSURL URLWithString:@"http://s1.dwstatic.com/group1/M00/18/23/1244a21b95f70cb20db2489fe69fb3bb.gif"];
    UIImage *placeholder = [UIImage imageNamed:@"1.png"];
    [imageView1 sd_setImageWithURL:url placeholderImage:placeholder completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, NSURL *imageURL) {
        NSLog(@"加载完成");
 

    }];
~~~

* 还可以实时查看进度

~~~
[imageView1 sd_setImageWithURL:url placeholderImage:placeholder options:options progress:^(NSInteger receivedSize, NSInteger expectedSize) {
        NSLog(@"进度：%f",1.0*receivedSize/expectedSize);
    } completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, NSURL *imageURL) {
         NSLog(@"加载完成");
 
    }];

~~~

* 上面的options有很多，比如

~~~
SDWebImageOptions options = SDWebImageRetryFailed |SDWebImageProgressiveDownload;
~~~

* 第一个支持下载失败重新下载，第二个支持边下边显示（如果图片过大的话，这样会提高用户体验）。<br>
有时候我们会遇到好多控制类都加载图片，使用这个库，他会帮你做缓存处理，但这样图片多的话就会报内存警告，我们只需要在appDelegate中处理便是，他也为我们提供了方法


~~~
-(void)applicationDidReceiveMemoryWarning:(UIApplication *)application
{
    //取消正在下载的
    [[SDWebImageManager sharedManager] cancelAll];
    //清除缓存(内存中的)
    [[SDWebImageManager sharedManager].imageCache clearMemory];
 
}
~~~

### 下面附一些SDWebImage的原理，就是执行流程
 
1. 入口 setImageWithURL:placeholderImage:options: 会先把 placeholderImage显示，然后 SDWebImageManager 根据 URL 开始处理图片。
2. 进入 SDWebImageManager-downloadWithURL:delegate:options:userInfo:，交给 SDImageCache 从缓存查找图片是否已经下载 queryDiskCacheForKey:delegate:userInfo:.
3. 先从内存图片缓存查找是否有图片，如果内存中已经有图片缓存，SDImageCacheDelegate回调 imageCache:didFindImage:forKey:userInfo: 到 SDWebImageManager。
4. SDWebImageManagerDelegate 回调 webImageManager:didFinishWithImage: 到 UIImageView+WebCache等前端展示图片。
5. 如果内存缓存中没有，生成 NSInvocationOperation添加到队列开始从硬盘查找图片是否已经缓存。
6. 根据 URLKey在硬盘缓存目录下尝试读取图片文件。这一步是在 NSOperation 进行的操作，所以回主线程进行结果回调 notifyDelegate:。
7. 如果上一操作从硬盘读取到了图片，将图片添加到内存缓存中（如果空闲内存过小，会先清空内存缓存）。SDImageCacheDelegate回调 imageCache:didFindImage:forKey:userInfo:。进而回调展示图片。
8. 如果从硬盘缓存目录读取不到图片，说明所有缓存都不存在该图片，需要下载图片，回调 imageCache:didNotFindImageForKey:userInfo:。
9. 共享或重新生成一个下载器 SDWebImageDownloader 开始下载图片。
10. 图片下载由 NSURLConnection来做，实现相关 delegate 来判断图片下载中、下载完成和下载失败。
11. connection:didReceiveData: 中利用 ImageIO做了按图片下载进度加载效果。
12. connectionDidFinishLoading: 数据下载完成后交给 SDWebImageDecoder 做图片解码处理。
13. 图片解码处理在一个 NSOperationQueue完成，不会拖慢主线程 UI。如果有需要对下载的图片进行二次处理，最好也在这里完成，效率会好很多。
14. 在主线程 notifyDelegateOnMainThreadWithInfo: 宣告解码完成，imageDecoder:didFinishDecodingImage:userInfo: 回调给 SDWebImageDownloader。
15. imageDownloader:didFinishWithImage: 回调给 SDWebImageManager告知图片下载完成。
16. 通知所有的 downloadDelegates下载完成，回调给需要的地方展示图片。
17. 将图片保存到 SDImageCache中，内存缓存和硬盘缓存同时保存。写文件到硬盘也在以单独 NSInvocationOperation 完成，避免拖慢主线程。
18. SDImageCache 在初始化的时候会注册一些消息通知，在内存警告或退到后台的时候清理内存图片缓存，应用结束的时候清理过期图片。
19. SDWI 也提供了 UIButton+WebCache 和 MKAnnotationView+WebCache，方便使用。
20. SDWebImagePrefetcher 可以预先下载图片，方便后续使用。

> 如果还想了解一些知识，这里的链接可以看看[百度文库这篇关于SDWebImage的文章](http://wenku.baidu.com/link?url=QFIAhWZsTKO_c1bRt7CyIMh9uFmAtCtD5Se2NhwBkZQ6_Q8ECWib1WANi604SXHH2XT1Jd66cJVXnlJwpU8_Am5i3_TxyReYUbzrgZs1v_a)