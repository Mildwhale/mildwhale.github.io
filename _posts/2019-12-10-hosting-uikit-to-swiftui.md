---
layout: post
title: "[iOS] SwiftUI와 UIKit 함께 사용하기"
description: "SwiftUI와 UIKit을 함께 사용하는 방법에 대해 소개합니다."
author: "kyujin.kim"
date: 2019-12-10
categories: [iOS, SwiftUI]
tags: [iOS, SwiftUI, UIHostingController, UIViewRepresentable, UIViewControllerRepresentable]
---

SwiftUI가 등장하면서 기존에 사용하던 UIKit은 어떻게 해야 하는지 고민했던 적이 있습니다. UIKit으로 만들어진 앱에 SwiftUI를 사용하거나, SwiftUI로 만들어진 앱에 UIKit 프레임워크를 사용하려면 어떻게 해야 할까요?

다행히도 Apple은 두 프레임워크의 상호 호환을 지원합니다. 이번 글에서는 두 프레임워크를 함께 사용하는 방법에 대해 알아보겠습니다.

## SwiftUI to UIkit
iOS 13에서 추가된 [UIHostingController](https://developer.apple.com/documentation/swiftui/uihostingcontroller)를 사용하면, SwiftUI [View](https://developer.apple.com/documentation/swiftui/view)를 UIKit에서 사용할 수 있습니다.

UIHostingController는 [Interfacing with UIKit](https://developer.apple.com/tutorials/swiftui/interfacing-with-uikit)문서에 다음과 같이 정의되어 있습니다.
> UIHostingController is a UIViewController subclass that represents a SwiftUI view within UIKit contexts.

설명 그대로 UIViewController의 서브 클래스이며, [init:rootView:](https://developer.apple.com/documentation/swiftui/uihostingcontroller/3278015-init) 생성자에 SwiftUI의 View를 파라미터로 넘겨주면 UIViewController로 사용할 수 있습니다.

특별히 더 설명할 것이 없기 때문에, 간단한 예제 코드를 소개하면서 다음 주제로 넘어가겠습니다.

```swift
// ContentView.swift
struct ContentView: View {
    var body: some View {
        Text("Wayne's World!!!")
    }
}

// AppDelegate.swift
let window = UIWindow(windowScene: windowScene)
let hostingController = UIHostingController(rootView: ContentView())
window.rootViewController = hostingController
window.makeKeyAndVisible()
self.window = window
```

## UIKit to SwiftUI
이번에는 UIKit으로 만들어진 UIView를 SwiftUI에서 사용하는 방법을 소개합니다. 이 방법을 사용하면 기존에 만들어진 멋진 UIKit Framework들을 재사용할 수 있습니다. 😆

### UIViewControllerRepresentable
> A view that represents a UIKit view controller.

[UIViewControllerRepresentable](https://developer.apple.com/documentation/swiftui/uiviewcontrollerrepresentable) Protocol을 구현하면 UIViewController를 SwiftUI에서 사용할 수 있습니다.

```swift
struct ViewControllerRepresentation: UIViewControllerRepresentable {
    
}
```

먼저 UIViewControllerRepresentable을 구현하는 struct을 생성합니다. 그럼 Xcode에서 Required 함수를 구현하지 않았다는 에러가 발생합니다.

```swift
struct ViewControllerRepresentation: UIViewControllerRepresentable {
    func makeUIViewController(context: Context) -> ViewController {
        UIStoryboard(name: "Main", bundle: nil)
      .instantiateViewController(withIdentifier: "ViewController")
      as! ViewController
    }
    
    func updateUIViewController(_ uiViewController: ViewController, context: Context) {
        
    }
}
```

위 예시와 같이 [makeUIViewController:context:](https://developer.apple.com/documentation/swiftui/uiviewcontrollerrepresentable/3278034-makeuiviewcontroller) 함수와 [updateUIViewController:_:context:](https://developer.apple.com/documentation/swiftui/uiviewcontrollerrepresentable/3278035-updateuiviewcontroller) 함수를 구현해주면 에러가 사라집니다.

`makeUIViewController:context:` 함수는 UIViewController를 생성하고 초기화를 수행하는 함수이며, `updateUIViewController` 함수는 ViewController의 업데이트가 필요할 때 호출됩니다. 이 위치에서 ViewController에 필요한 데이터 또는 정보를 갱신해주어야 합니다.

```swift
var body: some View {
    NavigationLink(destination: ViewControllerRepresentation()) {
          Text("Move to UIViewController")
    }
}
```

ViewControllerRepresentation 구조체를 생성하여 위 예시와 같이 호출해주면, UIViewController를 SwiftUI에서 사용할 수 있습니다!

### UIViewRepresentable
UIView도 UIViewRepresentable을 구현하면 SwiftUI에서 손쉽게 사용할 수 있습니다. 

UIViewRepresentable Protocol이 제공하는 함수는 이전 항목의 UIViewControllerRepresentable과 동일하기 때문에 자세한 설명은 생략하고, 예제 코드를 소개하고 마무리 하도록 하겠습니다.

```swift
struct ColorUISlider: UIViewRepresentable {
    var color: UIColor
    @Binding var value: Double

    func makeUIView(context: Context) -> UISlider {
        let slider = UISlider(frame: .zero)
        slider.thumbTintColor = color
        slider.value = Float(value)
        return slider
    }

    func updateUIView(_ uiView: UISlider, context: Context) {
        uiView.value = Float(self.value)
    }
}
```

## 마치며
위에서 설명한 내용을 실습할 수 있는 튜토리얼은 참고자료의 `Interfacing with UIKit`에 준비되어 있습니다. 이 주제와 이어지는 Data Dependency를 처리할 수 있는 방법과, 다양한 SwiftUI 튜토리얼을 제공하고 있으니 꼭 한번 실습해보시는 것을 추천합니다.

## 참고자료
[Interfacing with UIKit](https://developer.apple.com/tutorials/swiftui/interfacing-with-uikit)

[SwiftUI by Tutorials](https://store.raywenderlich.com/products/swiftui-by-tutorials)