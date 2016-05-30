---
layout: post
category: blog
published: false
title: Untitled
---

[원본 : @RULES THAT CAN HELP YOU WRITE BETTER UNIT TESTS](http://www.schibsted.pl/blog/rules-that-can-help-you-write-better-unit-tests/)

### Have you ever wanted to know what was the result of your unit test and act differently depending on the test result? In this short article I present a JUnit feature called Rules.

# INTRODUCTION

Probably most of us are familiar with the JUnit framework and use it every day. We write tests and execute them. When there are problems we can see them in the IDE Runner panel (as red items) or in the output produced by the CLI tool. JUnit provides a useful set of annotations @Before, @BeforeClass, @After and @AfterClass. Methods marked with these annotations set up and clean the test environment, for example: open and close the database connection, release external resources etc.

아마 우리 대부분은 JUnit framework에 친숙하며 이를 매일 사용하고 있다. 우리는 테스트를 작성하고 그것들을 실행한다. 문제가 발생했을 때는 IDE Runner panel에서 (붉은색 항목으로서) 또는 CLI tool에서 생성되는 출력에서 이를 볼 수 있다. JUnit은 유용한 어노테이션들인 @Before, @BeforeClass, @After 그리고 @AfterClass를 제공한다. 이들 어노테이션으로 마크된 메소드들은 테스트 환경을 설정하고 청소한다. 예를 들어: Database 연결을 열고 닫기, 외부 리소스를 해지하기 등등.

But what if we want to know whether a particular test succeeded or failed during test execution? Is there a way to intercept such event and react on it?

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
