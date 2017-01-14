---
layout: post
category: blog
title: "7가지 Singleton"
date: "2016-06-02 17:00:00 +0900"
categories: designpatter
published: true
splash: ""
tags: "designpattern"
comments: true
---

# 1. 기본적인 Singleton
    public class Singleton {
      private static Singleton instance;
      private Singleton() {}
      public static Singleton getInstance() {
        if (instance == null) {
          instance = new Singleton(); 
        }
      return instance; 
      }
    }

위 코드는 싱글톤의 가장 기본적인 형태이다. 위 코드의 문제는 두 개 이상의 쓰레드가 동시에 접근하는 경우 정상적인 동작(유일한 instance를 반환함)을 보장 할 수 없다는 점이다.

# 2. Synchronized Singleton
    public class Singleton {
      private static Singleton instance;
      private Singleton() {}
      public static synchronized Singleton getInstance() {
        if (instance == null) {
          instance = new Singleton(); 
        }
        return instance; 
      }
    }
    
위 코드는 동시성 문제를 해결하기 위해 getInstance() 메소드에 synchronized를 선언하였다.
하지만 위 코드에 여러 쓰레드가 동시에 접근할 경우 한 번에 한 쓰레드만 getInstance()메소드를 사용할 수 있으므로 (다른 쓰레드들은 먼저 접근한 쓰레드가 작업을 끝낼 때 까지 기다려야 한다.) 오버헤드가 증가한다.

# 3. Eager initialization
    public class Singleton {
      private static Singleton instance = new Singleton();
      private Singleton() {} 
      public static Singleton getInstance() {
        return instance;
      }
    }

인스턴스가 필요할 때 초기화 (Lazy initialization)하지 않고 프로그램이 처음 구동될 때 인스턴스 초기화까지 끝내 버릴 수도 있다. 초기화 비용이 비싸지 않는다면 그냥 이 방법을 사용하면 된다. 
가장 흔한 방법이며 구현도 편하고 보는 사람도 편하다.

# 4. Enum
    public enum Singleton {
      INSTANCE;

      public void execute (String arg) { //... perform operation here ... }
    }

자바 1.5 이후 버전에서 싱글톤을 구현하는 가장 좋은 방법은 하나의 요소를 갖는 enum 타입을 만드는 것이다. 이 방법은 복잡한 직렬화나 리플렉션 상황에서도 직렬화가 자동으로 지원되고, 인스턴스가 여러 개 생기지 않도록 확실하게 보장한 다는 것이다. public 메소드는 어떠한 타입의 인수(argument)들이라도 가질 수 있다.
이 방법은 JAVA는 enum 변수를 프로그램내에서 단 한 번만 초기화하는 것을 보장한다는 점을 통해 싱글톤을 구현한 것이다. 자바 enum 변수들은 전역 접근이 가능하므로 싱글톤이며 classloader에 의해 lazy 초기화된다. 결점은 enum type이 다소 유연하지 못하다는 점이다.

# 5. Initialization-on-demand holder idiom
	public class Something {
      private Something() {} 
      private static class LazyHolder { 
        private static final Something INSTANCE = new Something();
      }
      public static Something getInstance() { return LazyHolder.INSTANCE; }
    } 
    
이 구현은 좋은 성능을 가지면서 자바의 모든 버전에서 동기화 문제를 가지지도 않는다. 이 구현은 JVM 의 잘 정의된 초기화 실행 과정에 의지한다. (상세 내용은 Java Language Specification (JLS) 섹션 12.4를 참고하라)
클래스 Something이 JVM에 로드 될 때 클래스 초기화 단계가 시작된다. 이 클래스는 초기화할 static 변수를 가지고 있지 않기 때문에 클래스 초기화는 평범하게 끝나다. LazyHolder는 JVM이 LazyHolder가 실행되어야 한다고 판단하기 전까지는 초기화 되지 않는다. Statc class LazyHolder는 Something 클래스의 static method인 getInstance 메소드가 호출될 때 실행된다. 그리고 처음 실행될 때 JVM은 LazyHolder class를 로드하고 초기화한다. LazyHolder 클래스는 static 변수 INSTANCE가 outer class Something의 private 생성자에 의해서 초기화되는 것에 의해 초기화 된다.
클래스 초기화 단계가 JLS에 의해 보장되기 때문에 순차적이다. 즉 non-concurrent, static getInstance 메소드가 로드되고 초기화 될 때 추가적인 동기화 처리가 필요하지 않는다. 그리고 초기화 단계에서 순차적인 처리로 statice 변수 INSTANCE를 쓰기 때문에 getInstance는 이후의 모든 동시적인 호출에서 추가적인 동기화 오버헤드가 초래되지 않고 정확하게 동일하게 초기화된 INSTANCE를 반환한다.

## When to use it
만약 클래스의 초기화가 비싸거나 클래스 로딩 타임에 초기화되는 상황에서는 안전하지 못할 때 이 패턴을 사용한다. 패턴의 가장 어려운 부분은 싱글톤 인스턴스 접근과 관련된 동기화 오버헤드를 안전하게 제거하는 것이다.

## When not to use it
만약 INSTANCE의 생성자가 실패할 수 있다면 이 용법은 피해야 한다. 만약 INSTANCE의 생성자가 실패한다면 Something.getInstance()의 호출은 java.lang.ExceptionInInitializerError 에러가 발생하게 될 것이다.
이러한 생성자 초기화 실패를 처리하는 것은 싱글톤 패턴이 전반적으로 흔히 비난받는 부분이다.

# 6. DCL
자바 1.4 이전 버전에서는 멀티쓰레드 환경에서 비정상 동작을 할 수 있다.

# 7. Volatile Synchronized Singleton
public class Singleton {
  private volatile static Singleton INSTANCE;

  private Singleton() {}
  public static Singleton getInstance() { 
    if (INSTANCE == null) { (1)
      synchronized(Singleton.class) { 
        if (INSTANCE == null) { (2)
          INSTANCE = new Singleton();
        }
    }
    return instance;
  }
}

메소드 전체가 Synchronized로 선언되면 여러 스레드가 동시에 위 메소드에 접근한 경우 단 하나의 스레드만 해당 메소드에 진입할 수 있고 다른 스레드들은 대기하게 되어 성능 저하가 일어난다. 
위 코드이 경우 먼저 인스턴스 여부를 체크 하고, 인스턴스가 생성되지 않은 경우 Synchronized 구간에서 인스턴스를 생성한다. 이 때 INSTANCE static 변수의 원자성을 보장하기 위해 volatile를 사용한다.
