---
layout: post
category: blog
published: false
title: RxJava Completion Semantics
---
[원본](http://adelnizamutdinov.github.io/blog/2015/01/23/using-rxjavas-observable-semantics-for-greater-good/)

내가 처음 RxJava를 시작할 때, 나에게 RxJava는 단지 간편하고, 파라미터적인 동시성을 위한 것이였다. 이후 나는 함수적 구성과 진정한 코드 재활용의 힘을 발견하였다. 놀라는 것에서 멈추지 않고, `Observable`의 completion semantics와 `Subscriber.add(Subscription)` 메소드에서 이야기하고자 한다.

## Unsubscription callback

간단한 OkHttp 호출을 위한 간단한 `Observable` 구현을 보자. 특별히 꾸민 것 없이, 호출을 만들고 응답을 스트림에 전달하고 즉시 종료한다:

    Observable<Response> call(OkHttpClient client, Request request) {
      return Observable.create(subscriber -> {
        final Call call = client.newCall(request);
        try {
          subscriber.onNext(call.execute());
          subscriber.onCompleted();
        } catch (IOException e) {
          subscriber.onError(e);
        }
      });
    }

이것은 동작한다. 하지만 우리는 종종 많은 양의 Http를 만들고 그 응답들 중 일부는 무시할 필요가 있다: 예를 들면, 입력란에 자동 완성을 추가할 때이다.

RxJava는 이런 종류의 문제들을 잘 처리한다. 구독 해지를 위한 콜백을 추가하면 된다:

    Observable<Response> call(OkHttpClient client, Request request) {
      return Observable.create(subscriber -> {
        final Call call = client.newCall(request);
        subscriber.add(Subscriptions.create(call::cancel));
        try {
          subscriber.onNext(call.execute());
          subscriber.onCompleted();
        } catch (IOException e) {
          subscriber.onError(e);
        }
      });
    }

이제 우리가 `Subscription.unsubscribe()`이나 `Observable.takeUntil(Observable)`를 사용하면, RxJava 는 불필요한 네트워크 부하를 줄이기 위해 `Call.cancel()`를 호출 할 것이다. 그리고 우리는 Observable의 강력한 semantics 덕분에 이것을 완전히 꽁짜로 얻을 수 있다.

## 오래 지속되는 프로세스

안드로이드 세계로 가서 MediaPlayer 클래스를 살펴보자. 이것은 SDK 전체에서 가장 "명령적" 클래스 중 하나이다. 내포된 상태, 거의 모든 메소드에서 예외 발생, 그리고  기묘한 오류 전파. 그러면 어떻게 해야 이것을 안전하고 예측 가능한 RX 세상으로 가져올 수 있을까?

![img](http://developer.android.com/images/mediaplayer_state_diagram.gif)

먼저 이것이 어떤 종류를 발행하는 지를 알아내야 한다. `MediaPlayer`는 `Integer`이다:

만약 ongoing process를 Rx 스트림으로 나타내고 싶다면, 우리는 먼저 이것이 발행하는 값들의 종류를 알아내야 한다. `MediaPlayer`는 재생 시간이다 - `Integer`:

    Observable<Integer> stream(MediaPlayer mp) {
        return Observable.create(subscriber -> {
            subscriber.add(Subscriptions.create(() -> {
                if (mp.isPlaying()) {
                    mp.stop();
                }
                mp.reset();
                mp.release();
            }));
            mp.start();
            subscriber.add(ticks(mp)
                                   .takeUntil(complete(mp))
                                   .subscribe(subscriber));
        });
    }

    Observable<Integer> ticks(MediaPlayer mp) {
        return Observable.interval(16, TimeUnit.MILLISECONDS)
                .map(y -> mp.getCurrentPosition());
    }

    Observable<MediaPlayer> complete(MediaPlayer player) {
        return Observable.create(subscriber -> player.setOnCompletionListener(mp -> {
            subscriber.onNext(mp);
            subscriber.onCompleted();
        }));
    }

메인 아이디어는 이렇다:

1. `ticks`의 스트림은 `MediaPlayer`의 현재 위치에 매핑되는 인터벌 observable이다. 그리고 초당 60번 작동한다.
2. `complete`스트림은 단 하나의 항목만 발행한다 - `MediaPlayer`가 재생을 멈추는 순간.
3. Player를 `start()`하면, 정지되거나 `unsubscribed`될 때 까지 ticks를 발행한다.
4. tick 역시 구독 해지시 멈출 수 있도록 두 번째 `Subscription`을 추가한다. 이것을 외부 `Observable`로 호출하기 때문이다.
5. 가장 중요한 부분은 player가 `released`되는 첫번째 unsubscription 콜백이다.

이제 우리가 이 스트림을 구독할 때 마다, 음악 재생을 시작하고 현재 재생 위치를 발행한다. 만약 중지하면 player를 릴리즈한다. 만약 구독을 해지하면 player를 릴리즈한다. 만약 에러가 발생하면 player를 릴리즈한다.

**비동기** `Subscriber.add(Subscription)`를 **동기** `try-catch`의 `finally`블록으로 생각할 수 있다. 오래 동작하는 작업을 stream으로 표현할 수 있고, 자동적인 청소와 적절한 오류 전파를 꽁짜로 얻을 수 있다. 이것은 어디에서나 가능하다: 전화 통화, 긴 http 폴링, 오디오 녹음. 심지어 지옥같은 UI 에니메이션에서도!

이 방식은 비동기로 오래 동작하는 어떤 복잡한 작업을 간단하고 구성가능한 `Observable`로 바꿀 것이다.

    Observable<Pair<Integer, Integer>> play(MediaPlayer mp) {
        return prepare(mp).flatMap(RxMediaPlayer::stream);
    }

계속 구성하라!

### PS

[Here’s the complete code for the `RxMediaPlayer`](https://gist.github.com/adelnizamutdinov/8bc276f2eda05536bf0c)

[And bonus code for the `RxMediaRecorder`](https://gist.github.com/adelnizamutdinov/d1a99de548ee9629513d) :)

Please note that they’re not production-ready because they both lack `OnErrorListener`s for decent error propagation and recovery.