---
layout: post
category: blog
title: '[번역]RxJava를 이용한  Dagger 2 비동기 주입'
date: '2016-12-10 22:00:00 +0900'
tags:
  - android, dagger, di
published: true
---
[원본](https://medium.com/@froger_mcs/async-injection-in-dagger-2-with-rxjava-e7df503343c0#.uaoor0y54)  - *First version of this post was originally written on my dev-blog:*[*http://frogermcs.github.io/*

몇 주 전 `Producers`를 사용한 Dagger2의 비동기 의존성 주입에 대한 [글을 작성](https://medium.com/@froger_mcs/dependency-injection-with-dagger-2-producers-c424ddc60ba3)하였다. 백그라운드 스레드에서 객체 초기화를 수행하면 큰 장점을 하나 가진다 -리얼 타임에서 UI를 그릴 책임이 있는 메인 스레드를 차단하지 않는다(부드러운 인터페이스를 유지하기 위한 [초당 60 프레임](https://www.youtube.com/watch?v=CaMTIgxCSqU)).

느린 초기화가 모든 사람들에게 이슈가 되는 것이 아님을 언급할 가치가 있다. 하지만 당신이 당신의 코드를 정말 신경더라도 `init()` 메소드나 생성자에서 외부 라이브러리가 디스크/네트워크 작업을 수행할 가능성은 항상 존재한다. 만약 그것에 대해 확신이 서지 않는다면 Android 성능 통계 라이브러리인 [AndroidDevMetrics](https://github.com/frogermcs/AndroidDevMetrics)를 사용해 보라. 애플리케이션의 특정 화면을 표시하기 위해 필요한 시간과 (만약 Dagger 2를 사용한다면) 의존성 그래프에서 각 객체를 제공하는데 소모된 시간이 얼마인지를 알려준다.

Producers는 불행히도 Android를 위해 디자인되지 않았고, 몇가지 결점들이 존재한다:

- Guava를 의존으로 사용한다 (64k 메소드 이슈를 일으키고 빌드 타임을 늘릴 수 있다)
- 매우 빠르지 않다 (주입 매커니즘은 장치에 따라 수십 밀리 초 동안 메인 스레드를 차단할 수 있다)
- `@Inject` 어노테이션을 사용하지 않는다 (코드를 좀 더 엉망으로 만든다)

마지막 두 가지에 대해서는 우리가 할 수 있는게 많지 않지만, 첫번째 것은 Android 프로젝트에서 Producers를 위태롭게 한다.

### RxJava를 사용한 비동기 주입

다행히도 많은 Android 개발자는 애플리케이션에서 비동기 코드를 만들기 위해 RxJava(와 [RxAndroid](https://github.com/ReactiveX/RxAndroid))를 사용한다. 이것을 사용하여 Dagger2 비동기 주입을 해보자.

### 비동기 @Singleton 주입

여기 우리의 묵직한 객체가 있다:

```java
@Provides
@Singleton
HeavyExternalLibrary provideHeavyExternalLibrary() {
    HeavyExternalLibrary heavyExternalLibrary = new HeavyExternalLibrary();
    heavyExternalLibrary.init(); //This method takes about 500ms
    return heavyExternalLibrary;
}
```

이제 이 코드를 비동기적으로 호출할 `Observable<HeavyExternalLibrary>` 객체를 반환하는 추가 `provide…()` 메소드를 만든다:

```java
@Singleton
@Provides
Observable<HeavyExternalLibrary> provideHeavyExternalLibraryObservable(final Lazy<HeavyExternalLibrary> heavyExternalLibraryLazy) {
    return Observable.create(new Observable.OnSubscribe<HeavyExternalLibrary>() {
        @Override
        public void call(Subscriber<? super HeavyExternalLibrary> subscriber) {
            subscriber.onNext(heavyExternalLibraryLazy.get());
        }
    }).subscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread());
}
```

이를 줄 단위로 분석해보자:

* `@Singleton` —`HeavyExternalLibrary` 이 아니라 `Observable` 객체의 단일 인스턴스라는 것을 명심해야 한다. 또한 Singleton 역시 추가 Observable 객체를 만들지 못하게 한다.
* `@Provides` —이 메소드는 `@Module` 어노테이션 클래스의 일부이기도 하다. [Dagger 2 API](http://frogermcs.github.io/dependency-injection-with-dagger-2-the-api/)를 기억하는가?
* `Lazy<HeavyExternalLibrary> heavyExternalLibrary` 객체는 Dagger 내부적으로 `HeavyExternalLibrary` 객체를 초기화하지 못하도록 합니다 (그렇지 않으면 `provideHeavyExternalLibraryObservable()`객체의 호출 순간에 이미 생성될 것이다).
* `Observable.create(…)` 코드 —는 이 Observable이 구독 될 때마다 `heavyExternalLibraryLazy.get()`을 호출하여  `heavyExternalLibrary` 객체를 반환한다.
* `.subscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread());` —기본적으로 RxJava 코드는 Observable이 생성 된 스레드에서 실행된다. 이것이 우리가 백그라운드 스레드(이 경우 **Schedulers.io()**)로 실행을 이동하고 메인 스레드 (**AndroidSchedulers.mainThread()**)에서 결과를 감시해야하는 이유이다.

우리 Observable은 다른 어떤 객체처럼 그래프에 주입된다. 그러나 객체 `heavyExternalLibrary` 그 자체는 나중에 사용할 수 있습니다:

```java
public class SplashActivity {

	@Inject
	Observable<HeavyExternalLibrary> heavyExternalLibraryObservable;

	//This will be injected asynchronously
	HeavyExternalLibrary heavyExternalLibrary; 

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate();
		//...
		heavyExternalLibraryObservable.subscribe(new SimpleObserver<HeavyExternalLibrary>() {
            @Override
            public void onNext(HeavyExternalLibrary heavyExternalLibrary) {
	            //Our dependency will be available from this moment
	            SplashActivity.this.heavyExternalLibrary = heavyExternalLibrary;
            }
        });
	}
}
```

### 비동기로 신규 인스턴스 주입

위 코드는 singleton 객체를 삽입하는 방법을 보여준다. 새로운 인스턴스를 비동기적으로 주입하려면 어떻게 해야 할까?

우리 객체가 더 이상 `@Singleton`이 아님을 확인하라:

```java
@Provides
HeavyExternalLibrary provideHeavyExternalLibrary() {
    HeavyExternalLibrary heavyExternalLibrary = new HeavyExternalLibrary();
    heavyExternalLibrary.init(); //This method takes about 500ms
    return heavyExternalLibrary;
}
```

우리의 `Observable<HeavyExternalLibrary>`는 싱글톤일 수 있지만 이것의 `subscribe()`를 호출할 때 마다, `onNext()` 호출에서 `HeavyExeteranlLibrary`의 새로운 인스턴스를 얻을 것이다:

```java
heavyExternalLibraryObservable.subscribe(new SimpleObserver<HeavyExternalLibrary>() {
    @Override
    public void onNext(HeavyExternalLibrary heavyExternalLibrary) {
        //New instance of HeavyExternalLibrary
    }
});
```

### 비동기 주입 완결

Dagger2에서 RxJava를 사용하여 비동기 주입을 수행할 수 있는 또 다른 방법이 있다. Observable로 전체 주입 절차를 간단히 마무리 할 수 있다. 우리의 주입이 다음 방식으로 수행된다고 가정 해 보자(코드는 GithubClient 예제 프로젝트에서 가져왔다):

```java
public class SplashActivity extends BaseActivity {

    @Inject
    SplashActivityPresenter presenter;
    @Inject
    AnalyticsManager analyticsManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }

    //This method is called in super.onCreate() method
    @Override
    protected void setupActivityComponent() {
        final SplashActivityComponent splashActivityComponent = GithubClientApplication.get(SplashActivity.this)
                .getAppComponent()
                .plus(new SplashActivityModule(SplashActivity.this));
        splashActivityComponent.inject(SplashActivity.this);
    }
}
```

이를 비동기로 만들기 위해서는 Observable로  `setupActivityComponent()` 메소드를 래핑하기만 하면 된다.

```java
@Override
protected void setupActivityComponent() {
    Observable.create(new Observable.OnSubscribe<Object>() {
        @Override
        public void call(Subscriber<? super Object> subscriber) {
            final SplashActivityComponent splashActivityComponent = GithubClientApplication.get(SplashActivity.this)
                    .getAppComponent()
                    .plus(new SplashActivityModule(SplashActivity.this));
            splashActivityComponent.inject(SplashActivity.this);
            subscriber.onCompleted();
        }
    })
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(new SimpleObserver<Object>() {
                @Override
                public void onCompleted() {
                    //Here is the moment when injection is done.
                    analyticsManager.logScreenView(getClass().getName());
                    presenter.callAnyMethod();
                }
            });
}
```

설명했듯이 `@Inject` 어노테이션된 모든 객체는 미래 어느 시점에 주입될 것이다. 대신에 주입 절치는 비동기식이고 메인 스레드에 큰 영향을 미치지 않는다.

물론 `Observable` 객체와 `subscribeOn()`에 대한 추가 스레드들을 생성하는 것은 완전히 무료가 아니다 —약간의 시간이 걸리 것이다. 이는 `Producers` 코드에서 생성된 영향과 거의 비슷하다.

Thanks for reading!
