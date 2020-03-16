---
layout: post
title: "[SwiftUI] UIKit과 인터페이스 연결하기 - 2/2"
description: "SwiftUI에서 @State와 @Binding을 사용하여 UIPageControl 적용하기"
author: "kyujin.kim"
date: 2020-03-16
categories: [iOS, SwiftUI]
tags: [iOS, SwiftUI]
---

이번 글에서는 SwiftUI에서 **UIPageControl**(UIView)을 사용하는 방법에 대해 알아볼 것입니다. [이전 글](/2020-03-12-Interfacing-with-UIKit-1/)과 이어지는 내용이기 때문에, 아직 못보셨다면 이전 글을 먼저 보고 오셔야 합니다.

이 글은 **Swift Tutorials**의 **Interfacing with UIKit**을 기반으로 작성되었습니다.

---

## State와 Binding으로 Page 연결하기
![section3-1](/assets/images/swift-tutorials/interfacing-with-uikit/section3-1.png){: .center-image}{: width="300"}

UIPageControl을 추가하기 전에, 우리는 지금 몇 번째 페이지가 보여지고 있는지 알 필요가 있습니다. 이 기능을 구현하기 위해서는 **PageView**에 `@State` 변수를 정의하고, **PageViewController**의 `@Binding` 변수에 연결 해줘야 합니다. 왜냐하면, UIPageViewController의 Delegate는 PageViewController의 Coordinator에 구현 할 것이기 때문이죠.

### PageViewController에 currentPage 추가하기
**PageViewController**에 `currentPage`라는 변수를 추가하는 것으로 시작하겠습니다. 이때 이 변수는 **@Binding** 속성으로 지정되어야 합니다. 그리고 **setViewControllers(_:direction:animated:)** 메서드를 호출하는 곳에 currentPage를 사용하도록 수정합니다.

```swift
struct PageViewController: UIViewControllerRepresentable {
    @Binding var currentPage: Int

    .
    .
    .

    func updateUIViewController(_ pageViewController: UIPageViewController, context: Context) {
        pageViewController.setViewControllers(
            [controllers[currentPage]], direction: .forward, animated: true)
    }
}
```

### UIPageViewControllerDelegate 구현하기
PageViewController의 Coordinator는 UIPageViewDataSource만 구현되어있고, Delegate는 구현되어있지 않습니다. 현재 보여지고있는 페이지 정보를 가져오고 싶다면, **UIPageViewControllerDelegate**를 구현해야 합니다.

아래 코드를 참고하여 Coordinator 선언부에 `UIPageViewControllerDelegate`를 추가하고, **(_:didFinishAnimating:previousViewControllers:transitionCompleted completed: Bool)** 메서드를 구현합니다.

```swift
class Coordinator: NSObject, UIPageViewControllerDataSource, UIPageViewControllerDelegate {
    .
    .
    .

    func pageViewController(_ pageViewController: UIPageViewController, didFinishAnimating finished: Bool, previousViewControllers: [UIViewController], transitionCompleted completed: Bool) {
        if completed,
            let visibleViewController = pageViewController.viewControllers?.first,
            let index = parent.controllers.firstIndex(of: visibleViewController)
        {
            parent.currentPage = index
        }
    }
}
```

그리고, **makeUIViewController(context: Context)** 함수가 있는 곳으로 이동해서 pageViewController의 delegate를 설정해줍니다.

```swift
pageViewController.delegate = context.coordinator
```

### PageView에 currentPage 추가하기
다음으로 **PageView**에도 currentPage 변수를 추가하는데, 이때 이 변수는 **@State** 속성으로 지정되어야 합니다. 왜냐하면 이 변수가 변경될 때 마다, UIPageControl의 Page 위치를 갱신해야 하기 때문입니다.

PageViewController를 초기화하는 부분에서 `$` 연산자를 사용하는 것에 주의하면서, 아래 코드를 추가합니다.

