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

- Observable<Void>.empty()只发送一个complete事件，之后会立即终止执行。
- Observable<Any>.never()不发生任何事件，永远不停止。
- Observable<Int>.range(start: 1, count: 10)，会发射一系列的数

## Disposing和Terminating
记住，observable在被订阅之前什么都不做。正是订阅使得observable开始发送事件，直到它发出了一个.error或者.completed事件后终止。你也可以通过手动取消订阅来终止一个observable。
```
let observable = Observable.of("A", "B", "C")
let subscription = observable.subscribe { event in
	print(event)
}
subscription.dispose()
```
这样就显式的取消了订阅，这个例子中的observable便会停止发送事件。

当然一个个取消在工程上比较复杂，我们就引入了DisposeBag，只要吧subscribe添加到DisposeBag中，当disposeBag将要被销毁的时候(deallocated)，他将会对其中的每一个subscribe调用dispose()方法，相当于一个统一的处理方法。

```
let disposeBag = DisposeBag()
Observable.of("A", "B", "C")
	.subscribe { print($0) }
	.disposed(by: disposeBag)
```
不调用有可能会导致内存泄露。

你也可以用**create**操作符来指定observable可以发射给订阅者的事件，不仅仅是element哟，create方法能让你指定event。

```
let disposeBag = DisposeBag()
Observable<String>.create { observer in
	observer.onNext("1")
	observer.onCompleted()
	observer.onNext("?")
	return Disposables.create()
}
.subscribe( 
	onNext: { print($0) },
	onError: { print($0) },
	onCompleted: { print("Completed") }, 
	onDisposed: { print("Disposed") } 
)
.disposed(by: disposeBag)
```

上面那个create方法的描述如下
```
static func create(_ subscribe: @escaping 
(AnyObserver<String>) -> Disposable)
-> Observable<String>

// 更通用的方法声明
public static func create(_ subscribe: @escaping 
(RxSwift.AnyObserver<RxSwift.Observable.E>) -> Disposable) 
-> RxSwift.Observable<RxSwift.Observable.E>
```
subscribe参数是一个逃逸闭包，接收一个AnyObserver类型的参数并且返回一个Disposable。AnyObserver是一个用来将事件方便的加入observable队列中的通用类型，然后observable会把事件发生给订阅者。

简单来说，observer就像住在observable中的工人，observable一旦被subscribers订阅，observable就会把observer生产的东西发生给订阅者。

举个例子，observable就像神偷奶爸中的格鲁，observer就像是小黄人...只不过在这里格鲁是一个和平主义者，不会主动去攻击别人，只有当坏人subscribers找上门来，格鲁才会向敌人发射小黄人生产的子弹（或者是各种脑洞大开的武器弹药）。

当然我们可以订阅这个create出来的observable了，还可以订阅它的各种事件。

## 创建observable工厂
和创建一个observable并等待subscribers来订阅不同的是，我们可以创建一个observable工厂来给每一个订阅者创建一个新的observable。

```
let disposeBag = DisposeBag()
var flip = false
let factory: Observable<Int> = Observable.deferred {
	flip = !flip
	if flip {
		return Observable.of(1, 2, 3) 
	} else {
		return Observable.of(4, 5, 6)
	}
}

for _ in 0...3 {
	factory.subscribe(onNext: {
		print($0, terminator: "") 
	}) 
	.disposed(by: disposeBag)
	print()
}
// 打印结果: 123 456 123 456
```
打印结果表明，每次都是一个新的observable。也就是说，通过deferred关键字，我们的工厂方法每次都生产出一个新的observable。




