---
layout: post
category: blog
published: false
title: RxJava - Subjects의 문제
---
## A New Post

[원본 post](http://tomstechnicalblog.blogspot.kr/2016/03/rxjava-problem-with-subjects.html)

Subject는 Observable와 Observer 둘 다이다. Observer이므로 아무때나 무엇이든 그것의 onNext()메소드를 호출하고 그것의 Subscriber에게 아이템들을 . 또한 당신은 하나 이상의 존재하는 Observable을 가지고 Subject가 그들을 구독하도록 할 수도 있다. 
Subjects are both an Observable and an Observer. Because it is an Observer, anything at any time can call its onNext() method and push items up to its Subscribers. You can also take one or more existing Observables and have a Subject subscribe to them, and in turn pass their emissions up to the Subject's Subscribers. These features may seem convenient but can quickly encourage anti-patterns.

When I first started learning reactive programming, I was quickly introduced to the Subject and its various flavors like BehaviorSubject, ReplaySubject, and PublishSubject. To a newbie with an imperative programming background, these seemed like magical devices that bridged imperative and reactive operations together. I began to think subjects were a practical way to create source Observables, and I could push items from any source at any time through a chain of reactive operations. Makes sense right? Even some reactive tutorials encourage newbies to play with Subjects to get a feel for reactive programming, but I think this is wrong and creates a setback in a newbie's learning curve.

If you find yourself using Subjects quite often, you might want to reflect on what you think a source Observable does. On a philosophical level, reactive programming is about defining behaviors from a well-defined source all the way to the Subscriber. It is important to think about how emissions should originate and take form. 

But first let's discuss ideally how a reactive chain of Observables should work, and then later it will make sense why Subjects can undermine the integrity of a reactive application.

A Typical Observable Setup
From a practical standpoint, an Observable chain of operations is broken into three parts:

1) The Source Observable where emissions originate
2) Operators to transform the emissions
3) A Subscriber to consume the emissions

Of course, we can make #2 more complicated by using operators that work with other Observables, like merge() and flatMap(). But every Observable by itself should follow this structure. The simplest example would be emitting a fixed set of String values, mapping their lengths, and then printing them. 

//Source
Observable<String> values = Observable.just("Alpha", "Beta", "Gamma");

//Operators
Observable<Integer> lengths = values.map(String::length);

//Subscriber 
Subscription printSubscription = lengths.subscribe(System.out::println);

The three components, the source, the operators, and subscriber are very distinctly identified above. We can alternatively express this in one statement without intermediary variables. 

//Source, Operators, Subscriber
Observable.just("Alpha", "Beta", "Gamma")
    .map(String::length)
    .subscribe(System.out::println);

This entire Observable operation is also agnostic to which thread it is scheduled on, which is good for flexible concurrency and works easily with observeOn() and subscribeOn() as we will read about in Reason 3. But first let's just look at source Observables themselves and analyze their nature.

Reason 1: Keep Source Observables Predictable
What needs to be highlighted with the example above is the source emitting "Alpha", "Beta", and "Gamma". The Observable.just() factory created a source Observable emitting these three Strings. The benefit is the source is tightly controlled and predictable, and we know it will only emit those three Strings. We can trust nothing alien is ever going to enter that source and and be emitted up the chain.

For instance, we should never expect the word "Puppy" to be emitted by the source. We only expect "Alpha", "Beta", and "Gamma".

Observable.just("Alpha", "Beta", "Gamma")
    .map(String::length)
    .subscribe(System.out::println);

However if we use a Subject, this is not guaranteed, especially if it is exposed publicly and anything can call its onNext() method. (Not to mention, this is just downright messy compared to the code above).

PublishSubject<String> subject = PublishSubject.create();

subject.map(String::length)
    .subscribe(System.out::println);

subject.onNext("Alpha");
subject.onNext("Beta");
subject.onNext("Gamma");

//something accidentally pushes "Puppy" as an emission
subject.onNext("Puppy");

subject.onCompleted();

The Subject undermines having a well-thought, strictly-defined source Observable where the emissions come from a predictable source. You also lose some flexibility as the Subject is hot, and you can no longer create a cold Observable as we will read in #2.

This is a simplistic example of course, and more complex applications are not going to emit three emissions like "Alpha", "Beta", and "Gamma". Realistically, you might use RxJava-JDBC to create a source Observable which emits items from a database query. 

Database db = ...

Observable<String> values = db
        .select("SELECT MY_FIELD FROM MY_TABLE")
        .getAs(String.class);

values.subscribe(System.out::println);

While the results of the query can change each time it is subscribed to (because the table can be updated), we have strictly defined the source Observable and where its emissions come from. It will only emit items from this query and nowhere else. 

You can even avoid Subjects for hot sources too. A JavaFX Button being pressed, for example, can emit those action events in a hot Observable. We can use Observable.create() instead of a Subject to turn those events into an Observable. 

Button button = new Button("Press Me");

Observable<ActionEvent> actionEvents =  Observable.create(new Observable.OnSubscribe<ActionEvent>() {
    @Override
    public void call(final Subscriber<? super ActionEvent> subscriber) {
        final EventHandler<ActionEvent> handler = subscriber::onNext;

        button.addEventHandler(ActionEvent.ACTION, handler);

        subscriber.add(JavaFxSubscriptions.unsubscribeInEventDispatchThread(() -> 
            button.removeEventHandler(ActionEvent.ACTION, handler)));
    }
});

actionEvents.subscribe(ae -> System.out.println("Pressed!"));

Even better, we can use RxJavaFX and call a factory that does this for us.

Button button = new Button("Press Me");

