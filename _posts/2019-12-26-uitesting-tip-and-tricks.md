---
layout: post
title: "[iOS] Xcode를 이용한 UI 테스트 - 4"
description: "Tip & Tricks for UI Testing"
author: "kyujin.kim"
date: 2019-12-26
categories: [iOS, Testing]
tags: [iOS, UI Test]
---
UI 테스트용 환경을 구성하려면 어떻게 해야 할까요? 그리고, XCUIElement를 조금 더 쉽고 효율적으로 찾을 수 있는 방법은 없을까요?

이번에는 UI 테스트에서 활용할 수 있는 팁을 소개하려 합니다.  
드디어 `Xcode를 활용한 UI 테스트` 시리즈의 마지막 포스트네요, 마지막 글도 재미있게 봐주세요. 😃

> **이전글 보기**  
> [1. 테스트 프로젝트와 시나리오 소개]({% post_url 2019-11-30-uitest-1-beginning %})  
> [2. 테스트 케이스 작성]({% post_url 2019-12-01-uitest-2-testcase %})  
> [3. 테스트 녹화하기]({% post_url 2019-12-03-uitest-3-record %})  

<br/>

## ProcessInfo로 테스트 환경 구분하기
---
UI 테스트는 실제 앱을 실행하여 진행되기 때문에 다양한 이유로 테스트에 실패할 수 있습니다.  
대표적인 예로 `이전 로그인 상태의 유지` 그리고 `네트워크 타임아웃이 발생하는 경우`가 있습니다.

그럼 XCTest의 setUp 함수에서 로그아웃을 하고, 네트워크 응답을 Stub으로 구현하면 위 문제를 해결할 수 있지 않을까요? 정답은 `불가능`입니다. 왜냐하면 UI 테스트는 [블랙박스 테스트](https://ko.wikipedia.org/wiki/블랙박스_검사)에 해당되기 때문입니다.

다행히도 [ProcessInfo](https://developer.apple.com/documentation/foundation/processinfo)를 활용하면 테스트 환경을 구성할 수 있습니다. 

ProcessInfo는 앱이 실행 중인 프로세스의 정보를 제공하는데, 앱이 실행될 때 입력된 Arguments 정보도 제공하고 있습니다. 그리고 UI 테스트는 XCUIApplication으로 앱을 실행하는데, 이때 [Launch Arguments](https://developer.apple.com/documentation/xctest/xcuiapplication/1500477-launcharguments)를 설정할 수 있으며, 입력된 Arguments는 ProcessInfo의 Arguments를 통해 확인할 수 있습니다.

LaunchArguments와 ProcessInfo을 사용하는 예제 코드를 마지막으로 다음 주제로 넘어가겠습니다.

### 예제 코드
ExampleTest 클래스의 setUp에서 `launchArguments`를 설정하는 코드입니다.
```swift
import XCTest

class ExampleTest: XCTestCase {
    override func setUp() {
        let app = XCUIApplication()
        app.launchArguments.append("UI-TESTING")
        app.launch()
    }
}
```

ProcessInfo의 Arguments를 확인하고 if 구문 사이에 필요한 코드를 작성하면, UI 테스트를 위해 앱이 실행될 때마다 매번 코드가 실행됩니다.
```swift
import UIKit

class AppDelegate: UIResponder, UIApplicationDelegate {
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        if ProcessInfo.processInfo.arguments.contains("UI-TESTING") {
            // do something for UI Test
        }
    }
}
```
<br/>

## UI 테스트에 Accessibility 활용하기
---
UI 테스트는 Accessibility기능을 사용하여 동작합니다. 그래서 Accessibility Identifier를 사용하여 Element를 검색하는 기능을 지원하고 있습니다.

우리는 보통 아래와 같은 방식으로 Element를 찾곤 합니다. 버튼이나 라벨의 텍스트 또는 firstMathces 같은 방식을 이용해서 말이죠.
```swift
let loginButton = app.buttons["로그인"]
XCTAssertTrue(loginButton.exists)
```

하지만, 위와 같은 코드에서 로그인 버튼의 Title이 바뀐다면 어떻게 될까요? 버튼을 찾지 못해서 테스트는 실패하지 않을까요?
다행히도 Accessibitlity Identifier를 사용하면 위와 같은 문제가 발생하지 않고, 잘 구성된 Accessibility Identifier는 앱의 접근성을 높여주는 일석이조의 효과를 얻게 됩니다.

Accessibitlity Identifier는 Interface Builder 또는 코드를 사용하여 설정해줄 수 있고, 찾고자 하는 Element의 Identifier를 대괄호 사이에 입력하면 됩니다. 아래 예제 코드를 한번 보면 이해가 더 쉬울 것 같네요.

### 예제 코드
뷰 컨트롤러의 loginButton에 Accessibility Identifier를 설정해주는 예제 코드입니다.
```swift
import UIKit

final class ViewController: UIViewController {
    
    let loginButton = UIButton()

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        loginButton.accessibilityIdentifier = "loginButton"
    }
}
```

로그인 버튼을 찾기위해 Title값 대신 Identifier를 이용해서 찾는 방법이 적용된 예제입니다.  
변경 가능성이 큰 Title보다 Identifier를 사용하는 게 훨씬 더 안정적인 것 같지 않나요?
```swift
import XCTest

class ExampleTest: XCTestCase {
    override func setUp() {
        let app = XCUIApplication()
        app.launchArguments.append("UI-TESTING")
        app.launch()
    }

    func testLoginButtonExist() {
        // let loginButton = app.buttons["로그인"]
        let loginButton = app.buttons["loginButton"]
        XCTAssertTrue(loginButton.exists)
    }
}
```

<br/>

## 마치며
---
지금까지 총 4개의 글을 통해 UI 테스트에 대해서 알아보았습니다. 아직 걸음마 수준이지만 앞으로 더 나은 테스트를 작성하여 앱의 안정성과 유지보수성을 높일 수 있을 거라 기대합니다.

읽어주셔서 감사합니다! 😃

<br/>

## 참고자료
---
[UI Testing in Xcode - WWDC 2015](https://developer.apple.com/videos/play/wwdc2015/406/)  
[ProcessInfo - Apple](https://developer.apple.com/documentation/foundation/processinfo)  
[XCUIApplication.launchArguments - Apple](https://developer.apple.com/documentation/xctest/xcuiapplication/1500477-launcharguments)  
[Mocking network calls while UI Testing](https://medium.com/@Tovkal/mocking-network-calls-while-ui-testing-61e8a4a07f81)
