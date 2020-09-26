---
published: true
title: Essential C++ - Correct RAII
category: dev
layout: default
---

# Table of Contents

* TOC
{:toc}

# Use Objects for Resources

Pretty much this single rule: **don't use raw pointers for `new`'d objects in modern c++!** 

Why? Because you're liable for `delete`'ing the pointer. If it's not, you get a memory leak. If you run your program for a long time, you'll be wondering if you really bought an 16 GB macbook.

## How To Screw Up

This is a very subtle way to screw up:

```c++
void bar(){
    Foo* f = new Foo();
    ... // exception occurs!
    delete f; // We're good... right?
};

try{
    bar();
} catch(std::string& e){
    std::cout << e.what() << std::endl;
}
```

As a result of the exception, our flow is intercepted and we don't get to `delete f;`. That's a memory leak right there.

## How To Unscrew Yourself

**C++ objects are nice. They have destructors. Use it:**

```c++
void bar(){
    std::shared_ptr<Foo> f(new Foo());
    ... // exception occurs!
}; // we left scope, f is destroyed!

...
```

The standard library has `std::auto_ptr`, `std::shared_ptr`, and `std::unique_ptr` for the most basic RAII pointer management. _You probably shouldn't use `std::auto_ptr` unless you need to though_. We'll explain why below:


# Decide How to Implement Copy

There's multiple definitions of the idea "copy":

## The silent-killer-copy
```c++
std::auto_ptr<Foo> f1(new Foo());
std::auto_ptr<Foo> f2(new Foo());

f1 = f2; // f1 is nullptr
```

## The shared-refcount-copy
```c++
// denote content as c1. Count(c1) = 1
std::shared_ptr<Foo> f1(new Foo()); 
// denote content as c2. Count(c2) = 1. This syntax works too.
std::shared_ptr<Foo> f2 = std::make_shared<Foo>();

// Count(c1) = 0, Count(c2) = 2
f1 = f2; 
```

## The unique-move-copy
```c++
void foo(std::unique_ptr<Foo> f){
    ...
}

std::unique_ptr<Foo> f1(new Foo());

// COMPILER ERROR! Can't just copy unique_ptr!
std::unique_ptr<Foo> f2 = f1; 
// COMPILER ERROR AGAIN!
foo(f1);

// We're OK! Explicit move!
std::unique_ptr<Foo> f2 = std::move(f1);
// We're OK! Explicit move!
foo(std::move(f2));
```

## The weak-copy
```c++
std::weak_ptr<Foo> f1; // empty right now
{
    std::shared_ptr<Foo> f2 = std::make_shared<Foo>();
    f1 = f2;

    f1.use_count(); // returns 1.
    std::shared_ptr<Foo> f3 = f1.lock(); // takes ownership of the shared_ptr.
    ... // do stuff with f3
} // unlocks, f2 and f3 are destroyed.
f1.use_count(); // returns 0
f1.lock(); // returns an empty shared_ptr!
```

# Raw Resource Interface in RAII

Sometimes, an old library will not be using your fancy `std::???_ptr`'s, and will require passing in pointers themselves. You can't just pass in a smart pointer and expect it to work:

```c++
void bar(Foo* f){
    ...
}
std::shared_ptr<Foo> f = std::make_shared<Foo>();
bar(f); // Error! Not the correct type :(
```

## Explicit Conversion Solution

In all `std::???_ptr`'s, you can call the function `get()` to retrieve the underlying pointer:

```c++
void bar(Foo* f){
    ...
}
std::shared_ptr<Foo> f = std::make_shared<Foo>();
bar(f.get()); // We're good!
```

However, the drawback is that **`.get()` is often pretty ugly** if you wanted a smooth interface.

## Implicit Conversion Solution

You can specify a copy constructor, and you wouldn't need the `.get()`:

```c++
class FooPointer{
public:
    FooPointer(Foo f){
        _f = f;
    }
    public Foo(){
        return get(); // returns pointer
    }
    public get(){
        return _f;
    }
    ...
}

void bar(Foo f){
    ...
}

FooPointer f;
bar(f); // Works! Implicitly calls Foo()!
```

However, the drawback is that you might **cast `FooPointer` to `Foo` on accident**. That's why I don't think anyone should use this unless it's of great convenience.

# Use `new`/`delete` or `new[]`/`delete[]`

Pretty self explanatory:

```c++
std::string* s = new std::string("sup");
std::string* s_ = new std::string[100];

delete s;
delete [] s_;
```

Here's a picture for reference. You can see what happens when you make a mistake:

![array_layout]({{ site.url }}/assets/array_layout.png)

## Don't Typedef Arrays/Pointers To Arrays!

Typedefs are already super confusing if you don't use the simple ones, but to add fuel to the fire:

```c++
typedef std::string StringTenTuple[10];

std::string* s = new StringTenTuple;

delete s; // NOPE!
delete [] s; // YUP!
```

That triggers me holy.

# Dedicate Whole Expression to ONLY RAII

In fact, in `cppreference`, they even stated that there are some issues with this, and that's why `std::make_shared<Foo>(...)` was created:

```c++
void bar(std::shared_ptr<Foo> f, int v){
    ...
}

bar(std::shared_ptr<Foo>(new Foo()), bad_function()); // DONT DO THIS!
```

Why? Because of the way execution is done on these variables! There are 4 statements here:

1. `new Foo()`
2. `bad_function()`
3. `std::shared_ptr<Foo>(...)`
4. `bar(...)`

Now, if we followed this order, we would be in big trouble! We allocated `Foo` on the heap, but before we were able to safely stuff it into `std::shared_ptr<Foo>(...)`, we ran into an exception in `bad_function`. 

Did someone say **memory leak**?

That's why the standard library made `std::make_shared<Foo>(...)`:

```c++
void bar(std::shared_ptr<Foo> f, int v){
    ...
}

bar(std::make_shared<Foo>(), bad_function()); // THIS IS OK!
```

Since steps 1 and 3 are now in the same step. 

However, just **don't do this in the first place. Create your smart pointer on its own line.** 

```c++
void bar(std::shared_ptr<Foo> f, int v){
    ...
}

std::shared_ptr<Foo> f(new Foo()); // On its own line
bar(f, bad_function()); // ALWAYS SAFE!
```

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
