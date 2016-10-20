---
layout: post
category: blog
published: false
title: Java Concurrency 정리
splash: ''
tags: ''
---
##Chapter 2
**경쟁조건**Race Condition : 
	- (J.C)타이밍일 딱 맞았을 때만 정답을 얻는 경우에는 경쟁조건을 가진다고 한다.
	- (Wiki)Output이 다른 통제 불가한 이벤트들의 시퀀스나 타이밍에 의존하는 경우의 동작
	- 이를 피하려면 - 하나의 공유된 상태에 대한 복합 동작을 단일 연산으로 만들어야 한다.
    
**데이터 경쟁**Data Race : 공유된 필드에 대한 접근을 동기화로 보호하지 않았을 때 

**단일 연산** : 작업 A를 실행 중인 스레드 관점에서 다른 스레드에서 수행되는 작업 B를 볼 때, 작업 B가 모두 수행되었거나 전혀 수행되지 않은 두가지 상태로만 파악됨.

**복합 동작**Compound action : 점검 후 행동과 읽고 수정하고 쓰기 같은 일련의 동작들.

**암묵적 락**Intrinsic lock : (= 모니터 락(monitor lock)) 자바에 내장된 락. 모든 자바 객체에는 내장된 락이 있음

**재진입성**reentrant : 특정 스레드가 자기가 이미 획득한 락을 다시 확보 가능함.
**뮤텍스**Mutex(mutaul exclusion lock 상호 배제 락) : 한 번에 한 스레드만 특정 락 소유.
**세마포어**Semaphore : 최대 허용치만큼 접근 가능.

  - 상태 일관성을 위해서는 관련 있는 변수들은 하나의 단일 연산으로 갱신해야 함
  - 락 활용하는 일반적 예 - 모든 변경 가능한 변수를 객체 안에 캡슐화하고, 해당 객체의 암묵적인 락을 사용해 캡슐화한 변수에 접근하는 모든 코드 경로를 동기화함 -> 여러 스레드가 동시 접근하는 상태에서 내부 변수를 보호 (ex. Vector)
  - 여러 변수에 대한 불변조건(invariant)가 있으면 해당 변수들은 모두 같은 락으로 보호해야 한다.
  - 여러 메소드가 하나의 복합 동작으로 묶일 땐 락을 사용해 추가로 동기화해야 한다.

##Chapter 3

**재배치**reordering : 프로그램 코드가 실행되는 순서가 임의로 바뀜.(컴파일러,JVM,프로세서에 의해)

**스테일**stale 데이터 : 최신 값이 아닌 값을 읽어갈 때

**Volatile 변수** : 가시성은 보장, 연산의 단일성은 보장 안됨(ex, count++)

**스레드 한정** : 특정 객체를 단일 스레드에서만 활용하도록 한정confine하여 스레드 안정성 확보 (ex, UI스레드)
	스택 한정 (로컬 변수), ThreadLocal

**불변 객체**immutable : 언제라도 스레드에 안전

  - 생성자 실행하는 도중 this 변수가 외부로 유출되지 않게 해야 한다. (ex, 생성자에서 스레드 생성 후 실행, 생성자에서 이벤트 리스너 생성 후 등록)
  - 안전한 공개 방법 - 올바르게 생성자가 실행되고 난 다음
    - 객체에 대한 참조를 static 메소드에서 초기화
    - 객체에 대한 참조를 volatile 변수 또는 AtomicReference 클래스에 보관
    - 객체에 대한 참조를 올바르게 생성된 클래스 내부의 final 변수에 보관
    - 객체에 대한 참조를 락을 사용해 올바르게 막혀 있는 변수에 보관

##Chapter 4

------

http://stackoverflow.com/questions/4844637/what-is-the-difference-between-concurrency-parallelism-and-asynchronous-methods

- Concurrent (병행의) - 어떤 동일한 시간 내에 두 개 이상의 동작이 발생하는 것을 뜻하는 용어.
- Concurrency - 동시 발생
- Parallel [pӕrəlel] - 병행의
- Parallelism [|pӕrəlelɪzəm] - 병행, (=병렬 계산)
- Simultaneously [sàiməltéiniəsli] - 동시에
- Synchronization - 동기화
- Asynchronous [eɪ|sɪŋkrənəs] - 비동기의,비동기,비동기식
- Asynchrony [eisíŋkrəni] - 동기화 되지 않은 상태

Q. 동시발생concurrency은 두 작업이 별개의 스레드에서 병행parallel으로 수행되게 하는 것이다. 하지만, asynchronous 메소드들은 동일한 하나의 스레드에서 병행으로 수행된다. 어떻게 이것이 이루어지나? 그리고 병행parallelism은 어떠한가?

