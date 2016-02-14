---
layout: post
title: 비동기 메소드 유닛테스트
date: "2016-02-13 01:30:00 +0900"
categories: Unit Test
tags: "unittest,mockito"
published: true
---

원본 : [http://fernandocejas.com/2014/04/08/unit-testing-asynchronous-methods-with-mockito/](http://fernandocejas.com/2014/04/08/unit-testing-asynchronous-methods-with-mockito/)

After promising (and not keeping my promise) that I would be writing and maintaining my blog, here I go again (3289423987 attempt). But lest’s forget about that…
So in this occasion I wanted to write about Mockito…yes, this mocking framework that is a ‘must have’ when writing your unit tests ;).

## 소개
이 글은 당신이 이미 [단위 테스트](https://en.wikipedia.org/wiki/Unit_testing)가 무엇인지와 **왜 테스트를 작성해야 하는지**를 알고 있다고 가정한다.
또한 test double에 대해 이야기하는 [Martin Fowler의 유명한 글](http://martinfowler.com/articles/mocksArentStubs.html)을 강력하게 추천한다. 이것은 test double을 이해하기 위해 반드시 읽어야 한다.

## 흔한 사례의 시나리오
종종 우리는 콜백을 사용하는 메소드들을 테스트해야 하며, 이는 당연히 그것들이 비동기임을 의미한다. 이 메소드들은 테스트하기 쉽지 않으며 응답을 기다리기 위해 **Thread.sleep(milliseconds)** 메소드를 사용하는 것은 좋은 일이 아니며 당신의 테스트들을 [비결정적](http://martinfowler.com/articles/nonDeterminism.html)인 것으로 바꿀 수 있다(솔직히 말하면 나는 이것을 자주 보았다).
그러면 우리가 어떻게 해야 하는가? [Mockito](https://code.google.com/archive/p/mockito/) to the rescue!

## 예제 보기
우리는 **DummyCallback**을 구현하며 **doSomethingAsynchronously()** 메소드를 가지고 있는 **DummyCaller**라고 불리는 클래스를 가지고 있다고 가정하자. 그 메소드는 자신의 기능을 역시 **doSomethingAsynchronously(DummyCallback callback)** 메소드(하지만 이 메소드는 콜백을 파라미터로 (이 경우 우리의 **DummyCallback**) 받는다)를 가지고 있는 **DummyCollaborator**에게 위임한다. 그리고 이 메소드는 자신의 작업을 수행하기 위해 새로운 스레드를 만들고 종료될 때 우리에게 결과를 전달한다. 더 나은 방법으로 이 시나리오를 이해하기 위해 코드가 여기 있다:

{% highlight java %}
public interface DummyCallback {
	public void onSuccess(List<String> result);
	public void onFail(int code);
}
{% endhighlight %}

{% highlight java %}
public class DummyCaller implements DummyCallback {
 
	private final DummyCollaborator dummyCollaborator;
 
	private List<String> result = new ArrayList<String>();
 
	public DummyCaller(DummyCollaborator dummyCollaborator) {
		this.dummyCollaborator = dummyCollaborator;
	}
 
	public void doSomethingAsynchronously() {
		dummyCollaborator.doSomethingAsynchronously(this);
	}
 
	public List<String> getResult() {
		return this.result;
	}
 
	@Override
	public void onSuccess(List<String> result) {
		this.result = result;
		System.out.println("On success");
	}
 
	@Override
	public void onFail(int code) {
		System.out.println("On Fail");
	}
}
{% endhighlight %}

{% highlight java %}
public class DummyCollaborator {
 
	public static int ERROR_CODE = 1;
 
	public DummyCollaborator() {
		// empty
	}
 
	public void doSomethingAsynchronously (final DummyCallback callback) {
		new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					Thread.sleep(5000);
					callback.onSuccess(Collections.EMPTY_LIST);
				} catch (InterruptedException e) {
					callback.onFail(ERROR_CODE);
					e.printStackTrace();
				}
			}
		}).start();
	}
}
{% endhighlight %}

## 테스트 클래스 만들기
우리는 비동기 메소드를 테스트하는 두가지 방법을 가지고 있다. 하지만 첫번째로 우리는 테스트 클래스 DummyCollaboratorCallerTest를 만들 것이다 (for convention we just add Test at the end of the class so this becomes part of its name).

{% highlight java %}
public class DummyCollaboratorCallerTest {
 
	// Class under test
	private DummyCaller dummyCaller;
 
	@Mock
	private DummyCollaborator mockDummyCollaborator;
 
	@Captor
	private ArgumentCaptor<DummyCallback> dummyCallbackArgumentCaptor;
 
	@Before
	public void setUp() {
		MockitoAnnotations.initMocks(this);
		dummyCaller = new DummyCaller(mockDummyCollaborator);
	}
}
{% endhighlight %}

