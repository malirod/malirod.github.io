---
layout: post
title: Build systems. Cmake alternatives.
---

Let me share my experience gained while I was working on [logger-facade](https://github.com/malirod/logger-facade).

At the beginning I wanted to test some new build system to find something as a replacement to cmake.
Cmake is quite popular but on my opinion it has several drawbacks:
* this is not a build tool, it just generates make\solution files
* it has its own syntax which we have accept and live with to make some extra stuff we usually have to write shell scripts and use them from cmake files. This makes the whole system not solid. In case of cross-platform system we end up with hard mix of cmake, sh, bat\cmd files.
* Cmake doesn’t cache build results. so project build makes long time.

## [Gradle](https://gradle.org/)

At the beginning Gradle was chosen. Seems this is a trendy tool. It’s site states that it suites best for CI\CD. Gradle is a java-based general build tool not restricted with language or platform. As far as I understand, originally it was designed as a replacement for maven and not so long ago native projects support was introduced.

Some sketch of project was successfully build. But shortly I faced with the following issues:

* documentation looks great at the beginning, but if you need to do something which is not explicitly described in the tutorials, then it’s hard to figure out what to do.
* Gradle uses some Groovy base language for build scripts. This is something completely new to me and it was hard to proceed with customizations.

Finally I’ve got the impression, that Gradle owners makes money on selling support. Their documentation is cool-looking but useless, but there are a lot of proposals to buy a course\training.

So Grandle was abandoned.

## [Waf](https://github.com/waf-project/waf)

Waf is a Python-based framework for configuring, compiling and installing applications.
Please read a little it's readme to get some basic understanding. I want to add some words. This is successor of Scons (also quite popular build tool). Some time ago KDE community had plans to move from autotools to Waf. Bud since Waf was only alpha version at that moment they choose Cmake.

## Benefits

Waf is open source and has very good documentation ([Waf book](https://waf.io/book/)).
Build scripts are written in normal Python, so it’s possible to use full power of Python for any custom task. No need to install anything to start using Waf. Just put “waf” file to the project root and commit with sources. That’s it!
Out-of-the-box support of caching of build results - functionality similar to ccache is integrated.

With Waf further development went smoothly without big troubles.

## Conclusion

Waf tool looks promising.
[Final script](https://github.com/malirod/logger-facade/blob/master/wscript) looks understandable and maintainable. For small projects I would recommend to consider this build tool. Maybe it will suite big projects too, [there is quite impressive list of projects](https://waf.io/projects_using_waf.html), which use Waf.
