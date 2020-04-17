---
layout: post
title: "[iOS][RxSwift] iOS의 MVVM 아키텍처는 왜 제각각일까?"
description: "Input과 Output을 사용한 MVVM 아키텍처"
author: "kyujin.kim"
date: 2020-04-16
categories: [iOS, RxSwift]
tags: [iOS, RxSwift, MVVM]
---

신규 서비스의 iOS 앱 개발을 담당하게 되었습니다. 협업을 위해서는 어느 정도 대중적인 아키텍처가 필요했고, 팀원과의 논의를 통해 MVVM 아키텍처를 선택하게 되었습니다. 

![mvvm](/assets/images/mvvm/mvvm.jpeg){: .center-image}

익숙함이 가장 큰 무기인 **MVC**(~~Massive~~ View Controller)는 유지보수와 확장성을 생각했을 때, 가까운 미래에 고통을 받을 것이 분명하여 제외했습니다.

MVVM 아키텍처의 구현 패턴을 보다보니, `iOS MVVM은 표준이 없고 구현하는 사람마다 패턴이 조금씩 다르다`는 것을 알 수 있었습니다. 그중에, [Kickstarter](https://github.com/kickstarter/ios-oss)에서 사용하는 Input과 Output Protocol을 사용하는 방식이 눈에 띄어 프로젝트에 적용해보기로 했습니다.

그럼 간단한 예제를 통해 Input, Output Protocol을 사용한 MVVM 아키텍처의 구현을 알아보겠습니다.

## Example
### Protocol with Input&Output
```swift
protocol ViewModelType {
    associatedtype Dependency
    associatedtype Input
    associatedtype Output

    var dependency: Dependency { get }
    var disposeBag: DisposeBag { get set }
    
    var input: Input { get }
    var output: Output { get }
    
    init(dependency: Dependency)
}
```

ViewModel의 의존성인 **Dependency**, View에서 전달되는 이벤트인 **Input**과 Input의 결과를 출력하는 **Output**을 associatedtype으로 정의합니다.

그럼 이 Protocol을 사용하여 이름과 이메일 주소를 입력받고, 확인 버튼의 활성화 상태를 출력하는 ViewModel을 만들어보겠습니다.

### ViewModel
```swift
import Foundation
import RxSwift
import RxCocoa

final class MyViewModel: ViewModelType {
    struct Dependency {
        var name: String?
        var email: String?
    }

    struct Input {
        var nameText: AnyObserver<String?>
        var emailText: AnyObserver<String?>
    }

    struct Output {
        var isConfirmEnabled: Driver<Bool>
    }

    let dependency: Dependency
    var disposeBag: DisposeBag = DisposeBag()
    let input: Input
    let output: Output

    private let nameText$: BehaviorSubject<String?>
    private let emailText$: BehaviorSubject<String?>

    init(dependency: Dependency = Dependency(name: nil, email: nil)) {
        self.dependency = dependency

        // Streams
        let nameText$ = BehaviorSubject<String?>(value: nil)
        let emailText$ = BehaviorSubject<String?>(value: nil)
        let isConfirmEnabled$ = Observable.combineLatest(nameText$, emailText$)
            .map(validation)
            .asDriver(onErrorJustReturn: false)

        // Input & Output
        self.input = Input(nameText: nameText$.asObserver(),
                           emailText: emailText$.asObserver())
        self.output = Output(isConfirmEnabled: isConfirmEnabled$)

        // Binding
        self.nameText$ = nameText$
        self.emailText$ = emailText$
    }
}

private func validation(name: String?, email: String?) -> Bool {
    return name?.isEmpty == false && email?.isEmpty == false
}
```

이름과 이메일 입력이 Input에, 버튼 활성화 여부의 출력이 Output에 정의되어있는 것을 볼 수 있습니다. 스트림 생성과 관리는 init(dependency:)에서 담당하고 있습니다.

### View (Controller)
```swift
import UIKit
import RxSwift
import RxCocoa

final class View: UIViewController {
    private let nameTextField: UITextField = UITextField()
    private let emailTextField: UITextField = UITextField()
    private let confirmButton: UIButton = UIButton()
    
    let disposeBag = DisposeBag()
    var viewModel: MyViewModel = MyViewModel()

    override func viewDidLoad() {
        super.viewDidLoad()
        bind()
    }

    private func bind() {
        // Input
        nameTextField.rx.text
            .bind(to: viewModel.input.nameText)
            .disposed(by: disposeBag)

        emailTextField.rx.text
            .bind(to: viewModel.input.emailText)
            .disposed(by: disposeBag)

        // Output
        viewModel.output.isConfirmEnabled
            .drive(confirmButton.rx.isEnabled)
            .disposed(by: disposeBag)
    }
}
```

View는 textField의 text입력을 ViewModel의 input으로 전달하고, ViewModel의 output을 구독하여 화면에 반영합니다.

## 마무리
ViewModel의 Input과 Output을 통해 View와 ViewModel 간의 바인딩이 매우 간결한 것을 볼 수 있습니다. 이 구조에서는 기능의 수정 또는 추가 시, Input과 Output에 맞춰 적절한 코드를 추가해주면 됩니다. 

물론 이 구조가 만능은 아닙니다. 간단한 화면은 괜찮겠지만 화면 또는 기능이 조금만 복잡해져도 스트림이 꼬이지 않도록 신경을 많이 써야 할 것 같습니다.

아키텍처 후보에는 산타토익에서 사용하는 [**Geppetto**](https://github.com/geppetto-ios/Geppetto) 또는 [**ReactorKit**](https://github.com/ReactorKit/ReactorKit)도 있었지만, 신규 입사자의 Brain Heart를 고려하여 다음 기회로 미루기로 했습니다. 두 아키텍처 모두 훌륭하기 때문에 한 번쯤 알아보시는 것을 추천합니다.

이번 글은 여기서 마치겠습니다.  
읽어주셔서 감사합니다! 😆

## 참고자료
- http://minsone.github.io/programming/better-mvvm-architecture-from-kickstarter-oss
- https://github.com/kickstarter/ios-oss
- https://benoitpasquier.com/rxswift-mvvm-alternative-structure-for-viewmodel/