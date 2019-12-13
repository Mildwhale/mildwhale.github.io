---
layout: post
title: "[iOS] Xcode를 이용한 UI 테스트 - 2"
subtitle: "테스트 케이스 작성"
author: "kyujin.kim"
date: 2019-12-01
categories: [iOS, Testing]
tags: [iOS, UI Test]
---

이번 글에서는 iOS 앱의 UI 테스트 케이스를 어떻게 작성하는지 알아보려 합니다. 이전 글과 이어지는 내용이므로 아직 이전 글을 보지 않으셨다면 아래 링크를 눌러 이전 글을 먼저 보는 것을 추천합니다.

[[iOS] Xcode를 이용한 UI 테스트 - 1](http://mildwhale.github.io/2019-11-30-uitest-1-beginning/)

이제 미리 준비된 [프로젝트](https://github.com/Mildwhale/Github-SearchUser-Tutorial)를 살펴보고, 테스트 케이스 작성 전에 알아두면 좋은 기본기 소개부터 시작하겠습니다.

## 테스트 살펴보기
Starter 폴더에 있는 프로젝트를 실행하고 `UITest_SearchUser.swift` 파일을 열면 테스트 케이스와 시나리오 템플릿이 준비되어 있습니다.

`Command+R` 단축키로 앱을 실행해서 앱이 어떤 식으로 동작하는지 먼저 확인해 보면 테스트 작성에 도움이 될 겁니다.

```swift
class UITest_SearchUser: XCTestCase {

    let app = XCUIApplication() // 테스트를 위한 UIApplication
    
    override func setUp() {
        continueAfterFailure = false
        app.launch() // 테스트 앱 실행
    }

    override func tearDown() {
        
    }
    
    .
    .
    .
}
```

## XCTestCase
Xcode에서 사용하는 테스트 클래스의 기반이 되는 [XCTestCase](https://developer.apple.com/documentation/xctest/xctestcase)는 테스트에 필요한 기본적인 기능을 제공하기 때문에, 모든 테스트 클래스는 XCTestCase를 상속받아 구현해야 합니다.

### setUp & tearDown
테스트 전과 후에 필요한 작업을 실행할 수 있는 기회가 제공되는 [setUp](https://developer.apple.com/documentation/xctest/xctestcase/1496262-setup)과 [tearDown](https://developer.apple.com/documentation/xctest/xctestcase/1496280-teardown) 함수는 아래와 같은 순서로 호출됩니다.

<center>setUp() → test_function() → tearDown()</center>

UI 테스트를 위해서는 [XCUIApplication](https://developer.apple.com/documentation/xctest/xcuiapplication)의 [launch](https://developer.apple.com/documentation/xctest/xcuiapplication/1500467-launch) 함수를 호출하여 앱을 실행해주어야 하는데, setUp 에서 호출해주면 테스트 함수가 실행되기 직전에 매번 앱이 실행됩니다.

### XCUIElement
[XCUIElement](https://developer.apple.com/documentation/xctest/xcuielement)는 UI 테스트에서 UIButton 또는 UILabel 등의 컴포넌트를 대신하는 객체입니다.

UI 테스트는 사람이 실제로 앱을 사용하는 것처럼 현재 실행 중인 앱에서 필요한 UI 컴포넌트를 찾아 Tap을 하거나, 텍스트를 입력하고 스크롤을 하는 등의 동작을 구현해야 합니다.

Element 존재 여부인 [exists](https://developer.apple.com/documentation/xctest/xcuielement/1500760-exists), 터치를 위한 [tap](https://developer.apple.com/documentation/xctest/xcuielement/1618666-tap), 텍스트 입력을 위한 [typeText](https://developer.apple.com/documentation/xctest/xcuielement/1500968-typetext) 함수가 주로 사용하는 XCUIElement의 대표적인 Property 및 함수입니다.

### XCUIElementQuery
화면에 그려진 XCUIElement를 찾기 위해서는 [XCUIElementQuery](https://developer.apple.com/documentation/xctest/xcuielementquery)를 사용할 수 있는데, 클래스 이름처럼 Query를 통해 현재 앱에 보이고 있는 Element를 찾을 수 있습니다.

컴포넌트를 찾는 원리는 매우 간단한데요, 왜냐하면 뷰는 트리 형태로 구성되어 있기 때문이죠.  
아래 그림을 참고하면 Element를 찾는 방식을 쉽게 이해할 수 있습니다.

<center><img src="/assets/images/test-with-xcode/img2.png"></center>

위와 관련된 내용을 좀 더 알고 싶다면, [[WWDC 2015] UI Testing in Xcode](https://developer.apple.com/videos/play/wwdc2015/406/) 영상이 매우 큰 도움이 됩니다.

### Test함수 이름 짓기 (Naming Convention)
테스트를 위한 함수는 'test'라는 단어로 시작해야 한다는 [규칙](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/testing_with_xcode/chapters/04-writing_tests.html)을 가지고 있습니다.

XCTestCase를 상속받은 테스트 클래스에 `'test'`로 시작하는 함수는 모두 테스트 함수로 인식되기 때문에 test 이후 아무렇게나 작성해도 되지만, 보통은 `'테스트하려는 기능 또는 목적'`을 뒤에 붙여주는 형태로 이름을 짓습니다.

잘 만들어진 테스트 함수는 이름만 봐도 이 테스트가 어떤 목적을 가지고 있는지 알 수 있으나, 반대의 경우에는 테스트 코드를 모두 살펴봐야 하는 최악의 상황이 발생할 수 있습니다.

<center><img src="/assets/images/test-with-xcode/img3.jpeg"></center>

### 테스트 작성하기
Starter 프로젝트에 미리 정의되어 있는 첫 번째 테스트 함수의 이름은 `'test_SearchUserResultAvailable'`인데, 이름을 보면 사용자를 검색하고 결과물의 유무를 확인하는 테스트라고 추측할 수 있습니다.

이전 글에서 작성했던 테스트 시나리오를 한번 살펴보시죠. 

> *Case 1. 검색어를 입력하면, 검색 결과를 확인할 수 있는가?*  
> 1. SearchBar의 TextField를 탭  
> 2. 검색어 입력  
> 3. 검색 결과가 있는지 확인

첫 번째 시나리오를 작성하기 위해서는 앱에서 SearchBar Element를 찾아야 합니다.

```swift
let searchField = app.searchFields.firstMatch
```

app(XCUIApplication Element)에서 searchFields라는 Query로 Element를 찾은 뒤, 매칭 되는 첫 번째 Element를 가져오기 위해 위의 코드를 추가합니다.

그리고 SearchBar Element를 제대로 찾았는지 검사하면, 조금 더 안정적인 테스트를 만들 수 있겠죠?

```swift
XCTAssertTrue(searchField.exists)
```

exists 값을 통해 Element의 유무를 검증할 수 있는데, 이때 사용하는 [XCTAssert](https://developer.apple.com/documentation/xctest/boolean_assertions)는 다양한 기능을 제공하고 있으니 꼭 한번 살펴보는 것을 추천합니다.

이제 위에서 찾은 SearchBar를 터치하고, 텍스트를 입력할 차례입니다.

```swift
searchField.tap() // 1. SearchBar의 TextField를 탭
searchField.typeText("Mildwhale\n") // 2. 검색어 입력 후 키보드 내림
```

위 코드를 실행하면 searchField를 손으로 터치하는 것과 같은 동작을 실행하고, typeText 함수를 통해 원하는 텍스트를 입력하게 됩니다.

마지막으로 검색 결과를 잘 받아왔는지 확인해볼 차례입니다.

```swift
let resultCellOfFirst = app.cells.firstMatch
// XCTAssertTrue(resultCellOfFirst.exists) // 이렇게 검증하면 크래시가 발생합니다.
XCTAssertTrue(resultCellOfFirst.waitForExistence(timeout: 15.0)) // 3. 검색 결과가 있는지 확인 (최대 15초 동안 대기)
```

앱에 보이는 모든 테이블 셀 중, 첫 번째 셀을 가져오고 exists 상태인지의 여부를 확인하는 코드입니다.

이번에는 검색어가 입력되면 Github 서버의 응답을 받을 때까지 대기했다가, 셀의 exists 상태를 확인할 필요가 있습니다. 왜냐하면 검색 결과는 비동기 통신을 통해 받아오기 때문이죠.

이때 사용하는 함수는 [waitForExistence:timeout](https://developer.apple.com/documentation/xctest/xcuielement/2879412-waitforexistence) 이며, Element가 exists 상태로 변할 때까지 또는 timeout에 입력 한 시간만큼 대기하는 기능을 제공합니다.

이러한 방법은 비동기 작업의 결과를 검증해야 할 때 자주 사용되며, 관련 내용은 [이전 글](http://mildwhale.github.io/2019-11-22-async-test/)에서 확인하실 수 있습니다.

### 테스트 실행하기

놀랍게도 몇 줄의 코드를 추가하니 하나의 케이스가 완성되었습니다! 이제 테스트가 잘 동작하는지 확인해 볼 차례입니다.

```swift
class UITest_SearchUser: XCTestCase {

    let app = XCUIApplication()
    
    override func setUp() {
        continueAfterFailure = false
        app.launch() // 앱 실행
    }

    override func tearDown() {
        
    }
    
    /// 검색어를 입력하면, 검색 결과를 확인할 수 있는가?
    func test_SearchUserResultAvailable() {
        let searchField = app.searchFields.firstMatch
        XCTAssertTrue(searchField.exists) // 검색창의 TextField가 있는지 확인
        
        searchField.tap() // 1. SearchBar의 TextField를 탭
        searchField.typeText("Mildwhale\n") // 2. 검색어 입력 후 키보드 내림
        
        let resultCellOfFirst = app.cells.firstMatch
        XCTAssertTrue(resultCellOfFirst.waitForExistence(timeout: 15.0)) // 3. 검색 결과가 있는지 확인 (최대 15초 동안 대기)
    }
}
```

프로젝트에서 테스트하려는 시뮬레이터를 선택하고, `Command + U`를 누르면 시뮬레이터가 실행되면서 테스트가 진행됩니다.

<center><img src="/assets/images/test-with-xcode/img4.gif"></center>

---

## 마치며

우리는 총 4개의 테스트 중 1개를 함께 완성했습니다. 테스트 코드 작성에 익숙해지기 위해 나머지 3개의 케이스는 직접 구현해보는 것을 추천합니다. 중간에 막히는 부분이 있다면 레퍼런스 문서 또는 Final 프로젝트에 구현된 테스트 코드를 참고해주세요.

다음 글에서는 테스트를 좀 더 쉽고 편리하게 만들고, 테스트를 최적화하는 방법 등 테스트 꿀팁을 소개할 예정입니다.



## 참고자료
- [UI Testing in Xcode (WWDC 2015)](https://developer.apple.com/videos/play/wwdc2015/406/)