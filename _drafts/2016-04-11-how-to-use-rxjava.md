---
published: false
---

## How To Use RxJava
{% highlight java %}
{% endhighlight %}

# Hello World!

{% highlight java %}
  public static void hello(String... names) {
      Observable.from(names).subscribe(new Action1<String>() {
  
          @Override
          public void call(String s) {
              System.out.println("Hello " + s + "!");
          }
  
      });
  }
{% endhighlight %}

{% highlight java %}
hello("Ben", "George");
Hello Ben!
Hello George!
{% endhighlight %}


Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.
