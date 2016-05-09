---
layout: blog
category: post
published: false
title: "Espresso-5-MatchToolbarTitle"
---
[원본](http://blog.sqisland.com/2015/05/espresso-match-toolbar-title.html)

Espresso로 Title bar의 값을 체크하려면 어떻게 해야 할까?

<이미지>

[Hierarchy Viewer](http://developer.android.com/intl/ko/tools/help/hierarchy-viewer.html)를 사용해서 살펴보자. 만약 단말이 루팅되지 않았다면 ANDROID_HVPROTO=ddm과 함께 실행해야 한다는 것을 기억하자.

<이미지>

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

이 커스텀 matcher를 사용하기 위해서, 
