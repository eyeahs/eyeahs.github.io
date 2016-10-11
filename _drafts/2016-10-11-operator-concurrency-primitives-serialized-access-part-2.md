---
layout: post
category: blog
published: false
title: 'Operator concurrency primitives: serialized access (part 2)'
---
[원본](http://akarnokd.blogspot.kr/2015/05/operator-concurrency-primitives_11.html)

## Introduction

Part 1에서는, 어떤 RxJava 메소드들과 작업들이 순차적인 방식으로 일어나는 것을 보장하기 위해 접근을 직렬화할 필요성을 설명하였다. **emitter-loop** 접근을 자세히 설명하고 이를 사용하여 어떻게 직렬화 구성체을 작성할 수 있는지를 보여주었다. 그리고 이 구성체는 자바 JIT이 단일-스레드 사용을 감지하였을 때 biased locking과 lock elision을 사용함으로 거의 모든 동기적 사용에서 탁월함을 상기하게 하고 싶다. 하지만 비동기 실행이 우세하거나 발행이 다른 스레드에서 일어나면, emitter-loop는 그것의 블로킹 성질로 인해 병목을 겪을 수 있다.

이 블로그 포스트에서, 내가 **queue-drain**이라고 부르는 또 다른, 블로킹없는 직렬화 구성체를 설명하겠다.

## Queue-drain

Queue-drain 구성체는 emitter-loop와 비교해 상대적으로 짧다:

    class BasicQueueDrain {
        final AtomicInteger wip = new AtomicInteger();  // (1)
        public void drain() {
            // 준비 작업
            if (wip.getAndIncrement() == 0) {           // (2)
                do {
                    // 배출(draining) 작업
                } while (wip.decrementAndGet() != 0);   // (3)
            }
        }
    }

기본적인 구조체는 다음과 같이 작동한다:

1. 우리는 우리가 원자적(atomically) 값을 증가시킬 수 있는 숫자 변수가 필요하다. 나는 보통 이것을 **wip**(요컨대 work-in-progress)이라고 부른다. 이것은 완료되어야 할 작업의 양을 나타내며 심지어 기초가 되는 자바 런타임이 (2)와 (3)에 필수적인 내제 기능을 가지고 있다면 대기가 없는 (wait-free) queue-drain를 만들 수 있기까지 해준다. RxJava에서는 알려진 사용 패턴에서는 오버플로우가 발생하지 않을 거 같으므로 *AtomicInteger*를 사용한다. 물론, 필요하다면 *AtomicLong*으로 변경할 수 있다.
2. 현재 wip의 총계를 원자적으로 회수하고 1을 증가시킨다. 0에서 1로 증가시킨 스레드는 드레인 권한을 얻어내고 루프에 들어갈 수 있다. 다른 스레드들은 단지 값을 더 올리고 메소드를 종료한다.
3. 작업 항목이 배출(drain)되고 처리될 때 마다, 원자적으로 감소시키고 **wip**의 총계를 확인한다. 만약 0에 도달하면, 루프는 종료된다. 감소를 시키는 메소드는 오직 하나므로, 과정중에 어떠한 신호들도 잃어버리지 않음이 보장된다. (2)와 (3) 둘 다 원자적 변경이 있고 (3)에서 **wip == 1**인 상태 이후, 만약 (2)가 이기면 (3)에는 1이 남아있어서 다른 반복이 일어난다. 또는 만약 (3)이 이기면 (3)이 추가 개입없이 떠나는 동안 (2)는 **wip**를 0에서 1로 증가시키고 드레인 루프에 들어갈 것이다.

만약 당신이 이전 포스트의 emitter-loop 예제를 기억한다면, 두개의 데이터 구조들 사이에 유사성을 그릴 수 있다.

Emitter-loop에서는 스레드가 **emitting == false** 상태를 찾고 이를 true로 설정하면 발행 권한을 얻었다 ; queue-drain에서는 **wip**를 0에서 1로 증가시킬 수 있는 스레드가 승리한다. Emitter-loop에서는 발행 권한을 얻지 못한 스레드는 **missing** 플래그를 true로 설정한다; queue-drain에서는 패배한 스레드는 **wip**을 더 증가시킨다(2 그리고 그 이상으로).  떠나는 경계에서, emitter-loop는 **missing** 플래그가 설정되어 있는지를 확인하고 다시 루프한다; queue-draing에서는 감소 전에  **wip** 값이 1보다 크면  새로운 반복이 일어난다. emitter-loop에서는 **missing** 플래그가 깨끗하다면, 루프를 종료한다; queue-draing에서는 **wip**을 0으로 감소시키면 루프를 종료시킨다.

이를 마음에 두면, queue-draing 구조체를 사용하여 이전 포스트의 *ValueEmitterLoop*의 블로킹없는 변형을 만들 수 있다:

    class ValueQueueDrain<T> {
        final Queue<T> queue = new MpscLinkedQueue<>();     // (1)
        final AtomicInteger wip = new AtomicInteger();
        Consumer consumer;                                  // (2)

        public void drain(T value) {
            queue.offer(Objects.requireNonNull(value));     // (3)
            if (wip.getAndIncrement() == 0) {
                do {
                    T v = queue.poll();                     // (4)
                    consumer.accept(v);                     // (5)
                } while (wip.decrementAndGet() != 0);       // (6)
            }
        }
    }

이 구조체는 emitter-loop 변형과 꽤 비슷하다. 하지만 *synchronized* 블록들이 없어 '장황함'이 줄었다:

1. 소비될 수 있을 때 까지 값을 유지하기 위해 여전히 [JCTools](https://github.com/JCTools/JCTools)의 더 나은 queue를 사용한다. 값들의 최대치를 알수 있다면, *MpscArrayQueue*를 적용할 수 있다. 하지만 그것들은 **offer/pool**의 측면에서 [not wait-free](http://psy-lob-saw.blogspot.kr/2015/04/porting-dvyukov-mpsc.html)임을 주의하라.
2. **consumer** 인스턴스안으로 큐에 저장된 값들을 하나하나씩 소모시킬 것이다.
3. 먼저, **null**이 아닌 값을 큐에 제공한다.
4. drain 권한을 얻은 스레드는 큐에서 값을 꺼낸다. **wip**은 본질적으로 큐에 있는 항목 개수의 하한값을 나타내므로, **poll()**은 절대로 **null**값을 반환하지 않을 것이다.
5. consumer에 값을 전달한다.
6. **wip**가 남은 값이 있음을 알리면 다시 루프를 하고, 아니면 종료한다. 이제 (6)에 도착하였을 때 큐가 비어 있지 않았을 수 있다. 하지만 **wip**의 변화가 동기화 시점이므로 이는 문제가 되지 않는다: (3)과 증가 사이에 임의의 딜레이가 있더라도, drain 루프는 증가가 언제 일어나는지를 알고 종료할 수 있고, 딜레이된 스레드는 큐를 drain하는 것을 계속 진행할 것이다.

*ValueEmitterLoop*와 *ValueQueueDrain*의 동기 성능를 벤치마크로 측정한다면, 후자가 준비 기간 이후 처리량에서 불리하다. 

그 이유는 피할 수 없는 원자적 처리 때문인데, 원자적 처리에는 심지어 다툼이 있는 케이스가 아니라도 몇 사이클이 필요하다. 이는 현대 멀티코어 CPU들에서 필수적인 write-buffer flush에서 기인한다:**drain()**에서는 값마다 2개의 원자적 증가/감소가 있다. 게다가 Mpsc 큐 구현에 관련된 추가적인 원자적 처리가 있다. 대조적으로 *ValueListEmitterLoop*은 더 빠르다. 이는 로직에서 일단 synchronized 블록들이 최적화된다면 소위 **단축-경로(fast-path)**라는 것을 활용하기 때문이다.

내가 언급한 벤치마크은 consumer의 콜백에서 단순한 작업량을 가지고 있기 때문에 직렬화 접근의 오버헤드를 측정한다. 작업량 및 작업 분배에 따라 연비는 달라질 것이다.

queue-drain이 맞닥뜨릴 drain() 메소드의 동시 실행을 제거하기 위해 queue-drain가이런 단축경로를 가지도록 바꿀 수 있다. 이는 잘 상호 배치된 동시 사용이나 그저 평범한 순차적 접근에서 비롯될 것이다.

하지만 어떤 경우들에서 이런한 단축-경로 로직은 꽤 복잡하고 약간 느린 경로의 그것과 비슷한 로직을 가지며, 구현이 장황할 수 있음을 주의하라. 평범한 queue-drain을 먼저 벤처마크로 측정하거나 그것의 구현을 시도하기 전에 set-to-1 트릭(뒤에 설명할 것이다)을 적용하기를 제안한다.

*ValueQueueDrain* 예제에서, 단축 경로를 구현하는 것은 충분히 짧다(그리고 그럴 가치가 있다):

    class ValueQueueDrainFastpath<T> {
        final Queue<T> queue = new MpscLinkedQueue<>();
        final AtomicInteger wip = new AtomicInteger();
        Consumer consumer;

        public void drain(T value) {
            Objects.requireNonNull(value);
            if (wip.compareAndSet(0, 1)) {          // (1)
                consumer.accept(value);             // (2)
                if (wip.decrementAndGet() == 0) {   // (3)
                    return;
                }
            } else {
                queue.offer(value);                 // (4)
                if (wip.getAndIncrement() != 0) {   // (5)
                    return;
                }
            }
            do {
                T v = queue.poll();                 // (6)
                consumer.accept(v);
            } while (wip.decrementAndGet() != 0);
        }
    }

Even though *ValueQueueDrainFastpath*가 더 많은 원자성을 가지고 있는 것 처럼 보일지라도, offer와 poll에서 원자성을 피하기 때문에 순차적 사용에서 더 뛰어나다:

1. 값이 **null**이 아님을 확실하게 한 뒤, wip의 값을 1로 비교와 바꾸기(CAS)를 한다(0에서 1로의 변환를 시도한다). (여담이지만, CAS가  **wip.get() == 0 &&**을 추가하면 )
2. After making sure value is non-null, it tries to compare-and-swap (CAS) in the wip value of 1 (attempts the 0-1 transition). (On a side note, adding wip.get() == 0 && before the CAS might improve the performance a bit, but its effect is highly CPU and usage-pattern dependent because it creates a dependent load with respect to CAS. My benchmarks on different CPU types were inconclusive.)
3. We simply give the value directly to the consumer without even touching the queue. This is where the main performance improvement comes.
4. We now need to atomically decrement the the wip counter and if there was no concurrent call to drain() that reached (5). (comment에서는 ,we will quit이 따라와야 하지 않냐고??) One might think a CAS from 1 to 0 would work here, but if the CAS fails, we need to decrement wip anyway to be able to do the slow path (since we are still in 'emission' mode). 
5. If the CAS in (1) failed, we attempt the regular slow-path drain loop, therefore, we enqueue the value.
6. We increment wip atomically and if it wasn't zero we quit (the logical not is required because the drain loop is shared between the fast-path and the slow-path). This check is mandatory because a concurrent drain() call could enter the fast path and leave it before the first thread reaches (5), leaving wip at zero which when incremented to 1 is eligible to enter the drain loop.
7. Finally, if either the slow-path was taken or the fast-path couldn't do (3) in time, we perform the usual drain loop. While a thread is in the drain loop, all other threads will fail the CAS during the time and take the slow path.

Note that this fast-path approach is a tradeoff: one trades the potential sequential fast-path case with the one extra CAS failure on the slow path (or with the wip.get() == 0 check, a less likely CAS failure on the slow-path but a potential effect of a dependent load on the fast-path). Again, benchmark your case, don't just assume.

If you look closely, you can discover the similarity to the ValueListEmitterLoop shown in the previous post: basically the thread that wins the right of emission will call consumer.accept() first as the fast path and revert to the slow-path (the loop) if there were some activity in the meantime and missed was set to true (i.e., wip was greater than 1 here).  ( comment에서는 0 here이여야 하지 않는가라고?)

Apart from the fast-path optimization, there is another option to reduce the number of atomic operations in the drain loop. Remember the analogue of the wip variable between the emitter-loop and the queue-drain? It turns out any wip value above 1 just means there are more values available and the queue can 'tell' it if run out of values. We can save one atomic decrement per value if we could just drain the queue completely and loop again if some values arrived just before a single atomic decrement:

    class ValueQueueDrainOptimized<T> {
        final Queue<T> queue = new MpscLinkedQueue<>();
        final AtomicInteger wip = new AtomicInteger();
        Consumer consumer;

        public void drain(T value) {
            queue.offer(Objects.requireNonNull(value));
            if (wip.getAndIncrement() == 0) {
                do {
                    wip.set(1);                              // (1)
                    T v;
                    while ((v = queue.poll()) != null) {     // (2)
                        consumer.accept(v);
                    }
                } while (wip.decrementAndGet() != 0);        // (3)
            }
        }
    }     

The ValueQueueDrainOptimized example looks somewhat similar to ValueQueueDrain but works a bit differently:

1. Once we enter the drain loop, we set wip back to 1. Since we will completely drain the queue, there is no need to loop back again (and find a potentially empty queue again). This is equivalent when we set missing flag to false in the emitter-loop example.
2. Instead of one at a time, now we have an inner loop which drains the queue. We don't use queue.size() or queue.isEmpty() but rely on the fact that an empty queue will return null when polled, saving on more atomic operations.
3. We decrement wip and if it reaches zero, the loop quits. In the optimistic case, if there was a burst of offers just before (1), we drained them all and due to wip set to 1, there will be only a single iteration of the main drain loop. If there was an offer just before (3), it is not a problem because the synchronization is made sure by the two atomic increment/decrement pair: so either (3) doesn't decrement to zero and loops again or it decrements to zero and allows the if (getAndIncrement() == 0) enter the drain loop.

하지만 이 최적화 역시 한계가 있음을 주목하라. 

Note, however, that this optimization has its limits too. Even if it 'batches' the incoming values, the additional cache-coherence traffic due to (2) might not improve the performance in some cases, depending on the call pattern to drain().

After the examples and explanations, the queue-drain may seem too problematic: its optimization potential and performance is quite dependent on the usage pattern and one is more likely to use the emitter-loop approach instead.

However, there is an operator used by almost everyone which shines with the queue-drain pattern: observeOn. Its current implementation, although it still has room for improvements, utilizes the optimization shown in the ValueQueueDrainOptimized example along with the fact that the queue (now a wait-free SpscArrayQueue) offer and poll sides are accessed from exactly one thread each (except in certain backpressure scenarios, more on this on a later post).

## Conclusion

In this post, I've introduced the second serialization approach I call queue-drain. I showed its basic form and two optimized variants and explained that their performance is dependent on the value arrival pattern to the drain() method (and sometimes on CPU type). Make sure you benchmark your implementation.

Its non-blocking lock-free (and sometimes wait-free) structure makes it better suited for medium or highly-concurrent serialization scenarios or cases when its two parts (queueing and draining) can be made sure to run on different threads.

In the next post, I'll talk about various thread-safe Producer implementations; some of which will utilize the emitter-loop or queue-drain approach I detailed in the posts.