---
layout: post
category: blog
published: false
title: RxJava Wiki - Custom operator
---
# 커스텀 operator 구현하기

# 도입부

RxJava는 가장 흔한 반응적 데이터-흐름 패턴들을 지원하기 위해 100개 이상의 operator들을 가지고 있다. 일반적으로, 표준 보증으로 덜 흔한 패턴을 구성할 수 있게 해주는 operator들(대표적으로 `flatMap`, `defer` 그리고 `publish`)의 조합이 있다. 만약 당신이 흔치 않은 패턴이 필요하고, 올바른 operator들을 찾을 수 없을 거 같다면, 우리의 이슈 리스트(또는 Stackoverflow)에 먼저 문의해라.

만약 당신의 유스케이스에 적용할 수 있는게 없다면, 커스텀 operator을 구현하고 싶을 것이다. **Operator 작성은 어렵다**는 것을 경고한다: operator를 작성할 때는, `observable` **프로토콜**, **구독해지(unsubscription)**, **배압(backpressure)** 그리고 **동시실행**을 고려해야 하며 충실하게 글자 그대로 지켜야 한다.

이 페이지는 간결함을 위해 Java 8 문법을 사용한다.

# 고려사항

## Observable 프로토콜

`observable` 프로토콜은 당신이 `observable`의 메소드들, `onNext`, `onError` 그리고 `onCompleted`를 순차적 방식으로 호출해야 함을 명확하게 제시한다. 다른 말로하면, 이들은 동시적으로 호출될 수 없고, 직렬화되어야 한다. `SerializedObservable`과 `SerializedSubscriber` 레퍼들은 이를 할 수 있도록 당신을 도와준다. 이러한 직렬화가 발생해야 하는 경우들이 있음을 주의하라.

추가로, 여기에 `Observer`에 있는 메소드 호출의 기대 패턴이 있다:

	onNext* (onError | onCompleted)?

(역주 : `onNext`는 0번 이상 호출 된다. 그 다음 `onError` 또는 `onCompleted`가 둘 중 하나만의 호출이 뒤따르며, 이는 마지막 호출이 될 것이다.)

커스텀 operator와 그것의 push side도 이 패턴을 준수해야 한다 예를 들어, 당신의 operator가 `onNext`를 `onError`로 바꾸면, 상류(upstream)는 멈춰야 하며 하류(downstream)는 메소드 호출을 더 이상 할 수 없다.

## 구독해지

기본적인  `Observer` 메소드는 상류의 소스(upstream source)에게 이벤트 발행 중단의 신호를 보낼 직접적인 수단이 없다. 그것은  `Observable.subscribe(Observer)`가 반환하는 `Subscription`을 받아야 하며 **그리고** 스스로 비동기여야 한다.

이 결점은 `Subscription` 인터페이스를 구현한 `Subscriber` 클래스를 도입하여 해결되었다. 이 인터페이스는 `Subscriber`가 이벤트에 더 이상 관심이 없는가를 감지할 수 있게 해준다.

    interface Subscription {
       boolean isUnsubscribed();

       void unsubscribe();
    }

operator는 이를 통해 이벤트 발행 이전에 `Subscriber` 상태를 능동적으로 검사할 수 있다.

경우에 따라서는, 발행 직전에 확인하지 않고 child의 구독해지에 즉시 반응할 필요가 있다.  이를 지원하기 위해, `Subscriber` 클래스는 operator의  `Subscription`을 등록하는 `add(Subscription)` 메소드를 가지고 있다. 하류(downstream)에서 `Subscriber.unsubscribe()`를 호출하면 이 `Subscription`은 구독해지된다.
(역주: 이 `Subscription`이 구독해지되면 `Subscriptions.create()`에 전달한 `Action0`가 호출된다.)

    InputStream in = ...

    child.add(Subscriptions.create(() -> {
        try {
           in.close();
        } catch (IOException ex) {
           RxJavaHooks.onError(ex);
        }
    }));

## backpressure

이 기능의 이름은 자주 오해된다. 이는 하류가 몇 개의 `onNext` 이벤트를 받을 준비가 되었는지를 상류에게 알려주는 것에 대한 것이다. 예를 들어, 만약 하류가 5개를 요청하면, 상류는 `onNext`를 오직 5번만 호출할 수 있다. 만약 상류가 5개의 요소들 제공할 수 없고 3개만을 제공한다면, 상류는 3개의 요소들과 (operator의 목적에 따라) `onError`나 `onCompleted`를 뒤따라 전달할 것이다. 요청은 어느 정도 누적된다. 만약 하류가 5개를 요청한 뒤 2개를 요청하면, 처리되지 않은 7개의 요청이 있을 것이다.

Backpressure 처리는 대부분의 operator들에게 다량의 복잡함을 추가한다: 하류가 몇 개의 요청을 했는지를 추적해야 한다면, 몇 개가 전달되었는지(보통 요청량에서 제외한다)와 가끔은 몇 개의 항목들이 아직 전달 가능한지(하지만 요청이 없으면 전달될 수 없다)를 추적해야한다. 추가로, 하류는 어떤 스레드에서도 request할 수 있으며 `onXXX` 메소드와는 다르게 공통 스레드에서의 발생이 요구되지 않는다.

Backpressure '경로'는 상류와 하류 사이에 `Producer` 인터페이스를 통해 개설된다:

    interface Producer {
        void request(long n);
    }

상류가 backpressure를 지원하는 경우,  `Producer` 인터페이스를 구현하고 자신의 하류 `Subscriber`의 `Subscriber.setProducer(Producer)` 메소드를 호출 할 것이다. 그 다음에 그 하류는 무한 스트림을 시작하기 위해 `Long.MAX_VALUE`으로 응답할 수 있다(실질적으로 직접적인 상류와 하류 사이에는 backpressure가 존재하지 않는다), 또는 다른 양수 값으로도 가능하다. 0의 요청 값은 무시될 것이다.

Protocol-vise, Producer가 설정 될 수 있는 제한 시간이 없으며 절대로 나타나지 않을 수도 있다. Operator들은 이 상황을 처리할 준비가 되어 있어야 하며 상류가 무한 모드로 동작한다고 가정해야 한다(마치 `Long.MAX_VALUE` 요청 처럼)

대개, operator는 하류에서의 요청들과 구독해지들 모두를 처리하기 위해  `Producer`와 `Subscription`을 단일 클래스에 구현할 것이다:

    final class MyEmitter implements Producer, Subscription {
        final Subscriber<Integer> subscriber;

        public MyEmitter(Subscriber<Integer> subscriber) {
            this.subscriber = subscriber;
        }

        @Override
        public void request(long n) {
            if (n > 0) {
               subscriber.onCompleted();
            }
        }

        @Override
        public void unsubscribe() {
            System.out.println("Unsubscribed");
        }

        @Override
        public boolean isUnsubscribed() {
            return true;
        }    
    }

    MyEmitter emitter = new MyEmitter(child);

    child.add(emitter);
    child.setProducer(emitter);

불행히도, API의 실수로 인해 `Subscriber`에 `Producer`를 구현할 수 없다: `Subscriber`는 **거치 요청(deferred requesting)**(`setProducer`가 호출될 때 까지 지역 요청 합계를 저장하고 모운다)을 수행하기 위해 protected final `request(long n)` 메소드를 가지고 있다.

# 동시성

Operator들을 작성할 때, 보통 표준 자바 동시성의 기초 요소들을 통해 동시성을 처리해야 한다: `AtomicXXX` 클래스들, volatile 변수들, `Queue`들, 상호 배제(mutual exclusion), Executor들, 등등

### RxJava 도구들



