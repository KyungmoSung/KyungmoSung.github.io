---
layout: post
title:  "[ARC] Automatic Reference Counting 자동 참조 카운팅"
author: sung
categories: [ swift ]
tags: [ ARC, Reference Counting, Automatic Reference Counting, weak, strong, unowned, memory, swift ]
comments: true
---


Swift는 ARC ( *Automatic Reference Counting* )를 사용하여 앱의 메모리
사용량을 추적하고 관리합니다. 대부분의 경우 이는 Swift에서 메모리
관리가“작동하는 것”이므로 메모리 관리에 대해 스스로 생각할 필요는
없습니다. ARC는 해당 인스턴스가 더 이상 필요하지 않을 때 클래스
인스턴스가 사용하는 메모리를 자동으로 해제합니다.

그러나 경우에 따라 메모리를 관리하기 위해 ARC에 코드 부분 간의 관계에
대한 자세한 정보가 필요합니다. 이 장에서는 이러한 상황에 대해 설명하고
ARC가 모든 앱 메모리를 관리하는 방법을 보여줍니다. Swift [에서
ARC](https://developer.apple.com/library/content/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html)
를 사용하는 것은 Objective-C와 함께 ARC를 사용 [하기 위해 ARC 릴리스
노트](https://developer.apple.com/library/content/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html)
로
[전환에서](https://developer.apple.com/library/content/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html)
설명한 접근 방식과 매우 유사합니다 .

참조 카운팅은 클래스 인스턴스에만 적용됩니다. 구조와 열거는 참조 유형이
아닌 값 유형이며 참조로 저장 및 전달되지 않습니다.

ARC 작동 방식
--------------------------------------------------------------------------------------------------------------------------------------

클래스의 새 인스턴스를 만들 때마다 ARC는 해당 인스턴스에 대한 정보를
저장하기 위해 메모리 청크를 할당합니다. 이 메모리에는 해당 인스턴스와
관련된 저장된 속성 값과 함께 인스턴스 유형에 대한 정보가 있습니다.

또한 인스턴스가 더 이상 필요하지 않은 경우 ARC는 해당 인스턴스가
사용하는 메모리를 비워서 다른 용도로 메모리를 사용할 수 있습니다.
이렇게하면 클래스 인스턴스가 더 이상 필요하지 않을 때 메모리에서 공간을
차지하지 않습니다.

그러나 ARC가 여전히 사용중인 인스턴스를 할당 해제하면 더 이상 해당
인스턴스의 속성에 액세스하거나 해당 인스턴스의 메서드를 호출 할 수
없습니다. 실제로 인스턴스에 액세스하려고하면 앱이 중단 될 가능성이
높습니다.

인스턴스가 여전히 필요한 동안 사라지지 않도록 ARC는 현재 각 클래스
인스턴스를 참조하는 속성, 상수 및 변수의 수를 추적합니다. ARC는 해당
인스턴스에 대한 하나 이상의 활성 참조가 여전히 존재하는 한 인스턴스
할당을 해제하지 않습니다.

이를 가능하게하기 위해 클래스 인스턴스를 속성, 상수 또는 변수에 할당 할
때마다 해당 속성, 상수 또는 변수가 인스턴스를 *강력하게 참조* 합니다.
참조는 "강력한"참조라고하며,이 인스턴스는 해당 인스턴스를 확실하게
유지하고 강력한 참조가 남아있는 한 할당을 해제 할 수 없기 때문입니다.

작동중인 아크
--------------------------------------------------------------------------------------------------------------------------------------

다음은 자동 참조 카운팅 작동 방식의 예입니다. 이 예제는 다음과
`Person`같은 저장된 상수 속성을
정의하는 간단한 클래스로 시작 합니다 `name`.
```swift
class Person {
    let name: String
    init(name: String) {
        self.name = name
        print("\\(name) is being initialized")
    }
    deinit {
        print("\\(name) is being deinitialized")
    }
}
```
이 `Person`클래스에는 인스턴스의
`name`속성 을 설정하고 초기화가 진행
중임을 나타내는 메시지를 인쇄하는 이니셜 라이저가 있습니다. 이
`Person`클래스에는 클래스의 인스턴스가
할당 해제 될 때 메시지를 인쇄하는 초기화 해제 기능도 있습니다.

다음 코드 스 니펫은 type의 세 가지 변수를 정의하며 `Person?`,이 변수는 `Person`후속 코드 스 니펫에서 새 인스턴스에 대한 다중 참조를
설정하는 데 사용됩니다 . 이러한 변수는 선택적 유형 ( `Person?`, not `Person`)이므로 값으로 자동으로 초기화되며 `nil`현재 `Person`인스턴스를
참조하지 않습니다 .

```swift
var reference1: Person?
var reference2: Person?
var reference3: Person?
```

이제 새 `Person`인스턴스를 만들어 다음
세 가지 변수 중 하나에 할당 할 수 있습니다 .

```swift
reference1 = Person(name: "John Appleseed")
// Prints "John Appleseed is being initialized"
```
메시지 는 클래스의 이니셜 라이저 를 호출 할 때 인쇄됩니다 . 초기화가
완료되었음을
확인합니다.`"John Appleseed is being initialized"``Person`

새 `Person`인스턴스가
`reference1`변수 에 지정되었으므로 이제
`reference1`새 `Person`인스턴스에 대한 강력한 참조가 있습니다 . 최소한
하나의 강력한 참조가 있기 때문에 ARC는 이것이 `Person`메모리에 유지되고 할당 해제되지 않도록합니다.

동일한 `Person`인스턴스를 두 개의
변수에 더 할당하면 해당 인스턴스에 대한 두 개의 강력한 참조가
설정됩니다.

```swift
reference2 = reference1
reference3 = reference1
```
이제이 단일 인스턴스에 대한 *세 가지* 강력한 참조가 `Person`있습니다.

두 개의 `nil`변수 에 할당하여 이러한
강력한 참조 중 두 개 (원본 참조 포함)를 분리 하면 하나의 강력한 참조가
유지되고 `Person`인스턴스가 할당
해제되지 않습니다.

```swift
reference1 = nil
reference2 = nil
```
ARC는 `Person`세 번째이자 강력한 참조가
깨질 때까지 인스턴스 할당을 해제하지 않습니다 .이 시점에서 더 이상
`Person`인스턴스를 사용하지 않는 것이
분명 합니다.

```swift
reference3 = nil
// Prints "John Appleseed is being deinitialized"
```
클래스 인스턴스 간의 강력한 참조주기
-------------------------------------------------------------------------------------------------------------------------------------------------------------

위의 예에서 ARC는 `Person`생성 한 새
인스턴스 에 대한 참조 수를 추적하고 `Person`더 이상 필요하지 않은 경우 해당 인스턴스 를 할당 해제 할 수
있습니다.

그러나 클래스의 인스턴스가 강한 참조가없는 지점에 도달 *하지 않는*
코드를 작성할 수 있습니다 . 이는 두 클래스 인스턴스가 서로에 대한 강력한
참조를 보유하여 각 인스턴스가 다른 인스턴스를 계속 유지하는 경우 발생할
수 있습니다. 이를 *강력한 참조주기라고* 합니다.

클래스 간의 관계 중 일부를 강력한 참조가 아닌 약하거나 소유되지 않은
참조로 정의하여 강력한 참조주기를 해결합니다. 이 프로세스는 [클래스
인스턴스 간의 강력한 참조주기
해결에](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#ID52)
설명되어 있습니다. 그러나 강력한 참조주기를 해결하는 방법을 배우기 전에
이러한주기가 어떻게 발생하는지 이해하는 것이 좋습니다.

다음은 우연히 강력한 참조주기를 생성하는 방법에 대한 예입니다. 이
예제에서는 `Person`and 라는 두 클래스를
정의합니다.이 클래스 `Apartment`는
아파트 블록과 해당 거주자를 모델링합니다.

```swift
class Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
    deinit { print("\\(name) is being deinitialized") }
}

class Apartment {
    let unit: String
    init(unit: String) { self.unit = unit }
    var tenant: Person?
    deinit { print("Apartment \\(unit) is being deinitialized") }
}
```
모든 `Person`인스턴스에는
`name`유형 의 속성 `String`과 `apartment`초기에 선택적 속성이 `nil`있습니다. `apartment`사람이 항상 아파트를이 없을 수 있기 때문에 속성은 선택
사항입니다.

마찬가지로 모든 `Apartment`인스턴스에는
`unit`유형의 속성이 있으며 초기에
`String`선택적 `tenant`속성이 `nil`있습니다. 아파트에는 항상 임차인이있을 수 없으므로 임차인
속성은 선택 사항입니다.

이 두 클래스 모두 deinitializer를 정의하여 해당 클래스의 인스턴스가
초기화되지 않았다는 사실을 인쇄합니다. 이 인스턴스 여부를 확인 할 수
있습니다 `Person`및이
`Apartment`예상대로 해제되고있다.

이 다음 코드 스 니펫은 `john`and 라는
선택적 유형의 두 변수를 정의 하며 아래 `unit4A`에서 특정 `Apartment`및
`Person`인스턴스 로 설정 됩니다. 이 두
변수는 모두 `nil`선택 적이기 때문에
초기 값이입니다 .

```swift
var john: Person?
var unit4A: Apartment?
```
이제 특정 `Person`인스턴스와
`Apartment`인스턴스를 만들고 이러한 새
인스턴스를 `john`및 `unit4A`변수에 할당 할 수 있습니다 .

```swift
john = Person(name: "John Appleseed")
unit4A = Apartment(unit: "4A")
```
다음은이 두 인스턴스를 생성하고 할당 한 후 강력한 참조가 어떻게 보이는지
보여줍니다. `john`변수는 이제 새에 대한
강한 참조가 `Person`인스턴스를하고,
`unit4A`변수는 새에 대한 강한 참조가
`Apartment`인스턴스를 :

![../\_images/referenceCycle01\_2x.png](./자동%20참조%20카운팅%20—%20Swift%20프로그래밍%20언어%20(Swift%205.1)_files/referenceCycle01_2x.png)

이제 두 인스턴스를 서로 연결하여 사람이 아파트를 보유하고 아파트가
임차인을 갖도록 할 수 있습니다. 느낌표 ( `!`)는 변수 `john`및
`unit4A`선택적 변수 내에 저장된
인스턴스의 랩을 해제하고 액세스하는 데 사용 되므로 해당 인스턴스의
특성을 설정할 수 있습니다.

```swift
john!.apartment = unit4A
unit4A!.tenant = john
```
다음은 두 인스턴스를 서로 연결 한 후 강력한 참조가 어떻게 나타나는지
보여줍니다.

![../\_images/referenceCycle02\_2x.png](./자동%20참조%20카운팅%20—%20Swift%20프로그래밍%20언어%20(Swift%205.1)_files/referenceCycle02_2x.png)

불행히도이 두 인스턴스를 연결하면 두 인스턴스간에 강력한 참조주기가
생성됩니다. `Person`예 이제 강한 참조
갖는 `Apartment`인스턴스를 상기
`Apartment`인스턴스에 강한 참조 갖는
`Person`인스턴스. 따라서
`john`and `unit4A`변수가 보유한 강력한 참조를 끊을 때 참조 횟수는 0으로
떨어지지 않으며 ARC에서 인스턴스를 할당 해제하지 않습니다.

```swift
john = nil
unit4A = nil
```
이 두 변수를로 설정하면 초기화 해제 도구가 호출되지 않았습니다
`nil`. 강력한 참조주기는
`Person`및 `Apartment`인스턴스가 할당 해제되지 않도록하여 앱에서 메모리
누수를 유발합니다.

`john`및 `unit4A`변수를 `nil`다음 과 같이
설정 한 후 강력한 참조가 표시되는 방식은 다음과 같습니다 .

![../\_images/referenceCycle03\_2x.png](./자동%20참조%20카운팅%20—%20Swift%20프로그래밍%20언어%20(Swift%205.1)_files/referenceCycle03_2x.png)

`Person`인스턴스와 인스턴스 사이의
강력한 참조는 `Apartment`유지되며
깨지지 않습니다.

클래스 인스턴스 간의 강력한 참조주기 해결
------------------------------------------------------------------------------------------------------------------------------------------------------------------

Swift는 클래스 유형의 속성으로 작업 할 때 강한 참조주기를 해결하는 두
가지 방법, 약한 참조 및 소유되지 않은 참조를 제공합니다.

약하고 소유되지 않은 참조는 참조주기의 한 인스턴스가 다른 인스턴스 *를*
강력하게 유지 *하지 않고* 다른 인스턴스를 참조 할 *수* 있도록합니다.
그런 다음 인스턴스는 강력한 참조주기를 만들지 않고 서로를 참조 할 수
있습니다.

다른 인스턴스의 수명이 짧을 때, 즉 다른 인스턴스를 먼저 할당 해제 할
수있는 경우 약한 참조를 사용하십시오. 위의 `Apartment`예에서, 아파트가 일생 동안 어떤 시점에 임차인을
가질 수없는 것이 적절하므로이 경우 기준이 약한 기준이 기준주기를
중단하는 적절한 방법입니다. 반대로, 다른 인스턴스의 수명이 동일하거나
수명이 길면 소유하지 않은 참조를 사용하십시오.

### 약한 참조

*약한 참조는* 참조 예를 폐기에서 ARC 중지되지 않습니다 그래서가 참조하는
인스턴스에 강한 보류를 유지하고 있지 않은 참조입니다. 이 동작은 참조가
강력한 참조주기의 일부가되는 것을 방지합니다. `weak`속성 또는 변수 선언 앞에 키워드를 배치하여 약한 참조를
나타냅니다 .

약한 참조는 참조하는 인스턴스를 강력하게 유지하지 않기 때문에 약한
참조가 여전히 참조하는 동안 해당 인스턴스의 할당이 해제 될 수 있습니다.
따라서 ARC는 참조 `nil`하는 인스턴스가
할당 해제 될 때 자동으로 약한 참조를 설정합니다 . 또한 약한 참조
`nil`는 런타임에 값을 변경할 수
있어야하기 때문에 항상 선택적 유형의 상수가 아니라 변수로 선언됩니다.

다른 선택적 값과 마찬가지로 약한 참조에 값이 있는지 확인할 수 있으며 더
이상 존재하지 않는 유효하지 않은 인스턴스에 대한 참조로 끝나지 않습니다.

노트

ARC가에 대한 약한 참조를 설정하면 속성 관찰자는 호출되지 않습니다
`nil`.

아래 예제는 위의 예제 `Person`와
동일하지만 `Apartment`한 가지 중요한
차이점이 있습니다. 이번에는 `Apartment`형식의 `tenant`속성이 약한
참조로 선언되었습니다.

```swift
class Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
    deinit { print("\\(name) is being deinitialized") }
}

class Apartment {
    let unit: String
    init(unit: String) { self.unit = unit }
    weak var tenant: Person?
    deinit { print("Apartment \\(unit) is being deinitialized") }
}
```
두 변수 ( `john`및 `unit4A`) 의 강력한 참조 와 두 인스턴스 간의 링크는 이전과
같이 작성됩니다.

```swift
var john: Person?
var unit4A: Apartment?

john = Person(name: "John Appleseed")
unit4A = Apartment(unit: "4A")

john!.apartment = unit4A
unit4A!.tenant = john
```
다음은 두 인스턴스를 서로 연결 한 후 참조가 어떻게 보이는지 보여줍니다.

![../\_images/weakReference01\_2x.png](./자동%20참조%20카운팅%20—%20Swift%20프로그래밍%20언어%20(Swift%205.1)_files/weakReference01_2x.png)

`Person`인스턴스는 여전히 강한 참조가
`Apartment`인스턴스를하지만,
`Apartment`인스턴스는 지금이 *약한*
받는 참조 `Person`인스턴스를. 즉,
`john`변수를로 설정 하여 변수가 보유한
강력한 참조를 해제하면 `nil`더 이상
`Person`인스턴스에 대한 참조가 없습니다
.
```swift
john = nil
// Prints "John Appleseed is being deinitialized"
```
더 이상 `Person`인스턴스에 대한 참조가
없으므로 할당이 해제되고 `tenant`속성이
다음 과 같이 설정됩니다 `nil`.

![../\_images/weakReference02\_2x.png](./자동%20참조%20카운팅%20—%20Swift%20프로그래밍%20언어%20(Swift%205.1)_files/weakReference02_2x.png)

`Apartment`인스턴스에 대한 강력한 참조
는 `unit4A`변수 에서 유일하게 유지
됩니다. 당신이 중단되면 *그* 강한 참조는 더 이상 강한 참조가
`Apartment`인스턴스 :
```swift
unit4A = nil
// Prints "Apartment 4A is being deinitialized"
```
더 이상 `Apartment`인스턴스에 대한
참조가 없으므로 할당이 해제됩니다.

![../\_images/weakReference03\_2x.png](./자동%20참조%20카운팅%20—%20Swift%20프로그래밍%20언어%20(Swift%205.1)_files/weakReference03_2x.png)

노트

가비지 수집을 사용하는 시스템에서는 메모리 압력이 가비지 수집을 트리거
할 때만 강력한 참조가없는 개체가 할당 해제되기 때문에 약한 포인터를
사용하여 간단한 캐싱 메커니즘을 구현하는 경우가 있습니다. 그러나 ARC를
사용하면 마지막 강력한 참조가 제거 되 자마자 값이 할당 해제되어 약한
참조가 이러한 목적에 적합하지 않게됩니다.

### 소유되지 않은 참조

약한 참조처럼, *소유되지 않은 참조* 는 *참조* 하는 인스턴스를 강력하게
유지하지 않습니다. 그러나 약한 참조와 달리 소유하지 않은 참조는 다른
인스턴스의 수명이 동일하거나 수명이 길면 사용됩니다. `unowned`속성 또는 변수 선언 앞에 키워드를 배치하여
소유되지 않은 참조를 나타냅니다 .

소유하지 않은 참조에는 항상 값이 있어야합니다. 결과적으로 ARC는 소유되지
않은 참조 값을로 설정하지 않습니다. `nil`즉, 소유하지 않은 참조는 비 선택적 유형을 사용하여
정의됩니다.

중대한

참조가 *항상* 할당 해제되지 않은 인스턴스를 참조하는 경우에만 소유되지
않은 참조를 사용하십시오 .

해당 인스턴스 할당이 해제 된 후 소유되지 않은 참조의 값에
액세스하려고하면 런타임 오류가 발생합니다.

다음 예제에서는 두 개의 클래스를 정의 `Customer`하고 `CreditCard`, 어떤
모델 은행 고객과 해당 고객에 대한 가능한 신용 카드. 이 두 클래스는 각각
다른 클래스의 인스턴스를 속성으로 저장합니다. 이 관계는 강력한
참조주기를 만들 가능성이 있습니다.

와의 관계는 위의 약한 참조 예에서 와 의 관계 `Customer`와 `CreditCard`약간 다릅니다 . 이 데이터 모델에서 고객은 신용 카드를
보유하거나 보유하지 않을 수 있지만 신용 카드는 *항상* 고객과 연결됩니다.
인스턴스는 들보 다 오래 남았습니다 결코 그것을 참조하는. 이를 나타 내기
위해 클래스에는 선택적 속성이 있지만 클래스에는 소유되지 않은 (및 비
선택적) 속성이 있습니다.`Apartment``Person`**`CreditCard``Customer``Customer``card``CreditCard``customer`

또한 값과 인스턴스를 사용자 정의 이니셜 라이저 로 전달 *해야만* 새
`CreditCard`인스턴스를 작성할 *수*
있습니다 . 이를 통해 인스턴스가 작성 될 때 인스턴스에 항상 연관된
인스턴스가 있습니다.`number``customer``CreditCard``CreditCard``customer``CreditCard`

신용 카드에는 항상 고객이 있기 때문에 `customer`강력한 참조주기를 피하기 위해 해당 속성을 소유하지 않은
참조로 정의 합니다.

```swift
class Customer {
    let name: String
    var card: CreditCard?
    init(name: String) {
        self.name = name
    }
    deinit { print("\\(name) is being deinitialized") }
}

class CreditCard {
    let number: UInt64
    unowned let customer: Customer
    init(number: UInt64, customer: Customer) {
        self.number = number
        self.customer = customer
    }
    deinit { print("Card \#\\(number) is being deinitialized") }
}
```
노트

클래스 의 `number`속성은 32 비트 및 64
비트 시스템 모두에 16 자리 카드 번호를 저장할 수 있도록 속성의 용량이
충분히 커지 도록 `CreditCard`유형이
`UInt64`아닌으로 정의
됩니다.`Int``number`

다음 코드 스 니펫은이라는 선택적 `Customer`변수를 정의합니다.이 변수 `john`는 특정 고객에 대한 참조를 저장하는 데 사용됩니다. 이
변수는 선택 적이기 때문에 초기 값은 nil입니다.

```swift
var john: Customer?
```
이제 `Customer`인스턴스를 생성하고 이를
사용하여 `CreditCard`고객 인스턴스로 새
인스턴스 를 초기화하고 할당 할 수 `card`있습니다.

```swift
john = Customer(name: "John Appleseed")
john!.card = CreditCard(number: 1234\_5678\_9012\_3456, customer: john!)
```
두 인스턴스를 연결 했으므로 참조 모양은 다음과 같습니다.

![../\_images/unownedReference01\_2x.png](./자동%20참조%20카운팅%20—%20Swift%20프로그래밍%20언어%20(Swift%205.1)_files/unownedReference01_2x.png)

`Customer`예 이제 강한 참조 갖는
`CreditCard`인스턴스를 상기
`CreditCard`인스턴스에 소유되지 않은
기준 갖는 `Customer`인스턴스.

소유되지 않은 `customer`참조로 인해
`john`변수가 보유한 강력한 참조를 끊을
때 더 이상 `Customer`인스턴스에 대한
참조가 없습니다 .

![../\_images/unownedReference02\_2x.png](./자동%20참조%20카운팅%20—%20Swift%20프로그래밍%20언어%20(Swift%205.1)_files/unownedReference02_2x.png)

더 이상 `Customer`인스턴스에 대한
참조가 없으므로 할당이 해제됩니다. 이 문제가 발생하면 더 이상
`CreditCard`인스턴스에 대한 참조가
없으며 할당이 해제됩니다.

```swift
john = nil
// Prints "John Appleseed is being deinitialized"
// Prints "Card \#1234567890123456 is being deinitialized"
```
위의 마지막 코드 스 니펫 은 변수가로 설정된 후 `Customer`인스턴스 및 `CreditCard`인스턴스의 초기화 해제 기가 "초기화되지 않은"메시지를 인쇄
함 을 보여줍니다 .`john``nil`

노트

위의 예는 *안전한* 소유되지 않은 참조 를 사용하는 방법을 보여줍니다 .
Swift는 또한 성능상의 이유로 런타임 안전 검사를 비활성화해야하는 경우
*안전하지* 않은 참조를 제공합니다 . 안전하지 않은 모든 작업과 마찬가지로
해당 코드의 안전 점검 책임은 귀하에게 있습니다.

을 쓰면 안전하지 않은 참조를 나타냅니다 `unowned(unsafe)`. 참조하는 인스턴스가 할당 해제 된 후 안전하지
않은 소유되지 않은 참조에 액세스하려고하면 프로그램은 인스턴스가 있던
메모리 위치 (안전하지 않은 작업)에 액세스하려고 시도합니다.

### 소유되지 않은 참조 및 내재적으로 래핑되지 않은 선택적 속성

위의 약하고 소유되지 않은 참조에 대한 예는 강력한 참조주기를
중단해야하는보다 일반적인 두 가지 시나리오를 다룹니다.

`Person`및 `Apartment`예는 둘 수있는 두 가지 특성, 상황 표시
`nil`, 강한 기준주기를 일으킬
가능성이있다. 이 시나리오는 약한 참조로 가장 잘 해결됩니다.

`Customer`및 `CreditCard`예는 하나 개시킨다 등록 상황 표시 `nil`될 수없는 다른 속성 `nil`가능성이 강한 기준주기를 유발한다. 이 시나리오는 소유하지
않은 참조로 해결하는 것이 가장 좋습니다.

그러나 *두* 속성 *모두* 항상 값을 가져야하며 `nil`초기화가 완료된 후에는 *두* 속성이 없어야 하는 세 번째
시나리오 가 있습니다 . 이 시나리오에서는 한 클래스의 소유되지 않은
속성을 다른 클래스의 암시 적으로 래핑되지 않은 선택적 속성과 결합하는
것이 좋습니다.

이를 통해 초기화가 완료되면 참조주기를 피하면서 두 속성 모두에 직접
액세스 할 수 있습니다 (선택적 래핑 해제없이). 이 섹션에서는 그러한
관계를 설정하는 방법을 보여줍니다.

아래 예제는 두 개의 클래스를 정의 `Country`하고 `City`각각은 다른
클래스의 인스턴스를 속성으로 저장합니다. 이 데이터 모델에서 모든
국가에는 항상 수도가 있어야하며 모든 도시는 항상 국가에 속해야합니다.
이를 나타 내기 위해 `Country`클래스에는
`capitalCity`속성이 있으며
`City`클래스에는 `country`속성이 있습니다.

```swift
class Country {
    let name: String
    var capitalCity: City!
    init(name: String, capitalName: String) {
        self.name = name
        self.capitalCity = City(name: capitalName, country: self)
    }
}

class City {
    let name: String
    unowned let country: Country
    init(name: String, country: Country) {
        self.name = name
        self.country = country
    }
}
```
두 클래스 사이의 상호 의존성을 설정하기 위해 초기화 프로그램
`City`은 `Country`인스턴스 를 가져 와서이 인스턴스를 해당 `country`속성 에 저장 합니다.

의 초기화 프로그램 `City`은의 초기화
프로그램 내에서 호출됩니다 `Country`.
그러나 [2 단계
초기화에](https://docs.swift.org/swift-book/LanguageGuide/Initialization.html#ID220)
설명 된대로 새 인스턴스가 완전히 초기화 될 때까지 초기화 프로그램 이
초기화 프로그램에 `Country`전달 될 수
없습니다 .`self``City``Country`[](https://docs.swift.org/swift-book/LanguageGuide/Initialization.html#ID220)

이 요구 사항에 대처하기 위해 `capitalCity`속성을 `Country`암시
적으로 래핑되지 않은 선택적 속성으로 선언 하고 형식 주석 끝에 느낌표가
표시됩니다 ( `City!`). 즉,
`capitalCity`속성의 기본값은
`nil`다른 옵션과 같지만 [암시 적으로
래핑되지 않은
옵션에](https://docs.swift.org/swift-book/LanguageGuide/TheBasics.html#ID334)
설명 된대로 값을 언랩 할 필요없이 액세스 할 수 있습니다 .

때문에 `capitalCity`기본이
`nil`값을, 새로운 `Country`인스턴스가 완전히 즉시로 초기화 된 것으로 간주되는
`Country`인스턴스가 설정
`name`의 초기화 내에서 속성을. 이것은
`Country`이니셜 라이저가 속성을 설정
`self`하자마자 암시 적 속성 을 참조하고
전달하기 시작할 수 있음을 의미합니다 `name`. `Country`이니셜 따라서
통과 할 수 `self`위한 파라미터의
하나로서 `City`이니셜
`Country`이니셜 자체 설정되는
`capitalCity`속성.

이 모든 것은 강력한 참조주기를 만들지 않고 단일 문에서
`Country`및 `City`인스턴스를 만들 `capitalCity`수 있으며 느낌표를 사용하여 선택적 값을 풀지 않고도 속성에
직접 액세스 할 수 있음을 의미합니다.

```swift
var country = Country(name: "Canada", capitalName: "Ottawa")
print("\\(country.name)'s capital city is called \\(country.capitalCity.name)")
// Prints "Canada's capital city is called Ottawa"
```
위의 예에서 암시 적으로 래핑되지 않은 선택적 옵션을 사용하면 모든 2 단계
클래스 이니셜 라이저 요구 사항이 충족됩니다. `capitalCity`속성은 사용 및 초기화가 완료되면 여전히 강한
참조주기를 피하면서, 비 옵션 값처럼 액세스 할 수 있습니다.

폐쇄를위한 강력한 기준주기
---------------------------------------------------------------------------------------------------------------------------------------------------

위에서 두 클래스 인스턴스 속성이 서로에 대한 강력한 참조를 보유 할 때
강력한 참조주기를 만드는 방법을 살펴 보았습니다. 또한 약하고 소유되지
않은 참조를 사용하여 이러한 강력한 참조주기를 끊는 방법도 살펴
보았습니다.

클래스 인스턴스의 속성에 클로저를 할당하고 해당 클로저의 본문이
인스턴스를 캡처하는 경우에도 강력한 참조주기가 발생할 수 있습니다. 이
캡처는 클로저의 본문이와 같은 인스턴스의 속성에 액세스
`self.someProperty`하거나 클로저가와
같은 인스턴스의 메서드를 호출하기 때문에 발생할 수 있습니다
`self.someMethod()`. 두 경우 모두
이러한 액세스로 인해 클로저가 "캡처" `self`되어 강력한 참조주기가 생성됩니다.

이처럼 강력한 참조주기는 클래스와 같은 클로저가 *참조 유형* 이기 때문에
발생합니다 . 클로저를 속성에 할당하면 해당 클로저에 대한 *참조* 가 할당
됩니다. 본질적으로, 위와 동일한 문제입니다. 두 개의 강력한 참조가 서로를
유지합니다. 그러나 두 개의 클래스 인스턴스가 아닌 이번에는 클래스
인스턴스와 클로저가 서로를 살아있게 만듭니다.

Swift는 *클로저 캡처리스트* 라고하는이 문제에 대한 우아한 솔루션을 제공
*합니다* . 그러나 클로저 캡처 목록을 사용하여 강력한 참조주기를 중단하는
방법을 배우기 전에 이러한주기를 발생시키는 방법을 이해하는 것이
좋습니다.

아래 예제는 참조하는 클로저를 사용할 때 강력한 참조주기를 생성하는
방법을 보여줍니다 `self`. 이
예제는이라는 클래스를 정의합니다.이 클래스 `HTMLElement`는 HTML 문서 내의 개별 요소에 대한 간단한 모델을
제공합니다.
```swift
class HTMLElement {

    let name: String
    let text: String?

    lazy var asHTML: () -\> String = {
        if let text = self.text {
            return "\<\\(self.name)\>\\(text)\</\\(self.name)\>"
        } else {
            return "\<\\(self.name) /\>"
        }
    }

    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }

    deinit {
        print("\\(name) is being deinitialized")
    }

}
```
이 `HTMLElement`클래스는 머리글 요소,
단락 요소 또는 줄 바꿈 요소 `name`와
같은 요소 이름을 나타내는 속성을 정의합니다 . 또한 HTML 요소 내에서
렌더링 할 텍스트를 나타내는 문자열로 설정할 수 있는 선택적 속성을
정의합니다 .`"h1"``"p"``"br"``HTMLElement``text`

이 두 가지 간단한 속성 외에도 `HTMLElement`클래스는이라는 게으른 속성을 정의합니다 `asHTML`. 이 속성은 결합 폐쇄 참조 `name`및 `text`HTML
문자열 조각으로합니다. 이 `asHTML`속성은 유형 이거나 "매개 변수를 사용하지 않고 값을 반환하는
함수 "입니다.`() -> String``String`

기본적 `asHTML`으로이 속성에는 HTML
태그의 문자열 표현을 반환하는 클로저가 할당됩니다. 이 태그에는 선택적인
`text`값이 있거나 `text`존재하지 않으면 텍스트 내용 이 없습니다. 단락
요소에 대해 폐쇄는 반환 또는 여부에 따라 속성이 동일 또는
.`"<p>some text</p>"``"<p />"``text``"some text"``nil`

이 `asHTML`속성의 이름은 인스턴스
메서드와 비슷합니다. 그러나 `asHTML`인스턴스 메소드가 아닌 클로저 속성 이므로
`asHTML`특정 HTML 요소에 대한 HTML
렌더링을 변경하려는 경우 속성 의 기본값을 사용자 정의 클로저로 바꿀 수
있습니다.

예를 들어, 속성이 인 경우 표현이 빈 HTML 태그를 반환하지 못하도록하기
위해 속성 `asHTML`을 일부 텍스트로 기본
설정하는 클로저로 속성을 설정할 수 있습니다 .`text``nil`
```swift
let heading = HTMLElement(name: "h1")
let defaultText = "some default text"
heading.asHTML = {
    return "\<\\(heading.name)\>\\(heading.text ?? defaultText)\</\\(heading.name)\>"
}
print(heading.asHTML())
// Prints "\<h1\>some default text\</h1\>"
```
노트

이 `asHTML`속성은 요소가 실제로 일부
HTML 출력 대상에 대한 문자열 값으로 렌더링되어야하는 경우에만 필요하기
때문에 지연 속성으로 선언됩니다. `asHTML`게으른 속성이라는 사실 은 `self`초기화가 완료되고 `self`존재하는 것으로 알려진 후에 게으른 속성에 액세스 할 수 없기
때문에 기본 클로저 내에서 참조 할 수 있음을 의미 합니다.

이 `HTMLElement`클래스는 단일 이니셜
라이저를 제공하며 `name`,
`text`인수는 새로운 요소를 초기화하기
위해 인수 및 원하는 경우 인수 를 취합니다 . 이 클래스는 deinitializer를
정의하여 `HTMLElement`인스턴스 할당이
해제 될 때 표시되는 메시지를 인쇄합니다 .

`HTMLElement`클래스를 사용하여 새
인스턴스를 만들고 인쇄하는 방법은 다음과 같습니다 .
```swift
var paragraph: HTMLElement? = HTMLElement(name: "p", text: "hello, world")
print(paragraph!.asHTML())
// Prints "\<p\>hello, world\</p\>"
```
노트

위 `paragraph`변수는 *선택적* 으로 정의
`HTMLElement`되므로 `nil`강력한 참조주기가 있음을 보여주기 위해 아래로
설정할 수 있습니다 .

불행히도, `HTMLElement`위에서 언급 한
클래스는 `HTMLElement`인스턴스와
기본값으로 사용되는 클로저 사이에 강력한 참조주기를 만듭니다
`asHTML`. 주기는 다음과 같습니다.

![../\_images/closureReferenceCycle01\_2x.png](./자동%20참조%20카운팅%20—%20Swift%20프로그래밍%20언어%20(Swift%205.1)_files/closureReferenceCycle01_2x.png)

인스턴스의 `asHTML`속성은 해당 클로저에
대한 강력한 참조를 보유합니다. 그러나 클로저는 `self`( `self.name`및 참조
방법으로) 본문 내를 참조 `self.text`하므로 클로저 *는* 자체를 *캡처* 하므로
`HTMLElement`인스턴스에 대한 강력한
참조를 보유 합니다. 둘 사이에 강력한 참조주기가 생성됩니다. 클로저에서
값을 캡처하는 방법에 대한 자세한 내용은 값
[캡처를](https://docs.swift.org/swift-book/LanguageGuide/Closures.html#ID103)
참조하십시오 .

노트

클로저는 `self`여러 번 참조되지만
`HTMLElement`인스턴스에 대한 하나의
강력한 참조 만 캡처 합니다.

`paragraph`변수를로 설정하고 인스턴스에
`nil`대한 강력한 참조를 해제하면 강한
참조 주기로 인해 인스턴스와 해당 클로저의 할당이 해제되지
않습니다.`HTMLElement``HTMLElement`
```swift
paragraph = nil
```
`HTMLElement`초기화 해제 기 의 메시지는
인쇄 `HTMLElement`되지 않으며 인스턴스
할당이 해제되지 않았 음 을 나타냅니다 .

클로저에 대한 강력한 참조 사이클 해결
--------------------------------------------------------------------------------------------------------------------------------------------------------------

클로저 정의의 일부로 *캡처 목록* 을 정의하여 클로저와 클래스 인스턴스
간의 강력한 참조주기를 해결합니다 . 캡처 목록은 클로저 본문 내에서 하나
이상의 참조 유형을 캡처 할 때 사용할 규칙을 정의합니다. 두 클래스
인스턴스 간의 강력한 참조주기와 마찬가지로 캡처 된 각 참조를 강력한
참조가 아닌 약하거나 소유되지 않은 참조로 선언합니다. 약하거나 소유되지
않은 적절한 선택은 코드의 다른 부분 사이의 관계에 따라 다릅니다.

노트

Swift 는 폐쇄 내에서 구성원을 언급 할 때마다 ( 또는 ) 대신
`self.someProperty`또는 을 쓰도록
요구합니다 . 이를 통해 실수 로 캡처 할 수 있음을 기억할 수 있습니다
.`self.someMethod()``someProperty``someMethod()``self``self`

### 캡처 목록 정의

캡처 목록의 각 항목은 클래스 인스턴스 (예 :) 또는 일부 값 (예 :)으로
초기화 된 변수에 대한 참조와 키워드 `weak`또는 `unowned`키워드 의
쌍입니다 . 이 쌍은 쉼표로 구분 된 한 쌍의 대괄호 안에
작성됩니다.`self``delegate = self.delegate!`

캡처 목록을 클로저의 매개 변수 목록 앞에 놓고 제공된 경우 반환 유형 :
```swift
lazy var someClosure: (Int, String) -\> String = {
    [unowned self, weak delegate = self.delegate!] (index: Int, stringToProcess: String) -\> String in
    // closure body goes here
}
```
클로저가 컨텍스트에서 유추 될 수 있으므로 매개 변수 목록 또는 리턴
유형을 지정하지 않은 경우 캡처 목록을 클로저의 맨 처음에두고
`in`키워드를 입력하십시오.
```swift
lazy var someClosure: () -\> String = {
    [unowned self, weak delegate = self.delegate!] in
    // closure body goes here
}
```
### 약하고 소유되지 않은 참조

클로저와 캡처하는 인스턴스가 항상 서로를 참조하고 항상 동시에 할당 해제
될 때 클로저의 캡처를 소유하지 않은 참조로 정의하십시오.

반대로 캡처 된 참조가 `nil`향후 어느
시점에 있을 때 캡처를 약한 참조로 정의하십시오 . 약한 참조는 항상 선택적
유형이며 `nil`참조하는 인스턴스가 할당
해제되면 자동으로됩니다 . 이를 통해 클로저 바디 내에 존재하는지 확인할
수 있습니다.

노트

캡처 된 참조가 절대로되지 `nil`않으면
항상 약한 참조가 아닌 소유되지 않은 참조로 캡처해야합니다.

소유되지 않은 참조는 위의 [폐쇄에 대한 강력한
참조주기](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#ID56)
의 `HTMLElement`예에서 [강한
참조주기](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#ID56)
를 해결하는 데 사용하는 적절한 캡처 방법 입니다. `HTMLElement`주기를 피하기 위해 클래스를 작성하는 방법은 다음과
같습니다 .
```swift
class HTMLElement {

    let name: String
    let text: String?

    lazy var asHTML: () -\> String = {
        [unowned self] in
        if let text = self.text {
            return "\<\\(self.name)\>\\(text)\</\\(self.name)\>"
        } else {
            return "\<\\(self.name) /\>"
        }
    }

    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }

    deinit {
        print("\\(name) is being deinitialized")
    }

}
```
이 구현은 클로저 `HTMLElement`내에 캡처
목록을 추가한다는 점을 제외하면 이전 구현과 동일합니다
`asHTML`. 이 경우 캡처 목록은 "강한
참조가 아닌 소유되지 않은 참조로 자체 캡처"를
의미합니다.`[unowned self]`

`HTMLElement`이전과 같이 인스턴스를
만들고 인쇄 할 수 있습니다 .
```swift
var paragraph: HTMLElement? = HTMLElement(name: "p", text: "hello, world")
print(paragraph!.asHTML())
// Prints "\<p\>hello, world\</p\>"
```
캡처 목록을 사용하여 참조가 표시되는 방식은 다음과 같습니다.

![../\_images/closureReferenceCycle02\_2x.png](./자동%20참조%20카운팅%20—%20Swift%20프로그래밍%20언어%20(Swift%205.1)_files/closureReferenceCycle02_2x.png)

이번에 `self`는 클로저 에 의한 캡처 는
소유되지 않은 참조이며 `HTMLElement`캡처 한 인스턴스를 강력하게 유지하지 않습니다 . 당신이에서
강한 참조 설정 한 경우 `paragraph`에
변수 `nil`는 `HTMLElement`아래의 예에서의 deinitializer 메시지의 인쇄에서 볼
수 있듯이 인스턴스는, 할당 해제됩니다 :
```swift
paragraph = nil
// Prints "p is being deinitialized"
```
캡처 목록에 대한 자세한 내용은 [캡처
목록을](https://docs.swift.org/swift-book/ReferenceManual/Expressions.html#ID544)

### 원문
> [https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html]( https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html)

