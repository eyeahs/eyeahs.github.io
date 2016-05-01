---
published: false
---
어떤 어플리케이션에서 최고의 클래스들은 자신의 할 일을 하는 것들이다 : _BarcodeDecoder_, _KoopaPhysicsEngine_, 그리고 _AudioStreamer_. 이들 클래스들은 의존을 가지고 있다; 아마 _BarcodeCameraFinder_, _DefaultPhysicsEngine_, 그리고 _HttpStreamer_.

반대로, 어떤 어플리케이션에서 최악의 클래스들은 전혀 하는 일 없이 공간을 차지하는 것들이다 : _BarcodeDecodeFactory_, _CameraServiceLoader_, _MutableContextWrapper_. 이 클래스들은 관심의 대상들을 함께 묶어주는 투박한 덕트 테입이다.

Dagger는 boilerplate 작성 부담없이 [의존성 주입](https://en.wikipedia.org/wiki/Dependency_injection) 디자인 패턴을 구현하는 저 _FactoryFactory_ 클래스들의 대체품이다. 이것은 당신이 관심 대상이 되는 클래스들에 초점을 맞출 수 있게 해준다. 의존 관계들을 선언하고, 그들을 어떻게 만족할지 명시하고, 당신의 어플리케이션을 출시하라.

[javax.inject](http://docs.oracle.com/javaee/7/api/javax/inject/package-summary.html) 어노테이션들([JSR 330](https://jcp.org/en/jsr/detail?id=330))을 기반으로 하여, 각 **클래스는 테스트하기 쉽다**. 단지 _RpcCreditCardService을_ _FakeCreditCardService으로_ 교환하기 위한 한 뭉치의 boilerplate는 필요하지 않는다.

의존성 주입은 테스트만을 위한 것이 아니다. 이는 **재사용 가능하고, 교체 가능한** 모듈을 만드는 것을 쉽게 해준다. 당신은 동일한 AuthenticationModule을 당신의 앱 전체에 공유할 수 있다. 그리고 당신은 각 상황에 옳은 행동을 얻기 위해서 개발중에는 _DevLoggingModule_을 실행하고 운영에서는 _ProdLoggingModule_을 실행할 수 있다.

# 왜 Dagger 2는 다른가?

[의존성 주입](https://en.wikipedia.org/wiki/Dependency_injection) 프레임워크는 주입(injecting)과 설정(configuring)을 위한 온갖 종류의 다양한 API들과 함께 몇 년 동안 존재하고 있다. 그런대, 왜 바퀴를 재발명하는가? Dagger 2은 처음으로 **생성된 코드(generated code)로 전체 스택(full stack)을 구현한다**. 확실히 의존성 주입이 가능한 간단하고, 추적 가능하고(traceable), 성능 기준에 맞도록(performant) 사용자가 손으로 작성했을 코드를 흉내내어 생성하는 것이 원칙이다. 더 많은 디자인에 대한 배경은 [+Gregory Kick](https://plus.google.com/+GregoryKick/posts)의 [강연]((https://www.youtube.com/watch?v=oK_XtfXPkqw&feature=youtu.be))([슬라이드](https://docs.google.com/presentation/d/1fby5VeGU9CN8zjw4lAb2QPPsKRxx6mSwCe9q7ECNSJQ/pub?start=false&loop=false&delayms=3000&slide=id.p))을 보라.

# Dagger 사용하기

우리는 커피 메이커를 만들면서 의존성 주입과 Dagger의 작동 과정을 보여줄 것이다. 컴파일하고 실행할 수 있는 완전한 샘플 코드는 Dagger의 [커피 예제](https://github.com/google/dagger/tree/master/examples/simple/src/main/java/coffee)를 보라.

## 의존을 선언하기.

Dagger constructs instances of your application classes and satisfies their dependencies. It uses the javax.inject.Inject annotation to identify which constructors and fields it is interested in.

Use @Inject to annotate the constructor that Dagger should use to create instances of a class. When a new instance is requested, Dagger will obtain the required parameters values and invoke this constructor.

  class Thermosiphon implements Pump {
    private final Heater heater;

    @Inject
    Thermosiphon(Heater heater) {
      this.heater = heater;
    }

    ...
  }
Dagger can inject fields directly. In this example it obtains a Heater instance for the heater field and a Pump instance for the pump field.

  class CoffeeMaker {
    @Inject Heater heater;
    @Inject Pump pump;

    ...
  }
If your class has @Inject-annotated fields but no @Inject-annotated constructor, Dagger will inject those fields if requested, but will not create new instances. Add a no-argument constructor with the @Inject annotation to indicate that Dagger may create instances as well.

Dagger also supports method injection, though constructor or field injection are typically preferred.

Classes that lack @Inject annotations cannot be constructed by Dagger.

