---
layout: post
title: "OKHTTP 캐시의 내부는 어떻게 되어 있는가?"
date: "2016-02-13 00:55:46 +0900"
categories: network
published: true
---


원본 : [http://www.schibsted.pl/2016/02/hood-okhttps-cache/](http://www.schibsted.pl/2016/02/hood-okhttps-cache/)

내 생각엔 당신이 구현한 대부분의 앱들은 네트워크 작업을 사용할 것이다. 대부분의 경우 우리는 [OkHttp](http://square.github.io/okhttp/) 라이브러리를 사용하며, 보통 [Retrofit을](http://square.github.io/retrofit/) 통할것이다. OkHttp의 가장 멋진 기능들 중 하나는 그것의 캐시 구조이다.이 포스트에서 나는 캐시에 관련된 모든 단계들을 살펴보고 그것이 어떻게 작동하는지를 설명할 것이다.

내가 OkHttp를 사용할 때 마다 나는 어떤 HTTP 헤더를 사용해야 할 지, 클라이언트 앱으로서 내가 책임져야 하는 것과, 서버측에 기대할 것은 무엇인지 등을 생각할 시간이 필요하다. 그래서 나에게 이 포스트는 미래의 참조 문서가 될 것이다. 내가 계획한 것은 아니지만, 내가 다음 번에 캐쉬를 사용할 땐 확실히 모든 것을 잊어 버렸을 것이다.

이 글은 이미 OkHttp에 익숙한 고급 개발자들을 더 목표로 하고 있다. 나는 최소한 이 라이브러리와 캐시를 활성화하는 방법은 알고 있다는 것을 기대한다. 만약 당신이 그렇지 않다면 OkHttp의 위키 페이지를 먼저 가보도록 하라. 물론 내가 잘못을 할 가능성이 있지만, 난 그저 내가 어떻게 보았는지를 설명하는 것이다. 그러므로 만약 누군가가 실수를 발견했다면, 나에게 알려주길 바란다.

## 소스

다음은 내가 조사하였던 클래스들이다 : [CacheStrategy](https://github.com/square/okhttp/blob/master/okhttp/src/main/java/okhttp3/internal/http/CacheStrategy.java), [HttpEngine](https://github.com/square/okhttp/blob/master/okhttp/src/main/java/okhttp3/internal/http/HttpEngine.java). 나는 모든 로직이 있는 첫번째 클래스에 집중할 것이다. HttpEngine을 언급한 것은 CacheStrategy가 사용되는 곳이기 때문이며, 이로서 당신은 전후 관계를 알게 되었다.

- **CacheStrategy.Factory** 생성자 - 캐시 응답 후보의 헤더들을 읽고 클래스 멤버들로 변환한다.
- **CacheStrategy.getCandidate()** – 캐시 응답 후보를 살펴보고 필요하다면 원본 응답 헤더를 수정한다.

위 도막들은 결정적(crucial)이며 우리가 흥미를 가지고 있는 모든 중요한 정보들을 근본적으로 포함한다. 다른 메소드들 역시 중요하지만 저 두 클래스에서 그것들을 탐색할 수 있다.

## 캐시 후보란 무엇인가?

캐시된 것이 없고 실제로 무엇인가 해야할 것이 많은 API 호출을 해야하는 첫번째 HTTP 요청에 대해서는 설명할 필요가 없다.

일단 응답이 저장되면 우리는 그것을 차후의 호출을 위해 사용하기 위해 시도할 수 있다. 물론 모든 응답이 저장되지는 않는다. 이는 응답 코드에 의거한다. 받아들여지는 것들은 다음과 같다: 200, 203, 204, 300, 301, 404, 405, 410, 414, 501, 308. 302와 307도 있지만 그것들은 다음 조건들 중 하나를 만족해야 한다:

- **Expires** 헤더가 포함되어 있거나 (contains Expires header)
- **CacheControl**이 max-age를 포함하고 있거나
- **CacheControl**이 public을 포함하고 있거나
- **CacheControl**이 private을 포함한다.

부분적인 내용만 캐시하는 것은 지원하지 않는 다는 것에 주목해야 한다.

우리가 요청을 반복하였을 때, OkHttp는 캐시된 응답이 있는지를 확인할 것이다. 나는 그 객체를 캐시 후보로서 이후에 참조할 것이다. (I will refer to that object later as cache candidate.)

## CacheStrategy는 무엇인가?

CacheStrategy는 새로운 요청와 캐시 후보를 받은 뒤 HTTP 헤더를 확인하고 서로를 비교하여 그들 둘을 평가한다.

처음에, 캐시 후보의 헤더에서 일부 구성원들이 저장된다:

- Date
- Expires
- Last-Modified
- ETag
- Age

다음 몇몇 상태들이 확인된다. 여기에 그 목록이 있다:

1. 캐시 후보가 발견되었는지 확인한다. (Check if cache candidate was found.)
2. HTTPS 요청인 경우, In case HTTPS request check if required then the handshake is not missing in cache candidate.
3. 캐시 후보가 캐시 가능한지 확인한다; 사실은 OkHttp가 응답을 저장할 때 수행되는 것과 완전히 동일한 점검이다.
4. Check in request Cache-Control header if no-cache was set, if it’s true the cache candidate is not used and further checks are skipped.
5. Look for If-Modified-Since or If-None-Match in request headers, if one of those is found the cached candidate is not used and further checks are skipped.
6. Perform several time calculations like cache response age, cache freshness lifetime, maximum stale. I don’t want to write all details about it as this would complicate the post too much. You just need to know that headers listed above (Date, Expires…) and max-age, min-fresh, max-stale from request’s Cache-Control were used to calculate them in milliseconds.
The easiest way to show the next check is writing some pseudo code: if ("cache candidate's **no-cache**" && "cache candidate's **age**" + "request's **min-fresh**" **<** "cache candidate's **fresh lifetime**" + "request's **max-stale**")
7. If the condition above is met then our cached response is used and further checks are skipped.
If we get to that point the network operation will be performed. The next checks decide if it is a conditional request or not. Conditional means that we ask our server for response and it may or not return new content or say “use your cached response, it’s all fine :)”.

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
