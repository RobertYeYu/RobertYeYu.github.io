---
layout:     post
title:      "RxSwift-Operators"
subtitle:   " \"RxSwift之Operators\""
date:       2018-01-10 17:00:00
author:     "RobertYeYu"
header-img: "img/post-bg-2015.jpg"
tags:
    - RxSwift
    - 读书笔记
---



## 前言

> 数学是符号加逻辑。 —— 罗素

Operators是Rx的基础模块，可以用来转化、处理observables发射出来的事件。就像你可以用加减乘除来创建复杂的表达式一样，你也可以用这些operators链式表达app的逻辑。

---

## Catalog


1.  [Filtering Operators](#filtering--operators)
2.  [Transforming Operators](#transforming--operators)
3.  [Combining Operators](#combining--operators)
4.  [Time Based Operators](#time--based--operators)



## Filtering Operators

### Ignoring operators
**ignoreElements** 忽略全部*.next*，只会响应终止信号，*.completed*或者*.error*。
```swift
let strikes = PublishSubject<String>()
strikes.ignoreElements() 
strikes.onNext("X")
strikes.onNext("X")
strikes.onCompleted() // 只触发completed方法
```
**elementAt** 过滤后只留下指定index位置的元素。 
```swift
let strikes = PublishSubject<String>()
strikes.elementAt(2)
strikes.onNext("0")
strikes.onNext("1") 
strikes.onNext("2") // 触发
```
**filter** 条件过滤。
```swift
Observable.of(1, 2, 3, 4, 5, 6)
  .filter { integer in integer % 2 == 0 }
// 输出偶数2,4,6
```
### Skipping operators
**skip** 跳过若干个元素。
```swift
Observable.of("A", "B", "C", "D", "E", "F")
  .skip(3)
// 输出DEF
```
**skipWhile** 跳过前面所有符合条件的元素，一旦发现需要保留的元素，则跳过条件失效，保留后面所有的元素。
```swift
Observable.of(2, 2, 3, 4, 4)
  .skipWhile { integer in integer % 2 == 0 }
// 输出3,4,4
```
**skipUntil** 跳过所有元素，直到他参数中的那个Observable发射事件后，跳过失效。 
```swift
let subject = PublishSubject<String>() 
let trigger = PublishSubject<String>()
subject.skipUntil(trigger) 
subject.onNext("A") 
subject.onNext("B")
trigger.onNext("X") 
subject.onNext("C") // 输出C
```
### Taking operators
**take** 接受若干个元素
```swift
Observable.of(1, 2, 3, 4, 5, 6)
  .take(3) // 输出123
```
**takeWhileWithIndex**  根据条件接受元素，block中另外有index值
```swift
Observable.of(2, 2, 4, 4, 6, 6)
  .takeWhileWithIndex { integer, index in
    integer % 2 == 0 && index < 3 
  } // 输出224
```
**takeUntil** 接受元素，直到事件发生，和skipUntil相似
### Distinct operators
避免重复数据。
**distinctUntilChanged** 默认实现去重复
```swift
Observable.of("A", "A", "B", "B", "A")
  .distinctUntilChanged() // 输出ABA
```
**distinctUntilChanged** 当block中的条件触发，则过滤触发条件的元素。
```swift
Observable<Int>.of(10, 10, 20)
  .distinctUntilChanged { a, b in
    a.value == b.value // 这里需要返回一个Bool值
  } // 输出10,20
```


