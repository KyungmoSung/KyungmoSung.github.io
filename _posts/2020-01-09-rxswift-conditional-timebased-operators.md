---
layout: post
title:  "[RxSwift] Conditional, Time based Operators"
author: Sung Kyungmo
catalog: true
tags: [ios, swift, rxswift, operators]
comments: true
header-img:   "img/title-bg/rxswift.jpg"
header-mask:  0.6
---
# Conditional Operators

## amb

- 여러 옵저버블 중에서 가장 먼저 이벤트를 방출하는 옵저버블을 선택
- 여러 서버로 데이터를 요청하고 가장 빠른 응답을 처리하는 패턴 구현 가능


![/img/conditional-timebased-operators/amb.png](/img/conditional-timebased-operators/amb.png)

```swift
let a = PublishSubject<String>()
let b = PublishSubject<String>()
let c = PublishSubject<String>()

Observable.amb([a, b, c])
    .subscribe{ print($0) }
    .disposed(by: bag)

a.onNext("A")
b.onNext("B")

b.onCompleted() //무시됨
a.onCompleted()

----- RESULT -----
next(A)
completed
```

# Time-based Opertators

## interval

- 지정된 주기마다 정수를 방출
- 지속적으로(무한) 발행
- 중지하려면 직접 dispose 호출

![/img/conditional-timebased-operators/Interval.png](/img/conditional-timebased-operators/Interval.png)

```swift
let i = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)

let subscription1 = i.subscribe{ print("1 >> \($0)") }

DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
    subscription1.dispose()
}

var subscription2: Disposable?

DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
    subscription2 = i.subscribe{ print("2 >> \($0)") }
}

DispatchQueue.main.asyncAfter(deadline: .now() + 7) {
    subscription2?.dispose()
}


----- RESULT -----
1 >> next(0)
1 >> next(1)
1 >> next(2)
2 >> next(0)
1 >> next(3)
2 >> next(1)
1 >> next(4)
2 >> next(2)
2 >> next(3)
```

## intervalRange

- intervar + range
- 일정 시간 간격으로 m개만큼 값 발행

![/img/conditional-timebased-operators/intervalRange.png](/img/conditional-timebased-operators/intervalRange.png)

## timer

- 지연 시간과 반복 주기를 지정해서 정수를 방출
- 중지하려면 직접 dispose 호출

![/img/conditional-timebased-operators/Timer.png](/img/conditional-timebased-operators/Timer.png)

```swift
Observable<Int>.timer(.seconds(3), period: .seconds(1), scheduler: MainScheduler.instance)
    .subscribe{ print($0) }
    .disposed(by: bag)


----- RESULT -----
next(0)
next(1)
next(2)
next(3)
next(4)
next(5)
...
```

## timeout

- 지정된 시간 이내에 요소를 방출하지 않으면 에러 이벤트를 전달

![/img/conditional-timebased-operators/timeout.png](/img/conditional-timebased-operators/timeout.png)

```swift
let subject = PublishSubject<Int>()

// 에러 방출 error(Sequence timeout.)
//subject.timeout(.seconds(3), scheduler: MainScheduler.instance)
//    .subscribe{ print($0) }
//    .disposed(by: bag)

// 에러 대신 다른 옵저버블 방출
subject.timeout(.seconds(3), other:Observable.just(1), scheduler: MainScheduler.instance)
    .subscribe{ print($0) }
    .disposed(by: bag)


Observable<Int>.timer(.seconds(2), period: .seconds(5), scheduler: MainScheduler.instance)
    .subscribe(onNext: { subject.onNext($0) })
    .disposed(by: bag)


----- RESULT -----
next(0)
next(1)
completed
```

## delay

- Next 이벤트가 전달되는 시점을 지연시킴
- 구독시점을 지연시키지는 않음

![/img/conditional-timebased-operators/delay.png](/img/conditional-timebased-operators/delay.png)

```swift
Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
    .take(6)
    .debug()
    .delay(.seconds(3), scheduler: MainScheduler.instance)
    .subscribe { print($0) }
    .disposed(by: bag)


----- RESULT -----
2020-01-08 16:16:00.366: delay.playground:36 (__lldb_expr_9) -> subscribed
2020-01-08 16:16:01.379: delay.playground:36 (__lldb_expr_9) -> Event next(0)
2020-01-08 16:16:02.380: delay.playground:36 (__lldb_expr_9) -> Event next(1)
2020-01-08 16:16:03.379: delay.playground:36 (__lldb_expr_9) -> Event next(2)
2020-01-08 16:16:04.379: delay.playground:36 (__lldb_expr_9) -> Event next(3)
next(0)
2020-01-08 16:16:05.379: delay.playground:36 (__lldb_expr_9) -> Event next(4)
next(1)
2020-01-08 16:16:06.379: delay.playground:36 (__lldb_expr_9) -> Event next(5)
2020-01-08 16:16:06.379: delay.playground:36 (__lldb_expr_9) -> Event completed
2020-01-08 16:16:06.379: delay.playground:36 (__lldb_expr_9) -> isDisposed
next(2)
next(3)
next(4)
next(5)
completed
```

## delaySubscription

- 구독이 시작되는 시점을 지연시킴
- Next 이벤트는 지연시키지 않고 바로 전달

![/img/conditional-timebased-operators/delaySubscription.png](/img/conditional-timebased-operators/delaySubscription.png)

```swift
Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
    .take(6)
    .debug()
    .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
    .subscribe { print($0) }
    .disposed(by: bag)


----- RESULT -----
2020-01-08 16:18:14.426: delaySubscription.playground:36 (__lldb_expr_11) -> subscribed
2020-01-08 16:18:15.429: delaySubscription.playground:36 (__lldb_expr_11) -> Event next(0)
next(0)
2020-01-08 16:18:16.429: delaySubscription.playground:36 (__lldb_expr_11) -> Event next(1)
next(1)
2020-01-08 16:18:17.429: delaySubscription.playground:36 (__lldb_expr_11) -> Event next(2)
next(2)
2020-01-08 16:18:18.429: delaySubscription.playground:36 (__lldb_expr_11) -> Event next(3)
next(3)
2020-01-08 16:18:19.429: delaySubscription.playground:36 (__lldb_expr_11) -> Event next(4)
next(4)
2020-01-08 16:18:20.429: delaySubscription.playground:36 (__lldb_expr_11) -> Event next(5)
next(5)
2020-01-08 16:18:20.429: delaySubscription.playground:36 (__lldb_expr_11) -> Event completed
completed
2020-01-08 16:18:20.429: delaySubscription.playground:36 (__lldb_expr_11) -> isDisposed
```

## Reference
--- 
> [http://reactivex.io/](http://reactivex.io/)  
> [https://kxcoding.com/]( https://kxcoding.com/)
