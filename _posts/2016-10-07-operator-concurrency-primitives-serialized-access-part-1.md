---
layout: post
category: blog
title: 'Operator concurrency primitives: serialized access (part 1)'
date: '2016-10-07 20:50:00 +0900'
categories: RxJava
tags:
  - rxjava
published: true
---
[원본](http://akarnokd.blogspot.kr/2015/05/operator-concurrency-primitives.html)

## Introduction

[RxJava library](https://github.com/ReactiveX/RxJava)의 가장 중요한 요구는 **onNext**, **onError** 그리고 **onCompleted**의 *Observer*/*Subscriber* 메소드들이 순차적인 방식으로 호출되어야 한다는 것이다. 여러번, 우리는 이를 **직렬화된 접근(serialized access)**라 부른다. 하지만 이것은 일반적인 의미인 Java의 *Serializable* 인터페이스 또는 데이터 직렬화와는 관련이 없다.

이런 직렬화는 operator가 동시적 행동을 할 때 작동하기 시작한다. 다수의 데이터 스트림들을 **merge**,**zip** 또는 **combineLastest** operator등을 사용하여 합칠 때가 전형적인 예이다.

언뜻 보기에는 명백해 보이지 않아도 이런 직렬화가 요구되는 부분들이 있다. **takeUntil** operator가 그 예이다. 이 operator은, 하류(downstream) 연쇄(sequence)의 **onCompleted()**을 호출해서 메인 observable 연쇄를 종료하기 위해 또 하나의 *Observable*이 제공한 발행(emission)/종료(completion)을 사용한다. 메인-연쇄와 until-연쇄 둘 다 비동기로 동작할 수 있으므로, 둘 다 언제라도 서로에게 어떠한 **onXXX** 이벤트라도 발행할 수 있다. 하지만 여전히 하류(downstream)은 이벤트를 순차적인 방식으로 받아야 한다.

이러한 연쇄 직렬화 문제의 고전적 해결 방법은 *Observer* 메소드들에 **synchronized**를 사용하는 것이다. 이 방법이 작업이 완료되게 할 수 있을 것처럼 보일 것이다. 하지만 이 방법은 어느 하류(downstream)의 operator들이 상류(upstream)와 상호작용하는 중에 어떤 블로킹을 한다면 교착 상태(deadlock)가 되기 쉽다. 이런 교착 상태는 다른 RxJava의 기반 요소에서도 발생할 것이다.

그러므로, RxJava에서 lock을 유지하는 중에 다음 메소드들을 호출하는 금지한다.

- *Observer*: **onNext**, **onError**, **onCompleted**
- *Subscriber*: **onStart**, **setProducer**, **request**
- *Subscription*: **unsubscribe**
- *Scheduler.Worker*: **schedule**, **schedulePeriodically**
- *Producer*: **request**

(불행히, 우리는 이 제약을 자동으로 강제하게 할 수 없다. 그래서 풀-리퀘스트의 위반 사항을 손으로 체크한다.)

lock을 사용할 수 없으므로, 다른 직렬화 구성체(serialization construct)을 사용할 필요가 있다.

RxJava가 크게 두 종류의 직렬화 구성체을 사용한다. 나는 이 것들을 **emitter-loop**와 **queue-drain**으로 부른다.(내가 이것들을 발명한 것은 아니다. 하지만 내가 읽어본 대부분의 자바 동시성 책들은 이런 종류의 구성체을 언급하지 않았던 듯하다.)

Note: I'll be using Java 8 syntax in the remainder of the post to save on some boilerplate code. However, RxJava needs to be polyglot and is targeted at Java 6 source levels, therefore, make sure if you post a pull request to the core library, the syntax and **standard method calls** use Java 6 library features only.

## Emitter-loop

나는 이 구성체을 발행자-루프(emitter-loop)라고 부른다. 이는 다른 이들을 대신하여 발행 작업을 하는 스레드가 있고, 남은 작업이 없을 때 까지 발행을 유지하는 루프가 있음을 표시하기 위해 **boolean** 플래그를 사용하기 때문다.

이 구성은 synchronized 블록에서 접근할 수 있는 **boolean** 인스턴스 필드 emitting을 사용한다.

    class EmitterLoopSerializer {
        boolean emitting;
        boolean missed;
        public void emit() {
            synchronized (this) {           // (1)
                if (emitting) {
                    missed = true;          // (2)
                    return;
                }
                emitting = true;            // (3)
            }
            for (;;) {
                // do the emission work     // (4) 
                synchronized (this) {       // (5)
                    if (!missed) {          // (6)
                        emitting = false;
                        return;
                    }
                    missed = false;         // (7)
                }
            }
        }
    }

이제 이 예제에서 **emit()** 메소드가 어떻게 동작하는 지를 보자.

1. *synchronized* 블록에 들어갈 때, 두 상태 중 하나일 것이다 : 현재는 발행하는 이가 없거나 누군가가 발행하고 있다. 우리가 블록에 들어왔을 때, (4)에서 진행 중인 발행이 있으면, 그 스레드는 (1)이 종료되기 전까지 (5)의 synchronzed된 블록으로 들어갈 수 없다.
2. 이 경우에는 진행 중인 발행이 있으며, 수행할 작업이 더 있음을 알릴 필요가 있다. 이 예제는 이를 해결하기 위해 다른 *boolean* 플래그를 설정하는 간단한 방법을 보여준다. 어떤 종류의 발행이 있어야 할지에 따라, 남은 작업이 있음을 알려주는 지침(the more-work indicator)은 여러가지 데이터 타입을 가질 수 있다 (예를 들어,  RxJava의 일부 operator에서는 **java.util.List**). 나머지는 나중에 다루겠다.
3. 진행 중인 발행이 없는 경우, 오직 하나의 스레드만이 발행할 권리를 쟁취하게 될 것이다. 이 권리는 **emitting** 플래그를 true로 세팅하여 표시된다.
4. 스레드들 중 하나가 발행할 권리를 얻게되면, 우리는 루프로 들어가서 가능한 많은 발행 작업을 수행한다. 이것은 발행자-루프가 무엇을 해야하는지에 매우 높이 달려있다. 하지만 주의를 기울여 구현해야 한다. 만약 구현이 부적절하면, **신호 손실과 정체(signal loss and hangs)**로 이어질 수 있다.
5. 일단 루프가 모든 작업이 완료되었다고 생각하면, synchronized 블록으로 들어가길 시도한다. synchronized 블록으로 들어가기 직전에 다른 스레드들이 emit를 호출하였다면, 처리할 추가 작업들이 나타날 것이다. 오직 한 스레드만이 블록에 들어갈 수 있고 우리는 **missed** 플래그를 사용하므로, 그 블록에 들어가는 발행자-루프는 더 이상의 작업이 없으며 발행을 종료하거나 또는 추가 작업을 위해 다시 루프를 해야 함을 알게 될 것이다.
6. (5)의 synchronized 블록에 도달하는 동안 emit()를 호출한 스레드가 없었다면, 발행을 종료한다(플래그를 false로 세팅한다). 블록은 한번에 한 스레드만 허용하므로, 뒤이어 일어나는 (1)로의 입장은  false인 플래그를 볼 것이고 자신의 발행을 시작할 것이다.
7. 만약 빠트린 작업이 있다면, **missed** 플래그를 리셋하고 다시 루프를 진행한다. 플래그를 리셋하는 것은 무한 발행 루프를 피하기 위해 필수이다.

실제의 operator들에서는 발행자-루프를 위해 작업을 줄새우는('queue' up) 저마다의 특유의 방식이 있다.

그런 방법 중 하나는 발행자-루프의 앞에 어떤 스레드-세이프 자료 구조를 사용하고, 발행자-루프 내에서 작업을 추가하고, 자료 구조가 빌때까지 작업을 제거하는 것이다.

이를 보여주기 위해, **T** 값들의 발행을 직렬화하는 간단한 예제가 여기 있다.

    class ValueEmitterLoop<T> {
        Queue<T> queue = new MpscLinkedQueue<>();    // (1)
        boolean emitting;
        Consumer<? super T> consumer;                // (2)

        public void emit(T value) {
            Objects.requireNonNull(value);
            queue.offer(value);                      // (3)
            synchronized (this) {
                if (emitting) {
                    return;                          // (4)
                }
                emitting = true;
            }
            for (;;) {
                T v = queue.poll();                  // (5)
                if (v != null) {
                    consumer.accept(v);              // (6)
                } else {
                    synchronized (this) {
                        if (queue.isEmpty()) {       // (7)
                            emitting = false;
                            return;
                        }
                    }
                }
            }
        }
    }

*ValueEmitterLoop* 예제에서, 약간 다른 계속되는 발행 로직을 가진다:

1. 우리는 발행될 필요가 있는 값을 잡고 있기 위해 스레드-세이프한 queue를 사용한다. 자바의  *ConcurrentLinkedQueue*는 불필요한 고정비용(overhead)가 추가된다. 대신 나는  [JCTools](https://github.com/JCTools/JCTools)의 최적화된 queue 구현 사용을 제안한다.
2. 이 예제는 무한정 queue의 변형을 사용한다. 하지만 queue에 추가될 요소들의 최대값을 안다면(발행자-루프는 한정 backpressure 시나리오에 관계가 있기 때문이다.), 대신 *MpscArrayQueue*을 사용할 수 있다. 이 예제는 값들을 발행하기 위해 간단히 Consumer 콜백을 사용한다.
3. 먼저 무엇이던지 null이 아닌 값을 queue에 추가한다.(JCTools는 **null**값을 지원하지 않는다)
4. synchronized 블록에 들어갈 때, 다른 스레드가 발행 중임을 알게 되면, 당장 종료한다.  queue가 비었음이 이미 충분한 징후가 되므로 여기서는 missing 플래그를 설정할 필요가 없다.
5. 루프안에서, 우리는 queue에서 요소들을 꺼낸다.
6. **null** 요소들을 허용하지 않으므로 queue에서 반환된 null은 queue가 비었음을 나타낸다.
7. 일단 synchronized 블록에 들어가면, 여전히 queue가 비었는지를 확인한다. 만약 그렇다면 루프를 종료하고 emiting 플래그를 zero로 설정한다. queue는 synchronized 블록밖에도 제공되므로, (5)에서 값을 가져오고 그것이 **null**이여서 (7)의 뒤까지 진행되는 동안, (3)에서 추가 값이 queue에 들어갈 수 있다. 여기에 두가지 상호 배치가 일어날 수 있다. a) (7)이 실행되기 전에 새로운 값이 queue에 추가된다. 따라서 우리는 다시 루프한다; b) 발행자-루프를 떠나고, 다른 스레드가 발행자-루프로 들어올 수 있게 되는 (7) 이후에 새로운 값이 queue에 추가되면 queue의 폴링이 계속된다. 결론적으로 여기서 신호를 잃어버리는 일은 없다.

두번째 방법은 synchronized 블록안에서 'queueing' 작업을 하는 것이다. 그래서 스레드-안전하지 않은 자료 구조를 채택할 수 있다.

이 예제는 RxJava의 *SerializedObserver* 클래스에서 벌어지는 일이다. 이전 예제를 고쳐서 보여주겠다:

    class ValueListEmitterLoop<T> {
        List<T> queue;                           // (1)
        boolean emitting;
        Consumer<? super T> consumer;

        public void emit(T value) {
            synchronized (this) {
                if (emitting) {
                    List<T> q = queue;
                    if (q == null) {
                        q = new ArrayList<>();   // (2)
                        queue = q;
                    }
                    q.add(value);
                    return;
                }
                emitting = true;
            }
            consumer.accept(value);              // (3)
            for (;;) {
                 List<T> q;
                 synchronized (this) {           // (4)
                     q = queue;
                     if (q == null) {            // (5)
                         emitting = false;
                         return;
                     }
                     queue = null;               // (6)
                 }
                 q.forEach(consumer);            // (7)
            }        
        }
    }

*ValueListEmitterLoop*는 그것의 synchronized 블록들에서 훨씬 복잡한 로직을 가지고 있다. 하지만 동작이 그리 복잡하지는 않다:

1. 'queue' 동작을 위해  *java.util.List* 클래스를 사용한다. 그리고 이것의 non-null 여부가 빠트린 작업이 존재함의 지표가 된다. 처음에는 빠트린 작업이 없다. 그러므로 이 필드를 list 인스턴스 참조로 초기화하지 않는다.
2. 만약 발행 중인 다른 스레드가 있다면, queue에 추가한다(그리고 아직까지 누락된 작업이 없었다면 *ArrayList*로 초기화한다).
3. 발행을 쟁취한 스레드는 consumer에 즉시 직접 값을 발행할 수 있다. 이 작업을  **queue**를 통해 수행할 필요가 없으며(시간 절약을 위해), 이 시점에서는 **null**일 수 밖에 없으므로 요소들을 위해 queue를 체크할 필요가 없다(루프는 포괄적이고 queue가 **null**일 때만 종료하기 때문이다).
4. 이 시점에서 queue가 값을 쌓아 두고 있을 수 있으므로 synchronized 블럭에 들어가서 **queue**가 null이 아닌지를 확인할 수 있다.
5. 만약  **queue**가 **null**이면 발행을 종료한다. 다른 synchronized 블록에서 queue가 생성되었고/추가되었으므로, 경쟁이 없으며 발행 기회를 잃어버릴 가능성이 없다.
6. **queue**를 **null**로 설정한다. 이것은 더 이상 유요한 값들이 (이 시점에서) 없음을 알린다. 이것은 루프가 다시 시작되고 그동안 **emit()**이 호출되지 않았다면, 루프가 (5)의 체크 이후 발행을 종료하게 한다.
7. **q**의 값들을 단순히 for-each한다. 이것은 이 클래스에서 '분리되어' 있고 이것을 볼 수 있는 스레드가 없으므로 스레드-안전하다. 

불행히도, **consumer.accept()**같은 호출은 우리의 발행자-루프를 발행 상태로 두고 unchecked exception를 던질 수 있다. 이는 발행자-루프가 일반적인 값들 뿐만 아니라 에러 기반 발행을 사용할 때 흔하게 발생한다(*Notification*와 *NotificationLite*를 보라). 그런 경우에, 예외가 한 번 던져지면, 그냥 통과해버리는 다른 **emit()** 호출이 있을 수 있다.

이 상황을 피하기 위해, 각 호출을 **try-catch**로 감쌀 수 있다. 하지만 보통 **emit()**의 호출자는 이 문제를 통지받을 필요가 있다. 하지만 우리는 단순하게 예외가  자연스럽게 전파되도록 할 수 있다. 하지만 나가는 길에, 발행 플래그를 false로 다시 둔다.

    // same as above
    // ...
    public void emit(T value) {
        synchronized (this) {
            // same as above
            // ...
        }
        boolean skipFinal = false;             // (1)
        try {
            consumer.accept(value);            // (5)
            for (;;) {
                List<T> q;
                synchronized (this) {           
                    q = queue;
                    if (q == null) {            
                        emitting = false;
                        skipFinal = true;      // (2)
                        return;
                    }
                    queue = null;
                }
                q.forEach(consumer);           // (6)
            }
        } finally {
            if (!skipFinal) {                  // (3)
                synchronized (this) {
                    emitting = false;          // (4)
                }
            }
        }
    }

자바 6는 *Throwable* 예외(RxJava의 기본 catch 타입)를 catch 블록에서 다시 던지는 것이 허용되지 않으므로, 예외 던지기 처리를 강탈하기 위해 finally블록을 사용할 필요가 있다.

1. **skipFinal** boolean 변수를 선언한다. 이것이 true로 설정되면, 정상 종료를 가르키며, finally 블록의 로직을 건너띈다.
2. 빈 queue 경로에서, **skipFinal**을 true로 설정한다. 그래서 (3)이 false가 되고, synchronized 블록도 건너뛴다. emitting의 무조건 설정은 (2)에서 (3) 사이의 실행 기간 때문에 다른 누군가의 동작중인 발행 루프가 우발적으로 누설될 수 있기 때문이다.
3. 만약 (5)와 (6) 어느 쪽에서 예외를 던지든지, skipFinal은 여전히 false로 설정되어 있을 것이다. 이는 프로그램 흐름이 synchronized 블록에 입장할 수 있도록 한다. 일단 **finally** 블록이 끝나면, 던져진 예외는 호출 체인을 따라 가게 된다.
4. emitting을 false로 설정하여, 차후의 **emit()** 호출이 성공할 수 있게 한다. 

동시성 코드에 **synchronized**을 사용하는 것은 성능을 해칠 수 있다고 들었을 것이다. 그런데 왜 RxJava에서 이를 이렇게 많이 사용하는 것일까?

일반적으로, 경험에 근거한 첫번째 원칙은 당신 코드의 성능에 대해 추정하기 전에 먼저 측정을 하라는 것이다. 우리는 RxJava에서 그렇게 했고 놀라운 발견이 나왔다: **synchronized**는 더 높은 처리량을 제공한다 -in some benchmarks I might add.

RxJava는 threading/scheduling에 대해 고집이 없으므로, 동기 시나리오와 비동기 시나리오에서 동작을 해야 한다. RxJava를 사용하는 많은 어플리케이션은 대부분 동기 방식으로 자신들의 스트림들을 처리한다.

이제 동시적이고 단일-스레드 작업들을 좋아하는 자바는 이런 코드를 돕기 위해 **biased locking**과 **lock elision**같은 최적화를 특징으로 삼는다. 특히, 이 둘은 위 예제에서 **synchronized**를 제거 할 수 있고 추가적인 최적화들을 허용하며 그래서 순차적인 벤치마크 수행이 더 좋게 만든다(종종 오버헤드를 엄청난 규모로 줄인다).

대조적으로 (다음 파트에서 다룰 queue-drain 같은) lock-free atomics을 사용하는 대안들은 불가피한  오버헤드가 있고 복잡한 상태가 수반되면, 메모리 할당량이 늘어날 수 있다.

물론 java가 이 lock들에서 동시 실행을 감지하면, **biased locking**은 취소되고, 이때부터 일반적인 lock처럼 동작한다. (이 폐지는 세상을 멈추는 이벤트(stop-the-world)이고 상당한 지연을 유도할 수 있다.)

이런 확률론적인 성능 획득은 지나치기 어렵다. 그래서 RxJava는 가능한 여기에 의존하기로 결정하였다.

## Conclusion

RxJava operator를 작성할 때 이해할 필요가 있는 가장 중요한 알고리즘 중 하나는 이러한 이벤트를 연속으로 발행하게 하는 것 또는 작업이 한 번에 하나의 스레드에서만 동작하게 하는 것이다. 이 블로그 포스트에서, 나는 이른바 **발행자-루프**라는 접근을 소개하였다. 이것은 다른 대안들을 뛰어넘는 동시성 성능를 제공한다.

이것의 블로킹 성질과 bias-revocation의 비용 때문에, 동기로 실행되거나 시간의 50% 안에서 비동기 실행되는 operator에 이 테크닉을 이용할 것을 권고한다. 

하지만 만약 비동기-비율이 50%를 넘거나 언제나 관련된 다른 스레드가 있음(**observeOn**같은 경우)을 확신할 수 있다면 내가 queue-drain이라고 부르는 잠재적으로 더 나은 대안이 존재한다. queue-drain은 다음 파트에서 작성하도록 하겠다.
