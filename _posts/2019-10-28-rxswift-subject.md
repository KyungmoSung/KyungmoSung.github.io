---
layout: post
title:  "[RxSwift] Subject"
author: Sung Kyungmo
catalog: true
tags: [ios, swift, rxswift, subject]
comments: true
header-img:   "img/title-bg/rxswift.jpg"
header-mask:  0.6
---

# Subject
- Observable이자 Observer  
- **Observer** 역할로, 하나 이상의 Observable을 구독하며, **Observable** 역할로 아이템을 내보낼 수 있다.  
- 다른 Observable로 부터 이벤트를 받아 구독자로 전달


## PublishSubject
- 구독된 이후에 발생한 이벤트만 전달
- 해당 시간에 발생한 데이터 방출
- 시간에 민감한 데이터를 모델링 할 때 사용 (subscribe 되기 이전의 값이 필요 없는 경우)

![/img/subject/publishSubject_c.png](/img/subject/publishSubject_c.png)

![/img/subject/publishSubject_e.png](/img/subject/publishSubject_e.png)

```swift
let subject = PublishSubject<String>()

subject.onNext("123") // X

let ob1 = subject.subscribe{ print("1 >> \($0)") }
ob1.disposed(by: disposeBag)

subject.onNext("456") // ob1

let ob2 = subject.subscribe{ print("2 >> \($0)") }
ob2.disposed(by: disposeBag)

subject.onNext("789") // ob1, ob2

//subject.onCompleted() // ob1, ob2
subject.onError(MyError.error)

// Completed 이후에 구독시 바로 Completed이벤트 전달, Error도 똑같음
let ob3 = subject.subscribe{ print("3 >> \($0)") }
ob3.disposed(by: disposeBag)

----- RESULT -----
1 >> next(456)
1 >> next(789)
2 >> next(789)
1 >> error(error)
2 >> error(error)
3 >> error(error)
```

## BehaviorSubject
- 항상 최신의 next 이벤트를 방출하기 때문에 **초기값** 필요
- 마지막 값을 방출
- 항상 최신 데이터로 채워놓아야 하는 경우에 사용 (유저 프로필)

![/img/subject/behaviorSubject_c.png](/img/subject/behaviorSubject_c.png)

![/img/subject/behaviorSubject_e.png](/img/subject/behaviorSubject_e.png)

```swift
let pSub = PublishSubject<Int>() // 초기값 없음
// Subject에 이벤트가 전달되기 전까지 구독자에게 이벤트가 전달되지 않음
pSub.subscribe{ print("PublishSubject >> ", $0) }
    .disposed(by: disposeBag)

let bSub = BehaviorSubject<Int>(value: 0) // 초기값을 가지고 있음
// 생성과 함께 내부에 Next이벤트가 만들어짐, 구독과함께 이벤트 전달
bSub.subscribe{ print("BehaviorSubject >> ",$0) }
    .disposed(by: disposeBag)

bSub.onNext(1)

// 새로운 next이벤트가 전달되면 기존의 이벤트를 교체 -> 최신의 Next이벤트를 옵저버에게 전달
bSub.subscribe{ print("BehaviorSubject2 >> ",$0) }
    .disposed(by: disposeBag)

//bSub.onCompleted()
bSub.onError(MyError.error)

// 마지막이벤트인 completed,error만 전달
bSub.subscribe{ print("BehaviorSubject3 >> ",$0) }
    .disposed(by: disposeBag)

----- RESULT -----
BehaviorSubject >>  next(0)
BehaviorSubject >>  next(1)
BehaviorSubject2 >>  next(1)
BehaviorSubject >>  error(error)
BehaviorSubject2 >>  error(error)
BehaviorSubject3 >>  error(error)
```

## ReplaySubject
- **bufferSize** 만큼 이벤트를 저장해 subscribe 될 때 저장된 이벤트를 모두 방출한다
- 2개 이상의 이벤트를 저장하고 옵저버에게 전달하고 싶을때 사용
- 최신의 여러 값들을 보여주고 싶을 때 사용 (최근 검색어)

