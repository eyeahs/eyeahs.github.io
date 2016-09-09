# 

[원본](http://blog.danlew.net/2015/06/22/loading-data-from-multiple-sources-with-rxjava/)

내가 네트워크에서 조회한 `데이터`를 가지고 있다고 가정하자. 나는 이 데이터가 필요할 때 마다 단순히 매번 네트워크 조회를 할 수도 있지만, 이 데이터를 디스크와 메모리에 캐쉬한다면 훨씬 효율적으로 될 것이다.

좀 더 구체적으로 말하자면, 나는 이런 장치를 원한다:

1. 가끔은 신규 데이터를 위해 네트워크에서 조회한다.
2. 그렇지 않으면 가능한 빨리 데이터를 검색한다.

나는 이 장치의 구현을 RxJava를 사용하여 보여주고 싶다.

##  기본 패턴

각 소스(네트워크, 디스크, 메모리)에 대해 `Observable`이 주어진다면, 두 operator, `concat()` 과 `first()`를 사용하여 간단한 해결법을 만들 수 있다.

[`concat()`](http://reactivex.io/documentation/operators/concat.html)은 다수의  `Observables`를 받아서 그들을 차례로 연쇄시킨다. [`first()`](http://reactivex.io/documentation/operators/first.html)는 일련의 연속에서 오직 첫번째 항목만을 발행한다. 그러므로 만약 당신이  `concat().first()`을 사용한다면 이는 다중 소스에서 처음으로 발행된 항목을 검색한다.

Let's see it in action:

```
// 우리의 소스들 (독자들을 위해 연습 문제로 남겨둔다.)
Observable<Data> memory = ...;  
Observable<Data> disk = ...;  
Observable<Data> network = ...;

// 첫번째 소스의 데이터를 가지고 온다.
Observable<Data> source = Observable  
  .concat(memory, disk, network)
  .first();
```

**The key to this pattern is that concat() only subscribes to each child Observable when it needs to.** 만약 데이터가 캐쉬되어 있다면 더 느린 소스들을 검새할 필요가 없다. 그렇기 때문에 `first()`는 연쇄를 중단할 것이다. 다시 말해, 만약 `memory`가 결과를 반환하면, `disk`나 `network`를 괴롭히지 않을 것이다. 거꾸러 말하면 `memory`나 `disk` 둘 다 데이터를 가지고 있지 않다면, 이는 새로운 네트워크 요청을 만들 것이다.

 `concat()`내부의 소스 `Observables` 들의 순서를 고려해야 함을 주의하라.  `concat()`은 소스 `Observables` 들을 차례차례로 조사하기 때문이다.

## 데이터 저장하기.

다음 과정은 뻔하게도 들어온 소스들을 저장하는 것이다. 만약 네트워크 요청의 결과를 디스크에 저장하지 않거나, 디스크 요청을 메모리에 캐쉬하지 않았다면, 어떠한 절약도 절대 볼 수 없을 것이다! 위 코드는 거듭해서 네트워크 요청을 만들 것이다.

나의 해결책은 각 소스가 그들이 방출하는 데이터를 저장하게 하는 것이다:

```
Observable<Data> networkWithSave = network.doOnNext(data -> {  
  saveToDisk(data);
  cacheInMemory(data);
});

Observable<Data> diskWithCache = disk.doOnNext(data -> {  
  cacheInMemory(data);
});
```

이제, 당신이 `networkWithSave`와 `diskWithCache`를 사용하면 데이터는 자동으로 당신이 조회한 대로 저장될 것이다.

(이 전략의 다른 장점은 `networkWithSave`/`diskWithCache`를 우리의 다중 소스 패턴 내부만이 아니라 어디서든 사용할 수 있다는 것이다.)

## 한물간 데이터

불행히도 이제 우리의 데이터-저장 코드는 지나치게 너무 잘 동작한다. 이 코드는 데이터가 얼마나 낡았는지는 상관없이 언제나 동일한 데이터를 반환한다. 우리가 가끔은 신선한 데이터를 위해 서버에 돌아가고 싶다고 했던 것을 기억하자.

해결책은 `first()`에 있다. 이것 역시 거르기(filter)를 할 수 있다. 당장 자격이 없는 데이터는 거부하도록 설정하자:

```
Observable<Data> source = Observable  
  .concat(memory, diskWithCache, networkWithSave)
  .first(data -> data.isUpToDate());
```

이제는 최신임이 검증된 첫번째 항목만을 발행할 것이다. 그러므로 만약 우리의 소스들 중 하나가 한물한 `데이터`를 가지고 있다면 새로운 `데이터`를 찾기 위해 다음 소스에서 계속 하게 될 것이다.

## first() vs. takeFirst()

이 패턴을 위해 `first()`를 사용하는 대신 [`takeFirst()`](http://reactivex.io/RxJava/javadoc/rx/Observable.html#takeFirst(rx.functions.Func1))를 사용할 수도 있다.

두 호출간의 차이점은 `first()`는 어떤 소스도 올바른 데이터를 발행하지 않을 때 `NoSuchElementException` 예외를 던진다. 반면 `takeFirst()`는 예외없이 단순히 종료된다.

무엇을 사용할지는 당신이 데이터 부족을 명시적으로 처리할 지 말지에 달려있다.

Which you use depends on whether you need to explicitly handle a lack of data or not.

## Code Samples

Here's an implementation of the above code which you can check out here:[https://github.com/dlew/rxjava-multiple-sources-sample](https://github.com/dlew/rxjava-multiple-sources-sample)

If you'd like a real-world example, check out [the Gfycat app](https://github.com/dlew/android-gfycat), which uses this pattern [when retrieving Gfycat metadata](https://github.com/dlew/android-gfycat/blob/6154ab4bf056a080a0d4fbc69c63594d2b3a4387/Gfycat/src/main/java/net/danlew/gfycat/service/GfycatService.java#L61-L76). The code doesn't use all the capabilities shown above (since it doesn't need it), but it demonstrates the basic `concat().first()` setup.