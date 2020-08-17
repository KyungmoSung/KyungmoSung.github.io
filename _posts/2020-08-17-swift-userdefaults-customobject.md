---
layout: post
title:  "[Swift] UserDefaults: Custom Object(Model) 저장하기"
author: Sung Kyungmo
catalog: true
tags: [ios, swift, custom object, model, userdefaults, save, object]
comments: true
---
# UserDefaults에 Custom Object(Model) 저장하기

### Sample model 생성

```swift
struct Genre: Codable {
    let id: Int!
    let name: String!
}
```

`UserDefaults`에 저장할 샘플 데이터를 만들겠습니다.

`genre`와 여러개의 `Genre`를 가지는 `genres` `array` 두가지 샘플을 생성합니다.

```swift
var genre = Genre(id: 1, name: "액션")

// Array
var genres: [Genre] = [
    Genre(id: 1, name: "액션"),
    Genre(id: 2, name: "로맨스"),
    Genre(id: 3, name: "공포")
]
```

### UserDafaults에 Object 저장

genre, genres를 `key`값으로 데이터를 `PropertyListEncoder`로 인코딩하여 저장합니다.

```swift
UserDefaults.standard.set(try? PropertyListEncoder().encode(genre), forKey:"genre")

// Array
UserDefaults.standard.set(try? PropertyListEncoder().encode(genres), forKey:"genres")
```

### UserDefaults에서 Object 가져오기

저장할때 사용했던 `Key`값으로 데이터를 가져온후 `PropertyListDecoder`로 디코딩합니다.

```swift
if let data = UserDefaults.standard.value(forKey:"genre") as? Data {
    let genre = try? PropertyListDecoder().decode(Genre.self, from: data)
}

// Array
if let data = UserDefaults.standard.value(forKey:"genres") as? Data {
    let genres = try? PropertyListDecoder().decode([Genre].self, from: data)
}
```

### UserDefaults Custom Object(Model) 활용하기

`Computed Property`를 활용하여 `UserDefaults`에 데이터를 더 쉽에 읽고 쓸 수 있습니다.

`get`에서는 `UserDefaults`에 저장되어있는 데이터를 가져와 디코딩한후 반환합니다. 데이터가 없거나 디코딩이 실패하면 예제처럼 빈배열을 반환하거나 입맛에 맞게 `nil`을 반환하면 됩니다.

`set`에서는 `newValue`를 인코딩하여 `UserDefaults`에 저장합니다.

```swift
static var movieGenres: [Genre] {
    get {
        var genres: [Genre]?
        if let data = UserDefaults.standard.value(forKey:"genres") as? Data {
            genres = try? PropertyListDecoder().decode([Genre].self, from: data)
        }
        return genres ?? []
    }
    set {
        UserDefaults.standard.set(try? PropertyListEncoder().encode(newValue), forKey:"genres")
    }
}
```