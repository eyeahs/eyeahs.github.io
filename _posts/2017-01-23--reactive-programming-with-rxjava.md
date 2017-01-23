---
layout: post
category: blog
date: "2017-1-23 15:15:00 +0900"
published: true
title: '[번역] Reactive Programming with RxJava'
comments: true
---

# Foreword

2005년 10월 28일 새로이 임명된 Microsoft의 chief architect인 Ray Ozzie는 "The Internet Services Disruption"이라는 주제의 지금은 악명이 자자한 메모를 직원들에게 이메일로 보냈다. 이 메모에서 Ray Ozzie는 기본적으로 Microsoft, Google, Facebook, Amazon 및 Netflix와 같은 웹을 자사 서비스 전달의 주요 채널로 이용하는 기업들이 존재하는 세상이 어떻게 보이는지에 대해 개략적으로 설명한다.

개발자 관점에서 볼 때 Ozzie는 대기업의 임원들에게 주목할 만한 성명서를 작성하였다.

`복잡성이 죽인다. 개발자의 삶을 앗아 가고 제품의 계획, 개발 및 테스트를 어렵게 만들고 보안 문제가 나타나며 최종 사용자와 관리자의 좌절을 야기한다.`

우선 우리는 2005년에는 큰 IT기업이 SOAP, WS-* 및 XML과 같이 환각을 일으키는 복잡한 기술에 깊은 관심을 가지고 있음을 고려해야 한다. 이는 "microservice"라는 단어가 아직 만들어지기 전이며 더 작은 것들을 복잡한 서비스로 비동기적으로 구성하는 것과 실패, 지연, 보안 그리고 효율 같은 관심사를 처리하는 것의 복잡성을 개발자가 관리하는 것을 돕는 간단한 기술이 아직 지평선 위로 나타나지 않았다.

Microsoft의 클라우드 프로그래밍 팀에게 Ozzie의 메모는 대규모의 비동기 및 데이터 집중적 인터넷 서비스 아키텍처를 구축하기 위한 단순한 프로그래밍 모델을 개발하는데 주력을 다하게 하는 거친 모닝콜이였다. 많은 부정 후에 동기적 컬렉션을 위한 Iterable/Iterator 인터페이스을 이중화_dualize_함으로써 비동기 데이터 스트림을 변환하고 결합하기 위한 map, filter, scan, zip, groupBy 등의 모든 친숙한 시퀀스 연산자_operator_로 비동기 이벤트 스트림을 나타내기 위한 한 쌍의 인터페이스를 얻을 수 있었다. 그래서 Rx는 2007년 여름 어딘가에서 태어났다.

수많은 디자인적 선택들을 탐구했던 2년간의 격렬한 hackathon 이후 Rx.NET을 2009년 11월 18일에 출하했다. 이후 곧 Rx를 Windows Phone 7을 위한 Microsoft.Phone으로 이전시켰고 JavaScript와 C++같은 다양한 언어를 위한 Rx를 구현하고 Ruby와 Objective-C를 위한 실험적 버전에 잠깐 손을 대보았다.

Microsoft에서 최초의 Rx 사용자는 Jafar Husain이며 그가 2011년 Netflix에 합류했을 때 이 기술을 함께 도입하였다. Jafar는 사내에 Rx의 복음을 설교하였고 결국 Netflix UI의 클라이언트-사이드 스택을 비동기 스트림 처리를 채택하기 위해 재설계하였다. 또한 우리 모두에게 다행스럽게도 그는 Netflix의 미들-티어 API에서 작업하는 Ben Christensen에게 그의 열의를 전달하였다. Netflix는 미들-티어에서 Java를 사용하였기 때문에 Ben은 2012년에 RxJava에서 작업하기 시작했고 지속적인 오픈 소스 개발을 위해 2013년 초에 코드베이스를 Github에 옮겼다. Microsoft의 또다른 Rx 얼리어답터는 Paul Betts이며 그가 Github로 옮겼을 때 그는 2012년 봄 Justin Spahr-Summers같은 Github의 동료들에게 Objective-C용 ReactiveCocoa를 구현 및 출시할 것을 설득했다.

Rx가 업계에서 인기를 얻자 2012년 가을에 우리는 Rx.NET를 오픈소스로 만들도록 Microsoft Open Tech을 설득하였다. 그 후 곧 나는 Microsofts를 떠나 Applied Duality를 시작하고 Rx를 비동기적 실시간 데이터 스트림 처리를 위한 표준 크로스-랭귀지, 크로스-플랫폼 API로 만들기 위해 내 시간의 100%를 주력하였다.

2016년으로 넘어가면 Rx의 인기와 사용이 급증했다. Netflix API의 모든 트래픽은 모든 내부 서비스 트래픽을 차단하는 Hystrix 내고장성_fault-tolerance_ 라이브러리처럼 RxJava에 의존하며, 동류의 반응형 라이브러리 RxNettry와 Mantis를 경유한다. Netflix는 이제 모든 내부 서비스를 머신과 프로세스 경계를 넘어서 연결하는 완전히 반응적 네트워크 스택을 만들고 있다. RxJava는 안드로이드 영역에서도 역시 극도로 성공적이다. SoundCloud, Square, NTY, Seatgeek와 같은 회사는 모두 자사의 Android 앱을 위해 RxJava를 사용하며 RxAndroid 확장 라이브러리에 기여하고 있다. Couchbase와 Splunk같은 noSql 벤더 역시 자신의 데이터 접근 계층에 Rx기반 바인딩을 제공한다. RxJava를 채택한 또다른 Java 라이브러리로는 Camel Rx, Square Retrofit 및 Vert.x가 있다. JavaScript 커뮤니티에서는 RxJS가 널리 사용되고 있으며 Angular 2같은 인기있는 프레임워크를 강화한다. 커뮤니티는 환상적인 Marble Diagram과 David Gross(@CallHimMoorlock)의 설명처럼 다국어로 Rx 구현에 대한 정보를 찾을 수 있는 웹사이트를 뒷바침한다.

처음 시작한 이래 Rx는 개발자 커뮤니티의 요구와 제공으로 발전했다. .NET의 원본 Rx구현 비동기 이벤트 스트림의 변형에 정확하게 중점을 두었으며 backpressure를 필요로 하는 시나리오를 위해 비동기 열거형_enumerable_을 사용했다. Java에서는 async/await에 대한 언어적 지원이 없었으므로 커뮤니티는 Observer와 Observable 타입을 reactive pull 개념으로 확장하고 Producer 인터페이스를 도입하였다. RxJava의 구현은 매우 정교하며 고도로 최적화되어 있다.

RxJava의 세부 사항이 다른 Rx구현과 약간 다르긴 하지만, 여전히 특별히 분산 실시간 데이터 처리라는 새로운 영역에서 살아 남야하며 삶을 엿먹이는 돌발적 복잡성없이 필요 불가결한 복잡성에 집중하는 모든 개발자들을 위해 개발되었다. 이 책은 특히 RxJava의 개념과 사용에 대한 깊이있고 철저한 탐구이다. RxJava는 실세계에서 구현과 사용에 수천 시간의 경험을 가진 두 명의 저자가 전반적으로 사용한다. "reactive"하고 싶다면 이 책을 사는 것보다 더 좋은 방법은 없다.

**Erik Meijer, President and Founder, Applied Duality, Inc.**

# Introduction

## 누가 이 책을 읽어야 할까

Reactive Programming with RxJava는 중급 및 고급 Java 프로그래머를 대상으로 한다. 당신은 Java에 상당히 익숙해야 한다; 하지만 반응적 프로그래밍에 대한 사전 지식은 필요하지 않다. 이 책의 많은 개념들은 함수적 프로그래밍과 관련이 있지만 그것에 익숙할 필요 역시 없다. 이 책을 통해 이익을 얻을 수 있는 프로그래머 그룹은 두 가지이다:

* 서버에서 향상된 성능을 추구하거나 모바일 장치에서 더 유지 보수 가능한_maintainable_ 코드를 추구하는 장인. 이 범주에 속한다면 실용적인 조언뿐만 아니라 실제 문제에 대한 아이디어와 해결책을 찾을 수 있다. 이 경우 RxJava는 이 책이 마스터하는 데 도음이 되는 또 하나의 도구일 뿐이다.

* 반응형 프로그래밍이나 특히 RxJava에 대해 들어보았고 꽤 많은 이해를 원하는 호기심 많은 개발자. 당신이 이러하고, RxJava의 장점을 운영 코드에 적용할 계획이 아니라면 당신의 시야를 두드러지게 넓힐 수 있을 것이다.

또한 당신이 소프트웨어 아키텍트인 경우 이 책은 도움이 될 것이다. RxJava는 전체 시스템의 전반적 아키텍처에 영향을 미치므로 이를 알아둘 가치가 있다. 그러나 당신이 프로그래밍을 막 시작한 초보자라고 할지라도 기초를 설명하는 앞의 몇 챕터를 살펴보라. 변환과 구성같은 기본 개념은 매우 보편적이며 반응형 프로그래밍과 관련이 없다.

