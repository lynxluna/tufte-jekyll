---
layout: post
title: "Synchronised Asynchronous Operation: My Journey to Category Theory" 
date: 2017-04-13 00:18:27.000000000 +07:00
image: https://www.ykode.com/assets/posts/coffee-lambda.jpg
categories:
- Programming
- Category Theory
tags:
- category theory
- functional programming
status: draft
type: post
published: true
---

In the last few days I've been dabbled in the world of abstract algebra with the goal of understanding and
make sense the domain that I've been doing at office. In the world of distributed system, parallelism and
asynchrony is inevitable.  In the server, we have things like NodeJS or Vert.x. All client app, like web,
native mobile, are all built on top of the common theme of event loop and asynchronous event handling.

It does not take me long time to recognise that it happens everywhere. When I open the home screen of my app
these things happen: 

> The app will build the state of the home screen by asynchronously fetching bits of data from product catalogue
> to get the product information. It also need something from product recommendation to decide where to put that
> information on screen.  There are also banners from marketing services. To show that little number of item in
> the cart, it needs to get that data from order processing.  Not to mention all the interaction and animation. 
> Altogether build the state of the home screen. 

What will happen if the marketing service is down, or timed out? What will happen if the internet is dead, or
when it connected 3 seconds later. It is increasingly difficult to make sense how to build the home screen
with all edge cases included. The code in `MainActivity` class becomes mishmash of state management and
transition and hard to reason about without running it.

<!--more--> 

## Callbacks and their hell

Pattern like callbacks in Javascript starts to emerge which basically states:

> For every operation in which the result can be a success or failure, provide an anonymous or named function
> for both success and failure.

Platforms like Web Browser, NodeJS, or even Vert.x prohibit long computation and blocking thread. I/O will
take sometime to finish, so rather than blocking the whole event loop, we submit operations and give success
and failure callbacks when it happens. Consider this pseudocode

```javascript
function fetchFromDatabase(connection, success, failure) {
  connection.doQuery("SELECT FROM Order WHERE id= ?", 
    113243, success, failure);
}
```

So far so good, but how about after getting successful result, we want to also publish to the event queue and
consider successful if all of the operation succeeded.

```javascript
function fetchFromDatabaseAndSendToQueue(connection, queue, success, failure) {
  connection.doquery("SELECT FROM Order WHERE id =?", 
    113243, 
    function(result) {
      queue.send(result, success, failure)
    }, 
    failure);
}
``` 

We see that on every synchronous operation, We need to go one level deeper. The more operations we want,
the deeper the nesting and indentation is. This is known affectionally by programmers as the **callback
hell**.

## One Step to the Promise-land

To fix this problem ECMAScript 6 introduces `Promise`. `Promise` is a structure to express an asynchronous
operation. It says that it will solve the callback hell.

```javascript
let timeoutPromise = new Promise((resolve, reject) => {
  setTimeout(() => resolve("success"), 250);
});

timeOutPromise.then((msg) => console.log(msg));
```

Seems promising, the arrow function syntax on ES6 helps so much in cleaning up the syntax. However because
`Promise` is modelled after _operation_, it's still hard to compose them. Say we want some synchronous and
asynchronous operation as follows:

> Let get files from directory, open and process them, and then write its result.

```javascript
read_dir('/home/Documets')
  .then(files => Promise.all(files.map(file => {
    return read_file(file)
      .then(data => write_file(data.name, convert(data.data)))
  })))
  .then(() => console.log("success"), 
         e => console.log("error", e))
```

If we want to do writing after we succeeded listing all the files, then you'll have nested function. The more
imperative the domain is the Promise will be in deep `then` structure. For example: if we try to retry on
error, we'd need to put our retry on the reject part of the `then`. The same thing if we want to transform the
result of the promise and send them to different operation. There's no escape from hell. We just move from one
kind of hell to another.

## Ascension to Functional Nirvana

Let's digress a little bit to a structure that we know and love: array. In ECMAScript 6 we can do some basic
operations operation on them:

```javascript
  const numbers = [1, 2, 4, 88, 97, 22, 11];

  // this will create new list 
  // of square of all elements
  const squared = numbers.map( x => x * x );
  
  // this will create new list with odd numbers
  const odd     = numbers.filter( x => x % 2 == 1);

  // this will sum the whole array to one number
  const sum     = numbers.reduce( (a, x) => a + x, 0);
```
The method of `Array.prototype.map()`, `Array.prototype.filter()`, and `Array.prototype.reduce()` create a new
Array:
  
  - {% newthought "Map" %} creates a new array with each element the function {% m %}f(x) = x^2{% em %} denoted by 
    closure of `x => x * x` applied.
  - {% newthought "Filter" %} creates a new array with only elements that matches the predicate {% m %}x
    \mathbin{mod} 2 = 1 {% em %} denoted by closure of `x => x % 2 == 1`;
  - {% newthought "Reduce" %} apply sum function of {% m %}f(a, x) = a + x{% em %} with {% m %}a{% em %} is
    the accumulator and {% m %}x{% em %} is the current value. This will reduce the the whole list to one,
    single value. This operation also known as {% newthought "Folding" %}.

