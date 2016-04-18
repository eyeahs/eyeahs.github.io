---
layout: post
title: "Writing Great Unit Tests: Best and Works Practices"
date: "2016-04-18 16:07:46 +0900"
categories: unittest
published: true
---

# Writing Great Unit Tests: Best and Works Practices
[원본](http://blog.stevensanderson.com/2009/08/24/writing-great-unit-tests-best-and-worst-practises/) / Steve Sanderson, blog.stevensanderson.com
-_-
이 글은 최소한 유닛 테스트 경험이 조금이라도 있는 개발자를 목표로 한다. 만약 당신이 유닛 테스트를 작성해본적이 없다면 가이드를 읽고 먼저 해보기를 바란다.
_-_

좋은 유닛 테스트와 나쁜 유닛 테스트간의 차이점은 무엇일까? 좋은 유닛 테스트를 어떻게 작성하여야 하는지 어떻게 배웠는가? 이는 명확하지 않다. 심지어 당신이 수십년의 경험을 가진 우수한 코더이라고 해도, 당신의 현재 지식과 습관들은 당신이 좋은 유닛 테스트를 작성하도록 자동적으로 이끌지는 못한다. 이는 유닛 테스트는 또 다른 종류의 코딩이며 대부분의 사람들이 유닛 테스트들이 이루고자 하는 것에 대해서 도움이 안되는 잘못된 억측과 함께 출발하기 때문이다.

내가 본 대부분의 유닛 테스트들은 매우 도움이 되지 않는 것들이였다. 나는 개발자를 비난하는 중은 아니다: 보통 그와 그녀들은 유닛 테스트를 시작하라는 당부를 받았고, 그래서 NUnit을 설치하고 테스트 메소드를 대량으로 생상하기 시작한다. 빨강과 녹색불을 한 번 보고나면 그것을 정확히 했다고 추정한다. 잘못된 추정이다! **코드 변경에 들어갈 노력을 천문학적으로 부풀리는데 반해 프로젝트에 매우 작은 가치만을 더하는 나쁜 유닛 테스트를 작성하는 것은 압도적으로 쉽다.** Does that sound agile to you?

## 유닛 테스트는 버그 찾기에 대한 것이 아니다.
지금 나는 유닛 테스트를 강하게 지지한다. 하지만 이는 당신이 유닛 테스트는 Test Driven Development (TDD) 과정내에서 활용한다는 규칙을 이해하고 유닛 테스트는 버그를 찾기 위해 테스트하는 것과 관계가 있다는 오해를 진압할 때 만이다.

내 경험으로는, 유닛 테스트는 버그를 찾거나 퇴보를 감지하기에 효율적인 방법은 아니다. 유닛 테스트는 .정의 그대로, 당신의 코드 유닛 각각을 각기 검사한다. 하지만 당신의 어플리케이션이 실제로 작동 할 때 모든 유닛들은 함깨 동작한다. 그리고 전체는 독립적으로 테스트된 부분들의 전체 합보다 훨씬 복잡하고 미묘하다. 컴포넌트 X와 Y이 독립적으로 동작함을 증명하는 것은 그들이 서로 호환될 수 있거나 정확하게 설정되었음을 증명하지 않는다. Also, defects in an individual component may bear no relationship to the symptoms an end user would experience and report. 그리고 당신은 유닛 테스트를 위한 전제 조건(precondition)을 만들었으므로 그 유닛 테스트는 당신이 예측하지 못한 전제 조건에 의해 촉발된 문제를 검출하지 못할 것이다. (예를 들어 예상치 못한 IHttpModule이 새로 들어온 요청들을 방해한다면)
그래서 만약 당신이 버그를 찾는 것을 시도한다면, 당신이 손으로 테스트를 할 때 당연히 하듯이, 실제로 운영에서 동작하는 것처럼 전체 어플리케이션을 결합하여 실행해보는 것이 훨씬 더 효과적이다. 만약 당신이 미래에 일어날 파손을 검출하기 위한 이런 종류의 테스트를 자동화한다면 이는 통합 테스트라고 불릴 것이며 일반적으로 유닛 테스트와는 다른 다른 종류의 기술과 장비를 사용할 것이다.
각 작업에 가장 적절한 툴을 사용하고 싶지 않는가?

| 목적 |	유력한 기술 |
| --- | --------- |
| 버그 찾기 (원하는대로 동작하지 않는 것) | 손으로 테스트 (때때로 자동화된 통합 테스트 또한) | 
| 퇴행 감지 (동작하곤 했지만 예기치 않게 동작이 중단되는 것) | 자동화된 통합 테스트 (때때로 시간이 많이 걸리긴 하지만 손으로 테스트하기) |
| 소프트웨어 컴포넌트를 튼튼하게(robustly) 디자인하기 | 유닛 테스트 (TDD 과정속에서) |
(Note: 유닛 테스트가 효과적으로 오류를 검출하는 예외가 하나 있다. 이는 당신이 리팩토링을 할 때이다. 즉, 동작을 변경하겠다는 의도없이 유닛의 코드를 구조 조정할 때이다. 이 경우 유닛 테스트는 유닛의 동작이 변경되었을 경우 대게 당신에게 알려 줄 수 있을 것이다.)

### 그러면, 만약 유닛 테스트가 버그 찾기에 대한 것이 아니라면 이는 무엇에 대한 것인가?
나는 당신이 이미 대답을 백번은 들었음에 돈을 걸겠다. 하지만 테스트에 대한 오해가 개발자의 마음에 완고하게 버티고 있기 때문에 나는 원칙을 반복하겠다. TDD 구루들이 계속 말하는 것처럼, "TDD는 테스트 과정이 아니라 개발 과정이다". 더 자세히 말하면 :**TDD는 유닛 테스트를 통해 그들의 행동을 구체화하기 위해 소프트웨어 컴포넌트("유닛")을 쌍방향(interactively)으로 디자인하는 강건한 방식이다.** That’s all!

## Good unit tests vs bad ones
TDD는 당신의 디자인에 따라 독립적으로 움직이는 소프트웨어 컴포넌트를 당신이 만드는 것을 돕는다. 좋은 유닛 테스트 모음(suite)은 엄청나게 갑지다: 이는 당신의 디자인을 기록하고, 각 컴포넌트의 행동의 명확한 개요를 유지한채로 당신의 코드를 리팩토링하고 확장하기 쉽게 해준다. 하지만, 나쁜 유닛 테스트의 모음은 엄청나게 고통스럽다: 이는 아무것도 명확히 입증하지 못하고, 당신의 코드를 어떻게든 고치거나 리팩토링할 능력을 혹독하게 억제한다. 

당신의 테스트들은 다음 척도에서 어디에 놓여있는가?
![image.png]({{site.baseurl}}/_posts/2016-04-14-writing-great-unit-tests-images/image.png)

Sweet Spot A / **진정한 유닛 테스트** / 단일 컴포넌트를 디자인 한다
지저분한 잡종 / 불명확한 목표. 높은 유지비용, 많은 것을 입증하지 못함 (불행히도, 보통 이러하다)
Sweet Spot B / **통합 테스트** / 퇴행을 발견하기 위해 전체 시스템을 자동화한다.

TDD를 통해 만들어진 유닛 테스트는 이 척도의 좌측 극단에 위치한다. 그것들은 단일 유닛 코드의 행동에 대한 많은 지식을 담고 잇다. 만약 유닛의 행동이 변경되면 유닛 테스트도 그렇게 되어야 하며, 역도 동일하다. 하지만 그것들은 당신의 코드 베이스의 다른 부분에 대한 어떠한 지식과 추정도 가지고 있지 않기 때문에 **당신의 코드 베이스의 다른 부분들을 변경하는 것은 그것들이 실패하도록 만들지 않는다** (그리고 당신의 것들이 그렇게 되었다면 이는 그것들은 진짜 유닛 테스트가 아님을 보여준다). Therefore they’re cheap to maintain, and as a development technique, TDD scales up to any size of project.

척도를 다른 쪽 끝에서, 통합 테스트는 당신의 코드 베이스가 어떻게 유닛들로 세분화되었는지에 대한 지식을 가지고 있지 않지만, 대신 전체 시스템이 외부 사용자에게 어떻게 동작해야 하는지를 진술한다. 그것들은 유지하기 상당히 저렴하며 (당신이 당신의 시스템의 내부 동작을 어떻게 재구성하던지간에 이것은 외부 관철자에 영향을 줄 필요가 없기 때문이다.) 현재 기능들이 정확하게 동작하는지 훨씬 더 입증한다.

그 사이 어딘가 있는 것은 당신이 만든 추정이 무엇인지와 당신이 입증하고자 하는 것이 무엇인지가 불명확하다. 리팩토링은 실사용자 경험이 여전히 동작하던지 그렇지 않던 관계없이 테스트들을 부수거나, 부수지 않을 것이다. (데이터베이스 업그레이드같이) 당신이 사용하는 외부 서비스를 변경하는 것은 실사용자 경험이 여전히 동작하던지 그렇지 않던지 관계없이 테스트를 부수거나, 부수지 않을 것이다. 단일 유닛의 내부 작업을 변경하면 관계가 없어 보이는 수백개의 잡종 테스트들을 고쳐야 하도록 만들고, 어마어마한 양의 유지보수 시간을 소모하게 만들 것이다 -때때로 실제 어플리케이션 코드를 유지보수하는데 사용한 부위의 10배는 될 것이다. 그리고 이것은 잡종 테스트가 녹색이 되도록 하기 위해 더 많은 전제조건을 추가하는 것은 무엇도 진짜 입증하지 않는다는 것을 알기 때문에 좌절감을 줄 것이다.

## Tips for writing great unit tests
멍청한 논의는 충분하다 - 현실적인 조언들의 시간이다. 전술했던 척도의 Sweet Spot A에 안락하게 앉는 유닛 테스트를 작성하기 위한 안내와 고결한 다른 방법들이 여기 있다.

- 각각의 테스트는 나머지 모두와 직교(즉, 독립)하게 만들어라.
어느 주어진 행동은 하나(one, 역:코드 유닛을 말하는 듯)와 **오직 한 테스트**에만 명시되어야 한다. 그렇지 않으면 이후에 그 행동을 변경하였을 때 여러 테스트도 변경해야 한다. 이 규칙의 당연한 결과는 다음을 포함한다: **불필요한 assertion을 만들지 마라** 
당신이 테스트하는 특정 행위는 무엇인가? 다른 테스트에서도 역시 assert된 것에 Assert()를 하는 것은 역효과를 낳는다: 이는 유닛 테스트 커버리지를 1비트라도 늘리지 않고 무의미한 실패들의 빈도만 늘릴 뿐이다. 이는 불필요한 Verify() 호출에도 역시 적용된다 - 만약 이것이 테스트의 핵심 동작이 아니라면 이를 감시하게 만드는 것을 멈추어라! 가끔 TDD 사람들은 이것을 **"테스트 하나당 오직 하나의 논리적 주장만 가져라(have only one logical assertion per test)"**라고 말한다. 유닛 테스트는 특정 행동이 어떻게 동작해야 하는지에 대한 디자인 설명서이며 코드에서 일어나는 **모든**것에 대한 감시 리스트가 아님을 기억하라.

- 한 번에 하나의 코드만 테스트하라
당신의 아키텍처는 독립적이고 모두 함께 묶여있지 않는 테스팅 유닛(testing units)를 지원해야 한다 (즉 클래스들 또는 아주 작은 클래스 그룹들). 그렇지 않으면 테스트들간에 많은 겹침을 가지게 되어 한 유닛의 변경이 밖으로 쏟아져 나와 모든 것이 실패하게 만든다.
만약 그렇게 할 수 없다면 당신의 아키텍쳐가 당신 작업의 품질을 제한하게 된다 - Inversion of Control를 사용하는 것을 고려하라.

- 모든 외부 서비스와 상태를 mock으로 하라.
그렇지 않으면 이 외부 서비스들의 행동들은 여러 테스트들에서 겹쳐지고, 상태 데이터는 각각 다른 유닛 테스트들은 서로의 결과에 영향을 줄 수 있다.
만약 당신의 테스트가 특정 순서로 수행되어야 한다면 분명히 잘못된 길을 든 것이다. 아니면 당신의 데이터베이스나 네트워크 연결이 active할 때만 동작할 것이다.
(그런데 가끔 어떤 당신의 architecture가 당신의 코드가 테스트동안 static 변수에 손대도록 의도할 때가 있다. 가능하면 이를 피하라, 그럴 수 없다면, 최소한 각 테스트가 반드시 수행전에 known state로 관련 statics를 리셋하도록 하라.)

- 불필요한 전제 조건들을 피하라.
관련 없는 수많은 테스트들의 시작에서 공통 setup 코드를 가지는 것을 피하라. 그렇지 않으면, 각 테스트가 어떤 추정에 의지하는지가 불명확하며, 단지 단일 유닛을 테스트하는 것이 아님을 나타낸다.
가끔 나는 
가끔 나는 공통 setup 메소드들 매우 작은 수의 유닛 테스트들(대부분 한 웅큼)에서 공유하는 것이 유용하다는 것을 알게 되었다. 하지만 이 테스트들을 but only if all those tests require all of those preconditions. 이는 context-specification 유닛 테스트 패턴과 관련이 있다. 하지만 동일 setup 코드를 테스트에서 광법위하게 재사용하려고 하는 것은 유지보수 불가능하게 만들 위험이 여전이 있다.
(By the way, I wouldn’t count pushing multiple data points through the same test (e.g., using NUnit’s [TestCase] API) as violating this orthogonality rule. The test runner may display multiple failures if something changes, but it’s still only one test method to maintain, so that’s fine.)

**환경 설정(configuration settings)을 유닛 테스트하지 마라**
당연히 당신의 환경 설정은 코드 유닛의 일부가 아니다 (이것이 당신이 당신의 유닛의 코드에서 setting을 추출하는 이유이다). 당신이 당신의 환경 설정을 검증하는 유닛 테스트를 작성하는 것이 가능할지라도 이는 당신에게 불필요한 부자적 위치에 동일한 환경 설정을 명시하게 만드는 것일 뿐이다. 축하한다 : 이는 당신이 복불을 할 수 있다는 것을 증명한다! Personally I regard the use of things like filters in ASP.NET MVC as being configuration. Filters like [Authorize] or [RequiresSsl] are configuration options baked into the code. By all means write an integration test for the externally-observable behaviour, but it’s meaningless to try unit testing for the filter attribute’s presence in your source code – it just proves that you can copy and paste again. That doesn’t help you to design anything, and it won’t ever detect any 

**유닛 테스트의 이름을 명확하고 일관되게 짓도록 하라**
만약 당신이 ProductController의 Purchase 액션이 재고가 0일 때 어떻게 동작하는지를 테스트한다면, ProductPurchaseAction_IfStockIsZero_RendersOutOfStockView()라고 불리는 유닛 테스트를 가진 PurchasingTest라고 불리는 테스트 고정 클래스를 가질 것이다. 이 이름은 주제(ProductController's Purchase action), 시나리오 (재고가 0), 그리고 결과 ("재고 품절" 화면을 만든다)을 설명한다. 나는 이 네이밍 패턴에 이름이 존재하는지 아닌지 모르긴해도 다른 이들이 이를 따르고 있다는 것은 알고 있다. S/S/R은 어떤가? Purchase()나 OutOfStock()같이 서술적이지 않는 유닛 테스트 이름은 피하라. 만약 당신이 유지하고자 하는 것이 무엇인지 당신이 모른다면 유지보수는 어렵다.

#결론
의심의 여지없이, 유닛 테스트는 당신의 프로젝트의 품질을 *상당히* 증가시킬 수 있다. 우리 업계의 대부분의 사람들이 어떠한 유닛 테스트라도 없는 것보다는 낫다라고 주장한다. 하지만 나는 그에 반대한다 : 테스트 모음(test suite)는 위대한 자산이 될 수도 기여하는 바가 거의 없는 거대한 짐이 될 수도 있다. 이는 개발자들이 유닛 테스트의 목표과 원칙들을 얼마나 잘 이해하고 있는지에 의해 결정되는 테스트의 품질에 달려있다.

By the way, if you want to read up on integration testing (to complement your unit testing skills), check out projects such as Watin, Selenium, and even the ASP.NET MVC integration testing helper library I published recently.