```swift
struct PageView<Page: View>: View {
    @State var currentPage = 0

    .
    .
    .

    var body: some View {
        PageViewController(controllers: viewControllers, currentPage: $currentPage)
    }
}
```

`$currentPage`를 PageViewController로 넘겨주게 되면, @Binding으로 선언 된 PageViewController의 currentPage와 서로 연결됩니다. 그럼 UIPageViewController를 스와이프하여 Page가 변경될 때 마다, **PageView**의 currentPage가 업데이트 될 것입니다.

### UIPageControl을 대신하는 PageControl 만들기
![section4-1](/assets/images/swift-tutorials/interfacing-with-uikit/section4-1.png){: .center-image}{: width="300"}

UIKit으로 만들어진 UIPageControl을 사용하기 위해, 프로젝트에 `PageControl.swift` 라는 이름의 SwiftUI 파일을 추가합니다. 그리고 아래 코드를 참고하여 **UIViewRepresentable** 프로토콜을 따르는 **PageControl**을 추가합니다.

여기서 구현된 make, update 함수는 UIViewControllerRepresentable의 것과 동일한 생명주기를 갖습니다.

```swift
import UIKit

struct PageControl: UIViewRepresentable {
    var numberOfPages: Int
    @Binding var currentPage: Int

    func makeUIView(context: Context) -> UIPageControl {
        let control = UIPageControl()
        control.numberOfPages = numberOfPages

        return control
    }

    func updateUIView(_ uiView: UIPageControl, context: Context) {
        uiView.currentPage = currentPage
    }
}
```

다음으로 **PageView**의 **body**를 아래 코드와 같이 변경합니다. body를 아래와 같이 변경하면, PageViewController 우측 하단에 UIPageControl이 추가됩니다.

```swift
var body: some View {
    ZStack(alignment: .bottomTrailing) {
        PageViewController(controllers: viewControllers, currentPage: $currentPage)
        PageControl(numberOfPages: viewControllers.count, currentPage: $currentPage)
            .padding(.trailing)
    }
}
```

이제, 페이지 뷰 컨트롤러를 좌/우로 스와이프 하면 페이지가 이동하면서 PageControl의 위치도 같이 움직이는 것을 볼 수 있습니다.

### Coodinator로 Target-Action 구현하기
UIPageControl은 Delegate 패턴이 아닌 target-action 패턴을 사용하여 사용자의 터치 이벤트를 처리합니다. 이 패턴을 구현하는 방법은 DataSource와 Delegate를 Coordinator에 구현했던 방법과 매우 비슷합니다.

먼저, PageControl의 내부에 Coordinator와 makeCoordinator()를 추가해줍니다. PageViewController에서 했던 것과 같은 방법이죠.

여기서 추가된 Coordinator는 **@objc**로 선언된 메서드가 있습니다. 이 메서드를 UIPageControl의 target-action 동작에 사용 할 것입니다.

```swift
func makeCoordinator() -> Coordinator {
    Coordinator(self)
}

class Coordinator: NSObject {
    var control: PageControl

    init(_ control: PageControl) {
        self.control = control
    }

    @objc func updateCurrentPage(sender: UIPageControl) {
        control.currentPage = sender.currentPage
    }
}
```

다음으로 **valueChanged** 이벤트가 발생했을 때, UIPageControl의 타겟으로 coordinator를 지정하고, 실행 함수로 **updateCurrentPage(sender:)** 메서드를 지정해줍니다.

```swift
func makeUIView(context: Context) -> UIPageControl {
    let control = UIPageControl()
    control.numberOfPages = numberOfPages
    control.addTarget(
        context.coordinator,
        action: #selector(Coordinator.updateCurrentPage(sender:)),
        for: .valueChanged)

    return control
}
```

페이지뷰를 좌/우로 스와이프 해보고, PageControl의 좌/우를 터치해보세요. 이미지가 바뀌고, PageControl의 위치가 업데이트 된다면 모든 작업이 완료 된 것입니다.

![result](/assets/images/swift-tutorials/interfacing-with-uikit/result.gif){: .center-image}{: width="320"}

## 결론