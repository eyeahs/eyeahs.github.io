---
layout: post
category: blog
title: OKHTTP 캐시의 후드 아래는 무엇이 있을까?
date: '2016-02-13 00:55:46 +0900'
categories: network
published: true
---


[원본](http://www.schibsted.pl/2016/02/hood-okhttps-cache/)

**여러분들은 네트워크 작업을 이용하는 앱들을 구현할 것이다. 대부분의 경우 [Retrofit](http://square.github.io/retrofit/)을 통해 [OkHttp](http://square.github.io/okhttp/) 라이브러리를 사용한다. OkHttp의 가장 좋은 기능 중 하나는 캐시 구조이다. 이 글에서 나는 캐시와 관련된 모든 단계들을 살펴보고 어떻게 작동하는지를 설명할 것이다.**

내가 OkHttp를 사용할 때마다 이것이 어떻게 작동하는지, 내가 어떤 HTTP 헤더를 사용해야하는지, 클라이언트 애플리케이션으로서 내가 책임져야 하는 것이 무엇인지, 서버측에 기대할 것은 무엇인지 등에 생각할 시간이 필요하다. 그래서 나에게 이 글은 미래를 위한 참조 문서가 될 것이다. 내가 이를 계획한 것은 아니지만, 다음 번에 캐쉬를 사용할 땐 확실히 모든 것을 잊어 버렸을 것이다.

이 글은 이미 OkHttp에 익숙한 고급 개발자들을 대상으로 한다. 최소한 이 라이브러리와 캐시를 활성화하는 방법은 알고 있다는 것을 기대한다. 만약 그렇지 않다면 OkHttp의 위키 페이지를 먼저 보라. 내가 잘못을 할 가능성도 물론 있으며, 그저 내가 어떻게 보았는지를 설명하는 것이다. 그러므로 만약 누군가가 실수를 발견했다면, 나에게 알려주길 바란다.

## 소스

다음은 내가 조사하였던 클래스들이다 : [CacheStrategy](https://github.com/square/okhttp/blob/master/okhttp/src/main/java/okhttp3/internal/http/CacheStrategy.java), [HttpEngine](https://github.com/square/okhttp/blob/master/okhttp/src/main/java/okhttp3/internal/http/HttpEngine.java). 나는 모든 로직이 위치하는 첫번째 클래스에 집중할 것이다. HttpEngine을 언급한 것은 CacheStrategy가 사용되는 곳이기 때문이다. 이제 당신은 맥락을 알게 되었다.

- **CacheStrategy.Factory** 생성자 - 캐시 응답 후보의 헤더들을 읽고 클래스 멤버들로 변환한다.
- **CacheStrategy.getCandidate()** – 캐시 응답 후보를 살펴보고 필요하다면 원본 응답 헤더를 수정한다.

위의 비트는 중요하며 기본적으로 우리가 관심있는 모든 중요한 정보를 포함합니다. 다른 메소드들도 역시 중요하지만 저 두 메소드에서 탐색할 수 있다.

## 캐시 후보Cache candidate란 무엇인가?

캐시된 것이 없고 실제로 무엇인가 해야할 것이 많은 API 호출을 해야하는 첫번째 HTTP 요청에 대해서는 설명할 필요가 없다.

일단 응답이 저장되면 이후 호출을 위해 응답을 사용할 수 있다. 물론 모든 응답이 응답 코드에 근거하여 저장되는 것은 아니다. 허용되는 항목은 다음과 같다: 200, 203, 204, 300, 301, 404, 405, 410, 414, 501, 308. 302와 307도 있지만 다음 조건들 중 하나를 만족해야 한다:

- **Expires** 헤더가 포함되어 있거나 (contains Expires header)
- **CacheControl**에 max-age이 있거나
- **CacheControl**에 public이 있거나
- **CacheControl**에 private가 있다.

부분적인 내용만 캐시하는 것은 지원되지 않는다.

요청을 반복하면, OkHttp는 캐시된 응답이 있는지를 확인할 것이다. 나중에 그 객체를 캐시 후보로 참조할 것이다.

## CacheStrategy는?

CacheStrategy는 새로운 요청와 캐시 후보를 취한 뒤 HTTP 헤더를 확인하고 서로를 비교하여 그 둘을 평가한다.

처음에는 캐시 후보의 헤더에서 일부 구성원들이 저장된다:

- **Date**
- **Expires**
- **Last-Modified**
- **ETag**
- **Age**

다음에는 특정 조건들을 검증한다. 여기에 그 목록이 있다:

1. 캐시 후보가 발견되었는지 확인한다.
2. HTTPS 요청의 경우 요구되었는지 확인하여 캐시 후보에서 핸드 셰이크가 누락되지 않을 것이다.
In case HTTPS request check if required then the handshake is not missing in cache candidate.
3. 캐시 후보가 캐시 가능한지 확인한다; 이는 응답을 저장할 때 Okhttp에 의해 수행된 것과 동일한 검사이다.
4. 요청의 **Cache-Control** 헤더에서 no-cache가 설정되었는지 확인하고, 설정되었다면 캐시 후보는 사용되지 않으며 나머지 검사는 건너뛴다.
5. 요청 헤더에서 **If-Modified-Since** 또는 **If-None-Match**를 찾는다. 이 중 하나를 찾으면 캐시 후보는 사용되지 않으며 더이상의 검사는 건너뛴다.
6. 캐시 응답 시간cache response age, 캐시 최신 상태 수명cache freshness lifetime, 최대 유효 기간maximum stale같은 여러 시간 계산을 수행한다. 이 글이 너무 복잡해질 수 있으므로 자세한 내용들은 작성하지 않을 것이다. 위에 열거된 헤더들(Date, Expires...)과 요청의 Cache-Control의 **max-age**, **min-fresh**, **max-stale**이 밀리 초 단위로 계산될 때 사용됨만 알면 된다.
다음의 검사를 표시하는 가장 쉬운 방법은 의사 코드를 작성하는 것이다: if ("cache candidate's **no-cache**" && "cache candidate's **age**" + "request's **min-fresh**" **<** "cache candidate's **fresh lifetime**" + "request's **max-stale**")
7. 이 조건들이 충족되면 캐시된 응답이 사용되며 추가 검사는 건너 뛴다. 이 지점에 도달하면 네트워크 작업이 수행된다. 다음의 검사는 그것이 조건부 요청인지 아닌지를 판별한다. 조건부라는 것은  Conditional means that we ask our server for response and it may or not return new content or say “use your cached response, it’s all fine :)”.

CONDITIONAL REQUEST

This is a new section but in the code we continue with the same method in exactly the same spot as we finished in the previous section.

If cache response contains ETag header, the same ETag value is used in a new request under If-None-Match header.
If point 1 is false and cache candidate contains Last-Modified, then that value is set in request header under If-Modified-Since.
If point 1 and 2 are false and cache candidate contains Date, then that value is set in the request header under If-Modified-Since
If one of these points succeed we have a conditional request, otherwise we end up with an original request and no cache involved.

UPS! THERE IS MORE!

At the end of the process described above, when we leave CacheStrategy.getCandidate(), there is one more thing checked.  The library checks whether the header Cache-Strategy contains the parameter “only-if-cached“. In that case the network will not be asked for new data. OkHttp will be forced to use cache candidate if available or will throw HTTP 504 Unsatisfiable Request.

HOW TO MAKE IT WORK?

As you might have already figured it out yourself it’s all about setting up proper headers when making HTTP calls. Making the default calls without extra headers will work somehow too but I will write about it later.

NO INTERNET

This is one of the first problems I was facing. I thought that just enabling caching will do the trick but it didn’t. There are 2 solutions I know:

First one is about using Cache-Control : only-if-cached header. It will never reach server. Only check if there is available response cached and return it. Otherwise, 504 will be thrown, so don’t forget to handle that exception.
Second solution is about using Cache-Control : max-stale=[seconds] header. That one is more flexible. It indicates that client is willing to accept cached response if it’s not exceeding specified lifetime. It is verified by OkHttp, so there is no need to connect with the server. Although, if the time is exceeded, then the network operation is performed and we are getting new content from the server.
CARE FOR USER COSTS AND SPEED

The above solutions work out the offline mode and it’s all about OkHttp verification. Usually our apps are online and we would like to get new data and getting the same thing from the server over and over again generates redundant costs for users and slow down our app.

In a perfect situation we would send a request and get really new data. When data is not changed on the server we get empty data with 304 HTTP response code and OkHttp return cached response. That can be achieved only if your server is supporting caching. So basically what you need to do is checking with your back-end colleagues if ETag or Last-Modified-Since is supported. That’s it! OkHttp will do the rest for you.

NOTE: Don’t try to set If-Modified-Since or If-None-Match yourself. If you look at the point 5 in the section CACHE STRATEGY you will notice that setting these will skip further checking and perform network operation.

FORCE NETWORK

You can also force a network operation. Personally I have never had a chance to use this option but I see some possible scenarios. Let’s say you use max-stale that reaches the server after some longer time span. You may have a case when you want to allow the users to force refresh in some special situation. Nothing simpler. Use Cache-Control : no-cache in your request and you will get the new data from the server.

CONCLUSION

In my opinion OkHttp is fairly simple to use and does the most work for us. However, it’s good to understand the flow, to know which headers can be set and what they do. I hope that this article will help you.
