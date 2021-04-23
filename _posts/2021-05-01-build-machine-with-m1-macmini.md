---
layout: post
title: "M1 맥미니를 빌드머신으로 만들어보자"
description: "Self-hosted build machine with M1 (feat. Github Actions)"
author: "kyujin.kim"
date: 2021-04-01
categories: [CI/CD]
tags: [M1, Github Actions, CI/CD, Self-hosted]
---

크로키닷컴의 앱 챕터에서는 Cloud기반 CI/CD 솔루션인 [비트라이즈(Bitrise)](https://www.bitrise.io)를 사용하여, 지속적인 통합과 주기적 배포를 수행하고 있었습니다.

그러던 어느날, 몇 시간이 지나도 테스트용 빌드가 배포되지 않아 빌드 대기열을 열어보았더니, 수 많은 빌드와 'PR Test'들이 자신의 차례를 기다리고 있는 것을 목격하게 되었습니다.

> 비트라이즈 Cloud 환경에서, 지그재그 iOS 앱을 기준으로 Test에는 30분, 배포에는 1시간이 소요됩니다.

회사가 폭풍 성장하는 것을 보며, 앞으로 이런 문제가 더 자주 발생할 것 같다는 느낌적인 느낌으로 성능 좋은 빌드머신 도입을 **외쳤습니다**.

![please](https://media.giphy.com/media/cmrWcwC3A4ryH437Ru/giphy.gif){: width="400"}{: .center-image}

## Self-hosted runner?
다양한 CI/CD 솔루션들이 제공하는 Cloud 환경이 아닌, 사용자가 가진 자원으로 직접 구성할 수 있는 빌드 머신을 Self-hosted runner 라고 합니다. 환경별 차이점 및 장/단점은 [이곳](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners)에 잘 정리되어있습니다.

## M1 빌드 머신 준비하기
Xcode와 같이 iOS 프로젝트 빌드에 필요한 프로그램들을 설치합니다. 

## Github Actions와 빌드 머신 연결하기
![github-setting](https://docs.github.com/assets/images/help/organizations/organization-settings-tab.png)

## Workflow 작성하기
TBD

## 빌드 돌려보기
TBD

## 참고자료
- [About-self-hosted-runners (Github Actions)](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners)
- [Running CocoaPods on Apple Silicon (StackOverflow)](https://stackoverflow.com/questions/64901180/running-cocoapods-on-apple-silicon-m1)