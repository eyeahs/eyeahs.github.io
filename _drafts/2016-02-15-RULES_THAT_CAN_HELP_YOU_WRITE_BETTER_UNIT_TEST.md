---
published: false
---

@RULES THAT CAN HELP YOU WRITE BETTER UNIT TESTS

Have you ever wanted to know what was the result of your unit test and act differently depending on the test result? In this short article I present a JUnit feature called Rules.


INTRODUCTION

Probably most of us are familiar with the JUnit framework and use it every day. We write tests and execute them. When there are problems we can see them in the IDE Runner panel (as red items) or in the output produced by the CLI tool. JUnit provides a useful set of annotations @Before, @BeforeClass, @After and @AfterClass. Methods marked with these annotations set up and clean the test environment, for example: open and close the database connection, release external resources etc.

But what if we want to know whether a particular test succeeded or failed during test execution? Is there a way to intercept such event and react on it?

PROBLEM

Let’s suppose that I want to save a file (for further investigation) produced by the test, but only in case of test failure and delete that file if all is fine.

My first thought would be to introduce a private field to store boolean value which can tell if previously executed test failed or not and test it in the @After method.

This seems to be a non-elegant solution. It is a general problem to know how test ends up. There must be a more sophisticated solution.

SOLUTION

And indeed, there is such a possibility in the JUnit framework. A mechanism to which I refer is realized by the @Rule annotation. There you can read more about it. To tell the story short. Rule wraps the entire test execution together with before and after methods, so we can hook up just right after the @After method and examine the status of the execution. What’s more, there are a few convenient implementation of TestRule interface.

Below is a list of the most useful from my standpoint:

TemporaryFolder: create fresh files, and delete after test
TestWatcher: add logic at events during method execution
Timeout: cause test to fail after a set time, see also @Test annotation and timeout attribute to obtain similar behaviour.

READ MORE IN DEVELOPERS´BLOG
SOURCE CODE FINALLY…

Please feel free to copy that code to your IDE and run it.


1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
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