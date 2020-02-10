---
layout: post
title:  "[RxSwift] Combining Operators"
author: Sung Kyungmo
catalog: true
tags: [ios, swift, rxswift, operator]
comments: true
header-img:   "img/title-bg/rxswift.jpg"
header-mask:  0.6
---
# 결합 연산자
---

## startWith

- 옵저버블이 요소를 방출하기 전에 다른 항목을 앞에 추가
- 주로 기본값이나 시작값을 지정할때 활용

![startWith](/img/combining-operators/startWith.png)

```swift
let numbers = [1, 2, 3, 4, 5]

Observable.from(numbers)
    .startWith(0)
    .startWith(-1)
    .subscribe { print($0) }
    .disposed(by: bag)

----- RESULT -----
next(-1)
next(0)
next(1)
next(2)
next(3)
next(4)
next(5)
completed
```
## concat

- 두개의 옵저버블을 연결할때 사용
- 타입메소드와 인스턴스메소드로 구현되어 있음


![concat](/img/combining-operators/concat.png)

```swift
let fruits = Observable.from(["🍏", "🍎"])
let animals = Observable.from(["🐶", "🐱"])

// 타입 메소드
Observable.concat([fruits, animals])
    .subscribe{ print($0) }
    .disposed(by: bag)

// 인스턴스 메소드
fruits.concat(animals)
    .subscribe{ print($0) }
    .disposed(by: bag)

----- RESULT -----
next(🍏)
next(🍎)
next(🐶)
next(🐱)
completed
next(🍏)
next(🍎)
next(🐶)
next(🐱)
completed
```
## merge

- 여러 옵저버블이 배출하는 항목들을 하나의 옵저버블에서 방출하도록 병합
- concat과 동작방식이 다름
- concat: 하나의 옵저법블이 모든요소를 방출하고 Completed을 방출하면 이어지는 옵저버블을 연결
- merge: 두개이상의 옵저버블을 병합하고 모든 옵저버블에서 방출하는 요소들을 순서대로 방출하는 옵저버블을 리턴

![merge](/img/combining-operators/merge.png)

```swift
let oddNumbers = BehaviorSubject(value: 1)
let evenNumbers = BehaviorSubject(value: 2)
let negativeNumbers = BehaviorSubject(value: -1)

let source = Observable.of(oddNumbers, evenNumbers)

// 단순히 뒤에 연결하는것이 아니라 하나의 옵저버블로 합쳐줌
source
    .merge()
    .subscribe{ print($0) }
    .disposed(by: bag)

oddNumbers.onNext(3)
evenNumbers.onNext(4)

evenNumbers.onNext(6)
oddNumbers.onNext(5)

oddNumbers.onCompleted() //병합한 모든 옵저버블이 Completed되면 구독자에게 Completed전달
//oddNumbers.onError(MyError.error) //Error 전달, 이후 이벤트 받지 않음

evenNumbers.onNext(7)
evenNumbers.onCompleted()

----- RESULT -----
next(1)
next(2)
next(3)
next(4)
next(6)
next(5)
next(7)
completed
```
## combineLatest

- 소스 옵저버블을 결합
- 파라미터로 전달한 함수를 실행하고 결과를 방출하는 새로운 옵저버블을 리턴

![combineLatest](/img/combining-operators/combineLatest.png)

```swift
let greetings = PublishSubject<String>()
let languages = PublishSubject<String>()

Observable.combineLatest(greetings, languages) { lhs, rhs -> String in
    return "\(lhs) \(rhs)"
}
    .subscribe{ print($0) }
    .disposed(by: bag)

greetings.onNext("Hi")
languages.onNext("World!")

greetings.onNext("Hello")
languages.onNext("RxSwift")

greetings.onCompleted()

languages.onNext("SwiftUI")

languages.onCompleted()

----- RESULT -----
next(Hi World!)
next(Hello World!)
next(Hello RxSwift)
next(Hello SwiftUI)
completed
```
## zip

