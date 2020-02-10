---
layout: post
title:  "[RxSwift] Filtering Operators"
author: Sung Kyungmo
catalog: true
tags: [ios, swift, rxswift, operator]
comments: true
header-img:   "img/title-bg/rxswift.jpg"
header-mask:  0.6
---

# 필터링 연산자

---

## ignoreElements

- 옵저버블이 방출하는 next이벤트를 필터링하고 completed,error만 방출

```swift
Observable.from([1,2,3])
    .ignoreElements()
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
completed
```
## elementAt

- 특정 인덱스에 위치한 요소를 제한적으로 방출
- 구독자에게 하나의 next이벤트 전달하고 completed

```swift
Observable.from([1,2,3])
    .elementAt(2)
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(3)
completed
```
## filter

- 옵저버블이 방출하는 이벤트를 필터링

```swift
// 짝수만 필터링
Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
    .filter{ $0.isMultiple(of: 2) }
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(2)
next(4)
next(6)
next(8)
next(10)
completed
```
## skip

- 지정된 갯수(n)만큼 무시

```swift
Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
    .skip(5)
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(6)
next(7)
next(8)
next(9)
next(10)
completed
```
## skipWhile

- 클로저에 true로 리턴되는동안 무시
- false 리턴한 이후부터 조건 없이 방출

```swift
Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
    .skipWhile { !$0.isMultiple(of: 2) }
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(2)
next(3)
next(4)
next(5)
next(6)
next(7)
next(8)
next(9)
next(10)
completed
```
## skipUntil

- 옵저버블을 파라미터로 받음(트리거)
- 옵저버블이 next이벤트를 전달하기 전까지 원본 옵저버블 전달하는 이벤트를 무시

```swift
let subject = PublishSubject<Int>()
let trigger = PublishSubject<Int>()

subject
    .skipUntil(trigger)
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

subject.onNext(1)

trigger.onNext(0)

subject.onNext(2)

----- RESULT -----
next(2)
```
## take

- 지정된 갯수만큼 이벤트 방출

```swift
Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
    .take(3)
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(1)
next(2)
next(3)
completed
```
## takeWhile

- 클로저가 false를 리턴하면 더이상 요소를 방출하지 않음

```swift
Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
    .takeWhile{ !$0.isMultiple(of: 2) }
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(1)
completed
```
## takeUntil

- 옵저버블을 파라미터로 받음
- trigger가 next 이벤트를 전달하기 전까지 원본 옵저버블에서 next이벤트를 전달

```swift
let subject = PublishSubject<Int>()
let trigger = PublishSubject<Int>()

subject
    .takeUntil(trigger)
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

subject.onNext(1)
subject.onNext(2)

trigger.onNext(0)

subject.onNext(3)

----- RESULT -----
next(1)
next(2)
completed
```
## takeLast

- 정수를 파라미터로 받아서 옵저버블을 리턴
- 리턴되는 옵저버블에는 원본 옵저버블이 방출하는 요소중에서 마지막에 방출된 N개의 요소가 포함되어 있음
- 구독자로 전달되는 시점이 Delay됨

```swift
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
let subject = PublishSubject<Int>()

 subject
     .takeLast(2)
     .subscribe{ print($0) }
     .disposed(by: disposeBag)

 numbers.forEach {
     subject.onNext($0)
 }

subject.onCompleted() //버퍼에 저장된 요소가 구독자에게 방출됨
//subject.onError(MyError.error) // 에러가 발생한 경우 요소가 방출되지 않고 에러 이벤트만 방출

----- RESULT -----
next(9)
next(10)
completed
```
## single

- 첫번째 요소, 조건과 일치하는 첫번째 요소만 방출
- 옵저버블이 요소를 방출하지 않거나 2개 이상의 요소를 방출하면 에러 발생

```swift
Observable.just(1)
    .single()
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(1)
completed
```
```swift
Observable.from(numbers) // 에러
    .single()
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(1)
error(Sequence contains more than one element.)
```
```swift
Observable.from(numbers)
    .single{ $0 == 3 }
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(3)
completed
```
```swift
let subject = PublishSubject<Int>()

subject
    .single()
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

subject.onNext(100) //completed 이벤트가 전달되기 전까지 대기
// completed 이벤트가 전달되는 시점에 하나의 요소만 방출된 상태라면 구독자에게 completed가 전달되고
// 그 사이에 다른 요소가 방출되었다면 구독자에게 error 이벤트 전달
// 하나의 요소가 방출되는것을 보장

----- RESULT -----
next(100)
```
## distinctUntilChanged

