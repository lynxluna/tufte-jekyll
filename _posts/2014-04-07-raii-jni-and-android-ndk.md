---
layout: post
title: RAII, JNI, and Android NDK
date: 2014-04-07 14:04:19.000000000 +07:00
categories:
- Mobile
tags:
- c
- java
- jni
- raii
status: publish
type: post
published: true
meta:
  _publicize_pending: '1'
  publicize_facebook_url: https://facebook.com/194032247284171_738298992857491
  geo_latitude: "-6.885114"
  geo_longitude: '107.611825'
  geo_accuracy: '65'
  geo_address: Jalan Siliwangi, Coblong, Bandung, Jawa Barat 40132, Indonesia
  geo_public: '1'
  _wpas_done_5833442: '1'
  _publicize_done_external: a:1:{s:8:"facebook";a:1:{i:1072530386;b:1;}}
  publicize_google_plus_url: https://plus.google.com/112674531673783811161/posts/i1xPVq2ESZ9
  _wpas_done_5833439: '1'
  publicize_twitter_user: ykode
  publicize_twitter_url: http://t.co/b34hTo9qON
  _wpas_done_3612350: '1'
  publicize_linkedin_url: http://www.linkedin.com/updates?discuss=&scope=55847635&stype=M&topic=5858831475595558912&type=U&a=wTsL
  _wpas_done_3613001: '1'
  publicize_path_id: 53424e04ca7261c3a7b4b7df
  _wpas_done_5897407: '1'
---

If you have been doing multi-threaded C++ code, you'll know that [RAII
(Resource Acquisition is Initialisation)](http://en.wikipedia.org/wiki/Resourc
e_Acquisition_Is_Initialization) idiom is very valuable to ease the burden of
manually typing resource initialisation and release. You might have applied
that for mutex, threads, and other kind of locks. But how about JNI? I'll show
you in this post.

<!--more-->

When you're using JNI and multithreading, once in a while you'll stumble upon
`AttachCurrentThread` or `GetEnv` method to obtain the `JNIEnv` environment
and able to access the JVM. I usually cache the `JavaVM` instance within
`JNI_OnLoad()` function, so other than catching the `env` variable via JNI
function signature, from time to time I just want to get the env variable for
current thread by using `GetEnv` like this:

```cpp
JNIEnv *env;

// gVM is the cached global JavaVM pointer  
gVM->GetEnv((void**) &env, JNI_VERSION_1_6);

```


However, this function only able to get the env information if you called it
from the same thread where you started the JNI. If you create app with more
than one thread, you need to call `AttachCurrentThread` if your thread is not
the same as the thread where you call the JNI function. This thread is called
**Detached Thread**

```cpp
JNIEnv *env;

// attach current thread and get the env  
gVM->AttachCurrentThread(env, 0);

// ... code ...

gVM->DetachCurrentThread();  
```


As you can see, we need to call `DetachCurrentThread`. If we don't do this,
the JVM wil complain if the thread exits without detaching. Because one of my
module in [Compo](http://www.compofx.com/) can run in either main thread or
worker thread, I decided to do detection. Luckily, `GetEnv()` returns value of
`JNI_EDETACHED` if called from thread that is not attached yet.

I think it's a perfect case to do RAII on the JNI environment. Therefore, I
create a simple class to detect and attach the thread if it's not attached,
and detach the thread automatically.

```cpp
#include <jni.h>  
template <const unsigned int jni_version=JNI_VERSION_1_6>  
class ScopedJNI  
public:  
  inline ScopedJNI(JavaVM *vm, JNIEnv **env) : mVM(vm),  
    mStat(mVM->GetEnv((void**) env, jni_version)  
  {  
    // Detect if initial value is detached  
    if (JNI_EDETACHED == mStat) {  
      mVM->AttachCurrentThread(env, NULL);  
    }  
  }  
  inline ~ScopedJNI() {  
    if (JNI_EDETACHED == mStat) {  
      mVM->DetachCurrentThread();  
    }  
  }  
private:  
  // no copy constructor and default constructor  
  ScopedJNI();  
  ScopedJNI(const CompoScopedJNI &);

private:  
  JavaVM *mVM;  
  const int mStat;  
};  
```

This way we can use the code like this

```cpp
{  
// start a scope for JNI calls  
JNIEnv *env;  
ScopedJNI startJNI(gVM, &env);

// ** do JNI CALLS *//  
jclass str_Clazz = env->FindClass("java/lang/String");  
}  
// the env and startJNI is automatically released and destroyed  
```
Hope this helps!
