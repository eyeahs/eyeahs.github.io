Retrofit과 Mockito로 API 테스트하기

원본 : [Reliable API testing for Android with Retrofit and Mockito](http://mdswanson.com/blog/2013/12/16/reliable-android-http-testing-with-retrofit-and-mockito.html)

API와 상호 작용하는 HTTP 호출을 테스트하는 것은 언제나 어렵고 다루기 싫은 일이다. 실제 웹 서버에 접속하게 되면 많은 이슈들- 부서지기 쉬운 테스트(인터넷이나 API가 다운되어 생기는 테스트 실패)이자 불완전한 테스트("Rate limit 초과 케이스를 어떻게 유발시켜야 할까? 이것이 동작했으면 좋겠는데..") -이 발생한다.
이 이슈는 안드로이드처럼 HTTP 호출이 비동기여야 하는 플랫폼에서 더 한층 복잡해진다. 이제 당신은 혼란스럽게 타이밍을 추가하고, 아마 당신의 API 호출 테스트에 타올을 던질 준비가 되었을 것이다.

이런 이슈들을 해결하고 HTTP 호출들을 믿음직하게 사용하는 방법은 Mockito(자바를 위한 Test double 라이브러리)에 포함된 멋진 유틸리티인 ArgumentCaptor를 사용하는 것이다.
ArgumentCaptor는 하이브리드 Test double의 한 종류이다; 이는 약간 stub이고 약간 spy이지만 완전히 같지는 않다. 인수(argument) captor는 -놀랍지 않게- mock/stub에 전달된 인수들을 붙잡고 저장하는데 사용한다. 여기서의 진정한 승리는 붙잡은 인수로 메소드들을 호출하는 기능이다. 이는 Retrofit의 콜백같은 곳에서 엄청나게 잘 동작한다.
우리는 Retrofit으로 API 호출을 만들고 콜백을 제공한다. 이 라이브러리는 서버가 응답하였을 때 응답 데이터를 넘기면서 콜백을 실행한다.
유저 리파지토리를 조회하는 GitHub API가 있다고 하자.

{% highlight java %}
getApi().repositories("swanson", new Callback<List<Repository>>() {

@Override
public void success(List<Repository> repositories, Response response) {
if (repositories.isEmpty()) {
displaySadMessage();
}

mAdapter.setRepositories(repositories);
}

@Override
public void failure(RetrofitError retrofitError) {
displayErrorMessage();
}
});
{% endhighlight %}

여기에 우리가 테스트하고 싶은 세가지 케이스가 있다 : the happy path (우리는 몇몇 리파지토리들을 받았고 이를 우리의 어뎁터에 넘겨줬다), the error path (어떤 서버 에러가 있었고, 유저에게 토스트 메시지를 보여 준다), 그리고 a special case (유저는 리파지토리를 가지고 있지 않으며, 유저에게 토스트 메시지를 보여 준다)
두 번째와 세 번째 경우는 당신이 실제 API 서버에 접속하는 경우 테스트하기 어려울 것이다. 내가 알기로 Github는 최근 DDOS 이슈들이 있지만 당신의 에러 케이스를 테스트할 때 그것에 의지할 수는 없다.
하지만 ArgumentCaptor을 사용하면 콜백의 인수를 가로채서 우리가 보낸 데이터를 완전하게 통제할 수 있게 된다.
Happy path를 테스트하는 것을 보자. (나는 Robolectric을 사용하며 아마 당신도 그럴 것이다.)

{% highlight java %}
Mockito.verify(mockApi).repositories(Mockito.anyString(), cb.capture());

List<Repository> testRepos = new ArrayList<Repository>();
testRepos.add(new Repository("rails", "ruby", new Owner("dhh")));
testRepos.add(new Repository("android", "java", new Owner("google")));

cb.getValue().success(testRepos, null);

assertThat(activity.getListAdapter()).hasCount(2);
{% endhighlight %}

우리의 captor (cb)는 콜백을 잡은 뒤, getValue()를 호출한 다음, success 메소드를 호출 할 수 있으며 여기에 어떤 더미 객체들을 전달 할 수 있다.
당신안 아마 지금 이순간 "아하!"라고 했을 것이다. 뭐, 아니여도 괜찮다. Error path 테스트를 보자.

{% highlight java %}
Mockito.verify(mockApi).repositories(Mockito.anyString(), cb.capture());

cb.getValue().failure(null);

assertThat(ShadowToast.getTextOfLatestToast()).contains("Failed");
{% endhighlight %}

이전과 동일하다 - 우리는 콜백을 잡았다. 하지만 이번에는 우리는 API 에러를 가장하여 failure 메서드를 호출할 것이다. 만약 우리가 좀 더 개별적인 에러 처리가 필요하다면 (예를 들어 응답이 HTTP 400이며 로그인으로 리다이렉트; HTTP 500이면 토스트로 일반 시스템 에러 메시지를 띄우기.), 우리는 쉽게 적절한 RetrofitError 객체를 만들고 그것을 넘겨 줄 수 있다.
ArgumentCaptor의 힘은 여기서 빛을 발한다. 우리는 우리가 잡은 객체를 완전히 지배 할 수 있다. 우리는 우리가 원하는 어떤 데이터라도 먹일 수 있고 어떤 오류 조건도 발생시킬 수 있다.
번영을 위해, 특별한 경우를 테스트하자.

{% highlight java %}
Mockito.verify(mockApi).repositories(Mockito.anyString(), cb.capture());

List<Repository> noRepos = new ArrayList<Repository>();

cb.getValue().success(noRepos, null);

assertThat(ShadowToast.getTextOfLatestToast()).contains("No repos :(");
assertThat(activity.getListAdapter()).isEmpty();
{% endhighlight %}

(당신은 저 예제들의 완전한 소스와 완전한 예제 앱을 GitHub에서 찾을 수 있다.)
주목 할 만한 특별한 디테일이 있다. 만약 당신이 captor를 명시할 때  Mockito 어노테이션을 사용하면,

{% highlight java %}
@Captor
private ArgumentCaptor<Callback<List<Repository>>> cb;
{% endhighlight %}

당신의 setup 어딘가에서 확실히 다음을 해야 한다 :

{% highlight java %}
MockitoAnnotations.initMocks(this);
{% endhighlight %}

This approach to testing hits all the marks in my book: fast, robust, and easy to work with. It has allowed us to easily test rare edge cases (session timeout, server down for maintenance, extraordinary values) in my current project and achieve a high level of confidence that our app is working.
While this example is specific to a certain stack (Android, Robolectric, Retrofit, Mockito), a similar approach can be applied to nearly any application.
Happy testing!
