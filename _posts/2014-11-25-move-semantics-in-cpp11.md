---
layout: post
title: The Beauty of Move Semantics in C++11
date: 2014-11-25 00:57:27.000000000 +07:00
image: http://www.ykode.com/assets/posts/moderncpp.png
categories: programming
---

In the olden days of C++98, we were introduced to copy constructor. Copy constructor is a special kind of
constructor in C++ that is executed on initialisation, on being passed to function arguments by value, and on
return value. Copy constructor has served well for many cases. However, this poses problem on rvalue objects. They
are short-lived objects which will be copied several times. If you create a class holding a big memory blob, this
is very inefficient. In C++11, move semantics makes this operation very efficient and almost free. 

<!--more-->

Consider a class to manage an RGBA bitmap pixels. It holds a pointer to the pixel data.

```c++
class Bitmap {
private:
  int  mWidth;
  int  mHeight;
  uint8_t *mPixels;
  static const auto bpp = 4;

public:
  Bitmap( int width, int height );
  ~Bitmap();
 
  int width() const;
  int height() const;
  
  auto pixels() const -> const decltype(mPixels);
  
};

Bitmap::Bitmap( int width, int height ): 
  mWidth(width), mHeight(height),
  mPixels(new uint8_t[width * height * bpp]) {}

Bitmap::~Bitmap() { delete [] mPixels; }

int Bitmap::width() const { return mWidth; }

int Bitmap::height() const { return mHeight; }

auto Bitmap::pixels() const -> const decltype(mPixels) { return mPixels; }
```

You see weird declaration on `pixels()` accessor. It's a new way of declaring method and function or method in C++11.
We can put the return value in the right side. This return value will be valid within the scope of the function.
For example, in the example above, the implementation just use `decltype(mPixels)`, we don't need to write the
whole scope declaration `decltype(Bitmap::mPixels)` because the return type is already within the scope of `Bitmap`
class.

## Assignment and Initialisation

Let's try to assign one instance of `Bitmap` class to other class.

```c++
auto g = Bitmap(100, 100);
auto a = g;
```

Do you see the problem here? This code will compile. All member variables such as `mWidth`, `mHeight`, and
`mPixels` will be assigned to respective member variable in new instance by the default copy constructor. The
problem is it's not the contents that is copied; It's the pointer of `mPixels`. When `a` is out of scope, `mPixels`
will be deleted, but when `g` out of scope, `mPixels` pointer of `g` will be dangling and `delete` operation on the
destructor will fail, as it points to a block of memory that already freed before. We face a nasty **double
delete** problem. 

## Handy Copy Constructor

On a class holding big memory blob, and your intention is copying the contents, you should allocate a new memory
block on new instance with right size and copy the contents over. This is one of the step in the **Rule of Three**
in C++ standard to create exception-safe code

  - destructor
  - copy constructor
  - copy assignment operator

Our first rule has already been implemented so let's implement a copy constructor. The code for copy constructor
and copy assignment operator will be more or less like this.

```c++
Bitmap( const Bitmap &rhs ) : 
  mWidth(rhs.mWidth), mHeight(rhs.mHeight)
  mPixels(new uint8_t[ mWidth * mHeight * bpp]) 
{
  const size_t bmpsize = mWidth * mHeight * bpp;
  std::copy(rhs.mPixels, rhs.mPixels + bmpsize, mPixels);  
}

Bitmap& operator=(const Foo &rhs) 
{
  mWidth = rhs.width; mHeight = rhs.height;
  const size_t bmpsize = mWidth * mHeight * bpp;
  mPixels = new uint8_t[bmpsize];
  std::copy(rhs.mPixels, rhs.mPixels + bmpsize, mPixels);
  return *this;
}
```

When you do an initialisation like above, you'll get a new copy of the pixels rather than the pointer. There are
moments when the object is short-lived. On that situation, there will be multiple copies of the bitmap in one point
of time. This is prevalent on function return value and function parameters.

```c++
Bitmap doSomethingWithBitmap( Bitmap input_bitmap )
{
  // do something with input bitmap
  //.
  //.
  //.
  
  return input_bitmap;
} 

auto source_bitmap = Bitmap(250, 250);

// short lived object here, there will be multiple
// copies of Bitmap objects in one moment of a time
auto dest_bitmap = doSomethingWithBitmap(source_bitmap);
```

When bitmap is short lived there'll be multiple copies of bitmap lying around in memory during copy operation after
the bitmap returned from `doSomethingWithBitmap` call, because the result of that call is an rvalue.  This is
solved beautifully by C++11 by utilising **Move Constructor**. Before we're diving to that subject, let's talk
about lvalues and rvalues.

## The elusive lvalues and rvalues.

Let's look at these variable declarations in C++.

```c++
auto x = 9u;
auto str = std::string("Yohoo");
int *p = nullptr;
int &rx = x;
```

