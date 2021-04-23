---
layout: post
title: "M1 맥미니를 빌드머신으로 만들어보자"
description: "Github Actions의 Self-hosted runner 사용해보기"
author: "kyujin.kim"
date: 2021-04-01
categories: [CI/CD]
tags: [M1, Github Actions, CI/CD, Self-hosted]
---

크로키닷컴의 앱 챕터는 Cloud기반 CI/CD 솔루션인 [비트라이즈(Bitrise)](https://www.bitrise.io)를 사용하여, 유닛 테스트와 빌드 배포를 수행하고 있었습니다.

그러던 어느 날, 몇 시간이 지나도 테스트용 빌드가 배포되지 않아 빌드 대기열을 열어보았더니, 수 많은 테스트와 빌드들이 자신의 차례를 기다리고 있는 것을 목격하게 되었습니다.

> Cloud 환경에서, 지그재그 iOS 앱 테스트는 30분, 배포는 1시간 정도가 소요됩니다.

회사가 폭풍 성장하는 모습을 보니, 앞으로 이런 문제가 더 자주 발생할 것 같다는 확신이 들어, 성능 좋은 **로컬 빌드머신**이라는 약을 팔았고, 그것이 잘 팔려서(?) 오랜만에 글을 쓰게 됐습니다. 

![plan](/assets/images/build-machine-m1/plan.jpg){: width="600"}{: .center-image}
<center>네, 저는 다 계획이 있었습니다 😈</center>

---

## Self-hosted runner?
Github Actions는 사용자가 보유한 컴퓨팅 자원으로 빌드를 수행할 수 있는 **Self-hosted runner**를 지원합니다. 보통은 Cloud 환경에서 동작하는 가상 머신을 사용하지만, 이 글에서는 M1 맥미니를 빌드 머신으로 사용할겁니다.

![spec-of-mac-mini](/assets/images/build-machine-m1/spec-of-mac-mini.png){: .center-image}

> Cloud와 Self-hosted의 환경별 특징 및 장/단점은 [이곳](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners)에 잘 정리되어있습니다.

---

## M1 빌드 머신 준비하기
Xcode, Brew, Cocoapods, Fastlane 등 빌드 및 배포에 필요한 소프트웨어를 설치해줍니다.  
주의해야 할 점은, 소프트웨어에 따라 M1 Chip을 제대로 지원하지 않는 경우가 있습니다.

대표적으로 Cocoapods이 있는데요, Cocoapods을 설치하고 `pod install`을 실행하면 무시무시한 에러를 만나게 됩니다 😵  
```
LoadError - dlsym(0x7f8926035eb0, Init_ffi_c): symbol not found - /Library/Ruby/Gems/2.6.0/gems/ffi-1.13.1/lib/ffi_c.bundle
/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
...
```
하지만, 이 에러는 터미널을 [Rosetta로 실행](https://support.apple.com/ko-kr/HT211861)하고, 아래 명령어를 입력하면 간단하게 해결할 수 있습니다.
```shell
> sudo gem install cocoapods
> sudo gem install ffi
> pod install # 성공 🎉
```

---

## Github Actions에 빌드 머신 등록하기
빌드 머신의 기본 설정이 완료되었다면, 이제 Github Actions에 등록 할 차례입니다.  
Action을 사용 할 Repository의 `Settings` 메뉴로 진입합니다.

![settings](/assets/images/build-machine-m1/settings.png){: width="600"}{: .center-image}

Settings 화면의 왼쪽 사이드바에 있는 `Actions` 메뉴를 선택합니다.

![actions](/assets/images/build-machine-m1/actions.png){: .center-image}

Actions 화면 제일 아래에 있는 `Add Runner` 버튼을 누릅니다.

![add-runner](/assets/images/build-machine-m1/add-runner.png){: width="600"}{: .center-image}

macOS와 X64 Architecture를 선택합니다.

![os-arch](/assets/images/build-machine-m1/os-arch.png){: .center-image}

마지막으로, 빌드 머신의 터미널을 열고 `Download`와 `Configure`항목에 있는 쉘 스크립트를 순서대로 실행합니다.

`config.sh` 스크립트를 실행하면, 아래 이미지와 같이 설정 값을 입력할 수 있는 프롬프트가 보일겁니다. 구분하기 쉬운 이름을 입력하고 Enter. (⚠️ 이름은 수정이 불가능합니다 ⚠️)

![install](/assets/images/build-machine-m1/install.png){: width="600"}{: .center-image}

다음은, 빌드 머신을 분류할 수 있는 라벨을 입력하는 과정 입니다. 라벨은 언제든 수정할 수 있으니 일단 Enter.

![label](/assets/images/build-machine-m1/label.png){: width="600"}{: .center-image}

마지막으로, 소스코드를 체크아웃 할 경로를 지정해줍니다. 기본값을 사용하거나, 원하는 경로를 지정한 후 Enter를 누르면, 설정이 저장되었다는 메시지를 볼 수 있습니다.

![finish](/assets/images/build-machine-m1/finish.png){: width="600"}{: .center-image}

Github Actions에 빌드 머신 등록이 잘 됐는지 확인하기 위해, 다시 Actions 메뉴로 진입하여, 화면 하단의 `Self-hosted runners` 항목을 확인해봅시다.

![runners](/assets/images/build-machine-m1/runners.png){: width="600"}{: .center-image}

조금전에 등록한 빌드 머신이 idle 상태로 보이면 성공적으로 등록이 된 것 입니다 🎉

---

## Workflow 작성하기
TBD

## 빌드 돌려보기
TBD

## 마무리
TBD

## 참고자료
- [About-self-hosted-runners (Github Actions)](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners)
- [Running CocoaPods on Apple Silicon (StackOverflow)](https://stackoverflow.com/questions/64901180/running-cocoapods-on-apple-silicon-m1)