This way we can combine and compose their operation.

```javascript
  // this will sum the squared odd numbers
  const sumoddsq = numbers
    .filter( x => x % 2 == 1 )
    .map(x => x * x)
    .reduce( (a, x) => a + x, 0);
```

Which combine map, filter, and reduce to one very nice fluent{% sidenote 'fluent-note' 'Fluent method means
that readibility of source code is close to the written prose, in this case it can be read as Given an array
of numbers, let&#39;s filter the odd, square all of them, and then sum the whole thing'%} method. Most
operations to collection are mapping, filtering, and reducing of some sort. They remove the needs of having
loops on your API, make the code more readable and best of all they are pure function.

If we're going to have a free function of `map`, `filter`, and `reduce` instead of prototypical ones, the
operation can be translated as follows.

```javascript
reduce(
  map(
    filter(numbers, x => x % 2 == 1), 
  x => x * x), 
(a, x) => a + x, 0)
```
It's a function composition! I'm going to wear my math hat, ignoring the parameters other than `numbers` and
if filter is {% m %}f{% em %}, map is {% m %}m{% em %}, and reduce is {% m %}r{% em %} the composition is
similar this expression.

{% math %}(r \circ m \circ f) ([x]){% endmath %}

If we draw nice diagram to build intuition for the next attempt, it will be like this:

{% maincolumn 'assets/posts/MapFilterReduce.png' 'The map, filter, and reduce as operation of types'
%}

We transform an `Array of Int` to `Array of Int`, and then `Array of Int` to `Array of Int`, and finally
`Array of Int` to single value of `Int`. In other word, we transform values (Int) inside another type (Array)
to new values inside another type and finally turn that value out from that type.

## Making sense of the operation attempt

{% marginfigure 'monad' 'assets/posts/monad-x.png' 'A diagram showing operation that will
yield a success value or a failure' %}
Back to our original computation modeled in both callbacks and `Promise`s. It seems to me that they have a
common pattern: 

> A _computation_ that takes _an input_ and will _later_ returns a _success_ with some value or _failure_ with
> an _exception_.

Let's say we want to chain said operation by only picking up the success value. For example an operation of
fetching User Id from an HTTP endpoint and then use that information to get the order of that user. We can
draw the operation roughly as follows.

{% maincolumn 'assets/posts/Monad-Mon.png' 'Composition of two operations. The `UserLogin` operation yield
`User` object where it is used as input for `GetOrder` operation to yield the `Order` object' %}Thinking like
this, we suddenly find that our operation is actually composable. Ignoring the errors and operations, what we
did was converting one type to another. This is the basic of {% newthought 'Map' %} that we have learnt
before. 

If we imagine them to be values without operation, we basically compose two map operations, from `UserName` to
`User` and from `User` to `Order`. The mapping function goes from `UserName` to `User` and then to `Order`.

{% math %}
\begin{align}
F&:UserName \Rightarrow User \\
G&:UserName \Rightarrow Order \\
G \circ F&: UserName \Rightarrow User \Rightarrow Order
\end{align}
{% endmath %}

However, because they are _operations_ and not a simple value mapping, the {% m %}\circ{% em %} compose operator is
not enough. Lets write the math

{% math %}
\begin{align}
F&:UserName\Rightarrow UserLogin[User] \\
G&:User\Rightarrow OrderRequest[Order]
\end{align}
{% endmath %}

We have problem here. How we create a composition to unwrap {% m %}User{% em %} from {% m %}UserLogin{% em %}
so our operation compose? If don't do that, we're stuck with those two operations and to do the operation in
serial, we need to shove operation {% m %}G{% em %} inside the {% m %}UserLogin{% em %} type. Let's Shove it!

{% math %}UserName \Rightarrow UserLogin[User \Rightarrow OrderRequest[Order]] {% endmath %}

The notation is actually not correct, but for the sake of building intuition, let's just let it be. I just
want to model the callback. It's actually shoving the next operation inside the callback of previous
operation and this does not compose well. If we need new operation, we need to shove it inside {% m
%}OrderRequest{% em %} and so on. What was that? Yes, It's the callback/promise hell. We're going back to
square one. Why can't we _compose_ operations as easy as transforming list?

But worry no more, let's solve this by defining operator that will unwrap the value from previous operation
and feed that the next operator and let's name it _flatMap_ with this symbol {% m %}\rightarrowtail{% em %}
for our made-up operation. Suddenly, our operation compose!

{% math %}
\begin{align}
F&:UserName\Rightarrow UserLogin[User] \\
G&:User\Rightarrow OrderRequest[Order] \\
F \rightarrowtail G&: (UserName \Rightarrow UserLogin[User]) \rightarrowtail (User\Rightarrow OrderRequest[Order])
\end{align}
{% endmath %}

This composable model does not exist in Javascript but exists in Java 8 with `CompletableFuture` and Rx with
`Observable`. If you ever use those library, you know how easy it is to compose operations.
`CompletableFuture` has `thenCompose` method, and Rx has `flatMap` method.

```java
  // this returns CompletableFuture
  userRepository.findByName(userName)
    .thenComposeAsync(user -> orderRepo.findByUser(user))
    .thenRunAsync(order -> log.debug("Order fetched: " + order.id);

  // this returns Observable from RxJava
  rxUserRepository.findByName(userName).
    .flatMap(user -> rxOrderRepo.findByUser(user))
    .subscribe(order -> log.debug("Order fetched: " + order.id);
```

Here, `findByName` and `findByUser` returns `CompletableFuture` or `Observable` depends on the library used.
The code is nicely composable and easy to read.

## Back To Category Theory (ish)

So we're here, discovering pattern as we went through our journey of writing more composable and readable
code. So far we have recognised two patterns. First, applying function on top of value inside some context (in
this case array) and second, composing operations by taking out the value inside some context and (in our case
HTTP/Database call) and then use that to do another independent operation. These patterns happens a lot of
time.

This is where I start to search for "higher level language" to use those patterns on my app. A language that
can be used as modelling language for my problem domain. I've heard Category Theory and Lambda Calculus for
sometime, but I avoid it. Turns out, they are what exactly I need to model my problem. I started to follow and
watch Erik Meijer's excellent talks like this one:

<iframe width="560" height="315" src="https://www.youtube.com/embed/JMP6gI5mLHc" frameborder="0" allowfullscreen></iframe>

It turns out collection mapping and composable operation is two concepts in Category Theory called {%
newthought 'Functor' %} and {% newthought 'Monad' %}. Those two are so common in programs, that I don't even
think about their name. In essence, I'm discovering the 'Category of Programming' bottom-up by recognizing and
deducing patterns.

### Category

So let go into Category Theory. By definition a {% newthought 'Category' %} is a simple collections with
three components: a collection of {% newthought 'objects' %}, a collection of {% newthought 'morphism' %}, a
notion of compositon of said morphism, and an identity.

{% newthought 'Morphism' %} ties two objects together. Sometimes, it also called {% newthought 'arrow' %}.
Given a source object {% m %}X{% em %} and target object {% m %}Y{% em %}, a morphism {% m %}f{% em %} defined
as {% m %}f: X \rightarrow Y{% em %}.

{% marginfigure 'id-commut' 'assets/posts/CT_Commut_Diag.png' 'Category Theory as collection of objects,
morphism, identity, and composition of morphism with its laws' %}With collection of objects and morphisms,
there are laws regarding category. For each Category {% m %}C{% em %}:

A {% newthought 'Composition' %} between morphism of {% m %}f:X \rightarrow Y{% em %} and {% m %}g: Y
\rightarrow Z{% em %}, there must be a morphism denoted by {% m %}g \circ f: X \rightarrow Z{% em %}.

For every object there's a morphism {% m %}id{% em %} such that {% m %}id_X{% em %} is the {% newthought
'Identity' %} of object {% m %}Y{% em %} so that {% m %}id_Y \circ f = f {% em %}.

The composition of morphisms is {% newthought 'Associative' %} so that {% m %}(h \circ g) \circ f = h \circ (g
\circ f){% em %}

For programmer, **Objects** in Category Theory can be considered the same as **Types**. Morphism is the
function between data types. This will help us to model our problem into categories within a problem domain. 

Transforming, or morphing, between types are common when we're creating our application. The example above
we transform from a HTTP Request to User DTO to User Domain Object to Screen State. So basically we're moving
from one types to another, from one domain to another. A change of price in the catalogue can affect the
order, and so on. The understanding of Category Theory may at least make us able to model and reason our
problem domain.

## Summary

I'll conclude my blog here. I hope this helps you discover the merit of having some knowledge of Category
Theory to make a better software. I actually write this article just after reading and following lectures. So,
if I made mistake, please tell me in the comments.
