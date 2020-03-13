---
layout: post
title: "[SwiftUI] UIKit과 인터페이스 연결하기"
description: "SwiftUI에서 UIPageViewController 사용하기"
author: "kyujin.kim"
date: 2020-03-12
categories: [iOS, SwiftUI]
tags: [iOS, SwiftUI]
---

> SwiftUI는 모든 Apple 플랫폼의 기존 UI 프레임워크들과 함께 완벽하게 동작합니다. 예를 들어, UIKit의 뷰와 뷰 컨트롤러들을 SwiftUI의 뷰 안에 배치할 수 있고, 그 반대도 가능합니다.  
> [본문 링크](https://developer.apple.com/tutorials/swiftui/interfacing-with-uikit#track-the-page-in-a-swiftui-views-state)

이번 글에서는 **SwiftUI**로 페이징이 되는 **View**를 만들어 볼 것입니다. **UIPageViewController**와 **UIPageControl**을 사용할 것이며, **@State**와 **@Binding**을 사용하여 데이터를 업데이트하는 방법에 대해서도 알아볼 것 입니다.

샘플 프로젝트는 [이곳](https://docs-assets.developer.apple.com/published/0a3f2fc604/InterfacingWithUIKit.zip)에서 다운받으실 수 있습니다.  
_그리고, 이 글은 **Swift Tutorials**의 [Interfacing with UIKit](https://developer.apple.com/tutorials/swiftui/interfacing-with-uikit#track-the-page-in-a-swiftui-views-state)을 기반으로 작성되었습니다._

---

## 1. UIPageViewController를 대신하는 View 만들기
![section1-1](/assets/images/swift-tutorials/interfacing-with-uikit/section1-1.png){: .center-image}{: width="300"}

SwiftUI에서 UIKit의 뷰와 뷰 컨트롤러를 사용하기 위해서는, **UIViewRepresentable**과 **UIViewControllerRepresentable** 프로토콜을 따르는 타입을 만들어야 합니다. 여기서 만들어진 타입은 UIKit을 생성하고 설정할 수 있으며, SwiftUI는 이 타입의 생명주기를 관리하고 필요시 업데이트를 수행합니다.

### PageViewController 만들기
프로젝트를 열고, `PageViewController.swift`라는 이름의 SwiftUI 파일을 생성합니다. 그리고 UIViewControllerRepresentable을 따르는 PageViewController를 정의합니다.

```swift
import UIKit
import SwiftUI

struct PageViewController: UIViewControllerRepresentable {
    var controllers: [UIViewController]
}
``` 

이 PageViewController는 UIViewController 인스턴스의 배열을 가지고 있습니다. 이 배열은 나중에 UIPageViewController의 DataSource로 사용 할 예정입니다.

### UIViewControllerRepresentable 프로토콜 구현하기
UIViewControllerRepresentable 프로토콜은 필수적으로 2개의 메서드를 구현해야 합니다. 

첫 번째로 [makeUIViewController(context:)](https://developer.apple.com/documentation/swiftui/uiviewcontrollerrepresentable/3278034-makeuiviewcontroller) 메서드를 PageViewController의 추가합니다. 이 메서드에서는 UIPageViewController를 생성하고 설정할 수 있습니다.

SwiftUI는 View가 화면에 보여질 준비가 되었을 때 이 메서드를 단 한번만 호출하며, 뷰 컨트롤러의 생명주기를 관리합니다.

```swift
func makeUIViewController(context: Context) -> UIPageViewController {
    let pageViewController = UIPageViewController(
        transitionStyle: .scroll,
        navigationOrientation: .horizontal)

    return pageViewController
}
```

두 번째로 [updateUIViewController(_:context:)](https://developer.apple.com/documentation/swiftui/uiviewcontrollerrepresentable/3278035-updateuiviewcontroller) 메서드를 추가합니다. 이 메서드에서는 UIPageViewController에서 보여지는 뷰 컨트롤러를 설정하기 위해 **setViewControllers(_:direction:animated:)**를 호출하도록 되어 있습니다.

```swift
func updateUIViewController(_ pageViewController: UIPageViewController, context: Context) {
    pageViewController.setViewControllers(
        [controllers[0]], direction: .forward, animated: true)
}
```

updateUIViewController(_:context:) 메서드는 뷰 컨트롤러에 영향을 미치는 변화가 발생했을 때 호출되며, 이곳에서 Context의 정보로 뷰 컨트롤러를 업데이트 해야 합니다. 이 부분은 조금 뒤에 Coordinator와 함께 알아 볼 예정입니다.

모든 메서드를 추가하면 PageViewController.swift는 아래와 같은 모습을 갖추게 됩니다.
```swift
import UIKit
import SwiftUI

struct PageViewController: UIViewControllerRepresentable {
    var controllers: [UIViewController]

    func makeUIViewController(context: Context) -> UIPageViewController {
        let pageViewController = UIPageViewController(
            transitionStyle: .scroll,
            navigationOrientation: .horizontal)

        return pageViewController
    }

    func updateUIViewController(_ pageViewController: UIPageViewController, context: Context) {
        pageViewController.setViewControllers(
            [controllers[0]], direction: .forward, animated: true)
    }
}
```

### PageViewController를 사용하는 PageView 만들기
지금까지 UIPageViewController를 **대신하는** PageViewController를 만들었습니다. 그럼 이제 이 PageViewController를 사용하는 뷰를 만들 차례겠죠?

우선, `PageView.swift`라는 이름의 새로운 SwiftUI 파일을 생성하고, 아래 코드를 작성합니다.

Generic으로 정의된 **Page** 타입을 [UIHostingController](https://developer.apple.com/documentation/swiftui/uihostingcontroller)에 설정해주고, 이니셜라이저의 파라미터로 사용하는 것을 볼 수 있습니다. 다음으로, `body`를 보면 아까 위에서 만들었던 PageViewController를 Child View로 사용하고 있고, 이니셜라이저에서 설정한 `viewControllers`를 넘겨주고 있습니다.

```swift
import SwiftUI

struct PageView<Page: View>: View {
    var viewControllers: [UIHostingController<Page>]

    init(_ views: [Page]) {
        self.viewControllers = views.map { UIHostingController(rootView: $0) }
    }

    var body: some View {
        PageViewController(controllers: viewControllers)
    }
}

struct PageView_Previews: PreviewProvider {
    static var previews: some View {
        PageView(features.map { FeatureCard(landmark: $0) })
            .aspectRatio(3/2, contentMode: .fit)
    }
}
```

_여기서 사용한 UIHostingController는 UIViewController의 서브 클래스이며, SwiftUI View를 UIKit에서 사용할 수 있게 해줍니다._

---

## 2. UIPageViewController의 DataSource 만들기
![section2-1](/assets/images/swift-tutorials/interfacing-with-uikit/section2-1.png){: .center-image}{: width="300"}

지금까지 UIPageViewController를 View에서 사용할 수 있도록 만들어보았습니다. 이제 

In a few short steps, you’ve done a lot — the PageViewController uses a UIPageViewController to show content from a SwiftUI view. Now it’s time to enable swiping interactions to move from page to page.

A SwiftUI view that represents a UIKit view controller can define a Coordinator type that SwiftUI manages and provides as part of the representable view’s context.

### Step 1
Declare a nested Coordinator class inside PageViewController.

SwiftUI manages your UIViewControllerRepresentable type’s coordinator, and provides it as part of the context when calling the methods you created above.

```swift
class Coordinator: NSObject {
    var parent: PageViewController

    init(_ pageViewController: PageViewController) {
        self.parent = pageViewController
    }
}
```

### Step 2
Add another method to PageViewController to make the coordinator.
SwiftUI calls this makeCoordinator() method before makeUIViewController(context:), so that you have access to the coordinator object when configuring your view controller.

**Tip**  
You can use this coordinator to implement common Cocoa patterns, such as delegates, data sources, and responding to user events via target-action.
```swift
func makeCoordinator() -> Coordinator {
    Coordinator(self)
}
```

### Step 3
Add UIPageViewControllerDataSource conformance to the Coordinator type, and implement the two required methods.
These two methods establish the relationships between view controllers, so that you can swipe back and forth between them.
```swift
class Coordinator: NSObject, UIPageViewControllerDataSource {
    var parent: PageViewController

    init(_ pageViewController: PageViewController) {
        self.parent = pageViewController
    }

    func pageViewController(
        _ pageViewController: UIPageViewController,
        viewControllerBefore viewController: UIViewController) -> UIViewController?
    {
        guard let index = parent.controllers.firstIndex(of: viewController) else {
            return nil
        }
        if index == 0 {
            return parent.controllers.last
        }
        return parent.controllers[index - 1]
    }

    func pageViewController(
        _ pageViewController: UIPageViewController,
        viewControllerAfter viewController: UIViewController) -> UIViewController?
    {
        guard let index = parent.controllers.firstIndex(of: viewController) else {
            return nil
        }
        if index + 1 == parent.controllers.count {
            return parent.controllers.first
        }
        return parent.controllers[index + 1]
    }
}
```

### Step 4
Add the coordinator as the data source of the UIPageViewController.

```swift
pageViewController.dataSource = context.coordinator
```