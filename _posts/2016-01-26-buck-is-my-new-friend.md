---
layout: post
title: BUCK Is My New Friend
date: 2016-01-26 01:18:27.000000000 +07:00
image: https://www.ykode.com/assets/posts/buckbuild-logo.png
categories:
- Mobile Development
tags:
- mobile
- build
status: draft
type: post
published: true
---


{% marginfigure 'compiling-prog' 'https://xkcd.com/303/](https://imgs.xkcd.com/comics/compiling.png' %}

Happy belated New Year everybody! New year, new resolutions, new thing. There were a lot of things I
want to write this year and for my first article is about my passion: *software build*. I've spent
my last few years optimising developer productivity, and one most important thing that takes a lot
of precious developer time, other than loading the over-bloated IDE and ever-slow emulator/simulator
is *compiling*. To increase the productivity, we need to minimise and fasten the [feedback
loop](http://lethain.com/feedback-loops-in-software-development/). After [ditching the whole
IDE](/2015/10/20/android-from-scratch-kotlin.html) my next target is to find a faster build system
than Gradle, and luckily [I found one](https://buckbuild.com), and it's called **buck**, and it is
amazing.

<!--more-->

## BUCK? 

Let's start with some knowledge about buck. According to [their site](https://buckbuild.com):

{% marginfigure 'buck-logo' 'assets/posts/buckbuild-logo.png' 'Buck logo according to their site.' %}

> Buck is a build system developed and used by **Facebook**. It encourages the **creation of small,
> reusable modules** consisting of code and resources, and supports a variety of languages on many
> platforms.

At first, I just don't care abouth what is written in the bold above. My goal was to eliminate time
between builds, and it delivers it pretty well. I used it to compile my very basic RxSample project
and compare it with Gradle.

## The Comparison Benchmark

I created a simple test using my Macbook Pro (SSD with 16 GB RAM) with these prerequisites to make
the build is consistently reproducible:

1. All dependencies already been fetched.
2. Startup time is counted as well as the execution time.
3. Test again on rebuilding with minor changes on the `.java` file.

```console
% time ./gradlew assembleDebug 
```

And now with buck:

```console
% time buck build //...
```

The result is shown here:

![Gradle vs Buck](/assets/posts/gradle-buck.png)

- First Build is the time needed for clean build.
- Rebuild is the time needed when some files are changed.
- Nothing to build is the time needed when there's no file changed.

As you can see, it's clear, that Buck wins on all facets. This comparison is very raw as it's only
one project.  Buck excels on a project composed from multiple modules as each modules will be built
in parallel.

## Setting Up Your Project For Buck

Buck is created by encouraging the creation of modules in mind. While gradle is project-based, buck
is very simple file-based build tool. Unfortunately, the IDE support for buck is far from perfect.
To make the developers who use IDE happy, we could just add buck to our project.

The first step to create a buck project is to make a `.buckconfig` file in the project root. When
buck find this file it considers the directory as the project root, and can be referred as `//`. The
`.buckconfig` file is an INI file with entries are [documented in the
documentation](https://buckbuild.com/concept/buckconfig.html).

**.buckconfig**

```
;Setting for Android Build Tools and Target
[android]
build_tools_version = 23.0.2
target = Google Inc.:Google APIs:21

;Maven Repositories, where we will fetch the dependencies
;We only use jcenter
[maven_repositories]
jcenter = https://jcenter.bintray.com/
```

Buck has the concept of [**Build Rule**](https://buckbuild.com/concept/build_rule.html), [**Build
Target**](https://buckbuild.com/concept/build_target.html), and [**Build
File**](https://buckbuild.com/concept/build_file.html). In short, Build File is a file named `BUCK`
that placed in the directory that contains Build Rule to make the Build Target. According to buck
documentation, we can only refer source files in "nearest" build file or closest direct ancestor in
our projet file tree. For more information, please refer to buck documentation.

To make life simpler, we actually only have one `BUCK` file in the same directory as the
`build.gradle` file, in the `app` directory listing several build rules to generate the resulting
APK file. Let's do the dance:

```
% touch app/BUCK && vi app/BUCK
```

### Adding Dependencies

Buck actually does not care about our external dependencies as well as the transitive dependencies.
It declares itself a build tool, not a dependency tool. We can refer the external maven depencies
using `mvn` url parser on `remote_file` build rule and then use `android_prebuilt_aar` and
`prebuilt_jar` to make the file is able to be referenced as dependencies.

```python
# RxAndroid
remote_file(
  name = "rxandroid-aar",
  url  = "mvn:io.reactivex:rxandroid:aar:1.0.1",
  sha1 = "563323d4f90cdb87d067fcf02b06c5d16bb1a258"
)

android_prebuilt_aar(
  name = "rxandroid",
  aar  = ":rxandroid-aar" # Refer by project name references
)

# RxJava
remote_file(
  name = "rxjava-jar",
  url  = "mvn:io.reactivex:rxjava:jar:1.0.14",
  sha1 = "898f9ab61e2a23afd615f0b0389478bec86f49f9"
)

prebuilt_jar(
  name = "rxjava",
  binary_jar = ":rxjava-jar"
)

# RxBinding
remote_file(
  name = "rxbinding-aar",
  url  = "mvn:com.jakewharton.rxbinding:aar:0.2.0",
  sha1 = "97c550ecaf825c003b1b30cb1b754745fa79307a"
)

android_prebuilt_aar(
  name = "rxbinding",
  aar  = :rxbinding-aar"
)

# Android Support annotations
remote_file(
  name = "support-annotations-jar",
  url  = "mvn:com.android.support:support-annotations:jar:23.1.1",
  sha1 = "8d680ba5a623724d1fb0e81c36a790f023a6cede"
)
```

Buck's build rules are Python functions. From the example above, `remote_file()`,
`android_prebuilt_aar()`, and `prebuilt_jar()` are the build rules. Facebook decided buck built-in
build rules with "do one thing" mentality. The `remote_file()` build rule like above is only to
**fetch arbitrary file from somewhere**. 

In contrast with gradle which will reads the maven's `pom` to decide the transitive dependencies,
buck is relatively dumb. It only cares fetching files and matching the SHA1. If you have transitive
dependencies you'd need to specify them one-by-one explicitely as shown by `rxandroid` which depends
on `rxjava` above. The version is also need to be very specific. We cannot use dynamic version. This
is to make sure that the build is **reproducible**. 

Because `remote_file` is only for fetching arbitrary file and do not know the type of the file, the
`android_prebuilt_aar()` and `prebuilt_jar()` refer to the file downloaded by `remote_file` and says
that file need to be treated as `aar` or `jar`. Referring other build rule name can be done by
prefixing it with colon `:`. We're finished adding deps to our projects.

## Building the Application

To create an application, we need to create a [Debug
Keystore](http://developer.android.com/tools/publishing/app-signing.html). I usually avoid checked
in any keystore to the source code. If your developers are working with Android Studio, most of the
time it's saved in default location, in OSX and Linux, it is in `~/.android/debug.keystore`. So we
need to copy them locally.

```python
keystore(
  name = "debug-keystore",
  deps = [":debug-keystore-file"],
  store = "debug.keystore",
  properties = debug.properties,
)
```

You can see that we need to create a `properties` file. This file contains username and password for
debug files.  So let's create a property file:

```
key.alias = androiddebugkey
key.store.password = android
key.alias.password = android
```

Try to execute `buck build //app/...` and see if creates error. With this we're ready to create our
`APK`. But before that, let's see what is actually inside an APK:

1. A dex-ed `classes.dex` coming from `classes.jar`.
2. Resources.
3. Manifest.

Buck is very explicit. To create an APK we need output from `android_library()` with sources and
`android_resources()`. So let's define the resources

```python
android_resource(
  name = "res",
  res  = "src/main/res",
  package = "com.ykode.rxsample"
)
```

This rule will package the resources and generate `R.java`. So we then create an `android_library`
rule containing all classes and output from resources.

```python
android_library(
  name = "app-lib",
  srcs = glob["src/main/java/**/*.java"],
  deps = [
    ":res",
    ":rxbinding",
    ":rxjava",
    ":rxandroid",
    ":support-annotations
  ]
)
```

It lists all references to dependencies that we've been declared before in th `deps` parameter. We
also add `:res` as one of the deps because we need to refer the `R.java` implicitly generated from
previous step.

Now we have complete ingredients, let's create the APK using `android_binary()` build rule:

```python
android_binary(
  name = "apk-debug",
  manifest = "src/main/AndroidManifest.xml",
  keystore = ":debug-keystore",
  package_type = "debug",
  deps = [":res", ":app-lib"]
)
```

This will generate `APK` and placed in the `buck-out/gen` directory.

## Running on device or simulator

If you want to run on device or simulator, first we need to run the emulator. If you are not using
IDE, we can use `nohup` command to invoke the emulator to avoid closing when the terminal emulator
exists.

```console
% nohup emulator -avd <avd_name> > /dev/null 2>&1 &
```

This will run emulator for avd. To get the name of each avd use `android list avd` command. And then
we can run the app by using this buck command.

```console
% buck install --run //app:apk-debug
```

And voila, your app is installed and running. You can change some files and then run the command
again. In my case, the process took **less than 2 seconds** for each rebuild and installation.

![Build Run](/assets/posts/buildrun.gif)

# Conclusion

Buck is very promising build tool. While the first setup is not as easy as gradle, the speed
improvement worth it as it reduce the development time significantly by make feedback loop a lot
faster and unit test and integration testing loaded faster.

I intend to write more specifically about why buck actually enforces you to make modular module and
how I accomplishes it. So, stay tuned. 
