---
layout: post
title: Using StringReader and transient sequence in Libil Clojure 
Date: 2014-09-09T01:28:30+0700 
categories:
- Programming
tags:
- ruby
- clojure
- java
- stringreader
- tokenization
status: draft
type: post
published: true
anchors:
- The Naive Implementation
- String Reader Discovery
- Transients
- Summary

---

Clojure never stops to amaze me. I've implemented
[libil](/2014/09/05/the-writing-of-libil.html) in Clojure too. The `0.1.0` version of clojure
implementation makes use of persistent data and head-tail idiom, as you've usually see in LISP. I think it's
already good until I read the implementation of [clojure-csv](https://github.com/davidsantiago/clojure-csv). I
discovered new and faster way to tokenise string. 

<!--more-->

<a name="!"></a>

## The Naive Implementation

The naive implementation is very straightforward. It's implemented using head-tail idiom and treat the string like
a list. I haven't use it in the production, and haven't tried to benchmark it, so I don't know the result. This is
the complete tokenisation code.

```clojure
(defn tokenize-word
  "Tokenizing the word, to be able to be mapped"
  [^String w]
  (loop [l [] rstr w]
    (let [pair (apply str (take 2 rstr))
          fstr (str (first rstr))]
      (cond (empty? rstr) l
            (within? all-con (lower-case pair)) (recur (conj l pair) (apply str (-> rstr rest rest)))
            :else (recur (conj l fstr) (apply str (rest rstr)))))))
```

I use `rest` twice to skip the list if we meet digraphs like `ny`, `ng`, `dh` and `th`. There's a subtle bug in
the implementation. Upon reading the code, I feel something not right. I don't know what it is, my gut
just says so.


<a name="2"></a>

## String Reader Discovery

I then remember the `Reader` class from `java.io` namespace. Alright, I think `StringReader` is
perfect candidate for this as it will scan the String forward, char by char. In Clojure this can be implemented
like this.

```clojure
(def s (StringReader. "Sangi"))

(.read s) ;; returns ASCII code of character read
(.skip s 1) ;; skip one char
```

Due to digraphs, I think this is better solution. I can skip one char or read two chars at once. Next question,
will be: how do I somehow know the next character without advancing the cursor? Well, upon peeking the clojure-csv
code, I found out a way to mark before advancing, from `Reader` class.

```clojure

(defn- rdr-peek
  [^Reader rdr]
  (.mark rdr 1) ;; set a mark
  (let [c (.read rdr)] ;; read and advance
    (.reset rdr) ;; back
    c))
```

This will return the next char without advancing. This is very good for my use case. So the replacement
implementation becomes.

```clojure

(defn tokenize-rdr
"Tokenize using a Reader"
[^Reader rdr]
(loop [tokens []
       current (.read rdr)
       ahead (rdr-peek rdr)]
  (let [cc (str (char current))]
    (cond (== -1 ahead) (conj tokens cc) ;; Check if it's end of string
          (within? all-con (lower-case (str cc (char ahead)))) 
            ;; Check digraphs
            (let [digraph (str cc (char ahead))]
              (.skip rdr 1) ;; if digraph advance two chars
              (if (== -1 (rdr-peek rdr)) (conj tokens digraph)
                  (recur (conj tokens digraph) (.read rdr) (rdr-peek rdr))))
          :else (recur (conj tokens cc) (.read rdr) (rdr-peek rdr)))))) ;; if not digraphs advance 1 char


;; tokenize-word replacement
(defn tokenize-word
"Tokenize a Word"
[^String s]
  (tokenize-rdr (StringReader. s)))
```

It runs okay, until I discover another thing, transient collections.

<a name="3"></a>

## Transient collections

Persistent data structure is encouraged when using Clojure. Clojure also provides a mutating data structure called
**transients**. Wait What? [MUTATION, THAT'S HARAAAM!](https://en.wikipedia.org/wiki/Haram). 

![Haram](/assets/posts/haram.jpg)

Relax, Clojure is not pure functional language. As you can [read from the docs](http://clojure.org/transients):

> If a tree falls in the woods, does it make a sound?

> If a pure function mutates some local data in order to produce an immutable return value, is that ok?

In short, Clojure is pragmatist. Sometimes you can't rely on purely immutable data structure if you need
performance. From the implementation above, the new vector of tokens is created by taking old vector and appending
the last token with `conj`. In transients, we mutate the vector. The difference when using transient is rather than
using `conj`, we use `conj!`. In Clojure, function with exclamation mark signifies an _exciting_ part code. And we
still need to return a persistent data structure by calling `persistent!`. The implementation of `tokenize-rdr`
becomes:

```clojure
(defn tokenize-rdr
  "Tokenize a reader"
  [^Reader rdr]
  (loop [tokens (transient [])
         current (.read rdr)
         ahead (rdr-peek rdr)]
    (let [cc (-> current char str)]
      (cond (== -1 ahead) (persistent! (conj! tokens cc))
            (within? all-con (lower-case (str cc (char ahead))))
              (let [digraph (str (char current) (char ahead))]
                (.skip rdr 1)
                (if (== -1 (rdr-peek rdr)) 
                      (persistent! (conj! tokens digraph))
                      (recur (conj! tokens digraph) (.read rdr) 
                               (rdr-peek rdr))))
            :else (recur (conj! tokens cc) (.read rdr) (rdr-peek rdr))))))
```

The changes follows this pattern:

  1. Calling transient on the initial vector
  2. Using `conj!` rather than `conj`
  3. Call `persistent!` on function return

<a name="4"></a>

## Summary

From the time I started writing this small library, I've been learning a lot from it. I learnt two new languages
(Ruby and JavaScript) and discover their quirks and advantages. I discovered more and more about Clojure. It makes
me love it more.
