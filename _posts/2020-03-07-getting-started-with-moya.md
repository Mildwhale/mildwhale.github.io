---
layout: post
title: "Moya의 사용법을 알아봅시다"
description: "Moya를 사용하여 쉽고 간편하게 네트워킹 구현하기"
author: "kyujin.kim"
date: 2020-03-07
categories: [iOS]
tags: [iOS, Moya, RxMoya]
---

이번 글에서는 iOS의 네트워크 추상화 프레임워크인 [**Moya**](https://github.com/Moya/Moya)에 대해 알아볼 것 입니다.

Moya의 사용법을 소개하기 전에 현재 우리가 어떤 방식으로 네트워킹을 구현하고, 어떤 특징이 있는지 알아보는편이 좋겠죠? (바쁘신분은 스크롤을 내리셔도 됩니다 🥺)

## URLSession & Alamofire
우리는 보통 [URLSession](https://developer.apple.com/documentation/foundation/urlsession) 또는 조금 더 간편한 [Alamofire](https://github.com/Alamofire/Alamofire)를 사용하여 네트워킹을 구현하고 있으며, 높은 확률로 `APIManager` 또는 `NetworkModel`와 같은 이름의 네트워킹 레이어를 만들어 API를 관리하고 있을 것 입니다.

<script src="https://gist.github.com/Mildwhale/05e9992e99e1d031c9c18e4660dda612.js"></script>

이 방식을 보기좋게 그리면 아래와 같은 형태가 될 것입니다. 앱에서 시작하는 화살표들이 눈에띄지 않나요? 어디에서 어떤 방식으로든 네트워킹이 자유롭다는 것을 의미할 수도 있지만, 조금씩 열리는 지옥문의 냄새가 나는군요. 😈 ~~(정말 조금씩 열리지만 언젠간 우리를 괴롭힐겁니다)~~

![image0](/assets/images/getting-started-with-moya/myapplayer.png){: .center-image}{: width="300"}

이런 구현 방식에 대해 Moya에서는 3가지 문제점을 제시하고 있는데요, 모두 유지보수와 관련되어 있으며 실제 업무에서도 자주 겪는 문제입니다.
>Ad hoc network layers are common in iOS apps. They're bad for a few reasons:  
> 1. Makes it hard to write new apps ("where do I begin?")
> 2. Makes it hard to maintain existing apps ("oh my god, this mess...")
> 3. Makes it hard to write unit tests ("how do I do this again?")

전당포 아저씨는 잘생겼으니 오늘만 보고 살아도 되지만, 우리가 이번 버전에서만 잘 동작하도록 개발하면 큰일나겠죠? 🦑🦑🦑🦑🦑

![image1](/assets/images/getting-started-with-moya/oneday.png){: .center-image}{: width="500"}

### 그래서 Moya?
위에서 언급했던 문제들을 개선할 수 있도록, **Moya는 몇 가지 멋진 기능을 가지고 있습니다.**

> Some awesome features of Moya:
> 1. Compile-time checking for correct API endpoint accesses.
> 2. Lets you define a clear usage of different endpoints with associated enum values.
> 3. Treats test stubs as first-class citizens so unit testing is super-easy.

그리고 Moya를 적용하면 아래와 같이 깔끔한 네트워킹 레이어가 구성됩니다. 하지만 이런 의문이 들 수도 있을겁니다. `'내가 만든 APIManager만 쓰면 똑같은거 아니야?'`

![image2](/assets/images/getting-started-with-moya/moyalayer.png){: .center-image}{: width="200"}
 
물론 그렇게 해도 됩니다! 그런데 굳이 바퀴를 재발명할 필요가 있을까요? 이미 훌륭한 바퀴가 만들어져 있으니, 이 바퀴로 개발 시간도 단축하고 유지보수성과 안정성 이라는 이득을 챙기는 편이 더 낫지 않을까요? 🧐

## Moya 사용해보기

---

오늘 우리의 목표는, Github의 [Search API](https://developer.github.com/v3/search/#search-users)호출을 Moya로 구현하고, 테이블뷰에 보여주는 것 입니다.

맛보기에 사용 할 프로젝트는 [이곳](https://github.com/Mildwhale/Getting-Started-With-Moya)에서 다운받을 수 있습니다. `Starter`프로젝트로 직접 따라해볼 수도 있고, `Finished`프로젝트에서 완성된 프로젝트를 참고할 수도 있습니다.

프로젝트는 아래와 같은 환경에서 만들어졌습니다.

> * Xcode 11.3.1
> * Swift 5.1
> * iOS 13.x

### Carthage 설정하기
프로젝트를 실행해보기 전에 외부 프레임워크를 다운받아야 합니다. 만약 Carthage 설정이 필요하다면 [이곳](https://github.com/Carthage/Carthage#installing-carthage)을 참고해주세요.

1. 터미널을 열고 다운받은 프로젝트의 `Starter`폴더로 이동합니다.
2. `carthage update --platform ios --no-use-binaries --cache-builds` 명령어를 실행합니다.
3. Build가 완료될 때 까지 기다립니다.

### 프로젝트 살펴보기
프로젝트는 Moya를 제외한 모든 개발 및 설정이 완료되어 있는 상태이기 때문에, 추가적인 코딩 및 설정이 필요하지 않습니다. 왜냐하면 오늘은 Moya의 사용법만 알아 볼 것이기 때문이죠. 😎

먼저, `NetworkingHelper` 폴더의 `GithubAPI.swift`파일을 열어보면 아무것도 없는 것을 볼 수 있습니다. 우리는 Moya가 제공하는 Interface를 이용하여, 이 파일에 Github API에 대한 정보를 작성 할 계획입니다.

다음으로, `ViewController.swift`파일을 열어봅시다. 106 라인부터 `search`로 시작하는 함수의 인터페이스가 구현되어 있습니다. 이 함수에 Moya를 이용한 API request를 구현 할 계획입니다.

### API 작성하기
`GithubAPI.swift`파일에 GithubAPI enum과 searchUser라는 case를 추가합니다.

<script src="https://gist.github.com/Mildwhale/d4861d4daab119965afa8f1c81605b7a.js"></script>

[Github API v3 - Search user](https://developer.github.com/v3/search/#search-users)를 enum으로 추상화 했습니다. 그럼 GithubAPI를 Moya에서 바로 사용할 수 있을까요? 그렇지 않습니다. Moya는 [MoyaProvider](https://moya.github.io/Classes/MoyaProvider.html)로 request를 수행하는데, 이때 사용하는 파라미터는 [TargetType](https://moya.github.io/Protocols/TargetType.html)이라는 프로토콜을 따릅니다.

그럼 우리는 GithubAPI가 TargetType 프로토콜을 따르도록 구현해주어야겠죠? GithubAPI.swift 파일에 아래 코드를 추가합니다.

<script src="https://gist.github.com/Mildwhale/20858301629b0e2096caeef3a386655d.js"></script>

TargetType 프로토콜의 property는 각각 아래와 같은 역할을 담당합니다.
> * **baseURL**: 서버의 도메인
> * **path**: 서버의 도메인 뒤에 추가 될 Path (일반적으로 API)
> * **method**: HTTP method (GET, POST, ...)
> * **sampleData**: 테스트용 Mock Data
> * **task**: 리퀘스트에 사용되는 파라미터 설정
> * **validationType**: 허용할 response의 타입
> * **headers**: HTTP header

여기까지 했으면 request에 필요한 준비는 모두 끝났습니다. 매우 간단하죠? 👏

### Request 호출하기
이제 거의 다 됐습니다. `ViewController.swift` 파일을 열고 106 라인을 보면 search 함수의 인터페이스가 구현되어 있습니다. `searchWithRx`함수 내부에 아래 코드를 채워줍니다.

<script src="https://gist.github.com/Mildwhale/2ca483d60f3ffa44962aec7782725dcd.js"></script>

MoyaProvider를 생성할 때 generic 타입으로 GithubAPI를 지정해주는 것을 볼 수 있습니다. 그리고 바로 아래에서는 request 함수의 파라미터로 `searchUser`케이스를 넣어주고 있네요!

### 결과 감상하기
놀랍게도 모든 준비가 끝났고, 결과를 감상할 차례입니다. 앱을 실행하면 아주 `Gorgeous한 UI`가 눈앞에 펼쳐지고, SearchBar에 문자를 입력하면 Github API를 호출하여 사용자의 정보를 가져와 테이블뷰에 보여주는 것을 볼 수 있습니다. 😅

서론만 긴 글이 된 것 같아 뭔가 허전하지만, 이번 글은 여기서 마치겠습니다!  

응원의 댓글과 의견은 언제든 환영입니다. 😍

<br>

## 참고자료

---

* [Moya - Github Repository](https://github.com/Moya/Moya)
* [Moya - Reference](https://moya.github.io/index.html)
* [Github API v3 - Search](https://developer.github.com/v3/search)