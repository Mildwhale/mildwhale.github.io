---
layout: post
title: "[번역] Increasing Performance by Reducing Dynamic Dispatch"
description: "동적 디스패치를 줄여 성능을 향상시키는 방법"
author: "kyujin.kim"
date: 2020-01-31
categories: [iOS, Translation, Optimizing]
tags: [iOS, Access Control, Final]
---

다른 많은 언어들과 같이, Swift는 클래스가 슈퍼 클래스에 선언된 메서드와 프로퍼티를 재정의 할 수 있도록 허용하고 있습니다. 이는 프로그램이 런타임에 어떤 메서드나 프로퍼티를 참조해야 하는지 결정하고, 간접적인 호출 또는 간접적인 접근을 해야한다는 것을 의미합니다.  

동적 디스패치(dynamic dispatch)라고 하는 이 기술은 일정한 양의 런타임 오버헤드를 대가로 언어의 표현력을 향상 시킵니다. 이러한 오버헤드는 성능에 민감한 코드에게 바람직하지 않습니다. 

이 포스트는 역동성을 배제 하여 성능을 향상시키는 세 가지 방법 `final`, `private` 그리고 전체 모듈 최적화(Whole Module Optimization)를 소개하는 글 입니다.

> 원문 링크: [Increasing Performance by Reducing Dynamic Dispatch](https://developer.apple.com/swift/blog/?id=27)  

다음 예제를 보세요:

```swift
class ParticleModel {
    var point = ( 0.0, 0.0 )
    var velocity = 100.0

    func updatePoint(newPoint: (Double, Double), newVelocity: Double) {
        point = newPoint
        velocity = newVelocity
    }

    func update(newP: (Double, Double), newV: Double) {
        updatePoint(newP, newVelocity: newV)
    }
}

var p = ParticleModel()
for i in stride(from: 0.0, through: 360, by: 1.0) {
    p.update((i * sin(i), i), newV:i*1000)
}
```

컴파일러는 아래에 쓰여진대로 동적으로 디스패치 된 호출을 내보냅니다:

1. `p`의 `update` 호출.
2. `p`의 `updatePoint` 호출.
3. `p`의 튜플인 `point` 프로퍼티 가져오기.
4. `p`의 `velocity` 프로퍼티 가져오기.

`ParticleModel` 서브 클래스의 `point` 또는 `velocity`를 계산된 프로퍼티(computed property)로 오버라이드 하거나 `updatePoint()` 또는 `update()` 메서드를 새로운 구현으로 오버라이드 할 수 있기 때문에 동적 호출이 필요합니다. 이것은 당신이 이 코드를 봤을 때 예상하지 못했던 것일 수도 있습니다. 

Swift의 동적 디스패치 호출은 메서드 테이블에서 함수를 찾은 다음 간접 호출을 통해 동작하도록 구현되어있습니다. 이 간접 호출은 직접 호출보다 느리게 실행되며, 컴파일러의 다양한 최적화를 방해하기 때문에 비용을 더 비싸게 만듭니다. 성능이 중요한 코드에서 동적인 동작이 필요하지 않을 때, 이를 제한하여 성능을 향상시킬 수 있는 기술들이 있습니다.

### 선언을 재정의 할 필요가 없는 경우 final을 사용하세요.

`final` 키워드는 클래스, 메서드 또는 프로퍼티의 선언을 재정의할 수 없다고 명시하고 제한합니다. 이를 통해 컴파일러는 동적 디스패치(dynamic dispatch indirection)를 안전하게 제거할 수 있습니다. 

예를 들어, `point`와 `velocity`는 객체의 저장 프로퍼티(stored property)를 통해 직접적으로 접근할 수 있으며 `updatePoint()`는 직접 함수 호출을 통해 호출됩니다. 반면, `update()`는 여전히 동적 디스패치를 통해 호출되며, 서브 클래스가 커스터마이즈 된 기능으로 `update()`를 재정의 할 수 있습니다.

```swift
class ParticleModel {
    final var point = ( x: 0.0, y: 0.0 )
    final var velocity = 100.0

    final func updatePoint(newPoint: (Double, Double), newVelocity: Double) {
        point = newPoint
        velocity = newVelocity
    }

    func update(newP: (Double, Double), newV: Double) {
        updatePoint(newP, newVelocity: newV)
    }
}
```

클래스 자체에 속성을 추가하여 전체 클래스를 `final`로 표시 할 수 있습니다. 이것은 클래스의 서브 클래싱을 불가능하게 하며, 클래스의 모든 함수와 프로퍼티가 `final`임을 암시합니다.

```swift
final class ParticleModel {
    var point = ( x: 0.0, y: 0.0 )
    var velocity = 100.0
    // ...
}
```

### private 키워드를 적용하여 한 파일안에 참조된 선언에 대해 final을 유추하십시오.

`private` 키워드를 선언부에 사용하면 선언의 가시성은 현재 파일로 제한됩니다. 이를 통해 컴파일러는 잠재적으로 재정의 할 수 있는 모든 선언들을 찾을 수 있습니다. 이러한 재정의 선언이 없는 경우 컴파일러는 자동으로 `final` 키워드를 추론하고, 매서드와 프로퍼티 접근에 대한 간접 호출을 제거할 수 있습니다.

현재 파일에 `ParticleModel`을 재정의하는 클래스가 없다고 가정했을 때, 컴파일러는 동적으로 디스패치된 모든 호출을 직접 호출하는 private 선언으로 대체 할 수 있습니다.

```swift
class ParticleModel {
    private var point = ( x: 0.0, y: 0.0 )
    private var velocity = 100.0

    private func updatePoint(newPoint: (Double, Double), newVelocity: Double) {
        point = newPoint
        velocity = newVelocity
    }

    func update(newP: (Double, Double), newV: Double) {
        updatePoint(newP, newVelocity: newV)
    }
}
```

위의 예제와 같이 `point`와 `velocity`는 직접적으로 접근되며 `updatePoint()`도 직접 호출 됩니다. `final`에서 얘기했던것 처럼 `update()`는 `private`이 아니기 때문에 간접적으로 호출 됩니다.

`final`과 마찬가지로, 클래스 선언 자체에 `private`속성을 적용할 수 있으며 클래스의 모든 프로퍼티와 메서드도 클래스 처럼 `private`이 됩니다.

```swift
private class ParticleModel {
    var point = ( x: 0.0, y: 0.0 )
    var velocity = 100.0
    // ...
}
```

### 전체 모듈 최적화를 사용하여 내부 선언의 final을 추론하세요.

`internal`접근으로 선언(아무것도 선언되지 않은 경우의 기본값)하면 선언된 모듈 내부에서만 볼 수 있게됩니다. Swift는 보통 모듈을 구성하는 파일을 개별적으로 컴파일 하기 때문에, 컴파일러가 `internal` 선언이 다른 파일에서 재정의 되는지의 여부를 확인할 수 없습니다. 그러나, 전체 모듈 최적화를 활성화 하면 모든 모듈이 동시에 컴파일 됩니다. 이를 통해 컴파일러는 전체 모듈에 대해 추론을 할 수 있고, 눈에 보이는 재정의가 없는 경우 `internal`이 있는 선언에 대해 `final`을 유추할 수 있습니다.

원래 코드 스니펫으로 돌아가서, 이번에는 `ParticleModel`에 `public` 키워드를 추가했습니다.

```swift
public class ParticleModel {
    var point = ( x: 0.0, y: 0.0 )
    var velocity = 100.0

    func updatePoint(newPoint: (Double, Double), newVelocity: Double) {
        point = newPoint
        velocity = newVelocity
    }

    public func update(newP: (Double, Double), newV: Double) {
        updatePoint(newP, newVelocity: newV)
    }
}

var p = ParticleModel()
for i in stride(from: 0.0, through: times, by: 1.0) {
    p.update((i * sin(i), i), newV:i*1000)
}
```

전체 모듈 최적화를 사용하여 이 스니펫을 컴파일 할 때, 컴파일러는 `point`, `velocity`, 그리고 `updatePoint()` 메서드 호출에 대해 `final`을 추론할 수 있습니다. 대조적으로, `update()`는 public 접근 권한이 있기 때문에 `final` 추론이 불가능합니다.

## 참고자료
- [Increasing Performance by Reducing Dynamic Dispatch - Apple Blog](https://developer.apple.com/swift/blog/?id=27)
- [Access Control - Swift.org](https://docs.swift.org/swift-book/LanguageGuide/AccessControl.html)
- [Optimizing Swift Performance - WWDC 2015](https://developer.apple.com/videos/play/wwdc2015/409/?time=1399)
