---
layout: blog
category: post
splash: ""
tags: ""
published: false
title: "espresso-4-Advanced-Samples"
---
[원본](https://google.github.io/android-testing-support-library/docs/espresso/advanced/index.html)

# ViewMatchers

## 다른 View 옆의 view에 매칭하기
레이아웃은 유일점이 없는 뷰들을 포함할 수 있다(예를 들어 주소록 목록에서 반복되는 전화 버튼은 view 계층에서 다른 전화 버튼들과 동일한 R.id, 동일한 문자열, 그리고 동일한 속성들을 가지고 있을 것이다).

예를 들어, 이 activity에서, "7"이라는 글자는 여러 열에서 반복된다.
![]({{site.baseurl}}/https://google.github.io/android-testing-support-library/docs/images/hasSibling.png)

종종, 유일하지 않은 view는 그 옆에 위치한 어떤 유일한 레이블과 쌍을 이룰 수 있다 (예를 들어 주소록의 전화 버튼 옆의 이름). 이 경우, 당신은 당신의 선택을 좁히기 위해 hasSibling matcher를 사용할 수 있다:

	onView(allOf(withText("7"), hasSibling(withText("item: 0"))))
      .perform(click());

## onData와 커스텀 ViewMatcher로 데이터 매칭하기

아래의 Activity는 [SimpleAdapter](http://developer.android.com/intl/ko/reference/android/widget/SimpleAdapter.html)
의 원조를 받는 ListView를 포함한다. SimpleAdapter는 각 행을 위한 데이터를 Map<String, Object>에 보유하고 있다. 각 map은 키 "STR"에 content(string, "item:x")이 들어있는 entry와 키 "LEN"에 content의 길이인 Interger가 들어있는 entry를 가진다.

![]({{site.baseurl}}/https://google.github.io/android-testing-support-library/docs/images/list_activity.png)

"item: 50"을 가진 열을 클릭하는 코드는 다음과 같다:

	onData(allOf(is(instanceOf(Map.class)), hasEntry(equalTo("STR"), is("item: 50")))
	  .perform(click());
  
onData안의 Matcher<Object>를 분해해보자:

	is(instanceOf(Map.class))

이는 AdapterView의 특정 항목이 Map인 경우로 검색을 좁힌다.

우리의 경우, 목록 view의 모든 열이 해당되지만 우리는 명확히 "item: 50"을 클릭하기를 원한다. 그래서 다음과 같이 검색을 더 좁힌다:

	hasEntry(equalTo("STR"), is("item: 50"))

이 Matcher<String, Object>는 key "STR"과 value "item: 50"인 entry를 가진 어떤 Map를 매치한다. 이 코드는 길고 다른 위치에서 재사용하고 싶기 때문에 - 커스텀 "withItemContent" matcher를 만들도록 하자.

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
    
Map클래스의 객체와만 매치할 수 있기를 원하기 때문에 BoundedMatcher를 기반으로 사용한다. We override the matchesSafely method, put in the matcher we found earlier and match it against a Matcher<String> that can be passed as an argument. 이는 우리가 withItemContent(equalTo("foo"))를 할 수 있게 해준다. 코드를 간결하게 하기 위해, equalTo를 미리 하고 String을 받는 다른 matcher를 만든다.


    public static Matcher<Object> withItemContent(String expectedText) {
      checkNotNull(expectedText);
      return withItemContent(equalTo(expectedText));
    }
    
이제 항목을 클릭하기 위한 코드는 간단하다:

	onData(withItemContent("item: 50")) .perform(click());
    
이 테스트의 전체 코드는 [AdapterViewTest#testClickOnItem50](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/AdapterViewTest.java)과 [custom matcher](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/LongListMatchers.java)을 보라.

# View의 특정 자식 view에 매칭하기

위 샘플은 ListView의 열 전체의 중간을 클릭하는 문제가 있다. 만약 우리가 열의 특정 자식에게 작업을 하고 싶으면 어떻게 해야하나? 예를 들어, LongListActivity의 열에 첫 행의 String.length을 표시하는 두번째 행을 클릭하고 싶다. (이를 덜 추상적으로 하자면, 당신은 G+앱이 댓글 목록을 보여주며 각 댓글의 옆에 +1 버튼이 있는 것을 생각해보라)

![]({{site.baseurl}}/https://google.github.io/android-testing-support-library/docs/images/item50.png)

이제 당신의 DataInteraction에 onChildView 명시를 추가하라:

    onData(withItemContent("item: 60"))
      .onChildView(withId(R.id.item_size))
      .perform(click());

Note: 이 예제는 위 샘플의 withItemConent matcher를 사용한다! [AdapterViewTest#testClickOnSpecificChildOfRow60](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/AdapterViewTest.java)를 보라!


Matching a view that is a footer/header in a ListView

Headers and footers are added to ListViews via the addHeaderView/addFooterView APIs. To load them using Espresso.onData, make sure to set the data object (second param) to a preset value. For example:

public static final String FOOTER = "FOOTER";
...
View footerView = layoutInflater.inflate(R.layout.list_item, listView, false);
((TextView) footerView.findViewById(R.id.item_content)).setText("count:");
((TextView) footerView.findViewById(R.id.item_size)).setText(String.valueOf(data.size()));
listView.addFooterView(footerView, FOOTER, true);
Then, you can write a matcher that matches this object:

import static org.hamcrest.Matchers.allOf;
import static org.hamcrest.Matchers.instanceOf;
import static org.hamcrest.Matchers.is;

@SuppressWarnings("unchecked")
public static Matcher<Object> isFooter() {
  return allOf(is(instanceOf(String.class)), is(LongListActivity.FOOTER));
}
And loading the view in a test is trivial:

import static com.google.android.apps.common.testing.ui.espresso.Espresso.onData;
import static com.google.android.apps.common.testing.ui.espresso.action.ViewActions.click;
import static com.google.android.apps.common.testing.ui.espresso.sample.LongListMatchers.isFooter;

public void testClickFooter() {
  onData(isFooter())
    .perform(click());
  ...
}
Take a look at the full code sample at: AdapterViewtest#testClickFooter

Matching a view that is inside an ActionBar

The ActionBarTestActivity has two different action bars: a normal ActionBar and a contextual action bar that is created from a options menu. Both action bars have one item that is always visible and two items that are only visible in overflow menu. When an item is clicked, it changes a TextView to the content of the clicked item.

Matching visible icons on both of the action bars is easy:

public void testClickActionBarItem() {
  // We make sure the contextual action bar is hidden.
  onView(withId(R.id.hide_contextual_action_bar))
    .perform(click());

  // Click on the icon - we can find it by the r.Id.
  onView(withId(R.id.action_save))
    .perform(click());

  // Verify that we have really clicked on the icon by checking the TextView content.
  onView(withId(R.id.text_action_bar_result))
    .check(matches(withText("Save")));
}


The code looks identical for the contextual action bar:

public void testClickActionModeItem() {
  // Make sure we show the contextual action bar.
  onView(withId(R.id.show_contextual_action_bar))
    .perform(click());

  // Click on the icon.
  onView((withId(R.id.action_lock)))
    .perform(click());

  // Verify that we have really clicked on the icon by checking the TextView content.
  onView(withId(R.id.text_action_bar_result))
    .check(matches(withText("Lock")));
}


Clicking on items in the overflow menu is a bit trickier for the normal action bar as some devices have a hardware overflow menu button (they will open the overflowing items in an options menu) and some devices have a software overflow menu button (they will open a normal overflow menu). Luckily, Espresso handles that for us.

For the normal action bar:

public void testActionBarOverflow() {
  // Make sure we hide the contextual action bar.
  onView(withId(R.id.hide_contextual_action_bar))
    .perform(click());

  // Open the overflow menu OR open the options menu,
  // depending on if the device has a hardware or software overflow menu button.
  openActionBarOverflowOrOptionsMenu(getInstrumentation().getTargetContext());

  // Click the item.
  onView(withText("World"))
    .perform(click());

  // Verify that we have really clicked on the icon by checking the TextView content.
  onView(withId(R.id.text_action_bar_result))
    .check(matches(withText("World")));
}


This is how this looks on devices with a hardware overflow menu button:



For the contextual action bar it is really easy again:

public void testActionModeOverflow() {
  // Show the contextual action bar.
  onView(withId(R.id.show_contextual_action_bar))
    .perform(click());

  // Open the overflow menu from contextual action mode.
  openContextualActionModeOverflowMenu();

  // Click on the item.
  onView(withText("Key"))
    .perform(click());

  // Verify that we have really clicked on the icon by checking the TextView content.
  onView(withId(R.id.text_action_bar_result))
    .check(matches(withText("Key")));
  }


See the full code for these samples: ActionBarTest.java.

ViewAssertions

Asserting that a view is not displayed

After performing a series of actions, you will certainly want to assert the state of the UI under test. Sometimes, this may be a negative case (for example, something is not happening). Keep in mind that you can turn any hamcrest view matcher into a ViewAssertion by using ViewAssertions.matches.

In the example below, we take the isDisplayed matcher and reverse it using the standard “not” matcher:

import static com.google.android.apps.common.testing.ui.espresso.Espresso.onView;
import static com.google.android.apps.common.testing.ui.espresso.assertion.ViewAssertions.matches;
import static com.google.android.apps.common.testing.ui.espresso.matcher.ViewMatchers.isDisplayed;
import static com.google.android.apps.common.testing.ui.espresso.matcher.ViewMatchers.withId;
import static org.hamcrest.Matchers.not;

onView(withId(R.id.bottom_left))
  .check(matches(not(isDisplayed())));
The above approach works if the view is still part of the hierarchy. If it is not, you will get a NoMatchingViewException and you need to use ViewAssertions.doesNotExist (see below).

Asserting that a view is not present

If the view is gone from the view hierarchy (e.g. this may happen if an action caused a transition to another activity), you should use ViewAssertions.doesNotExist:

import static com.google.android.apps.common.testing.ui.espresso.Espresso.onView;
import static com.google.android.apps.common.testing.ui.espresso.assertion.ViewAssertions.doesNotExist;
import static com.google.android.apps.common.testing.ui.espresso.matcher.ViewMatchers.withId;

onView(withId(R.id.bottom_left))
  .check(doesNotExist());
Asserting that a data item is not in an adapter

To prove a particular data item is not within an AdapterView you have to do things a little differently. We have to find the AdapterView we’re interested in and interrogate the data its holding. We don’t need to use onData(). Instead, we use onView to find the AdapterView and then use another matcher to work on the data inside the view.

First the matcher:

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
Then the all we need is an onView that finds the AdapterView:

@SuppressWarnings("unchecked")
public void testDataItemNotInAdapter(){
  onView(withId(R.id.list))
      .check(matches(not(withAdaptedData(withItemContent("item: 168")))));
  }
And we have an assertion that will fail if an item that is equal to “item: 168” exists in an adapter view with the id list.

For the full sample look at AdapterViewTest#testDataItemNotInAdapter.

Idling resources

Using registerIdlingResource to synchronize with custom resources

The centerpiece of Espresso is its ability to seamlessly synchronize all test operations with the application under test. By default, Espresso waits for UI events in the current message queue to process and default AsyncTasks* to complete before it moves on to the next test operation. This should address the majority of application/test synchronization in your application.

However, there are instances where applications perform background operations (such as communicating with web services) via non-standard means; for example: creation and management of threads directly and the use of custom services.

In such cases, the first thing we suggest is to put on your testability hat and ask whether the user of non-standard background operations is warranted. In some cases, it may have happened due to poor understanding of Android and the application could benefit from refactoring (for example, by converting custom creation of threads to AsyncTasks). However, sometimes refactoring is not possible. The good news? Espresso can still synchronize test operations with your custom resources.

Here’s what you need to do:

Implement the IdlingResource interface and expose it to your test.
Register one or more of your IdlingResource(s) with Espresso by calling Espresso.registerIdlingResource in test setup.
To see how IdlingResource can be used take a look at the AdvancedSynchronizationTest and the CountingIdlingResource class.

Note that the IdlingResource interface is implemented in your app under test so you need to add dependencies carefully:

// IdlingResource is used in the app under test
compile 'com.android.support.test.espresso:espresso-idling-resource:2.2.2'

// For CountingIdlingResource:
compile 'com.android.support.test.espresso:espresso-contrib:2.2.2'
Customization

Using a custom failure handler

Replacing the default FailureHandler of Espresso with a custom one allows for additional (or different) error handling - e.g. taking a screenshot or dumping extra debug information.

The CustomFailureHandlerTest example demonstrates how to implement a custom failure handler:

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
This failure handler throws a MySpecialException instead of a NoMatchingViewException and delegates all other failures to the DefaultFailureHandler. The CustomFailureHandler can be registered with Espresso in the setUp() of the test:

@Override
public void setUp() throws Exception {
  super.setUp();
  getActivity();
  setFailureHandler(new CustomFailureHandler(getInstrumentation().getTargetContext()));
}
For more information see the FailureHandler interface and Espresso.setFailureHandler.

inRoot

Using inRoot to target non-default windows

Surprising, but true - Android supports multiple windows. Normally, this is transparent (pun intended) to the users and the app developer, yet in certain cases multiple windows are visible (e.g. an auto-complete window gets drawn over the main application window in the search widget). To simplify your life, by default Espresso uses a heuristic to guess which Window you intend to interact with. This heuristic is almost always “good enough”; however, in rare cases, you’ll need to specify which window an interaction should target. You can do this by providing your own root window (aka Root matcher:

onView(withText("South China Sea"))
  .inRoot(withDecorView(not(is(getActivity().getWindow().getDecorView()))))
  .perform(click());
As is the case with ViewMatchers, we provide a set of pre-canned RootMatchers. Of course, you can always implement your own Matcher.

Take a look at the sample or the sample on GitHub.
그외 
http://www.vogella.com/tutorials/AndroidTestingEspresso/article.html
https://androidresearch.wordpress.com/2015/04/04/an-introduction-to-espresso/

