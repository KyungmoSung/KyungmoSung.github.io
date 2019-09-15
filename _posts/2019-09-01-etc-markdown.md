---
layout: post
title:  "[Markdown] 마크다운이란 무엇인가? 마크다운 문법 작성 방법"
author: sung
categories: [ etc ]
tags: [ markdown, md, html, readme, blog ]
image: assets/images/190901_markdown.png
comments: true
---

> 마크다운(markdown)은 일반 텍스트 문서의 양식을 편집하는 문법이다. README 파일이나 온라인 문서, 혹은 일반 텍스트 편집기로 문서 양식을 편집할 때 쓰인다. 마크다운을 이용해 작성된 문서는 쉽게 HTML 등 다른 문서형태로 변환이 가능하다.

Github에서 Repository를 만들어보신 분 이라면 한번쯤 README.md 파일을 보셨을겁니다.<br>
이 파일이 바로 마크다운 문법으로 작성된 파일인데 확장자인 .md가 바로 Markdown의 약자입니다.<br>
저도 마크다운 문법을 몇번 작성해보긴 했지만 평소에 잘 사용하지 않다보니 블로그를 만드는 김에 사용법을 한번 정리해보려고 합니다.<br>
문법과 사용법이 간단하기 때문에 한번 보시면 문서를 작성하는데 어려움이 없을거라고 생각됩니다.<br>

### 장점
* 문법이 간결하다
* 관리가 쉽다
* 별도의 도구없이 작성가능하다.
* 다양한 형태로 변환이 가능하다.
* 텍스트로 저장되기 때문에 용량이 적어 보관이 용이하다
* 지원 가능한 플랫폼과 프로그램이 다양하다.

### 단점
* 표준이 없어 사용자마다 문법이 상이할 수 있다.
* 모든 HTML 마크업을 대신하지 못한다.

***
<br>

## 마크다운 문법

### 제목(Header)
---
`<H1>` 부터 `<H6>` 까지 표현 가능
```
Header <H1>
===========
```
```
Header <H2>
-----------
```
```
# Header <H1>
## Header <H2> 
### Header <H3>
#### Header <H4>
##### Header <H5>
###### Header <H6>
```

# Header <H1>
## Header <H2>
### Header <H3>
#### Header <H4>
##### Header <H5>
###### Header <H6>

<br>
### 강조(Emphasis)
---
```
*이탤릭체(single asterisks)*
_이탤릭체(single underscores)_
**볼드체(double asterisks)**
__볼드체(double underscores)__
<u>밑줄(underline)</u>
~~취소선(tilde)~~
```

*이탤릭체(single asterisks)*<br>
_이탤릭체(single underscores)_<br>
**볼드체(double asterisks)**<br>
__볼드체(double underscores)__<br>
<u>밑줄(underline)</u><br>
~~취소선(tilde)~~

<br>

### 목록(List)
---
##### 순서있는 목록
```
1. List1
2. List2
3. List3
    1. List3-1
    2. List3-2
```
1. List1
2. List2
3. List3
    1. List3-1
    2. List3-2
<br>

##### 순서없는 목록
```
* List
* List
* List
    * List
    * List
```
* List
* List
* List
    * List
    * List
    
<br>

### 인용(BlockQuote)
---
```
> blockquote1
>> blockquote2
>>> blockquote3
```
> blockquote1
>> blockquote2
>>> blockquote3

<br>

### 코드(Code)
---
##### 인라인(inline) 코드
```
This is `code`
```

This is `code`

##### 블록(block) 코드
4개의 공백 또는 하나의 탭으로 들여쓰기를 만나면 변환되기 시작하여 들여쓰지 않은 행을 만날때까지 변환이 계속된다.

```
    ```swift
        let foo = "Hello Wordl!"
    ```
```
```swift
let foo = "Hello Wordl!"
```
<br>

### 링크(Link)
---
```
[1]: https://github.com

[GitHub][1]

[Google](https://google.com)

[Naver](https://naver.com "naver.com")
```
[1]: https://github.com

[GitHub][1]

[Google](https://google.com)

[Naver](https://naver.com "naver.com")


<br>

### 이미지(Images)
---
사이즈 조절은 `<img width="" height=""></img>` 사용
```
![Alt text](/assets/images/1.jpg)
![Alt text](/assets/images/2.jpg "title")
```
![Alt text](/assets/images/1.jpg)
![Alt text](/assets/images/2.jpg "title")

<br>

##### 이미지 링크
```
[![Alt text](/assets/images/3.jpg)](https://github.com/KyungmoSung)
```
[![Alt text](/assets/images/3.jpg)](https://github.com/KyungmoSung)

<br>

### 표(Table)
---
헤더 셀을 구분할 때 3개 이상의 -(hyphen/dash) 기호가 필요<br>
헤더 셀을 구분하면서 :(Colons) 기호로 셀(열/칸) 안에 내용을 정렬<br>
가장 좌측과 가장 우측에 있는 |(vertical bar) 기호는 생략 가능<br>
```
| A | B | C |
|:---|:---:|---:|
| `row1` | aaa | `111` |
| `row2` | bbb |  |
| `row3` | ccc |  |
| `row4` | ddd |  |
```

| A | B | C |
|:---|:---:|---:|
| `row1` | aaa | `111` |
| `row2` | bbb |  |
| `row3` | ccc |  |
| `row4` | ddd |  |

<br>

### 수평선(Horizontal Rule)
---
각 기호를 3개 이상 입력, 페이지 나누기 용도로 많이 사용
```
---
***
___
```
---
***
___

<br>

### 줄바꿈(Line Breaks)
---
띄어쓰기 2번이나 `<br>`사용

<br>
### 참고자료
>https://ko.wikipedia.org/wiki/마크다운
>https://gist.github.com/ihoneymon/652be052a0727ad59601
>https://heropy.blog/2017/09/30/markdown/
