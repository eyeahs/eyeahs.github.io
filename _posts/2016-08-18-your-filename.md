---
layout: post
category: blog
published: false
title: ''
---
Reactive Programming을 조사하는 동안 내가 찾은 거의 모든 글들이 이 것은 배우기 어렵다는 의견으로 시작한한다. Reactive Programming에 대해 거의 또는 아무것도 아는 것이 없는 사람들을 위한 글은 찾기 어려웠다. 이 글은 신참자를 위해 Android에서 RxJava를 사용하여 Reactive Programming의 기초를 이해할 수 있도록 하려 한다.

# Reactive Programming이란 무엇인가?
**Reactive programming은 비동기 데이터 스트림(stream)을 이용한 프로그래밍이다.**

# 잠깐만. 그건 콜백을 통해서도 쉽게 할 수 있다. 그럼 Reactive programming이 다른건 무엇인가?
그렇다, 이 개념은 새로운 것이 아니다. 이는 명령적(imperatively)으로도 할 수 있고 - 그리고 보통 그렇다.

Let’s not only think about callbacks, but also the supporting mechanism that is required to get it up and running. 이 지원은 상태를 관리하고 상태 변경의 부작용(명령형 접근)을 걱정하는 것을 수반한다. 이 고려 사항들은 소프트웨어 공동체에서 수없이 많은 에러의 원인이 되어 왔다. Reactive programming은 함수적 접근을 취한다; 이는 전역 상태와 부작용들을 방지하며 대신, 스트림과 일을 한다.

# 스트림은 무엇인가?
> “Everything flows, nothing stands still” — Heraclitus

스트림은 데이터의 연속을 나타낸다. 교통망을 고려해보자. 특정 고속도로의 교통 수단은 가끔은 정체되는 끝없이 이어지는 객체의 스트림을 형성한다. with the occasional bottleneck. Reactive programming에서 우리는 데이터의 지속적인 흐름-스트림-을 받는다. 그리고 우리는 스트림에 적용하기 위해 작업(operation)들을 제공한다. 데이터의 출처는 문제가 되지 않는다(않아야 한다).

_스트림은 어디에나 있고(ubiquitous), 어떤 것도 스트림이 될 수 있다: 변수, 사용자 입력, 속성, 캐쉬, 데이터 구조 등등._

## 선언형 프로그래밍과 명령형 프로그래밍은 무엇인가?
* 명령형은 어떻게 해야 하는 지를 설명한다.
* 선언형은 무엇을 해야 하는 지를 말한다.


