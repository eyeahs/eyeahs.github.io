---
layout: post
category: blog
published: false
title: Activities Subcomponents Multibinding in Dagger 2
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

