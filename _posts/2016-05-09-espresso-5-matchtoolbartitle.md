---
layout: blog
category: post
published: true
title: "Espresso-5-MatchToolbarTitle"
date: "2016-05-09 15:35:00 +09:00"
---
[원본](http://blog.sqisland.com/2015/05/espresso-match-toolbar-title.html)

Espresso로 Title bar의 값을 체크하려면 어떻게 해야 할까?

![](https://raw.githubusercontent.com/eyeahs/eyeahs.github.io/master/_images/espresso5/my_awesome_title.png)

[Hierarchy Viewer](http://developer.android.com/intl/ko/tools/help/hierarchy-viewer.html)를 사용해서 살펴보자. 만약 단말이 루팅되지 않았다면 ANDROID_HVPROTO=ddm과 함께 실행해야 한다는 것을 기억하자.

![](https://raw.githubusercontent.com/eyeahs/eyeahs.github.io/master/_images/espresso5/hierarchy_viewer.png)

# 첫 시도 : TextView에 매치하기

계층안에는 우리의 _TextView_가 있지만 id가 없다. 하지만 이것이 _Toolbar_의 자식임을 알았으므로 _withParent_를 사용하여 여기에 매치할 수 있다.

    @Test
    public void toolbarTitle() {
      CharSequence title = InstrumentationRegistry.getTargetContext()
        .getString(R.string.my_title);
      matchToolbarTitle(title);
    }

    private static ViewInteraction matchToolbarTitle(
        CharSequence title) {
      return onView(
        allOf(
            isAssignableFrom(TextView.class))),
            withParent(isAssignableFrom(Toolbar.class))
        .check(matches(withText(title.toString())));
    }

이것이 title을 검증하기 위해 view를 찾는 방법이다:
* 이것은 _TextView_이다.
* 이것은 부모로 _Toolbar_을 가지고 있다.

이것이 동작하긴 하지만, 이는 public API의 일부가 아니며 다음 버전에서 변경될 가능성이 있는 _Toolbar_의 내부 구조에 달려있다. 이를 개선해보자.

# 두번째 시도 : Toolbar.getTitle()를 사용하는 커스텀 matcher

    private static ViewInteraction matchToolbarTitle(
        CharSequence title) {
      return onView(isAssignableFrom(Toolbar.class))
          .check(matches(withToolbarTitle(is(title))));
    }

    private static Matcher<Object> withToolbarTitle(
        final Matcher<CharSequence> textMatcher) {
      return new BoundedMatcher<Object, Toolbar>(Toolbar.class) {
        @Override public boolean matchesSafely(Toolbar toolbar) {
          return textMatcher.matches(toolbar.getTitle());
        }
        @Override public void describeTo(Description description) {
          description.appendText("with toolbar title: ");
          textMatcher.describeTo(description);
        }
      };
    }

_withToolbarTitle()_는 우리에게 타입 안정성을 제공하는 커스텀 _BoundedMatcher_를 반환한다. matchesSafely()에서Toolbar.getTitle()를 호출하여 그것의 값을 검증한다.

이 커스텀 matcher를 사용하기 위해서, 우리는 _Toolbar_ 자신을 검색하는 헬퍼 함수를 변경하고, 기대하는 title을 가지고 있는지 체크한다. string 대신 text matcher를 받는 _withToolbarTitle_에 _is(title)_을 전달하였음에 주목하라. 이 방법은 우리가 _startsWith_와  _endsWith_같은 다른 matcher를 사용할 수 있게 한다.

# 소스 코드와 노트들

https://github.com/chiuki/espresso-samples/ under toolbar-title

* 이 _Toolbar_는 우리가 AppCompat을 사용하므로 실제로는 _android.support.v7.widget.Toolbar_이다.
* 두 시도 모두, 단 하나의 _Toolbar_만 있다는 것을 가정한다. 당신이 하나 이상을 가지고 있다면, 당신이 매치하기 원하는 것을 정확히 찾아내기 위한 추가 matcher들이 필요할 수도 있다. 
* 우리가 toolbar title를 어떻게 매치하는지에 대한 상세 구현을 숨기기 위해 헬퍼 function _matchToolbarTitle()_을 사용한다. 이 방식으로, 밑에 있는 코드가 변경되어도, toolbar title을 매치하고자 하는 테스트들 모두를 위해 단 한 곳만 업데이트하면 된다.
* 헬퍼 function _matchToolbarTitle()_은 _ViewInteraction_을 반환한다. 이는 Toolbar의 다른 속성들을 검증하기 위한 호출들과 체이닝할 수 있게 한다. 예를 들어 당신은 이런 것들을 할 수 있다:

	matchToolbarTitle(title)
  		.check(matches(isDisplayed()));

Article inspired by discussion with Danny Roa, Jacob Tabak and Jake Wharton.

Like this article? Take a look at the outline of my Espresso book and fill in this form to push me to write it! Also check out the published courses: https://gumroad.com/chiuki