Observable<ActionEvent> actionEvents = JavaFxObservable.fromActionEvents(button);

actionEvents.subscribe(ae -> System.out.println("Pressed!"));

This way we can be sure that emissions for this source Observable will only come from this Button being pressed.

To summarize all the examples above, we should tightly control how a source Observable works and strictly define where its emissions come from. Having a Subject haphazardly allow emissions to come from any source can create a chaotic, unmaintainable application.

Reason 2: Mind Your Hots and Colds
The best definition I have heard of a Cold Observable versus a Hot Observable is by Nickolay Tsvetinov, author of Learning Reactive Programming with Java 8:
We can say that cold Observables generate notifications for each subscriber and hot
Observables are always running, broadcasting notifications to all of their subscribers.
Think of a hot Observable as a radio station. All of the listeners that are listening to
it at this moment listen to the same song. A cold Observable is a music CD. Many
people can buy it and listen to it independently.
A cold Observable is "replayable", meaning it will "replay" its emissions to each Subscriber independently. A hot Observable will emit the same item at a given time to all its Subscribers, and it may not even care if any Subscribers are present. 

Some things are more appropriately represented as a hot Observable, such as UI input events like a Button getting pressed. Data-driven Observables, such as the earlier example emitting "Alpha", "Beta", and "Gamma" or the database query, are often better represented as a cold Observable. That data is replayed to each Subscriber that needs it, and being cold guarantees no data was missed previously. Sometimes, it is more efficient to carefully play expensive emissions at the same time to all Subscribers, and this is where a ConnectableObervable can come in handy to turn a cold Observable into a hot one. 

But you need to be careful! Hot Observables are a timing-sensitive animal. This is why it is very important to prefer cold Observables by default. Many operators like flatMap() are in fact a Subscriber to another Observable. When used with hot Observables, this can lead to complicated, confusing cases where the data is being emitted in live time and not every Subscriber is able to capture all the data if they subscribed too late. 

For example, lets say we have three Strings being emitted from a source Observable. We want to subtract their lengths from the min length of the three Strings. We can wrongly use a PublishSubject to hotly emit the Strings to a minLength Observable and a Subscriber that flatMaps against this minLength. 

PublishSubject<String> source = PublishSubject.create();

Observable<Integer> minLength = source
        .map(String::length)
        .reduce((x,y) -> x < y ? x : y);

source.map(String::length)
        .flatMap(len -> minLength.map(min -> len - min))
        .subscribe(System.out::println);

source.onNext("Alpha");
source.onNext("Beta");
source.onNext("Gamma");

source.onCompleted();

But if you run this you may get an error or no console output at all. Why? The three values are being emitted hotly and the minLength can miss the emissions in the flatMap(). This would happen not just with a Subject but any hot Observable, including a ConnectableObservable. 

But if you use the Observable.just() factory, it will coldly replay the values each time it is subscribed to. The flatMap() will subscribe to the minLength, which will subscribe to the source and replay the data each time. This way every part of the Observable chain is given ALL the data. Each piece can replay the CD rather than worrying it missed the radio program.

Observable<String> source = Observable.just("Alpha","Beta","Gamma");

Observable<Integer> minLength = source
        .map(String::length)
        .reduce((x,y) -> x < y ? x : y);

source.map(String::length)
        .flatMap(len -> minLength.map(min -> len - min))
        .subscribe(System.out::println);

Also, is the code just not simpler? Cold Observables tend to make programs simpler and more reliable, and they should be preferred when possible. Yes, you could use caching of some form but this is often not good for performance. It is better for the cold source Observable to define how data is provided.

By the way, also be very mindful of infinite Observables and only use them when your source is truly infinite (like UI input events). Overusing infinite hot Observable can clash with operators that expect a finite set of emissions, and this hardly works well with complex reactive applications.

Reason 3: Thread Safety and Concurrency
Last but definitely not least, SUBJECTS BY DEFAULT ARE NOT THREADSAFE. This is a big reason to avoid them because reactive programming is all about making Observables safely agnostic to which threads they operate on. But if multiple concurrent calls to onNext() are occurring on different threads, it will break the Observable Contract and risk creating race conditions with the operators. Only one thread can be passing through a given operator at a time. But if the source Observable is making several simultaneous onNext() calls, it could have catastrophic behaviors. 

If you use the existing Observable factories and operators with RxJava, you can be safely sure that you will not have this issue. Subjects, incorrectly-built custom Transformers, and bad sources with concurrent calls to onNext() can all wreak havoc.

If you do have to use Subjects for those corner cases, it is always a good idea to turn your Subject into a SerializedSubject. That way if multiple threads do happen to call the onNext() concurrently, it will serialize the emissions safely so it does not break any Observables and Subscribers dependent on it. 

SerializedSubject<Integer,Integer> subject =
      PublishSubject.<Integer>create().toSerialized();

To learn more about concurrency with RxJava, read my article on observeOn() and subscribeOn().
Conclusion
Subjects are abused a lot and can undermine the benefits of reactive programming. Erik Meijer called them the "mutable variables" of reactive programming for good reason. Actually, the problems of mutable variables are almost identical to that of subjects: thread safety, bugs, not being sure where a value change came from, etc. An Observable is an immutable chain of operations from the source to the Subscriber, and the emissions are pushed in a predictable, contained manner. A Subject introduces mutability to that chain and can break it with unpredictable emission sources, lost data, and unwanted thread safety behaviors. 

Of course there are some uses for Subjects and they can come in handy, but I avoid subjects 99.9% of the time and am better off because of it. I am sure there are other technical reasons to avoid Subjects, and these are just my observations and experiences. Let me know if you have other reasons they should be avoided, or corner cases when they should be used.