여기서 우리는 [Mock](http://docs.mockito.googlecode.com/hg/org/mockito/MockitoAnnotations.html)과 [ArgumentCaptor](http://docs.mockito.googlecode.com/hg/org/mockito/ArgumentCaptor.html)을 초기화하기 위해 **MockitoAnotations**을 사용하였지만 여기에 대해 아직 걱정할 필요는 없다, cause this is what we will be seeing next.
여기서 고려해야 할 유일한 것은 mock과 테스트 중인 클래스 둘 다 각 테스트가 실행되기 전에 setup() 메소드(@Before 어노테이션을 사용한)에서 초기화된다는 것이다.
단위 테스트에서 CUT (테스트 중인 클래스, class under test)의 협력자들은 모두 test double이여야 함을 기억하라.
이제 우리의 두가지 테스트 해결책들을 보자.

## 콜백에 answer 준비하기
This is our test case using a [doAnswer()](http://mockito.googlecode.com/svn/branches/1.6/javadoc/org/mockito/Mockito.html) for stubbing a method with a generic Answer. This means that since we need a callback to return immediately (synchronously), we generate an answer so when the method under test is called, the callback will be executed right away with the data we tell it to return.
Finally we call our real method and verify state and interaction.

{% highlight java %}
	@Test
	public void testDoSomethingAsynchronouslyUsingDoAnswer() {
		final List<String> results = Arrays.asList("One", "Two", "Three");
		
        // Let's do a synchronous answer for the callback
		doAnswer(new Answer() {
			@Override
			public Object answer(InvocationOnMock invocation) throws Throwable {
				((DummyCallback)invocation.getArguments()[0]).onSuccess(results);
				return null;
			}
		}).when(mockDummyCollaborator).doSomethingAsynchronously(
				any(DummyCallback.class));
 
		// Let's call the method under test
		dummyCaller.doSomethingAsynchronously();
 
		// Verify state and interaction
		verify(mockDummyCollaborator, times(1)).doSomethingAsynchronously(
				any(DummyCallback.class));
		assertThat(dummyCaller.getResult(), is(equalTo(results)));
	}
{% endhighlight %}

## ArgumentCaptor 사용하기
두번째 방안은 [ArgumentCaptor](http://docs.mockito.googlecode.com/hg/org/mockito/ArgumentCaptor.html)을 사용하는 것이다. 여기서 우리는 콜백을 비동기로 다룰 것이다 : 우리는 ArgumentCaptor를 사용해 **DummyCallback** 객체를 잡아서 **DummyCollaborator**로 넘겨줄 것이다.
마지막으로 우리는 모든 assertions를 
테스트 메소드 레벨에서 그리고 상태와 상호 작용을 검증할 때 onSuccess()를 호출한다.
Here we treat our callback asynchronously: we capture the DummyCallback object passed to our DummyCollaborator using an ArgumentCaptor.
Finally we can make all our assertions at the test method level and call onSuccess() when we want to verify state and interaction.

	@Test
	public void testDoSomethingAsynchronouslyUsingArgumentCaptor() {
		// Let's call the method under test
		dummyCaller.doSomethingAsynchronously();
 
		final List<String> results = Arrays.asList("One", "Two", "Three");
 
		// Let's call the callback. ArgumentCaptor.capture() works like a matcher.
		verify(mockDummyCollaborator, times(1)).doSomethingAsynchronously(
				dummyCallbackArgumentCaptor.capture());
 
		// Some assertion about the state before the callback is called
		assertThat(dummyCaller.getResult().isEmpty(), is(true));
 
		// Once you're satisfied, trigger the reply on callbackCaptor.getValue().
		dummyCallbackArgumentCaptor.getValue().onSuccess(results);
 
		// Some assertion about the state after the callback is called
		assertThat(dummyCaller.getResult(), is(equalTo(results)));
	}
{% endhighlight %}

## 결론
The main difference between both solutions is that when using DoAnswer() we are creating an anonymous inner class, and casting (in an unsafe way) the elements from invocation.getArguments()[n] to the data type we want, but in case we modify our parameters the test will ‘fail fast’ letting know that something has happened. On the other side, when using ArgumentCaptor we probably have more control cause we can call the callbacks in the order we want in case we need it.
As interest in unit testing, this is a common case that sometimes we do not know how deal with, so in my experience using both solutions has helped me to have a robust approach when having to test asynchronous methods.
I hope you find this article useful, and as always, remember that any feedback is very welcome, as well as other ways of doing this. Of course if you have any doubt do not hesitate to contact me.

## Code Sample
[Here is the link where you can find this example and others](https://github.com/android10/Inside_Android_Testing). Most of them are related with Java and Android because this comes from a talk I gave a couple of months ago.
The presentation is in english but the video is in spanish (sorry for those who do not understand my argentinian accent…haha…BTW I will try to upload an english version as soon as possible…)

https://speakerdeck.com/android10/how-android-testing-changed-how-we-think-about-death

## Further Reading
I highly recommend to take a look at Mockito documentation to have a better understanding of the framework. The documentation is very clear and has great examples. See you!