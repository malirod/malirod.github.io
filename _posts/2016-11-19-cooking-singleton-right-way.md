---
layout: post
title: Cooking C++ Singleton. The right way.
---
Let me share ideas how to use singletons in C++ without its known drawbacks.
Don’t hesitate to give some feedback.

## Summary
Usually application has some global objects which can be used in different components on different levels\layers. E.g. config, engine, tasks scheduler etc. It’s desirable to have a possibility to pass such global objects to any component and layer of the app easily and have a possibility to set some other implementation of the object if required(mock for tests).

## Challenge
There are two approaches

#### Pass & store ref to global object.
E.g. config is created in main function and passed to all required places though ctors or methods args. So we end up with a code where global objects\interfaces are sticking around.

#### Use some kind of global objects

- Simple global object defined out of main and access with “extern” from the rest of components. This is really wrong approach due to many reasons, e.g. construction and destruction order is undefined, cannot use global objects from each other, not easy to replace real impl with some mock in tests.
- Use singleton pattern with deferred construction.
E.g. the following implementation (note: before c++11 this impl is not thread-safe since static construction is not thread-safe)

```c++
ObjType& GetInstance() {
 static ObjType instance;
 return instance;
}
```

If this function is called somewhere in the beginning of the main then we will have predictable init order. If code will be used before any thread is created then we don’t care about data races on construction of the static value, thus this is usable even in pre-C++11. But there are issues: hard to mock, destruction order is undefined and destruction starts after main when some parts of system can be already destroyed.

## Solution
Two simple ideas “template tagging” and “accessor” allow to keep benefits of global objects and remove all drawbacks. Tagging is implemented in the following way

```c++
template <typename T, typename Tag = T>
T& single() {
 static T t;
 return t;
}
```

This allows to get multiple singletons of the same type

```c++
class CustomTag;
auto& foo = single<Foo>();
auto& custom_foo = single<Foo, CustomTag>();
```

Accessor has the following implementation

```c++
template <typename T, typename Tag = T>
class SingleAccessor {
public:
 void Attach(T& single);
 void Detach();
 bool GetIsAttached() const;
 operator T&();
 T& GetRef();
private:
 T* ptr_single_ = nullptr;
};
```

## Usage sample
Define accessor(pay attention that accessor references to interface but not to the implementation)

```c++
using IoServiceAccessor =SingleAccessor<IIoService>;

IoServiceAccessor& GetDefaultIoServiceAccessorInstance() {
  return single<IoServiceAccessor>();
}
```

Somewhere in main

```c++
using IoServiceAccessor = SingleAccessor<IIoService>;

thread_pool_ = util::make_unique<ThreadPool>(thread_pool_size, "main");
GetDefaultIoServiceAccessorInstance().Attach(*thread_pool_);
GetDefaultSchedulerAccessorInstance().Attach(*thread_pool_);

// Do the job
// Access this way
auto& asio_service =
  GetDefaultIoServiceAccessorInstance().GetRef().GetAsioService();
// ...
GetDefaultIoServiceAccessorInstance().Detach();
GetDefaultSchedulerAccessorInstance().Detach();
```

See the following links for implementation details

- [Implementation](https://github.com/malirod/flat-async/blob/8fb7489757cd68c4271580f15d58dd5cf70ab605/flatasync/src/util/singleton.h)
- [Tests](https://github.com/malirod/flat-async/blob/8fb7489757cd68c4271580f15d58dd5cf70ab605/flatasync/test/util/singleton_test.cc)
- [Usage sample](https://github.com/malirod/flat-async/blob/8fb7489757cd68c4271580f15d58dd5cf70ab605/cppecho/src/core/engine_launcher.cc)

## Benefits
Construction and destruction of the objects are well defined. Since accessor is defined basing on interface we can replace implementation easily and this way pass desired implementation to any component\layer. Tagging allows to have few global objects E.g. “main scheduler” for common tasks, “network scheduler” for network io tasks etc.

## Disadvantages
I see no disadvantages, except the fact that such approach might encourage a developer to create too much singletons so app will become more complicated (architecture not clear and straightforward).
