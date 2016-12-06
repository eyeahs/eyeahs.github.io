---
layout: post
category: blog
published: false
title: FragmentStatePagerAdapter와 모험을
---
[원본 : Adventures with FragmentStatePagerAdapter](https://medium.com/inloop/adventures-with-fragmentstatepageradapter-4f56a643f8e0#.mm7leuau9)

많은 안드로이드 개발자들은 [FragmentPagerAdapter](https://developer.android.com/reference/android/support/v4/app/FragmentPagerAdapter.html)와 [FramgentStatePagerAdapter](https://developer.android.com/reference/android/support/v4/app/FragmentStatePagerAdapter.html)를 헛갈리거나 심지어 둘 사이의 차이를 모른다. 또한 종종 `notifyDatasetChanged()`를 동작시키면 실망하기도 한다. 이 adapter들을 사용하면 메모리 누수가 쉽게 일어날 수 있다. 나는 기초부터 시작해서 구현 세부 사항을 자세히 설명하고 잘 알려지지 않은 점들을 지적할 것이다 (FragmentPagerAdapter의 Fragment들은 Activity가 finish할 때만 메모리에서 해제된다는 것을 알고 있는가? Read on :-)).

## 기본적인 차이점

### FragmentPagerAdapter

제한된(고정된) 개수의 항목(Fragment)들에 적합하다. 왜? **한 번 생성되면 Fragment의 인스턴스를 FragmentManager에서 절대로 제거하지 않기 때문이다**(Activity가 종료되지 않는 한). **현재 보이지 않는 Fragment에서 View들을 [detach](https://developer.android.com/reference/android/app/FragmentTransaction.html#detach%28android.app.Fragment%29)한다.** 당신의 Fragment가 범위 밖으로 나가면 `onDestoryView()`가 호출되고, 나중에 이 Fragment로 돌아가면 `onCreateView()`가 호출 될 것이다.

### FragmentStatePagerAdapter

FragmentStatePagerAdapter는 메모리에 좀 더 요령이 있다. **범위 밖의 Fragment 인스턴스를 FragmentManager에서 완전히 제거한다.** 제거된 Fragment의 상태는 FragmentStatePagerAdapter 내부에 저장된다. **Fragment 인스턴스는 당신이 다시 돌아왔을 때 재생성되고 상태는 복원된다.** 이 Adapter는 개수가 미정인 리스트나 항목이 자주 변경되는 리스트에 적합하다.

## FragmentPagerAdapter - 메모리 누수 위험

FragmentPagerAdapter의 Fragment들은 detached만 되며 절대로 FragmentManager에서 제거되지 않는다(Activity가 종료되지 않는한). **FragmentPagerAdapter를 사용할 때는 반드시 `onDestroyView()`에서 현재 View 또는 Context에 대한 참조를 제거해야 한다.** 그렇지 않으면 Garage Collector는 전체 View 또는 심지어 Activity를 릴리즈 할 수 없다. 이는 View/Context 관련 필드를 null로 설정하기(ButterKnife는 자동으로 바인드를 해제할 수 있다)와 Context 또는 View를 누설시킬 수 있는 모든 리스너를 제거함을 의미한다.

그렇게 하지 않으면 메모리를 고갈 시킬 수 있다-당신의 FragmentPagerAdapter에 10개의 항목을 가지고 있다고 가정하자. 그것들 모두를 스와이프하면 ([`setOffScreenPageLimit()`](https://developer.android.com/reference/android/support/v4/view/ViewPager.html#setOffscreenPageLimit%28int%29)설정에 따라) 마지막 3개의 View들을 메모리에 유지하는 대신 10개의 View가 유지된다. 화면을 회전하면 사태가 더 악화된다(10개 중 7개는 여전히 "destory된" Activity에 참조가 유지된다)

### detach된 좀비 fragment들

또한 당신은 Fragment가 FragmentManager에서 절대 제거되지 않는 것의 영향에 대해 생각해 볼 필요가 있다. **adapter에 묶인 작은 list의 항목들을 끊임없이 변경하면 메모리에 수백개의 detach된 인스턴스들을 가질 수 있다.** 이것은 새 항목을 만들고 오래된 detach된 Fragment들을 유지할 것이다. 이는 FragmentStatePagerAdapter에서는 문제가 적다. 이것은 전체 Fragment 인스턴스들을 FragmentManager에서 제거하기 때문이다. 많은 Fragment들을 스와이프하면 사고를 일으킬 수 있다. 각각의 것들이 새로운 Bunde 인스턴스를 맵에 추가하기 때문에 상당히 커질 수 있다 (화면 회전 중에 [`TransactionTooLargeException`](https://developer.android.com/reference/android/os/TransactionTooLargeException.html)을 일으킬 가능성이 있다).

## notifyDatasetChanged()의 문제들

내 생각에는 우리 모두 `notifyDatasetChanged()`를 호출 할 때 문제를 겪었을 것이다. 이렇게 하는 것은 현재 보여지는 fragment들을 갱신하지 않으며 현재 fragment를 강제로 재생성하기 위해서 "앞뒤" 스와이프를 해야한다. 두 PagerAdapter 모두 Fragment 인스턴스들을 캐싱하고 재사용하고 있다. 그렇지 않다면 notifyDataSetChanged의 호출이 그것들을 (필요하지 않더라도) 다시 재생성하게 할 것이기 때문이다.

notifyDataSetChanged()는 데이트 세트가 변경되는 상황을 위한 것임에 주목하는 것 역시 중요하다. 이는 일부 항목들이 제거되거나 추가되었을 때을 의미한다. **notifyDataSetChanged() 메소드는 현재 표시되고 있는 Fragment나 그들의 view들을 갱신하기 위한 용도가 아니다.** 만약 일부 데이터가 변경되어 그들의 뷰들을 갱신하고자 한다면 당신의 Fragment에 Listener/Callback을 추가해야 한다.

## FragmentPagerAdapter & notifyDatasetChanged()
데이터 세트 변경을 지원하기 위해서 당신의 FragmentPAgerAdapter에 두 메소드를 오버라이드해야 한다.
### int getItemPosition(Object object)
> 호스트 뷰가 항목의 위치가 변경되었는지를 판단하기 위해 호출 된다. 주어진 항목의 위치가 변경되지 않은 경우 [POSITION_UNCHANGED](https://developer.android.com/reference/android/support/v4/view/PagerAdapter.html#POSITION_UNCHANGED)를 반환하고 항목이 더 이상 존재하지 않는다면 [POSITION_NONE](https://developer.android.com/reference/android/support/v4/view/PagerAdapter.html#POSITION_NONE)을 반환한다.
> 이것의 기본 구현은 항목들이 위치를 절대로 변경하지 않을 것이라고 가정하고 항상 [POSITION_UNCHANGED](https://developer.android.com/reference/android/support/v4/view/PagerAdapter.html#POSITION_UNCHANGED)를 반환한다.

`getItemPosition(Object object)`는 당신의 데이터 세트에서 현재 Fragment의 위치를 결정하는 일종의 로직(예를 들어 데이터 세트를 순회하며 Fragment에 대응하는 등장 위치를 찾는다)이다. 

항상 [POSITION_NONE](https://developer.android.com/reference/android/support/v4/view/PagerAdapter.html#POSITION_NONE)를 반환하는 것은 메모리와 성능에 비효율적이다. 이렇게 하면 데이터 세트에서의 Fragment의 위치가 변경되지 않아도 현재 보이는 fragment를 항상 detach하고 재생성한다. 그리고 오래된 fragment들은 당신이 Activity를 떠나기 전까지 메모리에 남아 있을 것이다.

### long getItemId(int position)
> 주어진 position에 있는 항목의 고유 식별자를 반환한다. 이것의 기본 구현은 주어진 위치를 반환한다. 항목들의 위치가 변경될 수 있다면 서브클래스는 이 메소드를 오버라이드 해야 한다.

이 메소드는 FragmentManager안에 detach된 Fragment 인스턴스가 존재하는지를 찾기 위해 `instantiateItem()`의 내부에서 사용된다. 이 메소드를 오버라이드하지 않고 `notifyDataSetChanged()`를 호출하면 현재 인덱스에 존재하는 Fragment의 인스턴스가 반환될 뿐이다. 당신은 그 Fragment를 위해 고유 식별자를 반환해야 할 필요가 있다.

## FragmentStatePagerAdapter - 상태 bundle "버그"
FragmentStatePagerAdapter에서 데이터 세트의 변경을 지원하도록 하기 위해서는 `getItemPosition()`을 오버라이드 해야 한다(여기에는 `getItemId()`메소드가 없다). 여기에는 FragmentPagerAdapter를 위해 언급했던 것과 동일한 것이 적용된다(항상 POSITION_NONE을 반환하지 마라).

잘 알려지지 않은 점이 하나 있는데 -그리고 이건 꽤 불쾌한 문제이다.. 이 이슈는 [이 블로그](http://speakman.net.nz/blog/2014/02/20/a-bug-in-and-a-fix-for-the-way-fragmentstatepageradapter-handles-fragment-restoration/)에 자세히 설명되어 있다. 문제는 FragementStatePagerAdapter가 상태 bundle들의 ArrayList를 유지한다는 것이다. **instantiateItem()메소드는 Fragment의 인스턴스가 존재하지 않는다면 새로운 인스턴스를 생성할 것이다 -하지만 그 다음에는 ArrayList를 조사하고 항목의 인덱스에 기반하여 Bundle를 선택한다!** 이는 그 인덱스에 이전에 존재하였던 (다른) Fragment 인스턴스에 속한 Bundle일 수 있다. 데이터 세트를 변경하고 adpater를 갱신한다면 이 문제에 봉착할 수 있다. 가능한 해결 방법은 링크된 블로그를 참조하라.

## Fragment{}는 현재 FragmentManager에 존재하지 않는다.
[이 이슈](https://code.google.com/p/android/issues/detail?id=37990)는 4년 전에 공개되었고 아직 고쳐지지 않았다! 이것은 [이 블로그](http://billynyh.github.io/blog/2014/03/02/fragment-state-pager-adapter/)에 자세히 설명되어 있다. 이는 정말 실망스러우며, 이 시점 이후에 당신만의 "수정된" 버전의 FragmentStatePagerAdapter를 가질 생각을 할 수 있을 것이다.

Adam Powell가 G+에 [댓글](https://plus.google.com/u/0/+BillyNgYuHang/posts/a1xwgEEehCs)을 남겼다.
> FragmentStatePagerAdapter는 PagerAdapters의 데이터 세트 변경 기능보다 먼저 실행됩니다. 데이터 세트 변경을 처리해야 하는 PagerAdapter가 있는 경우 FragmentStatePagerAdapter에서 시작하는 것보다 PagerAdapter에서 직접 계약을 구현하는 것이 더 쉽습니다.

글쎄 ... 좋았지만, 이것에 대해 크게 경고했었어야 한다. 간단한 해결책은 getItemPosition()에서 항상 POSITION_NONE을 반환하는 것이다. 그러나 이것은 성능에 영향을 미친다.

## PagerAdapter도 존재한다...
그리고 처음부터 이 사실을 언급했어야 할지도 모른다 -당신의 애플리케이션이 ViewPager를 가진 경우에도 항상 Fragment를 이용할 필요가 있는 것은 아니다. 어떠한 혼성 View 레이아웃들과 단순한 PagerAdapter를 사용하는 것이 더 훌륭하고 덜 복잡하다. 이는 당신이 모든 생명 주기 콜백이 필요한지 아닌지에 달려있다.