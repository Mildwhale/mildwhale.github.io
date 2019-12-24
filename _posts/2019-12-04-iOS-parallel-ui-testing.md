---
layout: post
title: "[iOS] Parallel UI Testing"
description: "UI 테스트 시간을 단축할 수 있는 방법을 소개합니다"
author: "kyujin.kim"
date: 2019-12-04
categories: [iOS, Testing]
tags: [iOS, UI Test]
---

뤼이드의 iOS 챕터는 '[산타토익](https://apps.apple.com/kr/app/산타토익-비인간적-점수상승/id1148006701)' iOS 앱을 배포하기 전에 사용자가 주로 사용하는 기능 위주로 문제가 없는지 확인하는 체크리스트가 있습니다.

이 체크리스트는 기본적으로 사람이 하나하나 확인해야 하지만, iOS 챕터에서는 체크리스트 테스트를 자동화하여 개발자의 리소스를 확보하고, 다른 일에 더 투자할 수 있는 환경을 만들기 위해 노력하고 있습니다.

지금까지 20개의 UI 테스트를 만들었으며, 이 테스트가 완료되기까지 약 1시간 정도의 시간이 소요됩니다. 개발자가 각자의 커밋을 올리게 되면(PR) 빌드머신의 큐에 쌓이게 되고, 빌드가 완료되기까지 최소 1시간 이상을 기다려야 한다는 얘기입니다.

뤼이드의 iOS 챕터는 제품의 안정성과 업무의 효율성을 높이기 위해 앞으로도 더 많은 테스트 코드를 작성할 계획을 가지고 있습니다.

문제는 테스트의 개수가 늘어날수록 테스트 시간도 비례한다는 것인데, 한정된 시간에 빌드가 계속 쌓여가다 보면 나중에는 테스트 결과를 확인하지 않고 Merge를 하게 되는 상황이 발생할 수도 있습니다. 결과는 안봐도 뻔하겠죠?

그래서 이 글에서는 테스트 시간을 획기적으로 줄일 수 있는 '**테스트 병렬화**'에 대해서 알아보려 합니다.

![image4](/assets/images/iOS-parallel-ui-testing/img4.gif){: .center-image}

## 테스트 비교하기
기존의 UI 테스트는 1개의 기기로 모든 테스트를 검증했다면, 병렬 테스트를 적용하면 동시에 2개 이상의 기기에서 테스트를 진행할 수 있게 됩니다. 운 좋게도 **Xcode 9 부터 병렬 테스트를 지원**하고 있습니다.

### 단일 테스트
병렬화를 적용하지 않은 상태에서 UI 테스트는 약 **30분**이 소요됩니다.
![image1](/assets/images/iOS-parallel-ui-testing/img1.png)

### 병렬 테스트
병렬화를 적용하여 총 3개의 시뮬레이터로 테스트를 진행하면 **10분** 정도가 소요됩니다. 시뮬레이터가 3배가 되니 시간은 1/3이 되었네요! 🎉
![image2](/assets/images/iOS-parallel-ui-testing/img2.png)

## 병렬 테스트 활성화
병렬 테스트를 활성화하는 방법은 매우 간단합니다!

### Xcode
![image3](/assets/images/iOS-parallel-ui-testing/img3.png)

1. Edit Scheme 메뉴 진입
2. Test 메뉴 선택
3. 병렬화하고자 하는 Tests의 `Options...` 선택
4. `Execute in parallel on Simulator`에 체크

### Fastlane
뤼이드의 iOS 챕터에서는 Fastlane을 사용하여 배포하고 있는데요, Scanfile에 아래 명령어를 추가하여 병렬 테스트를 활성화하고, 시뮬레이터의 개수를 4개로 지정하고 있습니다.

`xcargs("-parallel-testing-enabled YES -parallel-testing-worker-count 4")`

추가 정보는 [Scan - Fastlane docs](https://docs.fastlane.tools/actions/scan/) 에서 찾아보실 수 있습니다.

## Tip & Tricks
병렬 테스트를 적용해서 시간은 어느 정도 단축되었지만, 아직 조금 더 최적화 할 수 있는 방법이 남아있습니다!

### 시뮬레이터 개수 변경
UI 테스트에 사용하는 시뮬레이터의 개수는 시스템의 성능에 따라 자동으로 결정됩니다. 물론 개수를 직접 설정하는 방법도 제공됩니다.

Fastlane 항목에서 소개했던 `-parallel-testing-worker-count 4`는 시뮬레이터의 개수를 4개로 설정해주는 명령어입니다.

너무 많은 개수를 지정하면 시뮬레이터를 실행하는 데 모든 자원을 사용해버려서 테스트가 진행되지 않습니다. 빌드머신이 감당할 수 있는 최대의 개수를 지정해주는 것이 가장 좋겠죠?

### 테스트 코드 분리
병렬화 테스트는 Class 단위로 테스트를 진행합니다. 즉, 하나의 Class에 너무 많은 테스트 함수가 구현되어 있다면 이것을 적절히 나누어 주는 것이 좋습니다.

시뮬레이터는 3개인데 테스트 함수가 1개의 Class에 모두 구현되어있다면, 병렬 테스트의 장점을 전혀 살리지 못하게 됩니다 😭

### waitForExistence
비동기 동작을 검증하기 위해 sleep 또는 Thread.sleep을 사용하기 보다는 [waitForExistence:timeout](https://developer.apple.com/documentation/xctest/xcuielement/2879412-waitforexistence) 함수를 사용하는 것이 좋습니다. 

exists외에 다른 Property의 변화를 기다리고 싶다면 아래 코드가 도움이 될 겁니다.
<script src="https://gist.github.com/Mildwhale/296cfdc0d6df902b4061a82e2e7883f1.js"></script>

---

읽어주셔서 감사합니다 😁

## 참고자료
- [UI Testing in Xcode - WWDC 2015](https://developer.apple.com/videos/play/wwdc2015/406)
- [What's New in Testing - WWDC 2017](https://developer.apple.com/videos/play/wwdc2017/409)
- [Parallel testing: get feedback earlier, release faster](https://medium.com/azimolabs/parallel-testing-get-feedback-earlier-release-faster-b66d4dd08930)
- [Xcode 10 parallel testing support](https://github.com/fastlane/fastlane/issues/13394)