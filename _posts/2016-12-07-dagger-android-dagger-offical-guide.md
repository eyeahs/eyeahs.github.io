---
layout: post
category: blog
title: '[Dagger Official Reference] Dagger & Android'
date: '2016-12-08 10:30:00 +0900'
tags:
  - android, dagger, di
published: true
comments: true
---

Dagger2가 거의 모든 다른 의존성 주입 프레임워크를 넘어서는 주요 장점 중 하나는 엄격하게 생성된 구현(리플렉션 사용이 없음)이 Android 애플리케이션에 사용될 수 있음을 의미한다는 것이다. 하지만 Android 애플리케이션에서 Dagger를 사용할 때 고려해야할 사항들이 있다.

## 철학

안드로이드를 위해 작성된 코드가 Java 소스일지라도 종종 스타일 측면에서 상당히 다르다. 일반적으로 이런 차이점은 모바일 플랫폼 특유의 성능에 대한 고려에 맞추기 위해 존재한다.

그러나 Android 용 코드에 흔히 적용되는 대부분의 다른 Java 코드에 적용되는 패턴과 반대된다. Effective Java의 많은 조언들 조차 Android에서는 부적절한 것으로 간주된다.

관용적인 코드와 이식 가능한 코드의 목표를 동시에 달성하기 위해, Dagger는 ProGuard를 사용하여 컴파일 된 바이트 코드를 후처리한다. 이는 Dagger가 서버와 Android 모두에서 자연스럽게 느껴지고 보이는 코드를 생성하는 동시에, 다른 툴체인을 사용하여 두 환경에서 모두 효율적으로 동작하는 바이트 코드를 생성한다. 또한 Dagger는 생성된 Java 소스가 ProGuard 최적화와 일관성있게 호환되게 하는 명확한 목표를 가지고 있다.

물론 모든 문제가 그런 방식으로 해결될 수 있는 건 아니지만, 이것은 Android-특정 호환성은 기본 메커니즘에 의해 제공된다.

## tl:dr

Dagger는 Android 사용자가 ProGuard를 사용한다고 가정한다.

## ProGuard 권장 설정

응용프로그램과 관련된 ProGuard 설정을 위해 이 공간을 확인하라.
