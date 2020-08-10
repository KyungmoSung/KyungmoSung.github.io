---
layout: post
title:  "[RxSwift] RxSwift Traits"
author: Sung Kyungmo
catalog: true
tags: [ios, swift, rxswift, traits]
comments: true
header-img:   "img/title-bg/rxswift.jpg"
header-mask:  0.6
---

# RxSwift Traits

```swift
protocol GithubType {
  func saveRepo(repo: Repo) -> Observable<Void>
  func findRepo(id: String) -> Observable<Repo?>
  func myInfo() -> Observable<Info>
}
```
```swift
protocol GithubType {
  func saveRepo(repo: Repo) -> Completable
  func findRepo(id: String) -> Maybe<Repo>
  func myInfo() -> Single<Info>
}
```

## Single

- 반환 데이터(1개) 있는 성공(`success`) or 실패(`error`)
- 하나의 결과값이나 에러만 처리하고자 하는 경우 사용

- `Single` 만들기
 ```swift
func getRepo(_ repo: String) -> Single<[String: Any]> {
    return Single<[String: Any]>.create { single in
        let task = URLSession.shared.dataTask(with: URL(string: "https://api.github.com/repos/\(repo)")!) { data, _, error in
            if let error = error {
                single(.error(error))
                return
            }

            guard let data = data,
                  let json = try? JSONSerialization.jsonObject(with: data, options: .mutableLeaves),
                  let result = json as? [String: Any] else {
                single(.error(DataError.cantParseJSON))
                return
            }

            single(.success(result))
        }

        task.resume()

        return Disposables.create { task.cancel() }
    }
}
 ```
- `Single` 사용
     ```swift
    getRepo("ReactiveX/RxSwift")
        .subscribe { event in
            switch event {
                case .success(let json):
                    print("JSON: ", json)
                case .error(let error):
                    print("Error: ", error)
            }
        }
        .disposed(by: disposeBag)
    
    or
    
    getRepo("ReactiveX/RxSwift")
        .subscribe(onSuccess: { json in
                       print("JSON: ", json)
                   },
                   onError: { error in
                       print("Error: ", error)
                   })
        .disposed(by: disposeBag)
    ```

## Completable

- 반환 데이터 없는 성공(`completed`) or 실패(`error`)
- 완료만 의미가 있고, 결과값이 필요하는 않은 경우 사용

- `Completable` 만들기
```swift
func cacheLocally() -> Completable {
    return Completable.create { completable in
       // Store some data locally
       ...
       ...

       guard success else {
           completable(.error(CacheError.failedCaching))
           return Disposables.create {}
       }

       completable(.completed)
       return Disposables.create {}
    }
}
```
- `Completable` 사용
    ```swift
    cacheLocally()
        .subscribe { completable in
            switch completable {
                case .completed:
                    print("Completed with no error")
                case .error(let error):
                    print("Completed with an error: \(error.localizedDescription)")
            }
        }
        .disposed(by: disposeBag)
    
    or
    
    cacheLocally()
        .subscribe(onCompleted: {
                       print("Completed with no error")
                   },
                   onError: { error in
                       print("Completed with an error: \(error.localizedDescription)")
                   })
        .disposed(by: disposeBag)
    ```

## Maybe

- 반환 데이터 없는 성공(`completed`) or 반환 데이터 있는 성공(`success`) or 실패(`error`)
- `Single` + `Completable` 형태의  `Observable`

- `Maybe` 만들기
```swift
func generateString() -> Maybe<String> {
    return Maybe<String>.create { maybe in
        maybe(.success("RxSwift"))

        // OR

        maybe(.completed)

        // OR

        maybe(.error(error))

        return Disposables.create {}
    }
}
```
- `Maybe` 사용
    ```swift
    generateString()
        .subscribe { maybe in
            switch maybe {
                case .success(let element):
                    print("Completed with element \(element)")
                case .completed:
                    print("Completed with no element")
                case .error(let error):
                    print("Completed with an error \(error.localizedDescription)")
            }
        }
        .disposed(by: disposeBag)
    
    or
    
    generateString()
        .subscribe(onSuccess: { element in
                       print("Completed with element \(element)")
                   },
                   onError: { error in
                       print("Completed with an error \(error.localizedDescription)")
                   },
                   onCompleted: {
                       print("Completed with no element")
                   })
        .disposed(by: disposeBag)
    ```

## Reference
--- 
> [http://reactivex.io/](http://reactivex.io/)  
> [https://kxcoding.com/](https://kxcoding.com/)  
> [https://github.com/ReactiveX/RxSwift/blob/master/Documentation/Traits.md](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/Traits.md)  