## Ben Christensen의 노트

2012년에 나는 Netflix API의 새로운 아키텍처를 작업하였다. 그 과정에서 동시성 및 비동기 네트워크 요청을 수용해야 한다는 것이 분명해졌다. 처리 방식을 탐구하는 동안 Jafar Husain을 만났고 그는 Microsoft에서 "Rx"라고 불렀던 접근 방식을 내게 납득시키려고 했다. 당시 나는 동시성에 익숙했지만 여전히 이를 (자바는 나의 지배적인 생계 부양자이고 그래서 여기에 대부분의 시간을 보냈으므로) 자바 중심적 방식인 절차적_imperatively_으로 생각하고 있었다.

그래서 Jafar가 이 접근 방식을 나에게 납득시키려할 때, 함수형 프로그래밍 스타일때문에 개념을 파악하기가 어려웠고 이를 우선순위의 뒤에 두었다. 몇 달간의 논쟁과 토론이 이어지면서 전체 시스템 아키텍처가 성숙해짐에 따라 Jafar와 나는 내가 이론적 원칙들과 이어서 Reactive Extensions이 제공할 수 있는 우아함과 힘을 파악할 때 까지 화이트 보드 세션을 계속 하였다.

우리는 Rx 프로그래밍 모델을 Netflix API에 받아들이기로 결정했고 궁극적으로는 Microsoft에서 Rx.Net와 RxJS로 시작된 작명 규칙에 따라 RxJava라고 불리는 Reactive Extensions의 Java 구현을 만들었다.

대략 3년 동안 내가 RxJava에 대해 작업하는 동안 그것의 대부분은 GitHub에 열려있다. 나는 RxJava를  운영 시스템의 서버와 클라이언트 사이드 모두에 사용되는 성숙한 제품으로 바꾸기 위해 120명 이상의 기여자들과 성장하는 커뮤니티와 함께 작업을 하는 특권을 가졌다. GitHub에서 15,000개 이상의 별을 얻기에 충분할 정도록 성공하였고 이는 RxJava를 200대 프로젝트 중 하나로 만들었고 이는 Java를 사용하는 프로젝트 중 3번째로 높은 것이다.

Netflix의 George Campbell, Aaron Tull 및 Matt Jacobs는 life, Subscriber, backpressure 그리고 JVM-polyglot 지원을 포함해 초기 빌드에서 RxJava가 될 때까지의 성숙에 필수적인 요소였다. Dávid Karnok가 프로젝트에 참여하였고 지금은 커밋과 코드 라인에서 나를 능가했다. 그는 프로젝트 성공의 중요한 요소였으며 지금은 프로젝트 리드를 인계받았다.

Microsoft에있는 동안 Rx를 만든 Erik Meijer에게 감사를 표해야 한다. 그가 회사를 떠난 후 Netflix에서 나는 그와 RxJava에 협업 할 수 있는 기회를 얻었으며 지금은 페이스 북에서 그와 직접 일할 수 있을 만큼 운이 좋다. 나는 그가 토론하고 배울 수 있었던 화이트 보드에서 많은 시간을 보낼 수 있었음이 정말 영광이라고 생각합니다. 자신의 생각을 한단계 올리기 위해 Erik과 같은 멘토를 만나는 것은 진정한 차이를 만듭니다.

그 과정에서 나는 RxJava와 반응형 프로그래밍에 대한 많은 컨퍼런스에서 연설을 해야했고 그 과정을 통해 코드와 아키텍처에 대해 나 혼자서 했던 것보다 훨씬 더 많은 것을 배울 수 있도록 나를 도와준 많은 사람들을 만났다.

Netflix는 이 프로젝트에 대한 나의 시간과 노력을 지원하고 나 스스로는 절대 작성할 수 없었을 기술 문서에 대한 지원을 제공하는 것은 경이로웠다. 이 오픈 소스의 성숙함과 범위는 당신의 "하루 일과" 동안의 작업과 다른 스킬 셋을 가진 많은 사람들의 참여가 없었다면 성공할 수 없다.

첫 번째 쳅터는 반응형 프로그래밍이 유용한 프로그래밍 접근법인 이유와 특히 RxJava가 어떻게 이러한 원칙들을 구체적 구현하였는지를 소개하려는 나의 시도이다.

이 책의 나머지 부분은 놀라운 작업을 수행한 Tomasz에 의해 쓰여졌다. 나는 검토와 제안의 기회를 가지긴 했지만 이것은 그의 책이며, 그는 쳅터 2 이후의 세부 사항을 가르쳐 줄 것이다.

## Tomasz Nurkiewicz의 노트
내가 금융 기관에서 일하던 2013년쯤에 RxJava를 우연히 알게 되었다. 우리는 마켓 데이터의 거대한 스트림을 리얼타임으로 처리하는 것을 다루고 있었다. 그때까지 데이터의 파이프라인은 메시지를 전달하는 Kafka, trade를 처리하는 Akka, 데이터를 변환하는 Clojure 그리고 시스템 전반에 변화를 전파하기 위한 커스텀 제작된 언어로 구성되어 있었다. RxJava는 다양한 데이터 소스에 대해 잘 작동하는 균일한_uniform_ API를 가지고 있었기 때문에 매우 매력적인 선택이였다.

시간이 지남에 따라 확장성_scalability_과 처리량_throughput_이 필수적인 많은 시나리오들에 반응형 프로그래밍을 시도하였다. 반응형 방식으로 시스템을 구현하는 것은 확실히 더 부담이 된다. 그러나 하드웨어 활용도를 높이고 따라서 에너지를 절감하는 등의 이점이 훨씬 더 중요하다. 이 프로그래밍 모델의 장점을 완전히 이해하려면 개발자는 비교적 사용하기 쉬운 도구가 있어야 한다. 나는 Reactive Extensions가 추상화, 복잡성 및 성능면에서 최적의 위치에 있다고 믿는다.

별도로 명시하지 않는 한 이 책은 RxJava 1.1.6을 다룹니다. RxJava는 Java 6 이상을 지원하지만 거의 모든 예제는 Java 8의 lambda 구문을 사용한다. Android(쳅터 8)를 다루는 쳅터의 일부 예제에서는 람다식보다 verbose한 문법을 다루는 방법을 보여준다. 즉, 가독성을 향상시키기 위한 가장 짧은 구문(메서드 참조와 같은)을 항상 사용하는 것은 아니다.

# Chapter 1. RxJava를 사용한 반응형 프로그래밍

