---
layout: post
category: blog
title: 'Dagger2. provide 메소드 없이 interface 주입하기'
date: '2016-12-10 22:00:00 +0900'
tags:
  - android, dagger, di
published: true
---
MVP 디자인 패턴으로 Android 앱을 개발 중이고 Presenter를 의한 인터페이스 HomePresenter를 만들었다고 가정해보자. 그 직후 HomePresenterImp라는 인터페이스의 구현을 만들었다.

    public interface HomePresenter {
      Observable<List<User>> loadUsers();
    }
    public class HomePresenterImp implements HomePresenter {
      public HomePresenterImp(){
      }  
      @Override
      public Observable<List<User>> loadUsers(){
        //Return user list observable
      }
    }

이 presenter 클래스를 제공할 Dagger 모듈이 필요하다. 구현이 아니라 인터페이스를 주입하고자 하므로 생성자에 직접 @Inject를 추가할 수 없다. 그러므로 우리의 Dagger 모듈은 이렇게 될 것이다:

    @Module
    public class HomeModule {

      @Provides
      public HomePresenter providesHomePresenter(){
        return new HomePresenterImp();
      }
    }

만약 우리가 presenter 구현에 UserService라는 의존을 주입해야 한다면 어떻게 해야할까? 즉, 모듈안의 provide 메서드에 UserService를 추가하고 HomePresenterImp의 생성자에 전달해야 한다. 또한 HomePresenterImp안에 UserService를 필드와 생성자 파라미터에 추가해야 한다.
또는...
다음처럼 @Binds 어노테이션을 모듈에 적용할 수 있다:

    @Module
    public abstract class HomeModule {

      @Binds
      public abstract HomePresenter bindHomePresenter(HomePresenterImp   
        homePresenterImp);
    }

이는 Dagger에게 인터페이스를 주입할 때 어떤 구현이 사용되어야 하는지를 말해준다. 이것이 추상 클래스임을 알아차렸을 것이다. 이는 우리가 추상 메서드를 추가할 수 있음을 의미한다.
이 메서드 서명은 Dagger에게 HomePresenterImp(메서드 파라미터) 구현을 사용하여 HomePresenter 인터페이스(반환 파라미터)을 주입해야 함을 Dagger에게 알려준다.
이제 HomePresenterImp의 생성자에 @Inject 어노테이션을 추가하는 것만으로 충분하다.
더 이상 provide 메소드에 의존 파라미터를 추가할 필요가 없다.
간단히 @Inject를 생성자에 사용한다.

    @PerActivity
    public class HomePresenterImp implements HomePresenter {
      private UserService userService;
      @Inject
      public HomePresenterImp(UserService userService){
       this.userService = userService;
      }
      @Override
      public Observable<List<User>> loadUsers(){
        return userService.getUsers();
      }
    }
    
### TL DR;
인터페이스 주입을 위해 그저 구현 클래스의 생성자를 호출하는 provide 메소드를 가지고 있다면, @Binds 어노테이션을 사용하여 당신의 Dagger 모듈의 상용구 코드를 제거하라.
