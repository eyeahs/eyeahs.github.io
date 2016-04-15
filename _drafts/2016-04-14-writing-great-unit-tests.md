---
published: false
---


# Writing Great Unit Tests: Best and Works Practices
Steve Sanderson, blog.stevensanderson.com
이 글은 최소한 유닛 테스트 경험이 조금이라도 있는 개발자를 목표로 한다. 만약 당신이 유닛 테스트를 작성해본적이 없다면 가이드를 읽고 먼저 해보기를 바란다.
_-_

좋은 유닛 테스트와 나쁜 유닛 테스트간의 차이점은 무엇일까? 좋은 유닛 테스트를 어떻게 작성하여야 하는지 어떻게 배웠는가? 이는 명확하지 않다. 심지어 당신이 수십년의 경험을 가진 우수한 코더이라고 해도, 당신의 현재 지식과 습관들은 당신이 좋은 유닛 테스트를 작성하도록 자동적으로 이끌지는 못한다. 이는 유닛 테스트는 또 다른 종류의 코딩이며 대부분의 사람들이 유닛 테스트들이 이루고자 하는 것에 대해서 도움이 안되는 잘못된 억측과 함께 출발하기 때문이다.

내가 본 대부분의 유닛 테스트들은 매우 도움이 되지 않는 것들이였다. 나는 개발자를 비난하는 중은 아니다: Usually, he or she just got told to start unit testing, so they installed NUnit and started churning out [Test] methods. 빨강과 녹색불을 한 번 보고나면 그것을 정확히 했다고 추정한다. 잘못된 추정이다! **코드 변경에 들어갈 노력을 천문학적으로 부풀리는데 반해 프로젝트에 매우 작은 가치만을 더하는 나쁜 유닛 테스트를 작성하는 것은 압도적으로 쉽다.** Does that sound agile to you?

## 유닛 테스트는 버그 찾기에 대한 것이 아니다.
지금 나는 유닛 테스트를 강하게 지지한다. 하지만 이는 당신이 유닛 테스트는 Test Driven Development (TDD) 과정내에서 활용한다는 규칙을 이해하고 유닛 테스트는 버그를 찾기 위해 테스트하는 것과 관계가 있다는 오해를 진압할 때 만이다.

내 경험으로는, 유닛 테스트는 버그를 찾거나 퇴보를 감지하기에 효율적인 방법은 아니다. 유닛 테스트는 .정의 그대로, 당신의 코드 유닛 각각을 각기 검사한다. 하지만 당신의 어플리케이션이 실제로 작동 할 때 모든 유닛들은 함깨 동작한다. 그리고 전체는 독립적으로 테스트된 부분들의 전체 합보다 훨씬 복잡하고 미묘하다. 컴포넌트 X와 Y이 독립적으로 동작함을 증명하는 것은 그들이 서로 호환될 수 있거나 정확하게 설정되었음을 증명하지 않는다. Also, defects in an individual component may bear no relationship to the symptoms an end user would experience and report. 그리고 당신은 유닛 테스트를 위한 전제 조건(precondition)을 만들었으므로 그 유닛 테스트는 당신이 예측하지 못한 전제 조건에 의해 촉발된 문제를 검출하지 못할 것이다. (for example, if some unexpected IHttpModule interferes with incoming requests).

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
![image.png]({{site.baseurl}}/_drafts/image.png)

Sweet Spot A / **진정한 유닛 테스트** / 단일 컴포넌트를 디자인 한다
지저분한 잡종 / 불명확한 목표. 높은 유지비용, 많은 것을 입증하지 못함 (불행히도, 보통 이러하다)
Sweet Spot B / **통합 테스트** / 퇴행을 발견하기 위해 전체 시스템을 자동화한다.

TDD를 통해 만들어진 유닛 테스트는 이 척도의 좌측 극단에 위치한다. 그것들은 단일 유닛 코드의 행동에 대한 많은 지식을 담고 잇다. 만약 유닛의 행동이 변경되면 유닛 테스트도 그렇게 되어야 하며, 역도 동일하다. 하지만 그것들은 당신의 코드 베이스의 다른 부분에 대한 어떠한 지식과 추정도 가지고 있지 않기 때문에 **당신의 코드 베이스의 다른 부분들을 변경하는 것은 그것들이 실패하도록 만들지 않는다** (그리고 당신의 것들이 그렇게 되었다면 이는 그것들은 진짜 유닛 테스트가 아님을 보여준다). Therefore they’re cheap to maintain, and as a development technique, TDD scales up to any size of project.

At the other end of the scale, integration tests contain no knowledge about how your codebase is broken down into units, but instead make statements about how the whole system behaves towards an external user. They’re reasonably cheap to maintain (because no matter how you restructure the internal workings of your system, it needn’t affect an external observer) and they prove a great deal about what features are actually working today.

그 사이 어딘가 있는 것은 당신이 만든 추정이 무엇인지와 당신이 입증하고자 하는 것이 무엇인지가 불명확하다. 리팩토링은 실사용자 경험이 여전히 동작하던지 그렇지 않던 관계없이 테스트들을 부수거나, 부수지 않을 것이다. (데이터베이스 업그레이드같이)당신이 사용하는 외부 서비스를 변경하는 것은 실사용자 경험이 여전히 동작하던지 그렇지 않던지 관계없이 테스트를 부수거나, 부수지 않을 것이다. 단일 유닛의 내부 작업을 변경하면 관계가 없어 보이는 수백개의 잡종 테스트들을 고쳐야 하도록 만들고, 어마어마한 양의 유지보수 시간을 소모하게 만들 것이다 -때때로 실제 어플리케이션 코드를 유지보수하는데 사용한 부위의 10배는 될 것이다. 그리고 이것은 


