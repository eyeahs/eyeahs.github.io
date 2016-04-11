---
published: false
---

# How To Use RxJava
* Subscribe : 구독
* Observable / emit : Observable이 발행하는

# Hello World!
다음의 "Hellow World"를 자바로 구현한 샘플은 스트링의 목록에서 Observable을 만든 뒤, 이 Observable을 Observable가 발행하는 각각의 스트링을 위해 "Hello _String_"을 출력하는 메소드로 구독(subscribe)한다.

{% highlight java %}
  public static void hello(String... names) {
      Observable.from(names).subscribe(new Action1<String>() {
  
          @Override
          public void call(String s) {
              System.out.println("Hello " + s + "!");
          }
  
      });
  }
{% endhighlight %}

{% highlight java %}
hello("Ben", "George");
Hello Ben!
Hello George!
{% endhighlight %}

# How to Design Using RxJava
RxJava를 사용하기 위해서 Observables(데이터 항목들을 발행)를 만들고, 이 Observable들을 변형한다 / 여러 방법으로 / 당신이 흥미를 가지는 (Observable operator를 사용하여)적황한 데이터 항목
