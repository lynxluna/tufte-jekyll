---
layout: post
title: Introducing Choc
date: 2014-08-31T01:28:30+0700 
categories:
- Programming
tags:
- ios
- cocoa
- ruby
status: draft
type: post
published: true
icons:
- ruby
---

Have you ever been in the situation, when you include two or more dependencies only to find out that there are
duplicates object file on your dependencies and a proprietary library included on your XCode Project? The
proprietary library, say `libProprietary.a` includes `AFHTTPClient.o` from open source AFNetworking version 1.3.0
for example. You want to update it to 1.3.3 but you can't, you stuck with old version embedded in the proprietary
library. If this situation is familiar to you, then `choc` is the right tool for you. Read on.

<!--more-->

![ChocoBar](/assets/posts/choco-bar.jpg)

## Motivation

I've been in this situation on my last project maintaining legacy system. It includes proprietary, binary-only
libraries from former developer. I've successfully migrated the dependencies using amazing
[CocoaPods](http://cocoapods.org). However the proprietary library already includes old versions of Facebook SDK
and Google Analytics SDK which doesn't work and crash the app. If I include the newer libs via CocoaPods, the app
can't be compiled because of duplicate object file name. Thus, this tool was born.

I hacked the tool for a couple of hours, and I came up with the name `choc`. It just a wordplay of chocholate and
chop. This tool will remove unneeded object files from a universal library.

## How to Install and Use

The tool is written in ruby and installed using gem.

```bash
gem install choc
rbenv rehash # if you're using RBEnv
```

To use the tool, just invoke it with your proprietary library as the first parameters and the object list in the
next parameters, for example if you want to remove `JSONKit.o` and `AFHTTPClient.o`

```bash
choc libProprietary.a JSONKit.o AFHTTPClient.o
```

and it'll remove both object files from the lib. If you want to list the object files inside a library, you can do
it using `ar` tools included in the XCode.

```bash
ar -t libProprietary.a
```

I learnt ruby in the same day I wrote this tool, so pardon if the code is messy, features are minimal, and unexpected
bugs are many. But hey, you can't always file an issue in the [<i class="fa fa-github"></i> Github
Repository](https://github.com/lynxluna/choc). Pull Requests are welcome.
