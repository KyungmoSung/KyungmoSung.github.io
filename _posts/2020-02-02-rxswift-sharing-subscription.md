---
layout: post
title:  "[RxSwift] Sharing Subscription"
author: Sung Kyungmo
catalog: true
tags: [ios, swift, rxswift, operators]
comments: true
header-img:   "img/title-bg/rxswift.jpg"
header-mask:  0.6
---
# Sharing Subscription
구독 공유를 통해 불필요한 중복 작업을 방지

네트워크 요청, DB접근, 파일 읽기 등등...

모든 구독자가 하나의 구독을 공유하도록 구현

## multicast

- subject를 파라미터로 받음
- 이벤트는 구독자에게 전달되는것이 아니라 subject로 전달됨
- 전달받은 이벤트를 다수의 구독자에게 전달함
- 유니캐스트방식으로 동작하는 옵져버블을 멀티캐스트방식으로 바꿔줌
- ConnectableObservable을 리턴
- 구독자가 추가되어도 시퀀스가 시작되지 않음
- connect 매소드를 호출하는 시점에 시퀀스 시작
- 모든 구독자가 등록된 이후에 하나의 시퀀시를 시작하는 패턴
- subject를 직접 만들고 connect 메소드를 직접 호출해야되기 때문에 번거로움

```swift
let subject = PublishSubject<Int>()

let source = Observable<Int>
  .interval(.seconds(1), scheduler: MainScheduler.instance)
  .take(5)
  .multicast(subject)

source
  .subscribe { print("🔵", $0) }
  .disposed(by: bag)

source
  .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
  .subscribe { print("🔴", $0) } // 구독이 지연된동안 원본 옵저버블이 전달한 2개의 이벤트는 전달되지 않음
  .disposed(by: bag)

source.connect() // 시퀀스가 시작됨

----- RESULT -----
🔵 next(0)
🔵 next(1)
🔵 next(2)
🔴 next(2)
🔵 next(3)
🔴 next(3)
🔵 next(4)
🔴 next(4)
🔵 completed
🔴 completed
```

## publish

- PublishSubject를 자동으로 생성해주는 것을 제외하면 multicast와 동일함

![/img/sharing-subscription/publish.png](/img/sharing-subscription/publish.png)

```swift
let source = Observable<Int>
  .interval(.seconds(1), scheduler: MainScheduler.instance)
  .take(5)
  .publish()

source
  .subscribe { print("🔵", $0) }
  .disposed(by: bag)

source
  .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
  .subscribe { print("🔴", $0) }
  .disposed(by: bag)

source.connect()

----- RESULT -----
🔵 next(0)
🔵 next(1)
🔵 next(2)
🔴 next(2)
🔵 next(3)
🔴 next(3)
🔵 next(4)
🔴 next(4)
🔵 completed
🔴 completed
```

## replay

- 이후에 구독자에게 이전에 전달되었던 이벤트도 전달해야할 경우
- PublishSubject 대신 ReplaySubject로 만들면 됨
- ReplaySubject를 자동으로 생성해주는 replay 연산자를 사용하면 더 간결해짐

![/img/sharing-subscription/replay.png](/img/sharing-subscription/replay.png)

```swift
let source = Observable<Int>
  .interval(.seconds(1), scheduler: MainScheduler.instance)
  .take(5)
  .replay(5)

source
  .subscribe { print("🔵", $0) }
  .disposed(by: bag)

source
  .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
  .subscribe { print("🔴", $0) }
  .disposed(by: bag)

source.connect()

----- RESULT -----
🔵 next(0)
🔵 next(1)
🔴 next(0)
🔴 next(1)
🔵 next(2)
🔴 next(2)
🔵 next(3)
🔴 next(3)
🔵 next(4)
🔴 next(4)
🔵 completed
🔴 completed
```

## refCount

- ConnectableObservable 익스텐션에 구현되어있기 때문에 일반 Observable에서는 사용할 수 없음
- 내부에서 connect를 자동으로 호출함
- 다른 연산자는 ConnectableObservable을 직접 관리 해야하지만 (connect, dispose, take...) refCount는 자동으로 처리되기 때문에 더 간편함

![/img/sharing-subscription/refCount.png](/img/sharing-subscription/refCount.png)

