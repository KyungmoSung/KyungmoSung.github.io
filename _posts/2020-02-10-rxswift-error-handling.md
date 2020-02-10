---
layout: post
title:  "[RxSwift] Error Handling"
author: Sung Kyungmo
catalog: true
tags: [ios, swift, rxswift, error, operator]
comments: true
header-img:   "img/title-bg/rxswift.jpg"
header-mask:  0.6
---
# Error Handling

## catchError

- next이벤트와 completed이벤트는 구독자에게 그대로 전달
- error는 전달하지 않고 새로운 옵져버블로 교체
- 발생한 에러의 종류와 관계없이 항상 동일값이 리턴됨
- 네트워크 요청을 구현할때 많이 사용함

![/img/error-handling/catchError.png](/img/error-handling/catchError.png)

```swift
let subject = PublishSubject<Int>()
let recovery = PublishSubject<Int>()

subject
    .catchError { _ in recovery } // 에러 발생시 recovery로 고체
   .subscribe { print($0) }
   .disposed(by: bag)

subject.onError(MyError.error)

subject.onNext(1) // 소스 옵저버블은 새로운 요소를 전달하지 못함

recovery.onNext(2) // 새로운 요소를 방출할 수 있음
recovery.onCompleted()


----- RESULT -----
next(2)
completed
```
## catchErrorJustReturn

- catchError와 동일하지만 옵저버블 대신 기본값을 리턴

![/img/error-handling/catchErrorJustReturn.png](/img/error-handling/catchErrorJustReturn.png)

```swift
let subject = PublishSubject<Int>()

subject
    .catchErrorJustReturn(-1)
    .subscribe { print($0) }
    .disposed(by: bag)

subject.onError(MyError.error)


----- RESULT -----
next(-1)
completed
```

## retry

- 옵저버블에서 에러가 발생하면 구독을 해지하고 새로운 구독을 시작
- 파라미터에 최대 재시도 횟수 + 1(첫시도)

![/img/error-handling/retry.png](/img/error-handling/retry.png)

```swift
var attempts = 1

let source = Observable<Int>.create { observer in
  let currentAttempts = attempts
  print("#\(currentAttempts) START")

  if attempts < 3 {
    observer.onError(MyError.error)
    attempts += 1
  }

  observer.onNext(1)
  observer.onNext(2)
  observer.onCompleted()

  return Disposables.create {
    print("#\(currentAttempts) END")
  }
}

source
  .retry(5) // 4번 재시도
  .subscribe { print($0) }
  .disposed(by: bag)


----- RESULT -----
#1 START
#1 END
#2 START
#2 END
#3 START
next(1)
next(2)
completed
#3 END
```

## retryWhen

- 사용자가 재시도 버튼을 탭할때만 재시도를 할 경우
- 클로저를 파라미터로 받고 TriggerObservable 리턴

![/img/error-handling/retryWhen.png](/img/error-handling/retryWhen.png)

```swift
var attempts = 1

let source = Observable<Int>.create { observer in
   let currentAttempts = attempts
   print("START #\(currentAttempts)")
   
   if attempts < 3 {
      observer.onError(MyError.error)
      attempts += 1
   }
   
   observer.onNext(1)
   observer.onNext(2)
   observer.onCompleted()
   
   return Disposables.create {
      print("END #\(currentAttempts)")
   }
}

let trigger = PublishSubject<Void>()

source
  .retryWhen { _ in trigger }
   .subscribe { print($0) }
   .disposed(by: bag)

trigger.onNext(()) // triggerObservable에서 next 이벤트가 전달될때 재시도
trigger.onNext(())


----- RESULT -----
START #1
END #1
START #2
END #2
START #3
next(1)
next(2)
completed
END #3
```

## Reference
--- 
> [http://reactivex.io/](http://reactivex.io/)  
> [https://kxcoding.com/]( https://kxcoding.com/)  
