---
layout: post
title:  "[RxSwift] Transforming Operators"
author: Sung Kyungmo
catalog: true
tags: [ios, swift, rxswift, operators]
comments: true
header-img:   "img/title-bg/rxswift.jpg"
header-mask:  0.6
---
# 변환 연산자
---

## toArray

- 하나의 요소를 방출하는 옵저버블로 변환 (Single)
- 더이상 요소를 방출하지 않는 시점에 배열에 담아 전달

```swift
let subject = PublishSubject<Int>()

subject
    .toArray()
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

subject.onNext(1)
subject.onNext(2)
subject.onNext(3)
subject.onCompleted()

----- RESULT -----
success([1, 2, 3])
```
## map

- 원본 옵저버블이 방출하는 요소를 대상으로 함수를 실행하고 결과를 새로운 옵저버블로 리턴

```swift
let skills = ["Swift", "SwiftUI", "RxSwift"]

Observable.from(skills)
    .map { $0.count }
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(5)
next(7)
next(7)
completed
```
## flatMap

- 원본 옵져버블이 방출하는 항목을 새로운 옵져버블로 변환
- 새로운 옵져버블은 항목이 업데이트 될때마다 새로운 항목을 방출
- 최종적으로 하나의 옵져버블로 합쳐지고 모든항목이 옵져버블을 통해 방출
- 네트워크 요청을 구현할때 활용

```swift
let a = BehaviorSubject(value: 1)
let b = BehaviorSubject(value: 2)

let subject = PublishSubject<BehaviorSubject<Int>>()

subject
    .flatMap{ $0.asObserver() }
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

subject.onNext(a)
subject.onNext(b)

a.onNext(11)
b.onNext(22)

----- RESULT -----
next(1)
next(2)
next(11)
next(22)
```
## flatMapFirst

- 첫번째로 변환된 옵져버블이 방출하는 항목만 구독자로 전달

```swift
let a = BehaviorSubject(value: 1)
let b = BehaviorSubject(value: 2)

let subject = PublishSubject<BehaviorSubject<Int>>()

subject
   .flatMapFirst { $0.asObservable() }
   .subscribe { print($0) }
   .disposed(by: disposeBag)

subject.onNext(a)
subject.onNext(b)

a.onNext(11)
b.onNext(22)
b.onNext(222)
a.onNext(111)

----- RESULT -----
next(1)
next(11)
next(111)
```
## flatMapLatest

- 가장 최근의 항목을 방출한 옵저버블의 요소만 방출

```swift
let a = BehaviorSubject(value: 1)
let b = BehaviorSubject(value: 2)

let subject = PublishSubject<BehaviorSubject<Int>>()

subject
   .flatMapLatest { $0.asObservable() }
   .subscribe { print($0) }
   .disposed(by: disposeBag)

subject.onNext(a)
a.onNext(11)

subject.onNext(b)
b.onNext(22)

a.onNext(11)

----- RESULT -----
next(1)
next(11)
next(2)
next(22)
```
## scan

- 기본값으로 연산을 시작
- 원본 옵저버블이 방출하는 항목을 대상으로 변환을 실행한 다음 결과를 방출하는 하나의 옵저버블을 리턴
- 원본이 방출하는 항목의 수 = 구독자로 전달되는 항목의 수

```swift
// 1~10의 합
Observable.range(start: 1, count: 10)
    .scan(0, accumulator: +)
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(1)
next(3)
next(6)
next(10)
next(15)
next(21)
next(28)
next(36)
next(45)
next(55)
completed
```
## buffer

- 특정 주기동안 옵져버블이 방출하는 항목을 수집하고 하나의 배열로 리턴
- 컨트롤드 버퍼링이라고 함
- 파라미터는 최대시간, 최대갯수(하나만 충족하면 방출)
- 수집된 배열을 방출하는 옵저버블을 리턴

```swift
Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
    .buffer(timeSpan: .seconds(2), count: 3, scheduler: MainScheduler.instance)
    .take(5)
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next([0])
next([1, 2, 3])
next([4, 5])
next([6, 7])
next([8, 9])
completed
```
## window

- 최대시간, 최대갯수를 지정해 원본 옵저버블이 방출하는 항목들을 작은 단위의 옵저버블로 분해
- 옵저버블을 방출하는 옵저버블을 리턴(Inner Observable)

```swift
Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
    .window(timeSpan: .seconds(2), count: 3, scheduler: MainScheduler.instance)
    .take(5)
    .subscribe{
        print($0)
        
        if let observable = $0.element {
            observable.subscribe { print("inner : \($0)") }
        }
}
    .disposed(by: disposeBag)

----- RESULT -----
next(RxSwift.AddRef<Swift.Int>)
inner : next(0)
inner : completed
next(RxSwift.AddRef<Swift.Int>)
inner : next(1)
inner : next(2)
inner : next(3)
inner : completed
next(RxSwift.AddRef<Swift.Int>)
inner : next(4)
inner : next(5)
inner : completed
next(RxSwift.AddRef<Swift.Int>)
inner : next(6)
inner : next(7)
inner : completed
next(RxSwift.AddRef<Swift.Int>)
completed
inner : next(8)
inner : next(9)
inner : completed
```
## groupBy

- 방출되는 요소를 조건에 따라 그룹핑
- flatMap, toArray 를 활용해 최종 결과를 하나의 배열로 방출할 수 있음

```swift
let words = ["Apple","Banana","Orange","Book","City","Axe"]

// 문자열의 길이를 기준으로 그룹핑
Observable.from(words)
    .groupBy { $0.count }
    .subscribe(onNext: { (groupedObservable) in
        print("== \(groupedObservable.key)")
        groupedObservable.subscribe{ print(" \($0)") }
    })
    .disposed(by: disposeBag)

----- RESULT -----
== 5
 next(Apple)
== 6
 next(Banana)
 next(Orange)
== 4
 next(Book)
 next(City)
== 3
 next(Axe)
 completed
 completed
 completed
 completed
```
```swift
let words = ["Apple","Banana","Orange","Book","City","Axe"]

// 첫번째 문자로 그룹핑
Observable.from(words)
    .groupBy { $0.first }
    .flatMap{ $0.toArray() }
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(["City"])
next(["Orange"])
next(["Banana", "Book"])
next(["Apple", "Axe"])
completed
```
```swift
// 홀짝으로 그룹핑
Observable.range(start: 1, count: 10)
    .groupBy{ $0.isMultiple(of: 2) }
    .flatMap{ $0.toArray() }
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next([1, 3, 5, 7, 9])
next([2, 4, 6, 8, 10])
completed
```
<br>

## Reference
--- 
> [http://reactivex.io/](http://reactivex.io/)  
> [https://kxcoding.com/]( https://kxcoding.com/)
