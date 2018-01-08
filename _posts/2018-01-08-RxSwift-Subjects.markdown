---
layout:     post
title:      "RxSwift-Subjects"
subtitle:   " \"RxSwift之Subjects\""
date:       2018-01-08 17:00:00
author:     "RobertYeYu"
header-img: "img/post-bg-2015.jpg"
tags:
    - RxSwift
    - 读书笔记
---


## 引子
Observables是RxSwift的基础，但是我们在开发中经常需要在运行时向一个observable添加新值并发射给他的订阅者。所以我们需要一个既能充当**observable**又能充当**observer**的角色(还记得格鲁和小黄人的比喻么)。而这就是我们这篇要说的**Subject**。

## 什么是subjects
正如前面所说，subjects既是observer，又是observable。所以它既可以发射*.next*这样的事件，又可以接受订阅。

- PublishSubject: 开始的时候是空的，并且只给订阅者发送新值。也就是说订阅者只能收到订阅之后，PublishSubject发送的值。
- BehaviorSubject: 开始的时候需要有一个初始值，当接到订阅的时候，会把最新的那个值(有可能是初始值)发送给新的订阅者。
- Variable: 封装了BehaviorSubject。(preserves its current value as state, and replays only the latest/initial value to new subscribers.)

## 图解各种Subject
![java-javascript](/img/in-post/post-RxSwift/RxSwift_Subjects_01.png)![java-javascript](/img/in-post/post-RxSwift/RxSwift_Subjects_02.png)![java-javascript](/img/in-post/post-RxSwift/RxSwift_Subjects_03.png)

这三张图，分别生动形象的说明了PublishSubjects，BehaviorSubjects以及一个有点特别的ReplaySubjects。向上的箭头代表订阅，向下的箭头代表发射数据(ReplaySubjects中间那根线是从一开始就订阅了的)。

ReplaySubjects只有一点点不同，就是可以设置一个buffer，能缓存若干了值，在新的订阅者订阅的时候，发射给他。

## Variables
正如之前提到过的，Variable封装了BehaviorSubject并且存储了当前的值(value)。你可以通过**value**属性来获取Variable当前的值，并且，不像别的subjects或者observables，你也可以用这个**value**属性给variable设置新值。换言之，你不用*onNext(_:)*方法了。

因为他封装了BehaviorSubject，所以variable在创建的时候需要初始值，并且他会replay最近的一个值给订阅者。获得他subject拥有的行为，需要调用他的**asObservable()**方法。

Variable与其他Subject还有一点不同，就是它保证不会发射*.error*。并且Variable在销毁的时候(deallocated)，它会自动发送一个*.complete*方法。所以你既不能手动发射*.error*，也不能手动发射*.completed*。

```
var variable = Variable("Initial value")
let disposeBag = DisposeBag()

variable.value = "New initial value"

variable.asObservable()
	.subscribe { print(label: "1)", event: $0)} 
	.disposed(by: disposeBag)
// 到这里打印: 1) New initial value

variable.value = "1"

variable.asObservable()
	.subscribe { print(label: "2)", event: $0) }
	.disposed(by: disposeBag)

variable.value = "2"
// 到这里打印: 
1) 1 
2) 1  
1) 2
2) 2
```





