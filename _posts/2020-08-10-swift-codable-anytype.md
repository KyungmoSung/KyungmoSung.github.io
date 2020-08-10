---
layout: post
title:  "[Swift] JSON Codable: Any Type Custom"
author: Sung Kyungmo
catalog: true
tags: [ios, swift, codable, any]
comments: true
---

# Any type in Codable

API통신을 하다보면 같은 Model로 Response가 오지만 특정 값의 타입이 일정하지 않은 경우가 있습니다.

#### Case 1
```json
{
  "id": 1234,
  "name": "Sung"
}
```

#### Case 2
```json

{
  "id": "abcd",
  "name": "Sung"
}
```

#### Model
```swift
class Genre: Codable {
    let id: Int!
    let name: String!
}
```
Case1에서는 문제가 없지만 Case2에서는 `id`값이 `String`이기 때문에 typeMismatch Error가 발생합니다.

Codable에서는 `Any` 타입을 사용하기 어렵기 때문에 ObjC에 있는 `NSNumber`같이 특정타입으로 값을 꺼내쓸 수 있는 커스텀 타입을 만들어 활용할 수 있습니다.


## AnyValue

- 여기서는 `Int`, `String`이 들어갈 수 있는 `AnyValue`타입을 만들어보겠습니다.
- 디코딩 가능한 타입이 없는경우에는 실패 에러를 발생시킵니다.

```swift
enum AnyValue: Codable {
    case int(Int)
    case string(String)

    init(from decoder: Decoder) throws {
        if let int = try? decoder.singleValueContainer().decode(Int.self) {
            self = .int(int)
            return
        }

        if let string = try? decoder.singleValueContainer().decode(String.self) {
            self = .string(string)
            return
        }

        // 디코딩 가능한 타입이 없는경우 typeMismatch 에러 발생
        throw AnyValueError.typeMismatch
    }
    
    func encode(to encoder: Encoder) throws {
        var container = encoder.singleValueContainer()
        
        switch self {
        case .int(let value):
            try container.encode(value)
        case .string(let value):
            try container.encode(value)
        }
    }

    enum AnyValueError:Error {
        case typeMismatch
    }
}
```

## AnyValue Extension
- 이제 `AnyValue`타입의 값에 `Int`나 `String`값이 들어갈 수 있습니다.
- 필요한 값을 편하게 가져다 쓰기 위해서 `Extension`으로 구현해줍니다.

```swift
extension AnyValue {
    var intValue: Int? {
        switch self {
        case .int(let value):
            return value
        case .string(let value):
            return Int(value)
        }
    }
    
    var stringValue: String? {
        switch self {
        case .int(let value):
            return String(value)
        case .string(let value):
            return value
        }
    }
}
```

## AnyValue 사용

```swift
class Genre: Codable {
    let id: AnyValue!
    let name: String!
}
```

```swift
let intID = genre.id.intValue
let stringID = genre.id.stringValue
```