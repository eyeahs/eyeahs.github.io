---
published: false
---


# Writing Great Unit Tests: Best and Works Practices
Steve Sanderson, blog.stevensanderson.com

좋은 유닛 테스트와 나쁜 유닛 테스트간의 차이점은 무엇일까? 좋은 유닛 테스트를 어떻게 작성하여야 하는지 어떻게 배웠는가? 이는 명확하지 않다. 심지어 당신이 수십년의 경험을 가진 우수한 코더이라고 해도, 당신의 현재 지식과 습관들은 당신이 좋은 유닛 테스트를 작성하도록 자동적으로 이끌지는 못한다. 이는 유닛 테스트는 또 다른 종류의 코딩이며 대부분의 사람들이 유닛 테스트들이 이루고자 하는 것에 대해서 도움이 안되는 잘못된 억측과 함께 출발하기 때문이다.

내가 본 대부분의 유닛 테스트들은 매우 도움이 되지 않는 것들이였다. 나는 개발자를 비난하는 중은 아니다: Usually, he or she just got told to start unit testing, so they installed NUnit and started churning out [Test] methods. 빨강과 녹색불을 한 번 보고나면 그것을 정확히 했다고 추정한다. 잘못된 추정이다! 코드 변경에 들어갈 노력을 천문학적으로 부풀리는데 반해 프로젝트에 매우 작은 가치만을 더하는 나쁜 유닛 테스트를 작성하는 것은 압도적으로 쉽다. Does that sound agile to you?

# 유닛 테스티는 버그 찾기에 대한 것이 아니다.
지금 나는 유닛 테스트를 강하게 지지한다. 하지만 이는 당신이 유닛 테스트는 Test Driven Development (TDD) 과정내에서 활용한다는 규칙을 이해하고 유닛 테스트는 버그를 찾기 위해 테스트하는 것과 관계가 있다는 오해를 진압할 때 만이다.

내 경험으로는, 유닛 테스트는 버그를 찾거나 퇴보를 감지하기에 효율적인 방법은 아니다. 유닛 테스트는 .정의 그대로, 당신의 코드 유닛 각각을 각기 검사한다. 하지만 당신의 어플리케이션이 실제로 작동 할 때 모든 유닛들은 함깨 동작한다. 그리고 전체는 독립적으로 테스트된 부분들의 전체 합보다 훨씬 복잡하고 미묘하다. 컴포넌트 X와 Y이 독립적으로 동작함을 증명하는 것은 그들이 서로 호환될 수 있거나 정확하게 설정되었음을 증명하지 않는다. Also, defects in an individual component may bear no relationship to the symptoms an end user would experience and report. 그리고 당신은 유닛 테스트를 위한 전제 조건(precondition)을 만들었으므로 그 유닛 테스트는 당신이 예측하지 못한 전제 조건에 의해 촉발된 문제를 검출하지 못할 것이다. (for example, if some unexpected IHttpModule interferes with incoming requests).

So, if you’re trying to find bugs, it’s far more effective to actually run the whole application together as it will run in production, just like you naturally do when testing manually. If you automate this sort of testing in order to detect breakages when they happen in the future, it’s called integration testing and typically uses different techniques and technologies than unit testing. Don’t you want to use the most appropriate tool for each job?


| 목적 |	유력한 기술 |
| Finding bugs (things that don’t work as you want them to)	| Manual testing (sometimes also automated integration tests) | 
| Detecting regressions (things that used to work but have unexpectedly stopped working) | Automated integration tests (sometimes also manual testing, though time-consuming) |
| Designing software components robustly | Unit testing (within the TDD process) |
(Note: there’s one exception where unit tests do effectively detect bugs. It’s when you’re refactoring, i.e., restructuring a unit’s code but without meaning to change its behaviour. In this case, unit tests can often tell you if the unit’s behaviour has changed.)

Well then, if unit testing isn’t about finding bugs, what is it about?
I bet you’ve heard the answer a hundred times already, but since the testing misconception stubbornly hangs on in developers’ minds, I’ll repeat the principle. As TDD gurus keep saying, “TDD is a design process, not a testing process”. Let me elaborate: TDD is a robust way of designing software components (“units”) interactively so that their behaviour is specified through unit tests. That’s all!

Good unit tests vs bad ones
TDD helps you to deliver software components that individually behave according to your design. A suite of good unit tests is immensely valuable: it documents your design, and makes it easier to refactor and expand your code while retaining a clear overview of each component’s behaviour. However, a suite of bad unit tests is immensely painful: it doesn’t prove anything clearly, and can severely inhibit your ability to refactor or alter your code in any way.

Where do your tests sit on the following scale?

image

Unit tests created through the TDD process naturally sit at the extreme left of this scale. They contain a lot of knowledge about the behaviour of a single unit of code. If that unit’s behaviour changes, so must its unit tests, and vice-versa. But they don’t contain any knowledge or assumptions about other parts of your codebase, so **changes to other parts of your codebase don’t make them start failing **(and if yours do, that shows they aren’t true unit tests). Therefore they’re cheap to maintain, and as a development technique, TDD scales up to any size of project.

At the other end of the scale, integration tests contain no knowledge about how your codebase is broken down into units, but instead make statements about how the whole system behaves towards an external user. They’re reasonably cheap to maintain (because no matter how you restructure the internal workings of your system, it needn’t affect an external observer) and they prove a great deal about what features are actually working today.

Anywhere in between, it’s unclear what assumptions you’re making and what you’re trying to prove. Refactoring might break these tests, or it might not, regardless of whether the end-user experience still works. Changing the external services you use (such as upgrading your database) might break these tests, or it might not, regardless of whether the end-user experience still works. Any small change to the internal workings of a single unit might force you to fix hundreds of seemingly unrelated hybrid tests, so they tend to consume a huge amount of maintenance time – sometimes in the region of 10 times longer than you spend maintaining the actual application code. And it’s frustrating because you know that adding more preconditions to make these hybrid tests go green doesn’t truly prove anything.

Tips for writing great unit tests
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
