---
layout: post
category: blog
date: "2017-1-16 19:00:00 +0900"
published: true
title: "[번역]Clean tests"
comments: true
---

# Clean tests

# Part 1: Naming

[원본](https://android.jlelse.eu/clean-tests-part-1-naming-cce94edf0522#.79akj9apb)

Droidcon Berlin 2016에서 나는 더 클린한 단위 테스트를 작성하는 방법에 대한 [발표](http://www.slideshare.net/dpreussler/15-tips-to-improve-your-unit-tests-droidcon-berlin-2016-barcamp)를 했다.

몇 가지 제안에 대해 더 자세히 설명하고 싶다.

일반적인 단위 테스트를 살펴보자. 다음은 Java와 Android용 경량급 의존성 라이브러리인 [Toothpick](https://github.com/stephanenicolas/toothpick)에서 가져왔다.

```java
@Test
public void testGetParentScope_shouldReturnRootScope_whenAskedForSingleton() {
  //GIVEN
  Scope parentScope = Toothpick.openScope(“root”);
  Scope childScope = Toothpick.openScopes(“root”, “child”);
  //WHEN
  Scope scope = childScope.getParentScope(Singleton.class);
  //THEN
  assertThat(scope, is(parentScope));
}
```

이 코드 조각에서 내 첫 번째 요청은:

### 테스트 메소드를 "test"로 시작하지 마라

사람들은 왜 이렇게 하는 것일까? 누군가에게는 이는 JUnit3의 일률적 습관이다. 이는 대략 10년 전이였다. 당시에는 이것이 JUnit이 테스트 메소드를 찾기 위한 유일한 방법이였다. JUnit4부터 이를 위한 어노테이션이 존재한다:'@Test'

누군가는 테스트 메소드를 클래스안의 다른 메소드들과 구분하기 위해 이렇게 한다. 하지만 테스트 클래스 안에 다른 **public** 메소드가 있어서는 **안된다**! 그래서 이는 설명(또는 변명)이 되지 않는다.

내가 메소드 이름을 'test'로 시작하지 말라고 강조하는 이유는 간단하다. 이는 메소드 이름을 만드는 것을 훨씬 어렵게 만들 것이다. 이 고정 된 접두어는 당신을 제한한다.

좋은 이름 말하기:

### 테스트하는 메소드의 이름을 테스트에 넣지 마라!

단위 테스트의 초기 시절에는 테스트 대상이 되는 클래스의 모든 메소드에 대해 단일 테스트 메소드를 가지는 방법을 사용했다.

따라서 `getParentScope()`라는 메소드를 테스트 할 때는 테스트 메서드 이름을 `testGetParentScope()`로 지정했다.

하지만 곧 우리는 메소드 하나당 하나 이상의 테스트를 가질 필요가 있다는 것을 알게 되었다. 메소드는 한 가지 일만 하고 하나의 테스트는 단 하나만을 테스트하여야 하는 [클린 코드](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)의 원칙을 따를 때 특히 그렇다.

왜 메소드 이름을 실제로 유지하려 하는가?

나는 이것이 우리의 테스트 메소드의 **가독성을 해친다**고 생각한다. 테스트 메소드는 우리에게 자신이 하는 것과 무엇을 테스트하는지를 알려주어야 하지만 테스트를 할 때 어떤 메소드를 호출하는지는 아니다.

독일사람들은 이렇게 말한다: "Namen sall Schall und Rauch", 이름은 소리와 연기일 뿐이다. 이름은 오래가는 것이 아니다. 그리고 그것은 우리의 코드의 이름이다. 우리는 코드를 지속적으로 개선하므로 언제든지 그 이름을 리팩토링 할 수 있다. 그러나 우리가 그렇게 할 때 누가 테스트 이름을 업데이트해야 하는 것을 기억할까?

이는 메소드를 위한 JavaDoc과 비슷하다: 우리 생각보다 훨씬 빠르게 구닥다리가 될 것이다.

하지만 우리가 테스트하고자 하는 **행위**_behaviour_은 지속된다. 이제 테스트의 이름에 집중해보자!

이것 대신:

```` java
testGetParentScope_shouldReturnRootScope_whenAskedForSingleton
````

이렇게 하라:

```java
shouldReturnRootScope_whenAskedForSingleton()
```

나에겐 여전히 읽기 어려우므로, 나 개인적으로는 다음 스타일을 선호한다.

```java
should_return_root_scope_when_asked_for_singleton()
```

훨씬 나아 보이지 않는가? 이는 인간의 언어처럼 읽히며 생략된 Technobabble처럼 보이지 않는다.

그런데, 이는 당신의 이름을 생략하지 않는 또 다른 이유이다. 이것이 이름에 관한 전부이다!

### 테스트는 사양_specification_이다

Underscore들에 대한 나의 제안은 당신에게는 조금 이상하게 보일지도 모른다. 나에게도 이는 평범한 코드베이스에 있는 이름이 아니며 대부분의 사람들처럼 camel case를 사용한다. 그러나 테스트에는 약간 다른 요구사항들이 있다. 우리는 행위를 정의하고 있으므로 여기에 이것들을 문서처럼 작성한다. 우리의 테스트는 사양이다. 항상 이를 기억하라.

추가로: 우리의 테스트가 중단되면 그 이름을 어디에서 처음으로 보게 될까? CI 시스템의 리포트 페이지 또는 이메일 알림에서 존재할 가능성이 크다

이를 염두에 둘 때 더 좋은 이름이 된다:

```
Should return root scope when asked for singleton()
```

슬프게도 함수의 이름을 이렇게 붙일 수는 없다.

최소한 Java에서는. 다른 언어에서는 이것이 실제로 유효한 이름이다:

```
// Groovy
def “ Should return root scope when asked for singleton”()
// Kotlin
fun ` Should return root scope when asked for singleton`()
```

이는 읽을 수 없는 어셈블러 코드와는 멀리 떨어져있지만 여전히 당신은 미래를 명백히 볼 수 있다.

당신의 테스트는 일급 시민이므로 좋은 이름을 찾아야 한다. 그리고 이것은 사양 문서이므로 더 명확하고 읽기 쉬운 이름이 되도록 더 많은 시간을 쓰도록 하라.

In the next part we’ll continue here by looking into the actual test code itself. Stay tuned and name tests better!

# Part 2: Comments

[원본](https://android.jlelse.eu/clean-tests-part-2-comments-4016ee82f186#.ptxo6595u)

첫 번째 파트에서는 테스트의 이름을 살펴보았다. 나는 당신의 테스트를 test라는 단어로 시작하지 말며 호출하는 메소드 이름을 이름의 일부로 사용하지 말것은 권했다. 테스트는 당신의 사양이며 이는 가능한 사람이 읽을 수 있도록 작성해야 함을 의미한다. 하지만 이제 이름은 잊고 comment에 대하 논해보자.

### Given-When-Then

 [Toothpick](https://github.com/stephanenicolas/toothpick) 프로젝트에서 가져온 예제로 다시 구현을 체크해보자:

```java
//GIVEN
Scope parentScope = Toothpick.openScope(“root”);
Scope childScope = Toothpick.openScopes(“root”, “child”);
//WHEN
Scope scope = childScope.getParentScope(Singleton.class);
//THEN
assertThat(scope, is(parentScope));
```

Arrange-Act-Assert라고도 하는 이 Given-When-Then은 매우 일반적인 패턴이다. 하지만 이것을 볼 때마다 내 척추가 산산조각난다.

왜? 주석을 달아서 코드를 설명할 때 마다, 멈추고 심호흡하고 한 일에 대해 생각해야 한다.

테스트 코드가 아닌 경우에는 많은 독자들이 여기에 동의하겠지만 이 규칙은 테스트에서도 적용된다.

### 소음 줄이기

나를 더 나쁘게 하는 것은 모든 단일 테스트에 대해 이 주석 패턴이 반복된다는 것이다! 세 줄짜리 주석이 모든 단일 테스트에 새로운 라인들을 더한다!? 우리는 우리가 작성해야 하는 순수한 코드만을 갈구하고자 해야 한다. 당신은 이를 템플릿으로 사용하겠지만 이는 여전히 같은 추악한 결과이다.

여기에 동의하지 않는 경우라도 잠시 기다려라. 방금 말한 것을 잊고 이상적인 테스트를 생각해보자.

### 이상적 테스트

내가 생각하는 이상적 테스트는 **한 줄**이다!

이와 같은 것이다:

```java
assertTrue(tested.isTrue());
```

앞아서 봤던 다음 패턴은 두 라인 + 모든 주석들(공백 포함)로 나눌 필요가 있다:

```java
//GIVEN

//WHEN
boolean result = tested.isTrue();
//THEN
assertTrue(result);
```

이것이 진심으로 좋아보이는가? 전혀 그렇지 않다. 여기에는 너무 많은 노이즈가 있다.

당신은 빈 블럭을 제거하고 싶을 것이다. 하지만 이는 엄격한 순서를 가진 전체 아이디어를 파괴한다. 당신은 당신의 규칙을 사보타주하고 있다.

이 예제가 약간 변변찮은 것을 인정한다. 다른 예제를 보자:

```java
ImportantThing result = tested.getTheThing()
assertTrue(result.isTrue())
```

이 테스트에 여러 줄의 주석을 추가하면 더 읽기 쉬워질까?

나는 그렇게 생각하지 않는다. 이것은 상당히 잘 읽힐 수 있다!

이제 당신은 대부분의 테스트는 이보다 복잡하다고 말할 것이다.

그렇다!

사실 나는 다시 한 번 여기에 동의하지 않는다. 작은 테스트는 많은 테스트 베이스를 기반으로 해야 한다: Simple: 두 줄!

### 큰 테스트 악취

무엇이 당신의 테스트에서 코드를 세 개의 블록으로 나눌 필요가 있게 하는가? 이는 당신의 테스트가 너무 복잡함을 알린다!

중간(act or when) 부분은 한 메소드만을 호출하는 한 줄이여야 한다.

assertino 또는 then 역시 한 줄이여야 한다. 만약 더 많은 assertion이 있다면 이를 하나의 메소드로 추출해야 한다.

이제 setup, given 파트만 남는다. 너무 많은 것을 설정해야 한다면 호출하는 메소드가 하나 이상을 한다는 코드 악취이다! 복잡한 테스트는 구현이 너무 복잡하며 너무 많은 일을 함을 나타낸다! 리팩토링을 할 시간이며 나중에는 테스트가 더 쉬워질 것이다.

반면 짧은 메소드는 설명과 분리 둘 다 필요하지 않는다.

### 테스트 설정

물론 테스트 대상이 되는 객체를 설정할 필요가 있는 코드가 있다. 하지만 그 코드는 setUp() 메소드에 들어가야 한다!

코드가 일부 테스트에서만 필요하더라도 대부분의 코드를 그곳에 넣어야 한다. 왜?

아무도 당신의 setup()메소드를 읽지 않을 것이기 때문이다!

하지만 당신과 다른 개발자들은 당신의 테스트를 읽을 것이다!

테스트를 깨끗하게 유지하라!

### 명백한 것을 설명하지 마라

나는 GIVEN-WHEN-THEN 순서를 가지는 것이 좋음을 인정한다. 하지만 잠깐만, 우리는 이미 이를 가지고 있다! 테스트에서 어떻게 다른 순서를 가질 수 있겠는가? 시도해보라! 당신은 다른 것을 하기 전에 assert를 할 수 없다. assert 이후에 설정하는 것도 말이 안된다. 어째든 순서는 명백하다.

두 줄짜리 테스트는 기본적으로 WHEN과 THEN라인이다. 항상!

세 줄짜리 테스트는 아마 GIVEN 그리고 WHEN 그리고 THEN이다.

어쩌면 그들을 분리하는 새로운 줄을 유지하라.

어째든 분명한 사실을 주석으로 남기지 마라.

### Assertions

테스트의 마지막 부분인 어설션 코드가 읽기 힘들다면 주석을 고치지 말고 더 나은 어설션 라이브러리를 사용하라.

즉 다음 대신:

```java
assertEquals(3, list.size());
```

[fest assertions](https://github.com/alexruiz/fest-assert-2.x)을 작성할 수 있다:

```java
assertThat(list.size()).isEqualTo(3);
```

또는 [Hamcrest Matchers](http://hamcrest.org/JavaHamcrest/):

```java
assertThat(list, hasSize(3));
```

이제 주석에 관한 전체 에피소드를 이야기하였고 테스트에서 많은 노이즈를 제거했다. And there is more, stay tuned for part three!
