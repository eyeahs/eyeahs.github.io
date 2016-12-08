---
layout: post
category: blog
published: false
title: Dagger APIs Reference
---
# Annotation Type Binds

	@Documented
    @Retention(value=RUNTIME)
    @Target(value=METHOD)
    public @interface Binds

바인딩을 위임하는 [Module](https://google.github.io/dagger/api/latest/dagger/Module.html)의 abstract method를 어노테이트한다. 예를 들어, [Random](http://docs.oracle.com/javase/7/docs/api/java/util/Random.html?is-external=true)에 [SecureRandom](http://docs.oracle.com/javase/7/docs/api/java/security/SecureRandom.html?is-external=true)를 바인드하기 위해 module에 다음을 선언할 수 있다. `@Binds abstract Random bindRandom(SecureRandom secureRandom);`

@Binds method는 단순히 주입된 파라미터를 반환하는 [Provides](https://google.github.io/dagger/api/latest/dagger/Provides.html) 메소드의  편리한drop-in 대체품이다. 훨씬 더 효율적으로 구현이 생성되므로 @Binds를 사용하는 것이 더 좋다.

@Binds method는:

* abstract이여야 한다.
* Scope가 지정되어야 한다.
* Qualifier를 적용할 수 있다.
* 반환 타입에 타입을 지정할 수 있는 단일 파라미터가 있어야 한다. (@Provides와 마찬가지로) 반환 타입은 바운드 타입이다. 그리고 그 파라미터는 바운딩하고자 하는 타입이다.

멀티바인딩을 위해, 할당 가능assignability 여부가 다음과 비슷한 방식으로 검사된다.

##IntoSet
그 파라미터가 반환 타입의 멤버로 간주되면 [Set.add(E)](http://docs.oracle.com/javase/7/docs/api/java/util/Set.html?is-external=true#add-E-)의 유일한 파라미터에 할당 할 수 있어야 한다 -파라미터는 반환 타입에 할당될 수 있어야 한다.

##ElementsInfoSet
파라미터가 반환 타입의 멤버로 간주되면 [ElementsIntoSet](http://docs.oracle.com/javase/7/docs/api/java/util/Set.html?is-external=true#addAll-java.util.Collection-)의 유일한 파라미터에 할당 할 수 있어야 한다. -반환 타입이 `Set<E>`이면 파라미터는 `Collection<? extends E>`에 할당 가능해야 한다.

##IntoMap
파라미터가 V가 반환 타입에 바인딩되는 [Map](http://docs.oracle.com/javase/7/docs/api/java/util/Map.html?is-external=true)의 멤버로 간주되면 [Map.put(K, V)](http://docs.oracle.com/javase/7/docs/api/java/util/Map.html?is-external=true#put-K-V-)의 value 파라미터에 할당 할 수 있어야 한다. -파라미터는 반환 타입에 할당 가능해야 한다.