RxJava는 Java 및 Android를 대상으로 한 함수형 프로그래밍에 영향을 받은 반응형 프로그래밍_reactive programming_의 구현이다. RxJava는 전역 상태_global state_와 부수 효과_side effects_를 방지하는 것과 비동기 및 이벤트 기반_event-based_ 프로그램을 조직하기 위해 스트림_stream_ 기반으로 사고하는 것, 그리고 함수의 구성_composition_을 선호한다. RxJava는 Producer/Consumer 콜백의 Observer pattern에서 시작해서 구성_composing_, 변환_transforming_, 스케줄링_scheduling_, 스로틀링_throttling_, 오류 처리 및 생명 주기_lifecycle_ 관리를 가능하게 하는 수십개의 연산자_operator_로 확장된다.
RxJava는 성숙한 오픈 소스 라이브러리이며 서버와 안드로이드 모바일 장치 모두에서 폭넓게 채택되고 있다. 이 라이브러리와 함께 RxJava와 반응형 프로그래밍를 중심으로 프로젝트에 기여하고, 말하고, 쓰고, 서로 도울 수 있는 개발자들의 활발한 [커뮤니티](http://reactivex.io/tutorials.html)가 만들어졌다. 

이 챕터는 RxJava가 무엇인지와 동작 방법에 대한 개요를 다룬다. 그리고 이 책의 나머지 부분은 당신의 응용 프로그램에서 어떻게 사용하고 적용할지에 대한 모든 세부 내용들을 알려 줄 것이다. 반응형 프로그래밍에 대한 사전 경험이 없어도 이 책을 읽을 수 있다. 기초에서부터 시작해 RxJava의 개념과 실례을 익히도록 하여 당신의 유스 케이스에 RxJava의 장점을 적용할 수 있게 할 것이다.

## 반응형 프로그래밍과 RxJava

반응형 프로그래밍은 데이터 값이나 이벤트들 같은 변화에 반응_reacting_하는 데 초점을 맞춘 일반적 프로그래밍 용어이다. 이렇게 하는 것은 가능하며 보통은 절차적_imperatively_ 방식으로 완성되었다. 콜백은 절차적 방식으로 반응형 프로그래밍을 완수하기 위한 접근법이다. 스프레드시트는 반응형 프로그래밍의 좋은 예이다: 셀은 다른 셀에 의존한다. 다른 셀들을 의존하는 셀들은 다른 셀의 변화에 자동적으로 “반응_react_”한다.

***
NOTE : FUNCTIONAL REACTIVE PROGRAMMING?

함수형 프로그래밍이 Reactive Extenstion(일반적으로 Rx, 그리고 특히 RxJava)에 영향을 주었음에도 불구하고 RxJava는 Functional Reactive Programming(FRP, 함수적 반응형 프로그래밍)이 아니다. FRP는 연속 시간_continuous time_을 포함하는 반응형 프로그래밍의 매우 특수한 유형이다. 반면 RxJava는 이산 시간 이벤트만을 다룬다. 몇 년 전에 이미 다른 것에 의해 그 두 단어의 자연스러운 조합이 사용되었음을 알게 전까지는, 초기에 나는 작명 함정에 빠져 RxJava를 “함수적 반응형”이라고 선전하였다. 결과적으로 “반응형 프로그래밍”보다 더 명확하게 RxJava를 커버하는 잘 받아들어자는 일반적인 용어는 없다. FRP은 일반적으로 여전히 RxJava와 유사 솔루션을 나타내기 위해 잘못 사용되고 있으며 인터넷 상에서 때때로 그 의미가 확대되어야 할 지(지난 수년간 비공식적으로 사용되었으므로) 아니면 연속 시간 구현에 중점을 두는 것을 엄격하게 유지할 지에 대한 토론이 계속된다. 이러한 혼란을 해결하기 위해 우리는 RxJava가 실제로 함수형 프로그래밍에 영향을 받았으며 절차적 프로그래밍과는 다른 프로그래밍 모델을 단호히 채택한 것에 중점을 둘 수 있다. 이 챕터에서 “반응형_reactive_”을 언급할 때는 RxJava가 사용하는 반응형+함수형(reactive+functional) 스타일을 이야기하는 것이다. “절차적_imperative_”를 언급할 때는 반응형 프로그래밍은 절차적_imperatively_으로 구현할 수 없다는 이야기를 하는 것이 아니다; RxJava가 사용하는 함수형_functinal_ 방식의 반대로서 절차적_imperative_ 프로그래밍의 사용을 이야기한다.
내가 명확하게 절차적 접근과 함수형 접근을 비교할 때는, 나는 “반응적-함수형_reactive-functional_”과 “반응적-절차적_reactive_imperative_”을 엄밀히 말하기 위해 사용하는 것이다.
***

오늘날의 컴퓨터는 어느 순간에는 절차적_imperative_이 된다. 절차형_imperative_이 운영체제와 하드웨어에 일치하기 때문이다. 컴퓨터에는 반드시 무엇이 완료되어야 하며 어떻게 수행해야 하는지를 명시적으로 이야기해주어야 한다. 사람은 CPU나 관련 시스템처럼 생각하지 않으므로 우리는 추상_abstraction_을 추가한다. 고수준_higher-level_ 절차적 프로그래밍의 언어_idioms_가 내제된 바이너리_binary_와 어셈블리 인스트럭션_assembly instruction_을 위한 추상인 것처럼 반응적-함수형 프로그래밍도 추상이다. 모든 것은 절차형으로 완료된다는 사실을 기억하고 이해하는 것은 중요하다. 그것은 반응적-함수형 프로그래밍이 무엇을 다루고 결국 어떻게 수행되는 지에 대한 정신적 모델로서 우리를 돕기 때문이다.
그래서 반응적-함수형 프로그래밍은 (특히 스레드와 네트워크 영역에 걸친) 복잡한 상태와의 상호 작용을 컴퓨터처럼 생각하거나 절차적으로 정의할 필요없이 비동기적이고 이벤트 기반_event-driven_ 유스케이스의 프로그램 작성을 할 수 있게 해주는 —절차적 시스템 상위의 추상인—프로그래밍 접근법이다. 컴퓨터처럼 생각하지 않아도 되는 것에는 동시성_concurrency_과 병행성_parallelism_이 수반되며 올바르고 효율적으로 사용하기가 매우 어려운 것이 특징인 비동기, 이벤트-기반 시스템에 유용한 특성이다.
자바 커뮤니티에서 Java Concurrency in Practice by Brian Goetz and Concurrent Programming in Java by Doug Lea (Addison-Wesley)같은 책과  “Mechanical Sympathy” 같은 포럼들은 동시성_concurrency_의 통달에 대한 깊고 넓고 복잡함을 대표한다. 내가 RxJava를 사용하기 시작한 이래 이 책들과 포럼, 그리고 커뮤니티들의 전문가들과의 상호작용은 고성능이며, 효율적이고, 확장 가능하며_scalable_ 정확한 동시 수행_concurrent_ 소프트웨어가 진정 얼마나 어려운지를 이전보다 훨씬 더 확신하게 했다. 심지어 우리는 동시성_concurrency_과 병행성_parallelism_의 차원이 다른 분산 시스템_distributed system_을 적용한 것도 아니다.
그래서 반응적-함수형 프로그래밍이 해결할 수 있는 것이 무엇인지에 대한 간결한 대답은 동시성_concurrency_와 병행성_parallelism_이다. 좀 더 구어체적으로 이것은 반응형, 비동기 유스케이스를 절자척 방식으로 처리한 결과인 콜백 지옥을 해결한다. RxJava로 구현된 것과 같은 반응형 프로그래밍은 함수형 프로그래밍에 영향을 받았으며 반응적-절차적 코드의 전형적 함정을 피하기 위해 선언적_declarative_ 접근법을 사용한다.

## 반응형 프로그래밍이 필요할 때

반응형 프로그래밍은 다음과 같은 시나리오들에서 유용하다:

 * 마우스의 움직임과 클릭, 키보드 입력, 사용자가 그들의 단말과 이동하는 동안의 시간 경과에 따른 GPS 신호의 변화, 단말 자이로스코프 신호들, 터치 이벤트들과 같은 사용자 이벤트를 처리하기.
 * IO은 본질적으로 비동기(요청이 만들어지고, 시간이 흐르고, 응답을 받거나 받지 않을 수 있고, 그것이 다음 작업을 촉발한다)임을 고려해볼때 일부 또는 모든 latency-bound IO 이벤트들에 반응하고 처리하기
 * 통제 할 수 없는 생산자에 의해 응용 프로그램으로 입력되는 데이터 또는 이벤트들의 처리(서버의 시스템 이벤트, 전술한 사용자 이벤트, 하드웨어의 신호들, 센서를 통해 아날로그 세상에서 유발되는 이벤트들 등등)

그런데 만약 문제가 되는 코드가 오직 하나의 이벤트 흐름_stream_만을 처리하고 있다면 콜백을 통한 반응적-절차적 프로그래밍으로도 괜찮을 것이며 반응적-함수형 프로그래밍의 이득은 많지 않을 것이다. 만약 당신이 서로 다른 이벤트 흐름_stream_을 가지고 있고 이들 모두가 서로 완전히 독립적이라면 절차적 프로그래밍은 문제가 없을 것이다. 이런 간단한 유스케이스에서 절차적 접근은 반응형 프로그래밍의 추상 계층을 제거하여 현재 운영 체제와 언어 그리고 컴파일러가 더 최적화된 상태로 유지되게 하므로 가장 효율적일 것이다.
만약 당신의 프로그램이 대부분의 프로그램과 같다면 당신은 이벤트(또는 함수나 네트워크 호출의 비동기 응답)들을 결합하고, 서로 간에 상호 작용하는 조건 논리_conditional logic_가 필요하고, 그 것들 모두에 대한 실패 시나리오와 자원 정리_cleanup_를 다루어야 할 것이다. 여기가 바로 반응적-절차적 접근의 복잡성이 극적으로 증가하기 시작하는 곳이며 반응적-함수형 프로그래밍이 빛을 발하기 시작하는 곳이다. 내가 받아들이게 된 과학적이진 않은 관점은 반응적-함수형 프로그래밍은 초기에 높은 학습 곡선과 진입 장벽을 가지지만 복잡성의 상한은 반응적-절차적 프로그래밍에 비해 훨씬 낮다는 것이다. 일반적으로 Reactive Extensions (Rx)와 특히 RxJava의 tagline은 “비동기, 이벤트-기반 프로그램들을 구성하기 위한 라이브러리.”에서 비롯된 것이다. RxJava는 함수형_functional_이며 데이터-흐름_data-flow_ 프로그래밍에 영향을 받은 반응형 프로그래밍 원칙들의 구체적 구현이다. “반응형”이 되는 것에는 여러 가지 접근법들이 있으며 RxJava는 그것들 중 하나이다. 이제 RxJava의 동작을 파보도록 하자.

## How RxJava Works

RxJava에서 중심이 되는 것은 데이터 또는 이벤트의 흐름을 나타내는 **Observable** 타입이다. 이것은 푸시(반응_reactive_)를 위한 것이지만 풀_pull_(대화식_interactive_)으로도 사용될 수 있다. 이것은 즉각적_eager_이기보단 지연적_lazy_이다. 이것은 동기 또는 비동기로 사용될 수 있다. 이것은 시간의 흐름 동안의 0, 1, 다수 또는 무한의 값들 또는 이벤트들을 상징한다.

여기에 많은 유행어와 세부 사항들이 있으므로 이를 분석해보자. 자세한 내용은 ["rx.Observable의 해부학"](https://github.com/eyeahs/RPWR/wiki/Chapter2---Reactive-Extensions#rxobservable-%ED%95%B4%EB%B6%80%ED%95%99)에서 확인할 수 있다.

### Push versus Pull

RxJava는 푸시를 지원하기 위해 반응적이 된다. 그래서 **Observable** 그리고 관련된 **Observer**의 타입 시그너처는 여기에 푸시되는 이벤트를 지원한다. 이것은 보통 결과적으로 비동기를 수반하며 이는 다음 섹션에서 다룰 것이다. **Observable** 타입은 비동기 시스템에서 흐름 제어_flow control_나 배압_backpressure_에 대한 접근 방식으로서 비동기 피드백 채널(종종 비동기-풀_async-pull_ 또는 반응적-풀_reactive-pull_로도 언급된다)도 역시 지원한다. 이 쳅터의 후반 섹션에서 흐름 제어를 다르고 이 메커니즘이 어떻게 들어맞는지를 다룰 것이다.

푸시를 통한 이벤트 수신을 지원하기 위해 **Observable**과 **Observer**의 쌍은 구독_subscription_을 통해 연결된다. **Observable**은 데이터의 스트림을 상징하며 **Observer**에 의해 구독될 수 있다([“Observer＜T＞를 사용하여 모든 알림을 포착하기”](https://github.com/eyeahs/RPWR/wiki/Chapter2---Reactive-Extensions#observert%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-%EB%AA%A8%EB%93%A0-%EC%95%8C%EB%A6%BC%EC%9D%84-%ED%8F%AC%EC%B0%A9%ED%95%98%EA%B8%B0)에서 더 많이 배울 것이다).

````java
interface Observable<T> {
    Subscription subscribe(Observer s)
}
````

구독을 하면 **Observer**는 세 가지 타입의 이벤트를 푸시할 수 있다.
* **onNext()** 함수를 통한 데이터
* **onError()** 함수를 통한 오류(exception 또는 throwable)
* **onCompleted()** 함수를 통한 스트림 완료_completion_

````java
interface Observer<T> {
    void onNext(T t)
    void onError(Throwable t)
    void onCompleted()
}
````

**onNext()** 메서드는 전혀 호출 되지 않거나 또는 단 한 번, 여러 번 또는 무한 번 호출 될 수 있다. **onError()** 또는 **onCompleted()**는 종료 이벤트이다. 둘 중 하나만 단 한 번 호출 될 수 있음을 의미한다. 종료 이벤트가 호출되면 **Observable** 스트림은 끝이 나고 더 이상의 이벤트는 전달되지 않는다. 스트림이 무한하고 실패하지 않는다면 종료 이벤트는 절대로 나타나지 않는다. "흐름 제어_Flow Control_"와 "배압_Backpressue_"에서 볼 수 있듯 대화식 풀_interactive pull_을 허용하는 시그너처의 추가 타입이 존재한다.

````java
interface Producer {
   void request(long n)
}
````

이것은 **Subscriber**이라고 불리는 더 발전된 **Observer**에서 사용된다(자세한 내용은 [Subscription와 Subscriber＜T＞를 통한 Listener 제어](https://github.com/eyeahs/RPWR/wiki/Chapter2---Reactive-Extensions#subscription%EC%99%80-subscribert%EB%A5%BC-%ED%86%B5%ED%95%9C-listener-%EC%A0%9C%EC%96%B4)에서 제공된다).

````java
interface Subscriber<T> implements Observer<T>, Subscription {
    void onNext(T t)
    void onError(Throwable t)
    void onCompleted()
    ...
    void unsubscribe()
    void setProducer(Producer p)
}
````

**Subscription** 인터페이스의 일부인 **unsubscribe** 함수는 구독자_subscriber_가 **Observable** 스트림을 구독 해지할 수 있게 해준다. **setProducer** 함수와 **Producer** 타입은 흐름 제어_flow control_에 사용되는 생산자_producer_와 소비자_consumer_간의 쌍방향 대화 채널을 형성하기 위해 사용된다.

### 비동기 vs 동기

일반적으로 **Observable**은 비동기적이겠지만 반드시 그러할 필요는 없다. **Observable**은 동기적으로 될 수 있으며 기본은 사실 동기적이다. RxJava는 요청전에는 동시성을 추가하지 않는다. 동기적 **Observable**은 구독자의 스레드를 사용하여 구독되고 모든 데이터를 발행하고 완료_completion_한다(유한하다면). 블로킹 네트워크 I/O가 배경이 되는 **Observable**은 구독 스레드를 동기적으로 블록한 뒤에 블로킹 네트워크 I/O가 반환을 하면 **onNext()**를 통해 발행할 것이다.

예를 들어 다음은 완전히 동기적이다.

````java
Observable.create(s -> {
    s.onNext("Hello World!");
    s.onCompleted();
}).subscribe(hello -> System.out.println(hello));
````

**Observable.create**은 “[Observable.create() 마스터하기](https://github.com/eyeahs/RPWR/wiki/Chapter2---Reactive-Extensions#observablecreate-%EB%A7%88%EC%8A%A4%ED%84%B0%ED%95%98%EA%B8%B0)”에서 그리고 **Observable.subscribe**은 “[Observable에서 알림을 구독하기](https://github.com/eyeahs/RPWR/wiki/Chapter2---Reactive-Extensions#observable%EC%97%90%EC%84%9C-%EC%95%8C%EB%A6%BC%EC%9D%84-%EA%B5%AC%EB%8F%85%ED%95%98%EA%B8%B0)”에서 더 자세히 배울 것이다.

당신의 생각이 맞다. 이것은 일반적으로 반응형 시스템에게 기대하는 동작이 아니다. **Observable**을 동기적 블로킹 I/O로 사용하는 것은 나쁜 형태이다(만약 블로킹 I/O가 사용되어야 한다면 스레드를 사용해 비동기로 만들어질 필요가 있다). 하지만 가끔 메모리 안의 캐시에서 동기적으로 데이터를 가져와서 즉시 반환하는 것은 적절하다. 이전 예제에서 보여준 "Hello World"의 경우에도 동시성은 필요 없다. 사실 비동기적 스케줄링이 추가되면 훨씬 느려질 것이다. 따라서 일반적으로 중요한 사실상의 기준은 **Observable**의 이벤트 생산이 동기적인지 비동기적인지가 아니라 블로킹인지 블로킹이 아닌지이다. "Hello World" 예제는 스레드를 블록하지 않으므로 블록킹이 아니며 따라서 **Observable**의 (불필요 할지라도) 올바른 사용이다.

RxJava의 **Observable**은 비동기인지 동기인지, 그리고 동시성이 존재하는지 또는 이것이 어디서 온 것인지에 대해 의도적으로 불가지_agnostic_하다.  이는 의도된 것이며 **Observable**의 구현이 무엇이 가장 최적인 지를 결정하게 해준다. 왜 이것이 유용할까?

무엇보다 동시성은 스레드풀만이 아니라 다양한 장소에서 올 수 있다. 만약 데이터 소스가 이벤트 루프에 있어서 이미 비동기적이라면 RxJava는 더 이상의 스케줄링 오버헤드를 추가하거나 특정 스케줄링 구현을 강제하지 않아야 한다. 동시성은 스레드풀이나 이벤트 루프, 액터_actor_ 등에서 올 수 있다. 이것은 데이터 소스에서 기원하거나 추가 될 수 있다. RxJava는 비동기가 어디서 기원했는지의 측면에서 불가지_agnostic_하다.

둘째로 동기적 행위가 사용되는 좋은 이유가 두 가지 있다. 다음 서브섹션에서 이를 살펴보자.

*In-memory data*

만약 데이터가 (일정한 마이크로초/나노초 검색 시간을 가진) 로컬 in-memory 캐시에 존재한다면 이것을 비동기로 만들기 위해 스케줄링 비용을 지불하는 것은 이치에 맞지 않다. **Observable**은 구독 스레드에서 데이터를 동기적으로 조회하고 발행 할 수 있다:

````java
Observable.create(s -> {
    s.onNext(cache.get(SOME_KEY));
    s.onCompleted();
}).subscribe(value -> System.out.println(value));
````

스케줄링 선택은 데이터가 메모리에 있을 수도 있고 없을 수도 있을 때 힘을 발휘한다. 만약 메모리에 있으면 동기적으로 발행하라; 그러지 않으면 비동기적으로 네트워크 호출을 하고 데이터가 도착하면 반환하라. 이 선택은 **Observable**내부에 조건적으로 존재할 수 있다:

````java
// pseudo-code
Observable.create(s -> {
    T fromCache = getFromCache(SOME_KEY);
    if(fromCache != null) {
        // 동기적으로 발행
        s.onNext(fromCache);
        s.onCompleted();
    } else {
        // 비동기적으로 조회
        getDataAsynchronously(SOME_KEY)
            .onResponse(v -> {
                putInCache(SOME_KEY, v);
                s.onNext(v);
                s.onCompleted();
            })
            .onFailure(exception -> {
                s.onError(exception);
            });
    }
}).subscribe(s -> System.out.println(s));
````

*동기 계산 (예, 연산자_operator_)*

동기적으로 남는 더 일반적 이유는 연산자_operator_를 통한 스트림 구성과 변환이다. RxJava는 데이터를 조작하고 결합, 변환할 때 사용하는 **map()**, **filter()**, **take()**, **flatMap()**과 **groupBy()**같은 연산자_operator_들의 광범위한 API를 대부분 사용한다. 대부분의 연산자_operator_는 동기적이다. 이는 이벤트가 지나쳐 갈 때 연산자는 **onNext()** 내부에서 동기적으로 계산을 수행함을 의미한다.

이 연산자_operator_들은 성능을 이유로 동기적이다. 다음을 예제로 보자:

````java
Observable<Integer> o = Observable.create(s -> {
    s.onNext(1);
    s.onNext(2);
    s.onNext(3);
    s.onCompleted();
});

o.map(i -> "Number " + i)
 .subscribe(s -> System.out.println(s));
````

만약 **map** 연산자_operator_가 기본적으로 비동기라면 각 숫자 (1,2,3)은 스트링 결합이 ("Number" + i)를 수행할 스레드에 예정될 거이다. 이것은 매우 비효율적이며 스케줄링과 컨텍스트 스위칭_context switching_ 등에 의해 일반적으로 결정되지 않은 지연을 가진다.

여기서 이해해야 할 중요한 점은 **Observable** 자체는 비동기일 수 있는 반면 거의 모든 **Observable** 함수 파이프라인_function pipeline_은 동기식이라는 것이다(**timeOut** 또는 **observbeOn**처럼 동기일 필요가 있는 특정 연산자_operator_는 제외). 이 주제는 “Declarative Concurrency with observeOn()”와 “Timing Out When Events Do Not Occur”에서 보다 더 심층적인 처리 방법을 제공한다.

다음 예제는 동기와 비동기의 혼합을 보여준다:

````java
Observable.create(s -> {
   ... 비동기 구독과 데이터 발행 ...
})
.doOnNext(i -> System.out.println(Thread.currentThread()))
.filter(i -> i % 2 == 0)
.map(i -> "Value " + i + " processed on " + Thread.currentThread())
.subscribe(s -> System.out.println("SOME VALUE =>" + s));
System.out.println("Will print BEFORE values are emitted")
````

위 예제의 **Observable**은 비동기이다(이것은 구독자와는 다른 스레드에 발행된다). 그래서 **subscribe**는 블로킹이 아니다. 그리고 이벤트가 전파되고 "SOME VALUE =>" 출력이 표시되기 전에 마지막의 **println**이 출력된다.

하지만 **filter()**와 **map()** 함수들은 이벤트를 발행하는 호출 스레드에서 동기적으로 실행된다. 이는 우리가 원하는 일반적인 행동이다: 이벤트를 효율적으로 동기 계산하는 비동기 파이프라인(**Observable**와 구성된 연산자_operator_들).

앞서 말한 바와 같이 **Observable** 타입은 동기적 구체 구현과 비동기적인 구체 구현을 둘 다 자체적으로 지원하며 이는 의도된 것이다.

### 동시성_Concurrency_ 및 병렬성_Parallelism_

개개의 **Observable** 스트림은 동시성_concurrency_이나 병렬성_parallelism_을 허용하지 않는다. 대신 비동기 **Observable**들의 구성을 통해 이것들을 달성한다. 

병렬성_parallelism_은 일반적으로 다른 CPU들이나 머신들에서 작업들이 동시_simultaneous_ 실행되는 것이다. 반면 동시성_concurrency_은 여러 작업들의 구성 또는 인터리빙_interleaving_이다. 만약 단일 CPU가 여러 작업 (스레드 등)을 가지면 "시분할_time slicing_"을 통해 동시에 실행하지만 병렬로 실행하지는 않는다. 각 스레드는 작업의 완료 여부와 관계없이 다른 스레드에게 양보할 수 있으며 그 전까지는 CPU 시간의 일부를 가진다.

병렬 실행_Parallel execution_은 정의상 동시적_concurrent_이지만, 동시성_concurrency_이 반드시 병렬성인 것은 않는다. 멀티스레드로 동작한다는 것은 실제로는 동시성_concurrency_을 의미한다. 하지만 병렬성_parallelism_은 스레드들이 정확히 동일한 시간에 다른 CPU들에서 스케줄링되고 실행될 때만 존재한다. 그러므로 일반적으로 우리는 동시성_concurrency_과 동시 발생_concurrent_됨에 대해 이야기한다. 하지만 병렬성_parallelism_은 동시성_concurrency_의 특정한 형태이다.

RxJava **Observable**의 규약은 이벤트들(**onNext()**, **onCompleted()**, **onError()**)은 절대로 동시에 발행 될 수 없음이다. 다른 말로 단일 **Observable** 스트림은 언제나 직렬화_serialized_되어야 하며 스레드-세이프해야 한다. 각각의 이벤트는 동시에 발행되지만 않는다면 서로 다른 스레드들에서도 발행될 수 있다. 이것은 onNext()은 인터리빙_interleaving_ 또는 동시_simultaneous_ 실행이 되지 않아야 한다는 것을 의미한다. 만약 **onNext()**가 한 스레드에서 계속 실행되고 있으면 다른 스레드는 호출을 다시 시작할 수 없다(인터리빙_interleaving_).

이 예제는 문제가 없다:

````java
Observable.create(s -> {
  new Thread(() -> {
    s.onNext("one");
    s.onNext("two");
    s.onNext("three");
    s.onNext("four");
    s.onCompleted();
  }).start();
});
````

이 코드는 데이터를 순차적으로_sequentially_ 발행하므로 규약을 충족한다(하지만  이렇게 스레드를 **Observable**내부에서 시작하지 않을 것을 일반적으로 권고한다. 대신 “Multithreading in RxJava”에서 설명되는 스케줄러_schedulers_를 사용하라.)

이 코드는 위반 예제이다:

````java
// DO NOT DO THIS
Observable.create(s -> {
  // Thread A
  new Thread(() -> {
    s.onNext("one");
    s.onNext("two");
  }).start();

  // Thread B
  new Thread(() -> {
    s.onNext("three");
    s.onNext("four");
  }).start();

  // ignoring need to emit s.onCompleted() due to race of threads
});
// DO NOT DO THIS
````

이 코드는 두 스레드 모두 동시에 **onNext()**를 호출할 수 있으므로 위반이다. 이것은 규약을 위반한다(또한, **onComplete** 호출을 위해 두 스레드가 완료_complete_ 될 때까지 안전하게 기다려야하며 앞에서 말했 듯이 일반적으로 스레드를 수동으로 시작하는 것은 좋지 않다).

그러면 어떻게 해야 RxJava의 동시성과 (또는) 병렬성을 활용할 수 있을까? 구성_Composition_.

단일 **Observable** 스트림은 언제나 직렬화_serialized_되어 있다. 하지만 각 **Observable**의 스트림은 다른 것들과 독립적으로 동작할 수 있으므로 이는 동시적_concurrently_ 그리고/또는 병렬적_parallel_ 상태이다. 이는 **merge**와 **flatMap**이 RxJava에서 널리 사용되는 이유이다 --비동기 스트림들을 동시에 함께 구성함. (**merge**와 **flatMap**에 대해 더 자세한 내용은 “Wrapping Up Using flatMap()”과 “Treating Several Observables as One Using merge()”에서 배울 수 있다.)

여기에 두 비동기 **Observable**들이 서로 별개의 스레드에서 수행되고 서로 합쳐지는 메커니즘을 보여주기 위해 만들어진 예제가 있다:

````java
Observable<String> a = Observable.create(s -> {
  new Thread(() -> {
    s.onNext("one");
    s.onNext("two");
    s.onCompleted();
  }).start();
});

Observable<String> b = Observable.create(s -> {
  new Thread(() -> {
    s.onNext("three");
    s.onNext("four");
    s.onCompleted();
  }).start();
});

// a와 b를 동시에 구독하고 순차적인 세번째 스트림으로 합친다.
Observable<String> c = Observable.merge(a, b);
````

**Observable c**는 a와 b에서 항목들을 받으며 그것들의 비동기성으로 인해 세가지 일이 일어난다:
 * “one”은 “two” 이전에 나타난다.
 * “three”은 “four” 이전에 나타난다.
 * one/two와 three/four 사이의 순서는 불특정하다.

그렇다면 왜 **onNext()**가 동시에 호출되는 것이 허용되지 않는 것일까?

주된 이유는 **onNext()**는 사람이 이용하는 것을 염두에 두었으며, 동시 실행은 어렵기 때문이다. 만약 **onNext()**가 동시에 호출될 수 있다면 모든 **Observable**은 동시 호출 때문에 (이를 기대하지 않았거나 원하지 않았을 때에도) 방어적으로 코드가 작성될 필요를 의미한다.

두 번째 이유는 어떤 연산자_operator_는 동시에 발행하는 것이 불가능하기 때문이다; 예를 들어 **scan**과 **reduce**는 일반적이고 중요한 동작이다. **scan**과 **reduce**같은 연산자_operator_는 순차적_sequential_ 이벤트의 전파를 요구한다. 그래야지 결합적_associative_이지도 가환적_commutative_이지도 않은 이벤트의 스트림의 상태가 누적 될 수 있다. 동시적(**onNext()**의 동시 발생) **Observable** 스트림의 허용은 이벤트의 타입을 처리 가능한 것으로 한정하고 스레드-세이프한 데이터 구조를 요구할 것이다.

***
**NOTE**
Java8의 **Stream** 타입은 동시 발행을 지원한다. 이것이 **java.util.stream.Stream**의 [**reduce**함수가 반드시 결합적_associative_이여야 하는 이유이다](http://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#reduce-java.util.function.BinaryOperator-). 병렬 스트림의 동시 호출을 반드시 지원해야 하기 때문이다. 병렬성, (가환성_commutative_과 관련있는) 순서, reduction 기능 그리고 결합성_associativity_에 대한 [**java.util.stream** 패키지의 문서](http://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html)는 순차적_sequential_ 발행과 동시 발행을 모두 지원하는 동일한 **Stream** 타입의 복잡함을 보여준다.
***

세번째 이유는 동기화 오버헤드가 성능에 영향 주기 때문이다. 데이터가 거의 항상 순차적_sequentially_으로 도착할지라도 모든 observer들과 operator들은 스레드 세이프 해야 하기 때문이다. 보통 JVM은 동기화의 오버헤드를 잘 제거해 주지만 항상 가능한 것은 아니므로 (특히 원자성을 사용하는 논블록 알고리즘에서 그렇다.) 순차적 스트림에는 불필요한 성능 부담이 있게 된다.

추가로, 일반적으로 범용으로 세분화된_fine-grained_ 병렬 처리를 수행하는 것은 더 느리다. 병렬 처리는 보통 스레드 전환, 작업 스케줄링, 재조합 같은 오버 헤드를 보상하기 위해 일괄 처리 작업처럼 굵직하게_coarsely_ 수행 될 필요가 있다. 순차적 계산을 단일 스레드에서 동기적으로 실행하고 많은 메모리 및 CPU 최적화를 이용하는 것이 훨씬 더 효율적이다. **List**나 **array**은 모든 항목들이 미리 알려져있고 이를 일괄 처리 작업들로 나눌 수 있기 때문에 일괄 처리 작업으로 만들어진 병렬성을 위한 적절한 기본값을 가지는 것은 꽤 간단하다. (전체 목록을 단일 CPU에서만 처리하는 것이 보통 더 빠르지만 이는 리스트가 매우 크지 않고 항목당 계산이 오래 걸리지 않을 때이다). 하지만 스트림은 작업을 미리 알지 못하며 단지 **onNext()**를 통해 데이터를 전달 받으므로 자동적으로 작업을 청크_chunk_ 처리할 수 엇다.

사실 RxJava v1 이전에 **.parallel(Function f)** 연산자_operator_가 **java.util.stream.Stream.parallel()**와 비슷한 동작을 시도하기 위해 추가되었었다. 그것이 좋은 유용품으로 간주되었기 때문이다. 이것은 단일 **Observable**을 많은 **Observable**로 나누어 각각 병렬로 실행되고 그 후 모두 다 합치는 방식으로 RxJava의 규약을 위배하지 않게 하였다. 하지만 이것은 결국 v1 이전에 라이브러리에서 [제거되었다](https://github.com/ReactiveX/RxJava/blob/e8041725306b20231fcc1590b2049ddcb9a38920/CHANGES.md#removed-observableparallel). 매우 혼란스러웠고 거의 항상 나쁜 성능의 결과가 나왔기 때문이다. 계산에 관련된 병렬성을 이벤트 스트림에 추가하는 것은 거의 언제나 설명될 수 있고 테스트 되어야 한다. **ParallelObservable**는 연산자가 결합성_associativity_을 가정하는 하위 집합으로만 제한되는 경우에는 이치에 맞을 수도 있다. RxJava가 사용되는 몇 년 동안 이것은 노력할 가치가 있었던 적이 없다. **merge**와 **flatMap**의 구성이 이 유스 케이스 해결을 위한 효율적인 빌딩 블록이기 때문이다.

챕터 3는 동시성과 병렬성의 장점을 얻기 위해 연산자_operator_들을 사용해 **Observable**을 구성하는 방법을 알려 줄 것이다.


### 지연적_Lazy_ vs 즉각적_Eager_

**Observable**의 타입이 지연적_lazy_이라는 것은 구독되기 전에는 아무 것도 수행하지 않는다는 것을 의미한다. 이는 **Future**처럼 생성됨이 작업 시작을 나타내는 즉각적_eager_ 타입과는 다르다. 지연성_Lazyiness_은 캐시 없이도 경쟁 조건에 의한 데이터 손실이 없이 **Observable**들을 구성을 할 수 있게 해준다. **Future**는 단일 값이 캐쉬 될 수 있으므로 이는 걱정 거리가 아니다. 따라서 값이 구성 전에 전달이 되어도 값을 조회할 수 있다. 무제한 스트림은 무제한 버퍼에게 이와 동일한 보증을 제공할 것을 요구 할 수 있다. 그래서 **Observable**은 데이터가 흐르기를 시작하기 전에 모든 구성이 완료 될 수 있도록 지연적_lazy_이고 구독되기 전에는 시작하지 않을 것이다.

실제로 이는 두 가지를 의미한다:

**구조를 만드는 것이 아니라 구독_Subscription_이 작업을 시작하게 한다**
**Observable**의 지연성_lazyiness_으로 인해 생성은 (**Observable**객체의 할당하는 "작업"은 무시하고) 작업 발생의 실제 원인이 되지 않는다. 생성은 구독이 되었을 때 어떤 작업이 수행되어야 하는지를 정의 하는 것일 뿐이다. 다음과 같이 정의된 **Observable**을 고려해보자:

````java
Observable<T> someData = Observable.create(s -> {
    getDataFromServerWithCallback(args, data -> {
        s.onNext(data);
        s.onCompleted();
    });
})
````

**someData** 참조는 지금 존재한다. 하지만 **getDataFromServerWithCallback**은 아직 실행되지 않았다. **Observable** 래퍼가 수행되어야 할 한 단위의 작업(**Observable**내부에 존재하는 함수)을 명시하였다는 것만이 유일하게 발생한 일이다. 

이 **Observable**을 구독하는 것은 이 작업이 완료되게 한다:

````java
someData.subscribe(s -> System.out.println(s));
````

이것은 **Observable**이 의미하는 작업을 지연적으로 수행한다.

**Observable은 재활용 될 수 있다**
**Observable**이 지연적_lazy_이라는 것은 특정 인스턴스가 한 번 이상 호출 될 수 있음도 역시 의미한다. 이전 예제에서 다음과 같은 것을 계속 할 수 있다:

````java
someData.subscribe(s -> System.out.println("Subscriber 1: " + s));
someData.subscribe(s -> System.out.println("Subscriber 2: " + s));
````

이제 여기에 두 가지 서로 다른 구독이 있으며 각각은 **getDataFromServerWithCallback**을 호출하고 이벤트를 발행한다.

이 지연성은 **Future**같은 비동기 타입과는 다르다. **Future**의 생성은 작업이 이미 시작되었음을 의미한다. **Future**는 (여러번 작업을 트리거하기 위한 구독으로) 재사용될 수 없다. **Future**의 참조가 존재한다는 것은 작업이 이미 발생하였음을 의미한다. 앞선 예제 코드에서 즉각적_eagerness_인 부분이 정확히 어디에 존재하는지를 볼 수 있을 것이다. **getDataFromServerWithCallback** 메소드는 호출 즉시 실행되므로 즉각적_eager_이다. **getDataFromServerWithCallback**를 **Observable**로 래핑하는 것은 이것이 지연적_lazy_으로 사용될 수 있게 해준다.

이 지연성_laziness_은 구성을 할 때 강력하다. 예를 들어:

````java
someData
    .onErrorResumeNext(lazyFallback)
    .subscribe(s -> System.out.println(s));
````

이 경우 **lazyFallback Observable**은 완료 될 수 있는 작업을 의미하지만 이는 오직 누군가가 구독을 해 줄 때 만이다. 그리고 우리는 이것이 **someData**이 실패할 때만 구독되기를 기대한다. 물론 즉각적_eager_인 타입은 (**getDataAsFutureA()** 같은) 함수 호출을 이용해 지연적으로 만들어 질 수 있다.

즉각성_Eagerness_과 지연적_laziness_ 각각은 자신들만의 강점과 약점을 가지며 RxJava **Observable**은 지연적_lazy_이다. 그 결과 당신이 **Observable**을 가지고 있다면 당신이 구독하기 전에는 이것은 아무 것도 하지 않을 것이다.

이 주제는 “Embracing Laziness”에서 더 자세히 논의될 것이다.

### 쌍대성_Duality_

Rx **Observable**은 **Iterable**의 비동기 "쌍_dual_"이다. "쌍_dual_"이라는 것은 **Observable**이 **Iterable**의 (데이터의 역전된 흐름을 제외하고) 모든 기능들을 제공함을 의미한다: 이것은 풀_pull_이 아니라 푸시_push_(데이터의 역전된 흐름을 제외하고)이다. 다음의 표는 푸시와 풀 기능이 모두 제공하는 유형을 보여준다:

| Pull (Iterable)   | Push (Observable) |
| ----------------- | ------------------- |
| T next() | onNext(T) |
| throws Exception | onError(Throwable) |
| returns | onCompleted() |

위 표에 따르면 데이터를 소비자_consumer_가 **next()**를 통해 가져오는 대신 생산자가 **onNext(T)**에 푸시한다. 성공한 종료는 모든 항목들이 반복될 때까지 스레드를 차단하는 것이 아니라 **onCompleted()** 콜백을 통해 알려진다. 오류는 호출 스택에 예외로 던져지는 대신 **onError(Throwable)** 콜백에 이벤트로서 발행된다.

이것이 사실상 쌍으로서 동작한다는 사실은 **Iterable**와 **Iterator**로 pull을 통해 동기적으로 할 수 있는 모든 작업을 **Observable**와 **Observer**로 push를 통해 비동기적으로 할 수 있다는 것을 의미한다. 이는 동일한 프로그래밍 모델이 둘 모두에 적용 될 수 있음을 의미한다! 

예를 들어 Java 8의 경우  다음과 같이 동작하게 할 수 있도록 **java.util.stream.Stream** 타입을 통해 **Iterable**이 함수 구성이 되도록 업그레이드 할 수 있다:

````java
// 75개의 스트링을 가진 Iterable<String>로서의 Stream<String>
getDataFromLocalMemorySynchronously()
    .skip(10)
    .limit(5)
    .map(s -> s + "_transformed")
    .forEach(System.out::println)
````

이렇게 하면 **getDataFromLocalMemorySynchronously()**에서 75개의 스트링을 가져오고, 11-15 항목은 가지고 나머지는 무시하고, 스트링으로 변환하고, 이들을 출력한다. (**take**, **skip** 그리고 **limit**같은 연산자_operator_에 대해 더 자세한 것은 “Slicing and Dicing Using skip(), takeWhile(), and Others”에서 배운다.)

RxJava의 **Observable**는 동일한 방식으로 사용된다:

````java
// 75개의 스트링을 발행하는 Observable<String>
getDataFromNetworkAsynchronously()
    .skip(10)
    .take(5)
    .map(s -> s + "_transformed")
    .subscribe(System.out::println)
````

이것은 스트링 다섯 개를 받을 것이다(15개가 발행되었지만 처음의 10개는 버려졌다). 그 뒤 구독해지 된다(나머지 문자열들은 발행을 멈추거나 무시된다). 앞의 **Iterable/Stream** 예제처럼 문자열로 변환되고 출력된다.

즉 Rx **Observable**은 마치 **Iterable**와 **List**를 둘러싼 동기적 풀_pull_을 사용하는 **Stream**처럼 푸시_push_에 의한 비동기 데이터로 프로그래밍할 수 있게 해준다.

### Cardinality

**Observable** 타입은 다수의 값들의 비동기 푸시를 지원한다. 이는 다음 표의 오른쪽 아래, **Iterable**(또는 **Stream**, **List**, **Enumerable** 등)의 비동기 쌍과 **Future**의 다치형_multivalued_ 버전에 잘 맞는다:

|              | One                 | Many                    |
| ------------ | ------------------- | ----------------------- |
| Synchronous  | T getData()         | Iterable<T> getData()   |
| Asynchronous | Future<T> getData() | Observable<T> getData() |

이 섹션이 일반적으로 **Future**를 언급하고 있음을 주목하라. **Future**는 자신의 반응을 표현하기 위해 **Future.onSuccess(callback)** 구문을 사용한다. [CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html), [ListenableFuture](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListenableFuture.html) 또는 **Scala**의 [Future](http://docs.scala-lang.org/overviews/core/futures.html)같은 다른 구현들이 존재한다. 당신이 무엇을 하든지 
**java.util.Future**는 사용하지 마라. 이것은 값을 얻기 위해 블로킹을 요구한다.

그렇다면 **Observable**은 왜 **Future**을 대신하는 가치있는 존재일까? 가장 명확한 이유는 당신이 이벤트 스트림이나 다치형_multivalued_ 응답을 처리하고 있기 때문이다. 덜 명확한 이유는 여러 단일 값 응답의 구성 때문이다. 이들 각각을 살펴 보도록 하자.

- **Event 스트림**

이벤트 스트림은 복잡하지 않다. 여기서 설명 한 것처럼 시간의 흐름 동안 생산자는 소비자에게 이벤트를 푸시한다.

````java
// producer
Observable<Event> mouseEvents = ...;

// consumer
mouseEvents.subscribe(e -> doSomethingWithEvent(e));
````

이것은 **Future**에서는 제대로 동작하지 않는다:

````java
// producer
Future<Event> mouseEvents = ...;

// consumer
mouseEvents.onSuccess(e -> doSomethingWithEvent(e));”
````

**onSuccess** 콜백은 "마지막 이벤트"를 수신할 수 있다. 하지만 몇 가지 질문들이 남는다: 이제 소비자가 폴링을 해야 할 필요가 있는가? 생산자는 그것들을 대기열에 넣을 수 있는가? 아니면 각 패치들 사이에 잃어 버리는가? **Observable**은 확실히 여기에 유리하다. **Observable**이 없으면 **Future**로 이것을 모델링하는 것보다 콜백 접근법이 더 나을 것이다.

- **다중 값**

**Observable**의 다음 사용처은 다치형_multivalued_ 응답이다. 기본적으로 **List**, **Iterable** 또는 **Stream**이 사용되는 곳은 어디든지 **Observable**을 대신 사용할 수 있다.

````java
// producer
Observable<Friend> friends = ...

// consumer
friends.subscribe(friend -> sayHello(friend));
````

이것은 **Future**로 작업할 수 있다:

````java
// producer
Future<List<Friend>> friends = ...

// consumer
friends.onSuccess(listOfFriends -> {
   listOfFriends.forEach(friend -> sayHello(friend));
});
````

그렇다면 왜 **Observable<Friend>** 접근법을 사용하는가? 반환 할 데이터 리스트가 작다면 성능은 문제가 되지 않으며 주관적 선택 사항일 것이다. 만약 리스트가 크거나 또는 원격 데이터 소스가 목록의 다른 부분들을 다른 위치에서 조회해야 하는 경우 **Observable<Friend>** 접근을 사용하는 것은 성능 또는 지연 시간에 이점이 될 수 있다.

가장 설득력있는 이유는 전체 컬랙션이 도착하는 것을 기다리는 대신 항목이 받아지는 대로 처리 될 수 있기 때문이다. 백엔드에 다른 네트워크 지연들이 각각의 항목들에 다르게 영향을 줄 수 있을 때 특히 일치한다. 이는 (서비스-지향 또는 마이크로-서비스 아키텍처처럼) 롱-테일_long-tail_ 지연 및 공유 데이터 저장소로 인해 실제로 상당히 흔하다. 전체 컬렉션을 기다린다면 소비자는 언제나 컬렉션에 수행된 작업 집합의 최대 지연을 경험할 것이다. 항목들이 **Observable** 스트림으로 반환되면 소비자는 즉시 받으며 "첫 항목의 시간"은 마지막과 가장 느린 항목에 비해 현저히 낮을 수 있다. 이런 작업을 만들기 위해서 스트림의 순서는 희생되므로 서버에서 받은 순서로 항목이 발행될 수 있다. 순서가 궁극적으로는 소비자에게 중요하다면 랭킹이나 위치는 항목 데이터나 메타 데이터에 포함될 수 있으며, 필요에 따라 클라이언트가 항목을 정렬하거나 배치 할 수 있다.

또한, 전체 컬렉션을 메모리에 할당하고 수집하는 대신 메모리 사용량을 한 항목이 필요한 것으로 한정되도록 한다.

- **구성_Composition_**

다치형_multivalued_ **Observable** 타입은 **Future**에서 처럼 단일 값을 가진 응답을 작성할 때에도 유용한다.

여러 **Future**들을 병합할 때 다음처럼 단일 값을 가진 다른 **Future**를 발행한다:

````java
CompletableFuture<String> f1 = getDataAsFuture(1);
CompletableFuture<String> f2 = getDataAsFuture(2);

CompletableFuture<String> f3 = f1.thenCombine(f2, (x, y) -> {
  return x+y;
});
````

이것은 정확히 원하던 것이겠지만 사실 RxJava에서는 **Observable.zip**을 통해 가능하다.(“Pairwise Composing Using zip() and zipWith()”에서 더 자세히 알아 볼 것이다)

````java
Observable<String> o1 = getDataAsObservable(1);
Observable<String> o2 = getDataAsObservable(2);

Observable<String> o3 = Observable.zip(o1, o2, (x, y) -> {
  return x+y;
});
````

하지만 이는 어떤 것을 발행하기 전에 모든 **Future**들이 완료됨을 기다려야 함을 의미한다. 보통 **Future**의 완료로 반환된 값을 각각 발행하는 것이 더 나을 수도 있다. 이 경우 **Observable.merge** (또는 관련된 **flatMap**)이 더 좋을 것이다. 이것은 준비되자 마자 즉시 발행된 값들의 스트림으로 (각각이 하나의 값만 발행하는 **Observable**일지라도) 결과들을 구성할 수 있게 해준다.

````java
Observable<String> o1 = getDataAsObservable(1);
Observable<String> o2 = getDataAsObservable(2);

// 이제 o3는 대기 없이 각 항목을 발행하는 o1과 o2의 스트림이다.
Observable<String> o3 = Observable.merge(o1, o2);
````

- **Single**

Rx **Observable**이 다치형_multivalued_ 스트림을 처리하는 것에 뛰어난 반면에, 단일값 개념의 단순함은 API 디자인과 소비에 매우 좋다. 추가로 기본적인 요청/응답 동작은 애플리케이션에서 극도로 흔하다. 이런 이유로 RxJavas는 **Single** 타입을 제공한다. 이는 **Future**의 지연적_lazy_ 동치이다. 이를 두 가지 이점을 가진  **Future**으로 생각하라; 먼저 이것은 지연적이므로 여러번 구독될 수 있고 쉽게 구성할 수 있다. 두번째로 RxJava의 API와 잘 어울리므로 이것은 **Observable**과 쉽게 상호 작용할 수 있다.

예를 들어 다음 접근자를 고려하라:

````java
public static Single<String> getDataA() {
    return Single.<String> create(o -> {
      o.onSuccess("DataA");
    }).subscribeOn(Schedulers.io());
}

public static Single<String> getDataB() {
    return Single.just("DataB")
            .subscribeOn(Schedulers.io());
}
````

이것들은 이렇게 사용되고 선택적으로 구성될 수 있다:

````java
// merge a & b into an Observable stream of 2 values
Observable<String> a_merge_b = getDataA().mergeWith(getDataB());
````

두 **Single**들이 어떻게 **Observable**로 결합되는지를 보라. 이는 무엇이 먼저 완료되느냐에 따라 [A, B] 또는 [B, A]의 발행이 될 수 있다.

이전 예제로 돌아가서 우리는 이제 데이터 조회를 **Observable**대신 **Single**을 사용하여 나타낼 수 있으며 그것들을 값들의 스트림으로 결합할 수 있다.

````java
// Observable<String> o1 = getDataAsObservable(1);
// Observable<String> o2 = getDataAsObservable(2);

Single<String> s1 = getDataAsSingle(1);
Single<String> s2 = getDataAsSingle(2);

// o3 is now a stream of s1 and s2 that emits each item without waiting
// o3는 이제 대기 없이 항목을 발행하는 s1과 s2의 스트림이다.
Observable<String> o3 = Single.merge(s1, s2);
````

"stream of one"을 나타내기 위해 **Observable** 대신 **Single**을 사용하는 것은 소비를 단순화 한다. 개발자는 **Single** 타입의 다음 행동들 만을 고려하면 되기 때문이다.

* 에러로 응답
* 응답이 없음
* 성공으로 응답

이것을 **Observable**에서 고려해야 하는 추가적 상태들과 비교해보라:

* 에러로 응답
* 응답이 없음
* 데이터가 없는 성공적 응답과 종료
* 단일 값을 가지는 성공적 응답과 종료
* 다수의 값을 가지는 성공적 응답과 종료
* 하나 또는 더 많은 값들을 가지는 성공적 응답과 절대 종료되지 않음(다음 데이터를 기다림)

**Single**를 사용하면 정신 모형은 API를 소비하는데 더욱 간단해진다. 그리고 오직 **Observable**로 구성한 뒤에만 추가 상태를 고려할 필요가 있다. 데이터 API는 보통 서드 파티에서 오기는 하지만 일반적으로 개발자가 코드를 통제하므로 이는 보통 그것이 발생하기에 더 좋은 장소이다. 

"Observable versus Single**에서 **Single**에 대해 더 배우게 될 것이다.

- **Completable**

**Single**에서 더해서 RxJava는 반환 타입을 가지고 있지 않고 단지 성공 또는 실패 완료만을 나타낼 필요가 있는 놀랍게도 흔한 유스 케이스를 처리하는 **Completable** 타입 또한 가지고 있다. 보통 **Observable<Void>** 또는 **Single<Void>**가 결국 사용된다. 이는 어색하므로 여기서 보여지는 것처럼 **Completable**이 사용된다:

````java
Completable c = writeToDatabase("data");
````

이 유스 케이스는 반환 값이 기대되지 않지만 성공 또는 실패 완료 알림이 필요한 비동기 쓰기 작업을 할 때 흔하다. **Completable**를 사용한 앞의 코드는 이와 비슷하다:

````java
Observable<Void> c = writeToDatabase("data");
````

**Completable** 자신은 다음 처럼 완료와 실패, 두 가지 콜백의 추상이다:

````java
static Completable writeToDatabase(Object data) {
  return Completable.create(s -> {
    doAsyncWrite(data,
        // callback for successful completion
        () -> s.onCompleted(),
        // callback for failure with Throwable
        error -> s.onError(error));
  });
}
````

- **0에서 무한**

**Observable**은 0에서 무한대까지의 원소 개수_cardinality_지원한다(["무한 스트림"](https://github.com/eyeahs/RPWR/wiki/Chapter2---Reactive-Extensions#%EB%AC%B4%ED%95%9C-%EC%8A%A4%ED%8A%B8%EB%A6%BC)에서 더 알아본다). 그러나 단순하고 명확하게 하기 위해 **Single**은 "Observable of One"이며 **Completable**은 "Observable of None"이다.

새로 도입 된 타입을 사용하면 표가 다음처럼 된다:

|              | Zero                      | One                 | Many                    |
| ------------ | ------------------------- | ------------------- | ----------------------- |
| Synchronous  | void doSomething()        | T getData()         | Iterable<T> getData()   |
| Asynchronous | Completable doSomething() | Future<T> getData() | Observable<T> getData() |

## Mechanical Sympathy: Blocking versus Nonblocking I/O
....

## Reactive Abstraction

RxJava의 타입들과 연산자_operator_들은 궁극적으로 절차적_imperative_ 콜백 위에 있는 추상이다. 하지만 이 추상은 코딩 스타일을 완전히 변화시키며 비동기와 논블로킹 프로그래밍을 위해 가장 강력한 툴을 제공한다. 학습에 노력이 필요하며 스트림으로 사고하고 함수 구성에 익숙해지는 생각의 전환이 필요하지만 이를 달성하면 일반적인 일반적인 객체 지향 또는 절차적 프로그래밍 스타일과 함께 매우 효과적인 도구가 된다. 

이 책의 나머지는 RxJava의 작동 방식과 사용 방법에 대한 많은 세부 사항들을 설명한다. 챕터2에서는 **Observable**이 어디서 기원하는 지와 어떻게 사용하는 지를 설명한다. 챕터3은 수십 가지의 선언적이고 강력한 변형들을 안내한다.
