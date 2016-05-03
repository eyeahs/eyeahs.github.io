---
layout: blog
category: post
splash: ""
tags: ""
published: false
title: "Espresso #3 - Espresso Intents"
---

Espresso-Intents는 테스트 대상 어플리케이션에 의해 발송된 Intent들의 검증과 스터빙(stubbing)을 가능하게 하는 Espresso의 extension이다. 이는 Android Intent들을 위한 Mockito와 비슷하다.

# Download Espresso-Intents

* Android Supoort Repository를 설치하였는지 확인하라.
* 당신의 어플리케이션의 build.gradle 파일을 열어라. 보통 최상위 build.gradle 파일이 아니라 app/build.gradle이다.

다음 라인을 depedencies안에 추가한다:

    androidTestCompile 'com.android.support.test.espresso:espresso-intents:2.2.2'

Espresso-Intents는 Espresso 2.1+와 testing support library 0.3에서만 호환되므로 다음 라인들을 다음과 같이 업데이트했음을 확인한다:

    androidTestCompile 'com.android.support.test:runner:0.5'
    androidTestCompile 'com.android.support.test:rules:0.5'
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'

# IntentsTestRule
Espresso_Intents를 사용할 때 _ActivityTestRule_대신 _IntentsTestRule_을 사용한다. _IntentsTestRule_은 UI 기능 테스트에서 Espresso-Intents API들을 쉽게 사용할 수 있게 해준다. 이 클래스는 @Test로 어노테이션된 각각의 테스트 이전에 Espresso-Intents을 초기화하고 각 테스트가 실행된 후 Espresso_Intents를 릴리즈해주는 _ActivityTestRule_의 확장이다. Activity는 각각의 테스트 후에 종료될 것이고 이 규칙은 _ActivityTestRule_에서도 동일한 방식으로 사용될 수 있다.

# Intent 검증

Espresso-Intents는 테스트 대상 어플리케이션에서 activity들의 실행을 시도하는 모든 intents들을 저장한다. (Mockito.verify의 사촌인) Intented API을 사용하면, 당신은 주어진 intent가 보여졌는지를 assert 할 수 있다.

떠난 intent를 간단히 검증하는 예제 테스트:

@Test
public void validateIntentSentToPackage() {
    // 외부의 "phone" activity가 시작되는 결과를 만드는 사용자 액션
    user.clickOnView(system.getView(R.id.callButton));

    // "phone" Activity에 대한 resolving Intent가 발송되었는지 입증하기 위해
    // 미리 준비된 RecordedIntentMatcher를 사용한다.
    intended(toPackage("com.android.phone"));
}

## Intent stubbing

(Mockito.when의 사촌인) intending API를 사용하여, startActivityForResult로 시작된 Activity들에게 응답을 제공할 있다. (이는 특히 외부 activity들에게 유용한다. 우리는 외부 activity의 사용자 인터페이스를 조종 할 수도 없고 테스트 대상 activity에게 반환되는  ActivityResult를 통제할 수도 없기 때문이다):

intent stubbing 예제 테스트:

    @Test
    public void activityResult_IsHandledProperly() {
        // 특정 activity가 시작되었을 때 반환될 result를 만든다.
        Intent resultData = new Intent();
        String phoneNumber = "123-345-6789";
        resultData.putExtra("phone", phoneNumber);
        ActivityResult result = new ActivityResult(Activity.RESULT_OK, resultData);

        // Intent가 "contacts"에 보내 졌을 때 보여질 stubbing된 result를 설정한다.
        intending(toPackage("com.android.contacts")).respondWith(result));

        // "contacts" activity가 실행될 사용자 행위
        // Activity를 실행함으  phoneNumber가 반환되고 그것이 화면에 표시되는 것이 기대된다.
        onView(withId(R.id.pickButton)).perform(click());

        // 위에서 우리가 설정했던 데이터를 assert한다.
        onView(withId(R.id.phoneNumber).check(matches(withText(phoneNumber)));
    }

# Intent matchers

_intentding_과 _intented_ 메소드는 _Mathcer<Intent>_를 인수를 받는다. Hamcrest는 matcher 객체들의 라이브러리이다(_constraints_또는 _predicates_로도 알려져있다). 당신에게는 다음과 같은 선택지가 있다:

* 기존의 intent matcher를 사용: 거의 언제나 선호되는, 가장 쉬운 선택이다.
* 당신만의 intent matcher를 구현: 가장 유연한 선택이다 ([Hamcrest tutorial](https://code.google.com/archive/p/hamcrest/wikis/Tutorial.wiki)의 "Writing custom matcher" 제목의 섹션을 보라)

기존의 Intent matcher들로 intent 검증하는 예제:

    intended(allOf(
        hasAction(equalTo(Intent.ACTION_VIEW)),
        hasCategories(hasItem(equalTo(Intent.CATEGORY_BROWSABLE))),
        hasData(hasHost(equalTo("www.google.com"))),
        hasExtras(allOf(
            hasEntry(equalTo("key1"), equalTo("value1")),
            hasEntry(equalTo("key2"), equalTo("value2")))),
            toPackage("com.android.browser")));