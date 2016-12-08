---
layout: post
category: blog
published: false
title: Dagger Official Reference - Multibindings
---
Dagger는 multibindings을 사용하여 여러 객체들을 심지어 다른 모듈에 바인딩된 경우에도 컬렉션 안에 바인딩할 수 있다. Dagger는 컬렉션을 모아서 애플리케이션 코드가 개별 바인딩에 직접 의존하지 않고도 주입될 수 있도록 한다.

예를 들어, Multibindings을 사용하여 여러 모듈이 개별 플러그인 인터페이스 구현을 제공하여 중앙의 클래스가 전체 플러그인 세트를 사용할 수 있는 플러그인 아키텍처를 구현할 수 있다. 또는 여러 모듈들을 개별 서비스 제공자provider에게 Key가 name인 Map으로 제공할 수도 있다.

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
    
    
다른 바인딩과 마찬가지로, multibound Set<Foo>에 의존할 뿐만 아니라, `Provider<Set<Foo>>` 또는 `Lazy<Set<Foo>>`에도 의존할 수 있다. 하지만 `<Set<Provider<Foo>>`에는 의존할 수 없다.

Qualified multibound set을 제공하기 위해서는, 각 `@Provides` 메소드에 qualifier 어노테이션을 추가하라:

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

컴파일 타임에 map의 key를 알 수 있다면 multibindings를 사용하여 주입가능한 map에 entry를 제공할 수 있다.

