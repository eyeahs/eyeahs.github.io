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