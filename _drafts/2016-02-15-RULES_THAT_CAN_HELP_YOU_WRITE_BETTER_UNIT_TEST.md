---
layout: post
category: blog
published: false
title: Untitled
---

[원본 : @RULES THAT CAN HELP YOU WRITE BETTER UNIT TESTS](http://www.schibsted.pl/blog/rules-that-can-help-you-write-better-unit-tests/)

### 당신의 unit test의 결과가 무엇인지 알고 테스트 결과에 따라 다른 행동을 하고 싶었던 적이 있었는가? 이 짧은 글에서 나는 Rules라고 불리는 JUnit 기능을 소개한다.

# INTRODUCTION

아마 우리 대부분은 JUnit framework에 친숙하고 매일 사용하고 있을 것이다. 우리는 테스트를 작성하고 실행한다. 문제가 있으면 (붉은색 항목으로) IDE의 실행 창에서 또는 CLI tool이 생성하는 출력에서 이를 볼 수 있다. JUnit은 유용한 어노테이션들 @Before, @BeforeClass, @After 그리고 @AfterClass를 제공한다. 이들 어노테이션으로 마크된 메소드들은 테스트 환경을 설정하고 청소한다. 예를 들어: Database 연결을 열고 닫기, 외부 리소스를 해지하기 등등.

# PROBLEM

내가 테스트에서 생성된 파일을 (추가 조사를 위해) 저장하고 싶다고 하자. 단 테스트를 실패한 경우 
Let’s suppose that I want to save a file (for further investigation) produced by the test, but only in case of test failure and delete that file if all is fine.

My first thought would be to introduce a private field to store boolean value which can tell if previously executed test failed or not and test it in the @After method.

This seems to be a non-elegant solution. It is a general problem to know how test ends up. There must be a more sophisticated solution.

# SOLUTION

And indeed, there is such a possibility in the JUnit framework. A mechanism to which I refer is realized by the @Rule annotation. There you can read more about it. To tell the story short. Rule wraps the entire test execution together with before and after methods, so we can hook up just right after the @After method and examine the status of the execution. What’s more, there are a few convenient implementation of TestRule interface.

Below is a list of the most useful from my standpoint:

TemporaryFolder: create fresh files, and delete after test
TestWatcher: add logic at events during method execution
Timeout: cause test to fail after a set time, see also @Test annotation and timeout attribute to obtain similar behaviour.

READ MORE IN DEVELOPERS´BLOG
SOURCE CODE FINALLY…

Please feel free to copy that code to your IDE and run it.

    package test;

    import static org.junit.Assert.assertEquals;

    import org.junit.After;
    import org.junit.Before;
    import org.junit.Rule;
    import org.junit.Test;
    import org.junit.rules.TestRule;
    import org.junit.rules.TestWatcher;
    import org.junit.runner.Description;


    class UnderTestClass {

    public String doSth() {
    return "done";
    }
    }

    public class UnderTestClassTest {

    private UnderTestClass subject;

    @Before
    public void setup() {
    System.out.println("Setup...");
    subject = new UnderTestClass();
    }

    @After
    public void clean() {
    System.out.println("Cleaning after test...");
    }

    @Rule
    public TestRule rule = new TestWatcher() {

    @Override
    protected void failed(Throwable e, Description description) {
    System.out.println("Failed " + description);
    }

    @Override
    protected void finished(Description description) {
    System.out.println("Finished " + description + "\n");
    }

    @Override
    protected void starting(Description description) {
    System.out.println("Starting " + description);
    }

    @Override
    protected void succeeded(Description description) {
    System.out.println("Passed " + description);
    }
    };

    @Test
    public void testDoSth() {
    System.out.println("In test...");
    assertEquals("done", subject.doSth());
    }

    @Test
    public void testNothingAndFail() {
    System.out.println("In test...");
    assertEquals("done", "no");
    }
    }

You should be able to see that output in the console. Please notice highlighted lines. You can clearly see that the appropriate callbacks were called depending on the test result.

    Starting testDoSth(pl.snd.test.UnderTestClassTest)
    Setup…
    In test…
    Cleaning after test…
    Passed testDoSth(pl.snd.test.UnderTestClassTest)
    Finished testDoSth(pl.snd.test.UnderTestClassTest)

    Starting testNothingAndFail(pl.snd.test.UnderTestClassTest)
    Setup…
    In test…
    Cleaning after test…
    Failed testNothingAndFail(pl.snd.test.UnderTestClassTest)
    Finished testNothingAndFail(pl.snd.test.UnderTestClassTest)

CONCLUSION

In conjunction with the @Before and @After family we can separate test logic from the whole logistic connected with the external resources. This simple, yet underestimated feature of JUnit, gives us more control over our test and helps us keep them clean.
