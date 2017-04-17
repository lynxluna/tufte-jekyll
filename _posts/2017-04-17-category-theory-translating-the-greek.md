---
layout: post
title: "Category Theory: Translating The Greek" 
date: 2017-04-17 00:18:27.000000000 +07:00
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

In the spirit of making Category Theory more palatable, the people who talks on the lecture I follow are
mostly professor with programming career. So they walk the talk. One particular person, {% newthought 'Erick
Meijer' %}, stands out. He is a Dutch Computer Scientist who've worked for Microsoft on the .NET. And not
surprisingly, he's the author/contributor of the Reactive Extension (Rx) for .NET and Java. A library that I
love so much, I'm using it everywhere.

The good thing about Erick is that he _"translate the Greek"_ by mapping math jargons in the Category Theory
to something we can relate to: Programming. I'll summarise them in this article.

<!--more-->

## The Definition

Let's start with the easiest part of the Category Theory: The Definition. It's so simple, yet it has already
greek on it: _morphism_. For the sake of easier understanding, Erick has translated the greek for us:

{% marginnote 'table-def-3' 'The translation table between the math and the programming terms' %}

| **The Greekspeak** | **The Geekspeak**  |
|:---------------|---------------:|
| Category       | Programming Language or Domain |
| Object         | Types          |
| Morphism       | Static Method / Function |

So whenever those terms pop out. We can substitute the word to the word that we know. So let's look at the
diagram below:

{% maincolumn 'assets/posts/ct/ct-definition.png' 'The category theory definition' %}

This can be read like this:

> A programing language that has 5 types and there are static methods and functions taking one types and
> returns another.

## The Products Category

The {% newthought 'Product' %} in Category Theory is the _compound data type_ in the programming sense. One of
such product is _tuple_. So the model of tuple in category theory is as follows:

{% maincolumn 'assets/posts/ct/ct-product.png' 'Product in Category Theory' %}

So this is the definition from the Greekspeak:

> Let {% m %}\mathscr C{% em %} the category with some objects {% m %}A{% em %} and {% m %}B{% em %}.