- 옵저버블을 결합
- 클로저에게 중복된 요소를 전달하지 않음
- 인덱스를 기준으로 짝을 일치시켜 전달(첫번째끼리, 두번째끼리 결합)

![zip](/img/combining-operators/zip.png)

```swift
let numbers = PublishSubject<Int>()
let strings = PublishSubject<String>()

Observable.zip(numbers, strings) { "\($0) - \($1)" }
    .subscribe { print($0) }
    .disposed(by: bag)

numbers.onNext(1)
strings.onNext("one")

numbers.onNext(2)
numbers.onNext(3)
strings.onNext("two")

numbers.onCompleted()

strings.onNext("three")

strings.onCompleted()

----- RESULT -----
next(1 - one)
next(2 - two)
next(3 - three)
completed
```
## withLatestFrom

- 트리거 옵저버블이 Next 이벤트를 방출하면 데이터 옵저버블이 가장 최근에 방출한 Next 이벤트를 구독자에게 전달
- ex)회원가입 버튼을 탭하는 시점에 텍스트필드에 들어가있는 값을 가져올때

![withLatestFrom](/img/combining-operators/withLatestFrom.png)

```swift
let trigger = PublishSubject<Void>()
let data = PublishSubject<String>()

trigger.withLatestFrom(data)
    .subscribe { print($0) }
    .disposed(by: bag)

data.onNext("Hello")
trigger.onNext(())
trigger.onNext(())

data.onCompleted()
trigger.onNext(())

trigger.onCompleted()

----- RESULT -----
next(Hello)
next(Hello)
next(Hello)
completed
```
## sample

- 트리거 옵저버블이 Next 이벤트를 전달할 때마다 데이터 옵저버블이 Next 이벤트를 방출하지만, 동일한 Next 이벤트를 반복해서 방출하지 않음

![sample](/img/combining-operators/sample.png)

```swift
let trigger = PublishSubject<Void>()
let data = PublishSubject<String>()

data.sample(trigger)
    .subscribe{ print($0) }
    .disposed(by: bag)

trigger.onNext(())
data.onNext("Hello")

trigger.onNext(())
trigger.onNext(()) //중복

data.onCompleted()
trigger.onNext(())

----- RESULT -----
next(Hello)
completed
```
## switchLatest

- 가장 최근에 방출된 옵저버블을 구독하고, 이 옵저버블이 전달하는 이벤트를 구독자에게 전달

![switchLatest](/img/combining-operators/switchLatest.png)

```swift
let a = PublishSubject<String>()
let b = PublishSubject<String>()

let source = PublishSubject<Observable<String>>()

source
    .switchLatest()
    .subscribe{ print($0) }
    .disposed(by: bag)

a.onNext("a")
b.onNext("b")

source.onNext(a)

a.onNext("aa")
b.onNext("bb")

source.onNext(b)

a.onNext("aaa")
b.onNext("bbb")

//a.onCompleted() //completed 전달 X
//b.onCompleted() //completed 전달 X
//source.onCompleted()

a.onError(MyError.error) //error 전달 X
b.onError(MyError.error) //error 전달

----- RESULT -----
next(aa)
next(bbb)
error(error)
```
## reduce

- 시드 값과 옵저버블이 방출하는 요소를 대상으로 클로저를 실행하고 최종 결과를 옵저버블로 방출
- 중간결과와 최종결과가 모두 필요하면 **scan** 연산자
- 최종결과만 필요하면 **reduce** 연산자

![reduce](/img/combining-operators/reduce.png)

```swift
let o = Observable.range(start: 1, count: 5)

print("== scan")

o.scan(0, accumulator: +)
   .subscribe { print($0) }
   .disposed(by: bag)

print("== reduce")

o.reduce(0, accumulator: +)
    .subscribe{ print($0) }
    .disposed(by: bag)

----- RESULT -----
== scan
next(1)
next(3)
next(6)
next(10)
next(15)
completed
== reduce
next(15)
completed
```

## Reference
--- 
> [http://reactivex.io/](http://reactivex.io/)  
> [https://kxcoding.com/]( https://kxcoding.com/)
