## A deep dive into Android View constructors

[원본](http://blog.danlew.net/2016/07/19/a-deep-dive-into-android-view-constructors/)

나는 종종 Android _View_ 생성자를 둘러싼 혼란을 본다. 왜 그것들은 4개가 존재하는가? 각 매개 변수(parameter)는 무엇인가? 내가 구현해야 하는 생성자는 무엇인가?

## tl;dr
빠르고 실용적인 조언이 필요하다면, 여기 몇 개의 좋은 가이드라인이 있다.

* 코드에서 _Views_를 생성하려면 _view(Context)_를 사용하라
* XML에서 _Views_를 inflating할 때는 _view(Context, AttributeSet)_을 override하라
* 나머지는 무시하라. 아마 당신에게는 그것들이 필요하지 않을 것이다.

여전히 나와 함께할 사람들을 위해 - 뛰어들어가보자.

## 생성자 매개 변수들
많아 봐야, 생성자 매개 변수들은 4개가 있을 수 있다. 간단한 요약:
* _Context_ - _Views_내부의 모든 곳에서 사용된다.
* _AttributeSet_ - XML 속성(attribute)들 (XML에서 inflating될 때)
* _int defStyleAttr_ - _View_에 적용되는 기본 스타일 (테마에서 정의됨)
* _int defStyleResource_ - 만약 _defStyleAttr_이 사용되지 않을 때, _View_에 적용되는 기본 스타일.

## 속성들
어떻게 올바른 XML 속성들을 정의하는지에서 부터 이야기를 시작하자. 여기 XML안에 기본적인 _ImageView_가 있다.

	<ImageView  
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:src="@drawable/icon"
      />

_layout_width_, _layout_height_, _src_가 어디서 오는지 궁금한 적이 있었는가? 이는 난데없는 것이 아닌다; 사실은 당신은 저 속성들을 시스템이 _<declear-styleable>_을 통해 처리해야 하는 것으로 명시적으로 선언하였다. 예를 들어, _src_가 정의된 곳은 여기이다.

    <declare-styleable name="ImageView">  
      <!-- Sets a drawable as the content of this ImageView. -->
      <attr name="src" format="reference|color" />

      <!-- ...snipped for brevity... -->

    </declare-styleable>  
    
각 _declare-styleable_은 각 속성을 위해 _R.styleable.[name]_과 추가적인 _R.styleable.[name]_[attribute]_을 생성한다. 예를 들어 위는 _R.styleable.ImageView_와 _R.styleable.ImageView_src_를 생성한다.

What are these resources? The base R.styleable.[name] is an array of all the attribute resources, which the system uses to lookup attribute values. Each R.styleable.[name]_[attribute] is just an index into that array, so that you can retrieve all the attributes at once, then lookup each value individually.

If you think of it like a cursor, you can consider R.styleable.[name] as list of columns to query and each R.styleable.[name]_[attribute] as a column index.

For more on declare-styleable, here's the official documentation on creating your own.

## AttributeSet
The XML we wrote above is given to the View as an AttributeSet.

Usually you don't access AttributeSet directly, but instead use Theme.obtainStyledAttributes(). That's because the raw attributes often need to resolve references and apply styles. For example, if you define style=@style/MyStyle in your XML, this method resolves MyStyle and adds its attributes to the mix. In the end, obtainStyledAttributes() returns a TypedArray which you can use to access the attributes.

Greatly simplified, the process looks like this:

    public ImageView(Context context, AttributeSet attrs) {  
      TypedArray ta = context.obtainStyledAttributes(attrs, R.styleable.ImageView, 0, 0);
      Drawable src = ta.getDrawable(R.styleable.ImageView_src);
      setImageDrawable(src);
      ta.recycle();
    }
    
In this case, we're passing two parameters to obtainStyledAttributes(). The first is AttributeSet attrs, the attributes from XML. The second is the array R.styleable.ImageView, which tells the method which attributes we want to extract (and in what order).

With the TypedArray we get back, we can now access the individual attributes. We need to use R.styleable.ImageView_src so that we correctly index the attribute in the array.

(Recycling TypedArrays is important, too, so I left it in the sample above.)

Normally you extract multiple attributes at once. Indeed, the actual ImageView implementation is far more complex than what's shown above (since ImageView itself has many more attributes it cares about).

You can read more about extracting attributes in the official documentation.

## Theme Attributes
A sidenote, for completeness: The AttributeSet is not the only place we got our values from when using obtainStyledAttributes() in the last section. Attributes can also exist in the theme.

This rarely plays a role for View inflation because your theme shouldn't be setting attributes like src, but it can play a role if you use obtainStyledAttributes() for retrieving theme attributes (which is useful but is outside the scope of this article).

## Default Style Attribute
You may have noticed that I used 0 for the last two parameters in obtainStyledAttributes(). They are actually two resource references - defStyleAttr and defStyleRes. I'm going to focus on the first one here.

defStyleAttr is, by far, the most confusing parameter for obtainStyledAttributes(). According to the documentation it is:

> An attribute in the current theme that contains a reference to a style resource that supplies defaults values for the TypedArray.

Whew, that's a mouthful. In plain English, it's a way to be able to define a base style for all Views of a certain type. For example, you can set textViewStyle in your theme if you want to modify all your app's TextViews at once. If this didn't exist, you'd have to manually style every TextView instead.

Let's walk through how it actually works, using TextView as an example.

First, it's an attribute (in this case, R.attr.textViewStyle). Here's where the Android platform defines textViewStyle:

    <resources>  
      <declare-styleable name="Theme">

        <!-- ...snip... -->

        <!-- Default TextView style. -->
        <attr name="textViewStyle" format="reference" />

        <!-- ...etc... -->

      </declare-styleable>
    </resource>  

Again, we're using declare-styleable, but this time to define attributes that can exist in the theme. Here, we're saying that textViewStyle is a reference - that is, its value is just a reference to a resource. In this case, it should be a reference to a style.

Next we have to set textViewStyle in the current theme. The default Android theme looks like this:

    <resources>  
      <style name="Theme">

        <!-- ...snip... -->

        <item name="textViewStyle">@style/Widget.TextView</item>

        <!-- ...etc... -->

      </style>
    </resource>  

Then the theme has to be set for your Application or Activity, typically via the manifest:

    <activity  
      android:name=".MyActivity"
      android:theme="@style/Theme"
      />

Now we can use it in obtainStyledAttributes():

    TypedArray ta = theme.obtainStyledAttributes(attrs, R.styleable.TextView, R.attr.textViewStyle, 0); 
    
The end result is that any attributes not defined by the AttributeSet are filled in with the style that textViewStyle references.

Phew! Unless you're being hardcore, you don't need to know all these implementation details. It's mostly there so that that the Android framework can let you define base styles for various Views in your theme.

## Default Style Resource
defStyleRes is much simpler than its sibling. It is just a style resource (i.e. @style/Widget.TextView). No complex indirection through the theme.

The attributes from the style in defStyleRes are applied only if defStyleAttr is undefined (either as 0 or it isn't set in the theme).

## Precedence
We've now got a bunch of ways to derive the value for an attribute via obtainStyledAttributes(). Here's their order of precedence, from highest to lowest:

1. Any value defined in the AttributeSet.
2. The style resource defined in the AttributeSet (i.e. style=@style/blah).
3. The default style attribute specified by defStyleAttr.
4. The default style resource specified by defStyleResource (if there was no defStyleAttr).
5. Values in the theme.

In other words, any attributes you set directly in XML will be used first. But there are all sorts of other places those attributes can be retrieved from if you don't set them yourself.

## View constructors
This article was supposed to be about View constructors, right?

There are four of them total, each one adding a parameter:

    View(Context)

    View(Context, AttributeSet)

    View(Context, AttributeSet, defStyleAttr)

    View(Context, AttributeSet, defStyleAttr, defStyleRes)  

An important note: the last one was added in API 21, so you unless you've got minSdkVersion 21, you should avoid it for now. (If you want to use defStyleRes just call obtainStyledAttributes() yourself since it's always been supported.)

They cascade, so if you call one, you end up calling them all (via super). The cascading also means you only need to override the constructors you use. Generally, this means that you only need to implement the first two (one for code constructor, the other for XML inflation).

I usually setup my custom Views like so:

    SomeView(Context context) {  
      this(context, null);
    }

    SomeView(Context context, AttributeSet attrs) {  
      // Call super() so that the View sets itself up properly
      super(context, attrs);

      // ...Setup View and handle all attributes here...
    }

Within the two-arg constructor you can use obtainStyledAttributes() any way you want. A quick way to implement a default style is to just provide defStyleRes to it; that way you don't need to go through the pain in the butt that is defStyleAttr (which is more of a framework tool and not usually necessary for a single app).

Anyways, I hope this helps not only your understanding of View constructors but also how attributes are retrieved during View construction!