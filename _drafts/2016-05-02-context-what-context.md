---
published: false
---
[원본](https://possiblemobile.com/2013/06/context/)

### Context는 아마 안드로이드 어플리케이션에서 가장 많이 사용된 요소이다. 그리고 아마 가장 많이 오용되었을 것이다.

  Context 객체들은 매우 흔하고, 매우 자주 전달 되었기에 이는 당신이 의도하지 않은 상황을 만들기 쉽다. 리소스를 로드하고 새로운 Activity를 시작하고, 시스템 서비스를 얻고, 내부 파일 패스를 얻고, 뷰를 생성하는 것 모두 작업을 완수하기 위해서 _Context를_ 요구한다(and that’s not even getting started on the full list!). What I’d like to do is provide for you some insights on how Context works alongside some tips that will (hopefully) allow you to leverage it more effectively in your applications.

# Context Types
Not all Context instances are created equal.  Depending on the Android application component, the Context you have access to varies slightly:

**Application**은 당신의 어플리케이션 프로세스안에서 작동하는 싱글톤 인스턴스이다. Activity나 Service에서는 _getApplication()_를 통해, 그리고 Context에서 상속받은 어떤 객체에서도 _getApplicationContext()_를 통해 이것에 접근할 수 있다. 어디서 또는 어떻게 접근하였는지와 상관없이, 당신의 process내에서는 언제나 동일한 인스턴스를 받게 될 것이다.

## Activity/Service – inherit from ContextWrapper which implements the same API, but proxies all of its method calls to a hidden internal Context instance, also known as its base context.  Whenever the framework creates a new Activity or Service instance, it also creates a new ContextImpl instance to do all of the heavy lifting that either component will wrap.  Each Activity or Service, and their corresponding base context, are unique per-instance.
**Activity/Service**는 ContextWrapper
프레임워크가 새로운 Activity 또는 Service 인스턴스를 만들때는 언제든지, 새로운 ContextImpl 인스턴스를 생성한다.

**BroadcastReceiver**은 Context 그 자체는 아니지만, 새로운 broadcast 이벤트가 도착할때마다 프레임워크는 _onReceive()_을 통해 BroadcastReceiver에 Context를 전달한다. 이 인스턴스는 두가지 주요 함수들인 _registerReceiver()_와 _bindService()_의 호출이 허용되지 않는 _ReceiverRestrictedContext_이다. 이 두 함수들은 존재하는 BroadcastReceiver.onReceive()내부에서 허용되지 않는다. 리시버가 broadcast를 처리할 때 마다 Context의 새로운 인스턴스가 전달된다.

**ContentProvider**
## ContentProvider – is also not a Context but is given one when created that can be accessed via getContext().  If the ContentProvider is running local to the caller (i.e. same application process), then this will actually return the same Application singleton.  However, if the two are in separate processes, this will be a newly created instance representing the package the provider is running in.

# Saved References
The first issue we need to address comes from saving a reference to a Context in an object or class that has a lifecycle that extends beyond that of the instance you saved.  For example, creating a custom singleton that requires a Context to load resources or access a ContentProvider, and saving a reference to the current Activity or Service in that singleton.

**Bad Singleton**

    public class CustomManager {
        private static CustomManager sInstance;

        public static CustomManager getInstance(Context context) {
            if (sInstance == null) {
                sInstance = new CustomManager(context);
            }

            return sInstance;
        }

        private Context mContext;

        private CustomManager(Context context) {
            mContext = context;
        }
    }
The problem here is we don’t know where that Context came from, and it is not safe to hold a reference to the object if it ends up being an Activity or a Service.  This is a problem because a singleton is managed by a single static reference inside the enclosing class.  This means that our object, and ALL the other objects referenced by it, will never be garbage collected.  If this Context were an Activity, we would effectively hold hostage in memory all the views and other potentially large objects associated with it; creating a leak.

To protect against this, we modify the singleton to always reference the application context:

**Better Singleton**

    public class CustomManager {
        private static CustomManager sInstance;

        public static CustomManager getInstance(Context context) {
            if (sInstance == null) {
                //Always pass in the Application Context
                sInstance = new CustomManager(context.getApplicationContext());
            }

            return sInstance;
        }

        private Context mContext;

        private CustomManager(Context context) {
            mContext = context;
        }
    }
Now it doesn’t matter where our Context came from, because the reference we are holding is safe.  The application context is itself a singleton, so we aren’t leaking anything by creating another static reference to it.  Another great example of places where this can crop up is saving references to a Context from inside a running background thread or a pending Handler.

So why can’t we always just reference the application context?  Take the middleman out of the equation, as it were, and never have to worry about creating leaks?  The answer, as I alluded to in the introduction, is because one Context is not equal to another.

# Context Capabilities
The common actions you can safely take with a given Context object depends on where it came from originally.  Below is a table of the common places an application will receive a Context, and in each case what it is useful for:

| Application | Activity | Service | ContentProvider | BroadcastReceiver |
|-------------|----------|---------|-----------------|-------------------|
| Show a Dialog | NO | YES | NO | NO | NO |
| Start an Activity | NO(1) | YES | NO(1) | NO(1) | NO(1) |
| Layout Inflation | NO(2) | YES | NO(2) | NO(2) | NO(2) |
| Start a Service | YES | YES | YES | YES | YES |
| Bind to a Service | YES | YES | YES | YES | NO |
| Send a Broadcast | YES | YES | YES | YES | YES |
| Register BroadcastReceiver | YES | YES | YES | YES | NO(3) |
| Load Resource Values | YES | YES | YES | YES | YES |

1. 어플리케이션에서 Activity를 시작할 수 있지만 이는 새로운 task의 생성을 요구한다. 이는 특별한 유스케이스들에는 적합할 것이지만 당신의 어플리케이션에 비표준 백스택 동작들을 만들 수 있다. 그리고 일반적으로 이를 좋은 관례(good practice)로 고려하거나 추천할 수 없다.
2. 이는 적법하다. 하지만 당신의 Application에 정의된 테마가 아니라 당신의 앱이 실행되고 있는 시스템의 기본 테마로 inflation될 것이다.
3. 만약 리시버가 null
안드로이드 4.2 이상에서, 
2. This is legal, but inflation will be done with the default theme for the system on which you are running, not what’s defined in your application.
3. Allowed if the receiver is null, which is used for obtaining the current value of a sticky broadcast, on Android 4.2 and above.

# User Interface
You can see from looking at the previous table that there are a number of functions the application context is not properly suited to handle; all of them related to working with the UI.  In fact, the only implementation equipped to handle all tasks associated with the UI is Activity; the other instances fare pretty much the same in all categories.

Luckily, these three actions are things an application doesn’t really have any place doing outside the scope of an Activity; it’s almost like the framework was designed that way on purpose.  Attempting to show a Dialog that was created with a reference to the application context, or starting an Activity from the application context will throw an exception and crash your application…a strong indicator something has gone wrong.

The less obvious issue is inflating layouts.  If you read my last piece on layout inflation, you already know that it can be a slightly mysterious process with some hidden behaviors;  using the right Context is linked to another one of those behaviors.  While the framework will not complain and will return a perfectly good view hierarchy from a LayoutInflater created with the application context, the themes and styles from your app will not be considered in the process.  This is because Activity is the only Context on which the themes defined in your manifest are actually attached.  Any other instance will use the system default theme to inflate your views, leading to a display output you probably didn’t expect.

# The Intersection of these Rules
Invariably, someone will arrive at the conclusion that these two rules conflict.  There is a case in the application’s current design where a long-term reference must be saved and we must save an Activity because the tasks we want to accomplish include manipulation of the UI.  If that is the case, I would urge you to reconsider your design, as this would be a textbook instance of fighting the framework.

# The Rule of Thumb
In most cases, use the Context directly available to you from the enclosing component you’re working within.  You can safely hold a reference to it as long as that reference does not extend beyond the lifecycle of that component. As soon as you need to save a reference to a Context from an object that lives beyond your Activity or Service, even temporarily, switch that reference you save over to the application context.

For future insights and tutorials, please subscribe to our newsletter below the comments.