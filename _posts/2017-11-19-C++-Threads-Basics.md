---
published: true
title: C++ Concurrency - Basics of std::thread
category: dev
layout: default
---

# Table of Contents

* TOC
{:toc}

# `std::thread` Syntax and Functions

```c++
#include <thread>
#include <iostream>
using namespace std;

void hello_world(){
    cout << "hello world!" << endl;
}

int main(){
    thread t(hello_world); 
    t.join();
    assert(t.joinable() == false);
    return 0;
}
```

In this case, we create a thread, bind a function to it,
and then wait for it to finish using `join()`.

What if we called `detach()`? Then even after the instantiation `t` gets destroyed(after leaving its scope), the thread will still continue operations.

After `join()` is done executing, we can guarantee that 
`t` is no longer associated with the actual thread, since
the thread's execution actually finished.

This means, `joinable()` will become false because
it's only true for an active thread of execution.

# You must `join` or `detach`

What happens if we don't call `join()` or `detach()`,
and just allow the thread's destructor to get called? 
Then in the destructor of `t`, it will check for whether `joinable()`, and if it is, it will raise `std::terminate()`.

Now you must be thinking, why so violent? `std::terminate()` should only be called in a few special circumstances like double exception propagation.

Say instead of `terminate`, it just `detach`'s the thread in the destructor.
What would happen?

**We could be inevitably allowing undefined behavior as
the (destroyed) child thread could be using references
to the scope of which was already destroyed.**
Thus, the designers of `std::thread` thought `termiante` 
was a necessary condition to avoid difficult UB-debugging.

We can't allow a thread to not `join()` in exception handling, so for example:

```c++
int main(){
    thread t(my_func);
    try{
        do_something(); // exceptions could be called!
    }
    catch(...){
        t.join();
        throw;
    }
    t.join();
}
```

... which looks very ugly.

We can also implement a `thread_guard` using RAII:

```c++
class thread_guard{
    std::thread& t;
public:
    explicit thread_guard(std::thread& t_) : t(t_) {}
    ~thread_guard(){
        if(t.joinable())
            t.join(); // force join here
    }
};

...

int main(){
    thread t(my_func);
    thread_guard g(t); // RAII
    do_something(); // exceptions could be called!
}
```

In this case, when `main()` exits, `~g()` is called before
`~t()` is called. Therefore, we will `join()` on t before
the destructor of `t` is called which could possibly
completely kill us.

When you `detach()` a thread, the thread is usually called a **daemon thread**, which runs in the background with no way of communication.

A case study of this is creating a new document window in a GUI:

```c++
int main(){
    while(true){
        int command = get_input();
        if(command == OPEN_NEW){
            thread t(open_document);
            t.detach(); // we create it and let it run.
        }
    }
}
```

# Don't `detach()` While Using Locals

```c++
void print(char* cp){
    printf("%s\n", cp);
}

void foo(){
    char buffer[1024];
    sprintf(buffer, "hello world!");
    std::thread t(print, buffer); // buffer is an argument.
    t.detach(); // uh oh!
}
```

In the above situation, we passed in a local variable, 
`buffer`, which is used by the thread in `print()`.

If we did a `join()`, this is fine. However, we `detach()`'d.
This means if the following sequence occured:

1. Create thread, calls the print function
2. `detach()` gets called, `buffer` is freed.
3. `printf()` is called with `cp`.

... then we have undefined behavior!

So keep in mind that when we're detaching, **never reference
local variables.**

# Transfering ownership of threads

The ownership model of `std::threads` is very much like
`std::unique_ptr`. You should use `std::move` to move current thread context 
from one RAII container to another.

```c++
thread t1(my_func);
thread t2 = std::move(t1);
```

After this execution, `t1` now is not `joinable()`, as it has no active thread context,
and `t2` instead has the thread context.

What if we did this:

```c++
thread t1(my_func);
thread t2(my_func);
t2 = std::move(t1); // uh oh!
```

Here, `t2`'s original active context will be removed, without `join()`'s or `detach()`'s.
We can only expect an `std::terminate` here.

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
