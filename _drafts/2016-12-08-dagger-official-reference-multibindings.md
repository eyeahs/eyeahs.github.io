---
layout: post
category: blog
published: false
title: Dagger Official Reference - Multibindings
---
Dagger는 multibindings을 사용하여 여러 객체들을 심지어 다른 모듈에 바인딩된 경우에도 컬렉션 안에 바인딩할 수 있다. Dagger는 컬렉션을 모아서 애플리케이션 코드가 개별 바인딩에 직접 의존하지 않고도 주입될 수 있도록 한다.

예를 들어, Multibindings을 사용하여 여러 모듈이 개별 플러그인 인터페이스 구현을 제공하여 중앙의 클래스가 전체 플러그인 세트를 사용할 수 있는 플러그인 아키텍처를 구현할 수 있다. 또는 여러 모듈들을 개별 서비스 제공자provider에게 키가 name인 맵으로 제공할 수도 있다.

## Multibindings 설정

한 요소를 주입가능한 multibound 세트에 제공하기 위해서는 당신의 모듈에 [@IntoSet](https://google.github.io/dagger/api/latest/dagger/multibindings/IntoSet.html) 어노테이션을 추가하라.

    @Module
    class MyModuleA {
      @Provides @IntoSet
      static String provideOneString(DepA depA, DepB depB) {
        return "ABC";
      }
    }
    
또한 당신은 [@ElementsIntoSet](https://google.github.io/dagger/api/latest/dagger/multibindings/ElementsIntoSet.html)으로 어노테이트되고 subset을 반환하는 모듈 메소드를 추가하여 여러 항목들을 한 번에 제공할 수도 있다.

    @Module
    class MyModuleB {
      @Provides @ElementsIntoSet
      static Set<String> provideSomeStrings(DepA depA, DepB depB) {
        return new HashSet<String>(Arrays.asList("DEF", "GHI"));
      }
    }
    
이제 Component의 바인딩은 set에 의존할 수 있다:

    class Bar {
      @Inject Bar(Set<String> strings) {
        assert strings.contains("ABC");
        assert strings.contains("DEF");
        assert strings.contains("GHI");
      }
    }
    
또는 Component는 set을 제공할 수 있다:

    @Component(modules = {MyModuleA.class, MyModuleB.class})
    interface MyComponent {
      Set<String> strings();
    }

    @Test void testMyComponent() {
      MyComponent myComponent = DaggerMyComponent.create();
      assertThat(myComponent.strings()).containsExactly("ABC", "DEF", "GHI");
    
    
다른 바인딩과 마찬가지로, multibound Set<Foo>에 의존할 뿐만 아니라, Provider<Set<Foo>> 또는 Lazy<Set<Foo>>에도 의존할 수 있다. 하지만 <Set<Provider<Foo>>에는 의존할 수 없다.

Qualified multibound set을 제공하기 위해서는, 각 @Provides 메소드에 qualifier 어노테이션을 추가하라:

    @Module
    class MyModuleC {
      @Provides @IntoSet
      @MyQualifier
      static Foo provideOneFoo(DepA depA, DepB depB) {
        return new Foo(depA, depB);
      }
    }

    @Module
    class MyModuleD {
      @Provides
      static FooSetUser provideFooSetUser(@MyQualifier Set<Foo> foos) { … }
    }
    
## Map multibindings

컴파일 타임에 map의 키key를 알 수 있다면 multibindings를 사용하여 주입가능한 map에 항목entry를 제공할 수 있다.

Multibinding map에 항목entry를 제공하려면 @IntoMap 어노테이션과 항목entry을 위한 map의 키key값을 지정하는 커스텀 어노테이션을 가지고 값을 반환하는 메소드를 모듈에 추가하라. 각 Qualified multibound map에 항목을 제공하기 위해서는 @IntoMap 메소드에 Qualifier 어노테이션을 추가하라.

그 다음에는 map 자체(Map<K,V>) 또는 값 제공자value provider를 포함한 map(Map<K, Provider<V>)를 주입할 수 있다. 후자는 한 번에 하나의 값만을 추출하고자 하거나, 어쩌면 Map을 쿼리할 때 마다 각 값의 새로운 인스턴스를 가져오고자 할 때, 모든 값들이 인스턴스화 되는 것을 원하지 않으려는 경우에 유용한다.

## 간단한 Map keys

스트링, Class<?> 또는 박스형 primitive을 가진 맵의 경우, dagger.mapKeys의 표준 어노테이션 중 하나를 사용하라:

    @Module
    class MyModule {
      @Provides @IntoMap
      @StringKey("foo")
      static Long provideFooValue() {
        return 100L;
      }

      @Provides @IntoMap
      @ClassKey(Thing.class)
      static String provideThingValue() {
        return "value for Thing";
      }
    }

    @Component(modules = MyModule.class)
    interface MyComponent {
      Map<String, Long> longsByString();
      Map<Class<?>, String> stringsByClass();
    }

    @Test void testMyComponent() {
      MyComponent myComponent = DaggerMyComponent.create();
      assertThat(myComponent.longsByString().get("foo")).isEqualTo(100L);
      assertThat(myComponent.stringsByClass().get(Thing.class))
          .isEqualTo("value for Thing");
    }