---
layout: post
title: Bumpy Road to Android NDK
date: 2014-02-24 07:28:09.000000000 +07:00
categories:
- opinion
---
![Bumpy Road](/assets/posts/bumpy-road.jpg)

I was very agitated with the fact that doing native development on Android
actually sucks. There are many flowery stories about android, from market
dominance to adoption of it by many manufacturers, or even Google's own
competitors.  However, there's one story that will leave you a nightmare. It's
Android Native Development.  

<!--more-->

My project is mixed between platform code written in Java and Rendering code
written in C++. Unlike games who can just remove java code and do what you want
using `NativeActivity`, nor it is a pure-Java project. It sits in-between. It's
a custom view with things rendered inside on another thread, [like this
one](http://altdevblogaday.com/2013/08/18/the-main-loop-in-devils-attorney-on-
android/).

People who only use Java to create Android applications won't experience this.
I underestimated the difficulties by assuming that because of Android is at
least linux and posix, it'll be more or less easy like doing the same thing
with BlackBerry 10 or iOS until I knew it was not. So these are the things
that makes me very agitated and even posting my commit messages full of
profanity. Well, also some notes there. I had rather big C++ codebase I worked
before that should be easy to port as it's only needs to reimplement the
context creation and image loading, but it actually wasn't that easy.

## The Java Native Interface Shenaningans

JNI stands for Java Native Interface. Or, rather, Java **Nightmare**
Interface. If you ever know how it looks like, it's like this:

```c
/*  
 * Class: io_nomad_compo_CompoBridge  
 * Method: initWithActivity  
 * Signature: (Landroid/app/Activity;)V  
 */  
JNIEXPORT void JNICALL Java_io_nomad_compo_CompoBridge_initWithActivity  
(JNIEnv *, jclass, jobject);  
```


