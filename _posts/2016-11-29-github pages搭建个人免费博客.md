---
layout: post
title: 使用github pages搭建个人免费博客
date: 2016-11-29
tags: 博客
---

如果你不熟悉建站各种编码语言，也不想花很多钱，却很想拥有一个独立的个人博客，那Github Pages是你绝佳的选择。github作为现在最流行的代码仓库，已经得到了很多大公司和项目的青睐。github推出的Github Pages服务不仅可以方便的为项目建立介绍站点，也可以用来建立个人博客。跟着本博的顺序，你将顺利搭建自己的博客。

### Github Pages

#### **创建仓库**

> 登录你自己的```github```账号，接下来去这个页面去创建一个新仓库<https://github.com/new>
这个新仓库就是存放你博客的地方
注意一点，你的仓库名字格式应为：```用户名.github.io```

![](/images/posts/使用github pages搭建个人免费博客/image1.png)

> 建完仓库，进入你刚创建的仓库，找到设置，进入设置页面

![](/images/posts/使用github pages搭建个人免费博客/image2.png)

> 在设置页面，一般都默认就好，在```github pages```那栏点击```launch automatic page generator```

![](/images/posts/使用github pages搭建个人免费博客/image3.png)

#### **编辑用户界面**

> 到这一步，就是编辑的博客页面展示信息

```Page name``` 页面的标题

```tagline``` 页面副标题(描述)

```body``` 正文

![](/images/posts/使用github pages搭建个人免费博客/image4.png)

> 一般都默认就可以了，反正当前只是创建一个demo页面，后续还得自己改
，直接点击下面的绿色按钮 ```continue to layouts```

![](/images/posts/使用github pages搭建个人免费博客/image5.png)

#### **公布页面**

> 随便选择一个主题模板（不用纠结，这里的主题很少，一般都是```jekyll```或者```hexo```的主题）,然后点击 ```Publish page```，你的页面就出来了。可以直接在浏览器中输入你的博客地址:```用户名.github.io```就可以看到效果了。

![](/images/posts/使用github pages搭建个人免费博客/image6.png)

#### **克隆仓库**

> 使用```git```工具或者命令行来```git```到本地，然后做自己的修改就可以了，换主题请往后看。

### Jekyll
jekyll是一个基于ruby开发的，专用于构建静态网站的程序。它能够将一些动态的组件：模板、liquid代码等构建成静态的页面集合，Github-Page全面引入jekyll作为其构建引擎，这也是学习jekyll的主要动力。同时，除了jekyll引擎本身，它还提供一整套功能，比如web server。我们用jekyll –server启动本地调试就是此项功能。读者可能已经发现，在启动server后，之前我们的项目目录下会多出一个_site目录。jekyll默认将转化的静态页面保存在_site目录下，并以某种方式组织。使用jekyll构建博客是十分适合的，因为其内建的对象就是专门为blog而生的，在后面的逐步介绍中读者会体会到这一点。但是需要强调的是，jekyll并不是博客软件，跟workpress之类的完全两码事，它仅仅是个一次性的模板解析引擎，它不能像动态服务端脚本那样处理请求。

更多关于```jekyll```请看

```github```: <https://github.com/jekyll/jekyll>

静态博客地址：<https://jekyllrb.com/>

中文静态博客地址：<http://jekyllcn.com/>

#### **选择主题**

> 刚才只是搭建好你的仓库，现在你该考虑你的博客风格了，到这个网站选择你喜欢的模板<http://jekyllthemes.org/>

![](/images/posts/使用github pages搭建个人免费博客/image7.png)

> 找到喜欢的主题下载到本地解压，将本地仓库除了```.git```隐藏文件夹的都删除，，将下载下来的主题的文件全部复制出来到git仓库

![](/images/posts/使用github pages搭建个人免费博客/image8.png)

#### **jekyll是如何工作的**

在jekyll解析你的网站之前，需要确保网站有以下目录结构：    

~~~
|-- _config.yml
|-- _includes
|-- _layouts
|   |-- default.html
|   |-- post.html
|-- _posts
|   |-- 20011-10-25-open-source-is-good.html
|   |-- 20011-04-26-hello-world.html
|-- _site
|-- index.html
|-- images
   |-- css
       |-- style.css
   |-- javascripts
~~~


>* ```_config.yml```：保存配置，该配置将影响jekyll构造网站的各种行为。

>* ```_includes```：该目录下的文件可以用来作为公共的内容被其他文章引用，就跟C语言include头文件的机制完全一样，jekyll在解析时会对```{ % include file.ext %}```标记扩展成对应的在_includes文件夹中的文件。

