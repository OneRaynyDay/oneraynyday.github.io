---
published: true
title: C++ Concurrency - Asynchronous Waiting
category: dev
layout: default
---

# Table of Contents

* TOC
{:toc}

Before, we talked about the basics of how C++ threads are
used, and how threads can protect data by using `mutex`es,
`lock_guard`s, `unique_lock`s, `recursive_mutex`, and `once_flag`s.

Now, we talk about how threads can wait for other threads to complete tasks.

# Waiting for an event/condition

There are multiple ways to check when a condition becomes true:

1. Spin-lock checking.
2. Sleep for small periods of time while spin-lock checking.
3. Use a **condition variable** which associates with some event, which will notify the thread waiting.

# `condition_variable` and `condition_variable_any`

Let's use a case study:

```c++
mutex m;
queue<data> q;
condition_variable c;

void prepare(){
    while(more_data()){
        data d = prepare_data();
        lock_guard l(m);
        q.push(data);
        c.notify_one(); // notify a waiting thread!
    }
}

void process(){
    while(true){
        unique_lock l(m);
        c.wait(l, []{return !q.empty();}); // notified!
        data d = q.front();
        q.pop();
        l.unlock();
        process(d);
    }
}
```

What happens here is that `wait(lock, boolfunc)` actually unlocks the
mutex and sets the thread to sleep.

Specifically, what `wait(lock, boolfunc)` does is the following:

1. Checks the condition of the `boolfunction`(could be a lambda like in e.x.)
2. If (1) is not true, then unlocks the lock and sets the thread asleep again(blocked state), waiting for (1) again.
3. If (1) is true, then keeps the lock locked, and proceeds.

As you can see, we can't simply use `lock_guard` to perform these operations, since
(2) needs to unlock the lock.

## Details of `wait()`

Actually, `wait()` does not need to check the bool function once - it could check it for any number of times.

In addition, a situation could occur, called a "spurious wake". In this case, threads can
be woken up regardless of whether they were notified, and they will test the `boolfunc`.

## An example of `safe_queue`

A `safe_queue` should support the following functions:

```c++
safe_queue(const safe_queue& other);
void push(T new_value); // Adds new value into queue
bool wait_and_pop(T& value); // assign popped front() into value
bool empty(); // checks whether any element left
```

We will have the following members:

```c++
mutex m;
queue<T> q;
condition_variable c;
```

For the copy constructor:

```c++
safe_queue(const safe_queue& other){
    // lock the other queue so we can copy their values safely
    lock_guard<mutex> l(other.m);
    q = other.q;
}
```

For `push`:

```c++
void push(T new_value){
    lock_guard<mutex> l(m);
    q.push(new_value);
    c.notify_one(); 
}
```

For `wait_and_pop`:

```c++
void wait_and_pop(T& value){
    unique_lock<mutex> l(m);
    c.wait(l, [this]{ return !q.empty(); });
    value = q.front();
    q.pop();
}
```

For `empty`:

```c++
bool empty(){
    lock_guard<mutex> l(m);
    return q.empty();
}
```

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
