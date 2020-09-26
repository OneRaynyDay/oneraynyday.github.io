---
published: true
title: C++ Concurrency - Sharing Data
category: dev
layout: default
---

# Table of Contents

* TOC
{:toc}

# Inherent Problems

Every thread is considered a **lightweight process**. It has its own stack space,
but will share heap space with other threads of the same process.

When we're sharing data, the issue arises **when the data is mutable**.

There are 2 ways to deal with these **race conditions**:

1. Use a protective data structure(locks).
2. Use atomic operations(lock-free programming).

1 is usually much easier than 2. So we will cover 1 here.

# `std::mutex`

An `std::mutex` supports `lock()` and `unlock()`, but we normally should never call this
directly.

RAII comes to the rescue: `std::lock_guard` is a class template and allows us to lock
the `mutex` during constructor call and unlock during destructor call.

```c++
#include <mutex>
using namespace std;
std::mutex my_mutex;
void foo(){
    lock_guard<mutex> guard(my_mutex); // locks the mutex
    doSomethingSynchronized();
} // unlocks when scope exits.
```

## Hide Your Shared Data from User

```c++
// relatively good code
class data_wrapper
{
private:
    some_data data;
    std::mutex m;
public:
    template <typename Function>
    void process_data(Function func){
        std::lock_guard<std::mutex> l(m);
        func(data); // calls the function
    }
};

...

// evil code
some_data* unprotected;
data_wrapper x;

void badfunc(some_data& protected_data){
    unprotected=&protected_data;
}

void foo(){
    x.process_data(badfunc); // now we have unprotected access!!!
}
```

A key takeaway from this is: **don't pass pointers/references to protected data outside of the lock!**

# Deadlock

This happens when two threads, each acquire lock A and B, and require each other's lock.
They wait on each other's locks forever because neither wants to give up their lock.

This can be solved if you force an order on the lock, i.e. A must go before B during a contention.

However, sometimes it's not possible, like when each lock holds critical data for different parts of the same object... and so we can try to lock both at the same time, but
hand over ownership in a smooth way using `std::adopt_lock`, which is an empty struct tag.

```c++
class X{
...
    friend void swap(X& lhs, X& rhs){
        if(&lhs == &rhs)
            return;
        std::lock(lhs.m, rhs.m);
        std::lock_guard<std::mutex> lock_a(lhs.m, std::adopt_lock); // a acquires ownership
        std::lock_guard<std::mutex> lock_b(rhs.m, std::adopt_lock); // b too!
        swap(lhs.something, rhs.something);
    } 
};
```

Some tips:

1. Don't nested lock.
2. Avoid calling user-supplied code while holding lock. (User could try to lock)
3. Acquire locks in fixed order.

# `std::unique_lock`

`std::unique_lock` is more flexible than `std::lock_guard`. It doesn't always own a mutex.

The ability to not own one allows us to construct the `unique_lock` without forcing to
lock anything:

```c++
    ... // same example as before
    std::unique_lock<std::mutex> lock_a(lhs.m, std::defer_lock); 
    std::unique_lock<std::mutex> lock_b(rhs.m, std::defer_lock); 
    std::lock(lock_a, lock_b); // finally lock here!
```

You can also call `unlock()` and `lock()` on the `unique_lock` itself. It's good for when you want to control when you want to lock/unlock the specific resource depending on branching conditions(i.e. does this thread really need to hold on to this lock for that long?).

# `std::call_once`

You can call a function only once by using a `std::once_flag`:

```c++
std::once_flag f;
std::shared_ptr<resource> resource_ptr;

void init_resource(){
    resource_ptr.reset(new resource);
}

void foo(){
    std::call_once(resource_flag, init_resource); // this will only be called once.
    ...
}
```

# `std::recursive_mutex` is C++'s `ReentrantLock`

Some times, you may need to recursively lock the same mutex from the same thread multiple times.
A normal lock cannot handle this situation, and it will deadlock.
This is most likely a bad design decision, and you need to lock N time and release N times, otherwise a deadlock will occur.

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
