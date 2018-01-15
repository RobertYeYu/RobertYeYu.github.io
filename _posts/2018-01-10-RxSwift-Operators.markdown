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


## Transforming Operators

### Transforming elements
**toArray** 将元素转为数组
**map** 和swift标准库的map函数一样，只不过作用于Observables。
**mapWithIndex** block中另外返回index参数，用于控制在某位置进行map操作
```swift
Observable.of(1, 2, 3, 4, 5, 6)
  .mapWithIndex { integer, index in
    index > 2 ? integer * 2 : integer 
  }
  .subscribe(onNext: {print($0) })
// 输出1，2，3，8，10，12
```
### Transforming inner observables
![java-javascript](/img/in-post/post-RxSwift/RxSwift_Operators_01.png)
**flatMap** 在swift标准库里，flatMap会将二维数组降维，将其子数组中的元素取出来。flatMap在Rx中的设计原理也类似，将Observable从Observable中取出来，并flat down。
```swift
let disposeBag = DisposeBag()
let ryan = Student(score: Variable(80))
let charlotte = Student(score: Variable(90))
let student = PublishSubject<Student>()

student.asObservable()
  .flatMap { $0.score.asObservable() }
  .subscribe(onNext: {
    print($0) 
  })
  .disposed(by: disposeBag)

student.onNext(ryan)        // 输出80
ryan.score.value = 85       // 输出85
student.onNext(charlotte)   // 输出90
ryan.score.value = 95       // 输出95
charlotte.score.value = 100 // 输出100
```


## Combining Operators

### Prefixing and concatenating
![java-javascript](/img/in-post/post-RxSwift/RxSwift_Operators_02.png)
**startWith** 为队列添加前缀值
```swift
let numbers = Observable.of(2, 3, 4)
let observable = numbers.startWith(1) 
observable.subscribe(onNext: { value in print(value) })
// 输出1 2 3 4
```
![java-javascript](/img/in-post/post-RxSwift/RxSwift_Operators_03.png)
**.concat** 合并两个队列
```swift
let first = Observable.of(1, 2, 3) 
let second = Observable.of(4, 5, 6)
observable = Observable.concat([first, second])

observable.subscribe(onNext: { value in print(value) })
```
以这种方式新创建的Observable会订阅第一个Observable，直到第一个队列结束后，切换到第二个队列，以此类推。如果其中任何一个发出.error事件，链接后的新建队列会转发error并立即退出。还有一种链接方式是，first.concat(second)，效果相同。

还有一种添加前缀并链接的方法，就是just加concat。
```swift
let numbers = Observable.of(2, 3, 4)
let observable = Observable 
  .just(1)
  .concat(numbers)
```
### Merging
![java-javascript](/img/in-post/post-RxSwift/RxSwift_Operators_04.png)
**merge()** 一个merge()observable会订阅每一个队列里面的元素，每当收到一个事件的时候便立即转发，没有预先设定的顺序。
```swift
let left = PublishSubject<String>() 
let right = PublishSubject<String>()
let source = Observable.of(left.asObservable(), right.asObservable())

let observable = source.merge() 
let disposable = observable.subscribe(onNext: { value in print(value) })
```
- merge()会在source队列和它所有的内部队列结束之后结束
- 内部队列结束的顺序是无所谓的
- 如果任何一个队列发射一个*.error*信号，merge生成的队列立刻转发这个错误，然后就终止了。
**merge(maxConcurrent)** 这个重载方法会一直订阅进入的队列，直到达到上限。之后，再进入的observable会排队等待。每当source队列中有一个observable完成之后，他会按照顺序从等待队列中订阅一个新的observable。
### Combining elements
![java-javascript](/img/in-post/post-RxSwift/RxSwift_Operators_05.png)
**combineLatest** 从图上就可以看出，combineLatest操作符会把两个队列中最新的两个元素组合起来返回。
```swift
let left = PublishSubject<String>() 
let right = PublishSubject<String>()

let observable = Observable.combineLatest(left, right, resultSelector: { lastLeft, lastRight in 
  "\(lastLeft) \(lastRight)" 
})
```
combineLatest的一些重载
简洁优雅
```swift
let observable = Observable.combineLatest(left, right) { ($0, $1) } 
  .filter { !$0.0.isEmpty }
```

两个参数可以不是相同的类型
```swift
let choice : Observable<DateFormatter.Style> = Observable.of(.short, .long)
let dates = Observable.of(Date())
let observable = Observable.combineLatest(choice, dates) { (format, when) -> String in 
  let formatter = DateFormatter() 
  formatter.dateStyle = format 
  return formatter.string(from: when) 
}
observable.subscribe(onNext: { value in print(value) })
```

接受一个集合类型，因为是集合类型，那么每个元素的类型就必须相同了
```swift
let observable = Observable.combineLatest([left, right]) { strings in  
  strings.joined(separator: " ") 
}
```

![java-javascript](/img/in-post/post-RxSwift/RxSwift_Operators_06.png)
**zip** zip就是两两结合咯，不管哪个队列的先发射，都要等一个新的小伙伴一起。
### Triggers


