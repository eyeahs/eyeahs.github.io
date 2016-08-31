# Introduction

http://akarnokd.blogspot.kr/2015/06/subjects-part-1.html

I have the feeling many would like to bury Subjects and I'm yet going to do a multi-post series about them.

몇몇은 Subject를 반응성 세계의 가변 상태로 간주하는데, 나는 이를 부정하지 않는다. 하지만 그들은 계속해서
Observable.create()를 사용하는 예를 좀 더 보여주면서 _Subject를 사용하지 마라_라고 말한다.

    Observable.create(s -> {
       int i = 0;
       while (true) {
           s.onNext(i++);
       }
    }).subscribe(System.out::println);

unsubscription과 backpressure는 어떻게 할것인가? 아무리 많은 operator들도 이 source를 위해 고칠 수가 없다. 하지만 당신은 언제든지 Subject에 onBackpressureXXX 전략을 적용할 수 있다.


Subjects are by no means bad or broken, but as with other components of the reactive paradigm, one must learn when and how to use them. For those who are lecturing about them, they should reconsider when and how they introduce Subjects to their audience. I suggest introducing them after introducing regular fluent operators but before talking about create().

In this series, I'm going to try and introduce Subjects, detail their requirements and structure and show how one can build its own custom Subject.