---
layout: post
title: "[iOS] Xcode를 이용한 UI 테스트 - UI Recording"
author: "kyujin.kim"
date: 2019-12-05
categories: [iOS, Testing]
tags: [iOS, UI Test]
---

이번 글은 UI 테스트 코드를 조금 더 쉽게 작성할 수 있는 방법인 **UI Recording**에 대해 알아보려 합니다.

테스트에 사용한 프로젝트는 [**Github**](https://github.com/Mildwhale/Github-SearchUser-Tutorial)에서 다운받으실 수 있습니다.

## UI Recording?
UI Recording은 사용자의 기기, 시뮬레이터, macOS의 유저 인터렉션을 코드로 생성해주는 기능을 지원합니다.

이 기능을 활용하면 UI 테스트를 위한 테스트 코드를 만들거나, 기존의 테스트를 확장하는 데 큰 도움이 됩니다. 🤩

그럼 Xcode를 사용해서 UI Recording 하는 방법을 알아보겠습니다!

## Xcode에서 녹화하기
iOS 프로젝트의 UI Recording을 하는 방법은 매우 간단합니다. 

우선 시뮬레이터로 앱을 실행한 뒤 테스트 코드가 들어가야 할 위치를 선택합니다. 그리고 디버그 Breakpoint 버튼 왼쪽의 빨간색 녹화 버튼을 누릅니다.

![image1](/assets/images/ios-ui-recording/img1.png){: .center-image}

녹화 모드가 시작되고 시뮬레이터로 앱을 사용해보시면, 내가 실행하는 동작이 테스트 코드로 변환되는것을 볼 수 있습니다.

녹화를 중지하고 싶으면 Xcode에서 아까 눌렀던 녹화 버튼을 한 번 더 누르면 됩니다.

![image2](/assets/images/ios-ui-recording/img2.png){: .center-image}

## 리펙토링
위에서 녹화한 테스트 코드는 `SearchBar를 탭하고 'Q'를 입력`하는 코드입니다. 그런데 UI Recording을 통해 생성된 테스트 코드는 어딘가 매우 불편합니다. 그대로 사용해도 문제는 없지만 리펙토링 통해 조금 더 보기 좋고, 효율적인 코드를 만들 수 있을 것 같습니다.

SearchBar의 TextField는 [XCUIElement.searchField](https://developer.apple.com/documentation/xctest/xcuielementtypequeryprovider/1500393-searchfields)를 통해 찾아낼 수 있습니다.

```swift
let searchField = app.searchField.firstMatch
```

문자 Q를 입력하기 위해서 app의 keys를 찾아 탭 하지 않고, searchField의 [XCUIElement.typeText](https://developer.apple.com/documentation/xctest/xcuielement/1500968-typetext) 함수를 사용하여 입력 할 수 있습니다.

```swift
searchField.typeText("Q")
```

간단한 리펙토링을 거치면 약 10줄이 넘던 코드가 2줄에 완성되는 마법(?)을 볼 수 있습니다.

```swift
let searchField = app.searchField.firstMatch
searchField.typeText("Q")
```

## 마치며
이번 글에서는 UI Recording을 통해 테스트 코드를 손쉽게 생성할 수 있는 방법에 대해 알아보았습니다.

UI 테스트 코드 작성에 익숙하지 않을 때 매우 유용한 방법이라 생각하지만, **리펙토링이 필요하고 테스트를 검증하기 위한 코드는 직접 추가**해야 하는 불편함이 있습니다.

그래도 UI 테스트에 입문하는 분들에겐 매우 유용한 기능임에 틀림없습니다!


---
읽어주셔서 감사합니다 🙃


## 참고문서
[Recording UI Tests](https://developer.apple.com/library/archive/documentation/ToolsLanguages/Conceptual/Xcode_Overview/RecordingUITests.html)

[UI Testing in Xcode](https://developer.apple.com/videos/play/wwdc2015/406)