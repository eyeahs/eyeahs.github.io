---
layout: post
category: blog
published: false
title: Untitled
---
어떤 어플리케이션에서 최고의 클래스들은 자신의 할 일을 하는 것들이다 : _BarcodeDecoder_, _KoopaPhysicsEngine_, 그리고 _AudioStreamer_. 이들 클래스들은 의존을 가지고 있다; 아마 _BarcodeCameraFinder_, _DefaultPhysicsEngine_, 그리고 _HttpStreamer_.

반대로, 어떤 어플리케이션에서 최악의 클래스들은 전혀 하는 일 없이 공간을 차지하는 것들이다 : _BarcodeDecodeFactory_, _CameraServiceLoader_, _MutableContextWrapper_. 이 클래스들은 관심의 대상들을 함께 묶어주는 투박한 덕트 테입이다.

Dagger는 boilerplate 작성 부담없이 [의존성 주입](https://en.wikipedia.org/wiki/Dependency_injection) 디자인 패턴을 구현하는 저 _FactoryFactory_ 클래스들의 대체품이다. Dagger는 흥미있는 클래스들에 초점을 맞출 수 있게 해준다. 의존 관계들을 선언하고, 그들을 어떻게 만족할지 명시한 뒤, 어플리케이션을 출하하라.

[javax.inject](http://docs.oracle.com/javaee/7/api/javax/inject/package-summary.html) 어노테이션들([JSR 330](https://jcp.org/en/jsr/detail?id=330))을 기반으로 하여, 각 **클래스는 테스트하기 쉽다**. 단지 _RpcCreditCardService을_ _FakeCreditCardService으로_ 교환하기 위한 boilerplate들은 필요하지 않는다.

의존성 주입은 테스트만을 위한 것이 아니다. 이는 **재사용 가능하고, 교체 가능한** 모듈을 만드는 것을 쉽게 해준다. 당신은 동일한 AuthenticationModule을 당신의 앱 전체에 공유할 수 있다. 그리고 당신은 각 상황에 적절한 행동을 얻기 위해서 개발중에는 _DevLoggingModule_을 실행하고 운영에서는 _ProdLoggingModule_을 실행할 수 있다.

# 왜 Dagger 2는 다른가?

[의존성 주입](https://en.wikipedia.org/wiki/Dependency_injection) 프레임워크는 주입(injecting)과 설정(configuring)을 위한 온갖 종류의 다양한 API들과 함께 몇 년 동안 존재하고 있다. 그런대, 왜 바퀴를 재발명하는가? Dagger 2은 처음으로 **생성된 코드(generated code)로 전체 스택(full stack)을 구현한다**. 확실히 의존성 주입이 가능한 간단하고, 추적 가능하고(traceable), 성능 기준에 맞도록(performant) 사용자가 손으로 작성했을 코드를 흉내내어 생성하는 것이 원칙이다. 더 많은 디자인에 대한 배경은 [+Gregory Kick](https://plus.google.com/+GregoryKick/posts)의 [강연]((https://www.youtube.com/watch?v=oK_XtfXPkqw&feature=youtu.be))([슬라이드](https://docs.google.com/presentation/d/1fby5VeGU9CN8zjw4lAb2QPPsKRxx6mSwCe9q7ECNSJQ/pub?start=false&loop=false&delayms=3000&slide=id.p))을 보라.

# Dagger 사용하기

우리는 커피 메이커를 만들면서 의존성 주입과 Dagger의 작동 과정을 보여줄 것이다. 컴파일하고 실행할 수 있는 완전한 샘플 코드는 Dagger의 [커피 예제](https://github.com/google/dagger/tree/master/examples/simple/src/main/java/coffee)를 보라.

## 의존을 선언하기.

Dagger는 당신의 어플리케이션 클래스들의 인스턴스들을 구성하고 그들의 의존을 만족시킨다. Dagger는 자신이 관심을 가질 필드들과 생성자들을 확인하기 위해 javax.inject.Inject 어노테이션을 사용한다.

Dagger가 클래스의 인스턴스를 생성할 때 사용해야 할 생성자에 @Inject 어노테이션을 사용한다. 새로운 인스턴스가 요구될 때 Dagger는 필요한 파라미터를 획득하고 이 생성자를 불러낼 것이다.

    class Thermosiphon implements Pump {
      private final Heater heater;

      @Inject
      Thermosiphon(Heater heater) {
        this.heater = heater;
      }

      ...
    }
  
Dagger는 필드에 직접 주입할 수 있다. 이 예제에서 Dagger는 heater 필드를 위해 Heater 인스턴스를, pump 필드를 위해 Pump 인스턴스를 획득한다.

    class CoffeeMaker {
      @Inject Heater heater;
      @Inject Pump pump;

      ...
    }
  
만약 당신의 클래스가 @Inject 어노테이션된 필드를 가지고 있고 @Inject 어노테이션된 생성자가 없다면 Dagger는 요청을 받았을 때 이 필드들을 주입해줄 것이지만 새로운 인스턴스를 생성해주지는 않을 것이다. Dagger가 인스턴스도 생성해야 하는 것을 알려주기 위해 @Inject 어노테이션이 된 인수가 없는 생성자를 추가하라.

생성자나 필드 주입이 보통 더 선호되지만 Dagger는 method 주입도 지원한다.

@Inject 어노테이션이 누락된 클래스는 Dagger에 의해 생성될 수 없다.

## 의존들을 만족시키기

By default, Dagger satisfies each dependency by constructing an instance of the requested type as described above. When you request a CoffeeMaker, it’ll obtain one by calling new CoffeeMaker() and setting its injectable fields.

But @Inject doesn’t work everywhere:

* Interfaces can’t be constructed.
* Third-party classes can’t be annotated.
* Configurable objects must be configured!
* For these cases where @Inject is insufficient or awkward, use an [@Provides](http://google.github.io/dagger/api/latest/dagger/Provides.html)-annotated method to satisfy a * dependency. The method’s return type defines which dependency it satisfies.

For example, provideHeater() is invoked whenever a Heater is required:

    @Provides static Heater provideHeater() {
      return new ElectricHeater();
    }

It’s possible for @Provides methods to have dependencies of their own. This one returns a Thermosiphon whenever a Pump is required:

    @Provides static Pump providePump(Thermosiphon pump) {
      return pump;
    }
  
All @Provides methods must belong to a module. These are just classes that have an [@Module](http://google.github.io/dagger/api/latest/dagger/Module.html) annotation.

    @Module
    class DripCoffeeModule {
      @Provides static Heater provideHeater() {
        return new ElectricHeater();
      }

      @Provides static Pump providePump(Thermosiphon pump) {
        return pump;
      }
    }

By convention, @Provides methods are named with a provide prefix and module classes are named with a Module suffix.

## 그래프 만들기(Building the Graph)

_@Inject_와 _@Provides_어노테이션된 클래스들은 그들의 의존들로 연결된 객체들의 그래프를 형성한다.
The @Inject and @Provides-annotated classes form a graph of objects, linked by their dependencies. Calling code like an application’s main method or an Android Application accesses that graph via a well-defined set of roots. In Dagger 2, that set is defined by an interface with methods that have no arguments and return the desired type. By applying the @Component annotation to such an interface and passing the module types to the modules parameter, Dagger 2 then fully generates an implementation of that contract.

    @Component(modules = DripCoffeeModule.class)
    interface CoffeeShop {
      CoffeeMaker maker();
    }
  
The implementation has the same name as the interface prefixed with Dagger. Obtain an instance by invoking the builder() method on that implementation and use the returned builder to set dependencies and build() a new instance.

    CoffeeShop coffeeShop = DaggerCoffeeShop.builder()
        .dripCoffeeModule(new DripCoffeeModule())
        .build();

Note: If your @Component is not a top-level type, the generated component’s name will be include its enclosing types’ names, joined with an underscore. For example, this code:

    class Foo {
      static class Bar {
        @Component
        interface BazComponent {}
      }
    }

would generate a component named DaggerFoo_Bar_BazComponent.

Any module with an accessible default constructor can be elided as the builder will construct an instance automatically if none is set. And for any module whose @Provides methods are all static, the implementation doesn’t need an instance at all. If all dependencies can be constructed without the user creating a dependency instance, then the generated implementation will also have a create() method that can be used to get a new instance without having to deal with the builder.

	CoffeeShop coffeeShop = DaggerCoffeeShop.create();

Now, our CoffeeApp can simply use the Dagger-generated implementation of CoffeeShop to get a fully-injected CoffeeMaker.

    public class CoffeeApp {
      public static void main(String[] args) {
        CoffeeShop coffeeShop = DaggerCoffeeShop.create();
        coffeeShop.maker().brew();
      }
    }

Now that the graph is constructed and the entry point is injected, we run our coffee maker app. Fun.

    $ java -cp ... coffee.CoffeeApp
    ~ ~ ~ heating ~ ~ ~
    => => pumping => =>
     [_]P coffee! [_]P

## Bindings in the graph

The example above shows how to construct a component with some of the more typcial bindings, but there are a variety of mechanisms for contributing bindings to the graph. The following are available as dependencies and may be used to generate a well-formed component:

* Those declared by @Provides methods within a @Module referenced directly by @Component.modules or transitively via @Module.includes
* Any type with an @Inject constructor that is unscoped or has a @Scope annotation that matches one of the component’s scopes
* The component provision methods of the component dependencies
* The component itself
* Unqualified builders for any included subcomponent
* Provider or Lazy wrappers for any of the above bindings
* A Provider of a Lazy of any of the above bindings (e.g., Provider<Lazy<CoffeeMaker>>)
* A MembersInjector for any type

## Singletons and Scoped Bindings

Annotate an @Provides method or injectable class with @Singleton. The graph will use a single instance of the value for all of its clients.

  @Provides @Singleton static Heater provideHeater() {
    return new ElectricHeater();
  }

The @Singleton annotation on an injectable class also serves as documentation. It reminds potential maintainers that this class may be shared by multiple threads.

  @Singleton
  class CoffeeMaker {
    ...
  }

Since Dagger 2 associates scoped instances in the graph with instances of component implementations, the components themselves need to declare which scope they intend to represent. For example, it wouldn’t make any sense to have a @Singleton binding and a @RequestScoped binding in the same component because those scopes have different lifecycles and thus must live in components with different lifecycles. To declare that a component is associated with a given scope, simply apply the scope annotation to the component interface.

  @Component(modules = DripCoffeeModule.class)
  @Singleton
  interface CoffeeShop {
    CoffeeMaker maker();
  }

Components may have multiple scope annotations applied. This declares that they are all aliases to the same scope, and so that component may include scoped bindings with any of the scopes it declares.

## Reusable scope

Sometimes you want to limit the number of times an @Inject-constructed class is instantiated or a @Provides method is called, but you don’t need to guarantee that the exact same instance is used during the lifetime of any particular component or subcomponent. This can be useful in environments such as Android, where allocations can be expensive.

For these bindings, you can apply @Reusable scope. @Reusable-scoped bindings, unlike other scopes, are not associated with any single component; instead, each component that actually uses the binding will cache the returned or instantiated object.

That means that if you install a module with a @Reusable binding in a component, but only a subcomponent actually uses the binding, then only that subcomponent will cache the binding’s object. If two subcomponents that do not share an ancestor each use the binding, each of them will cache its own object. If a component’s ancestor has already cached the object, the subcomponent will reuse it.

There is no guarantee that the component will call the binding only once, so applying @Reusable to bindings that return mutable objects, or objects where it’s important to refer to the same instance, is dangerous. It’s safe to use @Reusable for immutable objects that you would leave unscoped if you didn’t care how many times they were allocated.

  @Reusable // It doesn't matter how many scoopers we use, but don't waste them.
  class CoffeeScooper {
    @Inject CoffeeScooper() {}
  }

  @Module
  class CashRegisterModule {
    @Provides
    @Reusable // DON'T DO THIS! You do care which register you put your cash in.
              // Use a specific scope instead.
    static CashRegister badIdeaCashRegister() {
      return new CashRegister();
    }
  }


  @Reusable // DON'T DO THIS! You really do want a new filter each time, so this
            // should be unscoped.
  class CoffeeFilter {
    @Inject CoffeeFilter() {}
  }
