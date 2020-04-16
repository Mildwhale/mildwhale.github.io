---
layout: post
title: "[iOS][RxSwift] Input과 Output을 사용한 MVVM 아키텍처 만들기"
description: "iOS의 MVVM 아키텍처는 왜 제각각일까?"
author: "kyujin.kim"
date: 2020-04-16
categories: [iOS, RxSwift]
tags: [iOS, RxSwift, MVVM]
---

## MVVM?
신규 서비스의 iOS 앱 개발을 담당하게 되어, 아키텍처에 대해 많은 고민을 했고 Rx+MVVM 아키텍처를 선택하게 됨
익숙함이 가장 큰 무기인 MVC(~~Massive~~ View Controller)는 매우 효율적이지만, 유지보수 과정에서 큰 고통을 받을 것이 분명하여 탈락.
나름의 기준으로 요즘 대세라고 생각되는 MVVM을 도입하기로 함.
Reactive함과 Binding의 편의성을 위해 RxSwift도 같이 사용!

산타토익에서 사용하는 [**Geppetto**](https://github.com/geppetto-ios/Geppetto) 또는 [**ReactorKit**](https://github.com/ReactorKit/ReactorKit)을 사용해볼까 싶었지만, 투머치인것 같아서 일단 접어두었음.

그런데 Rx+MVVM으로 앱을 개발해본 경험이 거의 없다보니, 다른 사람들은 어떻게 하나 참고하려고 찾아봤더니,, 웬걸? 사람마다 다 다르다 ...
이것저것 보다보니 Input과 Output Protocol을 적용한 방식이 눈에 띄었음.

이 구조를 사용하여 간단한 뷰 모델을 만들어보고, 어떻게 사용하는지 알아봅시다.

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

뷰 모델 초기화에 필요한 데이터인 **Dependency**, 입력을 담당하는 **Input**과 출력을 담당하는 **Output**을 associatedtype으로 정의합니다.

그리고 이 프로토콜을 따르는 뷰모델을 만들어볼 차례입니다.

### ViewModel
```swift
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

    init(dependency: Dependency) {
        self.dependency = dependency

        let nameText$ = BehaviorSubject<String?>(value: nil)
        let emailText$ = BehaviorSubject<String?>(value: nil)
        let isConfirmEnabled$ = Observable.combineLatest(nameText$, emailText$)
            .map(validation)
            .asDriver(onErrorJustReturn: false)

        self.input = Input(nameText: nameText$.asObserver(),
                           emailText: emailText$.asObserver())
        self.output = Output(isConfirmEnabled: isConfirmEnabled$)
        self.nameText$ = nameText$
        self.emailText$ = emailText$
    }
}

private func validation(name: String?, email: String?) -> Bool {
    return name?.isEmpty == false && email?.isEmpty == false
}
```

ViewModelType을 구현하는 뷰모델을 만들어보았습니다. 모든 스트림을 init(dependency:)에서 정의하는 모습을 볼 수 있습니다. 코드의 마지막줄에는 이름과 이메일 주소가 비어있는지 확인하는 함수가 구현되어있네요.

그럼 뷰에서는 이 뷰모델을 어떻게 쓰면 되는지도 알아봐야겠죠?

### View
```swift
import UIKit
import RxSwift
import RxCocoa

final class View: UIViewController {
    private let nameTextField: UITextField = UITextField()
    private let emailTextField: UITextField = UITextField()
    private let confirmButton: UIButton = UIButton()
    
    let disposeBag = DisposeBag()
    var viewModel: MyViewModel = MyViewModel(dependency: MyViewModel.Dependency(name: nil, email: nil))

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

## 마무리
신규 서비스의 iOS 앱 개발을 담당하게 되어, 아키텍처에 대해 많은 고민을 했고 Rx+MVVM 아키텍처를 선택하게 됨
익숙함이 가장 큰 무기인 MVC(~~Massive~~ View Controller)는 매우 효율적이지만, 유지보수 과정에서 큰 고통을 받을 것이 분명하여 탈락.
나름의 기준으로 요즘 대세라고 생각되는 MVVM을 도입하기로 함.
Reactive함과 Binding의 편의성을 위해 RxSwift도 같이 사용!

ReactorKit이나 산타토익에서 사용하는 Geppetto를 사용해볼까 싶었지만, 투머치인것 같아서 일단 접어두었음.

그런데 Rx+MVVM으로 앱을 개발해본 경험이 거의 없다보니, 다른 사람들은 어떻게 하나 참고하려고 찾아봤더니,, 웬걸? 사람마다 다 다르다 ...
이것저것 보다보니 Input과 Output Protocol을 적용한 방식이 눈에 띄었음.

이 구조를 사용하여 간단한 뷰 모델을 만들어보고, 어떻게 사용하는지 알아봅시다.




## 참고자료
https://github.com/geppetto-ios/Geppetto
https://github.com/kickstarter/ios-oss
https://benoitpasquier.com/rxswift-mvvm-alternative-structure-for-viewmodel/
http://minsone.github.io/programming/better-mvvm-architecture-from-kickstarter-oss
https://github.com/ReactorKit/ReactorKit