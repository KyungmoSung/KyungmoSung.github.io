---
layout: post
title:  "[WWDC2019] SwiftUI 소개: 첫번째 앱 만들기"
author: sung
categories: [ swift ]
tags: [swift, swiftui, wwdc, wwdc2019]
comments: true
---
지난 WWDC 2019에서 SwiftUI가 발표되었습니다.
그동안 macOS 베타버전은 버그가 많아서 카탈리나로 업데이트를 못하고 있다가 드디어 모하비에서 카탈리나로 업데이트를 완료했습니다. 업데이트 기념으로 항상 궁금했던 SwiftUI를 사용해보고 WWDC에서 나왔던 데모 프로젝트를 같이 한번 만들어보도록 하겠습니다.
>SwiftUI는 Xcode 11 이상에서 사용 가능하며 Preview기능은 macOS Catalina 부터 사용 가능합니다.

<br>

## SwiftUI 는 무엇인가?
---
SwiftUI는 Swift의 성능을 바탕으로 모든 Apple 플랫폼에서 사용자 인터페이스를 구축할 수 있는 혁신적이고 간소화된 방법입니다. 단 하나의 도구 구성 및 API를 통해 모든 Apple 기기에서 사용할 수 있는 사용자 인터페이스를 구축합니다. 읽기 쉽고 작성하기 편한 선언적 Swift 구문을 통해 SwiftUI는 새로운 Xcode 디자인 도구와 매끄럽게 연동되면서 코드와 디자인이 완벽하게 동기화되도록 합니다. 또한 유동적 글자 크기 조절, 다크 모드, 현지화 및 손쉬운 사용을 자동 지원하므로 SwiftUI 코딩 첫 줄부터 가장 강력한 UI 코드를 작성할 수 있습니다.

<br>

## 프로젝트 생성
---
새로운 프로젝트를 만들어보겠습니다. 아래 이미지와 같이 User Interface를 보면 새롭게 추가된 SwiftUI가 보입니다. 선택하고 프로젝트 생성!
> 주의! Product Name을 SwiftUI로 만들게 되면 프로젝트명과 SwiftUI 프레임워크와 충돌이 일어나 제대로 import를 못해 에러가 일어납니다.

![](../assets/images/swift-ui-wwdc/1.png)

프로젝트를 생성하면 바로 ContentView.swift 가 열리고 오른쪽에는 정체를 알수없는 프리뷰 화면도 보입니다.   
우측 상단에 Resume버튼을 누르면 신기한 일이 일어납니다!  
빌드가 시작되고 잠시후 시뮬레이터가 엑스코드 속으로 들어온듯한 캔버스 화면이 뜹니다.  

## 프로젝트 시작
---
```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Text("Hello World")
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

`ContentView`는 말그대로 하나의 뷰를 나타내고 `ContentView_Preview`는 캔버스에 무엇을 보여줄지 설정하는 구조체 입니다. 이제 화면 중앙에 텍스트를 변경해보겠습니다.  
`Text("Hello World")`에 텍스트를 변경하면 실시간으로 캔버스에 적용됩니다.

## Stack
---
조금더 복잡한 레이아웃을 만들기 위해서 `HStack`(Horizontal), `VStack`(vertical)을 사용해 보겠습니다. (기존에 `StackView`와 같은 개념이라고 생각하시면 됩니다.)  
Cmd키를 누른상태에서 `Text()`를 선택하면 아래와 같은 메뉴들이 나타납니다.  
Embed in HStack을 선택하면 Text가 Stack안으로 들어가게 됩니다.

![](../assets/images/swift-ui-wwdc/2.png)

`HStack`안에 `Image`와 `VStack`을 넣어서  좌측에는 이미지 우측에는 2줄의 텍스트를 만들어보겠습니다.

```swift
struct ContentView: View {
    var body: some View {
        HStack {
            Image("swiftui")
            VStack {
                Text("Hello SwiftUI")
                Text("Hello SwiftUI")
            }
        }
    }
}
```

<img src="../assets/images/swift-ui-wwdc/3.png" alt="drawing" width="200"/>  

이미지를 넣었는데 텍스트에 비해 이미지가 너무 크네요.. 이미지 사이즈를 변경하려면 `resizable()` 옵션을 주고 `contentMode`, `frame`을 변경해주면 됩니다.

```swift
struct ContentView: View {
    var body: some View {
        HStack {
            Image("swiftui")
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(width: 50)

            VStack(alignment: .leading) {
                Text("SwiftUI")
                Text("Hello SwiftUI")
                    .foregroundColor(.gray)
            }
        }
    }
}
```
<img src="../assets/images/swift-ui-wwdc/4.png" alt="drawing" width="200"/>  

이미지 사이즈를 변경하고 VStack에 alignment와 텍스트의 Color를 설정했습니다.

## List
---
이제 `TableView`처럼 리스트 형태를 만들어보겟습니다.
좀전에 만들었던 `HStack`을 Cmd클릭한후 메뉴에서 `Embed in List` 선택해주면 간단하게 리스트 형태가 만들어지게 됩니다.
![](../assets/images/swift-ui-wwdc/5.png)
```swift
struct ContentView: View {
    var body: some View {
        List(0 ..< 5) { item in
            Image("swiftui")
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(width: 50)
            
            VStack(alignment: .leading) {
                Text("SwiftUI")
                Text("Hello SwiftUI")
                    .foregroundColor(.gray)
            }
        }
    }
}
```
<img src="../assets/images/swift-ui-wwdc/6.png" alt="drawing" width="300"/>

## List에 데이터 적용
---
실제 데이터를 넣어보기 위해 나라 구조체를 만들고 테스트를 위한 데이터를 미리 준비합니다.
```swift
struct Country: Identifiable {
    var id = UUID()
    var korName: String
    var engName: String
    var hasVideo: Bool = false
    
    var imageName: String { return engName }
}

#if DEBUG
let testData = [
    Country(korName: "대한민국", engName: "South Korea", hasVideo: true),
    Country(korName: "미국", engName: "United States", hasVideo: true),
    Country(korName: "중국", engName: "China", hasVideo: true),
    Country(korName: "일본", engName: "Japan", hasVideo: true),
    Country(korName: "영국", engName: "United Kingdom", hasVideo: true),
    Country(korName: "프랑스", engName: "France", hasVideo: true),
    Country(korName: "이탈리아", engName: "Italy", hasVideo: true),
    Country(korName: "독일", engName: "Germany", hasVideo: true),
]
#endif
```

이제 다시 돌아와 `Country`타입 배열 `countries`을 선언하고 빈배열로 초기화 시켜줍니다. `List`에 `countries`를 넣어주고 이미지와 텍스트에 표시할 각 데이터를 연결해줍니다. 마지막으로 캔버스에서 프리뷰를 보기위해 `testData`를 넣어줍니다.
```swift
struct ContentView: View {
    var countries: [Country] = []
    var body: some View {
        List(countries) { country in
            Image(country.imageName)
            
            VStack(alignment: .leading) {
                Text(country.korName)
                Text(country.engName)
                    .foregroundColor(.secondary)
            }
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView(countries: testData)
    }
}
```
<img src="../assets/images/swift-ui-wwdc/7.png" alt="drawing" width="300"/>

## 네비게이션 추가
---

### 참고자료
>[https://developer.apple.com/kr/xcode/swiftui/](https://developer.apple.com/kr/xcode/swiftui/)
>[https://developer.apple.com/videos/play/wwdc2019/204/](https://developer.apple.com/videos/play/wwdc2019/204/)<br>
