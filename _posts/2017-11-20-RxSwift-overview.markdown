---
layout:     post
title:      "RxSwift简介"
subtitle:   " \"初探RxSwift\""
date:       2017-11-20 15:00:00
author:     "RobertYeYu"
header-img: "img/post-bg-2015.jpg"
tags:
    - RxSwift
    - 读书笔记
---

> “Don't worry, Be happy ”


## 废话
终于下定决心写一些东西，因为最近总感觉，有一些东西就在嘴边，却怎么也想不起来。所以还是得记下来。烂笔头就这样用起来吧。

## RxSwift概览

> RxSwift is a library for composing asynchronous and event-based code by using observable sequences and functional style operators, allowing for parameterized execution via schedulers.

What！！！
本来我想翻译一下这句官方定义，但还是算了。但愿有一天我能信心满满的来翻译这一句话。但是，还是有几个点值得一看。

- asynchronous 异步
- event-based	 基于事件
- observable sequences 这个怎么说呢，反正一切都是sequences
- functional 函数式的

是不是很fashion，几乎每个点都是当下流行的一本书了

## 异步
在app中，在任一时刻很多事情会同时发生，下载文件，刷新界面，向磁盘写入，等等...
Cocoa和UIKit中有很多异步API:

- NotificationCenter: 通知
- The delegate pattern: 代理模式基本是我们常用的一种数据回传方式
- Grand Central Dispatch: GCD可以帮助我们把业务逻辑抽象出来。可以把将这些代码顺序执行，也可以按照优先级在不同的队列上并行执行
- Closures: 闭包可以让你把一段代码和上下文的参数传递给别的类，那样，接收到闭包的类就可以决定是否要执行这一段代码，执行多少次以及在特定上下文下执行

异步所造成的问题就是，数据会被很多源操作。比如你循环打印一个数组中的元素，那你打印出来的array一定是你传入的array，那如果我每点击一下按钮打印一个元素，那你点第五下的时候，可能数组的第五个元素都被替换甚至删除了。

另外，UI的刷新必须在主线程上，我们很多时候会忽略这个问题，有的时候tableView刷新，控件的位置错乱，就要看看是不是在主线程上执行的刷新方法了。

## 异步术语
### 1. State, and specifically, shared mutable state
State直白来看就是状态。就像你的电脑，也许刚打开的时候很好用，然而开了两三天之后就卡了。但是硬件还是那样，软件代码也没有变，发生变化了的就是State。

在内存和硬盘中的数据，对用户输入的反馈，从服务器下载内容留存下的痕迹--所有的这些就组成了电脑的State。

处理我们app的State，特别是那些当需要在多个异步组件中共享的时候，RxSwift的价值就体现出来了。

### 2. Imperative programming
命令式编程
```
override func viewDidAppear(_ animated: Bool) {
	super.viewDidAppear(animated)
	setupUI() 
	connectUIControls()
	createDataSource() 
	listenForChanges()
}
```

典型的命令式编程，在我的代码里几乎随处可见，这种方式从代码上来看，我们根本不知道他做了什么。是否更新了ViewController的属性？最关键是问题是，代码是不是必须按照这个顺序执行？那如果我们要更换其中两个方法的位置，结果会相同么？如果和你搭档的开发同学要在另一个ViewController中用这其中的两个方法，你对其没有任何约束和指导，甚至连信息都不是很明确，谨慎起见，就一定要看里面的实现。

### 3. Side effects
> Side effects are any change to the state outside of the current scope.

很简单，举个例子，就像之前的代码，connectUIControls，看这个方法应该是刷新了UI界面，这就是这一行代码的side effects。改变内存中的数据，刷新界面上的label，这些都是side effects。我们的代码就是要来制造这些side effects的，所以，关键在于怎么能保证这些side effects是可控制的。造成的这些影响，都出于你的本意。

RxSwift被设计来执行这样的事情

### 4. Declarative code
> Declarative code lets you define pieces of behavior, and RxSwift will run these behaviors any time there’s a relevant event and provide them an immutable, isolated data input to work with.

声明式的代码可以定义一种行为(behavior)，只有有相关的事件发生，RxSwift就会接受一组不可变且隔离的数据，来运行这一段代码。

这种数据形式使得在并行环境下，数据不会相互影响，设计逻辑的时候就可以简单处理，在某代码块中，代码也会顺序执行。

- immutable
- isolated

### 5. Reactive systems
Reactive systems我们一般翻译成响应式系统，特别抽象，主要包含以下几种特征

- Responsive: 总是让UI保持与数据同步，反映App的最新状态
- Resilient: 字面意思是有复原能力的。也就是说每一段描述行为的业务代码(Behavior)都是相互隔离并且可以从错误中恢复并继续执行。
- Elastic: 有弹性，可伸缩。就是说一串链式调用的代码处理了工作流上的很多部分，比如拉取数据，处理数据，控制事件等等。比如Worker.loadData.map.delay.shareTo.work
- Message driven: 组件之间通过消息机制来传递信息，以提高隔离性和可重用性，从生命周期和具体的类实现中解耦（TODO: 具体怎么讲呢）




 	


