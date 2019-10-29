---
layout: post
title:  "[RxSwift] Create Operators"
author: sung
categories: [ rxswift ]
tags: [swift, rxswift, Operators]
image: 
comments: true
---
## 생성 연산자

---

## just

- 파라미터로 전달한 단일 요소(element)를 그대로 방출

```swift
Observable.just(1)
   .subscribe { event in print(event) }
   .disposed(by: disposeBag)

Observable.just([1, 2, 3])
   .subscribe { event in print(event) }
   .disposed(by: disposeBag)

----- RESULT -----
next(1)
completed
next([1, 2, 3])
completed
```

## of

- 파라미터로 전달한 요소(elements)들을 하나씩 방출
- 배열도 그대로 방출
- 배열의 요소를 하나씩 방출하고 싶을땐 from 연산자 사용

```swift
Observable.of(1, 2, 3)
   .subscribe { element in print(element) }
   .disposed(by: disposeBag)

Observable.of([1, 2], [3, 4], [5, 6])
   .subscribe { element in print(element) }
   .disposed(by: disposeBag)

----- RESULT -----
next(1)
next(2)
next(3)
completed
next([1, 2])
next([3, 4])
next([5, 6])
completed
```

## from

- 배열의 요소들을 하나씩 순서대로 방출

```swift
Observable.from([1,2,3])
   .subscribe { element in print(element) }
   .disposed(by: disposeBag)

----- RESULT -----
next(1)
next(2)
next(3)
completed
```

## range

- range(n, m)
- n부터 **1씩 증가**하는 m개의 **정수** 시퀀스 생성

```swift
Observable.range(start: 1, count: 5)
   .subscribe { print($0) }
   .disposed(by: disposeBag)

----- RESULT -----
next(1)
next(2)
next(3)
next(4)
next(5)
completed
```

## generate

- 일정 크기만큼 증가하거나 감소하는 시퀀스 생성

```swift
// 0~10까지 2씩 증가하는 옵저버블
Observable.generate(initialState: 0, condition: { $0 <= 10 }, iterate: { $0 + 2 })
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

// 10~0까지 2씩 감소하는 옵저버블
Observable.generate(initialState: 10, condition: { $0 >= 0}, iterate: { $0 - 2 })
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(0)
next(2)
next(4)
next(6)
next(8)
next(10)
completed
next(10)
next(8)
next(6)
next(4)
next(2)
next(0)
completed
```

## repeatElement

- 동일한 요소를 반복적으로 방출하는 연산자
- take(N) 으로 갯수를 지정하지 않으면 무한대로 방출

```swift
Observable.repeatElement("A")
    .take(3)
    .subscribe { print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(A)
next(A)
next(A)
completed
```

## defer

- timer와 비슷
- subscribe하기 전까지 미뤄지기 때문에 최신 데이터를 얻을 수 있음
- 특정 조건에 따라 옵저버블을 생성할 수 있음
- 옵저버블을 리턴하는 클로저를 파라미터로 받음

```swift
let animals = ["🐶", "🐱", "🐹"]
let fruits = ["🍎", "🍇", "🍓"]
var flag = true

let factory: Observable<String> = Observable.deferred {
    flag.toggle()
    return Observable.from(flag ? animals : fruits)
}

factory
    .subscribe { print($0) }
    .disposed(by: disposeBag)

factory
    .subscribe { print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
next(🍎)
next(🍇)
next(🍓)
completed
next(🐶)
next(🐱)
next(🐹)
completed
```

## create

- 동작을 직접 제어하고 싶을때 사용
- 옵저버블을 파라미터로 받아 Disposable을 리턴하는 클로저를 전달
- Disposable 이 아니라 Disposables 리턴

```swift
// 해당 URL의 HTML을 String으로 방출하는 옵저버블
Observable<String>.create{ (observer) -> Disposable in
    guard let url = URL(string: "https://www.apple.com") else {
        observer.onError(MyError.error)
        return Disposables.create()
    }
    
    guard let html = try? String(contentsOf: url, encoding: .utf8) else {
        observer.onError(MyError.error)
        return Disposables.create()
    }
    
    observer.onNext(html)
    observer.onCompleted()
    
    return Disposables.create()
}
    .subscribe { print($0) }
    .disposed(by: disposeBag)
```

## empty

- 어떠한 요소(next이벤트)도 방출하지 않는 옵저버블 생성
- completed 이벤트를 전달하고 종료

```swift
Observable<Void>.empty()
    .subscribe { print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
completed
```

## error

- 어떠한 요소(next이벤트)도 방출하지 않는 옵저버블 생성
- error 이벤트를 전달하고 종료

```swift
Observable<Void>.error(MyError.error)
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

----- RESULT -----
error(error)
```
<br>

## Reference
--- 
> [http://reactivex.io/](http://reactivex.io/)  
> [https://kxcoding.com/]( https://kxcoding.com/)
