---
layout: post
category: blog
published: false
title: "Espresso - Custom Idling Resource"
---
[원본](http://blog.sqisland.com/2015/04/espresso-custom-idling-resource.html)

Espresso의 주요 기능 중 하나는 테스트 작업과 테스트되는 어플리케이션간의 동기화이다. 이는 idling이라는 개념을 기반으로 이루어졌다: Espresso는 다음 동작을 수행하고 다음 assertion을 검증하기 전에 어플리케이션이 "idle"이 되기를 기다린다.

## Idle

당신의 어플리케이션이 Idle이 된다는 것의 의미는 무엇일까? Espresso는 여러 조건을 검증한다.

* 현재 message queue에 UI 이벤트들이 존재하지 않는다.
* 기본 AsyncTask 스레드풀에 작업이 존재하지 않는다.

이는 UI 렌더링과 AsyncTask 완료를 기다릴 책임이 있다. 하지만, 만약 당신의 어플리케이션이 다른 방식으로 오랫 동안 동작하는 작업을 수행한다면, Espresso는 그 작업들의 종료를 기다리는 방법을 모른다. 이런 경우 당신은 커스텀 [IdlingResource](http://developer.android.com/intl/ko/reference/android/support/test/espresso/IdlingResource.html)를 작성하여 Espresso가 대기하도록 할 수 있다.

## IntentServiceIdlingResource

당신이 어떤 긴 계산을 하는 _IntentService_를 사용하고 이는 broadcast를 통해 당신의 activity에게 결과를 반환한다고 하자. 우리는 결과가 올바르게 표시되었는지를 검증하기 전에 Espresso가 결과를 대기하기를 원한다.

_IdlingResource_를 구현하기 위해, 당신은 3가지 함수를 오버라이드 해야한다 : _getName()_, _registerIdleTransitionCallback()_ 그리고 _isIdleNow()_.

    @Override
    public String getName() {
      return IntentServiceIdlingResource.class.getName();
    }

    @Override
    public void registerIdleTransitionCallback(
        ResourceCallback resourceCallback) {
      this.resourceCallback = resourceCallback;
    }

    @Override
    public boolean isIdleNow() {
      boolean idle = !isIntentServiceRunning();
      if (idle && resourceCallback != null) {
        resourceCallback.onTransitionToIdle();
      }
      return idle;
    }

    private boolean isIntentServiceRunning() {
      ActivityManager manager = 
        (ActivityManager) context.getSystemService(
          Context.ACTIVITY_SERVICE);
      for (ActivityManager.RunningServiceInfo info : 
              manager.getRunningServices(Integer.MAX_VALUE)) {
        if (RepeatService.class.getName().equals(
              info.service.getClassName())) {
          return true;
        }
      }
      return false;
    }

Idle 로직은 _isIdleNow()_안에 구현되어 있다. 우리의 경우, 우리는 _ActivityManager_에게 _IntentService_가 동작하고 있는지 문의한다. 만약 _IntentService_가 동작하고 있지 않으면 우리는 _resourceCallback.onTransitionToIdle()_을 호출하여 Espresso에게 알려준다. 

# Idling resource 등록하기

Espresso가 이를 대기하기 위해서 당신의 커스텀 idling resource을 등록할 필요가 있다. 당신의 테스트의 _@Before_ 메소드에 이것을 하고, _@After_에서 해지하라.

    @Before
    public void registerIntentServiceIdlingResource() {
      Instrumentation instrumentation 
        = InstrumentationRegistry.getInstrumentation();
      idlingResource = new IntentServiceIdlingResource(
        instrumentation.getTargetContext());
      Espresso.registerIdlingResources(idlingResource);
    }

    @After
    public void unregisterIntentServiceIdlingResource() {
      Espresso.unregisterIdlingResources(idlingResource);
    }
    
    Full example


# Full example
Check out the source code for a full example. Try commenting out the IdlingResource registration and watch the test fail.

Source code: https://github.com/chiuki/espresso-samples/ under idling-resource-intent-service

Like this article? Take a look at the outline of my Espresso book and fill in this form to push me to write it! Also check out the published courses: https://gumroad.com/chiuki
