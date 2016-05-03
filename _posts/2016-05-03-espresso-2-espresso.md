---
layout: blog
category: blog
splash: ""
tags: ""
published: false
title: "Espresso #2 - Espresso 기초"
---
Espresso API는 테스트 작성자에게 사용자가 어플리케이션과 상호 작용 할 때 무엇을 할 것인지 대해 생각하도록 격려한다 - UI 요소들을 위치에 두고 그들과 상호작용하는 것. 동시에 이 프레임워크는 어플리케이션의 Activity들과 View들에 직접 접근하는 것을 막는다. UI 스레드에서 시작된 이 객체들을 잡고 작업하는 것이 비정상적인 테스트의 주된 원인이기 때문이다. 그러므로, 당신은 Espresso API에서 getView나 getCurrentActivity같은 메소드들을 볼 수 없을 것이다. 하지만 당신은 여전히 당신의 _ViewAction_들과 _ViewAssertions_들을 구현함을 통해 View에서 안전하게 작업을 할 수 있다.

다음은 Espresso의 주요 구성 요소들의 개요이다:

* **Espresso** – View들과의 상호 작용의 진입점(onView와 onData를 통해서). 또한 어떤 뷰와도 묶일 필요가 없는 APIs들을 노출한다(pressBack).
* **ViewMatchers** – _Matcher<? super View>_ 인터페이스를 구현한 객체들의 모임. 현재 View 계층 내에 위치한 view의 위치를 찾기 위해 하나 또는 그 이상의 ViewMatchers를 _onView_ 메소드에게 전달할 수 있다.
* **ViewActions** – _ViewInteraction.perform()_ 메소드에게 전달할 수 있는 _ViewAction들의_ 모임 (예를 들어, click())
* **ViewAssertions** – _ViewAssertion_들의 모임 
A collection of ViewAssertions that can be passed the ViewInteraction.check() method. Most of the time, you will use the matches assertion, which uses a View matcher to assert the state of the currently selected view.

Example:

    onView(withId(R.id.my_view))      // withId(R.id.my_view) is a ViewMatcher
      .perform(click())               // click() is a ViewAction
      .check(matches(isDisplayed())); // matches(isDisplayed()) is a ViewAssertion

# Finding a view with onView

In the vast majority of cases, the onView method takes a hamcrest matcher that is expected to match one — and only one — view within the current view hierarchy. Matchers are powerful and will be familiar to those who have used them with Mockito or Junit. If you are not familiar with hamcrest matchers, we suggest you start with a quick look at this presentation.

Often the desired view has a unique R.id and a simple withId matcher will narrow down the view search. However, there are many legitimate cases when you cannot determine R.id at test development time. For example, the specific view may not have an R.id or the R.id is not unique. This can make normal instrumentation tests brittle and complicated to write because the normal way to access the view (with findViewById()) does not work. Thus, you may need to access private members of the Activity or Fragment holding the view or find a container with a known R.id and navigate to its content for the particular view.

Espresso handles this problem cleanly by allowing you to narrow down the view using either existing ViewMatchers or your own custom ones.

Finding a view by its R.id is as simple as:

onView(withId(R.id.my_view))
Sometimes, R.id values are shared between multiple views. When this happens an attempt to use a particular R.id gives you an AmbiguousViewMatcherException (for example). The exception message provides you with a text representation of the current view hierarchy, which you can search for and find the views that match the non-unique R.id:

java.lang.RuntimeException:
com.google.android.apps.common.testing.ui.espresso.AmbiguousViewMatcherException:
This matcher matches multiple views in the hierarchy: (withId: is <123456789>)
…

