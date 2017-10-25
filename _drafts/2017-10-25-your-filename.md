---
layout: post
category: blog
published: false
title: ''
---
## A New Post

http://hannesdorfmann.com/android/mosby3-mvi-1

> Once I have figured out that I have modeled my Model classes wrong all the time, a lot of issues and headache I previously had with some Android platform related topics are gone. Moreover, finally I was able to build Reactive Apps using RxJava and Model-View-Intent (MVI) as I never was able before although the apps I have built so far are reactive too but not on the same level of reactiveness as I’m going to describe in this blog post series. In the first part I would like to talk about Model and why Model matters.

So what do I mean with “modeled Models in a wrong way”? Well, there are a lot of architectural patterns out there to separate the “View” from your “Model”. The most popular ones, at least in Android development, are Model-View-Controller (MVC), Model-View-Presenter (MVP) and Model-View-ViewModel (MVVM). Do you notice something by just looking at the name of these patterns? They all talk about a “Model”. I realized that most of the time I didn’t have a Model at all.

Example: Just load a list of persons from a backend. A “traditional” MVP implementation could look like this:

    class PersonsPresenter extends Presenter<PersonsView> {

      public void load(){
        getView().showLoading(true); // Displays a ProgressBar on the screen

        backend.loadPersons(new Callback(){
          public void onSuccess(List<Person> persons){
            getView().showPersons(persons); // Displays a list of Persons on the screen
          }

          public void onError(Throwable error){
            getView().showError(error); // Displays a error message on the screen
          }
        });
      }
    }

But where or what is the “Model”? Is it the backend? No, that is business logic. Is it the result List? No, that is just one thing our View displays amongst others like a loading indicator or an error message. So, what actually is the "Model"?

From my point of view there should be a “Model” class like this:


