---
layout: post
title:  "[Swift] JSON Codable: Enum with unknown value"
author: Sung Kyungmo
catalog: true
tags: [ios, swift, codable, enum, alamofire, json, unknown, model]
comments: true
---

# Codable에서 Enum 사용
Json 데이터의 값이 특정한(규칙적인) 형태라면 열거형인 `Enum`으로 매칭시켜 쉽게 사용할 수 있습니다.

예를들어 아래와같이 자동차 Data가 있을때 연료의 종류(가솔린, 디젤, LPG, 배터리)는 몇가지로 한정되어있기 때문에 `Enum`으로 매칭시키기에 적합합니다.

### JSON Data

```json
{
    "id": 1234,
    "name": "테스트카",
    "fuelType": "G"
}
```

### 기존 Car 모델

```swift
struct Car: Codable {
    var id: Int?
    var name: String?
    var fuelType: String?
}
```

### FuelType Enum(열거형) 정의

- 우선 `FuelType`을 `Enum`으로 정의해줍니다.
- `Codable` 프로토콜을 따르는 `Car` 모델에서 `FuelType`을 사용하기 위해서는 `FuelType`도 `Codable` 프로토콜을 채택해야 합니다.
- `Codable` 프로토콜을 추가해주지 않으면 `Type 'Car' does not conform to protocol 'Decodable'` 에러가 발생합니다.
- 각 case `rawValue`에는 JSON Data와 매칭되는 `FuelType` 값을 넣어줍니다.

```swift
enum FuelType: String, Codable {
    case gasoline  = "G" // 가솔린
    case diesel    = "D" // 디젤
    case lpg       = "L" // LPG
    case battery   = "B" // 전기
}
```

## unknown value default case 추가

- JSON Data를 파싱할때 `FuelType`에 정의되어있지 않은 값이나 빈 문자열이 올 경우에는 `DecodingError.dataCorrupted`에러가 발생합니다.
- 이런경우 아무것도 해당되지 않는 `unknown` case를 추가하여 `default` 로 사용할 수 있습니다

```swift
enum FuelType: String, Codable {
    case gasoline  = "G" // 가솔린
    case diesel    = "D" // 디젤
    case lpg       = "L" // LPG
    case battery   = "B" // 전기
    case unknown
}

public init(from decoder: Decoder) throws {
    self = try FuelType(rawValue: decoder.singleValueContainer().decode(RawValue.self)) ?? .unknown
}
```

### Car 모델에 Enum 적용

```swift
struct Car: Codable {
    var id: Int?
    var name: String?
    var fuelType: FuelType?
}
```