That's the way java interact with your native code. It looks like a hairy
hedgehog. So I usually prettify the function name. But it leads to another
problem: manual native registration. I managed to do that after employing my
Google jujitsu. I think Google purposely made Android NDK hard to program, so
it can get my search data. Not to mention that there's no reference online, I
need to read their html docs. I decided to go to [MobilePearls' excellent
documentation on Android
NDK](http://mobilepearls.com/labs/native-android-api/).

## The AssetManager Complexity

I'm relatively new in Android. I barely touch its development for a week. I
thought it was just a normal `fread` calls to access `assets/` path. But I was
wrong. I need to ping-pong `AssetManager` object by attaching it on first
`Activity` shown. The code more or less looks like this.

```c 
JNIEnv *env;  
g_VM->AttachCurrentThread(&env, NULL);

if (g_JavaAssetManager != 0) {  
  LOGI("AssetManager already attached in [0x%X]",  
    (unsigned) mJavaAssetManager);  
  return;  
}

g_JavaAssetManager = env->NewGlobalRef(manager);  
g_AssetManager = AAssetManager_fromJava(env, manager);

if (manager == 0 ) {  
  LOGE("Asset Manager Attachments Failed %x", (unsigned) manager);  
}  
```

This way I can refer to both AssetManager instance and avoid re-attaching
because, you know, Android will re-create Activity **AS IT WISHES**. So I need
to create a global ref of it and save it for later use, and cache its Native
representation. This will leak memory. The other options is just to attach the
`AssetManager` onto `ApplicationContext`

## Third, Sneaky Bitmap

Unlike BlackBerry 10 which has `QImage` and iOS with `UIImage` which is able
to be accessed on C++ codes. As well as just using simple directory structure
to put their assets. Android decides to leave the programmers to decide on
their own, and use mumbo-jumbo on the assets to be put-zip-aligned on the APK
and cannot be accessed except using AssetManager which is *drum roll* a Java
object.

You go Java and they'll do the decoding job, or you stuck in C++ and do the
decoding yourself by putting more dependencies and redo whatever they have
been done. I did't want to put another dependencies and bloat my code more so
I decided to take a shot on doing it by calling Java object from JNI. When the
other platforms could be easily done using this

{% marginnote 'ios-sidenote' 'This is how you do in Objective-C' %}
```objc
UIImage *image = [UIImage imageNamed:@"demo.jpg"];  
``` 

{% marginnote 'qt-sidenote' 'This is how you do in Qt' %}
```cpp
QImage demoImage = QImage("demo.jpg");  
```

Android decided to make me do this

```cpp  
JNIEnv *env;  
JavaVM *jvm = g_Bridge->getVM();  
jvm->AttachCurrentThread(&env, NULL);

jclass BFC_Clazz = env->FindClass("android/graphics/BitmapFactory");

jmethodID BFC_decodeStreamMethod = env->GetStaticMethodID(BFC_Clazz,
"decodeStream",  
"(Ljava/io/InputStream;)Landroid/graphics/Bitmap;");

jobject amj = g_Bridge->getAssetManager();

jclass AM_Clazz = env->GetObjectClass(amj);  
jmethodID AM_openMID = env->GetMethodID(AM_Clazz, "open",  
"(Ljava/lang/String;)Ljava/io/InputStream;");

jstring fullPathStr = env->NewStringUTF(fullPath.c_str());

jobject IS_Object = env->CallObjectMethod(amj, AM_openMID, fullPathStr);  
if (env->ExceptionCheck()) {  
  LOGE("Error loading file %s from asset", fullPath.c_str());  
  env->ExceptionClear();  
  return;  
}

jobject BMP_Object = env->CallStaticObjectMethod(BFC_Clazz,  
BFC_decodeStreamMethod, IS_Object);  
```


And then after that I can do something with the bitmap object by locking using
`jnigraphics` routines. It was fun and entertaining. I thought I was kind of
masochist or something.

## Crippled C++ Runtime

Android NDK has clang as one of its toolchain options, which is good. The
problem is the fact that stlport. STLPort is a C++ STL library which has
[relatively liberal license](http://www.stlport.org/doc/license.html), but
incomplete. Android NDK also includes other STL library, made by FSF (gnustl)
which has more features. However it's licensed under GPLv3 which I rather
avoid.

Gnustl license has exception on linking it dynamically, but I will need more
code on Java side to load its library. In my opinion, it's abstraction leak.
The only thing I need was `shared_ptr` to wrap my pointers and tracking the
reference count. iOS has it, current SDK of BlackBerry 10 have it on
`tr1/shared_ptr.h`. What about Android's STLPort? It's missing, not even in
the tr1. I finally solved it by including boost's `shared_ptr`. Which involves
these step.

  1. Downloading [Boost Library](http://boost.org/)
  2. Compile Boost's `bcp`
  3. Use `bcp` to copy only submodules needed. (In my case it's `shared_ptr`)

And the additional bloat? **57722 LOC spans on 338 files**. Yeah, disk is
cheap nowadays. a mere **3.8 MB** of source code doesn't hurt. Isn't it? ;)

**EDIT:**  
The reason I went to Boost was the legal reason. I know that I can put
`System.loadLibrary("gnustl_shared")` before loading my lib. I haven't
convinced if I'm able to include that to a proprietary library like discussed
in [here](https://groups.google.com/forum/#!topic/android-ndk/OWl_orR0DRQ).

**UPDATE:**  
After I read the license term and [FAQ from the
libstdc++](http://gcc.gnu.org/onlinedocs/libstdc++/faq.html#faq.license.what)
itsself I decided to get rid of boost and tinythread. I move to
`gnustl_static` for now, and use the `pthread` api because my synchronization
is not that complicated, only one rendering thread per view, each graphics
service and state are bound to that thread, not shared with othre threads.

## Clunky Emulator

I, by no means, having slow computers. I got a powerful i7 Linux based
computer with 8 GB RAM. But when I start Android Emulator. It IS SLOW, like
really, really, SLOW. My heart broken when I started the emulator and waiting
for 10 minutes to be usable only to find that I cannot connect adb. However,
Thanks to folks from [genymotion](http://www.genymotion.com). You rocks! The
idea of virtualbox-backed x86 emulator is novel and useful. My library uses
OpenGL ES 2.0 heavily and it works very good. But yeah, not Google provided
emulator. It sucks.

## Useless IDE

Eclipse is good. If you're doing Java-only development it saves a lot of
typing. However, mixing SDK with NDK seemed to confuse Eclipse. It refused to
compile my native code, when the code itself is compilable using `ndk-build`.
I went back to vim and [Gradle](http://tools.android.com/tech-docs/new-build-
system/user-guide), which I think, surprisingly, works very well. I missed the
visual breakpoint that IDE has rather than text-based GDB. But I'm okay with
it. The only good thing about it is its' XML UI Editor. If only, I can pull
that out from Eclipse as stand-alone apps.

## Summary

If you mix SDK and NDK on Android, you need to be ready to face the hard
truth, sleepless nights, and pulling hair.
