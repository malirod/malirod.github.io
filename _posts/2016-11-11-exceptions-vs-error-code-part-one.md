---
layout: post
title: Exceptions vs ErrorCode. Part 1.
---
Usually in application we have to deal with some kind of errors. There are two general idea how to deal with them.

## Error code

### Benefits

Can be used everywhere (embedded systems, mobile platforms etc). No special skills required from developer to be able to works with such approach.

### Drawbacks

Usually error codes are ignored. No mechanism to force error handling. This might complicate ongoing support. Cannot break execution if required, we have to use some kind of abort (or we have to handle all cases manually).

## Exceptions

### Benefits

Quite clear idea, no overhead if not used (means that if nothing is thrown then no performance penalty), can break execution.

### Drawbacks

It’s not easy to write exception safe code. The ones, who thinks that he (she) knows all about exceptions, please see the series of talks listed [here (Part I, Part II, Part III)](http://exceptionsafecode.com/)

The link will be very useful for the ones know little about exceptions.

### Problem

What if we have to return from a function error or some value (object)?
Of course we can use `std::pair` or `std::tuple`, but  this applicable only to object with default ctor (we have to have a possibility to create some dummy value if there is error). Also this approach forces to create all items of pair\tuple even if only one is required.

Plus It’s desirable to force caller to handle result somehow.

## Solution

[ErrorOr template](http://llvm.org/docs/doxygen/html/classllvm_1_1ErrorOr.html) is the starting point for the solution.

**TO BE CONTINUE...**
