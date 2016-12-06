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
