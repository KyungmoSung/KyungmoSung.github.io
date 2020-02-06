---
layout: post
title:  "[RxSwift] Scheduler"
author: Sung Kyungmo
catalog: true
tags: [ios, swift, rxswift, scheduler]
comments: true
header-img:   "img/title-bg/rxswift.jpg"
header-mask:  0.6
---
# Scheduler

스케줄러는 특정코드가 실행되는 컨텍스트를 추상화한것

컨텍스트는 로우레벨 Thread가 될 수도 잇고 Dispatch Queue, Operation Queue가 될 수도 있음

추상화된 컨텍스트이기 때문에 쓰레드와 1대1로 매칭되지 않음


> Cocoa → GCD  
> RxSwift → Scheduler


## subscribeOn

- 시퀀시가 시작될때 어느 스케줄러에서 실행될지 지정(최상위부터 적용)

## observeOn

- 다음에 오는 작업들이 어느 스케줄러에서 실행될지 지정(하위부터 적용)

![/img/scheduler/scheduler.png](/img/scheduler/scheduler.png)

```swift
let backgroundScheduler = ConcurrentDispatchQueueScheduler(queue: DispatchQueue.global())

Observable.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
    .subscribeOn(MainScheduler.instance)
    .filter { num -> Bool in
        print(Thread.isMainThread ? "Main" : "Background", ">> filter")
        return num.isMultiple(of: 2)
    }
    .observeOn(backgroundScheduler)
    .map { num -> Int in
        print(Thread.isMainThread ? "Main" : "Background", ">> map")
        return num * 2
    }
    .observeOn(MainScheduler.instance)
    .subscribe {
        print(Thread.isMainThread ? "Main" : "Background", ">> subscribe",$0)
    }
    .disposed(by: bag)

----- RESULT -----
Main >> filter
Main >> filter
Background >> map
Main >> filter
Main >> filter
Main >> filter
Main >> filter
Main >> filter
Main >> filter
Main >> filter
Background >> map
Background >> map
Background >> map
Main >> subscribe next(4)
Main >> subscribe next(8)
Main >> subscribe next(12)
Main >> subscribe next(16)
Main >> subscribe completed
```

# 내장 스케줄러(Builtin schedulers)

## CurrentThreadScheduler (Serial scheduler)

- 현재 스레드에 있는 작업의 단위들을 스케쥴해줍니다. 이것은 요소를 생성하는 연산자의 기본 스케줄러입니다.
- 만약 `CurrentThreadScheduler.instance.schedule(state) { }가` 어떤 스레드에서 처음으로 호출 된 경우, 예약 된 작업이 즉시 실행되고 재귀 적으로 예약 된 모든 작업이 일시적으로 대기하는 숨겨진 큐가 생성됩니다.
- 만약 콜 스택의 몇몇 부모 프레임이 이미 `CurrentThreadScheduler.instance.schedule(state) { }`를 실행중이라면, 예약 된 작업은 현재 실행중인 작업과 이전에 대기열에 추가 된 모든 작업이 완료되면 대기열에 추가되고 실행됩니다.

## MainScheduler (Serial scheduler)

- `MainThread`에서 실행되어야 하는 추상적인 작업에서 사용합니다. `schedule`메소드가 메인 스레드에서 호출된 경우에, 스케줄링 없이 작업을 실행합니다.
- 이 스케줄러는 보통 UI 작업에서 쓰입니다.

## SerialDispatchQueueScheduler (Serial scheduler)

- 특정 `dispatch_queue_t`에서 실행되어야 하는 추상적인 작업에서 사용합니다. 컨커런트 디스패치 큐에 전달된 경우에도 시리얼 디스패치 큐로 변환됩니다.
- 시리얼 스케줄러는 `observeOn`를 위한 특정 최적화를 가능하게 해줍니다.
- 메인 스케줄러는 `SerialDispatchQueueScheduler`의 인스턴스입니다.

## ConcurrentDispatchQueueScheduler (Concurrent scheduler)

- 특정 `dispatch_queue_t`에서 실행되어야 하는 추상적인 작업에서 사용합니다. 시리얼 디스패치 큐에도 보낼 수 있으며 아무런 문제가 발생하지 않습니다.
- 이 스케줄러는 백그라운드에서 작업이 실행되어야 할 때 적합합니다.

## OperationQueueScheduler (Concurrent scheduler)

- 특정 `NSOperationQueue`에서 실행되어야 하는 추상적인 작업에서 사용합니다.
- 이 스케줄러는 백그라운드에서 수행해야하는 큰 작업이 있고 `maxConcurrentOperationCount`를 이용해서 컨커런트 처리 과정을 미세 조정하려는 경우에 적합합니다.
- 이 스케줄러는 백그라운드에서 수행해야하는 더 큰 작업 청크가 있고 maxConcurrentOperationCount를 사용하여 동시 처리를 미세 조정하려는 경우에 적합합니다.

## Reference
--- 
> [http://reactivex.io/](http://reactivex.io/)  
> [https://kxcoding.com/]( https://kxcoding.com/)  
> [https://pilgwon.github.io/blog/2018/05/31/RxSwift-Schedulers.html](https://pilgwon.github.io/blog/2018/05/31/RxSwift-Schedulers.html)  
> [https://github.com/ReactiveX/RxSwift/blob/master/Documentation/Schedulers.md#builtin-schedulers](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/Schedulers.md#builtin-schedulers)

