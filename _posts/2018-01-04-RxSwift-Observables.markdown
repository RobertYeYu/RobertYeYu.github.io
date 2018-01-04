---
layout:     post
title:      "RxSwift-Observables"
subtitle:   " \"RxSwift之Observables\""
date:       2018-01-04 15:00:00
author:     "RobertYeYu"
header-img: "img/post-bg-2015.jpg"
tags:
    - RxSwift
    - 读书笔记
---

## observable基础概念
“observable” “observable sequence” 和 “sequence”在Rx中交替出现，其实都是一个意思，还有一些称其为“stream”，我们暂且称这些为“sequence”，一切都是“sequence”。Observable也是一种特别的sequence，最特别且重要的一点是，Observable是异步的。Observables产生events，这个过程叫做“emitting”，发射，这些发射出来的事件(events)带着值(values)，比如数字，对象，或者是点击手势。
observable就是一个能发射带有值的事件的异步队列。

## observable的生命循环
- 一个observable会发射包含元素的**next**事件，直到：
- ...发射了一个**error**事件，这个事件会包含着错误信息
- ...发射了一个**completed**事件，observable会正常终止
- 一旦observable终止，就不能再发送事件了

## 创建observables
使用**just**，**of**，**from**创建Observable

```
let one = 1 
let two = 2
let three = 3

let observable = Observable.just(one) // Observable<Int>类型，就只有这个一个元素
let observable = Observable.of(one, two, three) // Observable<Int>类型，包含三个元素
let observable = Observable.of([one, two, three]) // Observable<[Int]>类型，包含一个元素
let observable = Observable.from([one, two, three]) // Observable<Int>类型，包含三个元素
```

## 订阅observables
我们很熟悉NotificationCenter，他会将post出来的通知广播给所有订阅者，然而Observables不是这样的。
```
let observer = NotificationCenter.default.addObserver( 
	forName: .UIKeyboardDidChangeFrame, 
	object: nil, 
	queue: nil 
) { notification in 
	// Handle receiving notification 
}
```
订阅Observables不是用**addObserver**而是用**subscribing**，并且.default获取一个单例，而每一个observable都是不同的。
最重要的是，一个observable在它被订阅之前是不会发送事件的，换句话说，observable只是要有一个订阅者(subscriber)才会开始发送事件。

订阅Observables之后会接受到三种事件，next，error以及complete，并且可以对这三种不同的事件分别处理。

只有next事件带有发射的元素，

```
observable.subscribe { event in
	// event: next(1) next(2) next(3) completed
	if let element = event.element { print(element) }
	// event.element: 1 2 3
}

// 可以简单写成
// 处理Next事件
observable.subscribe(onNext: { element in print(element) })
```

Observable<Void>.empty()只发送一个complete事件，之后会立即终止执行。
Observable<Any>.never()不发生任何事件，永远不停止。
Observable<Int>.range(start: 1, count: 10)，会发射一系列的数



