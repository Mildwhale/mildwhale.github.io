---
layout: post
title: "[iOS] Xcode에서 비동기 작업 테스트하기"
description: "비동기 API를 테스트 하기 위한 방법을 소개합니다"
author: "kyujin.kim"
date: 2019-11-22
categories: [iOS, Testing]
tags: [iOS, UI Test, Unit Test]
---

Xcode를 이용한 Test(Unit, UI)를 구현할 때, 비동기 작업이 완료될 때까지 기다려야 하는 상황이 있습니다. 특히, 네트워크 통신이 필요한 요즘의 앱에서는 필수적이죠.

예를 들면, 서버에서 데이터를 받아오는 것을 테스트한다거나, 받아온 데이터를 가공하고 다음 동작을 테스트를 하는 케이스가 있을 것입니다.

이번 글에서는 비동기 API를 테스트하는 방법과, Property의 값 변화를 기다렸다가 처리하는 방법에 대해서 알아보려고 합니다.

## 비동기 API 테스트
APIProvider 클래스의 asyncTask 함수의 호출 결과를 검증하기 위한 간단한 테스트 코드입니다.

<script src="https://gist.github.com/Mildwhale/a874b30f7516dc6aa7381b92961e7da0.js"></script>

하지만 이 테스트는 반드시 실패하게 됩니다.  

왜냐하면 `'asyncTask:completionHandler:'`는 비동기 함수라서 테스트의 마지막 줄에서 Assert 검증 시 resultOfTask의 값은 nil이기 때문입니다. 😫

이런 상황은 [XCTestCase](https://developer.apple.com/documentation/xctest/xctestcase)의 [wait:for:timeout](https://developer.apple.com/documentation/xctest/xctestcase/2806856-wait)함수를 활용하면 간단히 해결할 수 있습니다.

---

<script src="https://gist.github.com/Mildwhale/b4bb3f95b41d3b1d3a7e8dc2418ba6c5.js"></script>

XCTextExpectation을 생성하고(14 line), 대기하기를 원하는 위치에서 wait 함수를 호출(21 line)합니다.  
그리고 비동기 작업이 끝나는 시점에서 expectation의 fulfill(18 line) 함수를 호출하면 됩니다.

테스트 코드는 21번 라인에서 최대 5초 동안 대기하고, 다음 라인으로 넘어가게 됩니다.

이렇게 테스트를 작성하면 비동기 작업이 있는 기능도 쉽게 테스트할 수 있게 됩니다.

## Property 값 변화 기다리기
UITest를 작성하다 보면, UI 컴포넌트의 상태 변화를 기다려야 하는 경우가 있습니다. 예를 들면, 올바른 이메일 주소인지 검증이 되면(비동기) 특정 버튼이 활성화(isEnabled) 되는 상황입니다.

[XCUIElement](https://developer.apple.com/documentation/xctest/xcuielement) 클래스는 [waitForExistence:timeout](https://developer.apple.com/documentation/xctest/xcuielement/2879412-waitforexistence) 함수를 통해, exists 여부를 기다렸다가 처리할 수 있지만, isEnabled는 비슷한 기능이 제공되지 않습니다.

XCTestCase의 wait:for:timeout과 Expectation의 fulfill을 사용하자니, 값이 변하는 시점에 fulfill을 호출해줄 방법이 없습니다.

그렇다면 아래 코드처럼 구현하면 될까요? 아니면 KVO 또는 Observer를 직접 구현하고, wait:for:timeout을 사용하면 될까요? 어떤 방법이든 좋아 보이지는 않습니다.

```swift
func waitForEnabled(element: XCUIElement, timeout: TimeInterval) -> Bool {
    var interval: TimeInterval = 0

    while (element.isEnabled == false) && interval < timeout {
        Thread.sleep(forTimeInterval: 0.1)
        interval += 0.1
    }

    return element.isEnabled
    // (어떻게든 동작은 할 것 같지만, 좀 더 아름다운 방법이 있을 것 같습니다)
}
```

이번에는 [XCTNSPredicateExpectation](https://developer.apple.com/documentation/xctest/xctnspredicateexpectation)와 [XCTWaiter](https://developer.apple.com/documentation/xctest/xctwaiter)를 활용해보겠습니다.

XCTestExpectation과 유사한 XCTNSPredicateExpectation은, Predicate의 조건을 직접 지정해줄 수 있고, fulfill 함수를 직접 호출할 필요가 없습니다.

XCTWaiter는 XCTestCase에서 사용했던 wait:for:timeout 함수를 편리하게 사용할 수 있도록 제공되는 클래스입니다.

<script src="https://gist.github.com/Mildwhale/296cfdc0d6df902b4061a82e2e7883f1.js"></script>

2개의 클래스를 활용해서, XCUIElement의 extension 함수를 몇 개 만들어보았습니다.

그리고, 위에서 만든 함수를 아래와 같이 매우 편리하게 사용할 수 있습니다.

```swift
class MyTest: XCTestCase {
    func testUITest() {
        let app = XCUIApplication()
        let loginButton = app.buttons["로그인"].firstMatch
        
        XCTAssertTrue(loginButton.waitForEnabled(timeout: 3)) // 최대 3초간 대기.
        loginButton.tap()
    }
}
```

## 마치며
이번 글에서는 2개의 Property만 작성했지만, 적당한 Predicate format을 지정하면 다양한 Property의 변화를 감지할 수 있을 것입니다.

최근 들어 테스트 코드를 작성하는 일이 많아졌는데요, 케이스를 작성하고 잘 동작하는 것을 보고 있으면 기분이 매우 좋습니다. 게다가 서비스의 안정성도 확보할 수 있으니 일석이조겠죠?

이번 글은 여기서 마치도록 하겠습니다. 😍