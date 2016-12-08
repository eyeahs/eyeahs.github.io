---
layout: post
category: blog
published: false
title: '[Dagger Official Reference] Subcomponents'
---
Subcomponent는 부모 component의 객체 그래프를 상속하고 확장하는 component이다. 애플리케이션의 서로 다른 부분을 캡슐화하거나 component내에서 하나 이상의 scope를 사용하기 위해 subcomponent를 사용해 애플리케이션의 객체 그래프를 서브 그래프로 분리할 수 있다.

Subcomponent에 바인딩된 객체는 그것 자체의 모듈에 바인드된 객체뿐만 아니라 그것의 부모나 조상의 component에 바인딩된 객체에도 의존할 수 있다. 반면, 부모 component에 바인딩된 객체는 subcomponent에 바인딩된 객체에 의존할 수 없으며 한 subcomponent에 바인딩된 객체는 형제 subcomponet의 바인딩된 객체에 의존할 수 없다.

즉, Subcomponent의 부모 component의 객체 그래프는 subcomponent 자신의 객체 그래프의 서브그래프subgraph이다.

## Subcomponent 선언하기
최상위 component와 마찬가지로 [애플리케이션이 관심을 가지는 타입을 반환하는 추상 메소드를 선언](https://google.github.io/dagger/api/latest/dagger/Component.html#component-methods)하는 추상 클래스 또는 인터페이스를 작성하여 subcomponent를 만들 수 있다. Subcomponent를 [@Component](https://google.github.io/dagger/api/latest/dagger/Component.html)로 어노테이션하는 대신 [@Subcomponent](https://google.github.io/dagger/api/latest/dagger/Subcomponent.html)로 어노테이션하고 [@Module](https://google.github.io/dagger/api/latest/dagger/Module.html)들를 설치할 수 있다. [Component builder](https://google.github.io/dagger/api/latest/dagger/Component.Builder.html)와 비슷하게, [@Subcomponent.Builder](https://google.github.io/dagger/api/latest/dagger/Subcomponent.Builder.html)는 subcomponent를 생성할 때 필요한 module들을 제공하는 인터페이스를 지정한다.

    @Subcomponent(modules = RequestModule.class)
    interface RequestComponent {
      RequestHandler requestHandler();

      @Subcomponent.Builder
      interface Builder {
        Builder requestModule(RequestModule module);
        RequestComponent build();
      }
    }
    
## Subcomponent를 부모 Component에게 추가하기.
Subcomponent를 부모 component에 추가하려면, 부모 component에 설치할 `@Module`의 [subcomponents](https://google.github.io/dagger/api/latest/dagger/Module.html#subcomponents()) 속성에 subcomponent의 클래스를 추가하라. 그러면 부모 component에서 subcomponent의 빌더 클래스를 요청할 수 있다.

    @Module(subcomponents = RequestComponent.class)
    class ServerModule {}

    @Singleton
    @Component(modules = ServerModule.class)
    interface ServerComponent {
      RequestRouter requestRouter();
    }

    @Singleton
    class RequestRouter {
      @Inject RequestRouter(
          Provider<RequestComponent.Builder> requestComponentProvider) {}

      void dataReceived(Data data) {
        RequestComponent requestComponent =
            requestComponentProvider.get()
                .data(data)
                .build();
        requestComponent.requestHandler()
            .writeResponse(200, "hello, world");
      }
    }
    
## Subcomponent와 scope
애플리케이션의 component를 subcomponent로 분리하는 한 가지 이유는 [범위scope](http://docs.oracle.com/javaee/7/api/javax/inject/Scope.html)를 사용하기 위해서 이다. 일반적으로, scope가 지정되지 않은 바인딩들을 사용하면 주입된 타입의 사용자는 새로운, 개별 인스턴스를 얻게 된다. 그러나 만약 바인딩이 scope를 가지면 해당 scope의 생명 주기 내부의 해당 바인딩의 사용자는 바인딩된 타입의 동일한 인스턴스를 받게 된다.

표준 Scope는 [@Singleton](http://docs.oracle.com/javaee/7/api/javax/inject/Singleton.html)이다. singleton-scope의 사용자는 모두 동일한 인스턴스를 얻게 된다.

Dagger에서 Component는 [@Scope](http://docs.oracle.com/javaee/7/api/javax/inject/Scope.html) 어노테이션을 추가하여 그 scope와 연결할 수 있다. 이 경우 Component의 구현은 재사용될 수 있도록 모든 scope된 객체들의 참조들을 보유한다. 어노테이션된 스코프가 추가된 [@Provides](https://google.github.io/dagger/api/latest/dagger/Provides.html) 메소드를 가진 Module들은 동일한 scope로 어노테이션된 Component에만 설치 될 수 있다.

([@Inject](http://docs.oracle.com/javaee/7/api/javax/inject/Inject.html) 생성자를 가진 타입 역시 scope 어노테이션을 추가할 수 있다. 이런 "암시적 바인딩"은 해당 스코프를 가진 component 또는 그것의 자손 component에 의해 사용될 수 있다. 그 scope된 인스턴스는 올바른 scope에 바인드딩 될 것이다)

서로 도달 할 수 없는 두 개의 subcomponent는 scope가 지정된 객체를 저장할 위치에 대한 불명확성이 없으므로 동일한 scope와 연관될 수 있지만, Subcomponent는 조상 Component와 동일한 scope로 연결할 수 없다. (두 Subcomponent들은 동일한 scope 어노테이션을 사용하더라도 효과적으로 서로 다른 scope 인스턴스들를 가지게 된다.)

예를 들어 아래의 component 트리에서, `BadChildComponent`는 그것의 부모인 `RootComponent`와 동일한 `@RootScope`를 가지며 이것은 오류이다. 하지만 `SiblingComponentOne`과 `SiblingComponentTwo`는 다른 바인딩의 동일한 타입과 혼동을 일으킬 방법이 없기 때문에 모두 `@ChildScope`를 사용할 수 있다.

    @RootScope @Component
    interface RootComponent {
      BadChildComponent.Builder badChildComponent(); // ERROR!
      SiblingComponentOne.Builder siblingComponentOne();
      SiblingComponentTwo.Builder siblingComponentTwo();
    }

    @RootScope @Subcomponent
    interface BadChildComponent {…}

    @ChildScope @Subcomponent
    interface SiblingComponentOne {…}

    @ChildScope @Subcomponent
    interface SiblingComponentTwo {…}
    
Subcomponent는 그것의 부모 안에서 생성되므로, 그것의 수명은 부모의 수명보다 엄격하게 작아야 한다. 부모 component의 scope를 "더 큰 것"으로 그리고 subcomponent의 scope를 "더 작은 것"으로 간주하는 것이 이치에 맞음을 의미한다. 실제로 당신은 거의 언제나 루트 component가 [@Singleton](http://docs.oracle.com/javaee/7/api/javax/inject/Singleton.html) scope를 사용하기를 원한다.

아래 예제에서 `RootComponent`는 [@Singleton](http://docs.oracle.com/javaee/7/api/javax/inject/Singleton.html) scope에 있다. `@SessionScope`는 [@Singleton](http://docs.oracle.com/javaee/7/api/javax/inject/Singleton.html) scope에 중첩되어 있고, `@RequestScope`는 `@SessionScope`안에 중첩되어 있다. `FooRequestComponent`와 `BarRequestComponent`는 둘 다 `@RequestScope`와 연관되어 있음을 주목하라. 이는 서로가 누구의 조상도 아니며, 형제이기 때문에 가능한다.

    @Singleton @Component
    interface RootComponent {
      SessionComponent.Builder sessionComponent();
    }

    @SessionScope @Subcomponent
    interface SessionComponent {
      FooRequestComponent.Builder fooRequestComponent();
      BarRequestComponent.Builder barRequestComponent();
    }

    @RequestScope @Subcomponent
    interface FooRequestComponent {…}

    @RequestScope @Subcomponent
    interface BarRequestComponent {…}
    
## 캡슐화를 위한 하위 구성 요소
Subcomponent를 사용하는 또 다른 이유는 애플리케이션의 서로 다른 부분을 서로 캡슐화하기 위해서이다. 예를 들어, 서버의 두 서비스(또는 애플리케이션의 두 화면)이 동일한 바인딩을 공유하고, 
