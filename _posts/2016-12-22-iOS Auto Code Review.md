---
layout: post
title: iOS Auto Code Review（本地）整理
date: 2016-12-22
tags: 博客
---

Code Review在项目开发中是非常必要的。但是大部分团队，要么因为人员的流动性或者自身技术的问题，造成Code Review的难度太大，太耗时间。而现在我们可以把一些基础的跟业务没有关系的Review交给计算机，我们只关心业务代码是否有缺陷，这将大大减轻我们的压力，提高团队合作代码的质量。

## Objective-C

![效果图](/images/posts/iOS Auto Code Review/image1.png)

### 1. OCLint简单介绍

[OCLint](http://oclint.org/)是一个静态代码分析工具，目的是用于提升代码的质量和减少代码可能存在的缺陷。主要是用于 C、C++ 和 Objective-c 语言，同时解决一些潜在的问题。
OCLint会精确的指出哪一行代码违反了哪一等级的规则。在我们本地编译的时候，他会做这么几件事：

1. 生成 xcodebuild.log 文件（当然如果使用的是oclint-xcodebuild的话）
2. 生成 compile_commands.json 文件
3. OCLint读取自定义的或者自带的Rules,逐个扫描compile_commands.json中的每一个文件
4. OCLint将生成的报告展示在Xcode上
	
### 2. OCLint使用（安装到代码脚本配置）

#### 1. 安装
我们选择HomeBrew安装，如图
	
![OCLint安装](/images/posts/iOS Auto Code Review/image2.png)
	
生成compile_commands.json文件有三种方式，oclint-xcodebuild、xcpretty、xctool，oclint-xcodebuild由于OCLint团队不再维护，xctool团队也说对高版本的Xcode不再支持，所以我们使用OCLint团队推荐的xcpretty，而且xcpretty编译更快。
	
~~~
gem install xcpretty
~~~

#### 2. 使用
官方有详细的介绍[Using OCLint in Xcode](http://docs.oclint.org/en/stable/guide/xcode.html)这里就贴出我使用的shell脚本吧。
		
~~~	
export LC_ALL="en_US.UTF-8"
if [ -f ~/.bash_profile ]; then
source ~/.bash_profile
fi
hash oclint &> /dev/null
if [ $? -eq 1 ]; then
echo >&2 "oclint not found, analyzing stopped"
exit 1
fi
cd ${SRCROOT}
xcodebuild -target target名 -configuration Debug -scheme scheme名 -dry-run | xcpretty -r json-compilation-database --output compile_commands.json
oclint-json-compilation-database -- -report-type xcode
~~~

### 3. 规则的使用，以及自定义自己公司的规则
OCLint提供的规则[Rule Index](http://docs.oclint.org/en/stable/rules/index.html)，截止目前一共71条。

* 忽略一写文件或者文件夹


~~~
-e pods
~~~


* 禁用一些规则

~~~
-disable-rule=InvertedLogic 
-disable-rule=CollapsibleIfStatements \
-disable-rule=UnusedMethodParameter \
~~~
	
* 更改一些规则的阈值
	
~~~
-rc=LONG_LINE=200
-rc=NCSS_METHOD=100
~~~
	
**对于自定义规则目前还没有研究，这个需要学习CMake以及Clang AST等等相关知识，只能待以后研究。**

## Swift

![效果图](/images/posts/iOS Auto Code Review/image3.png)

### 1. SwiftLint简单介绍
	
SwiftLint 是一个用于强制检查 Swift 代码风格和规定的一个工具，基本上以 GitHub's Swift 代码风格指南为基础。

SwiftLint Hook 了 Clang 和 SourceKit 从而能够使用 AST 来表示源代码文件的更多精确结果。

### 2. SwiftLint使用（安装到代码脚本配置）
使用 Homebrew 安装
	
~~~
brew install swiftlint
~~~
	
整合 SwiftLint 到 Xcode 体系中去从而可以使警告和错误显示到 IDE 上，只需要在 Xcode 中添加一个新的"Run Script Phase"并且包含如下代码即可：
	
~~~
if which swiftlint >/dev/null; then
  swiftlint
else
  echo "warning: SwiftLint not installed, download from https://github.com/realm/SwiftLint"
fi
~~~

### 3. 规则的使用，以及自定义公司的规则

在控制台输入`swiftlint rules`可以看到当前所以的规则：
	
![SwiftLint当前的所以规则](/images/posts/iOS Auto Code Review/image4.jpg)
	
最好的方式是SwiftLint和[Swift Style Guide](https://github.com/kevindelord/swift-style-guide)配合，执行下面这条命令（在项目根目录下）
	
~~~
curl https://raw.githubusercontent.com/kevindelord/swift-style-guide/master/.swiftlint.yml > .swiftlint.yml
~~~
	
这个文件里面你可以包含以下规则（只是部分）:

* `disabled_rules`：关闭某些默认开启的规则
* `opt_in_rules`：一些规则是可选的
* `whitelist_rules`：不可以和`disabled_rules`或者`opt_in_rules`并列。类似一个白名单，只有在这个列表中的规则才是开启的。
	
~~~
disabled_rules: # 执行时排除掉的规则
  - colon
  - comma
  - control_statement
opt_in_rules: # 一些规则仅仅是可选的
  - empty_count
  - missing_docs
  # 可以通过执行如下指令来查找所有可用的规则:
  # swiftlint rules
included: # 执行 linting 时包含的路径。如果出现这个 `--path` 会被忽略。
  - Source
excluded: # 执行 linting 时忽略的路径。 优先级比 `included` 更高。
  - Carthage
  - Pods
  - Source/ExcludedFolder
  - Source/ExcludedFile.swift
# 可配置的规则可以通过这个配置文件来自定义
# 二进制规则可以设置他们的严格程度
force_cast: warning # 隐式
force_try:
  severity: warning # 显式
# 同时有警告和错误等级的规则，可以只设置它的警告等级
# 隐式
line_length: 110
~~~

可以用相应的语法自定义规则：
	
~~~
  pirates_beat_ninjas: #规则标示符
    name: "Pirates Beat Ninjas" #规则名称， 可选
    regex: "([n,N]inja)" # 匹配的模式
    match_kinds: # 需要匹配的语法类型， 可选
      - comment
      - identifier
    message: "Pirates are better than ninjas." # 提示信息，可选
    severity: error # 提示级别， 可选
  no_hiding_in_strings:
    regex: "([n,N]inja)"
    match_kinds: string
  no_chinese_word:
    regex: "([\u4e00-\u9fa5])"
    message: "不能有中文"
    match_kinds: string
    severity: error
~~~

## Objective-C和Swift混合项目

目前我是创建两个shell脚本来分别执行，如图

![效果图](/images/posts/iOS Auto Code Review/image5.png)

## 参考引用

[http://oclint.org/](http://oclint.org/)

[https://github.com/realm/SwiftLint](https://github.com/realm/SwiftLint)

[http://kevindelord.io/2016/04/06/integrate-swiftlint/](http://kevindelord.io/2016/04/06/integrate-swiftlint/)

[https://shengpan.net/auto-code-review/](https://shengpan.net/auto-code-review/)
