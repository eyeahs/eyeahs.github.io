---
layout: post
category: blog
published: false
title: "Espresso - RecyclerView"
---
[원본](https://medium.com/@_rpiel/recyclerview-and-espresso-a-complicated-story-3f6f4179652e#.d3sr7snaz)

# RecyclerView and espresso, a complicated story

Espresso는 _RecyclerView_에 표시되는 항목을 체크하기 위해서는 크게 도움이 되지는 않는다. _ListView_(또는 _AdapterView_의 다른 종류)를 사용하는 흔한 패턴은 _onData_를 사용하는 것이다 :

	onData(is(instanceOf(Item.class)))
	   .check(matches(hasDescendant(withText("Some text"))));

_ListView_에서 _onData_는 각 항목으로 스크롤한 뒤 matcher를 적용해주기 때문에 꽤 편리한다. 불행히도 _RecyclerView_를 위한 비슷한 것은 없다.

_onView_를 사용하면 간단하게 화면에 표시되고 있는 view를 체크할 수 있다.

	onView(withId(R.id.recycleview))
	   .check(matches(hasDescendant(withText("Some text"))))
       
또는 [특정 자식](http://stackoverflow.com/questions/24748303/selecting-child-view-at-index-using-espresso/30073528#30073528)을 타기팅한 matcher를 통해:

	onView(nthChildOf(withId(R.id.recycleview), 0)
	   .check(matches(hasDescendant(withText("Some text"))))

거의 가깝다. 하지만 이는 보이지 않는 view에는 동작하지 않는다. 따라서 우리는 주어진 position으로 이동한 뒤 해당 포지션의 view를 체크하길 원할것이다.

    int i = 10;
    onView(withId(R.id.recyclerview))
    onView(viewMatcher)
            .perform(scrollToPosition(i))
            .check(new RecyclerItemViewAssertion<>(i, items.get(i), new ItemViewAssertion<Item>() {
                @Override
                public void check(Item item, View view, NoMatchingViewException e) {
                matches(hasDescendant(withText(item.getDisplayName())))
                    .check(view, e);
                }
            }));

(이 글의 마지막에 있는 RecyclerItemViewAssertion과 ItemViewAssertion을 위한 링크를 보라)
이는 약간 혼잡스럽지만 꽤 잘 동작한다. 이를 목록의 항목을 가져오는 메소드에 래핑한 하면 다음 코드 라인들을 사용하여 빠르게 RecyclerView를 체크할 수 있다.

	RecyclerViewInteraction.    
        <Item>onRecyclerView(withId(R.id.recyclerview))
        .withItems(items)
        .check(new ItemViewAssertion<Item>() {
            @Override
            public void check(Item item, View view, NoMatchingViewException e) {
            matches(hasDescendant(withText(item.getDisplayName())))
                .check(view, e);
            }
        });
        
어떤가? 물론 이는 onData처럼 강력하지는 않지만 문제를 해결하기 위한 일을 할 것이다.

전체 코드는 [이 gist](https://gist.github.com/RomainPiel/ec10302a4687171a5e1a)를 보라.