+----->SomeView{id=123456789, res-name=plus_one_standard_ann_button, visibility=VISIBLE, width=523, height=48, has-focus=false, has-focusable=true, window-focus=true,
is-focused=false, is-focusable=false, enabled=true, selected=false, is-layout-requested=false, text=, root-is-layout-requested=false, x=0.0, y=625.0, child-count=1}
****MATCHES****
|
+------>OtherView{id=123456789, res-name=plus_one_standard_ann_button, visibility=VISIBLE, width=523, height=48, has-focus=false, has-focusable=true, window-focus=true,
is-focused=false, is-focusable=true, enabled=true, selected=false, is-layout-requested=false, text=Hello!, root-is-layout-requested=false, x=0.0, y=0.0, child-count=1}
****MATCHES****
Looking through the various attributes of the views, you may find uniquely identifiable properties (in the example above, one of the views has the text “Hello!”). You can use this to narrow down your search by using combination matchers:

onView(allOf(withId(R.id.my_view), withText("Hello!")))
You can also use not to reverse any of the matchers:

onView(allOf(withId(R.id.my_view), not(withText("Unwanted"))))
See ViewMatchers for the view matchers provided by Espresso.

Note: In a well-behaved application, all views that a user can interact with should either contain descriptive text or have a content description (see Android accessibility guidelines. If you are not able to narrow down an onView search using ‘withText’ or ‘withContentDescription’, consider treating it as an accessibility bug.

Note: Use the least descriptive matcher that finds the one view you’re looking for. Do not over-specify as this will force the framework to do more work than is necessary. For example, if a view is uniquely identifiable by its text, you need not specify that the view is also assignable from TextView. For a lot of views the R.id of the view should be sufficient.

Note: If the target view is inside an AdapterView (such as ListView, GridView, Spinner) the onView method might not work and it is recommended to use the onData method instead.

Performing an action on a view

When you have found a suitable matcher for the target view, it is possible to perform ViewActions on it using the perform method.

For example, to click on the view:

onView(...).perform(click());
You can execute more than one action with one perform call:

onView(...).perform(typeText("Hello"), click());
If the view you are working with is located inside a ScrollView (vertical or horizontal), consider preceding actions that require the view to be displayed (like click() and typeText()) with scrollTo(). This ensures that the view is displayed before proceeding to the other action:

onView(...).perform(scrollTo(), click());
Note: scrollTo() will have no effect if the view is already displayed so you can safely use it in cases when the view is displayed due to larger screen size (for example, when your tests run on both smaller and larger screen resolutions).

See ViewActions for the view actions provided by Espresso.

Checking if a view fulfills an assertion

Assertions can be applied to the currently selected view with the check() method. The most used assertion is the matches() assertion, it uses a ViewMatcher to assert the state of the currently selected view.

For example, to check that a view has the text “Hello!”:

onView(...).check(matches(withText("Hello!")));
Note: Do not put “assertions” into the onView argument - instead, clearly specify what you are checking inside the check block. For example:

If you want to assert that “Hello!” is content of the view, the following is considered bad practice:

// Don't use assertions like withText inside onView.
onView(allOf(withId(...), withText("Hello!"))).check(matches(isDisplayed()));
On the other hand, if you want to assert that a view with the text “Hello!” is visible - for example after a change of the views visibility flag - the code is fine.

Note: Be sure to pay attention to the difference between asserting that a view is not displayed and asserting that a view is not present in the view hierarchy.

Get started with a simple test using onView

In this example SimpleActivity contains a Button and a TextView. When the button is clicked the content of the TextView changes to "Hello Espresso!". Here’s how to test this with Espresso:

1. Click on the button

The first step is to look for a property that helps to find the button. The button in the SimpleActivity has a unique R.id - perfect!

onView(withId(R.id.button_simple))
Now to perform the click:

onView(withId(R.id.button_simple)).perform(click());
2. Check that the TextView now contains “Hello Espresso!”

The TextView with the text to verify has a unique R.id too:

onView(withId(R.id.text_simple))
Now to verify the content text:

onView(withId(R.id.text_simple)).check(matches(withText("Hello Espresso!")));
Using onData with AdapterView controls (ListView, GridView, …)

AdapterView is a special type of widget that loads its data dynamically from an Adapter. The most common example of an AdapterView is ListView. As opposed to static widgets like LinearLayout, only a subset of the AdapterView children may be loaded into the current view hierarchy and a simple onView() search would not find views that are not currently loaded. Espresso handles this by providing a separate onData() entry point which is able to first load the adapter item in question (bringing it into focus) prior to operating on it or any of its children.

Note: You may choose to bypass the onData() loading action for items in adapter views that are initially displayed on screen because they are already loaded. However, it is safer to always use onData().

Warning: Custom implementations of AdapterView can have problems with the onData() method, if they break inheritance contracts (particularly the getItem() API). In such cases, the best course of action is to refactor your application code. If you cannot do so, you can implement a matching custom AdapterViewProtocol. Take a look at the default AdapterViewProtocols provided by Espresso for more information.

Get started with a simple test using onData

This simple test demonstrates how to use onData(). SimpleActivity contains a Spinner with a few items - Strings that represent types of coffee beverages. When an item is selected, there is a TextView that changes to "One %s a day!" where %s is the selected item. The goal of this test is to open the Spinner, select a specific item and then verify that the TextView contains the item. As the Spinner class is based on AdapterView it is recommended to use onData() instead of onView() for matching the item.

1. Click on the Spinner to open the item selection

onView(withId(R.id.spinner_simple)).perform(click());
2. Click on the item “Americano”

For the item selection the Spinner creates a ListView with its contents - this can be very long and the element not contributed to the view hierarchy - by using onData() we force our desired element into the view hierarchy. The items in the Spinner are Strings, we want to match an item that is a String and is equal to the String “Americano”:

onData(allOf(is(instanceOf(String.class)), is("Americano"))).perform(click());
3. Verify that the TextView contains the String “Americano”

onView(withId(R.id.spinnertext_simple))
  .check(matches(withText(containsString("Americano"))));
Debugging

Espresso provides useful debugging information when a test fails:

Logging

Espresso logs all view actions to logcat. For example:

  ViewInteraction: Performing 'single click' action on view with text: Espresso
 
View hierarchy

Espresso prints the view hierarchy in the exception string when onView() fails. * If onView() does not find the target view, a NoMatchingViewException is thrown. You can examine the view hierarchy in the exception string to analyze why the matcher did not match any views. * If onView() finds multiple views that match the given matcher, an AmbiguousViewMatcherException is thrown. The view hierarchy is printed and all views that were matched are marked with the MATCHES label:

java.lang.RuntimeException:
com.google.android.apps.common.testing.ui.espresso.AmbiguousViewMatcherException:
This matcher matches multiple views in the hierarchy: (withId: is <123456789>)
…

+----->SomeView{id=123456789, res-name=plus_one_standard_ann_button, visibility=VISIBLE, width=523, height=48, has-focus=false, has-focusable=true, window-focus=true,
is-focused=false, is-focusable=false, enabled=true, selected=false, is-layout-requested=false, text=, root-is-layout-requested=false, x=0.0, y=625.0, child-count=1}
****MATCHES****
|
+------>OtherView{id=123456789, res-name=plus_one_standard_ann_button, visibility=VISIBLE, width=523, height=48, has-focus=false, has-focusable=true, window-focus=true,
is-focused=false, is-focusable=true, enabled=true, selected=false, is-layout-requested=false, text=Hello!, root-is-layout-requested=false, x=0.0, y=0.0, child-count=1}
****MATCHES****
When dealing with a complicated view hierarchy or unexpected behavior of widgets it is always helpful to use the Android View Hierarchy Viewer for an explanation.

AdapterView warnings

Espresso warns users about presence of AdapterView widgets. When an onView() operation throws a NoMatchingViewException and AdapterView widgets are present in the view hierarchy, the most common solution is to use onData(). The exception message will include a warning with a list of the adapter views. You may use this information to invoke onData to load the target view.