---
layout: post
title: "Github Actions의 Self-hosted runner 사용기"
description: "M1 맥미니를 빌드 머신으로 만들어보자"
author: "kyujin.kim"
date: 2021-04-01
categories: [CI/CD]
tags: [M1, Github Actions, CI/CD, Self-hosted]
---

크로키닷컴의 앱 챕터는 Cloud기반 CI/CD 솔루션인 [비트라이즈(Bitrise)](https://www.bitrise.io)를 사용하여, 유닛 테스트와 빌드 배포를 수행하고 있었습니다.

그러던 어느 날, 몇 시간이 지나도 테스트용 빌드가 배포되지 않아 빌드 대기열을 열어보았더니, 수 많은 테스트와 빌드들이 자신의 차례를 기다리고 있는 것을 목격하게 되었습니다.  

배포가 제대로 되지 않아, 테스트 일정에 영향을 주기도 했고, 개발자 개인 맥북으로 배포하는 좋지 않은 상황도 종종 발생했습니다.

회사가 폭풍 성장하는 모습을 보니, 앞으로 이런 문제가 더 자주 발생할 것 같다는 확신이 들어, 성능 좋은 **로컬 빌드머신**이라는 약을 팔았고, 그것이 잘 팔려서(?) 오랜만에 글을 쓰게 됐습니다. 

> 이 글을 끝까지 읽고 따라오시면, 1시간이 걸리던 배포를 15분으로 단축시킨 마법을 함께 경험하실 수 있습니다 😎

![plan](/assets/images/build-machine-m1/plan.jpg){: width="600"}{: .center-image}
<center>네, 저는 계획이 다 있었습니다 😏</center>


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
Repository의 Actions 메뉴로 진입해서 `New workflow` 버튼을 누릅니다.

![new-workflow](/assets/images/build-machine-m1/new-workflow.png){: .center-image}

`set up a workflow yourself ->` 를 누르면, 웹 에디터로 진입합니다.

![setup-workflow](/assets/images/build-machine-m1/setup-workflow.jpg){: width="600"}{: .center-image}

이렇게 웹 에디터로 워크플로우를 작성할 수도 있고, Repository Root 경로의 `/.github/workflows`에 파일을 직접 추가할 수도 있으니, 편한 방법으로 하면 됩니다.

---

Github에서 아래와 같은 다양한 [샘플 소스](https://github.com/actions/starter-workflows/blob/main/ci/swift.yml)를 제공하고 있습니다. 그리고, [Documents](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions)를 참고하여 다양한 액션을 구현할 수 있습니다.
```yaml
name: Swift # 워크플로우의 이름

on:
  push: # 브랜치에 Push 수행 시 액션 실행 조건
    branches: [ $default-branch ]
  pull_request: # PR이 만들어졌을 때 액션 실행 조건
    branches: [ $default-branch ]

jobs:
  build:
    # 워크플로우를 실행 할 머신, 이곳에 Label을 입력할 수 있고
    # []로 여러개의 라벨을 지정할 수 있음
    runs-on: macos-latest

    steps: # 워크플로우의 실제 동작
    - uses: actions/checkout@v2 # 소스 체크아웃
    - name: Build
      run: swift build -v # 빌드
    - name: Run tests
      run: swift test -v # 테스트
```

### 유닛 테스트
유닛 테스트는 fastlane의 `scan` 명령어로 간단히 실행할 수 있습니다. 웹 에디터에서 새로운 워크플로우를 생성하고, 아래와 같이 워크플로우를 작성합니다.

```yaml
name: Unit Test

on: [ push, pull_request ]

jobs:
  build:
    runs-on: macOS

    steps:
    - uses: actions/checkout@v2

    - name: Cocoapod # 프로젝트에서 Cocoapod을 사용하지 않는다면 빼도 됩니다.
      run: pod install

    - name: Fastlane Scan # Workspace와 Scheme는, 본인의 프로젝트에 맞게 설정해야 합니다.
      run: fastlane scan --workspace "MyProject.xcworkspace" --scheme "MyScheme"
```

작성이 다 됐으면 리모트에 커밋합니다. 그리고 다시 Actions 메뉴로 들어가면 위에서 작성했던 워크플로우가 시작 된 것을 볼 수 있습니다.

![run-action](/assets/images/build-machine-m1/run-action.png){: width="600"}{: .center-image}

진행중인 워크플로우를 클릭해보면, 유닛 테스트가 진행중인 모습을 볼 수 있습니다 😎

![running](/assets/images/build-machine-m1/running.png){: width="600"}{: .center-image}

### 테스트 플라이트 배포
이번에는 fastlane의 `pilot`을 사용해, Testflight로 앱을 업로드 해보려 합니다.  
다시 웹 에디터를 열어 새로운 워크플로우를 만들어봅시다.

아래 워크플로우는, fastlane의 gym을 사용하여 아카이브를 만들고, pilot을 사용해 업로드를 하도록 되어있습니다.

```yaml
name: Uplaod To Testflight

on: 
  push:
    branches:
      - release/* # release로 시작하는 브랜치가 Push됐을 때 동작하는 조건입니다.
  workflow_dispatch:

jobs:
  build:
    runs-on: macOS

    steps:
    - uses: actions/checkout@v2

    - name: Cocoapod # 프로젝트에서 Cocoapod을 사용하지 않는다면 빼도 됩니다.
      run: pod install
      
      
    - name: Fastlane Gym # Workspace와 Scheme는, 본인의 프로젝트에 맞게 설정해야 합니다.
      run: fastlane gym --workspace "MyProject.xcworkspace" --scheme "Scheme" --clean

    - name: Fastlane Pilot
      run: fastlane pilot upload
```

유닛 테스트때와 특히 다른점은, `workflow_dispatch` 라는 조건이 추가되었다는 것 입니다. 어떤 것이 다른지 확인해보기 위해 Actions 메뉴로 진입하고, Upload To Testflight 워크플로우를 선택합니다.

화면 우측에 `run-workflow` 라는 버튼이 보이네요? 😲 궁금하니깐 한번 눌러봐야겠네요.

![run-workflow](/assets/images/build-machine-m1/run-workflow.png){: .center-image}

여기까지 왔으니 브랜치를 선택하고, `Run workflow` 버튼도 눌러봐야겠죠? 😎

![use-workflow-from](/assets/images/build-machine-m1/use-workflow-from.png){: .center-image}

새로고침을 해보니, 배포용 워크플로우가 실행된 모습을 볼 수 있습니다! 🎉  

![upload-to-testflight](/assets/images/build-machine-m1/upload-to-testflight.png){: width="600"}{: .center-image}

이렇게 `workflow_dispatch`를 사용하면, 워크플로우를 수동으로 시작할 수 있습니다.

---

## 마무리
그동안 지그재그 iOS 앱을 기준으로, 비트라이즈 Cloud 환경에서 앱 테스트는 `20분`, 배포는 `60분` 정도가 소요됐는데, M1 맥미니를 Self-hosted runner로 사용하니, 테스트는 `3분`, 배포는 `15분`으로 단축되었습니다 🎉

![gaebi](/assets/images/build-machine-m1/gaebi.png){: width="400"}{: .center-image}

물론 Cloud 환경의 Concurrency를 늘리면 대기열의 문제를 일시적으로 해소할 수 있겠지만, 1시간이 소요되는 긴 배포 시간은 해결할 수 없었습니다. 게다가 매달 비트라이즈에 지불하는 비용도 만만치 않기에, 로컬 빌드 머신이 적절한 타협점이라 생각했습니다.

그럼 저는 이 글을 들고, M1 맥미니 더 사달라고 약을 팔러 가보겠습니다.  
읽어주셔서 감사합니다 🙇🏻‍♂️

---

## 참고자료
- [Github Actions](https://docs.github.com/en/actions)
- [About-self-hosted-runners (Github Actions)](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners)
- [Starter-workflows](https://github.com/actions/starter-workflows)
- [Running CocoaPods on Apple Silicon (StackOverflow)](https://stackoverflow.com/questions/64901180/running-cocoapods-on-apple-silicon-m1)