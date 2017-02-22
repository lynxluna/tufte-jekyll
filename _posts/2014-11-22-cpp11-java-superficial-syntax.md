---
layout: post
title: The Superficial Look at C++11 with some nips to Java
date: 2014-11-22 01:18:27.000000000 +07:00
image: http://www.ykode.com/assets/posts/moderncpp.png
categories:
- Opinion
tags:
- c++
- java
status: publish
type: post
published: true
---

I remembered when I was opening my first book of C++: [The C++ How to
program](http://www.amazon.com/How-Program-Harvey-Paul-Deitel/dp/0131857576/ref=sr_1_1?s=books&ie=UTF8&qid=1416651342&sr=1-1&keywords=c%2B%2B+how+to+program+5th+edition)
out of necessity (I don't recommend this book anymore). It was immediately obvious that I would not
be the fan of C++ syntax and I also found out that my friends were driven away by it. Java was a new
kid on the block that time and it bravely took jab on C++ complexity and syntax. I am not fond of
Java. but I thought that its syntax is easier to comprehend than C++.

<!--more-->  

However, recent rapid changes on C++ standardisation changes my opinion on it. The syntax of C++11 and soon C++14
is nice. It's less verbose than before, even compared to Java. Here's my list on how C++11 (and 14) is better
language. I'll use with some nips to Java and also comparison with previous standard.

### C++ is more concise on constant value and mutation

When I programmed in C++ and then program in Java, I feel lost.
[Const-correctness](http://www.parashift.com/c++-faq-lite/const-correctness.html) is non-existent in Java, in C++
it's considered best practice. When you don't mutate a variable you declared it as `const`, in Java there's no such
idiom.

```c++
class Human {
private:
  std::string name;
public:  
  Human( const std::string &newName ) : name(newName) {}
  const std::string getName() const
  {
    // if you put mutating statement here, there will be syntax error
    return name;
  }
};
```

```java
class Human {
  private String name;
  
  public Human(String newName) {
    name = newName; // even constructor is a mutating statement
  }
  public getName() {
    // we still can mutate name here
    return name; 
  }

}
```
C++ is very strict on this. In a `const` method, you absolutely unable to change anything. You just able to read. This encourage safer programming. There's no side effects when you call the accessor.

### Resource Acquisition is Initialisation

C++ has very nice construct when you operating on something that need something like mutex, when you lock and then
unlock a resources. In C++, we can do it by utilising destructor. C++ will unwind the stack memory and call when it
gets out of scope, the behaviour is so predictable and deterministic, it's basically a last in first out operation.

```c++
class ScopedMutex {
public:
  ScopedMutex(const Mutex &newMutex) : mtx(m) {
    m.lock();
  }
  
  ~ScopedMutex() {
    m.unlock();
  }
private:
  Mutex mtx;
}

// usage
{
 ScopedMutex mtx(lock);
 // do something inside the scope
 
 // when getting out of scope `mtx` object is destroyed and unlock
}
```

This techniques is one form of [RAII](http://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization) that is
only possible to be done in C++ and some other languages. This is just impossible to do in Java, due to the nature
of Java Runtime.

### Used correctly, operator overloading is not too bad

When you have object that is a mathematical expression (such as a vector or matrix), it's more natural if you're using normal operator rather than calling a method.

```c++
class Vector2D
{
public:
  float x, y;
  
  // constructor
  Vector2D(const float nx, const float ny) :
    x(nx), y(ny) {}
  
  // copy constructor
  Vector2D(const Vector2D &other) : x(other.x), y(other.y) {}
  
  // = operator overloading
  Vector2D &operator=( const Vector2D &other ) {
     x = other.x; y = other.y;
     return *this;
  }
};

const Vector2D operator+( const Vector2D &lhs, const Vector2D &rhs ) {
  return Vector2D(lhs.x + rhs.x, lhs.y + rhs.y);
}

const Vector2D operator-( const Vector2D &lhs, const Vector2D &rhs ) {
  return Vector2D(lhs.x - rhs.x, lhs.y - rhs.y);
}
```

The usage is pretty simple and straightforward

```c++
// usage
const auto a = Vector2D(3, 4);
auto b = Vector2D b(6, 7);
const auto c = a + b; // use `+` and then assign it to c using copy constructor
b = a - c; // change b to be value of vector a - b
const auto all = a + b + c - (c - b); // some abritary operation
```

The caveat is that don't overdo it. Overload an operator when it makes sense. With `auto`, the type is inferred by
the compiler (more on this later). In Java tough, you need to define the `add()` method inside a class and call it.

```java
class Vector2D {
  public float x;
  public float y;
  
  public Vector2D( float nx, float ny ) { x = nx; y = ny; }
  
  public Vector2D add( Vector2D rhs ) {
    return new Vector2D( x + rhs.x, y + rhs.y);
  }
  
  public Vector2D subtract( Vector2D rhs ) {
    return new Vector2D( x - rhs.x, y - rhs.y );
  }
}


Vector2D va = new Vector2D(3,4);
Vector2D vb = new Vector2D(6,7);
Vector2D vc = va.add(vb);
vb = vc.subtract(va);
Vector2D vd = va.add(vb.add(vc)).subtract(vc.subtract(vb)); // OMG!
```
Comparing the code, I'd argue that C++ code is far more readable than the Java counterparts, and more verbose too.

### `auto` do wonders

Static typing should not be verbose. In C++11 we can use `auto` keyword to determine the type of a variable.
There's also custom literal suffix to determine the type of a ambiguous literal. In C++14 there's even an
`std::string` suffix. Gone are the days of gruelsome typing of iterator types.

```c++
auto a = 1; // a is integer
auto b = 20u; // b is unsigned integer
auto c = 1000ll; // c is a `long long` type

// c++14
auto s = "SomeString"s;

// iterator
std::vector<std::string> strs {"Didiet", "Noor"};
auto is = std::begin(strs);
```

### Range based `for` is magic

Finally, in C++ we're able to do range based `for`. This replaces for-iterator idiom that is commonly used before C++11

```c++
std::vector<const std::string> strs {"Didiet", "Noor"};

for( auto str : strs ) {
  std::cout << str << "\n";
}
```

### More explicit overriding using `override` and `final`

C++11 introduces `override` to mark a method is an override of the base class' method. Thus curing the plague of old C++, which you accidentaly not overriding a method but overloading it due to different method signature (e.g. const vs non-const).

```c++
class B 
{
public:
   virtual void f(short) {std::cout << "B::f" << std::endl;}
};

class D : public B
{
public:
   virtual void f(int) override {std::cout << "D::f" << std::endl;}
};
```

C++11 also introduces `final` to mark a method cannot be overriden in the derived classes.

```c++
class B 
{
public:
   virtual void f(int) {std::cout << "B::f" << std::endl;}
};

class D : public B
{
public:
   virtual void f(int) override final {std::cout << "D::f" << std::endl;}
};

class F : public D
{
public:
   virtual void f(int) override {std::cout << "F::f" << std::endl;} // compilation error
};
```

In the sample code, class `F` cannot override the `f()` method because it was marked `final` by `D`. This is one of the keyword I wish to be borrowed from Java, and finally standardised in C++11.

### Pointers? Smart pointers is your friend.

Have you irritated by naked pointers in C++ programs? In C++11 the usage of smart pointers is very encouraged. Naked pointer is still usable in many cases, but you should use smart pointers whenever possible.

```c++
auto humanU = std::unique_ptr<Human>(new Human("Didiet", 29)); // unique pointer
auto humanS = std::make_shared<Human>(new Human("Ryo", 35));   // shared pointer
auto humanW = std::weak_ptr<Human>(humanS);                    // weak pointer
```

There are three common smart pointers to be used in C++. 


A **unique pointer** is a used when an ownership of a memory resources is not shared, but can be
transferred to other unique pointer.

{% marginnote 'smart-ptr' 'These pointers has been around since C++0x, new usage of pointers in
C++11 and up should use this smart pointer semantics instead of a naked pointer' %}

A **shared pointer** is a reference-counted pointer, it's more or less like in Objective-C. Shared
pointers should be used if the ownership of a memory resources is shared.

A **weak pointer** only able to be constructed from a shared pointer. This is used to break reference cycle. For
example on parent-child relationship, when a parent needs to reference the children and the children need to
reference their parents. If the children also own the parent, the object will never be released. I prefer C++ way
of doing smart pointer than Objective C. In objective C you need to mark a weak pointer with `__weak` which is a
syntax distraction.

## Summary

{% marginfigure 'mf-cpp-sank' 'http://img.youtube.com/vi/ltCgzYcpFUI/0.jpg' 
   '<a href="http://www.youtube.com/watch?v=ltCgzYcpFUI">Why C++ Sails But Vasa Sank</a> by Scott Meyers ' %}

C++ is a powerful language, and C++11 adds more strength and reduce syntax noise. There are tons of features in the
new standard in C++14 that makes it's more pleasant to code C++ than ever before, even if it's compared to Java. If you
want the full list you can read an [excellent article by Herb
Stutter](http://herbsutter.com/elements-of-modern-c-style/) and an excellent presentation on how C++ still alive
[on Youtube](https://www.youtube.com/watch?v=ltCgzYcpFUI).