![/img/subject/replaySubject.png](/img/subject/replaySubject.png)

```swift
let rSub = ReplaySubject<Int>.create(bufferSize: 3)

(1...10).forEach{ rSub.onNext($0) }

// 버퍼의 크기만큼 이벤트를 저장했다가 전달
rSub.subscribe{ print("Observer 1 >> ", $0) }
    .disposed(by: disposeBag)

rSub.subscribe{ print("Observer 2 >> ", $0) }
    .disposed(by: disposeBag)

// 가장 마지막 이벤트가 삭제되고 새로운 이벤트가 저장됨
rSub.onNext(11)

// 구독을 시작할때 버퍼 크긴만큼 이벤트 전달
rSub.subscribe{ print("Observer 3 >> ", $0) }
    .disposed(by: disposeBag)

rSub.onCompleted()
//rSub.onError(MyError.error)

// 이벤트 종료(Completed,error)와 상관없이 버퍼크기만큼의 이벤트 전달
// next이벤트 buffer개 + 완료이벤트(Completed,error) = 3 + 1 = 4
rSub.subscribe{ print("Observer 4 >> ", $0) }
    .disposed(by: disposeBag)

----- RESULT -----
Observer 1 >>  next(8)
Observer 1 >>  next(9)
Observer 1 >>  next(10)

Observer 2 >>  next(8)
Observer 2 >>  next(9)
Observer 2 >>  next(10)

Observer 1 >>  next(11)
Observer 2 >>  next(11)

Observer 3 >>  next(9)
Observer 3 >>  next(10)
Observer 3 >>  next(11)

Observer 1 >>  completed
Observer 2 >>  completed
Observer 3 >>  completed

Observer 4 >>  next(9)
Observer 4 >>  next(10)
Observer 4 >>  next(11)
Observer 4 >>  completed
```
## AsyncSubject
- 이전 subject들은 subject로 이벤트가 전달되면 구독자에게 바로 전달하지만 AsyncSubject는 Completed가 발생하면 마지막 next이벤트 하나를 구독자에게 전달

![/img/subject/asyncSubject_c.png](/img/subject/asyncSubject_c.png)

![/img/subject/asyncSubject_e.png](/img/subject/asyncSubject_e.png)

```swift
let subject = AsyncSubject<Int>()

subject
    .subscribe{ print($0) }
    .disposed(by: bag)

subject.onNext(1)
subject.onNext(2)
subject.onNext(3)
subject.onCompleted() // completed 기준으로 가장 최근의 next 전달
//subject.onError(MyError.error) // error발생시 최근의 next이벤트는 전달하지 않음 error만 전달

----- RESULT -----
next(3)
completed
```
## Relay
- Relay는 Subject를 랩핑하고 있음
- RxSwift 프레임워크가 아닌 RxCocoa 에서 제공
- next 이벤트만 전달
- value를 변경하기 위해서 `onNext` 대신 `accept`를 사용한다
- Subject와 다르게 `error` 나 `complete` 를 통해서 완전종료될 수 없다
- 구독자가 disposed 되기 전까지 이벤트 처리
- subscribe 하고 싶을 때는 `asObservable`을 사용한다.
- 주로 UI에 사용

```swift
let pRelay = PublishRelay<Int>()
pRelay.subscribe {
    print("1: \($0)")
}
.disposed(by: bag)

pRelay.accept(1) //onNext 대신에 accept 사용

let bRelay = BehaviorRelay(value: 1)
bRelay.accept(2)

bRelay.subscribe {
    print("2: \($0)")
}
.disposed(by: bag)

bRelay.accept(3)

print(bRelay.value) // get-only

----- RESULT -----
1: next(1)
2: next(2)
2: next(3)
3
```
## PublishRelay
- UI Event에서 사용하기 적절

## BehaviorRelay
- `.value`를 사용해 현재의 값을 꺼낼 수 있다. (get-only-property)
<br>

## Reference
--- 
> [http://reactivex.io/](http://reactivex.io/)  
> [https://kxcoding.com/]( https://kxcoding.com/)
