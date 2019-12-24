---
layout: post
title: "[iOS] 레퍼런스 카운팅 (Reference Counting)"
description: "iOS의 메모리 관리 기법에 대하여"
author: "kyujin.kim"
date: 2019-10-31
categories: [iOS, Optimizing]
tags: [iOS, ARC, Memory]
---

'iOS와 OS X의 메모리 관리와 멀티스레딩 기법'이라는 책을 접하게 되어 내용을 정리하고자 합니다.  
이번 글의 주제는 메모리 관리 기법의 기본인 '레퍼런스 카운팅'과 '메모리 누수' 입니다.

## 레퍼런스 카운팅
레퍼런스 카운팅(Reference Counting)은 iOS(Mac OS)에서 사용하는 메모리 관리 방법입니다. 이 방법은 객체를 참조하는 수(Reference Count)의 증가 또는 감소를 통해 메모리를 관리합니다.

객체를 생성하거나 참조 하게되면 레퍼런스 카운트는 `+1 증가`하게 되고, 해당 객체의 사용이 끝나거나 객체를 반환(release)할 경우 레퍼런스 카운트가 `-1 감소`됩니다.

최종적으로 해당 객체의 `레퍼런스 카운트가 0이 될 경우 해당 객체는 메모리에서 소멸`됩니다.  
객체의 생성부터 소멸까지의 과정은, 아래 그림을 통해 확인할 수 있습니다.

![image1](/assets/images/optimizing/img1.png)

## 메모리 누수 (Memory Leak)
프로그래머가 동적으로 할당 한 메모리는, 더 이상 필요하지 않을 경우 해제 하는 것을 원칙으로 합니다. 할당 된 메모리는 사용하지 않을 경우 해제되어야 하지만, 프로그래머의 실수로 메모리를 해제하지 않거나, 해제될 수 없는 경우 ['메모리 누수(Memory Leak)'](http://ko.wikipedia.org/wiki/메모리_누수)가 발생하게 됩니다.

메모리가 제때 해제되지 않으면 메모리 부족으로 애플리케이션이 강제 종료되기도 합니다. PC에 비해 메모리의 용량이 작은 모바일 환경에서의 메모리 누수는 치명적이므로, 메모리 관리에 신경을 써야 합니다. 

ARC의 등장 이후 조금 덜 신경써도 되지만, 여전히 메모리 누수는 경계 대상입니다.

![image2](/assets/images/optimizing/img2.jpeg)

## 마치며
이번에는 레퍼런스 카운팅과 메모리 누수에 대해 간단히 알아보았습니다. 다음 글에서는 레퍼런스 카운트의 관리 방법에 대해 알아보겠습니다.

감사합니다.

> (이 포스트는 ARC가 출시되기 전 최초 작성되었으며, 2019년 10월 31일 재작성 되었습니다)

## 참고자료
[Advanced Memory Management Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html)