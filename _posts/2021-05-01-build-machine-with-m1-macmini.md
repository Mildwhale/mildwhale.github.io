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

![line-up](https://media.giphy.com/media/3orif9DJNKDfXqXbBS/giphy.gif){: width="400"}{: .center-image}

> 지그재그 iOS 앱을 기준으로 PR Test에는 30분, 배포에는 1시간이 소요됩니다.

회사가 폭풍 성장하면서, 더 많은 인원이 동시다발적으로 여러 Feature를 개발 할 것이고, 그 Feature들을 테스트 하기 위해 수많은 빌드를 배포해야 하는데, 비트라이즈로는 감당할 수 없는 수준에 도달했다는 생각이 들어 성능 좋은 빌드머신 도입을 외쳤습니다.

## Q. Concurrency를 늘리고, 성능이 좋은 가상머신 옵션을 선택하면 되지 않나요?

