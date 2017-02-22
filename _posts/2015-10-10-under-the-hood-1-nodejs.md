---
layout: post
title: Under The Bonnet 1 - NodeJS
date: 2015-10-10 01:18:27.000000000 +07:00
image: https://www.ykode.com/assets/posts/architecture_node.png 
categories:
- Programming
tags:
- libuv
- nodejs
- asynchronous
status: published
type: post
published: true
---

In the past few weeks I've put my thinking onto the trends of technology, particularly in programming and
electronics. In the spirit of [Martin Fowler on Making The Magic Go
Away](https://blog.8thlight.com/uncle-bob/2015/08/06/let-the-magic-die.html), I try to peek under the bonnet (or
you prefer 'hood' in AE) of a piece of recent technology. The first victim is the trendy framework that used by
high profile site like Paypal and Über, or even [Ohdio](http://ohdio.fm): [NodeJS](https://nodejs.org/en/). I hate
JavaScript, so why I looked at it?  Initially I want to bash node, but it turns out I'm amazed by the simplicity
and the elegance on how it was built. It does not make me love JavaScript, but it makes me intrigued on how it
works. In this article, I'll explain what happens inside NodeJS by explaining it in [Rubber Duck
Debugging](https://en.wikipedia.org/wiki/Rubber_duck_debugging) style.

<!--more-->  

## Old Is New Again

On the day when Windows 3.1 was released, in a good ol' 16-bit age, **there was NO THREAD**. Bummer. The way
Microsoft has done on Windows 3.1 was a thing named [cooperative
multitasking](https://en.wikipedia.org/wiki/Computer_multitasking#Cooperative_multitasking). The app developer must
voluntarily ceded its control and limit its execution for other app to be able to run simultaneously. If you happen
to read and write on [Floppy Disks](https://en.wikipedia.org/wiki/Floppy_disk) you'll block the other process.
There was no multi-thread until the release of Windows 95 with pre-emptive multitasking. It still run on single
processor, so it was just as good as running them interleaved with other tasks.

![Old is New Again](/assets/posts/old-is-new.jpg)

> If you're a redneck CS you may cringe when you read this. I'm aware that 'thread' is available on UNIX
> during that time in the form of [processes] that share memory. I use Windows to make the reader easily find
> resonance to ease understanding.

### Evented I/O
 
Multithreading was cool and hot (sounds like on oxymoron). Programming multithreaded app, however, is not trivial.
Things like [deadlock](https://en.wikipedia.org/wiki/Deadlock),
[livelock](https://en.wikipedia.org/wiki/Deadlock#Livelock),
[starvation](https://en.wikipedia.org/wiki/Starvation_(computer_science)), and [race
condition](https://en.wikipedia.org/wiki/Race_condition) is hard to debug. It is hard enough for single processor.
And then, multicore becomes mainstream. It's a blessing for users (more tasks, yay!). For developers, it adds
another nightmare: now I don't know if my thread running on which processor, they can be run in parallel.
Developers are seeking for a way to reduce the pain. Nowadays most of the operating system has an asynchronous I/O
and I/O notification in the form of [epoll](https://en.wikipedia.org/wiki/Epoll),
[kqueue](https://en.wikipedia.org/wiki/Kqueue), or [I/O Completion
Ports](https://en.wikipedia.org/wiki/Input/output_completion_port). You don't need to block your I/O read and write
operations. Instead you can wait for a notification to be sent. The time spent for waiting, can be used for another
tasks without spawning a new thread. It basically brings up 'cooperative multitasking' from OS level to application
level. It's like we're going back in time, refining our former art. 

Unlike a thread-based app, when we need more power, we just spawn another process that shares **nothing**,
no shared memory, no shared stack. This avoid many problems related to multithreaded programs. This is the
basic philosophy of NodeJS.

## Node Architecture

The block diagram below is the simplified version the architecture of NodeJS.

![Simplified NodeJS Architecture](/assets/posts/architecture_node.png)

That's essentially what NodeJS is. A JavaScript runtime and an evented I/O engine. The birth of node is in the
right time and in the right place: when browsers are competing each other for the fastest JavaScript
implementation, and at that time V8 was leading in the term of performance. The choice of JavaScript is, as so
much I hate it, is actually reasonable. It's a "language of the web" that, according to [Douglas
Cockford](http://www.crockford.com), is actually doing something right in its core:

> "... despite JavaScript’s astonishing shortcomings, deep down, in its core, it got something very right. When you
> peel away the cruft, there is an expressive and powerful programming language there. That language is being
> used well in many Ajax libraries to manage and augment the  DOM, producing an application platform for
> interactive applications delivered as web pages. Ajax has become popular because JavaScript works. It works
> surprisingly well" ~ **Douglas Cockford**

JavaScript is suitable becaues it is one of the popular language that introduces asynchronous programming. You can
ask a web frontend devs (in my case, it's [my coworker](https://twitter.com/anima)), if you can block the web UI
and make the user wait for process, and I can be sure that he/she will ask you if you're kidding. The JavaScript
JIT, runtime, and binding is provided by V8. I won't talk much about V8. Covering it will take a book than a post.
In short, V8 provides the JIT and runtime support for JavaScript. It is written in C++ by Google, initally for
their Chromium project.

## `libuv`, Is The Beast.

For Node to work, it needs to be bound to an evented I/O engine. That's where `libuv` comes. It provides a
cross-platform abstraction on top of epoll/kqueue/poll/iocp. Before `libuv`, node used `libev`, which itself
are modeled after `libevent`. Due to its limitation on Unix/Linux only platform, the [team decided to write an
"IOCP version" of
`libev`](https://groups.google.com/forum/#!activity/nodejs/2JvBi5ikhDgJ/nodejs/UwHkaOksprw/OD_7KxImEjYJ), and
`libuv` was born. Over the time, the responsibility of `libev` was being taken over by `libuv` [due to
performance reason and specific NodeJS need](https://github.com/joyent/libuv/issues/485). This is what happens
inside `libuv`:

![Event Loop](/assets/posts/event-loop.jpg)

It provides esentially three things: I/O event abstraction, the event loop and a thread pool. It includes many
things inside it, you can take a look at [their documentation](http://docs.libuv.org/en/v1.x/). I may have
separate post about `libuv` later on.

## One Code to Run Them All

`libuv` is written in C for portability. It can be compiled and run on most of mainstream operating systems,
including iOS and Android. NodeJS binds V8 with `libuv` and adds some additional support like compression,
HTTP parsing, and TLS support. You can see the dependencies by reading [its build
script](https://github.com/nodejs/node/blob/master/node.gyp). If you dig the entry point of NodeJS in
[`node.cc`](https://github.com/nodejs/node/blob/master/src/node.cc), all it does is setting up the V8
environment and starting the `libuv` event loop:

```cpp
int Start(int argc, char** argv) {
  PlatformInit();

  CHECK_GT(argc, 0);

  // Hack around with the argv pointer. Used for process.title = "blah".
  argv = uv_setup_args(argc, argv);

  // This needs to run *before* V8::Initialize().  The const_cast is not
  // optional, in case you're wondering.
  int exec_argc;
  const char** exec_argv;
  Init(&argc, const_cast<const char**>(argv), &exec_argc, &exec_argv);

#if HAVE_OPENSSL
  // V8 on Windows doesn't have a good source of entropy. Seed it from
  // OpenSSL's pool.
  V8::SetEntropySource(crypto::EntropySource);
#endif

  const int thread_pool_size = 4;
  default_platform = v8::platform::CreateDefaultPlatform(thread_pool_size);
  V8::InitializePlatform(default_platform);
  V8::Initialize();

  int exit_code = 1;
  {
    NodeInstanceData instance_data(NodeInstanceType::MAIN,
                                   uv_default_loop(),
                                   argc,
                                   const_cast<const char**>(argv),
                                   exec_argc,
                                   exec_argv,
                                   use_debug_agent);
    StartNodeInstance(&instance_data);
    exit_code = instance_data.exit_code();
  }
  V8::Dispose();

  delete default_platform;
  default_platform = nullptr;

  delete[] exec_argv;
  exec_argv = nullptr;

  return exit_code;
}
``` 

In my opinion, Node's code is very clean, readable, and modular. I usually search the entry point to study the code.
The `Start` function is NodeJS entry point.  It's the only code called by `main` function inside
`node_main.cc`. We can clearly see what the code doing here:

- Run Node's Platform Specific : `PlatformInit()`.
- Initialise the `libuv` : `Init`.
- Set up the V8 Platform and Runtime: `V&::InitializePlatform()` and `V8::Initialize`.
- Create and start a node instance by filling up `NodeInstanceData` and `StartNodeInstance`. This is where the
  event loop runs.
- Catch the exit code.
- Dispose V8 engine.
- Clean up the resources.

So whenever you run any NodeJS application, this piece of code is always called in the first time. From here
we can traverse back to the call graph. I'll be going through each of the most important functions.

### `PlatformInit()` Function

Still in the same file, `node.cc` if we traverse back the call graph we have `PlatformInit()` function. Personally, I
think the name is a little bit misnomer. The function is used to register and handling [POSIX
signals](https://en.wikipedia.org/wiki/Unix_signal#POSIX_signals) and setting up logging. It also setting up
the handler for `SIGINT` and `SIGTERM`. On other platforms that do not define `__POSIX` like Windows, this
function is [NO-OP](https://en.wikipedia.org/wiki/NOP).

### `Init()` Function

This function is straightforward too. In this function we pass the command line arguments to both `libuv` and
`V8`. It's also where the timer is initialised to get relative application uptime. Other tasks that this
function does is setting up the debug message dispatching. I'm surprised that Node is actually using `libuv`'s
default loop, as it calls `uv_default_loop()` everywhere. When the function succeeded it will set the
`node_is_initialized` variable to `true`. Thus marking that the node is ready to start its event loop.

### `StartNodeInstance()` Function

So this is the crux of all of it. The function is very much self-explanatory. _It starts an instance of Node
process_. First it will get `instance_data` from parameter passed to it. And then it will set up the V8
`Isolate`. From V8 documentation:

> `Isolate` represents an isolated instance of the V8 engine. V8 isolates have completely separate states.
> Objects from one isolate must not be used in other isolates. The embedder can create multiple isolates and
> use them in parallel in multiple threads. An isolate can be entered by at most one thread at any given time.
> The Locker/Unlocker API must be used to synchronize.

An *embedder* on our case refers to NodeJS. To create an `Isolate` we need a `CreateParams`. In this param,
Node assign an `array_buffer_allocator` which it will assign to an instance of `ArrayBufferAllocator`. The
definition of the allocator is on `node_internals.h`, so I guess it is something that node-specific. Node is
using a `malloc` and `calloc` based allocator. You can see it in the `Allocate` implementation in `node.cc`
file.

The heart of the process is in here:

```cpp

//... redacted

Locker locker(isolate);
Isolate::Scope isolate_scope(isolate);
HandleScope handle_scope(isolate);
Local<Context> context = Context::New(isolate);
Environment* env = CreateEnvironment(isolate, context, instance_data);

//... redacted

LoadEnvironment(env);

//... redacted
{
      SealHandleScope seal(isolate);
      bool more;
      do {
        v8::platform::PumpMessageLoop(default_platform, isolate);
        more = uv_run(env->event_loop(), UV_RUN_ONCE);

        if (more == false) {
          v8::platform::PumpMessageLoop(default_platform, isolate);
          EmitBeforeExit(env);

          // Emit `beforeExit` if the loop became alive either after emitting
          // event, or after running some callbacks.
          more = uv_loop_alive(env->event_loop());
          if (uv_run(env->event_loop(), UV_RUN_NOWAIT) != 0)
            more = true;
        }
      } while (more == true);
}
// ... redacted
```

The first part of the code is required to start a V8 instance. It set the `Locker`, `HandleScope`, and `Context`.
For more information about them, you can refer to [V8 Embedder Guide](https://developers.google.com/v8/embed). The
second part, the `LoadEnvironment` is a magic. It's actually loading a bootstrap script, namely `node.js`. I think
the framework is named after this bootstrap script. It is the first script executed, and further it will also load
your app. See this part of code in the `node.cc` inside `LoadEnvironment` function:

```cpp
// ... redacted
  
  Local<String> script_name = FIXED_ONE_BYTE_STRING(env->isolate(), "node.js");
  Local<Value> f_value = ExecuteString(env, MainSource(env), script_name);
  if (try_catch.HasCaught())  {
    ReportException(env, try_catch);
    exit(10);
  }
  CHECK(f_value->IsFunction());
  Local<Function> f = Local<Function>::Cast(f_value);

// ... redacted

  Local<Value> arg = env->process_object();
  f->Call(global, 1, &arg);

```

These piece of code will load the `node.js` bootstrap script and then execute the top level function. In the
`node.js` script it is the `process` function. After that, all execution are handed over to modules. Modules
provides additional functions outside what provided by V8. By using modules, V8 can communicate and access
underlying systems. You can inspect the source code of `node.js` to see which modules is loaded on startup.
`libuv` functionalities also wrapped in modules. The built-in modules are located in the `lib` directory.
There are a bunch of JavaScript files there that provides a standard library for NodeJS. Most of the modules
provides `libuv` functionalities to the JavaScript world.

Going back to `StartNodeInstance` function, we see a `do..while` block. This is the loop where all the
processing going on. It pump the V8 message, and give the control to `libuv` to inspect if there's more event
request over there by calling `uv_run` with `UV_RUN_ONCE` on the event loop. It'll poll the I/O once and
return nonzero if there is pending callbacks are expected and will save that to `more` as an indicator there
are callbacks. If `more` is `false` then we do one more message pump and running the `BeforeExit` event hook and
checking if the event loop is still alive by checking the `uv_loop_alive` and after that it'll run the event
loop again by calling `uv_run`. This `BeforeExit` event can make asynchronous call and causes NodeJS to
continue.

## Conclusion

NodeJS is a very interesting piece of engineering. I'm surprised on how simple the premise of the framework
is. At beginning it was just a JavaScript wrapper for `libev` and `libaio` to provide JavaScript programmable
server-side evented I/O.  It grows interest over the time and creating a whole ecosystem by itself. Despite
its interesting history of breakup between [nodejs and
iojs](http://anandmanisankar.com/posts/nodejs-iojs-why-the-fork/), the contentious [tension between Joyent
employee and node contributor](https://www.joyent.com/blog/the-power-of-a-pronoun), it evolves, and becomes
one of the most important project in Open Source community. The code base is still relatively small, the code
is readable and easy to follow. I hope this UTB article can help you understand how Node works and can be a
starting point if you want to contribute on NodeJS. 

If you want to see the very first presentation by Dahl in 2009 explaining NodeJS, it's still perserved in
[JSconf.eu website](http://www.jsconf.eu/2009/video_nodejs_by_ryan_dahl.html).
