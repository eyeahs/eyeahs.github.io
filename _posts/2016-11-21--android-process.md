---
layout: post
category: blog
title: '[번역]Android process 중단이 당신의 앱에 미치는 영향'
date: '2016-11-21 19:00:00 +0900'
categories: Android
tags:
  - Android
published: true
---
원본: https://medium.com/inloop/android-process-kill-and-the-big-implications-for-your-app-1ecbed4921cb#.3h94ew3cz

많은 개발자들이 Android에서 Dependency Injection (예: Dagger)를 사용하고 MVP나 MVVM같은 패턴들을 적용하면서, 이 주제는 그 어느 때보다 더 필요해졌다. 최근에는 내가 “숨겨진 singleton”이라고 부르는 것의 사용이 급증하였고, 동일한 안티-패턴이 별도로 딸려나왔다.

**당신의 Android 애플리케이션(프로세스)가 pause 상태 또는 stop 상태로 들어가면 언제든지 종료될 수 있다.** Activity나 Fragment, View의 상태는 저장될 것이다 — 시스템은 프로세스를 다시 시작하고, 최상단의 activity를 재생성하고 (백스택의 Activity들은 뒤로 복귀할 때 언제든지 재생성될 것이다) 저장된 상태를 가진 Bundle을 받을 것이다.

그리고 여기에 어떤 개발자들은 완전 알아차리지 못한 문제가 있다 — 전체 프로세스는 중단되었다. **따라서 모든 Singleton (또는 “application scope” 객체들), 임시 데이터, 당신의 “retained Fragments”에 저장된 모든 데이터 — 모든 것들은 당신이 애플리케이션을 막 실행시킨 것과 같은 상태에 있을 것이다. 단 하나의 차이점은 — 상태가 복구되면, 사용자는 그가 앱을 떠났던 시점에 있다.**

당신의 Activity는 어떤 공유 객체나 최근 데이터를 가지고 있는 어떤 주입된 의존에 의지한다고 상상해보라. 대부분의 경우 데이터가 null일 것을 예상하지 않았으므로 애플리케이션은 NullPointerException 크래쉬가 발생할 것이다.

## 백그라운드에 있는 애플리케이션의 중단&복구 테스트를 어떻게 해야 할까?

1. 당신의 애플리케이션을 시작하고, 어떤 새로운 Activity를 열고, 어떤 작업을 하라.
2. 홈 버튼을 눌른다(애플리케이션은 정지stop 상태로 백그라운드 상태에 있게 될 것이다).
3. 애플리케이션을 죽인다 — 가장 간단한 방법은 Android Studio에 있는 빨간색 “stop” 버튼을 클릭하는 것이다.
4. 당신의 애플리케이션으로 돌아가라 (최근 앱에서 실행하라)
5. 클래쉬가 발생하였나? 당신의 애플리케이션이 무언가 잘못된 일을 한 것이다 :-)

![백그라운드에서 애플리케이션을 "종료kill”하는 가장 쉬운 방법](https://cdn-images-1.medium.com/max/1600/1*-muHYaKZh6uyylOOz5nLuQ.png)

**이 시나리오에서 디버그 설정에서 “Don’t Keep Activities” 옵션으로 당신의 앱을 테스트하는 것은 확실히 충분하지 않다.** 이 설정은 단지 당신의 Activity들이 자신의 상태를 복구할 수 있는지를 테스트하는 것이다. 하지만 이 방식으로는 process는 절대로 중단되지 않는다. 크래쉬는 당신이 애플리케이션을 배포할 때 발생하기 시작한다. 사용자는 당신의 앱을 열어두고, 백그라운드에 하루 쯤 두었다가 다시 돌아온다.

**개발자 옵션에서 “No background processes”를 설정하는 것도 가능하다.** 애플리케이션을 백그라운드에 두고, 다른 애플리케이션을 실행한 다음 다시 복귀한다 — 애플리케이션 프로세스는 재시작될 것이다 (이는 메모리 부족이나 베터리 절약으로 인한 중단과 동일하다)

![백그라운드 프로세스 제한을 0으로 설정하라](https://cdn-images-1.medium.com/max/1600/1*0Ue0iQx3LxRcZ4gWf4HJdg.png)

## 당신 앱의 트러블 메이커

* Singleton
* 변경할 수 있는 데이터mutable data를 유지하는 모든 공유 인스턴스 (당신이 어떤 상태를 보관하는 주입된 의존들 같은 것)
* Application 클래스에 저장된 데이터와 상태
* 가변 정적 필드 mutable static field
* Retained Fragment (상태는 복구되고, 데이터는 잃어버린다)
* 기본적으로 `onSaveInstanceState` 에 저장되지 않은 모든 것과 거기에 의존하는 당신

여기에는 단독 해결책은 없으며 당신의 애플리케이션의 유형에 달려있다. 일반적으로 당신은 이 목록에 언급된 모든 항목에서 벗어나려고 노력해야 하지만, 이는 항상 쉽거나 가능하지가 않는다.

당신은 상태를 “재초기화reinitialise”할 수 있어야 한다 — Database나 SharedPreferences 둘 중 하나에서 데이터를 불러오거나 또는 필요한 모든 것을 다시 조회하라.

당신의 애플리케이션이 로그인 화면과 타임아웃을 가지고 있을 수 있다 — 이 경우 프로세스 중단kill 시나리오가 감지될 경우 로그인 화면으로 사용자를 돌려보내는 것도 허용 가능한 방식이다.

## Android 플랫폼의 규칙을 알아야한다
모든 아키텍처, 프레임워크 또는 라이브러리는 Android 플랫폼의 규칙에 따라 동작해야 한다. 그러므로 새로운 라이브러리나 접근 방식을 볼 때마다 - 상태 복원을 처리하는지와 그 방법에 대해 생각해보라.

그리고 언제나처럼 — 이 경우에 대해 당신의 애플리케이션을 테스트하라. 이 특정 문제는 간단한 테스트 중과 개발 중에는 절대로 볼 수 없을 것이므로 유해하다. 하지만 최종 사용자는 주기적으로 이 상황을 겪을 것이다.

Follow me and read about possible solutions to this problem in the next article.
