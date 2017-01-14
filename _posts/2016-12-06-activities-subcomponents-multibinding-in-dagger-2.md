---
layout: post
category: blog
title: '[번역]Activities Subcomponents Multibinding in Dagger 2'
date: '2016-12-07 15:35:00 +0900'
tags:
  - anroid, dagger, di
published: true
comments: true
---

[원본:Activities Subcomponents Multibinding in Dagger 2](http://frogermcs.github.io/activities-multibinding-in-dagger-2/)

몇 달 전 MCE³ 컨퍼런스에서 그레고리 킥(Gregory Kick)은 프레젠테이션에서 Subcomponent(e.g. to Activities)들을 제공하는 새로운 컨셉을 보여 주었다. 새로운 접근 방식은 (Activity들의 Subcomponent들의 팩토리를 사용하던) AppComponent 객체 참조가 없어도 ActivitySubcomponent를 만드는 방법을 제공한다. 이것을 현실로 만들기 위해 Dagger의 새로운 버전인 2.7 버전을 기다려야 했다.

## 문제
Dagger 2.7 버전 이전에는, Subcomponent(예를 들어 `AppComponent`의 Subcomponent인 `MainActivityComponent`)를 만들기 위해서는 그것의 Factory를 부모 Component에 선언해야 했다:

	AppComponent.java
    
	@Singleton
    @Component(
        modules = {
            AppModule.class
        }
    )
    public interface AppComponent {
        MainActivityComponent plus(MainActivityComponent.ModuleImpl module);

        //...
    }

이 선언 덕분에 Dagger는  `MainActivityComponent`가 `AppComponent`에 의존을 가진다는 것을 알게된다.

이렇게 하면, `MainActivity`로의 주입은 다음과 같을 것이다:

	ActivityComponent.java
    
    @Override
    protected ActivityComponent onCreateComponent() {
        ((MyApplication) getApplication()).getComponent().plus(new MainActivityComponent.ModuleImpl(this));
        component.inject(this);
        return component;
    }

이 코드의 문제점들은:

* Activity가 `AppComponent`에 의존한다(`((MyApplication)getApplication()).getComponent())`)-우리가 Subcomponent를 만들고자 할 때 마다, 우리는 부모 Component 객체에 접근해야 한다.
* `AppComponent`는 모든 Subcomponent들(과 그들의 builder들)의 factory들의 선언을 가지고 있어야 한다.

## Modules.subcomponents
Dagger 2.7부터 Subcomponent의 부모를 선언하는 새로운 방법이 생겼다. `@Module` 어노테이션은 이 모듈이 설치된 Component의 자식들인 Subcomponent 클래스들의 리스트를 가지는 optional [`subcomponents`](http://google.github.io/dagger/api/2.7/dagger/Module.html#subcomponents--) 필드를 가진다.

### 예제:
	ActivityBindingModule.java
    
	@Module(
            subcomponents = {
                    MainActivityComponent.class,
                    SecondActivityComponent.class
            })
    public abstract class ActivityBindingModule {
        //...
    }
    
`ActivityBindingModule`은 `AppComponent`에 설치된다. 이는 `MainActivityComponent`와 `SecondActivityComponent` 둘 다 `AppComponent`의 Subcomponent임을 의미한다.
이런 방식으로 선언된 Subcomponent들은 (이 포스트의 첫 코드 목록처럼)`AppComponet`에 명시적으로 선언될 필요가 없다.

## Activities Multibinding
이제 `Modules.subcomponents`를 사용하여 어떻게 Activity Multibinding을 만들고 Activity에 AppComponent 객체를 전달하는 것을 제거하는지를 보자(이는 이 [프리젠테이션](https://www.youtube.com/watch?v=iwjXqRlEevg&feature=youtu.be&t=1693)도 역시 설명하고 있다). 코드에서 가장 중요한 부분만을 살펴 볼 것이다. 전체 구현은 Github: [Dagger2Recipes-ActivitiesMultibing](https://github.com/frogermcs/Dagger2Recipes-ActivitiesMultibinding)에서 볼 수 있다.

우리의 앱은 두 개의 간단한 화면을 포함한다 : `MainActivity`와 `SecondActivity`. 우리는 `AppComponent`객체를 전달하지 않고 둘 모두에게 Subcomponent들을 제공할 수 있기를 원한다.

이제 모든 ActivityComponent 빌더를 위한 base interface를 만들어보자:

	ActivityComponentBuilder.java
    
	public interface ActivityComponentBuilder<M extends ActivityModule, C extends ActivityComponent> {
        ActivityComponentBuilder<M, C> activityModule(M activityModule);
        C build();
    }
    
Subcomponent 예제: `MainActivityCOmponent`는 다음과 같을 것이다:

	MainActivityComponent.java
    
    @ActivityScope
    @Subcomponent(
            modules = MainActivityComponent.MainActivityModule.class
    )
    public interface MainActivityComponent extends ActivityComponent<MainActivity> {

        @Subcomponent.Builder
        interface Builder extends ActivityComponentBuilder<MainActivityModule, MainActivityComponent> {
        }

        @Module
        class MainActivityModule extends ActivityModule<MainActivity> {
            MainActivityModule(MainActivity activity) {
                super(activity);
            }
        }
    }
    
이제 각 Activity 클래스들을 위해 만들어진 builder를 얻을 수 있도록 Subcomponent 빌더들의 맵을 가지고 싶다. 이를 위해 Multibinding 기능을 사용하도록 하자.

	ActivityBindingModule.java
    
    @Module(
            subcomponents = {
                    MainActivityComponent.class,
                    SecondActivityComponent.class
            })
    public abstract class ActivityBindingModule {

        @Binds
        @IntoMap
        @ActivityKey(MainActivity.class)
        public abstract ActivityComponentBuilder mainActivityComponentBuilder(MainActivityComponent.Builder impl);

        @Binds
        @IntoMap
        @ActivityKey(SecondActivity.class)
        public abstract ActivityComponentBuilder secondActivityComponentBuilder(SecondActivityComponent.Builder impl);
    }
    
`ActivityBindingModule`은 `AppComponent`에 설치된다. 설명처럼, `MainActivityComponent`와 `SecondActivityComponent`는 `AppComponent`의 Subcomponent가 될 것이다.

이제 우리는 `Subcomponents` 빌더의 맵을 주입할 수 있다.(예를 들어 `MyAppliation` 클래스에):

	MyApplication.java
    
	public class MyApplication extends Application implements HasActivitySubcomponentBuilders {

      @Inject
      Map<Class<? extends Activity>, ActivityComponentBuilder> activityComponentBuilders;

      private AppComponent appComponent;

      public static HasActivitySubcomponentBuilders get(Context context) {
          return ((HasActivitySubcomponentBuilders) context.getApplicationContext());
      }

      @Override
      public void onCreate() {
          super.onCreate();
          appComponent = DaggerAppComponent.create();
          appComponent.inject(this);
      }

      @Override
      public ActivityComponentBuilder getActivityComponentBuilder(Class<? extends Activity> activityClass) {
          return activityComponentBuilders.get(activityClass);
      }
	}
    
추가적인 추상을 가지기 위해 `HasActivitySubcomponentBuilders` 인터페이스를 만든다 (빌더들의 `Map`은 `Application`클래스에 주입될 수 없기 때문이다):

    public interface HasActivitySubcomponentBuilders {
        ActivityComponentBuilder getActivityComponentBuilder(Class<? extends Activity> activityClass);
    }
    
그리고 Activity 클래스에 주입하는 마지막 구현은:

	MainActivity.java
	
    public class MainActivity extends BaseActivity {

        //...

        @Override
        protected void injectMembers(HasActivitySubcomponentBuilders hasActivitySubcomponentBuilders) {
            ((MainActivityComponent.Builder) hasActivitySubcomponentBuilders.getActivityComponentBuilder(MainActivity.class))
                    .activityModule(new MainActivityComponent.MainActivityModule(this))
                    .build().injectMembers(this);
        }
    }
    
이는 우리의 제일 첫번째 구현과 매우 유사하지만, 앞서 언급했듯, 가장 중요한 점은 `ActivityComponent`객체를 더 이상 Activity에 전달하지 않는다는 것이다.

## 사용 사례 - instrumentation tests mocking
Loose coupling과 고정된 순환 의존성fixed circular dependency(Activity <-> Application)은 특히 작은 프로젝트나 팀에서는 항상 큰 문제이지는 않으나, 우리의 구현이 도움이 될 수 있는 실제 사용 사례를 고려해보도록 하자 -instrumentation test의 dependency mocking.

현재 Android Instrumentation Test에서 dependency mocking를 하는 가장 잘 알려진 방법 중 하나는 [DaggerMock](https://medium.com/@fabioCollini/android-testing-using-dagger-2-mockito-and-a-custom-junit-rule-c8487ed01b56#.pr1gg69ar)([Github project link](https://github.com/fabioCollini/DaggerMock))를 사용하는 것이다. DaggerMock이 강력한 도구이긴 하지만, 이것이 내부적으로 어떻게 동작하는지 이해하는 것은 꽤 어렵다. 다른 것들 중에서는 추적하기 쉽지않는 리플랙션 코드가 존재하기도 한다.

Subcomponent를 AppComponent 클래스에 접근하지 않고 Activity에서 직접 생성하면 우리 앱의 나머지 부분과 분리된 모든 단일 Activity를 테스트 할 방법이 생긴다.
멋진 것 같으니, 이제 코드를 살펴보도록 하자.

Instrumentation 테스트에서 사용되는 Application 클래스:
	
	ApplicationMock.java
        
	public class ApplicationMock extends MyApplication {

        public void putActivityComponentBuilder(ActivityComponentBuilder builder, Class<? extends Activity> cls) {
            Map<Class<? extends Activity>, ActivityComponentBuilder> activityComponentBuilders = new HashMap<>(this.activityComponentBuilders);
            activityComponentBuilders.put(cls, builder);
            this.activityComponentBuilders = activityComponentBuilders;
        }
    }

`putActivityComponentBuilder()` 메소드는 주어진 Activity Class의 ActivityComponentBuilder의 구현을 교체할 수 있는 방법을 제공한다.

이제 Espresso Instrumentation Test의 예를 보자:

	rawMainActivityUITest.java 
    
    @RunWith(AndroidJUnit4.class)
    public class MainActivityUITest {

        @Rule
        public MockitoRule mockitoRule = MockitoJUnit.rule();

        @Rule
        public ActivityTestRule<MainActivity> activityRule = new ActivityTestRule<>(MainActivity.class, true, false);

        @Mock
        MainActivityComponent.Builder builder;
        @Mock
        Utils utilsMock;

        private MainActivityComponent mainActivityComponent = new MainActivityComponent() {
            @Override
            public void injectMembers(MainActivity instance) {
                instance.mainActivityPresenter = new MainActivityPresenter(instance, utilsMock);
            }
        };

        @Before
        public void setUp() {
            when(builder.build()).thenReturn(mainActivityComponent);
            when(builder.activityModule(any(MainActivityComponent.MainActivityModule.class))).thenReturn(builder);

            ApplicationMock app = (ApplicationMock) InstrumentationRegistry.getTargetContext().getApplicationContext();
            app.putActivityComponentBuilder(builder, MainActivity.class);
        }

        @Test
        public void checkTextView() {
            String expectedText = "lorem ipsum";
            when(utilsMock.getHardcodedText()).thenReturn(expectedText);

            activityRule.launchActivity(new Intent());

            onView(withId(R.id.textView)).check(matches(withText(expectedText)));
        }

    }
    
단계별로:

* `MainActivityComponent.Builder`와 Mocking되어야 하는 모든 의존(이 경우 `Utils`만)들의 Mock을 제공한다. 우리의 Mock된 `Builder`는 `MainActivityPresenter`에 Mocking된 Utils 객체를 주입하는 `MainActivityComponent`의 커스텀 구현을 반환한다.
* 그다음 우리의 `MainActivityComponent.Builder`는 `MyApplication`(line 28)에 주입된 Builder의 원본을 교체한다 : `app.putActivityComponentBuilder(builder, MainActivity.class);`
* 마지막 테스트 - 우리는 `Utils.getHardcodedText()` 메소드를 mocking한다. Activity가 생성될 때 주입이 수행된다:`activityRule.launchActivity(new Intent());`. 그 다음에는 우리는 Espresso로 결과를 체크한다.

이것이 전부이다. 거의 모든 것이 `MainActivityUITest`클래스에서 일어나며 코드는 매우 간단하고 이해하기 쉽다.

### Source code

직접 구현을 테스트하고 싶다면 Instrumentation Test에서 Activity Multibinding 및 Mock Dependencies를 생성하는 방법을 보여주는 예제가 있는 소스 코드를 Github: [Dagger2Recipes-ActivitiesMultibinding](https://github.com/frogermcs/Dagger2Recipes-ActivitiesMultibinding)에서 볼 수 있다.


Thanks for reading!