Multibinding map에 entry를 제공하려면 [@IntoMap](https://google.github.io/dagger/api/latest/dagger/multibindings/IntoMap.html) 어노테이션과 entry을 위한 map의 key값을 지정하는 커스텀 어노테이션을 가지고 값을 반환하는 메소드를 모듈에 추가하라. 각 Qualified multibound map에 entry을 제공하기 위해서는 `@IntoMap` 메소드에 Qualifier 어노테이션을 추가하라.

그 다음에는 map 자체(`Map<K,V>`) 또는 값 제공자value provider를 포함한 map(`Map<K, Provider<V>`)를 주입할 수 있다. 후자는 한 번에 하나의 값만을 추출하고자 하거나, 어쩌면 Map을 쿼리할 때 마다 각 값의 새로운 인스턴스를 가져오고자 할 때, 모든 값들이 인스턴스화 되는 것을 원하지 않으려는 경우에 유용한다.

## 간단한 Map keys

스트링, `Class<?>` 또는 박스형 primitive을 가진 Map의 경우, dagger.mapKeys의 표준 어노테이션 중 하나를 사용하라:

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
    
Enum 또는 구체적으로 매개변수화된parameterized 클래스가 key인 map의 경우 타입이 Map의 key 타입인 멤버를 가진 어노테이션을 작성하고, [@MapKey](https://google.github.io/dagger/api/latest/dagger/MapKey.html) 어노테이션을 추가하라:

    enum MyEnum {
      ABC, DEF;
    }

    @MapKey
    @interface MyEnumKey {
      MyEnum value();
    }

    @MapKey
    @interface MyNumberClassKey {
      Class<? extends Number> value();
    }

    @Module
    class MyModule {
      @Provides @IntoMap
      @MyEnumKey(MyEnum.ABC)
      static String provideABCValue() {
        return "value for ABC";
      }

      @Provides @IntoMap
      @MyNumberClassKey(BigDecimal.class)
      static String provideBigDecimalValue() {
        return "value for BigDecimal";
      }
    }

    @Component(modules = MyModule.class)
    interface MyComponent {
      Map<MyEnum, String> myEnumStringMap();
      Map<Class<? extends Number>, String> stringsByNumberClass();
    }

    @Test void testMyComponent() {
      MyComponent myComponent = DaggerMyComponent.create();
      assertThat(myComponent.myEnumStringMap().get(MyEnum.ABC)).isEqualTo("value for ABC");
      assertThat(myComponent.stringsByNumberClass.get(BigDecimal.class))
          .isEqualTo("value for BigDecimal");
    }
    
당신의 어노테이션의 단일 멤버는 배열을 제외하면 모두 유요한 어노테이션 멤버가 될 수 있으며, 임의의 이름을 가질 수 있다.

## 복잡한 Map keys

Map의 key가 단일 어노테이션 멤버만으로 표현될 수 없다면, `@MapKey`의 `unwrapValue`를 `false`로 설정함으로서 전체 어노테이션을 map의 key로 사용할 수 있다. 이 경우, 어노테이션은 배열 구성원들도 가질 수 있다.

    @MapKey(unwrapValue = false)
    @interface MyKey {
      String name();
      Class<?> implementingClass();
      int[] thresholds();
    }

    @Module
    class MyModule {
      @Provides @IntoMap
      @MyKey(name = "abc", implementingClass = Abc.class, thresholds = {1, 5, 10})
      static String provideAbc1510Value() {
        return "foo";
      }
    }

    @Component(modules = MyModule.class)
    interface MyComponent {
      Map<MyKey, String> myKeyStringMap();
    }

### 어노테이션 인스턴드를 생성하기 위해 @AutoAnnotation을 사용하기.

Map이 복잡한 key를 사용하는 경우 런타임에 `@MapKey` 어노테이션의 인스턴스를 만들어 map의 `get(Object)` 메소드에 전달할 필요가 있을 수 있다. 이를 위한 가장 간단한 방법은 `@AutoAnnotation` 어노테이션을 사용하여 당신의 어노테이션을 인스턴스화하는 static 메소드를 만드는 것이다. 자세한 내용은 [@AutoAnnotation](https://github.com/google/auto/blob/master/value/src/main/java/com/google/auto/value/AutoAnnotation.java)의 문서를 참조하라.

    class MyComponentTest {
      @Test void testMyComponent() {
        MyComponent myComponent = DaggerMyComponent.create();
        assertThat(myComponent.myKeyStringMap()
            .get(createMyKey("abc", Abc.class, new int[] {1, 5, 10}))
            .isEqualTo("foo");
      }

      @AutoAnnotation
      static MyKey createMyKey(String name, Class<?> implementingClass, int[] thresholds) {
        return new AutoAnnotation_MyComponentTest_createMyKey(name, implementingClass, thresholds);
      }
    }
    
## 컴파일타임에 key를 알지 못하는 Map

multibinding은 Map의 key가 컴파일 타임에 알 수 있고 어노테이션으로 표현 될 수 있는 경우에만 동작한다. 만약 Map의 key가 이런 제약 조건에 맞지 않는 경우, multibound Map을 만들 수 없다. 하지만 multibound Map이 아니도록 변환할 수 있는 객체의 Set을 바인딩하기 위해 Set multibinding을 사용하면 이 문제를 해결할 수 있다.

    @Module
    class MyModule {
      @Provides @IntoSet
      static Map.Entry<Foo, Bar> entryOne(…) {
        Foo key = …;
        Bar value = …;
        return new SimpleImmutableEntry(key, value);
      }

      @Provides @IntoSet
      static Map.Entry<Foo, Bar> entryTwo(…) {
        Foo key = …;
        Bar value = …;
        return new SimpleImmutableEntry(key, value);
      }
    }

    @Module
    class MyMapModule {
      @Provides
      static Map<Foo, Bar> fooBarMap(Set<Map.Entry<Foo, Bar>> entries) {
        Map<Foo, Bar> fooBarMap = new LinkedHashMap<>(entries.size());
        for (Map.Entry<Foo, Bar> entry : entries) {
          fooBarMap.put(entry.getKey(), entry.getValue());
        }
        return fooBarMap;
      }
    }

이 방법은 `Map<Foo, Provider<Bar>>` 같은 자동화된 바인딩을 제공해주지 않는다. 만약 Provider의 map을 원한다면, multibound set안의 `Map.Entry` 객체가 provider를 포함해야 한다. 그러면 multibound map은 `Provider` 값을 가질 수 있다.

    @Module
    class MyModule {
      @Provides @IntoSet
      static Map.Entry<Foo, Provider<Bar>> entry(
          Provider<BarSubclass> barSubclassProvider) {
        Foo key = …;
        return new SimpleImmutableEntry(key, barSubclassProvider);
      }
    }

    @Module
    class MyProviderMapModule {
      @Provides
      static Map<Foo, Provider<Bar>> fooBarProviderMap(
          Set<Map.Entry<Foo, Provider<Bar>>> entries) {
        return …;
      }
    }

## Multibindings 선언하기

선언할 set이나 map을 반환하는 [@Multibindings](https://google.github.io/dagger/api/latest/dagger/multibindings/Multibinds.html)-어노테이션된 추상 메소드를 모듈에 추가하여 바인됭된 Multibindings set이나 map을 선언할 수 있다.

최소한 하나의 [@IntoSet](https://google.github.io/dagger/api/latest/dagger/multibindings/IntoSet.html), [@ElementsIntoSet](https://google.github.io/dagger/api/latest/dagger/multibindings/ElementsIntoSet.html) 또는 [@IntoMap](https://google.github.io/dagger/api/latest/dagger/multibindings/IntoMap.html) 바인딩을 가진 set이나 map에 대해 [@Multibinds](https://google.github.io/dagger/api/latest/dagger/multibindings/Multibinds.html)를 사용할 필요가 없지만, 비어있는 경우에는 선언을 해야 한다.

    @Module
    abstract class MyModule {
      @Multibinds abstract Set<Foo> aSet();
      @Multibinds @MyQualifier abstract Set<Foo> aQualifiedSet();
      @Multibinds abstract Map<String, Foo> aMap();
      @Multibinds @MyQualifier abstract Map<String, Foo> aQualifiedMap();
    }

주어진 set 또는 map multibinding은 오류없이 여러 번 선언 될 수 있다. Dagger는 절대로 @Multibinds 메소드를 구현하거나 호출하지 않는다.

## 대안 : 빈 set를 반환하는 @ElementsIntoSet

빈 set의 경우에만, 대안으로, 빈 set을 반환하는 @ElementsInfoSet 메소드를 추가할 수 있다.

    @Module
    class MyEmptySetModule {
      @Provides @ElementsIntoSet
      static Set<Foo> primeEmptyFooSet() {
        return Collections.emptySet();
      }
    }

## 상속된 subcomponent multibindings

