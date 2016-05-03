---
layout: blog
category: blog
splash: ""
tags: ""
published: true
title: "Espresso #1 - Espresso setup instructions"
---
[원문](https://google.github.io/android-testing-support-library/docs/espresso/setup/index.html)

이 가이드는 SDK 메니저를 사용하여 Espresso 설치에서 Gradle를 사용한 빌드까지 다룬다. Android Studio가 권장된다.

## 테스트 환경 설정

비정상 동작을 피하기 위해서, 테스트에 사용되는 가상 또는 실제 단말의 시스템 애니메이션을 끄는 것을 강하게 권한다.

* 단말의 Setting->Developer 옵션에서 다음 3가지 설정을 비활성화하라 :
- 창 애니메이션 비율 (Window animation scale)
- 전환 애니메이션 비율 (Transition animation scale)
- Animator 길이 비율 (Animator duration scale)

## Download Espresso

* Extras내의 최신 Android Supoort Repository를 설치하였는지 확인하라 ([가이드](https://google.github.io/android-testing-support-library/downloads/index.html))
* 당신의 어플리케이션의 build.gradle 파일을 열어라. 보통 최상위 build.gradle 파일이 아니라 app/build.gradle이다.
* 다음 라인들을 dependencies안에 추가하라 :

	androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
	androidTestCompile 'com.android.support.test:runner:0.5'

* 상세 내용은 [downloads](https://google.github.io/android-testing-support-library/downloads/index.html) 섹션을 보라 (espresso-contrib, espresso-web, etc.)

# Instrumentation runner 설정

같은 build.gradle 파일의 android.defaultConfig안에 다음 라인을 추가하라.

	testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"

# build.gradle 파일 예제

    apply plugin: 'com.android.application'

    android {
        compileSdkVersion 22
        buildToolsVersion "22"

        defaultConfig {
            applicationId "com.my.awesome.app"
            minSdkVersion 10
            targetSdkVersion 22.0.1
            versionCode 1
            versionName "1.0"

            testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
        }
    }

    dependencies {
        // 테스트에서도 포함되는 어플리케이션의 의존들
        compile 'com.android.support:support-annotations:22.2.0'

        // 테스트에서만 사용되는 의존들
        androidTestCompile 'com.android.support.test:runner:0.5'
        androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
    }

# Analytics

In order to make sure we are on the right track with each new release, the test runner collects analytics. More specifically, it uploads a hash of the package name of the application under test for each invocation. This allows us to measure both the count of unique packages using Espresso as well as the volume of usage.

If you do not wish to upload this data, you can opt out by passing the following argument to the test runner: disableAnalytics "true" (see how to pass custom arguments).

# 첫번째 테스트 추가

안드로이드 스튜디오는 _src/androidTest/java/com.example.package/_에 기본적으로 테스트들을 생성한다.

Rules를 사용한 JUnit4 테스트 예제:

    @RunWith(AndroidJUnit4.class)
    @LargeTest
    public class HelloWorldEspressoTest {

        @Rule
        public ActivityTestRule<MainActivity> mActivityRule = new ActivityTestRule(MainActivity.class);

        @Test
        public void listGoesOverTheFold() {
            onView(withText("Hello world!")).check(matches(isDisplayed()));
        }
    }

# Running tests

## In Android Studio

테스트 설정을 만들어라

안드로이드 스튜디오에서 :

* _Run menu_을 열고 -> _Edit Configurations_
* 새로운 _Android Tests configuration_추가한다
* 모듈을 선택한다

	android.support.test.runner.AndroidJUnitRunner
  
새롭게 만들어진 설정을 실행하라

## From command-line via Gradle

다음을 실행하라.

	./gradlew connectedAndroidTest