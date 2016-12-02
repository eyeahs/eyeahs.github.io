---
layout: post
category: blog
title: '[번역]Presenter에게 모든 생명주기 이벤트가 필요한 것은 아니다.'
date: '2016-12-02 18:40:00 +0900'
categories: Android
tags:
  - Android
published: true
---
MVP는 Android 개발의 새로운 유행이며 대략 10억개의 방법이 있다. 흔히 반복되는 실수 중 하나는 개발자가 presenter에 너무 많은 Activity/Fragment 생명주기 이벤트를 포함시켜 View와 Presenter 계층간의 분리를 파괴하는 것이다. MVP가 우리에게 주는 이점, 즉 테스트 가능하고 민첩하며, 관심사의 분리는 UI와 Presenter계층이 서로 분리 된 상태를 유지해야한 달성할 수 있다. 나의 개발자 동무, 당신은 안드로이드에 구속되지 않는 Presenter를 지키기 위해 최선을 다해야 합니다.

## 나의 제안이 여기 있다

당신의 Presenter에는 단 두 개의 생명주기 관련 메소드만이 존재하도록 하자. 하나는 onViewAttached()이고 다른 하나는 onViewDetached()이다.

이는 다음과 같을 것이다:

    public interface Presenter {
      void onViewAttached(MVPView view); 
      void onViewDetached();
    }

모든 Presenter가 이 인터페이스를 implement하고 이들 메소드를 차례로 구현해야 한다.

## 좋다, 하지만 View는 언제 이들을 호출해야 하는가?

나는 onStart와 onStop 동안 onViewAttached와 onViewDetached를 각각 호출하는 것을 선호한다. 이렇게 하면 onViewAttached가 호출될 때 UI는 완전히 상호 작용할 준비가 되어 있을 것이다. 

그 모습은 다음과 같을 것이다:

    public class BaseActivity extends AppCompatActivity { 
      @Override 
      protected void onStart() { 
        super.onStart();
        presenter.onViewAttached(this);
      }
    @Override protected void onStop() { 
        super.onStop();
        presenter.onViewDetached();
      } 
    }

당신의 모든 Activity들은 이 클래스를 상속 받고 이 메소드들을 공짜로 호출할 수 있다. Fragment들은 비슷한 BaseFragment같은 것을 만들고 이를 상속받을 수 있다.

## 상태 저장과 복원은 어떻게 하는가?

여기에는 두 가지 방안이 있다:

**View가 스스로 상태를 관리하도록 한다:** 이 경우 당신의 Presenter는 어떠한 추가적 메소드들도 필요하지 않고 View는 상태 복원을 자기 자신에서 다룰 것이다.

**Presenter에서 상태를 관리한다:** 여기서, 당신의 BasePresenter안에 두개의 추가적인 메소드를 가질 수 있다 :  onSaveState와 onRestoreState이며 각각 onSaveInstanceState와 onRestoreInstanceState에서 호출 될 것이다. 당신은 Bundle 객체를 이 메소드들에 직접 전달할 수 있다. 이는 우리의 "Presenter에는 Android가 없다" 규칙을 깨트릴 것이다. 하지만 Mockito를 사용하여 테스트 중에는 Bundle을 mock하는 방법등으로 큰 문제없이 처리할 수 있다. 또 다른 방법은 bundle을 전혀 사용하지 않고 다른 곳(DB, File, SharedPrefs등)에 상태를 저장할 수 있다. 하지만 이는 복구가 더 느릴 수 있고 더 이상 필요하지 않을 때는 저장된 상태를 적절하게 청소해야 한다. 구현과 테스트가 더 편리한 방안으로 선택하도록 하라.

## 그런데 내 Activity가 새로운 것인지 아닌지 알고 싶다!

당신의 View가 새로운 인스턴스인지 아니면 재생성되기 이전의 것이지 알아야 하는 경우가 있다. 이는 특정 View 인스턴스가 처음으로 생성된 경우에만 어떤 일을 할 필요가 있는 상황에서 유용할 것이다. 이를 위하, onViewAttached 메소드에 boolean 파라미터 isNew를 추가할 수 있다.

따라서 우리의 Presenter 인터페이스는 다음처럼 될 것이다:

    public interface Presenter { 
      void onViewAttached(MVPView view, boolean isNew);
      void onViewDetached();
    }

그리고 우리의 BaseActivity는 이렇게 된다:

    public class BaseActivity extends AppCompatActivity { 
      private boolean isNewActivity;
    @Override
      protected void onCreate(Bundle savedInstanceState) {
        super.onCreate();
        
        // Check if it's a new view
        isNewActivity = (savedInstanceState == null);
      } 
      
      @Override protected void onStart() { 
        super.onStart();
        presenter.onViewAttached(this, isNewActivity);
        isNewActivity = false;
      } 
     
      @Override protected void onStop() { 
        super.onStop();
        presenter.onViewDetached();
      }
    }

이는 기본적으로 Activity의 새로운 인스턴스 생성 여부를 계속 알고 있기 위함이며 Presenter의 결속때마다 이 값을 전달한다.

## 나는 생명주기 이벤트가 더 많이 필요한 유스케이스를 가지고 있다!

좋다. 그렇다면 추가하라. 이 두 개의 이벤트가 당신의 세상의 모든 유스케이스를 다룰 수 있을 거라고 주장하는 것이 아니며, 당신은 때때로 더 많은 이벤트가 필요할 것이다. 그러나, 당신이 이를 필요로 할 때, 깊이 주의하고 지나치지는 않았는지 조심하도록 하라.

## 결론

MVP- 좋다

Presenter안의 너무 많은 생명주기 이벤트 - 나쁘다


### 추가 Comment

Vinay B. N : 내 생각에는 isFinishing()을 파라미터로 detachView(isFinishing())에 전달하면 네트워크 작업이 구성 변경에도 살아남도록 유지하는 것 같은 유스케이스들까지 커버할 수 있을 것이다.
