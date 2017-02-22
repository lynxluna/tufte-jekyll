---
layout: post
title: The Writing of Libil
date: 2014-09-05T01:28:30+0700 
categories:
- Programming
tags:
- ruby
- clojure
- caesar
- cipher
- javascript
status: draft
type: post
published: true
icons:
- ruby
---

In the past few days. I've been playing with some new languages that I've not been familiar with. One of them is
Ruby. I want to be familiar with this language ever since I started writing `Vagrantfile` for my infrastructure
needs, and `Rakefile` for my iOS development need. I currently have fully automatic workflow that can be easily
plugged to any Continuous Integration software based on Ruby. I haven't played with Rails yet as I didn't find any
need of it. For a new language, I decided to create a common algorithm to be solved. When I was studied in
Yogyakarta, there's a slang language used there. This slang language is a form of vocalised Caesar Cipher with
local script as the rule. I decided to create a RubyGems based on it for fun.

<!--more-->

## The Rule

The rule of this slang language, also known as **Bahasa Walikan** (flipped language) is based on Javanese Script.
Javanese script is a syllabic script based on 20 basic letters arranged in four rows:

![Hanacaraka](/assets/posts/hanacaraka.jpg)

Javanese script have two kinds of `d` and `t` sound, one labial one retroflex. Retroflex `t` sounds more or less
like Indian's `t` and retroflex `d` is spelled deep and plosive. Both retroflexs are written as digraphs 'th' and
'dh' respectively.  As it's syllabic script, to write using it, you need to break down the word or sentences by
syllable.

The rule is very simple, flip any consonant from row 1 with row 3 conterparts and row 2 with row 4. Thus syllable
`ka` becomes `nya` and `ba` becomes `sa` and so on. There's exception tho. If your word started with vocals you
need to regard them as `ha` as it's written in the script that way. Thus, `aku` becomes "haku". 

Let's try it using simple Javanese word `bali` which means _going home_:

  1. Separate the syllables : `ba` `li`
  2. Flip it: `sa` `ngi`

![Walikan](/assets/posts/hancaraka2.png)

So `bali` becomes `sangi` and vice versa. It's modified caesar cipher anyway. Rumour says that this cipher is used
by the people during the war of independence. Well, I think it's very convenient way of quickly encrypting message.
And people in Jogja nowadays, use it as day to day conversation without even thinking. They know 'sangi' is 'bali'
by heart.

## The Algorithm

Well if it's written in Javanese Script, the algorithm is very-very easy to write. However, as the script usage is
considered archaic nowadays, Javanese usually use latin script to express their language (yeah, we got latinised
too, grmbl). I want to do this as functional as possible. The steps I used are below:

  1. Tokenize the word to letters, carefully considering digraphs. Save it to an array.
  2. Map the array.
  3. Join the mapped syllables to form the final word.

Easy! I used `Enumerable` in ruby to execute this algorithm. The most challenging part of this task is the
tokenizing algorithm. Thankfully digraphs are only 4, so I can just use simple conditional to do it.

## Implementation

I don't know if it's more effective or not. But I think Ruby's StringScanner is convenient to tokenise string. This
is the raw implementation of the string tokenization.

```ruby
# This is to tokenize the word to be mapped later on when converting
  def self.tokenize(word)
    # scan linearly, not yet know how to tokenize fast
    t = []
    ss = StringScanner.new(word)
    loop do
      break if ss.eos?
      c = ss.getch
      if (c.downcase == 'n')
        if not ss.eos?
          cc = ss.getch
          if (cc.downcase == 'y' or cc.downcase == 'g')
            t << [c, cc].join('')
          else
            t << c; t << cc
          end
        else
          t << c
        end
      elsif ( c.downcase == 'd' or c.downcase == 't')
        if not ss.eos?
          cc = ss.getch
          if (cc.downcase == 'h')
            t << [c, cc].join('')
          else
            t << c; t << cc
          end
        else
          t << c
        end
      else
        t << c
      end
    end 
    return t   
  end
```

I'm a little bit tripped on digraphs. I was a little bit rusty in programming. I might find something better for
this. However I think the code is good enough for now. So if you have word like `Cahe Dhewe` this will be tokenised
to `['C', 'a', 'h', 'e', ' ', 'Dh', 'e', 'w', 'e']`. Afer transformed to token array, it's just piece of cake. Just
map it! The core algorithm is like this:

```ruby
remap_tokens = tokens.map do |t|
      lowcase = t.downcase
      is_low = (lowcase == t)
      x = t

      if (Libil::CONSONANT_MAP.include?(lowcase)) 
       x = Libil::c_remap(lowcase)
      end
      
      if not is_low
        x = x.capitalize
      end
      x 
    end
return remap_tokens.join('')
```

What this functions do is create a new array based on the rules we've been set and return the joined array. Pretty
simple eh?

## Usage

Usage is pretty simple. Just install the RubyGems using `gem install libil` and you can use it as command line.

```bash
$ libil aku bali
panyu sangi
```

or if you want to use it on your ruby apps, include `libil` on your code.

```ruby
require 'libil'
converted = Libil::convert("Aku Bali");
```

## Ports

Over several days I've ported the library to some of my favourite languages, JavaScript and Clojure. All are
available from my [Github Page](http://libil.github.io). Please check it out. Clojure implementation is using
recursive implementation and make use of head-tail structure common in LISP. JavaScript implementation is able to
be run in browser and also as `npm` module.

## Summary and What's next.

This is made as an exercise to prevent my rusty mind. I'm actually thinking of doing the mapping parallel to
improve the time complexity from O(n) to O(log n) using divide and conquer approach. This might be good for long
paragraphs and string.
