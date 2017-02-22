---
layout: post
title: Reactive Programming with RxJava in Android
date: 2015-02-20 01:01:27.000000000 +07:00
image: https://www.ykode.com/assets/posts/RxAndroid/RxAndroid.jpg
categories: programming mobile
---

Every UI element on your Android application emits events. The code that you've been writing is a reaction to some
event, such as user tapping a button or a REST response from your backend service. We usually handle it by multiple
instances of event listeners such as `OnClickListener`. All is well until you try track and handle multiple states.
The common example is validation of registration form. To tackle this, smart people from Netflix have made a nifty
little library called [RxJava](https://github.com/ReactiveX/RxJava) and its binding for Android named
[RxAndroid](https://github.com/ReactiveX/RxAndroid). This library will add [Functional Reactive
Programming](http://en.wikipedia.org/wiki/Functional_reactive_programming) extension to your Android application to
handle complex asynchronous multiple event handling. This tutorial is loosely based on Ray Wenderlich's excellent
tutorial on [ReactiveCocoa on iOS](http://www.raywenderlich.com/62699/reactivecocoa-tutorial-pt1). 

<!--more-->

## Functional Reactive Programming?

Functional Reactive Programming consists of two models of programming combined. They are, in short:

- Functional Programming. A model of programming that transform and compose stream of immutable sequences by
  applying `map`, `filter` and `reduce`. Events are immutable because you can't change history.
- Reactive Programming. This model of programming focuses on data flow and change propagation.

The easy analogy for FRP is a spreadsheet application. You can chain the formula. And whenever the source cell is
changed, the result will change too. The formula itself does not change, but the change of value from the source
cell propagates to the result cell.

## FRP with RxJava

In Java environment (Android included), FRP Functionalities are provided by RxJava. RxJava is a port of Rx, a
reactive extension for Microsoft .NET platform ported to JVM. 

### Observable and Observer

RxJava uses [Observer Pattern](http://en.wikipedia.org/wiki/Observer_pattern).  The basic building
block of RxJava is `Observer`{% sidenote 'obs-sidenote' 'The smallest building block is actually an
`Observer`, but in practise we will most often using `Subscriber` because that is how we hook up the
`Observables`'%}.  `Observable` emits data and a `Subscriber` consumes those data. The difference
with standard Observer pattern is `Observable` doesn't start working until there's subscribers that
subscribe to them.

The very basic "Hello World" code for RxJava can be written as follows.

```java
package com.ykode.belajar;

import rx.Observable;
import rx.Subscriber;
import rx.functions.Action1;

public class Hello {
  public static void main(String [] args) {
    Observable<String> helloObservable = Observable.create( // .... [1]
      new Observable.OnSubscribe<String>() {
        @Override
        public void call(Subscriber<? super String> sub) {
          sub.onNext("Hello, World!");
          sub.onCompleted();
        };
    });

    helloObservable.subscribe( new Action1<String>() {      // .... [2]
      @Override
      public void call(String s) {
        System.out.println("Subscriber : " + s);
      }
    });
  }
}
```

WTF! Very long code, 26 LOC for a simple hello world! This code is written with Java 7 syntax in mind, hence the
verbosity. This is one reason why [I hate Java](/2013/06/22/thoughts-on-java-and-jvm.html). Let's bear with it for
a while. So what's happening? And what we're getting with this code?

1. The `helloObservable` will emits "Hello World". 
2. `helloObservable` is being subscribed. The subscriber is an instance of `Action1`. It takes 1 parameter
   in form of a `String` and print them.

We can modify the code with two subribers and print the same word on second subscriber.

```java

    helloObservable.subscribe( new Action1<String> {
      @Override
      public void call(String s) {
        System.out.println("Sub1 : " + s);
      }
    });

    helloObservable.subscriber( new Action1<String> {
      @Override
      public void call(String s) {
        System.out.println("Sub2 : " + s);
      }
    });
```

This code will add two subscribers to our code and will print "Hello World!" in both subscribers' code paths.

```
Sub1 : Hello, World!
Sub2 : Hello, World!
```

If you change the string from "Hello World!" to "Bonjour Monde!". All subscribers will print the changed code. This
is the **change propagation** that shapes the reactive programming.

### Transformation

The "Functional" part of FRP allows us to transform `Observable` such as `map` and `filter`. To make the example
more clear, let's change the `Observable` code to take the items from an `Iterable`. We'll use `List` of `String`
containing me and my friends' name.

```java
package com.ykode.belajar;

import rx.Observable;
import rx.Subscriber;
import rx.schedulers.Schedulers;
import rx.functions.Action1;

import java.util.Arrays;
import java.util.List;

public class Hello {

  // Getting current thread name
  public static String currentThreadName() {
    return Thread.currentThread().getName();
  }

  public static void main(String [] args) {

    List<String> names = Arrays.asList("Didiet", "Doni", "Asep", "Reza", 
        "Sari", "Rendi", "Akbar");

    Observable<String> helloObservable = Observable.from(names); // ... [1]

    helloObservable.subscribe( new Action1<String>() {
      @Override
      public void call( String s ) {
        String greet = "Sub 1 on " + currentThreadName() + 
            ": Hello " + s + "!";
        System.out.println(greet);                               // ... [2]
      }
    });     
}

```

This code will print out greetings to each name from `names`.

1. The `Observable` is built using `from` method. The subscribers will receive the item from the `Iterable`
   collection one by one.
2. The subscriber will then print greeting, with the name appended.

The result is as follows:

```
Sub 1 on main: Hello Didiet!
Sub 1 on main: Hello Doni!
Sub 1 on main: Hello Asep!
Sub 1 on main: Hello Reza!
Sub 1 on main: Hello Sari!
Sub 1 on main: Hello Rendi!
Sub 1 on main: Hello Akbar!
```

If we want to print an uppercased name of the person. We can use `map` method from observable. The `map` method
will transform the value. 

```java
import rx.functions.Func1;  // ..... [1]

// ... code redacted ...

    helloObservable.map( new Func1<String, String>() {
      @Override
      public String call( String s ) {
        return s.toUpperCase();
      }
    }).subscribe( new Action1<String>() {
      @Override
      public void call( String s ) {
        String greet = "Sub 1 on " + currentThreadName() + 
            ": Hello " + s + "!";
        System.out.println(greet);
      }
    });

```

The subscriber will print a greeting with uppercased names.

```
Sub 1 on main: Hello DIDIET!
Sub 1 on main: Hello DONI!
Sub 1 on main: Hello ASEP!
Sub 1 on main: Hello REZA!
Sub 1 on main: Hello SARI!
Sub 1 on main: Hello RENDI!
Sub 1 on main: Hello AKBAR!
```

### Lambda expression

As you can see the code for transformation and subscription is very verbose. You'd need to use `Action` classes
such as `Action1`, `Action2` and so on or `Func` classes such as `Func1`, `Func2` and so on if you intend to return
values. They are generics, and you'd need to specify the parameters and return type to build that. They also have
only one method `call`. This boilerplate code eschews many people, including me. Luckily in Java 8, we have lambda
expression. With lambda expression our last code will be so much smaller.

```java
    helloObservable.map( s -> s.toUpperCase() )
      .subscribe( s -> {
        String greet = "Sub 1 on " + currentThreadName() +
          ": Hello" + s + "!";
        System.out.println(greet);
      });
```

Yap! 13 lines of code becomes 6 lines of code. To make it more readable, lets build the greeting string using `map`.

```java
    helloObservable.map( s -> s.toUpperCase() )
      .map( s -> "Sub 1 on " + currentThreadName() + ": Hello " + s + "!")
      .subscribe( s -> System.out.println(s));
```

Shazam! 3 lines of code. We shorten the `greet` variable to `s` without making the code looks convoluted. The new
code is even more readable than before.

## Doing it in Android.

_**Important Update:** The code on this article is outdated and leaking memory. I've fixed the leak and updated the
build script and dependencies on [my newer article](/2015/10/22/android-rxjava-rxandroid-memory-leak.html)._

The easiest way to explain is by use case. We'll create a registration form. This registration form will
take two fields. One for email, and the other is for user name. The registration form has these behaviours:

- The initial state of both fields are blank.
- If the field is valid according to the criteria, the text will be colored black.
  - Criteria for email is a valid email using simple regex check.
  - Valid user name if it's more than 4 character in length.
- If the field is not valid, the text will be colored RED.
- The **Register** button is only active when both fields are valid.

If you used to create Android applications, you'd now how many handlers and anonymous classes you will write to track
the state of the fields and to create the behaviour of the register button.

### Preparation

Create a simple Android application using [Android Studio](http://tools.android.com/download/studio). You can name
your application anything, I'd name my application `RxSample`.

![New Project](/assets/posts/RxAndroid/02_RxSample.png "New Project RxSample")

And then create a simple Activity app with Fragment and run at least on Jelly Bean. We want this simple app to be
as modern as possible. I then add two fields using `LinearLayout` and one `Register` button. The resulit is more or
less like below. The `id` of each control is written in red texts. I tested it on Genymotion and looks like the
layout works well. 

![Layout](/assets/posts/RxAndroid/03_AndroidLayout.png "Layout") 


### Adding RxAndroid and Retrolambda

We'd like to use RxJava on Android. We can use the plain RxJava if we want. However, there's a nice wrapper called
RxAndroid available. Open your `build.gradle` for the **app** module and then add this line to the `dependencies`
section, so it becomes:

```groovy
dependencies {
  compile fileTree(dir: 'libs', include: ['*.jar'])
  compile 'com.android.support:appcompat-v7:21.0.3'
  compile 'io.reactivex:rxandroid:0.24.0'
}
```

This will add RxAndroid support to your application. And ready for reactive goodness.

How about lambda expression? Unfortunately, Android SDK doesn't support lambda expression as it only supports Java
6 compatible bytecode.  This will cause error on compilation when compiling from java bytecode to dex. If you want
to stay out of hackery, then you'd need to use functions objects with a tradeoff of verbosity. 

I don't want to be verbose, so I'll take a little risk. Introducing RetroLambda, a tool to convert your Java 8
bytecode to Java 6 bytecode so you can use lambda expression. A little bit of warning tough, that this tool is a
form of hackery, there's no guarantee that the tool will be compatible with future Android SDK. But for now, with
SDK 5.0, looks like it works so well, except there's a little problem with the Google Play services. This is no
showstopper as there's workaround for that.

First of all, let's use retrolambda gradle plugin. This can be done by adding `classpath` dependencies on the root
`build.gradle`, not in the **app**.

```groovy
    dependencies {
        classpath 'com.android.tools.build:gradle:1.0.1'
        classpath 'me.tatarka:gradle-retrolambda:2.5.0'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
```

And then we apply the plugin on the **app** gradle.

```groovy
apply plugin: 'com.android.application'
apply plugin: 'me.tatarka.retrolambda'

```

If you're using Android Studio, it will use Java 7 language level and prevents you to use lambda expression. To
avoid this, add these lines to your `android` section of your configuration.

```groovy
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
```

This will force the IDE to use Java 8 as the language level for syntax highlighting. If you want to use Google Play
Services newer than `5.0.77` you'd need to configure retrolambda to be able to compile your app.

```groovy
retrolambda {
  jvmArgs '-noverify'
}
```

And lastly, we'd need to configure proguard by adding this line to the proguard rules file. If you're using Android
Studio, the file name is **proguard-rules.pro**

```
-dontwarn java.lang.invoke.*
```

Sync your project and we're ready for reactive goodness.

## Capturing Events

From earlier example. We've observed a single value, and also iterable. In Android, what we observe is a list of
events. We'll start with the event on `edtUserName`. We'd like to capture the text and print them in the ADB
console. Setting up the Android observers should be done on `onStart` lifecycle.

```java
=@Override
protected void onStart() {
  super.onStart();

  Observable<OnTextChangeEvent> userNameText = 
    WidgetObservable.text((EditText) findViewById(R.id.edtUserName));

  userNameText.subscribe( e -> Log.d("[Rx]", e.text().toString()));
}
```

When we run this app, we'll get these output.

![Android Animation 1](/assets/posts/RxAndroid/04_RxAndroid_Text.gif)


### Filtering Event

Let's change it a little bit. We'd only want to print the text when the length of the text is more than 4.  This is
our first condition of valid user name.

```java
userNameText.filter( e -> e.text().length() > 4)
  .subscribe( e -> Log.d("[Rx]", e.text().toString()));
```

Run the app once again we'll see the difference.

![Android Animation 1](/assets/posts/RxAndroid/05_RxAndroid_Filtered.gif)

Here, we've set up a very simple pipeline for the data flow. This is how we express functionality in Reactive
Programming. To make it easy to grok, I've drawn a picture of the data flow diagram.

![Data Flow 1](/assets/posts/RxAndroid/06_Pipeline_Filtered.png)

### Mapping/Transforming Event

Now, let's implement the behaviour of the validity of each field. Map the text from each fields and verify.
We'd build a pipeline similar to this. Well apply our knowledge of using `map` on our Android code.

![Data Flow 2](/assets/posts/RxAndroid/07_Pipeline_Color.png)

To achieve this we'll define several `Observable`s for each `EditText`. The condition of validity are

1. UserName should be more than 4 characters in length.
2. Email should match a simple regex validaton.

```java
    @Override
    protected void onStart() {
        super.onStart();

        final Pattern emailPattern = Pattern.compile(
                "^[_A-Za-z0-9-\\+]+(\\.[_A-Za-z0-9-]+)*@"
                + "[A-Za-z0-9-]+(\\.[A-Za-z0-9]+)*(\\.[A-Za-z]{2,})$"); // ..... [1]

        EditText unameEdit = (EditText) findViewById(R.id.edtUserName);
        EditText emailEdit = (EditText) findViewById(R.id.edtEmail);

        Observable<Boolean> userNameValid = WidgetObservable.text(unameEdit) //  [2]
                .map(e -> e.text())
                .map(t -> t.length() > 4);                                     

        Observable<Boolean> emailValid = WidgetObservable.text(emailEdit) 
                .map(e -> e.text())
                .map(t -> emailPattern.matcher(t).matches());

        emailValid.map(b -> b ? Color.BLACK : Color.RED)
                .subscribe( color -> emailEdit.setTextColor(color));     // ... [3]

        userNameValid.map( b -> b ? Color.BLACK : Color.RED)
                .subscribe( color -> userNameEdit.setTextColor(color));  


    }

```

The code is pretty straightforward.

1. This is the email pattern we'll use for email validation. We're using classes from `java.util.regex` package.
2. We create an `Observable<Boolean>` for the UserName and email.
3. We map the validity `Observable` to a color observable and subscribe to them to change the text Color.

The result is as expected.

![Color](/assets/posts/RxAndroid/08_Sim_Color.gif)

### Branching and Combining Event

Lastly, we'd like to implement a use case where the Register button only enabled when both fields are enabled. For
this we'll need to combine two observables to one. To do that we use `combineLatest` method. Append this code to
`OnStart`.

```java
Button registerButton = (Button) findViewById(R.id.buttonRegister);

Observable<Boolean> registerEnabled = 
  Observable.combineLatest(userNameValid, emailValid, (a,b) -> a && b);
registerEnabled.subscribe( enabled -> registerButton.setEnabled(enabled));
```

The `combineLatest` method takes two or more `Observable`s and a `Func2` or a lambda expression containing operation
to combine both `Observable` results. The combination will emits another `Observable` with the combined result from
both sources `Observable`s.

![Enabled](/assets/posts/RxAndroid/09_Sim_Enabled.gif)

As you can see, the text fields retain their behaviour and also affects the behaviour of the Register Button
according to their validity condition. The final pipeline is below.

![Full Pipeline](/assets/posts/RxAndroid/10_Pipeline_Full.png)

From the pipeline above, we can see that we branch the signals and combine them to one signal to create a behaviour
of the register button. All of them are done in less than 20 lines of code if you're using both RxAndroid and
lambda expression.

### Distinct

Our application behaves as expected. However there's a small minor detail that might affect the performance of the
code. The items from `userNameValid`, `emailValid` and `registerEnabled` are emitted on each user input. This
causes the subscribers are called **every time the user type in the fields**. To prove this, we'd need to modify
the code a little bit.

```java
emailValid.doOnNext( b -> Log.d("[Rx]", "Email " + (b ? "Valid" : "Invalid")))
  .map(b -> b ? Color.BLACK : Color.RED)
  .subscribe(color -> emailEdit.setTextColor(color));

userNameValid.doOnNext( b -> Log.d("[Rx]", "Uname " + (b ? "Valid" : "Invalid")))
  .map(b -> b ? Color.BLACK : Color.RED)
  .subscribe(color -> userNameEdit.setTextColor(color));

// and the registerenabled

registerEnabled.doOnNext( b -> Log.d("[Rx]", "Button " + (b ? "Enabled" : "Disabled")))
  .subscribe( enabled -> registerButton.setEnabled(enabled));

```

When we run the application, the ADB logcat will show that the subscriber is called on every letter input
regardless of the validity status change.

![NoDistinct](/assets/posts/RxAndroid/11_Sim_NoDistinct.gif)

This is not the behaviour we want. We only want the subscriber only called **when the validity changes**. For this,
we'd need to call `distinctUntilChanged()` method on our `Observable`s. Thus, the code becomes:

```java
emailValid.distinctUntilChanged()
  .doOnNext( b -> Log.d("[Rx]", "Email " + (b ? "Valid" : "Invalid")))
  .map(b -> b ? Color.BLACK : Color.RED)
  .subscribe(color -> emailEdit.setTextColor(color));

userNameValid.distinctUntilChanged()
  .doOnNext( b -> Log.d("[Rx]", "Uname " + (b ? "Valid" : "Invalid")))
  .map(b -> b ? Color.BLACK : Color.RED)
  .subscribe(color -> userNameEdit.setTextColor(color));

// and registerEnabled

registerEnabled.distinctUntilChanged()
  .doOnNext( b -> Log.d("[Rx]", "Button " + (b ? "Enabled" : "Disabled")))
  .subscribe( enabled -> registerButton.setEnabled(enabled));
```

This way, the item will only be emitted if it changes and only call the subscriber when it does change. Let's see
the difference.

![Distinct](/assets/posts/RxAndroid/12_Sim_Distinct.gif)

We can then delete the `doOnNext` method so we don't log into logcat.

## Conclusion

Functional and Reactive Programmig is the new hotness on modern concurrent and parallel event driven applications.
RxJava and RxAndroid is a way to tap that potential. This article is only a tip of the iceberg of the power and
abilities of RxJava and RxAndroid. Hopefully, you can get a good fundamental knowledge after reading this article.
The basics might needs a while to wrap on your brain, but once you get it, it's simple. `Observable`s are stream of
events and we define the behaviour by transforming and combining them.

The main goal of Rx is to make your code cleaner and easier to test and understand. With the help of lambda
expression, your code is more expressive than without it. By chaining the observers, we can see the whole pipeline.
I think it's a lot cleaner than defining handlers and anonymous classes and state tracking instance variable.

Wait for next article when I write about multithreading in RxJava as well as how we do network call and treat them
as `Observable`.

The complete code for this article can be downloaded from [YKode Github](https://github.com/ykode/RxSample).




