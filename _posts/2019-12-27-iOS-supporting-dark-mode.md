---
layout: post
title: "iOS 13에서 다크 모드 지원하기 (Supporting Dark Mode)"
description: "아직도 다크 모드 지원을 안 한 앱이 있다고요?"
author: "kyujin.kim"
date: 2019-12-27
tags: [iOS, Design]
---

~~네 다크 모드 지원을 아직 안 한 앱이 여기 있었습니다. 불과 3주 전까지만 해도 말이죠.~~ 🤭

[Implementing Dark Mode on iOS](https://developer.apple.com/videos/play/wwdc2019/214/) 세션을 본 iOS 엔지니어라면 한번쯤은 내 앱에도 적용해보고 싶다는 생각을 했을 것 입니다. 하지만 곧 거대한 레거시 프로젝트의 장벽과 밀린 업무에 가로막히게 되고, 길을 잃은 다크 모드는 머릿속 어딘가에서 끊임없이 우리를 유혹할 것 입니다. 😘

![brainImage](/assets/images/ios-dark-mode/brain.png){: .center-image}{: width="400" height="400"}

결국 다크 모드의 유혹을 이기지 못한 뤼이드의 iOS 챕터는 신규 프로젝트에 다크 모드를 적용해보기로 했습니다. 그리고 다크 모드를 적용하는 과정에서 알게 된 내용과 경험을 소개하고 공유하기 위해 이 글을 작성했습니다.

그럼 다크 모드를 지원하기 위해 UIColor를 어떻게 사용해야 하는지부터 알아보시죠. 😎

<br/>

## UIColor in iOS 13
Apple은 기본적으로 [SystemColor](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/color/) 사용을 권장합니다. SystemColor는 `UIColor.system...`과 같이 `system`으로 시작하거나 Color 이름에 `system`이 포함되어 있으며, 라이트 또는 다크 모드의 전환 시 별도의 설정이 필요하지 않습니다.

그렇다면 우리 앱 만의 특별한 Color를 사용해야 하는 서드파티 앱들은 어떻게 해야 할까요?  
다행히 iOS 13부터 [UIColor](https://developer.apple.com/documentation/uikit/uicolor)를 동적으로 변경할 수 있는 API가 추가되었고, 이 API를 사용하면 우리만의 Color를 만들어 사용할 수 있습니다. 🎉

![uicolorImage](/assets/images/ios-dark-mode/uicolor-before-after.png){: .center-image}{: width="600"}

그럼 UIColor를 동적으로 만들려면 어떻게 해야 할까요?

### Color Set - Assets Catalog
Assets Catalog의 Color Set을 사용한다면 Appearences 속성을 `Any, Light, Dark`로 변경하고, 각 Mode에 맞는 Color를 설정해주기만 하면 됩니다. 매우 간단하죠?  
*(Assets에 등록된 Image도 같은 방법으로 다크 모드 설정을 지원합니다)*

![assetCatalogImage](/assets/images/ios-dark-mode/assetcatalog1.png){: .center-image}{: width="600"}

하지만 Assets Catalog의 Color Set은 iOS 11부터 지원하는 기능입니다. 따라서 코드로 Color를 사용하는 대부분의 레거시 프로젝트는 다음에 소개하는 방법이 유효할겁니다.

### DynamicProvider - UIColor
iOS 13에서 추가된 [init(dynamicProvider:)](https://developer.apple.com/documentation/uikit/uicolor/3238041-init)과 [resolvedColor(with:)](https://developer.apple.com/documentation/uikit/uicolor/3238042-resolvedcolor) API를 사용하면 코드로 동적인 UIColor를 만들 수 있습니다. 특이하게도 새로 추가된 API 모두 [UITraitCollection](https://developer.apple.com/documentation/uikit/uitraitcollection)을 파라미터로 받고 있습니다. 그럼 UIColor를 만들어보기 전에 UITraitCollection에 대해 간단히 알아보고 넘어가는게 좋겠죠?

#### UITraitCollection
> The iOS interface environment for your app, defined by traits such as horizontal and vertical size class, display scale, and user interface idiom. - *Apple Documentation*

UITraitCollection은 앱 Interface의 정보*(Trait)*를 가지고 있습니다. 여기서 말하는 trait의 종류는 단말의 크기와 방향, Scale 등이 있으며 라이트, 다크 모드의 설정값인 [UserInterfaceStyle](https://developer.apple.com/documentation/uikit/uitraitcollection/1651063-userinterfacestyle)도 Trait의 일종입니다.

이 Trait이 변경되면 개발자는[traitCollectionDidChange(_:)](https://developer.apple.com/documentation/uikit/uitraitenvironment/1623516-traitcollectiondidchange)델리게이트 메서드를 통해 Trait의 변경을 알 수 있고, UI를 적절히 업데이트 할 수 있습니다.  

보다 자세한 내용은 [Making Apps Adaptive, Part 1](https://developer.apple.com/videos/play/wwdc2016/222/)을 참고해주세요.

![traitCollectionImage](/assets/images/ios-dark-mode/traitcollection.png){: .center-image}{: width="600"}

어떤 API를 사용해야 하는지 알아봤으니, 이제 나만의 Color를 만드는 방법을 알아보겠습니다.

<br/>

## DynamicProvider 사용하기
뤼이드의 iOS 챕터는 산타클로젯[^1] 이라는 디자인 시스템을 구성해서 ['AI 토익 튜터, 산타'](https://apps.apple.com/kr/app/ai-토익-튜터-산타/id1148006701)와 신규 프로젝트에서 사용 중입니다. 산타클로젯이 제공하는 Color에 DynamicProvider를 적용하면 다크 모드를 지원할 수 있지 않을까요?

그럼 산타클로젯에 구현된 예제 코드를 통해 DynamicProvider를 어떤 식으로 사용하면 되는지 알아보겠습니다.

### 예제 코드
```swift
import UIKit

typealias SCColor = UIColor

public extension SCColor {
    static var black1: SCColor { 
        return SCColor(hexString: "040404") 
    }

    static var white1: SCColor { 
        return SCColor(hexString: "FFFFFF") 
    }

    static var colorA: SCColor {
        return color(light: .white1, dark: .black1)
    }

    private static func color(light: SCColor, dark: SCColor) -> SCColor {
        if #available(iOS 13, *) {
            return UIColor { (traitCollection: UITraitCollection) -> UIColor in
                return traitCollection.userInterfaceStyle == .dark ? dark : light
            }
        } else {
            return light
        }
    }    
}
```

다크 모드를 지원하기 전에는 `SCColor.black1`, `SCColor.white1`와 같이 단일 Color를 사용하여 앱을 개발했습니다. 지금은 디자이너와의 협의를 통해 라이트 모드에서는 white1, 다크 모드에서는 black1으로 보여지는 `colorA`와 같은 추상적인 이름을 사용하기로 했습니다.

위 코드에서 눈여겨봐야 할 부분은 `private static func color(light:dark:)` 메서드인데요, iOS 13부터 [init(dynamicProvider:)](https://developer.apple.com/documentation/uikit/uicolor/3238041-init)을 사용하여 UIColor를 생성하는 것을 볼 수 있습니다.

이 생성자의 파라미터로 입력되는 traitCollection은 기본적으로 iOS 13에서 추가된 [UITraitCollection.current](https://developer.apple.com/documentation/uikit/uitraitcollection/3238080-current)입니다.

![dynamicUIColorImage](/assets/images/ios-dark-mode/dynamicuicolor.png){: .center-image}{: width="600"}

이렇게 만들어진 Color를 어떻게 사용하면 되냐고요? 그냥 평소처럼 사용하면 됩니다. 😅

```swift
import UIKit
import SantaCloset

class MyViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()

        self.view.backgroundColor = SCColor.colorA
    }
}
```

iOS 13 이상의 단말에서 위의 코드를 실행하고 다크 모드로 변경하면 backgroundColor가 바뀌는 것을 볼 수 있습니다. 물론 iOS 13 미만의 단말에서는 light Color만 적용됩니다.

![modeChangeImage](/assets/images/ios-dark-mode/modechange.gif){: .center-image}{: width="600"}

<br/>

## UI 업데이트
UIView를 사용하여 UI를 구현했다면 이 정도의 작업만으로도 대부분의 View는 다크 모드를 지원하기에 충분합니다. 왜냐하면 UserInterfaceStyle이 변경되는 것을 OS가 감지하고 모든 Window와 View를 Redraw[^2] 하기 때문이죠.

이 과정에서 아래 나열된 메서드들이 호출되는데, 개발자들은 해당 메서드에서 추가적인 작업을 수행할 수 있습니다.

![updateMethodsImage](/assets/images/ios-dark-mode/updatemethods.png){: .center-image}{: width="600"}

위에 나열된 메서드를 지원하는 객체는 traitCollection의 변경을 감지하여 자동으로 Redraw를 하지만, CALayer는 traitCollection의 변경을 감지할 수 없습니다. 왜냐하면 UITraitCollection은 UIKit을 대상으로 만들어졌기 때문이죠.

```swift
let layer = CALayer()
layer.borderColor = UIColor.label.cgColor
```

따라서 예제 코드와 같이 borderColor를 설정해주는 코드가 있다면, 위에 나열된 메서드에서 적절히 업데이트를 해줘야 합니다. Apple에서는 CALayer를 업데이트 하기 위해 3가지 방법을 제시합니다.

### resolvedColor(with:)
첫 번째 방법은 UIColor의 [resolvedColor(with:)](https://developer.apple.com/documentation/uikit/uicolor/3238042-resolvedcolor) 메서드를 활용하는 방법입니다.

```swift
override func layoutSubviews() {
    super.layoutSubview()
    let traitCollection = view.traitCollection

    let resolvedColor = UIColor.label.resolvedColor(with: traitCollection)
    view.layer.borderColor = resolvedColor.cgColor
}
```

위 예제 코드와 같이 현재 view의 traitCollection을 이용해서 resolvedColor(with:)를 호출하면, traitCollection에 적합한 Color를 가져올 수 있습니다.

*실제로 저희 iOS 챕터에서 다크 모드 지원을 작업하던 중 NavigationBar의 backgroundColor가 제대로 변경되지 않는 문제가 있었고, 이 방법을 사용하여 문제를 해결할 수 있었습니다.*

### traitCollection.performAsCurrent(_:)
두 번째 방법은 traitCollection의 [performAsCurrent(_:)](https://developer.apple.com/documentation/uikit/uitraitcollection/3238082-performascurrent) 메서드를 활용하는 방법입니다.

```swift
override func layoutSubviews() {
    super.layoutSubview()
    let traitCollection = view.traitCollection

    traitCollection.performAsCurrent {
        self.view.layer.borderColor = UIColor.label.cgColor
    }
}
```

위 예제 코드와 같이 performAsCurrent 블럭에서 borderColor를 지정해주면, performAsCurrent를 호출하는 traitCollection을 사용하여 Color를 가져와서 설정할 수 있습니다.

### UITraitCollection.current
DynamicProvider 단락에서 UIColor는 기본적으로 UITraitCollection.current를 사용한다고 언급했었습니다.  
마지막 방법은 이 메커니즘을 활용합니다.

```swift
override func layoutSubviews() {
    super.layoutSubview()
    let layer = CALayer()
    let traitCollection = view.traitCollection

    let savedTraitCollection = UITraitCollection.current

    UITraitCollection.current = traitCollection
    layer.borderColor = UIColor.label.cgColor

    UITraitCollection.current = savedTraitCollection
}
```

현재 view의 traitCollection을 current에 설정해준 뒤 UIColor를 가져오면, 마지막으로 설정했던 Collection을 참조하여 Color를 가져오게 됩니다.  

current를 변경하는 것은 조금 위험해 보이지만, 실행 중인 스레드(메서드 내)에서만 영향을 미치고 메인 스레드에는 영향을 주지 않기 때문에 안전한 방법[^3]입니다.

물론 예제 코드의 마지막 줄처럼 모든 작업이 완료된 후 이전 상태의 traitCollection으로 되돌려 주는 것이 좋습니다.

<br/>

## 미립자 팁
앱에서 라이트, 다크 모드를 컨트롤하고 싶다면 어떻게 해야 할까요? 그리고 우리 앱은 아직 다크 모드를 지원하지 않으니, 라이트 모드로만 보이고 싶을 때에는 어떻게 해야 할까요?

마지막 주제는 다크 모드와 관련된 간단한 팁 입니다. 이 주제는 [이곳](https://developer.apple.com/documentation/xcode/supporting_dark_mode_in_your_interface/choosing_a_specific_interface_style_for_your_ios_app)과 [WWDC](https://developer.apple.com/videos/play/wwdc2019/214/)에 잘 정리되어 있으니, 직접 보시는 것도 좋습니다.

### 라이트 모드로 고정하기
iOS 13 이후 모든 앱은 다크 모드에서도 동작하게 됩니다. 하지만 아직 다크 모드를 지원하지 않는 앱이라면, 의도치 않은 Color로 보일 수 있습니다. 그래서 다크 모드를 지원하기 전까지 항상 라이트 모드로 앱이 보이도록 하기 위해 info.plist에 [UIUserInterfaceStyle](https://developer.apple.com/documentation/bundleresources/information_property_list/uiuserinterfacestyle) Key가 추가되었습니다.

info.plist에 `User Interface Style` Key를 추가하고 Value를 `light`로 설정하면 앱은 라이트 모드로만 보이게 됩니다.

<br/>

> **Important**  
> Supporting Dark Mode is strongly encouraged. Use the UIUserInterfaceStyle key to opt out only temporarily while you work on improvements to your app's Dark Mode support.

하지만 Apple은 다크 모드 지원을 **강력히 권장**하기 때문에, 다크 모드를 빨리 적용하고 이 옵션을 사용하지 않는 편이 좋을 것 같습니다.

### 특정 View의 스타일 변경
iOS 13부터 UIView에 추가된 [overrideUserInterfaceStyle](https://developer.apple.com/documentation/uikit/uiview/3238086-overrideuserinterfacestyle) Property를 변경하면 앱의 특정 View 또는 앱 전체의 스타일을 코드로 변경할 수 있습니다.

이 Property는 휴대폰의 설정을 오버라이드 하기 때문에, 오버라이드한 View와 하위 View의 traitCollection에만 영향을 미칩니다.

![overrideView](/assets/images/ios-dark-mode/overridestyle.png){: .center-image}{: width="600"}

앱 전체적으로 스타일을 지정하고 싶다면 앱의 RootViewController를 가지고 있는 window의 스타일을 설정해주면 됩니다.

```swift
if let window = UIApplication.shared.windows.first {
    window.overrideUserInterfaceStyle = .dark
}
```

<br/>

## 마치며
지금까지 다크 모드에 대응하는 방법을 알아보았습니다.  
이 글을 작성하면서 '생각보다 간단한데 왜 진작에 다크 모드를 지원하지 않았을까'하는 반성의 시간을 가질 수 있었습니다.

지금 내 머릿속에 다크 모드가 잠자고 있다면, 한번 도전해보는 것은 어떨까요? 😃  
읽어주셔서 감사합니다!

<br/>

## 참고자료
- [Implementing Dark Mode on iOS - WWDC 2019](https://developer.apple.com/videos/play/wwdc2019/214/)  
- [Supporting Dark Mode in Your Interface - Apple Documentation](https://developer.apple.com/documentation/xcode/supporting_dark_mode_in_your_interface)  
- [Choosing a Specific Interface Style for Your iOS App - Apple Documentation](https://developer.apple.com/documentation/xcode/supporting_dark_mode_in_your_interface/choosing_a_specific_interface_style_for_your_ios_app)  

[^1]: 산타클로젯은 Custom UIView와 Extensions 그리고 Color 세트로 구성되어있는 뤼이드 iOS 챕터의 디자인 시스템 입니다.  
[^2]: When the user changes the system appearance, the system automatically asks each window and view to redraw itself. - [Supporting Dark Mode in Your Interface](https://developer.apple.com/documentation/xcode/supporting_dark_mode_in_your_interface#topics)  
[^3]: This looks a little intimidating but it's absolutely safe. - [Implementing Dark Mode on iOS](https://developer.apple.com/videos/play/wwdc2019-214/?time=1487)