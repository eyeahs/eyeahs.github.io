---
layout: post
category: blog
title: '[번역]System Services는 System만의 것이 아니다.'
date: '2016-11-09 22:00:00 +0900'
categories: Android
tags:
  - Android, Dagger2
published: true
---
[원본](https://medium.com/@theMikhail/system-services-are-not-just-for-the-system-ce33aab4594a#.281es4t7w)

Dagger와 Custom View를 사용하면서 흥미로운 동작을 마주쳤다. 대부분의 경우, 나의 앱은 2개의 Component를 가진다. 싱글톤을 위한 최상위 레벨 AppComponent와 “Activity당 하나”이여야 하는 다른 것들을 위한 Activity Component가 있다. 나의 Activity들 안에는 일반적인 Dagger의 boilerplate가 존재한다:

	activityComponent = application.getAppComponent()
		.plusActivityComponent(new ActivityModule(this));

추가적으로 나는 activity component를  getter로 노출하는 것을 선호한다. 그래서 View들은 activity scoped 객체에 접근할 수 있다:

	public interface ActivityComponentProvider {
    	Activitycomponent getActivityComponent();
	}

View안에서는 이렇게 할 수 있다:

	public VrView(Context context, AttributeSet attrs, int defStyleAttr) {
      super(context, attrs, defStyleAttr);
      inflate(getContext(), R.layout.vr_view_contents, this);
	  ((ActivityComponentProvider)context)
    	.getActivityComponent().inject(this);
	...
    
대부분의 경우 이는 잘 동작한다. 하지만 어제는 대부분의 경우에 해당하지 않았다. 이 글은 내가 <android.support.design.widget.AppBarLayout> tag를 포함하는 xml layout을 만들면서 시작되었다. 나는 그 내부에 커스텀 뷰 <com.sample.myview>를 넣었다. 위 예제와 비슷하게 주입 코드를 view의 생성자에 추가한 뒤 코드를 실행했더니 크래쉬가 발생했다. 놀랍게도  “java.lang.ClassCastException: android.view.ContextThemeWrapper cannot be cast to android.app.Activity” 에러가 발생했다. 조금 뒤져보니, 그 문제는 AppCompat이 view를 inflate하는 방법에서 문제가 발생함을 추정할 수 있었다. 일반적인 widget들이나 layout group들과는 달리 <AppBarLayout>의 자식들은 자식 view들를 inflate하기 위해 포함된 Activity의 context 대신 ContextWrapper를 사용한다.
억지 기법brute force은 activity를 얻을 때 까지 view의 context에  `getBaseContext()`를 계속 반복하는 것이다. 하지만 이는 이상하고 hacky해 보인다. 대신 나는 Dagger Component를 activity에서 view로 전달하는 흥미로운 방법으로 향하였다(Jake Wharton 감사!)
이 기발한 접근법은 activity의 `getSystemService`를 오버라이드하는 것을 지렛대로 사용한다. 예제를 살펴보자.
먼저 당신의 activity에 `getSystemService`를 오버라이드 해야 한다.

	@Override
	public Object getSystemService(String name) {
		return super.getSystemService(name);
	}

 `context.getSystemService`은 보통 `LayoutInflator`같은 것들을 참조하기 위해 사용된다. 당신의 activity에 이 메소드를 오버라이드하면, 이제 당신은 특정 키를 사용하여 당신이 원하는 어떠한 인스턴스라도 돌려받을 수 있게 된다. 우리의 경우에는 Activity component를 돌려받기를 원한다.

    @Override
    public Object getSystemService(String name) {
        if("Dagger".equals(name)){
            return activityComponent;
        }
        return super.getSystemService(name);
    }

만약 우리가 원하는 키가 아닌 경우 super로 호출을 해야함을 주의하라.
이제 `getContext().getSystemService(key)` 의 호출을 통해 view의 생성자 내부에서 activity component에 접근할 수 있다. 이 설정의 멋진 점은 context에서 activity를 찾기 위해 당신이 쌓여진 것들을 벗길 필요 없이, 당신을 위해 시스템이 이를 해줄 것이라는 것이다.

    public VrView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        inflate(getContext(), R.layout.vr_view_contents, this);
        ActivityComponent activityComponent = (ActivityComponent) getContext().getSystemService("Dagger");
    activitycomponent.inject(this);

`getContext` 를 당신의 activity로 캐스트하는 위험 대신, 당신은 안드로이드 시스템 서비스 아키텍처를 지렛대로 사용할 수 있다.
마지막으로 Android Studio의 lint를 행복하게 만들기 위해

    ActivityComponent activityComponent = (ActivityComponent) getContext().getSystemService("Dagger");

위에 

	//noinspection ResourceType

을 추가하라.
이것이 다다! 당신은 이제 view에서 당신의 activity component의 참조를 얻을 수 있다. 보너스: 비슷한 방식으로 당신은 Application 클래스의 `getSystemService` 를 오버라이드하고 여기서 app component를 전달할 수 있다. 이는 당신이 당신의 Activity에서 다음과 같은 것을 할 수 있게 해줄 것이다.

	AppComponent component = (AppComponent) 	getAppplicationContext().getSystemService("AppComponent");

Thanks again to those that helped me grok this yesterday.
