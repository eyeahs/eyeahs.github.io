---
layout: post
category: blog
published: false
title: 'Don''t break the chain: use RxJava''s compose() operator'
---
http://blog.danlew.net/2015/03/02/dont-break-the-chain/

RxJava의 장점들 중 하나는 일련의 operator들을 통해 데이터가 어떻게 변경되는지를 볼 수 있다는 것이다.

	Observable.from(someSource)  
	    .map(data -> manipulate(data))
	    .subscribeOn(Schedulers.io())
	    .observeOn(AndroidSchedulers.mainThread())
	    .subscribe(data -> doSomething(data));

다양한 stream들에서 재사용하고 싶은 operator들의 집합이 있다면 어떻게 될까? 예를 들어 나는 데이터를 작업 스레드에서 처리한 다음 메인 스레드에서 그것을 구독하기 위해 subscribeOn()과 observeOn()을 자주 사용한다. 
What if you have a set of operators that you want to reuse for multiple streams? For example, I frequently use subscribeOn() and observeOn() because I want to process data in a worker thread then subscribe to it on the main thread. It'd be great if I could apply this logic to all my streams in a consistent, reusable manner.

# The Bad Way

The following is the anti-pattern I used for many months and is bad (and I feel bad).

First, create a method that applies the schedulers:

	<T> Observable<T> applySchedulers(Observable<T> observable) {  
	    return observable.subscribeOn(Schedulers.io())
	        .observeOn(AndroidSchedulers.mainThread());
	}

Then, wrap your Observable chain:

	applySchedulers(  
	    Observable.from(someSource)
	        .map(data -> manipulate(data))
	    )
	    .subscribe(data -> doSomething(data));

The code works but it's ugly and confusing - what does applySchedulers() actually apply to? It's no longer a series of operators, so it's hard to follow. There's no way to format this code so it isn't awkward.

Now, just imagine how bad this anti-pattern gets when you use it multiple times for a single stream. *shudder*

# Introducing Transformers

The wise people behind RxJava realized this would be a problem and provided a solution: Transformer, which is used with Observable.compose().

Transformer is actually just Func1<Observable<T>, Observable<R>>. In other words: feed it an Observable of one type and it'll return an Observable of another. That's exactly the same as calling a series of operators inline.

Let's write a method that creates a schedulers Transformer:

	<T> Transformer<T, T> applySchedulers() {  
	    return new Transformer<T, T>() {
	        @Override
	        public Observable<T> call(Observable<T> observable) {
	            return observable.subscribeOn(Schedulers.io())
	                .observeOn(AndroidSchedulers.mainThread());
	        }
	    };
	}

If we're using lambdas this looks a lot prettier:

	<T> Transformer<T, T> applySchedulers() {  
	    return observable -> observable.subscribeOn(Schedulers.io())
	        .observeOn(AndroidSchedulers.mainThread());
	}

Regardless, let's see how our code looks now:

	Observable.from(someSource)  
	    .map(data -> manipulate(data))
	    .compose(applySchedulers())
	    .subscribe(data -> doSomething(data));

That's much better! We've got reusable code and the chain is preserved.

Update 3/11/2015: If you are compiling on JDK 7 or below, you'll have to do a bit of extra work to make compose() work with generics. In particular, you have to tell the compiler the return type, like this:

	Observable.from(someSource)  
	    .map(data -> manipulate(data))
	    .compose(this.<YourType>applySchedulers())
	    .subscribe(data -> doSomething(data));

# Reusing Transformers

In the previous example I used methods to create a new Transformer instance on each invocation. You could create an instanced version and save yourself from unnecessary object instantiation instead. Transformers are about code reuse, after all.

If you're always transforming from one concrete type to another it's fairly simple to create an instance:

	Transformer<String, String> myTransformer = new Transformer<String, String>() {  
	    // ...Do your work here...
	};

What about with our scheduler Transformer? It doesn't care about the type at all, but you can't define a generic instance:

	// Doesn't compile; where would T come from?
	Transformer<T, T> myTransformer;  

You could make it of type Transformer<Object, Object>, but then the resulting Observable would lose its type information and be less useful.

To solve this problem I took a hint from Collections, which has a bunch of type-safe, immutable empty collection creation methods (e.g. Collections.emptyList()). Internally it's using non-generic instances, then wrapping it in a method which adds generics.

Here's how we'd define our schedulers Transformer instance:

	final Transformer schedulersTransformer =  
	    observable -> observable.subscribeOn(Schedulers.io())
	        .observeOn(AndroidSchedulers.mainThread());

	@SuppressWarnings("unchecked")
	<T> Transformer<T, T> applySchedulers() {  
	    return (Transformer<T, T>) schedulersTransformer;
	}

Now we've only got one instantiation - great!

A warning: Whenever you do unchecked casts you can run into trouble. Make sure that your Transformer is truly type agnostic. Otherwise there is the potential that you could run into a ClassCastException at runtime, even though your code compiles. In this case, we know it's safe because schedulers don't even interact with the emitted items.

# What About flatMap()?

At this point, you may be wondering what the difference is between using compose() and flatMap(). They both emit Observable<R>, which means both can reuse a series of operators, right?

The difference is that compose() is a higher level abstraction: it operates on the entire stream, not individually emitted items. In more specific terms:

1. compose() is the only way to get the original Observable<T> from the stream. Therefore, operators that affect the whole stream (like subscribeOn() and observeOn()) need to use compose().

In contrast, if you put subscribeOn()/observeOn() in flatMap(), it would only affect the Observable you create in flatMap() but not the rest of the stream.

2. compose() executes immediately when you create the Observable stream, as if you had written the operators inline. flatMap() executes when its onNext() is called, each time it is called. In other words, flatMap() transforms each item, whereas compose() transforms the whole stream.

3. flatMap() is necessarily less efficient because it has to create a new Observable every time onNext() is called. compose() operates on the stream as it is.

If you want to replace some operators with reusable code, use compose(). flatMap() has many uses but this is not one of them.