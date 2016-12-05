---
layout: post
category: blog
published: false
title: FragmentStatePagerAdapter와 모험을
---
많은 안드로이드 개발자들은 FragmentPagerAdapter와 FramgentStatePagerAdapter를 헛갈리거나 심지어 둘 사이의 차이를 모른다. 또한 종종 notifyDatasetChanged()를 동작시키면 실망하기도 한다. 이 adapter들을 사용하면 메모리 누수가 쉽게 일어날 수 있다. 나는 기초부터 시작해서 구현 세부 사항을 자세히 설명하고 잘 알려지지 않은 점들을 지적할 것이다 (FragmentPagerAdapter의 Fragment들은 Activity가 finish할 때만 메모리에서 해제된다는 것을 알고 있는가? Read on :-)).

기본적인 차이점

FragmentPagerAdapter

제한된(고정된) 개수의 항목(Fragment)들에 적합하다. 왜? 한 번 생성되면 Fragment의 인스턴스를 FragmentManager에서 절대로 제거하지 않기 때문이다(Activity가 종료되지 않는 한). 현재 보이지 않는 Fragment에서 View들을 detach한다. 당신의 Fragment가 범위 밖으로 나가면 onDestoryView()가 호출되고, 나중에 이 Fragment로 돌아가면 onCreateView()가 호출 될 것이다.

FragmentStatePagerAdapter

FragmentStatePagerAdapter는 메모리에 좀 더 요령이 있다. 범위 밖의 Fragment 인스턴스를 FragmentManager에서 완전히 제거한다. 제거된 Fragment의 상태는 FragmentStatePagerAdapter 내부에 저장된다. Fragment 인스턴스는 당신이 다시 돌아왔을 때 재생성되고 상태는 복원된다. 이 Adapter는 개수가 미정인 리스트나 항목이 자주 변경되는 리스트에 적합하다.

FragmentPagerAdapter - 메모리 누수 위험

FragmentPagerAdapter의 Fragment들은 detached만 되며 절대로 FragmentManager에서 제거되지 않는다(Activity가 종료되지 않는한). FragmentPagerAdapter를 사용할 때는 반드시 onDestroyView()에서 현재 View 또는 Context에 대한 참조를 제거해야 한다. 그렇지 않으면 Garage Collector는 전체 View 또는 심지어 Activity를 릴리즈 할 수 없다. 이는 View/Context 관련 필드를 null로 설정하기(ButterKnife는 자동으로 바인드를 해제할 수 있다)와 Context 또는 View를 누설시킬 수 있는 모든 리스너를 제거함을 의미한다.

그렇게 하지 않으면 메모리를 고갈 시킬 수 있다-당신의 FragmentPagerAdapter에 10개의 항목을 가지고 있다고 가정하자. 그것들 모두를 스와이프하면 (setOffScreenPageLimit()설정에 따라) 마지막 3개의 View들을 메모리에 유지하는 대신 10개의 View가 유지된다. 화면을 회전하면 사태가 더 악화된다(10개 중 7개는 여전히 "destory된" Activity에 참조가 유지된다)

detach된 좀비 fragment들

또한 당신은 Fragment가 FragmentManager에서 절대 제거되지 않는 것의 영향에 대해 생각해 볼 필요가 있다. adapter에 묶인 작은 list의 항목들을 끊임없이 변경하면 메모리에 수백개의 detach된 인스턴스들을 가질 수 있다. 이것은 새 항목을 만들고 오래된 detach된 Fragment들을 유지할 것이다. 이는 FragmentStatePagerAdapter에서는 문제가 적다. 이것은 전체 Fragment 인스턴스들을 FragmentManager에서 제거하기 때문이다. 많은 Fragment들을 스와이프하면 사고를 일으킬 수 있다. 각각의 것들이 새로운 Bunde 인스턴스를 맵에 추가하기 때문에 상당히 커질 수 있다 (화면 회전 중에 TransactionTooLargeException을 일으킬 가능성이 있다).

notifyDatasetChanged()의 어려움들

내 생각에는 우리 모두 notifyDatasetChanged()를 호출 할 때 문제를 겪었을 것이다. 이렇게 하는 것은 현재 보여지는 fragment들을 갱신하지 않으며 현재 fragment를 강제로 재생성하기 위해서 "앞뒤" 스와이프를 해야한다. 두 PagerAdapter 모두 Fragment 인스턴스들을 캐싱하고 재사용하고 있다. 
