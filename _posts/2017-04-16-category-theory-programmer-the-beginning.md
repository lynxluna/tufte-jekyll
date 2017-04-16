---
layout: post
title: "Category Theory: The Beginning" 
date: 2017-04-13 00:18:27.000000000 +07:00
image: "https://www.ykode.com/assets/mainimage/category-theory.png"
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
{% marginfigure 'id-eilenberg' 'assets/posts/person/eilenberg.jpg' 'Samuel Eilenberg and Mac Lane introduces
Category Theory on their paper titled "General Theory of Natural Equivalences" in 1945' %} 
When I first heard about the word {% newthought "Category Theory" %} on my journey to functional programming,
I was quick to dismiss it. I write codes for living, so I thought those are the mathematical models of
something that has no value on my day-to-day job. Any article I read about CT is laden with math symbols I did
not understand and the vocabulary is not familiar for me. I'm living in non-English speaking country, so
English math vocabulary is a thing I'm studying at my 20s, not in my teenage years. So I didn't see any value
of Category Theory on my programming. I can program just fine without any knowledge of Category Theory. I
heard and I swept it under the rug.

That changed when I realised steady stream of new languages claiming to be Functional Languages entering the
mainstream. Libraries popping out to support functional style programming. I sense something changing in the
programming world, and my foreboding become more apparent recently. Nowadays, the way people do programming is
different compared to 10 years ago. Multicore is the *norm*, running on multiple small machines vs big
mainframe is how you do big software nowadays. New techniques based on old ideas coming out. Programmers are
now constantly put in a hot water of concurrency. It's hard to make sense and prove the program correctness
nowadays. Software becomes so complex and it's easy to lose track.

<!--more-->

We're in the middle of what physicisits will say a _phase transition_. Paradigm will shift due too multicore
revolution and the cloud. Object Oriented Programming make sense in last century, but it starts to show
cracks. It does not model concurrency and parallelism. Multi-threading doesn't really solve the problem. It
introduces buggy design when used recklessly. People starts to face deadlock, livelock, starvation, and races
everyday. Not only on single machine but also on their datacenter due to separation of processing onto
multiple machines. We need something better to model and solve this problem.

It turns out, computer operations can be modeled onto mathematics formula, axioms, and laws.  Something that
hard to be made sense of, now can be proven. Mathematicians have been doing this for years, and now we can
steal their knowledge to model our computation even before we go down to hours of coding session. 

Category Theory forces us to think about the _Type_, which fortunately, can be verified by compiler. Compiler
is good on following instructions to the minute details. We're not, we tend to forget things. We invent [Test
Driven Development](https://en.wikipedia.org/wiki/Test-driven_development) to make sure our software is
correct and consistent. We invent Domain Driven Design to abstract out our code and domain from the
infrastructure nitty gritty. And now we have Category Theory to build the Domain Model and making sure of its
correctness.

## 'The' Category

{% newthought 'Category' %} is a embarrasingly simplistic yet powerful concept. It has two things: {%
newthought 'Objects'%} and {% newthought 'Morphism' %}. A Categories must follows {% newthought 'Composition
Law' %} and obeying some _Properties_. 

### Objects

Let's start with Objects. In this article a Category is defined by cursive script. Let's say we have category
{% m %} \mathscr C {% em %}, The objects of Category {% m %} \mathscr C {% em %} are expressed with {% m %}
Ob(\mathscr C) {% em %} and can be represented below:

{% maincolumn 'assets/posts/ct/ct-objects.png' 'Representation of objects within Category in Category Theory' %}

From diagram above we do know that {% m %}A{% em %}, {% m %}B{% em %}, {% m %}C{% em %}, {% m %}D{% em %}, and
{% m %}E{% em %} are {% m %}Ob(\mathscr C){% em %}. We don't know what are them. We also don't know their
properties. We just see it as black box. We don't focus too deeply on that.

Objects in Category Theory loosely corresponds to {% newthought 'types' %} in programming. As a programmer,
it's easier to think object as types. It can be anything, `Integer`, `Boolean`, `Order`, `GameObject`,
anything. In fact, by thinking like this, we're jumping into Category of Programming.

### Morphism

{% newthought 'Morphism' %} is relation between objects in a category. Morphism also called {% newthought
'Arrow' %}. I use it interchangeably.

{% maincolumn 'assets/posts/ct/ct-definition.png' 'Morphisms between objects in category C. Each object has
relation with other objects' %}

If {% m %}A{% em %} and {% m %}B{% em %} are objects in {% m %}\mathscr C{% em %}, then {% m %}f: A
\rightarrow B{% em %} is a morphism from {% m %}A{% em %} to {% m %}B{% em %} and {% m %}Hom(A, B){% em %} is
collection of _all_ morphisms from {% m %}A{% em %} to {% m %}B{% em %}. {% m %}Hom(A, B){% em %} is also
called the {% newthought 'Hom-Set' %} of {% m %}A{% em %} to {% m %}B{% em %}. All objects and morphisms in
the figure above defines the category {% m %}\mathscr C{% em %}. A morphism can go from one object to another
or to itself. For example, morphism {% m %}id_A: A \rightarrow A{% em %} goes from {% m %}A{% em %} to {% m
%}A{% em %}.

In programming, morphisms corresponds to {% newthought 'functions' %}. So morphisms from object {% m %}A{% em
%} to {% m %}B{% em %} in Category Theory can be interpreted in programming as a function that takes parameter
of type {% m %}A{% em %} and returns something of type {% m %}B{% em %}.

In the study of Category Theory, we focus more into morphisms or functions. We understand objects or types
from the relations from each other and composition of those relations.

### Composition

To form a category there must be a composition between morphisms. Composition denoted by small circle {% m
%}\circ{% em %} pronounced "after". If {% m %}f{% em %} and {% m %}g{% em %} are morphisms which {% m %}f: A
\rightarrow B{% em %} and {% m %}g: B \rightarrow C {% em %}. There must be {% m %} g \circ f: A \rightarrow
C{% em %}.

{% maincolumn 'assets/posts/ct/ct-composition.png' "Composition between morphism <math><mrow><mi>f</mi></mrow></math> 
and <math><mrow><mi>g</mi></mrow></math> within category 𝒞" %}

In programming, composition is the same as applying one function to another. See code below written in
Java.

```kotlin

fun <A, B> f(a: A): B { /* do something to A and returns B */ }
fun <B, C> g(b: B): C { /* do something to B and returns C */ }

// compose the function
fun <A, C> gAfterf: (a: A): C = g(f(a)) 

```

We'll go back to composition later on. For now let's move on to the _Identity_ properties.

## Category Laws

Inside category, there are laws to be satisfied: the {% newthought 'Identity Law' %} and the {% newthought
'Associativity Law' %}.

### Identity Law

Identity is a morphism form an object to itself which: For each object {% m %}X{% em %} in {% m %}\mathscr C{%
em  %}, there's an identity {% m %}f: X \rightarrow X {% em %} so that for each {% m %}f: A \rightarrow B{% em
%}

{% math %}id_B \circ f = f = f \circ id_A{% endmath %}

This can be expressed in picture:

{% maincolumn 'assets/posts/ct/ct-identity.png' 'Identity property of morphism in Category' %}.

In programming, a morphism is a function that returns the same value of the same type. Written in Kotlin:

```kotlin
fun <A> identity(a: A) = a
```
It seems to be useless. However, as Category Theory does not concern much about objects, but its morphisms.
This identity morphism is essential for further study of Category Theory. Think identity morphism like number
zero. It was made to express nothingness. Identity morphism express _no operation_.

### Associativity Law

Composition is associative. If we have morphism of {% m %}f{% em %}, {% m %}g{% em %}, {% m %}h{% em %} that
can be composed. Meaning their objects match end-to-end, this associativity law holds:

{% math %}h \circ g \circ f = (h \circ g) \circ f = h \circ (g \circ f){% endmath %}

Therefore, we don't need to put parenthesis when expressing function composition. If you like pretty pictures,
the law can be represented by this:

{% maincolumn 'assets/posts/ct/ct-assoc.png' 'Associativity law should hold between objects within category.' %}

In programming, associativity holds for [functions](https://en.wikipedia.org/wiki/Pure_function). But when
we're dealing with other categories such as Set, it maybe less obvious.

## Composition and Programming

{% marginfigure 'id-miller' 'assets/posts/person/George_A_Miller.jpg' 'George A. Miller is American
psychologist who had written coincidence between limit of short-term memory and one-dimensional judgement
task.' %} When we code, we chop up our big problem into smaller problems and then to smaller units. In the
context of domain driven design, we divide our program into domains. Each domains can consists of several
services. Each services consists of many functions and every function consists of expressions. We compose
these smaller parts to build the bigger parts. We do this because of limitation of our brain. A psychologist,
George A. Miller has even said on his paper about this phenomena.

A beautiful or elegan code is a chunk of code that can fit and digested easily by our brain. A good size of
composable code has, to quote from [Barotz Milewski](https://bartoszmilewski.com): "Their surface area has to
increase slower than their volume". The _surface area_ is the information we need to _compose_ code, the
_volume_ is information we need to _implement_ them.

One thing that struck me is that Category Theory doesn't want us to peek inside the object. An object is black
box, a 'something'. We only know their properties by examining the relation between them. In Object-Oriented
Programming this means that we model our program based on their interface (just surface area, no volume) with
their methods is the morphism. The litmus test for non-composable program can be summarised to one sentence,
this also paraphrased from Barotz Milewski:

> The moment you have to peek and dive into implementation details to understand how to compose it with other
> object. You've lost advantage of your programming paradigm.

We access and services by their interfaces like REST API right? You should not even know how they work. All
you know you want to compose your system or app with theirs. All you want to know is how to call them.

# Summary

This article is the discovery I made when I study Category Theory on my free time. It helps me decompose
problem into composable smaller chunk and write less code to achieve same result. Hope this article can wet
your appetite on studying and utilising Category Theory when you create code.