```swift
let source = Observable<Int>
  .interval(.seconds(1), scheduler: MainScheduler.instance)
  .debug()
  .publish()
  .refCount()

let observer1 = source
   .subscribe { print("🔵", $0) } // connect

DispatchQueue.main.asyncAfter(deadline: .now() + 3) { // 3초 후 구독 취소
   observer1.dispose() // 다른 구독자가 없기때문에 disconnect
}

DispatchQueue.main.asyncAfter(deadline: .now() + 7) { // 7초 후 구독 시작
  let observer2 = source.subscribe { print("🔴", $0) } // connect

   DispatchQueue.main.asyncAfter(deadline: .now() + 3) { // 3초 후 구독 취소
      observer2.dispose() // disconnect
   }
}

----- RESULT -----
2020-02-02 18:43:54.433: refCount.playground:37 (__lldb_expr_35) -> subscribed
2020-02-02 18:43:55.436: refCount.playground:37 (__lldb_expr_35) -> Event next(0)
🔵 next(0)
2020-02-02 18:43:56.436: refCount.playground:37 (__lldb_expr_35) -> Event next(1)
🔵 next(1)
2020-02-02 18:43:57.436: refCount.playground:37 (__lldb_expr_35) -> Event next(2)
🔵 next(2)
2020-02-02 18:43:57.531: refCount.playground:37 (__lldb_expr_35) -> isDisposed
2020-02-02 18:44:01.438: refCount.playground:37 (__lldb_expr_35) -> subscribed
2020-02-02 18:44:02.440: refCount.playground:37 (__lldb_expr_35) -> Event next(0)
🔴 next(0)
2020-02-02 18:44:03.439: refCount.playground:37 (__lldb_expr_35) -> Event next(1)
🔴 next(1)
2020-02-02 18:44:04.440: refCount.playground:37 (__lldb_expr_35) -> Event next(2)
🔴 next(2)
2020-02-02 18:44:04.440: refCount.playground:37 (__lldb_expr_35) -> isDisposed
```

## share

- share연산자가 리턴하는 옵저버블은 refCount옵저버블
- replay: 버퍼 사이즈
- whileConnected: 새로운 구독자(커넥션)가 추가되면 서브젝트를 생성하고 이어지는 구독자는 이 서브젝트를 공유, 커넥션이 종료되면 서브젝트는 사라지고 커넥션마다 새로운 서브젝트가 생성됨 → 커넥션은 다른 커넥션과 격리되어있음(isolated)
- forever: 모든 구독자(커넥션)이 하나의 서브젝트를 공유함

```swift
let source = Observable<Int>
  .interval(.seconds(1), scheduler: MainScheduler.instance)
  .debug()
  .share() //replay: 0, scope: whileConnected
//.share(replay: 5)
//.share(replay: 5, scope: .forever)

let observer1 = source
   .subscribe { print("🔵", $0) }

let observer2 = source
   .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
   .subscribe { print("🔴", $0) }

DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
   observer1.dispose()
   observer2.dispose()
  //disconnect
}

DispatchQueue.main.asyncAfter(deadline: .now() + 7) {
   let observer3 = source.subscribe { print("⚫️", $0) }

   DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
      observer3.dispose()
   }
}


----- RESULT ----- (replay: 0, scope: .whileConnected)
-> subscribed
-> Event next(0)
🔵 next(0)
-> Event next(1)
🔵 next(1)
-> Event next(2)
🔵 next(2)
-> Event next(3)
🔵 next(3)
🔴 next(3)
-> Event next(4)
🔵 next(4)
🔴 next(4)
-> isDisposed
-> subscribed
-> Event next(0)
⚫️ next(0)
-> Event next(1)
⚫️ next(1)
-> Event next(2)
⚫️ next(2)
-> isDisposed


----- RESULT -----(replay: 5, scope: .whileConnected)
-> subscribed
-> Event next(0)
🔵 next(0)
-> Event next(1)
🔵 next(1)
-> Event next(2)
🔵 next(2)
🔴 next(0)
🔴 next(1)
🔴 next(2)
-> Event next(3)
🔵 next(3)
🔴 next(3)
-> Event next(4)
🔵 next(4)
🔴 next(4)
-> isDisposed
-> subscribed
-> Event next(0)
⚫️ next(0)
-> Event next(1)
⚫️ next(1)
-> Event next(2)
⚫️ next(2)
-> isDisposed


----- RESULT -----(replay: 5, scope: .forever)
-> subscribed
-> Event next(0)
🔵 next(0)
-> Event next(1)
🔵 next(1)
-> Event next(2)
🔵 next(2)
🔴 next(0)
🔴 next(1)
🔴 next(2)
-> Event next(3)
🔵 next(3)
🔴 next(3)
-> Event next(4)
🔵 next(4)
🔴 next(4)
-> isDisposed
⚫️ next(0)
⚫️ next(1)
⚫️ next(2)
⚫️ next(3)
⚫️ next(4)
-> subscribed
-> Event next(0)
⚫️ next(0)
-> Event next(1)
⚫️ next(1)
-> Event next(2)
⚫️ next(2)
-> isDisposed
```

## Reference
--- 
> [http://reactivex.io/](http://reactivex.io/)  
> [https://kxcoding.com/]( https://kxcoding.com/)
