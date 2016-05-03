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
레이아웃은 그들 스스로 어떠한 유일점이 없는 어떤 뷰들 포함할 수 있다(예를 들어 주소록 목록에서 반복하는 전화 버튼은 view 계층에서 다른 전화 버튼들과 동일한 R.id, 동일한 문자열, 그리고 동일한 속성들을 가지고 있을 수 있다).

예를 들어, 이 activity에서, "7"이라는 글자는 여러 열에서 반복된다.
![]({{site.baseurl}}/https://google.github.io/android-testing-support-library/docs/images/hasSibling.png)

종종, 유일하지 않은 view는 그 옆에 위치한 어떤 유일한 레이블과 쌍을 이룰 수 있다(예를 들어 주소록의 전화 버튼 옆의 이름). 이 경우, 당신은 당신의 선택을 좁히기 위해 hasSibling matcher를 사용할 수 있다:

	onView(allOf(withText("7"), hasSibling(withText("item: 0"))))
      .perform(click());

## onData와 커스텀 ViewMatcher로 데이터 매칭하기

아래의 Activity는 각 행을 위해 Map<String, Object>안에 데이터를 가지고 있는 [SimpleAdapter](http://developer.android.com/intl/ko/reference/android/widget/SimpleAdapter.html)
의 원조를 받는 ListView를 포함한다. 각 map은 content(string, "item:x")이 들어있는 키 "STR"과 content의 길이인 Interger가 들어있는 키 "LEN"를 가진다.

![]({{site.baseurl}}/https://google.github.io/android-testing-support-library/docs/images/list_activity.png)

"item: 50"을 가진 열을 클릭하는 코드는 다음과 같다:

	onData(allOf(is(instanceOf(Map.class)), hasEntry(equalTo("STR"), is("item: 50")))
	  .perform(click());
  
onData안의 Matcher<Object>를 분해해보자:

	is(instanceOf(Map.class))

AdapterView에서 Map인 특정 항목으로 검색을 좁힌다.

우리의 경우, 이는 목록 view의 모든 열이다. 하지만 우리는 명확하게 "item: 50"을 클릭하기를 원한다. 그래서 다음과 같이 검색을 더 좁힌다:

	hasEntry(equalTo("STR"), is("item: 50"))

This Matcher<String, Object> will match any Map that contains an entry with any key and value = “item: 50”. As the code to look up this is long and we want to reuse it in other locations - let us write a custom “withItemContent” matcher for that.

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
    
We use a BoundedMatcher as a base because we want to be able to only match on objects of class Map. We override the matchesSafely method, put in the matcher we found earlier and match it against a Matcher<String> that can be passed as an argument. This allows us to do withItemContent(equalTo("foo")). For code brevity, we create another matcher that already does the equalTo for us and accepts a String.

    public static Matcher<Object> withItemContent(String expectedText) {
      checkNotNull(expectedText);
      return withItemContent(equalTo(expectedText));
    }
    
Now the code to click on the item is simple:

	onData(withItemContent("item: 50")) .perform(click());
    
For the full code of this test, take a look at [AdapterViewTest#testClickOnItem50](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/AdapterViewTest.java) and the [custom matcher](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/LongListMatchers.java).

# View의 특정 자식 view에 매칭하기

위 샘플은 ListView의 열 전체의 중간을 클릭하는 문제가 있다. 만약 우리가 열을 특정 자식에게 
The sample above issues a click in the middle of the entire row of a ListView. But what if we want to operate on a specific child of the row? For example, we would like to click on the second column of the row of the LongListActivity, which displays the String.length of the first row (to make this less abstract, you can imagine the G+ app that shows a list of comments and each comment has a +1 button next to it):

![]({{site.baseurl}}/https://google.github.io/android-testing-support-library/docs/images/item50.png)

이제 당신의 DataInteraction에 onChildView 명시를 추가하라:

    onData(withItemContent("item: 60"))
      .onChildView(withId(R.id.item_size))
      .perform(click());

Note: 이 예제는 위 샘플의 withItemConent matcher를 사용한다! [AdapterViewTest#testClickOnSpecificChildOfRow60](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso/sample/src/androidTest/java/android/support/test/testapp/AdapterViewTest.java)를 보라!



그외 
http://www.vogella.com/tutorials/AndroidTestingEspresso/article.html
https://androidresearch.wordpress.com/2015/04/04/an-introduction-to-espresso/