The `x`, `str`, `p`, and `rx` are all **lvalues**. Literals like `9u`, and string literal `"Yohoo"` is an example
of **rvalues**. Another category of rvalues is **temporaries**. When an expression is evaluated, an implementation
may create a temporary object for intermediary result. In above example `std::string("Yohoo")` may create a
temporary. Temporary values will expire once the expression is fully evaluated. It'll be destroyed when it goes out
of scope or reaching the nearest semicolon. Therefore, these expressions contain short lived objects:

```c++
auto source_bitmap = Bitmap(200, ,8);
auto dest_bitmap = source_bitmap;
dest_bitmap = doSomethingWithBitmap(source_bitmap);
```

Other kind of rvalues are function names which is an rvalues that evaluates to function's address. Similarly,
array's name is an rvalue expression that evaluates to the address of the first element of an array.

```c++
int doSomething();
int (*doSomethingFunc)() = doSomething;
char s[2014];
char *pStr = s;
```

Rvalues are short-lived. You'd need to capture them to lvalues if you wish to access outside the context of the
expression. 

### Bindings

In C++11, you can prolong the lifetime of an rvalue by binding it to an rvalue references marked by double
ampersand `&&`. 

```c++
int && yield() {
  return 20;
}

int && rv = yield();
int x = rv;
```

On that sample `rv` is an **rvalue references** holding the reference to `20` which is an rvalue. If you attach
lvalue reference to rvalue, this will yield an error.

```c++
// Compilation Error
int & yield() {
  return 20;
}
```

### The rvalues and lvalues categories 

In C++11, unlike your mom and pop's C++, has another property to ponder on: *movability* other than *identity*
which is usually already understood. So on that example `auto num = 9u` says that `num` is an **identity** for
value `9u`. Identity can be a name, a pointer, or a reference that enables you to determine wether two objects are
similar or not.

  - **Has identity but not movable**, is a non-movable object with identity, this is classic C++ lvalues. The
    expression `*p` where `p` is the pointer to an object is an lvalue.
  - **Has both identity and movabile**, dubed **xvalues** which is an object that "near death" in a sense of
    lifetime. An **xvalue** is a certain kind of expression involving rvalue references such as `std::move(obj)`.
  - **Has identity**, generalised lvalues, lvalues and xvalues are included in this category.
  - **Movable**, this is an **rvalue**. This includes xvalues, temporaries, and value without identity.
  - **No identity but movable**, this is called **pure rvalue** (prvalue) which is an rvalues **but not** xvalues.
    This includes literals and functions that don't return a reference.

## Introducing Move Constructor

Having enough knowledge about rvalues and lvalues, we'd step in to move constructor. Unlike copy constructor, move
constructor doesn't copy the resources (thanks Mr. Obvious). This improves the performance of all applications.
Move constructor construct is similar to copy constructor except it only moves the resources, in this example we'll
just swap current memory content `mPixels` with the content in the rvalue using rvalue reference.

```c++
Bitmap (Bitmap &&other) noexcept:
  mWidth(other.mWidth), mHeight(other.mHeight),
  mPixels(other.mPixels)
{
  other.mWidth = other.mHeight = 0;
  other.mPixels = nullptr;
}
```

This will cleanup the rvalue before destroying and take the `mPixels` data from rvalue to us. This operation is
very cheap and doesn't involve copying.  This is actually one of two additional rules added to the Rules of Three.
In C++11 we have the Rule of **FIVE**, yes five, the first additional one is the move constructor, the other is
move assignment.

```c++

// utility swap function
friend inline void swap( Bitmap &first, Bitmap &second ) {
  std::swap(first.mWidth, second.mWidth);
  std::swap(first.mHeight, second.mHeight);
  std::swap(first.mPixels, second.mPixels);
}

Bitmap & operator=( Bitmap &&other ) noexcept
{
  swap(other, *this);
  return *this;
}
```

This is a very cheap and safe technique. No copy involved, and in the end our old data will be destructed after the
rvalue `other` is gone. Neat!

Concluding the article, let's revamp the assignment operator a little bit with **copy and swap idiom**, You see
that we have assignment operator for each copy and move operation. Well, we could elide both to one method.

```c++
Bitmap &operator=(Bitmap other)
{
  swap(other, *this);
  return *this;
}
```

Yes, we remove the rvalue reference and pass it by value and now we only need this only assignment operator.

![What?](/assets/posts/Despicable-Me-what-gif.gif)

You might wonder why? In C++11, if a value that you've passed into the method is an lvalue, it will call the copy
constructor which will allocate and copy contents. However, if the value passed is an rvalue, it'll call the move
constructor.

```c++
auto b1 = Bitmap(1024, 1024);
auto b2 = Bitmap(2048, 2048);

b2 = b1; // this will copy
b1 = doSomethingWithBitmap(b2); // this will do copy-and-swap
b2 = Bitmap(100, 100); // rvalue, move assignment
```

## Summary

Move semantics make program screams. Short lived objects are managed beautifully with this idiom. It's also
exception safe. It significantly increase the performance and safety of your C++ program and it's easy to implement
too.
