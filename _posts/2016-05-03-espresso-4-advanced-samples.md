---
layout: post
category: blog
published: true
title: "espresso-4-Advanced-Samples"
date: "2016-05-08 10:50:00 +09:00"
---
[원본](https://google.github.io/android-testing-support-library/docs/espresso/advanced/index.html)

# ViewMatchers

## 다른 view 옆의 view에 연결하기
레이아웃은 유일한 값이 없는 뷰들을 가질 수 있다(예를 들어 주소록 목록에서 반복되는 전화 버튼은 view 계층에서 다른 전화 버튼들과 동일한 R.id, 동일한 문자열, 그리고 동일한 속성들을 가질 수 있다).

예를 들어, 이 activity에서, "7"이라는 텍스트는 여러 열에서 반복된다.
![]({{site.baseurl}}https://google.github.io/android-testing-support-library/docs/images/hasSibling.png)

가끔은 유일하지 않은 view는 그 옆에 위치한 어떤 유일한 레이블과 쌍을 이룰 수 있다(예를 들어 주소록의 전화 버튼 옆의 이름). 이런 경우, 선택을 좁히기 위해 hasSibling matcher를 사용할 수 있다:

	onView(allOf(withText("7"), hasSibling(withText("item: 0"))))
      .perform(click());

## onData와 커스텀 ViewMatcher로 데이터에 연결하기

아래의 Activity는 [SimpleAdapter](http://developer.android.com/intl/ko/reference/android/widget/SimpleAdapter.html)
의 도움을 받는 ListView를 포함한다. SimpleAdapter는 Map<String, Object>에 각 행을 위한 데이터를 가지고 있다. 각 map은 키 "STR"에 content(string, "item:x")를 가지는 entry와 키 "LEN"에 content의 length인 Interger를 가지는 entry를 가진다.

![]({{site.baseurl}}/https://google.github.io/android-testing-support-library/docs/images/list_activity.png)

"item: 50"을 가진 열을 클릭하는 코드는 다음과 같다:

	onData(allOf(is(instanceOf(Map.class)), hasEntry(equalTo("STR"), is("item: 50")))
	  .perform(click());
  
onData내부의 Matcher<Object>를 분리해서 보자:

	is(instanceOf(Map.class))

이는 AdapterView의 항목들 중 Map인 경우로 검색을 좁힌다.

우리의 경우, 목록 view의 모든 열이 해당되지만 우리는 명확히 "item: 50"을 클릭하기를 원한다. 그래서 다음과 같이 검색을 더 좁힌다:

	hasEntry(equalTo("STR"), is("item: 50"))

이 Matcher<String, Object>는 key가 "STR"이고 value는 "item: 50"인 entry를 포함하는 어떤 Map과 연결할 것이다. 이 검색을 위한 코드는 길다. 그리고 우리는 이 코드를 다른 위치에서도 재사용하고 싶기 때문에 - 이를 위한 커스텀 "withItemContent" matcher를 만들도록 하자.

    @SuppressWarnings("rawtypes")
    public static Matcher<Object> withItemContent(final Matcher<String> itemTextMatcher) {
      // 테스트가 잘못된 matcher를 생성하였을 때 빠르게 실패하기 위한 전제 조건을 사용한다.
      checkNotNull(itemTextMatcher);
      return new BoundedMatcher<Object, Map>(Map.class) {
        @Override
        public boolean matchesSafely(Map map) {
          return hasEntry(equalTo("STR"), itemTextMatcher).matches(map);
        }

        @Override
        public void describeTo(Description description) {
          description.appendText("with item content: ");
          itemTextMatcher.describeTo(description);
        }
      };
    }
    
Map 클래스의 객체에 일치 할 수 있도록 BoundedMatcher를 기반으로 한다. matchesSafely 메소드를 오버라이드한 뒤 인수로 전달받는 Matcher<String>를 이전에 찾은 matcher에 넣고 이를 비교해본다. 이는 우리가 withItemContent(equalTo("foo"))를 할 수 있게 해준다. 코드를 간결하게 하기 위해, equalTo를 미리 수행한 String을 받는 다른 matcher를 만들자.
    public static Matcher<Object> withItemContent(String expectedText) {
      checkNotNull(expectedText);
      return withItemContent(equalTo(expectedText));
    }
    
이제 항목을 클릭하기 위한 코드는 간단하다:

	onData(withItemContent("item: 50")).perform(click());
    
이 테스트의 전체 코드는 [AdapterViewTest#testClickOnItem50](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/AdapterViewTest.java)과 [custom matcher](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/LongListMatchers.java)을 보라.

## View의 특정 자식 view와 일치시키기

위 샘플은 ListView의 열 전체의 중앙을 클릭하는 문제가 있다. 만약 우리가 열의 특정한 자식에게 작업을 하고 싶으면 어떻게 해야할까? 예를 들어, LongListActivity의 열 내부에 있는 첫 행의 String.length을 표시하는 두번째 행을 클릭하고 싶다. (이를 덜 추상적으로 말하자면, 당신은 G+앱이 댓글 목록을 보여주며 각 댓글의 옆에 +1 버튼이 있는 것을 생각해보라)

![]({{site.baseurl}}/https://google.github.io/android-testing-support-library/docs/images/item50.png)

이제 당신의 DataInteraction에 onChildView 명시를 추가하라:

    onData(withItemContent("item: 60"))
      .onChildView(withId(R.id.item_size))
      .perform(click());

Note: 이 예제는 위 샘플의 withItemConent matcher를 사용한다! [AdapterViewTest#testClickOnSpecificChildOfRow60](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/AdapterViewTest.java)를 보라!

## Matching a view that is a footer/header in a ListView

Header들과 Footer들은 ListView에 addHeaerView/addFooterView API를 통해 추가된다. 이들을 Espresoo.onData를 사용하여 로드하기 위해서는 데이터 객체 (두번째 파라미터)를 preset value로 추가해야 한다. 예를 들어:

    public static final String FOOTER = "FOOTER";
    ...
    View footerView = layoutInflater.inflate(R.layout.list_item, listView, false);
    ((TextView) footerView.findViewById(R.id.item_content)).setText("count:");
    ((TextView) footerView.findViewById(R.id.item_size)).setText(String.valueOf(data.size()));
    listView.addFooterView(footerView, FOOTER, true);

그러면 이 객체에 일치시키는 matcher를 작성할 수 있다:

    import static org.hamcrest.Matchers.allOf;
    import static org.hamcrest.Matchers.instanceOf;
    import static org.hamcrest.Matchers.is;

    @SuppressWarnings("unchecked")
    public static Matcher<Object> isFooter() {
      return allOf(is(instanceOf(String.class)), is(LongListActivity.FOOTER));
    }

그리고 테스트에서 view를 로드하는 것은 간단하다:

    import static com.google.android.apps.common.testing.ui.espresso.Espresso.onData;
    import static com.google.android.apps.common.testing.ui.espresso.action.ViewActions.click;
    import static com.google.android.apps.common.testing.ui.espresso.sample.LongListMatchers.isFooter;

    public void testClickFooter() {
      onData(isFooter())
        .perform(click());
      ...
    }
    
전체 코드 샘플은 [AdapterViewtest#testClickFooter](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/AdapterViewTest.java)을 보라.

## ActionBar 내부의 view와 일치시키기

[ActionBarTestActivity](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/main/java/android/support/test/testapp/ActionBarTestActivity.java)는 두 개의 다른 action bar들을 가진다 : 일반적인 ActionBar와 [options menu](http://developer.android.com/intl/ko/guide/topics/ui/menus.html#options-menu)에서 생성된 Contextual Action bar이다. 두 action bar들은 언제나 visible한 항목 하나와 overflow 메뉴에서만 항상 visible한 항목 두 개를 가진다. 항목이 클릭되면, 이는 TextView를 클릭된 항목의 내용으로 변경한다.

두 action bar에 모두 있는 visible 아이콘의 매칭은 쉽다:

    public void testClickActionBarItem() {
      // Contexual action bar가 숨겨지도록 한다.
      onView(withId(R.id.hide_contextual_action_bar))
        .perform(click());

      // 아이콘을 클릭한다 - 이것은 r.Id를 통해 찾을 수 있다.
      onView(withId(R.id.action_save))
        .perform(click());

      // TextView의 내용을 체크하여 아이콘이 실제로 클릭되었는 지를 검증한다.
      onView(withId(R.id.text_action_bar_result))
        .check(matches(withText("Save")));
    }

![]({{site.baseurl}}/https://google.github.io/android-testing-support-library/docs/images/actionbar_normal_icon.png)

contextual action bar를 위한 코드도 동일하게 보인다:

public void testClickActionModeItem() {
  // Contextual action bar가 표시되도록 한다.
  onView(withId(R.id.show_contextual_action_bar))
    .perform(click());

  // 아이콘을 클릭한다.
  onView((withId(R.id.action_lock)))
    .perform(click());

        // TextView의 내용을 체크하여 아이콘이 실제로 클릭되었는 지를 검증한다.
  onView(withId(R.id.text_action_bar_result))
    .check(matches(withText("Lock")));
}

![]({{site.baseurl}}/https://google.github.io/android-testing-support-library/docs/images/actionbar_contextual_icon.png)

Overflow 메뉴의 항목을 클릭하는 것은 일반 action bar보다 약간 다루기 힘들다. 어떤 단말은 하드웨어 overflow 메뉴 버튼(options menu에 overflowing item을 열 것이다)을 가지고 있고 어떤 단말은 소프트웨어 overflow menu button(일반 overflow menu를 열것이다)가지고 있기 때문이다. 운이 좋게도, Espresso는 우리를 위해 이를 처리한다.

일단 Action bar에서는:

public void testActionBarOverflow() {
      // Contexual action bar가 숨겨지도록 한다.
  onView(withId(R.id.hide_contextual_action_bar))
    .perform(click());

  // 단말이 hardware 또는 software overflow 메뉴 버튼을 가지고 있냐에 따라
  // Overflow 메뉴 또는 options 메뉴를 연다.
  openActionBarOverflowOrOptionsMenu(getInstrumentation().getTargetContext());

  // 항목 클릭
  onView(withText("World"))
    .perform(click());

  // TextView의 내용을 체크하여 아이콘이 실제로 클릭 되었는지를 검증한다.
  onView(withId(R.id.text_action_bar_result))
    .check(matches(withText("World")));
}

![]({{site.baseurl}}/https://google.github.io/android-testing-support-library/docs/images/actionbar_normal_hidden_overflow.png)

하드웨어 overflow 메뉴 버튼을 가진 단말에서는 이렇게 보여진다:

![]({{site.baseurl}}/https://google.github.io/android-testing-support-library/docs/images/actionbar_normal_hidden_no_overflow.png)

Contextual action bar를 위해서도 정말 매우 쉽다:

public void testActionModeOverflow() {
  // Contextual action bar를 보여준다.
  onView(withId(R.id.show_contextual_action_bar))
    .perform(click());

  // contextual action mode를 위해 option menu를 연다.
  openContextualActionModeOverflowMenu();

  // 항목을 클릭한다.
  onView(withText("Key"))
    .perform(click());

  // TextView의 내용을 체크하여 아이콘이 실제로 클릭 되었는지를 검증한다.
  onView(withId(R.id.text_action_bar_result))
    .check(matches(withText("Key")));
  }

![]({{site.baseurl}}/https://google.github.io/android-testing-support-library/docs/images/actionbar_contextual_hidden.png)

이 셈플들의 전체 코드를 보라 : [ActionBarTest.java](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/ActionBarTest.java)

# ViewAssertions

## 표시되지 않은 view를 assert하기

일련의 행위들을 수행한 뒤에는 테스트 대상 UI의 상태를 assert하고 싶을 것이다. 가끔은 이는 negative case일 수도 있다 (예를 들어, '어떤 것이 일어나지 않았다'). 어떤 hamcrest view matcher라도 ViewAssertions.matcher를 이용해 ViewAssertion으로 바꿀 수 있음을 잊지마라.

아래 예제에서, 우리는 isDisplayed matcher를 가지고 표준 "not" matcher를 사용하여 역으로 만든다:

    import static com.google.android.apps.common.testing.ui.espresso.Espresso.onView;
    import static com.google.android.apps.common.testing.ui.espresso.assertion.ViewAssertions.matches;
    import static com.google.android.apps.common.testing.ui.espresso.matcher.ViewMatchers.isDisplayed;
    import static com.google.android.apps.common.testing.ui.espresso.matcher.ViewMatchers.withId;
    import static org.hamcrest.Matchers.not;

    onView(withId(R.id.bottom_left))
      .check(matches(not(isDisplayed())));
      
위의 접근법은 view가 여전히 계층의 일부이라면 동작할 것이다. 그렇지 않다면, 당신은 NoMatchingViewException을 받을 것이고 ViewAssertions.doesNotExist를 사용할 필요가 있다(아래를 보라).

## 존재하지 않는 view를 assert하기

만약 view가 view 계층에서 사라지면 (예를 들어, 행위가 다른 activity로 이동함을 야기할 경우 발생할 것이다), ViewAssertions.doesNotExist를 사용하여야 한다:

    import static com.google.android.apps.common.testing.ui.espresso.Espresso.onView;
    import static com.google.android.apps.common.testing.ui.espresso.assertion.ViewAssertions.doesNotExist;
    import static com.google.android.apps.common.testing.ui.espresso.matcher.ViewMatchers.withId;

    onView(withId(R.id.bottom_left))
      .check(doesNotExist());

## Adapter안에 없는 데이터를 assert하기

특정 데이터 항목이 AdapterView안에 존재하지 않는 다는 것을 증명하기 위해서는 조금 다른 일을 해야 한다. 우리가 관심을 가지는 AdapterView를 찾고 그것이 가지고 있는 데이터를 심문한다. onData()를 사용할 필요가 없다. 대신 AdapterView를 찾기 위해 onView를 사용한 뒤 view 내부의 데이터에서 작업을 하기 위해 다른 matcher를 사용한다.

첫번째 matcher:

    private static Matcher<View> withAdaptedData(final Matcher<Object> dataMatcher) {
      return new TypeSafeMatcher<View>() {

        @Override
        public void describeTo(Description description) {
          description.appendText("with class name: ");
          dataMatcher.describeTo(description);
        }

        @Override
        public boolean matchesSafely(View view) {
          if (!(view instanceof AdapterView)) {
            return false;
          }
          @SuppressWarnings("rawtypes")
          Adapter adapter = ((AdapterView) view).getAdapter();
          for (int i = 0; i < adapter.getCount(); i++) {
            if (dataMatcher.matches(adapter.getItem(i))) {
              return true;
            }
          }
          return false;
        }
      };
    }
    
그 뒤 우리에게 필요한 것은 AdapterView를 찾기 위한 onView뿐이다:

    @SuppressWarnings("unchecked")
    public void testDataItemNotInAdapter(){
      onView(withId(R.id.list))
          .check(matches(not(withAdaptedData(withItemContent("item: 168")))));
      }
      
그러면 R.id.list인 adapter view에 존재하는 항목이 "item: 168"과 동일하면 실패하게 되는 assertion을 가지게 된다.

For the full sample look at [AdapterViewTest#testDataItemNotInAdapter](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/AdapterViewTest.java).

# Idling resources

## 커스텀 resources와 동조하는 registerIdlingResource 사용하기

Espresso의 가장 중요한 특징은 모든 테스트 작업과 테스트 대상 어플리케이션이 끊어짐 없이 동조하는 능력이다. 기본적으로 Espresso는 다음 테스트 작업으로 이동하기 전에 현재 메시지 큐 내부의 UI 이벤트가 처리되고 기본적인 AsyncTask가 종료되기를 기다린다. This should address the majority of application/test synchronization in your application.

하지만 어플리케이션에서 (웹 서비스와 커뮤니케이션같이) 비-표준 수단을 통한 백그라운드 작업을 수행하는 경우가 있다; 예를 들어; 직접 스레드들을 생성, 관리하고 커스텀 서비스를 사용.

In such cases, the first thing we suggest is to put on your testability hat and ask whether the user of non-standard background operations is warranted. In some cases, it may have happened due to poor understanding of Android and the application could benefit from refactoring (for example, by converting custom creation of threads to AsyncTasks). However, sometimes refactoring is not possible. The good news? Espresso can still synchronize test operations with your custom resources.

우리가 해야 할 일이 여기 있다:

* [IdlingResource](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/idling-resource/src/main/java/android/support/test/espresso/IdlingResource.java) 인터페이스를 구현한 다음 이를 당신의 테스트에 노출하라.
* [Espresso.registerIdlingResource](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/core/src/main/java/android/support/test/espresso/Espresso.java)를 test setup에서 호출하여 하나 또는 그 이상의 IdlingResource(s)를 Espresso에 등록하라

어떻게 IdlingResource를 사용하는지를 보기 위해서 [AdvancedSynchronizationTest](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/AdvancedSynchronizationTest.java) 과 [CountingIdlingResource](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/contrib/src/main/java/android/support/test/espresso/contrib/CountingIdlingResource.java) 클래스를 보라.

IdlingResource 인터페이스는 당신의 테스트 대상 어플리케이션에 구현되어야 함으로 당신은 조심스럽게 의존을 추가할 필요가 있다:

    // IdlingResource is used in the app under test
    compile 'com.android.support.test.espresso:espresso-idling-resource:2.2.2'

    // For CountingIdlingResource:
    compile 'com.android.support.test.espresso:espresso-contrib:2.2.2'

# Customization

## 커스텀 failure handler 사용하기

Espresso의 기본 FailureHandler를 커스텀으로 변경하면 추가적(또는 다른) 에러 처리가 가능한다 - 예를 들어, 스크린샷을 찍거나 추가적인 디버그 정보를 dumping하기

[CustomFailureHandlerTest](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/CustomFailureHandlerTest.java?autodive=0%2F%2F%2F%2F) 예제는 어떻게 커스텀 failure handler를 구현하는 지를 보여준다:

    private static class CustomFailureHandler implements FailureHandler {
      private final FailureHandler delegate;

      public CustomFailureHandler(Context targetContext) {
        delegate = new DefaultFailureHandler(targetContext);
      }

      @Override
      public void handle(Throwable error, Matcher<View> viewMatcher) {
        try {
          delegate.handle(error, viewMatcher);
        } catch (NoMatchingViewException e) {
          throw new MySpecialException(e);
        }
      }
    }

이 failure handler는 NoMatchingViewException대신 MySpecialException을 던지고 모든 다른 failure들은 DefaultFailureHandler에 위임한다. CustomFailureHandler는 테스트의 setup()에서 Espresso에 등록될 수 있다:

    @Override
    public void setUp() throws Exception {
      super.setUp();
      getActivity();
      setFailureHandler(new CustomFailureHandler(getInstrumentation().getTargetContext()));
    }
    
For more information see the [FailureHandler](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/core/src/main/java/android/support/test/espresso/FailureHandler.java) interface and [Espresso.setFailureHandler](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/core/src/main/java/android/support/test/espresso/Espresso.java).

# inRoot

## Using inRoot to target non-default windows

Surprising, but true - Android supports multiple windows. Normally, this is transparent (pun intended) to the users and the app developer, yet in certain cases multiple windows are visible (e.g. an auto-complete window gets drawn over the main application window in the search widget). To simplify your life, by default Espresso uses a heuristic to guess which Window you intend to interact with. This heuristic is almost always “good enough”; however, in rare cases, you’ll need to specify which window an interaction should target. You can do this by providing your own root window (aka Root matcher:

    onView(withText("South China Sea"))
      .inRoot(withDecorView(not(is(getActivity().getWindow().getDecorView()))))
      .perform(click());

As is the case with ViewMatchers, we provide a set of pre-canned RootMatchers. Of course, you can always implement your own Matcher.

Take a look at the sample or the sample on GitHub.

그외 
http://www.vogella.com/tutorials/AndroidTestingEspresso/article.html
https://androidresearch.wordpress.com/2015/04/04/an-introduction-to-espresso/