>* ```_layouts```：该目录下的文件作为主要的模板文件。

>* ```_posts```：文章或网页应当放在这个目录中，但需要注意的是，文章的文件名必须是```YYYY-MM-DD-title```。

>* ```_site```：上面提到过，这是jekyll默认的转化结果存放的目录。

>* ```images```：这个目录没有强制的要求，主要目的是存放你的资源文件，图片、样式表、脚本等。

以上就是大概的简介，可以在```_config.yml```文件中修改网站以及自己的个人信息，```_posts```文件夹放你的博文，切记文件名格式，更多详细的语法上面贴出来的```jekyll```的官方博客以及```github```可以参考。

现在可以将改动提交到远端仓库，提交后请求：```用户名.github.io```看看是不是变了。


### Jekyll安装本地服务
如果每次改动都需要提交到远端仓库才能看到效果的话，那就太麻烦了，有时候我们需要在本地测试好了，才提交上去，这就需要我们安装Jekyll本地服务。

**注意**：

   1. ruby版本，macOS自带了ruby,但是不建议用。
   2. 由于国内网络，gem官方的源可能不好用，切换成国内的。
	  
#### **Ruby安装**
* jekyll本身基于Ruby开发，想要本地搭建测试环境需要有Ruby环境。
虽然macOS有自带ruby，但是不推荐。我们使用rvm来管理ruby。
rvm官网：<https://rvm.io/> 安装有介绍。

~~~
\curl -sSL https://get.rvm.io | bash -s stable --ruby
~~~

* 安装完成查看版本

~~~
rvm -v
~~~

![](/images/posts/使用github pages搭建个人免费博客/image9.png)

* ```rvm list known``` 查看可用的ruby版本

![](/images/posts/使用github pages搭建个人免费博客/image10.png)

* 找到最新的版本（不一定是非要安装最新的）

~~~
rvm install 版本号(如2.2.0)
~~~

* 因为系统自带ruby，我们需要切换成我们自己的ruby版本

~~~
rvm use 2.2.0 --default
~~~

* 查看 ```rvm list``` 就可以看到当前默认的是我们自己的ruby版本。```ruby -v``` 也可以看到使我们自己的的ruby版本。

#### **安装jekyll**

* 首先，我们需要切换rubygem仓库镜像地址

~~~
sudo gem sources --remove http://rubygems.org/
sudo gem sources -a https://gems.ruby-china.org/
~~~

* 更新gem自身版本

~~~
gem update --system
~~~

![](/images/posts/使用github pages搭建个人免费博客/image11.png)

* 执行下面的命令安装jekyll

~~~
gem install jekyll
~~~

 如果有报错提示权限问题的话，在Mac OS X El Capitan或者更高版本下
 
~~~
gem install -n /usr/local/bin jekyll
~~~

* 待安装成功后，终端文件夹路径切到本地仓库下，执行

~~~
jekyll server
~~~

如果报错，看下报错信息（原谅我当时没有保存报错信息），如果有提到bundler的话。

~~~
gem install bundler
~~~

待成功后继续执行

~~~
jekyll server
~~~

如果还是报错，如果有提到 bundle install 的话,执行：

~~~
bundle install
~~~

继续执行

~~~
jekyll server
~~~

![](/images/posts/使用github pages搭建个人免费博客/image12.png)

控制台提示: Server address: http://127.0.0.1:4000/，将这个地址粘贴到浏览器中请求看看，你的博客出来了。

这样就可以本地测试，如果修改了，启动jekyll服务进行本地测试，测试没有问题再提交到远端github仓库。

### 域名绑定
购买域名，国内，国外的域名服务商都可以。我自己在阿里云买的域名。
假如你现在购买成功了如：```abc.com```
你需要在阿里云后台去配置这个域名的解析

![](/images/posts/使用github pages搭建个人免费博客/image14.png)

```ping 用户名.github.io``` 在终端执行，会看到一个IP，记住这个IP

![](/images/posts/使用github pages搭建个人免费博客/image15.png)

![](/images/posts/使用github pages搭建个人免费博客/image16.png)

在git本地仓库下，新建一个CNAME文件，里面写上你购买的域名，保存提交。过一会，就可以在你的github的仓库设置中看到你的域名已经配置上去了。
这下可以在浏览器中访问```abc.com```就可以看到你的博客了。

### 总结

	至此就搭建完成，可以愉快的写作了！搭建的过程挺不顺利的，安装本地测试服务的时候遇到各种版本、路径、权限问题，不过都解决了。大家如果有什么问题，欢迎留言。
	
	感谢帮过我的各个博主。
	http://www.ezlippi.com/
	http://baixin.io/
	http://cyzus.github.io/