---
layout: post
category: blog
title: RxJava - takeUntil 실전 예제
date: '2016-09-27 18:28:00 +0900'
categories: RxJava
tags:
  - rxjava
published: true
---

[원본](https://medium.com/@vanniktech/rxjava-practial-takeuntil-example-bc9766918cad#.bat6wchsk)

지난 주 나는 다음 문제를 만났다:

- 동일한 타입의 서로 다른 모델 객체들을 백엔드로 전송해야 한다.
- 모든 모델들을 한 번에 전달 할 수 있는 백엔드 API가 없다
- 일단 백엔드가 성공 응답을 반환하면 남은 모델 객체들의 전송을 중단해야 한다.

모델 객체의 원천(source)은 이미 반응적이며 하나의 모델을 백엔드로 전달하는 방식도 역시 이미 반응적이기 때문에 반응적인 접근을 유지하기 하였다.

나는 여기에 가장 적절한 operator가 무엇인지 생각하였고 [takeUntil](http://reactivex.io/documentation/operators/takeuntil.html)을 기억해냈다.

문서에서:

> 원천 Observable에서 발행된 항목들을 발행하는 Observable을 반환하고, 각 항목마다 특정 술어(predicate)를 확인한 뒤, 조건이 만족된다면 완료한다.

꽤 듣기 좋다. 그런다면 이것들을 어떻게 연결시켜야 할까?

먼저 우리가 전송하고자 하는 모델을 얻자.

```java
modelProvider.getItems()
```

이것이 우리의 원천 Obseravable이다. 다음에 할 일은 각 객체마다 백엔드 요청을 하는 것이다. 여기에는 [FlatMap](http://reactivex.io/documentation/operators/flatmap.html)이 적합하다.

여기서 백엔드 요청을 전달하기 위해 [Retrofit](http://square.github.io/retrofit/)를 사용한다. 이 요청은 부응하는 응답을 발행하는 Observable을 반환한다.

우리의 Retrofit Observable이 발출하는 객체가 이것이 성공인지 아닌지를 확인하는 기능이 있다고 하자. 우리는 이것을 확인해야 한다. 만약 성공이라면 백엔드로의 요청은  멈추어 더 이상 없어야 한다. 만약 성공이 아니라면 계속 진행해야 한다. 이제 takeUntil operator를 적용해보자.

```java
modelProvider.getItems()
    .flatMap(retroApiInterface::doBackendRequest)
    .takeUntil(response -> response.isSuccessful())
```

이제 한번이라도 성공 응답이 오면 원천 Observable은 자동으로 멈출것이며 따라서 백엔드 요청의 전송도 중단된다. 성공 응답이 아닌 경우에는 추가적인 백엔드 요청이 만들어 질 거이다.

이제 모든 것이 끝났다고 생각할 수도 있지만, 실제로는 우리가 다루지 않은 2가지 경우가 남아있다.

- 원천 Observable이 발행하는 항목이 없으면 어떻게 되는가?
- 백엔드가 한번도 성공 응답을 주지 않으면 어떻게 되는가?

두 시나리오 모두 Subscriber의 onComplete로 끝이 날 것이며 이것이 우리가 원하는 것은 아닐 것이다. 이는 [lastOrDefault](http://reactivex.io/documentation/operators/last.html) Operator로 해결할 수 있다.

```java
modelProvider.getItems()
    .flatMap(retroApiInterface::doBackendRequest)
    .takeUntil(response -> response.isSuccessful())
    .lastOrDefault(ServerResponse.createUnsuccessful())
```

우리는 원천 Observable에서 마지막으로 발행된 뒤 Retrofit Observable로 flatMap된 것을 원한다. 만약 원천 Observable이 발행하는 것이 없다면 우리 스스로 invalid response를 만들 것이다.

이제 오직 하나의 값만 발행하므로 이를 Single로 변환할 수 있다.

```java
modelProvider.getItems()
    .flatMap(retroApiInterface::doBackendRequest)
    .takeUntil(response -> response.isSuccessful())
    .lastOrDefault(ServerResponse.createUnsuccessful())
    .toSingle()
```

이제 구독을 할 수 있으며 발행된 응답에 더하여 에러 케이스까지 잘 처리할 수 있다.

```java
modelProvider.getItems()
    .flatMap(retroApiInterface::doBackendRequest)
    .takeUntil(response -> response.isSuccessful())
    .lastOrDefault(ServerResponse.createUnsuccessful())
    .toSingle()
    .subscribe(response -> {
        if (response.isSuccessful()) {
            // We made it.
        } else {
            // Not successful.
        }
    }, throwable -> {
        // Some error happened along the way.
    })
```

이제 문제는 작은 코드로 반응적 방식으로 해결 되었다. I’m more than happy to receive any feedback.

**Note:** 단순함을 위해 스케쥴링은 모두 제거했다. 보통은 백엔드 요청을 백그라운드로 옮기고 구독은 UI 변경을 할 수 있도록 UI 쓰레드로 되돌리기 위해 Scheduler를 적용할 것이다.

**Edit: ** [Ivan Škorić](https://medium.com/@skoric)가 지적한 바와 같이 takeUntil 대신 firstOrDefault를 사용하여 더 짧게 만들 수 있다 :

```java
modelProvider.getItems()
    .flatMap(retroApiInterface::doBackendRequest)
    .firstOrDefault(ServerResponse.createUnsuccessful(), response -> response.isSuccessful())
    .toSingle()
```

