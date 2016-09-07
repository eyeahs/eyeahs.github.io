---
layout: post
category: blog
title: RxJava의 Side effect 메소드
date: '2016-09-07 17:15:00 +0900'
categories: RxJava
tags:
  - rxjava
published: true
---
[원본](http://www.grokkingandroid.com/rxjavas-side-effect-methods/)

RxJava의 Observable 클래스는 방출된 아이템들의 스트림을 당신이 필요한 종류의 데이터로 변환할 때 사용할 수 있는 수많은 메소드들을 가지고 있다. 이 메소드들은 RxJava의 핵심이며 매력에 큰 부분을 차지한다.

다지만 아이템들의 스트림을 전혀 수정하지 않는 메소드들도 존재한다. 나는 이런 메소드들을  Side effect 메소드라고 부른다.

## Side effect 메소드란 무슨 의미일까?

Side effect 메소드 그 자체는 당신의 스트림에 영향을 주지 않는다. 대신 이것들은 특정 이벤트가 발생하였을 때 당신이 이 이벤트들에 반응할 수 있게 하기 위해 호출된다.

예를 들어  : 만약 당신이 어떤 에러가 발생하였을 때 마다 당신의 **Subscriber**의 콜백 밖에서 무엇인가를 하고 싶다면, **doOnError()**를 사용하고 여기에 에러가 발생할 때마다 사용될 [함수형 인터페이스](http://www.lambdafaq.org/what-is-a-functional-interface/)를 전달해야 한다.

```java
someObservable
  .doOnError(new Action1() {
     @Override
     public void call(Throwable t) {
        // use this callback to clean up resources,
        // log the event or or report the
        // problem to the user
     }
  })
  //…
```
가장 중요한 부분은 call() 메소드이다. 이 메소드의 코드는 Subscriber의 onError()메소드가 호출되기 전에 실행 될 것이다.

예외외에도 RxJava는 당신이 반응할 수 있는 더 많은 이벤트들을 제공한다.

### 이벤트들과 그에 대응하는 side effect 작업들

| Method            | Functional Interface     | Event                                    |
| ----------------- | ------------------------ | ---------------------------------------- |
| doOnSubscribe()   | Action0                  | Subscriber가 Observable을 구독.              |
| doOnUnsubscribe() | Action0                  | Subscriber가 subscription를 구독해지.          |
| doOnNext()        | Action1\<T\>               | 다음 아이템이 방출됨.                             |
| doOnCompleted()   | Action0                  | Observable이 더 이상 아이템을 방출하지 않음.           |
| doOnError()       | Action1\<T\>               | 에러가 발생됨.                                 |
| doOnTerminate()   | Action0                  | 에러가 발생하거나 Observable이 더 이상 방출할 아이템이 없을 때 - 종료 메소드 이전에 호출된다. |
| finallyDo()       | Action0                  | 에러가 발생하거나 Observable이 더 이상 방출할 아이템이 없을 때 - 종료 메소드 이후에 호출된다. |
| doOnEach()        | Action1\<Notification\<T\>\> | 아이템이 방출되거나, Observable가 종료되거나 오류가 발생하였을 때. 알림 객체는 이벤트 종류에 대한 정보를 포함한다. |
| doOnRequest()     | Action1\<Long\>            | 다운스트림 operator가 아이템 방출을 요청               |

\<T\>는 방출된 항목의 타입 또는 **onError()**  메소드의 경우에는 던져진 Throwable의 타입을 나타낸다.

함수형 인터페이스들은 모두 [Action0](http://reactivex.io/RxJava/javadoc/rx/functions/Action0.html) 또는 [Action1](http://reactivex.io/RxJava/javadoc/rx/functions/Action1.html) 타입이다. 이는 이 인터페이스들의 단일 메소드가 어떤 것도 반환하지 않으며, 특정 이벤트에 따라 0개 또는 1개의 인수를 가지고 감을 의미한다.

이 메소드들은 반환하는 것이 없으므로, 방출되는 아이템들을 변경하기 위해 사용할 수 없다. 따라서 아이템들의 스트림을 전혀 변경하지 않는다. 대신 이 메소드들은 디스크에 쓰거나, 상태를 정리하는 등 이벤트들의 스트림 대신 시스템 자신의 상태를 조작하는 것 같은 부수 효과(side effect)를 일으키는 것을 의도한다.

**Note:** Side effect 메소드 자체는 (doOnNext(), doOnCompleted() 기타 등등) Observable를 반환한다. 이는 [Fluent interface](https://en.wikipedia.org/wiki/Fluent_interface)를 유지하기 위함이다. 하지만 반환된 Observable은 원본 Observable과 동일한 타입이며 동일한 항목을 방출한다.

## Side effect 메소드는 어디에 유용한가?

이제 Side effect 메소드가 아이템들의 스트림을 변경하지 않기 때문에 이들에게 다른 용도가 있을 것이다. 나는 이들을 활용할 수 있는 여기 3가지 예제를 제시한다.

* 디버깅을 위해 **doOnNext()**를 사용하라
* **flatMap()**의 에러 처리를 위해 **doOnError()**를 사용하라
* 네트워크 결과를 저장하고 캐쉬하기 위해 **doOnNext()**를 활용하라

이제 이 예제들을 상세하게 보도록 하자.

### 디버깅을 위해 **doOnNext()**를 사용하라

RxJava를 사용할 때 당신의 **Observable**이 왜 기대한 대로 동작하지 않는지 종종 궁금하다. 특히 막 시작했을 때 그렇다. 어떤 소스를 당신이 구독하고 싶은 것으로 변환하기 위해 Fluent API를 사용하므로, 당신은 이 변환 파이프라인의 끝에서 당신이 받는 것만 볼 수 있다.

나는 Java의 Stream에서도 동일한 초기 경험을 했었다. 보통 당신도 동일한 경험을 할 것이다. 당신은 어떤 타입의 한 스트림에서 다른 타입의 다른 스트림으로 이동하는 fluid API를 가지고 있다. 만약 이것이 기대한 대로 동작하지 않는다면 어떻게 될까?

Java 8 Stream의 **peek()**메소드를 사용할 수 있다. 내가 RxJava를 시작할 때 이와 비슷한 어떤 것을 사용할 수 있을지가 궁금했다. 물론 존재한다. RxJava는 더 많이 제공한다.

당신은 처리 파이프라인 안에서 일어나는 일이 무엇인지 그리고 중간 결과가 무엇인지 보기 위해 어디서든  **doOnNext()** 메소드를 사용할 수 있다.

여기 그 예제가 있다:

```java
Observable someObservable = Observable
        .from(Arrays.asList(new Integer[]{2, 3, 5, 7, 11}))
        .doOnNext(System.out::println)
        .filter(prime -> prime % 2 == 0)
        .doOnNext(System.out::println)
        .count()
        .doOnNext(System.out::println)
        .map(number -> String.format(“Contains %d elements”, number));

Subscription subscription = o.subscribe(
        System.out::println,
        System.out::println,
        () -> System.out.println(“Completed!”));
```

그리고 그 코드의  결과가 여기 있다:

    2
    3
    3
    5
    5
    7
    7
    11
    11
    4
    Contains 4 elements
    Completed!

이 방법으로 당신은 당신의 Observable이 기대한대로 동작하지 않을 때 무슨 일이 일어나고 있는지에 대한 값진 정보를 얻을 수 있다.

**doOnError()** 와 **doOnCompleted()** 메소드들은 당신의 파이프라인의 상태를 디버깅할 때도 유용할 수 있다.

**Note:** If you’re using RxJava while developing for Android please have a look at the Frodo and Fernando Ceja’s post explaining about the motivation for and usage of Frodo. With Frodo you can use annotations to debug your **Observables** and **Subscribers**.


**doOnNext()**와 **doOnError()**를 사용한 이 방식은 시스템의 상태를 많이 바꾸지는 않는다 – apart from bloating your log and slowing everything down. 🙂

하지만 이 operator들을 위한 다른 방식들이 있다. 그리고 이 케이스들에서 당신은 실제로 시스템의 상태를 변경하기 위해 이 메소드들을 사용한다. 이것들을 살펴보도록 하자.

### **flatMap()**의 에러 처리를 위해 **doOnError()**를 사용하라

당신이 네트워크를 통해 어떤 리소스에 접근하기 위해 [Retrofit](http://square.github.io/retrofit/)를 사용한다고 가정하자. Retrofit은 observable을 지원하므로 당신은 **flatMap()**을 사용하여  이 호출들을 당신의 processing chain안에서 손쉽게 사용할 수 있다.

네트워크 관련 호출은 다양한 방식으로 -특히 모바일에서- 잘못 될 수 있다. 이 경우 당신은 Observable이 동작을 멈추는 것을 원하지 않을지도 모른다. 만약 당신이 당신의 Subscriber의 onError() 콜백 하나에만 의지한다면 그럴 수 있을 것이다.

하지만 **flatMap()** 메소드 내부에 Observable이 있다는 것을 잊지말자. 그래서 당신은 UI를 어떻게 해서든 변경하기 위해 doOnError() 메소드를 사용할 수 있다. 그러면서 다음 이벤트들을 위한 Observable 스트림은 여전히 동작한다.

이는 이렇게 된다:

```java
flatMap(id -> service.getPost()
   .doOnError(t -> {
      // report problem to UI
   })
   .onErrorResumeNext(Observable.empty())
)
```
이 방법은 잠재적으로 되풀이해서 일어나는 UI 이벤트의 결과로 리모드 리소스를 쿼리할 때 특히 유용하다.

### 네트워크 결과를 저장하고 캐쉬하기 위해 **doOnNext()**를 활용하라

체인 안의 특정 시점에서 네트워크 조회를 하면, 응답 결과를 로컬 데이터베이스나 다른 캐쉬안에 저장하기 위해 doOnNext()를 사용할 수 있다.

간단히 하면 다음과 같을 것이다:

    // getOrderById는 네트워크에서 새로운 orders를 받은 뒤
    // orders의 observable을 반환한다.
    // Observable<Order> getOrderById(long id) {…}
    
    Observable.from(aListWithIds)
         .flatMap(id -> getOrderById(id)
                              .doOnNext(order -> cacheOrder(order))
         // 다음 처리를 계속 진행한다.
이 패턴을 더 자세히 적용하려면 다수의 소스에 접근하는 것에 대한 Daniel lew의  훌륭한 [블로그 포스트](http://blog.danlew.net/2015/06/22/loading-data-from-multiple-sources-with-rxjava/)를 보라.

## 마무리

지금까지 보았듯이, RxJava의 side effect 메소드들을 다양한 방식으로 사용할 수 있다. Side effecy 메소드들은 방출된 항목들의 스트림을 변경하지는 않지만, 전체 시스템의 상태를 변경할 수 있다. 이는 네트워크 조회의 결과인 객체를 데이터베이스에 저장하는 것 같은 처리 파이프라인의 내부 특정 시점의  **Observable**의 현재 아이템들을 가장 간단히 로깅할 수 있는 방법이다. 

In my next post I am going to show you how to use RxJava’s hooks to get further insights. Stay tuned!
