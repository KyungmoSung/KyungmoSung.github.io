---
layout: post
title:  "[RxSwift] Observable"
author: Sung Kyungmo
catalog: true
tags: [swift, rxswift, Observable]
comments: true
header-img:   "img/title-bg/rxswift.jpg"
header-mask:  0.6
---
# Observable
Observable은 Observable Sequence, Sequence라고 부르기도 한다.

Observable이 이벤트를 방출하는것을 **emit**한다고 표현한다

## Observable의 생명주기

---

- Observable은 어떤 구성요소(값)를 가지는 **`next`** 이벤트를 방출(Emission)할 수 있다.
- Observable은 **`error`** 이벤트를 방출(Notification)하여 완전 종료될 수 있다.
- Observable은 **`complete`** 이벤트를 방출(Notification)하여 완전 종료 될 수 있다.
- **`error`, `complete`**는 가장 마지막에 전달된다.
```swift
public enum Event<Element> {
   /// Next elemet is produced.
   case next(Element)
   
   /// Sequence terminated with an error.
   case error(Swift.Error)
   
   /// Sequence completed successfully.
   case completed
}
```
## Observable 생성

---

- **`just`:** 하나의 단일이벤트를 방출하는 옵져버블 생성

```swift
let observable = Observable.just(1)

// next(1)
// completed
```

- **`of`:**  순차적으로 이벤트를 방출하는 옵져버블 생성

```swift
let observable = Observable.of(1, 2, 3)

// next(1)
// next(2)
// next(3)
// completed
```
```swift
let observable = Observable.of([1, 2, 3])

// next([1, 2, 3])
// completed
```

- **`from`:** 배열 요소들을 순차적으로 방출하는 옵져버블 생성

```swift
let observable = Observable.from([1, 2, 3])

// next(1)
// next(2)
// next(3)
// completed
```

- **`create`**는 직접 onNext, onComplete, onError를 직접 호출해야 함
<br>

## Observable 구독

---

Observable은 정의일뿐 구독자(subscriber)가 구독(`subscribe`)을 하기 전까지는 아무런 이벤트도 전달하지 않는다
```swift
observable.subscribe(onNext: { (element) in
    print(element)
})
```

## Disposing과 종료

---

`Completed`, `Error`로 종료되면 리소스가 자동 해제된다.

직접 리소스를 해제시키고 종료하려면 `dispose()`를 호출하면 된다.

`DisposedBag`을 사용하여 여러개의 Disposable을 한꺼번에 관리하고 할당해제할 수 있다.
```swift
var disposeBag = DisposeBag()

Observable.from([1, 2, 3])
    .subscribe {
        print($0)
    }
    .disposed(by:disposeBag)
```
<br>

## Reference
--- 
> [http://reactivex.io/](http://reactivex.io/)  
> [https://kxcoding.com/]( https://kxcoding.com/)
