---
layout: post
title:  "[iOS] Xcode11 더 간편해진 UIScrollView 만들기"
subtitle: "contentLayoutGuide, frameLayoutGuide 를 이용한 스크롤뷰 레이아웃 구성"
author: Sung Kyungmo
catalog: true
tags: [ios, xcode11, xcode, uiscrollview, scrollview, contentLayoutGuide, frameLayoutGuide]
comments: true
header-mask:  0.6
---
Xcode 11 버전에서 인터페이스 빌더로 스크롤뷰를 만들때 `contentLayoutGuide`, `frameLayoutGuide`가 기본적으로 활성화 되도록 추가되었습니다.

## Xcode 11 Release Notes
- Interface Builder
>Content and Frame Layout guides are supported for UIScrollView and can be enabled in the Size inspector for more control over your scrollable content. (29711618)

릴리즈 노트를 보면 content 및 frame 레이아웃 가이드는 UIScrollView에서 지원되며, 스크롤 가능한 컨텐츠를 보다 효과적으로 제어할 수 있도록 Size inspector에서 활성화할 수 있다고 나와있습니다.
![](/img/xcode-scrollview/0.png)

Size inspector에서 확인해보면 `Content Layout Guides` 체크박스가 추가되어있습니다.
스크롤뷰 생성시 기본적으로 활성화 되어있기 때문에 따로 체크할 필요는 없습니다.

## contentLayoutGuide
`ScrollView`의 변환되지 않은 컨텐츠 사각형을 기반으로 하는 레이아웃 가이드.
- `ScrollView`의 컨텐츠 영역과 관련된 오토 레이아웃 제약 조건을 만드려면 이 레이아웃 가이드를 사용하세요.

> ![](/img/xcode-scrollview/contentLayoutGuide.png)

## frameLayoutGuide
`ScrollView`의 변환되지 않은 프레임 사각형을 기반으로 하는 레이아웃 가이드.
- 컨텐츠 사각형 반대로 `ScrollView` 자체의 프레임 사각형을 포함하는 오토 레이아웃 제약 조건을 만드려면 이 레이아웃 가이드를 사용하세요.

> ![](/img/xcode-scrollview/frameLayoutGuide.png)

# Interface Builder로 ScrollView 만들기

## ScrollView 추가

`ScrollView`를 추가하고 화살표를 눌러 `ContentLayoutGuide`와 `FrameLayoutGuide`가 포함되어있는걸 확인할 수 있습니다.

![](/img/xcode-scrollview/1.png)

## ScrollView Constraints 설정

`ScrollView`를 추가했으니 알맞게 제약을 설정해줍니다.  
저는 화면을 꽉 채우기 위해 `ScrollView`를 `Safe Area`에 맞췄습니다.

![](/img/xcode-scrollview/2.png)

## Content Layout Guide 설정

`ScrollView`안에 컨텐츠를 넣기 위한 상위 `View` 하나를 추가해줍니다.  
이때 `View`의 제약조건을 `ScrollView`에 걸지 않고 `Content Layout Guide`에 걸어줍니다.

![](/img/xcode-scrollview/3.png)

## Frame Layout Guide 설정

이제 `Frame Layout Guide` 에 `Equal Widths` 나 `Equal Heights` 제약을 추가해 고정시킬 방향을 설정합니다.
- `Equal Widths`는 가로길이를 `Frame Layout Guide`에 고정시켜 **세로 스크롤**이 필요할때 사용합니다.
- `Equal Heights`는 세로길이를 `Frame Layout Guide`에 고정시켜 **가로 스크롤**이 필요할때 사용합니다.

![](/img/xcode-scrollview/4.png)

## 컨텐츠 추가
마지막으로 `View`에 필요한 UI를 작성합니다.
저는 단순히 스크롤을 테스트하기 위해 `Label`을 5개 추가했습니다.
여기서 중요한 점은 `View`의 `Heigth`가 명확해야 하므로 상단부터 하단까지의 제약을 추가합니다.

![](/img/xcode-scrollview/5.png)

## 마무리
이제 스크롤이 완성되었습니다. 앱을 실행하게 되면 아래와같이 스크롤이 되는것을 확인할 수 있습니다.

![](/img/xcode-scrollview/scroll.gif)

기존엔 인터페이스 빌더로 `ScrollView`를 만들때 제약조건이 복잡했지만, Xcode 11에 `Content Layout Guide`와 `Frame Layout Guide`가 추가되면서 좀 더 명확하고 간결하게 `ScrollView`를 만들 수 있게 되었습니다.


## Reference
--- 
> [https://developer.apple.com/documentation/uikit/uiscrollview](https://developer.apple.com/documentation/uikit/uiscrollview)  
> [https://developer.apple.com/documentation/xcode_release_notes/xcode_11_release_notes]( https://developer.apple.com/documentation/xcode_release_notes/xcode_11_release_notes)
