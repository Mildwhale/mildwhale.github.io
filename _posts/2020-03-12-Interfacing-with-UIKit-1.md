---
layout: post
title: "[SwiftUI] UIKit과 인터페이스 연결하기 - 1/2"
description: "SwiftUI에서 UIPageViewController로 페이징 구현하기"
author: "kyujin.kim"
date: 2020-03-12
categories: [iOS, SwiftUI]
tags: [iOS, SwiftUI]
---

> SwiftUI는 모든 Apple 플랫폼의 기존 UI 프레임워크들과 함께 완벽하게 동작합니다. 예를 들어, UIKit의 뷰와 뷰 컨트롤러들을 SwiftUI의 뷰 안에 배치할 수 있고, 그 반대도 가능합니다.  
> -- **from** [**Interfacing with UIKit**](https://developer.apple.com/tutorials/swiftui/interfacing-with-uikit#track-the-page-in-a-swiftui-views-state)

이번 시리즈에서는 페이징을 지원하는 **SwiftUI View**를 만들어 볼 것입니다. **UIPageViewController**와 **UIPageControl**을 사용할 예정이고, **@State**와 **@Binding**으로 데이터를 업데이트하는 방법에 대해서도 알아볼 것입니다. 내용이 길기 때문에 2개의 글로 나누어 진행할 예정입니다.

샘플 프로젝트는 [이곳](https://docs-assets.developer.apple.com/published/0a3f2fc604/InterfacingWithUIKit.zip)에서 다운받으실 수 있습니다.  
그리고, 이 글은 **Swift Tutorials**의 **Interfacing with UIKit**을 기반으로 작성되었습니다.

---

## UIPageViewController를 대신하는 View 만들기
![section1-1](/assets/images/swift-tutorials/interfacing-with-uikit/section1-1.png){: .center-image}{: width="300"}

SwiftUI에서 UIKit의 뷰와 뷰 컨트롤러를 사용하기 위해서는, **UIViewRepresentable**과 **UIViewControllerRepresentable** 프로토콜을 따르는 View를 만들어야 합니다. 이 프로토콜들을 따르는 View는 UIKit을 생성하고 설정할 수 있으며, SwiftUI가 이 View의 생명주기를 관리하고 필요할 때 업데이트를 수행합니다.

그럼 페이징을 담당할 View를 먼저 만들어 보겠습니다.

### PageViewController 만들기
프로젝트를 열고, `PageViewController.swift`라는 이름의 SwiftUI 파일을 생성합니다. 그리고 아래 코드를 참고하여 UIViewControllerRepresentable을 따르는 PageViewController를 정의합니다.

```swift
import UIKit
import SwiftUI

struct PageViewController: UIViewControllerRepresentable {
    var controllers: [UIViewController]
}
``` 

이 PageViewController는 UIViewController 인스턴스의 배열을 가지고 있습니다. 이 배열은 나중에 UIPageViewController의 DataSource로 사용 할 예정입니다.

### UIViewControllerRepresentable 프로토콜 구현하기
UIViewControllerRepresentable 프로토콜은 2개의 메서드를 필수적으로 구현해야 합니다. 

첫 번째로, [makeUIViewController(context:)](https://developer.apple.com/documentation/swiftui/uiviewcontrollerrepresentable/3278034-makeuiviewcontroller) 메서드를 PageViewController의 추가합니다. 이 메서드에서는 UIPageViewController를 생성하고 설정 할 수 있습니다.

```swift
func makeUIViewController(context: Context) -> UIPageViewController {
    let pageViewController = UIPageViewController(
        transitionStyle: .scroll,
        navigationOrientation: .horizontal)

    return pageViewController
}
```

SwiftUI는 View가 화면에 보여질 준비가 되었을 때 이 메서드를 단 한번만 호출하여, 뷰 컨트롤러의 생명주기를 관리합니다.

두 번째로, [updateUIViewController(_:context:)](https://developer.apple.com/documentation/swiftui/uiviewcontrollerrepresentable/3278035-updateuiviewcontroller) 메서드를 추가합니다. 이 메서드는 뷰 컨트롤러에 영향을 미치는 변화가 발생했을 때 호출되며, 이곳에서 뷰 컨트롤러의 속성을 업데이트 할 수 있습니다.

```swift
func updateUIViewController(_ pageViewController: UIPageViewController, context: Context) {
    pageViewController.setViewControllers(
        [controllers[0]], direction: .forward, animated: true)
}
```

이 메서드에서는 UIPageViewController에서 보여지는 뷰 컨트롤러를 설정하기 위해 **setViewControllers(_:direction:animated:)**를 호출하도록 되어 있습니다.

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

프로젝트에 `PageView.swift`라는 이름의 새로운 SwiftUI 파일을 생성하고, 아래 코드를 추가합니다.

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

Generic으로 정의된 **Page** 타입을 [UIHostingController](https://developer.apple.com/documentation/swiftui/uihostingcontroller)에 설정해주고, 이니셜라이저의 파라미터로 사용하는 것을 볼 수 있습니다. 그리고 **body**에서는 init(_:)에서 설정한 **viewControllers**로 PageViewController를 생성하고, Child View로 사용하는 것을 볼 수 있습니다.

여기서 사용한 UIHostingController는 UIViewController의 서브 클래스이며, SwiftUI View를 UIKit에서 사용할 수 있게 해줍니다.

---

## UIPageViewController의 DataSource 구현하기
![section2-1](/assets/images/swift-tutorials/interfacing-with-uikit/section2-1.png){: .center-image}{: width="300"}

지금까지 UIPageViewController를 View에서 사용할 수 있도록 만들었습니다. 이제 조금만 더 하면 좌,우 스와이프를 통해 페이지 이동을 구현할 수 있습니다. 

이 기능을 구현하기 위해서는 UIPageViewController의 Delegate를 사용해야 하는데, SwiftUI에서는 이것을 **Coordinator**라는 새로운 패턴을 사용하여 지원합니다.

_우리가 UIKit에서 사용하던 [Coordinator 디자인 패턴](https://www.raywenderlich.com/158-coordinator-tutorial-for-ios-getting-started)과는 전혀 다르기 때문에, 햇갈리지 않도록 주의해야 합니다._

### Coordinator 정의하기
> SwiftUI는 UIViewControllerRepresentable 타입의 Coordinator를 관리하고, 위에서 만들었던 메소드(make, update)를 호출 할 때 컨텍스트의 일부로 제공합니다.

먼저, PageViewController의 내부에 **Coordinator**라는 이름의 클래스를 추가합니다. UIPageViewController의 Delegate에서 뷰 컨트롤러의 정보에 접근해야 하기 때문에, parent를 추가했습니다. 

```swift
class Coordinator: NSObject {
    var parent: PageViewController

    init(_ pageViewController: PageViewController) {
        self.parent = pageViewController
    }
}
```

그리고 바로 아래에 **makeCoordinator()** 함수를 추가합니다. 이 함수는 **makeUIViewController(context:)**함수가 호출되기 전에 자동으로 호출됩니다.

뷰 컨트롤러를 설정할 때 주어지는 Context를 통해 Coordinator에 접근할 수 있습니다.

```swift
func makeCoordinator() -> Coordinator {
    Coordinator(self)
}
```

그리고 Coordinator는 UIKit의 일반적인 패턴인 Delegate, DataSource, Target-Action 등의 이벤트를 구현하고 사용할 수도 있습니다.

### UIPageViewControllerDataSource 추가하기
Coordinator에 UIPageViewControllerDataSource 프로토콜을 추가하고, 2개의 필수 메서드를 구현합니다. 이 메서드들은 뷰 컨트롤러간의 관계를 설정하여, 서로간에 스와이프가 가능하도록 도와줍니다.

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

마지막으로 UIPageViewController의 dataSource를 설정해주면 모든 작업이 완료됩니다. 위에서 만들었던 makeUIViewController(context:) 함수에서 dataSource로 coordinator를 설정해줍니다.

```swift
pageViewController.dataSource = context.coordinator
```

완성된 PageViewController.swift는 아래와 같습니다.
<script src="https://gist.github.com/Mildwhale/5913a37a85cdef1f4c524fa91502daaa.js"></script>

### Preview로 결과 확인하기
PageView.swift 파일로 이동하여 SwiftUI 프리뷰를 실행(Resume)하고, 화면 중앙의 이미지를 좌,우로 스와이프 해보면 페이징이 구현 된 것을 볼 수 있습니다.

![section2-2](/assets/images/swift-tutorials/interfacing-with-uikit/section2-2.gif){: .center-image}

---

## 마치며
지금까지 UIPageViewController를 사용하여 SwiftUI의 View의 페이징을 구현하고, Coordinator로 dataSource를 설정하는 방법을 알아보았습니다.

개인적으로 Coordinator가 어떤 역할을 하는지에 대한 궁금증이 해결된 것 같아 기분이 좋습니다.

다음 글에서는 UIPageControl을 사용하여 현재 페이지의 위치를 표시해볼 겁니다. 이 기능을 구현하기 위해서는 어떻게 해야 할까요? 

그럼 [다음 글](/2020-03-16-Interfacing-with-UIKit-2/) 에서 봽겠습니다.
읽어주셔서 감사합니다! 😉