이들 3가지 개념들 간의 차이점은 무엇인가?

A. 당신이 정확히 추측한대로 **Concurrent**와 **parallel**은 실질적으로 동일한 원리principle이다. 비록 나는 **parallel** 작업이 "동일한 시간"에 수행되는, 진정한 멀티테스킹이라 말하고 반면 **concurrent**는 여전히 **parallel**하게 수행되는 것처럼 보이지만, 그 작업들은 한 수행 스레드를 공유하고 있음을 의미한다.

**Asynchronous** 메소드들은 이전의 두 개념들과는 직접적으로 관련이 없으며, **asynchrony**는 **concurrent** 또는 **parallel** 작업의 효과를 내기 위해 사용된다. 하지만 실제적으로 **asynchronous** 메소드 호출은 현재의 응용 프로그램 밖에서 수행할 필요가 있는 process를 위해 사용되며 우리는 우리의 응용프로그램을 응답을 기다리도록 멈추고 기다리는 것을 원하지 않는다.

예를 들어, 데이터베이스에서 데이터를 가지고 오는 것은 시간이 걸리는데 우리는 데이터를 기다리는 동안 UI를 멈추는 것은 원치 않는다. **Asynch**(비동기) 호출은 콜백 레퍼런스를 가져가고 원격 시스템에 요청이 완료되자마자 당신의 코드에 실행을 다시 반환한다. 원격 시스템이 어떤 처리가 필요하든 간에 수행하는 동안 당신의 UI는 계속해서 사용자에 응답할 수 있고, 일단 그것이 데이터를 당신의 콜백 메소드에 반환하면  그 메소드는 적절히 UI를 업데이트할 수 있다(또는 그 업데이트를 밀쳐낸다)

사용자 측면에서 이는 멀티테스킹같아 보이지만 그렇지는 않을 것이다.

EDIT

It's probably worth adding that in many implementations an asynchronous method call will cause a thread to be spun up but it's not essential, it really depends on the operation being executed and how the response can be notified back to the system.



A. 간단히 말해

**Concurreny**는 중첩된overlapping 시간 동안, 특정한 순서없이, 시작하고, 동작하고, 완료되는 다수의 작업들을 의미한다. **Parallelism**은 ,예를 들어 멀티-코어 프로세서에서, 다수의 작업들 또는 유일한 작업의 여러 부분들이 문자 그대로 동시에 수행될 때이다.

> Concurrency와 parallelism는 동일한 것이 아님을 기억하라.

**concurrency와 parallelism간의 차이점**

**concurrency**와 **parallelism**간에 주목할만한 차이점을 나열해보면

**Concurrency** is when two tasks can start, run, and complete in overlapping time periods. **Parallelism** is when tasks literally run at the same time, eg. on a multi-core processor.

**Concurrency**은 process들을 독립적으로 수행하는 것의 구성이다. 반면 **parallelism**은 

 is the composition of independently executing processes, while parallelism is the simultaneous execution of (possibly related) computations.

Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once.



An application can be concurrent – but not parallel, which means that it processes more than one task at the same time, but no two tasks are executing at same time instant.

An application can be parallel – but not concurrent, which means that it processes multiple sub-tasks of a task in multi-core CPU at same time.

An application can be neither parallel – nor concurrent, which means that it processes all tasks one at a time, sequentially.

An application can be both parallel – and concurrent, which means that it processes multiple tasks concurrently in multi-core CPU at same time.

**Concurrency**

> Concurrency is essentially applicable when we talk about minimum two tasks or more. When an application is capable of executing two tasks virtually at same time, we call it concurrent application. Though here tasks run looks like simultaneously, but essentially they MAY not. They take advantage of CPU time-slicing feature of operating system where each task run part of its task and then go to waiting state. When first task is in waiting state, CPU is assigned to second task to complete it’s part of task.
>
> Operating system based on priority of tasks, thus, assigns CPU and other computing resources e.g. memory; turn by turn to all tasks and give them chance to complete. To end user, it seems that all tasks are running in parallel. This is called concurrency.

**Parallelism**

> Parallelism does not require two tasks to exist. It literally physically run parts of tasks OR multiple tasks, at the same time using multi-core infrastructure of CPU, by assigning one core to each task or sub-task.
>
> Parallelism requires hardware with multiple processing units, essentially. In single core CPU, you may get concurrency but NOT parallelism.

**Asynchronous methods**

> This is not related to Concurrency and parallelism, asynchrony is used to present the impression of concurrent or parallel tasking but effectively an asynchronous method call is normally used for a process that needs to do work away from the current application and we don't want to wait and block our application awaiting the response.
