---
layout: post
category: blog
published: false
title: Dagger & Android (Dagger Offical Guide)
---
Dagger2가 거의 모든 다른 의존성 주입 프레임워크를 넘어서는 주요 장점 중 하나는 엄격하게 생성된 구현(리플렉션 사용이 없음)이 Android 애플리케이션에 사용될 수 있음을 의미한다는 것이다. 하지만 Android 애플리케이션에서 Dagger를 사용할 때 고려해야할 사항들이 있다.

## 철학

안드로이드를 위해 작성된 코드가 Java 소스일지라도 종종 스타일 측면에서 상당히 다르다. 일반적으로, 이런 차이점은 모바일 플랫폼 
