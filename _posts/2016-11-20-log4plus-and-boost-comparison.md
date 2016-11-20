---
layout: post
title: "Comparison: log4cplus vs boost"
---

Lets compare [log4cplus](https://github.com/log4cplus/log4cplus) and [Boost Logger](http://www.boost.org/doc/libs/1_61_0/libs/log/doc/html/index.html).
Log4cplus is ready-to-go solution similar to log4cxx, log4j, log4net, python logger etc. Boost Log is a framework which allows to build you own logger, based on slightly different ideas (non-hierarchical).

Personally I have some experience with log4cplus and had no experience with boost. Since boost is very popular and there is a high chance that it’s already used in the project the one is working on, then it’s attractive choice to use Boost’s logger and don’t add additional third party library for logging.

To have arguments to choose log4cplus or boost I’ve created the pet project which aim is to test log4cplus and boost and give some arguments.

I’ve created [logger-facade project](https://github.com/malirod/logger-facade). This project allows to test different scenarios with both loggers and test them with sanitizers and valgrind to catch memory and thread issues. Here is also [Travic-CI log](https://travis-ci.org/malirod/logger-facade/builds/169418326) which test some cases and have some execution timings.

Both loggers can work in sync and async modes, build statically. Loggers were tested with sanitizers (address, memory, thread, undefined behaviour) and valgrind.

Found several issues
* in boost with async logger there are numerous data races. Possibly this is because of not proper usage of boost, but this requires additional and deep investigation (possibly contacting with maintainers of the lib) because existing documentation and samples are quite limited.
* Log4cplus has minor warning from memory sanitizer about usage of un-init var during library initialization.
* Log4cplus has several memory leaks during initialization when async logger is used. These leaks are constant, it means that logging itself doesn’t leak. I’m contacting with maintainer on [this matter]( https://github.com/log4cplus/log4cplus/issues/203)

I had interesting timings locally which reproduced on CI (see [the CI build log](https://travis-ci.org/malirod/logger-facade/builds/169418326))

Build of test app with boost takes almost twice longer than with log4cplus. Logging with boost is almost twice slower than with log4cplus. Async logging is **slower** than Sync logging for both boost and log4cplus.
This is actually quite interesting. Possibly there is no need to bother with async logging at all.

 **Mode** | **Timing**
:---:|---:
Build (clang, debug) with boost | 11.681s
Build (clang, release) with boost | 18.037s
Build (clang, debug) with log4cplus | 6.683s
Build (clang, release) with log4cplus | 11.635s
Run (clang, debug, async, boost) | 317 ms
Run (clang, debug, async, log4cplus) | 155 ms
Run (clang, debug, sync, boost) | 256 ms
Run (clang, debug, sync, log4cplus) | 137 ms


Additionally I want to mention support from log4cplus maintainer. During my investigations I’ve posted several issues to GitHub and I’ve got feedback very fast (during few hours) and [crash](https://github.com/log4cplus/log4cplus/issues/205) which I’ve found on unstable version was addressed with pull request with fix during the same day.

Taking into account mentioned above I think that Boost is not a silver bullet and even if project already has Boost, then log4cplus is still better choice and it worth to add additional third party library.
