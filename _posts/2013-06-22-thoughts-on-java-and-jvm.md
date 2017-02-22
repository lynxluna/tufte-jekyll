---
layout: post
title: Thoughts on Java and JVM
date: 2013-06-22 01:18:27.000000000 +07:00
categories: opinion
---

{% newthought 'I have very long relationship with Java' %}. I came in contact with Java at my college
study. I was shopping for new language besides C and C++ that I used it everyday. I use it to create
[ActiveX Components](https://en.wikipedia.org/wiki/ActiveX) to be sold and consumed by Visual
~~ActiveX Editor~~ Basic, things like unzipping, reading/writing binary files, let alone displaying
3D object.  I got tired with C++ and decided on trying other language. And the first I encounter was
Java.

<!--more-->

Java was introduced by my _mentor_ as a _better C++ with garbage collector._ I thought: "Great, no
nasty memory management!". And its lauded slogan "_write once run anywhere"  _made me think "O
great, I can program my refrigerator (if it's powered by Java)". I started to try and follow some
tutorials and courses about this once exciting language.

The familiarity of the syntax enticed me to try the language. However, the more I used it the more I
felt burden of using it. Let me explain..

{% newthought 'First, It was slow.' %} My computer was my trusty [Pentium 166
MMX](http://www.cpu-world.com/CPUs/Pentium/Intel-Pentium%20MMX%20166%20-%20A80503166.html) with
whopping 32 MB of RAM and 2 GB hard drive with Sound Blaster sound card. Not a recent spec for 2003,
however I ran Visual C++ 6 just fine and made decent income to pay my tuition. Java was really a
burden back then. my computer ran as slow as a fat stuffed snail. Beside that I feel _cramped_ in
Java. 2 minutes startup times just to load a 100-LOC file? _Noupe._

**Second, It's not as powerful. **The freedom of accessing memory and know exactly when and where
did your objects are is lost. The syntax is not better than C++ (oh that curly braces and
semicolon), it's less powerful, and ate all my computer's resource, before even me made a single
cent of out it. Oh, and don't say _write once, run **anywhere**_, it's not. My C code runs on my
small Microcontroller to an IBM mainframes. Java does not.

**Third, No improvement on syntax. **C++ syntax is ugly enough. Java tries to _fix it_ by removing
many operators and functionality, and introduced garbage collection. Shortly I know that actually I
was not searching for a way to avoid memory allocation and deallocation but an elegant and readable
language than C++. If it is less powerful, make it pretty to the eyes, I said. And Java as a
language could not deliver that.

**Fourth, Sun's (now Oracle) way of marketing. **They demonized C and C++ to promote Java. You might
laugh now but Java was the one that bashed C and C++ in term of _security_ (Hi Java7! ;)). It
offered silver bullet to avoid stack overflow, memory corruption, and memory leak. That time, Java
adherents were using the same song to promote Java. I am using C++ in day to day basis. I rarely
used `new` and `delete` manually, there are smart pointers you can use, reference counting, and
parent-child relationship like those available on Qt avoid shooting yourself on the foot. And
moreover, seasoned C++ developers usually have no problem with memory management.

**Fifth, it's mobile ecosystem**. Remember Java ME? It was once the king. Most of the phone brands
ran it. BlackBerry OS, Nokia, Sony, name it. But Java kingship in mobile doesn't last long.
Actually, it's only one smartphone platform that still use Java nowadays, **only** Android and it
doesn't use Sun/Oracle Java ME. BlackBerry was the one who still use Java ME with BlackBerry
extension, but they're moved on to BlackBerry 10 which doesn't use Java ME. And guess what? C and
C++ are _still_ the **most portable** language on the planet even if Sun demonized it in great
extent. Almost all smartphone ecosystems nowadays support C and C++. 

However Java has one wonderful thing: The JVM. The JVM is wonderful. It's mature and battle tested,
if you want to create your next-gen software, JVM based languages are your best bet.