Anywhere in between, it’s unclear what assumptions you’re making and what you’re trying to prove. Refactoring might break these tests, or it might not, regardless of whether the end-user experience still works. Changing the external services you use (such as upgrading your database) might break these tests, or it might not, regardless of whether the end-user experience still works. Any small change to the internal workings of a single unit might force you to fix hundreds of seemingly unrelated hybrid tests, so they tend to consume a huge amount of maintenance time – sometimes in the region of 10 times longer than you spend maintaining the actual application code. And it’s frustrating because you know that adding more preconditions to make these hybrid tests go green doesn’t truly prove anything.

## Tips for writing great unit tests
Enough vague discussion – time for some practical advice. Here’s some guidance for writing unit tests that sit snugly at Sweet Spot A on the preceding scale, and are virtuous in other ways too.

*Make each test orthogonal (i.e., independent) to all the others 
**Any given behaviour should be specified in one *and only one test. Otherwise if you later change that behaviour, you’ll have to change multiple tests. The corollaries of this rule include: 
**Don’t make unnecessary assertions** Which specific behaviour are you testing? It’s counterproductive to Assert() anything that’s also asserted by another test: it just increases the frequency of pointless failures without improving unit test coverage one bit. This also applies to unnecessary Verify() calls – if it isn’t the core behaviour under test, then stop making observations about it! Sometimes, TDD folks express this by saying “*have only one logical assertion per test*”. Remember, unit tests are a design specification of how a certain behaviour should work, not a list of observations of *everything *the code happens to do. </li>

*Test only one code unit at a time 
**Your architecture *must support testing units (i.e., classes or very small groups of classes) independently, not all chained together. Otherwise, you have lots of overlap between tests, so changes to one unit can cascade outwards and cause failures everywhere. 
If you can’t do this, then your architecture is limiting your work’s quality – consider using Inversion of Control.
**Mock out all external services and state 
**Otherwise, behaviour in those external services overlaps multiple tests, and state data means that different unit tests can influence each other’s outcome. 
You’ve definitely taken a wrong turn if you have to run your tests in a specific order, or if they only work when your database or network connection is active. 
(By the way, sometimes your architecture might mean your code touches static variables during unit tests. Avoid this if you can, but if you can’t, at least make sure each test resets the relevant statics to a known state before it runs.)
*Avoid unnecessary preconditions 
**Avoid having common setup code that runs at the beginning of lots of unrelated tests. Otherwise, it’s unclear what assumptions each test relies on, and indicates that you’re not testing just a single unit. 
An exception: Sometimes I find it useful to have a common setup method shared by a *very small number of unit tests (a handful at the most) but only if all those tests require all of those preconditions. This is related to the context-specification unit testing pattern, but still risks getting unmaintainable if you try to reuse the same setup code for a wide range of tests.
(By the way, I wouldn’t count pushing multiple data points through the same test (e.g., using NUnit’s [TestCase] API) as violating this orthogonality rule. The test runner may display multiple failures if something changes, but it’s still only one test method to maintain, so that’s fine.) </li>

**Don’t unit-test configuration settings 
**By definition, your configuration settings aren’t part of any unit of code (that’s why you extracted the setting out of your unit’s code). Even if you could write a unit test that inspects your configuration, it merely forces you to specify the same configuration in an additional redundant location. Congratulations: it proves that you can copy and paste! Personally I regard the use of things like filters in ASP.NET MVC as being configuration. Filters like [Authorize] or [RequiresSsl] are configuration options baked into the code. By all means write an integration test for the externally-observable behaviour, but it’s meaningless to try unit testing for the filter attribute’s presence in your source code – it just proves that you can copy and paste again. That doesn’t help you to design anything, and it won’t ever detect any defects. </li>

Name your unit tests clearly and consistently 
**If you’re testing how ProductController’s Purchase action behaves when stock is zero, then maybe have a test fixture class called PurchasingTests with a unit test called ProductPurchaseAction_IfStockIsZero_RendersOutOfStockView(). This name describes the **subject (ProductController’s Purchase action), the scenario (stock is zero), and the result (renders “out of stock” view). I don’t know whether there’s an existing name for this naming pattern, though I know others follow it. How about S/S/R?  Avoid non-descriptive unit tests names such as Purchase() or OutOfStock(). Maintenance is hard if you don’t know what you’re trying to maintain. </li> </ul>

Conclusion
Without doubt, unit testing *can *significantly increase the quality of your project. Many in our industry claim that any unit tests are better than none, but I disagree: a test suite can be a great asset, or it can be a great burden that contributes little. It depends on the quality of those tests, which seems to be determined by how well its developers have understood the goals and principles of unit testing.

By the way, if you want to read up on integration testing (to complement your unit testing skills), check out projects such as Watin, Selenium, and even the ASP.NET MVC integration testing helper library I published recently.

Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.
