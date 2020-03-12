---
layout: post
title: "[SwiftUI] UIKit과 인터페이스 연결하기"
description: "SwiftUI에서 UIPageViewController 사용하기"
author: "kyujin.kim"
date: 2020-03-12
categories: [iOS, SwiftUI]
tags: [iOS, SwiftUI]
---

> [본문 링크](https://developer.apple.com/tutorials/swiftui/interfacing-with-uikit#track-the-page-in-a-swiftui-views-state)  
> SwiftUI는 모든 Apple 플랫폼의 기존 UI 프레임워크들과 함께 완벽하게 동작합니다. 예를 들어, UIKit의 뷰와 뷰 컨트롤러들을 SwiftUI의 뷰 안에 배치할 수 있고, 그 반대도 가능합니다.

이번 글은 **SwiftUI**에서 **UIPageViewController**와 **UIPageControl**을 사용하는 방법에 대해 알아 볼 것입니다.

UIPageViewController를 사용하여 페이징이 되는 **View**(SwiftUI)를 만들어보고, **@State**와 **@Binding** 변수들을 사용하여 데이터를 업데이트하고 UI에 표시 해볼 것 입니다.

샘플 프로젝트는 [이곳](https://docs-assets.developer.apple.com/published/0a3f2fc604/InterfacingWithUIKit.zip)에서 다운받으실 수 있습니다.  
_참고로, 이 글은 **Swift Tutorials**의 [Interfacing with UIKit](https://developer.apple.com/tutorials/swiftui/interfacing-with-uikit#track-the-page-in-a-swiftui-views-state)을 기반으로 작성되었습니다._

## UIPageViewController를 대신하는 View 만들기

![section1-1](/assets/images/swift-tutorials/interfacing-with-uikit/section1-1.png){: .center-image}{: width="300"}

SwiftUI에서 UIKit의 뷰와 뷰 컨트롤러를 사용하기 위해서는, **UIViewRepresentable**과 **UIViewControllerRepresentable** 프로토콜을 따르는 타입을 만들어야 합니다. 여기서 만들어진 타입은 UIKit을 생성하고 설정할 수 있으며, SwiftUI는 이 타입의 생명주기를 관리하고 필요 시 업데이트를 수행합니다.

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

두 번째로 [updateUIViewController(_:context:)](https://developer.apple.com/documentation/swiftui/uiviewcontrollerrepresentable/3278035-updateuiviewcontroller) 메서드를 추가합니다. 이 메서드에서는 배열의 첫 번째 뷰 컨트롤러를 보여주기 위해 **setViewControllers(_:direction:animated:)**를 호출하도록 되어 있습니다.

이 메서드는 뷰 컨트롤러에 영향을 미치는 변화가 발생했을 때 호출되며, 이곳에서 Context에 주어지는 정보로 뷰 컨트롤러를 업데이트 해야 합니다.

```swift
func updateUIViewController(_ pageViewController: UIPageViewController, context: Context) {
    pageViewController.setViewControllers(
        [controllers[0]], direction: .forward, animated: true)
}
```

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

### Step 4 
Create another SwiftUI view to present your UIViewControllerRepresentable view.

Create a new SwiftUI view file, named PageView.swift, and update the PageView type to declare PageViewController as a child view.

Notice that the generic initializer takes an array of views, and nests each one in a UIHostingController. A UIHostingController is a UIViewController subclass that represents a SwiftUI view within UIKit contexts.

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
        PageView()
    }
}
```

### Step 5
Update the preview provider to pass the required array of views, and the preview starts working.

```swift
struct PageView_Previews: PreviewProvider {
    static var previews: some View {
        PageView(features.map { FeatureCard(landmark: $0) })
            .aspectRatio(3/2, contentMode: .fit)
    }
}
```

### Step 6
Pin the PageView preview to the canvas before you continue — this view is where all the action is.

![section1-2](/assets/images/swift-tutorials/interfacing-with-uikit/section1-2.png){: .center-image}{: width="300"}

## Section 2 - Create the View Controller's Data Source

![section2-1](/assets/images/swift-tutorials/interfacing-with-uikit/section2-1.png){: .center-image}{: width="300"}

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