- 동일한 항목이 연속으로 방출되지 않도록 함
- 단순히 연속된 동일한 항목만 확인
- 이전에 동일한 항목이 방출되었더라도 신경쓰지 않음

```swift
let numbers = [1, 1, 3, 2, 2, 3, 1, 5, 5, 7, 7, 7]

Observable.from(numbers)
    .distinctUntilChanged()
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(1)
next(3)
next(2)
next(3)
next(1)
next(5)
next(7)
completed
```
## debounce

- 검색 기능에 주로 사용
- 파라미터에 전달하는 시간은 연산자가 next 이벤트를 방출할지 결정하는 조건으로 사용됨
- 옵저버가 next 이벤트를 방출한다음 지정된 시간동안 이벤트를 방출하지 않는다면 해당시점에 가장 마지막으로 방출된 next이벤트를 구독자에게 전달함
- 반대로 지정된 시간 이내에 또 다른 next이벤트를 방출했다면 타이머를 초기화함
- 타이머를 초기화한후 다시 지정된 시간만큼 대기
- 이 시간안에 이벤트가 방출되지 않는다면 마지막 이벤트를 방출하고 타이머를 다시 초기화

```swift
let buttonTap = Observable<String>.create{ observer in
    DispatchQueue.global().async {
        for i in 1...10 {
            observer.onNext("Tap \(i)")
            Thread.sleep(forTimeInterval: 0.3)
        }
        
        Thread.sleep(forTimeInterval: 1) //next(Tap 10)
        
        for i in 11...20 {
            observer.onNext("Tap \(i)")
            Thread.sleep(forTimeInterval: 0.5)
        }
        
        observer.onCompleted() //next(Tap 20) & completed
    }
    
    
    return Disposables.create()
}

buttonTap
    .debounce(.milliseconds(1000), scheduler: MainScheduler.instance)
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(Tap 10)
next(Tap 20)
completed
```
## throttle

- 지정된 주기동안 하나의 이벤트만 구독자에게 전달
- 짧은시간동안 반복되는 탭, 델리게이트에 주로 사용

```swift
let buttonTap = Observable<String>.create{ observer in
    DispatchQueue.global().async {
        for i in 1...10 {
            observer.onNext("Tap \(i)")
            Thread.sleep(forTimeInterval: 0.3)
        }
        
        Thread.sleep(forTimeInterval: 1)
        
        for i in 11...20 {
            observer.onNext("Tap \(i)")
            Thread.sleep(forTimeInterval: 0.5)
        }
        observer.onCompleted()
    }
    return Disposables.create()
}

buttonTap
    .throttle(.milliseconds(1000), scheduler: MainScheduler.instance)
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(Tap 1)
next(Tap 4)
next(Tap 7)
next(Tap 10)
next(Tap 11)
next(Tap 12)
next(Tap 14)
next(Tap 16)
next(Tap 18)
next(Tap 20)
completed
```
```swift
// latest: true -> 지정된 주기 마지막 이벤트를 전달(지정된 주기를 엄격하게 지킴)
Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
    .debug()
    .take(10)
    .throttle(.milliseconds(2500), latest: true, scheduler: MainScheduler.instance)
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
 -> subscribed
 -> Event next(0)
next(0)
 -> Event next(1)
 -> Event next(2)
next(2)
 -> Event next(3)
 -> Event next(4)
 -> Event next(5)
next(5)
 -> Event next(6)
 -> Event next(7)
next(7)
 -> Event next(8)
 -> Event next(9)
 -> isDisposed
next(9)
completed
```
```swift
// latest: false -> 지정된 주기가 지난후 처음 이벤트를 전달(지정된 주기를 초과할 수 있음)
Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
    .debug()
    .take(10)
    .throttle(.milliseconds(2500), latest: false, scheduler: MainScheduler.instance)
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
 -> subscribed
 -> Event next(0)
next(0)
 -> Event next(1)
 -> Event next(2)
 -> Event next(3)
next(3)
 -> Event next(4)
 -> Event next(5)
 -> Event next(6)
next(6)
 -> Event next(7)
 -> Event next(8)
 -> Event next(9)
next(9)
completed
 -> isDisposed
```
<br>

## Reference
--- 
> [http://reactivex.io/](http://reactivex.io/)  
> [https://kxcoding.com/]( https://kxcoding.com/)
