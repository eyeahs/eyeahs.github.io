---
layout: post
category: blog
title: Mockito로 비동기 메소드를 단위 테스트하기
date: "2016-02-13 01:30:00 +0900"
categories: Unit Test
tags: 
  - "unittest,mockito"
published: true
splash: ""
---


원본 : [http://fernandocejas.com/2014/04/08/unit-testing-asynchronous-methods-with-mockito/](http://fernandocejas.com/2014/04/08/unit-testing-asynchronous-methods-with-mockito/)

After promising (and not keeping my promise) that I would be writing and maintaining my blog, here I go again (3289423987 attempt). But lest’s forget about that…
So in this occasion I wanted to write about Mockito…yes, this mocking framework that is a ‘must have’ when writing your unit tests ;).

## 소개
이 글은 당신이 이미 [단위 테스트](https://en.wikipedia.org/wiki/Unit_testing)가 무엇인지와 **왜 테스트를 작성해야 하는지**를 알고 있다고 가정한다.
또한 test double을 설명하는 [Martin Fowler의 유명한 글](http://martinfowler.com/articles/mocksArentStubs.html)을 강력 추천한다. 이 글은 test double을 이해하기 위해서 반드시 읽어야 한다.

## 흔한 사례의 시나리오
우리는 종종 콜백을 이용하는 메소드들을 테스트해야 하며, 이는 당연히 그것들이 비동기임을 의미한다. 이 메소드들은 테스트하기가 쉽지 않다. 그리고 응답을 기다리기 위해 **Thread.sleep(milliseconds)** 메소드를 사용하는 것은 좋은 일이 아니며 당신의 테스트들을 [비결정적](http://martinfowler.com/articles/nonDeterminism.html)인 것으로 만들 수 있다(솔직히 말해서 나는 이런 것들을 자주 보았다).
그러면 우리가 어떻게 해야 하는가? [Mockito](https://code.google.com/archive/p/mockito/)가 해결책이다!

## 예제 보기
우리는 **DummyCallback**을 구현하고 **doSomethingAsynchronously()** 메소드를 가지고 있는 **DummyCaller**라는 클래스를 가지고 있다고 가정하자. 이 메소드는 자신의 기능을 **doSomethingAsynchronously(DummyCallback callback)** 메소드(콜백-이번에는 **DummyCallback**-을 파라미터로 받는)를 가지고 있는 **DummyCollaborator**에게 위임한다. 그리고 이 메소드는 자신의 작업을 수행하기 위해 새로운 스레드를 만들고 종료될 때 우리에게 결과를 전달한다. 다음은 시나리오를 더 이해하기 위한 코드이다 :

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
비동기 메소드를 테스트 방법은 두 가지가 있다. 일단 테스트 클래스 DummyCollaboratorCallerTest를 먼저 만든다 (편의상 클래스 끝에 오직 Test만을 붙일 것이며 그 결과가 클래스의 이름이 된다).

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

여기서 우리는 [Mock](http://docs.mockito.googlecode.com/hg/org/mockito/MockitoAnnotations.html)과 [ArgumentCaptor](http://docs.mockito.googlecode.com/hg/org/mockito/ArgumentCaptor.html)을 초기화하기 위해 **MockitoAnotations**을 사용하다. 이것들에 대해 벌써부터 염려할 필요는 없다. 조금 뒤에 살펴 볼 것이다. 
여기서 신경써야 할 유일한 점은 mock과 테스트 중인 클래스 둘 다 각각의 테스트가 실행되기 전에 setup() 메소드(@Before 어노테이션을 사용한)에서 초기화된다는 것이다.
**단위 테스트에서 CUT (테스트 중인 클래스, class under test)의 협력자들은 모두 test double이여야 함을 기억하라.**

이제 우리의 테스트 해결책 두 가지를 보자.

## 콜백에 answer 준비하기
이 테스트 케이스는 메소드를 generic Answer로 스텁(stub)하기 위해 [doAnswer()](http://mockito.googlecode.com/svn/branches/1.6/javadoc/org/mockito/Mockito.html)을 사용한다. 이는 즉시(동기로) 복귀하는 콜백이 필요하기 때문에 answer를 만들었으며, 테스트되는 메소드가 호출될 때 우리가 반환하도록 한 데이터와 함께 콜백이 즉시 실행됨을 의미한다.
마지막으로 실제 메소드를 호출한 뒤 상태와 상호 작용을 검증(verify)한다.

{% highlight java %}
	@Test
	public void testDoSomethingAsynchronouslyUsingDoAnswer() {
		final List<String> results = Arrays.asList("One", "Two", "Three");
		
        // 콜백을 위해 동기 응답을 한다.
		doAnswer(new Answer() {
			@Override
			public Object answer(InvocationOnMock invocation) throws Throwable {
				((DummyCallback)invocation.getArguments()[0]).onSuccess(results);
				return null;
			}
		}).when(mockDummyCollaborator).doSomethingAsynchronously(
				any(DummyCallback.class));
 
		// 테스트 대상인 메소드를 호출한다.
		dummyCaller.doSomethingAsynchronously();
 
		// 상태와 상호 작용을 검증한다.
		verify(mockDummyCollaborator, times(1)).doSomethingAsynchronously(
				any(DummyCallback.class));
		assertThat(dummyCaller.getResult(), is(equalTo(results)));
	}
{% endhighlight %}

## ArgumentCaptor 사용하기
두번째 방안은 [ArgumentCaptor](http://docs.mockito.googlecode.com/hg/org/mockito/ArgumentCaptor.html)을 사용하는 것이다. 여기서는 콜백을 비동기로 다룬다 : 우리는 ArgumentCaptor를 사용해 **DummyCallback** 객체를 잡아서(capture)  **DummyCollaborator**에게 전달한다.
결국 우리는 모든 assertions를 테스트 메소드 레벨에서 만들 수 있으며 상태와 상호 작용을 검증하기 위해 onSuccess()를 호출할 수 있다.

{% highlight java %}
	@Test
	public void testDoSomethingAsynchronouslyUsingArgumentCaptor() {
		// 테스트 대상인 메소드를 호출한다.
		dummyCaller.doSomethingAsynchronously();
 
		final List<String> results = Arrays.asList("One", "Two", "Three");
 
		// 콜백을 호출한다. ArgumentCaptor.capture()은 matcher처럼 동작한다.
		verify(mockDummyCollaborator, times(1)).doSomethingAsynchronously(
				dummyCallbackArgumentCaptor.capture());
 
		// 콜백이 호출되기 전의 상태를 assertion한다.
		assertThat(dummyCaller.getResult().isEmpty(), is(true));
 
		// 일단 만족한다면, callbackCaptor.getValue()의 응답을 동작시킨다.
		dummyCallbackArgumentCaptor.getValue().onSuccess(results);
 
		// 콜백이 호출된 후의 상태를 assertion한다.
		assertThat(dummyCaller.getResult(), is(equalTo(results)));
	}
{% endhighlight %}

## 결론
두 해결책들간의 주된 차이점은 DoAnswer()를 사용하였을 떄는 익명 클래스를 만들어야 하며 invocation.getArguments()[n]에서 요소들을 (안전하지 않은 방식으로) 우리가 원하는 데이터 타입으로 캐스팅해야 하지만, 무엇이 발생하였는지 알 수 있게 테스트가 '빨리 실패'하도록 파라미터를 변경할 수 있다. 반면 ArgumentCaptor의 경우 콜백을 우리가 필요한 상황에서 우리가 원하는 순서대로 콜백을 호출 할 수 있기 때문에 더 많은 제어를 가질 수 있을 것이다. 
유닛 테스트에 관심을 가지면, 종종 어떻게 처리햐여야 할 지 모르는 상황이 흔하므로 내 경험으론 비동기 메소드들을 테스트해야 할 때 두 해법을 모두 사용하는 것이 탄탄한 접근법을 가지는데 도움이 된다.
I hope you find this article useful, and as always, remember that any feedback is very welcome, as well as other ways of doing this. Of course if you have any doubt do not hesitate to contact me.

## Code Sample
[Here is the link where you can find this example and others](https://github.com/android10/Inside_Android_Testing). Most of them are related with Java and Android because this comes from a talk I gave a couple of months ago.
The presentation is in english but the video is in spanish (sorry for those who do not understand my argentinian accent…haha…BTW I will try to upload an english version as soon as possible…)

https://speakerdeck.com/android10/how-android-testing-changed-how-we-think-about-death

## Further Reading
프래임워크를 더 이해하기 위해 Mockito 문서를 보기를 강력하게 추천한다. 그 문서는 매우 명확하고, 좋은 예제들을 가지고 있다. See you!
