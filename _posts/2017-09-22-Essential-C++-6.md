---
published: true
title: Essential C++ - Object Oriented Programming
category: dev
layout: default
---

# Table of Contents

* TOC
{:toc}

# Public Inheritance is "is-a"

What is "is-a"? It's a directed relationship:

For example, "`B`,`C`'s are `A`'s" translates directly to:

```
  A (Base class)
 / \
B   C (Derived classes)
```

This is good, since that means we can have the following signature perform polymorphic behavior:

```c++
void func(const A& obj){ // Remember polymorphism only works in pointer-types!
    ... // do something
}

// All are OK!
func(A());
func(B());
func(C());
```

## Nitty Gritty: "is-a" is tricky

If you expose an interface in `A`, you must expose it to `B` and `C` and they **must make sense, since you said `B` and `C`'s are `A`'s.**

For example, you may think that:

```
Bird (Base class)
 | 
Penguin (Derived class)
```

And we think that `Bird`s can fly, so we expose a `fly()` function. However, `Penguin`s cannot fly! We can resolve this in 2 ways:

### Solution 1: Generalize base class to introduce granular relationships

If you make the inheritance more granular you may be able to make bird not have a fly() function, but rather have `Flyable` birds have a fly() function. Then `Penguin` will derive from `Nonflyable`.

```
     Bird (Base class)
    /       \
Nonflyable  Flyable
   |
Penguin (Derived class)
```

### Solution 2: Throw exception

Another solution is to place the onus on the user. **It's the user's fault that they called a `fly()` function on a `Penguin`, not that the behavior is undefined, but it's most likely not what they're looking for, so we throw.**

```c++
class Penguin{
public:
    void fly(){
        error("Penguins can't fly");
    }
};

Penguin p;
p.fly(); // COMPILER ERROR!
```

# Composition is "has-a"

In contrast to "is-a", which is implemented using inheritance, "has-a" is implemented via 

# Avoid hiding inherited names

While reading, I found this to be really surprising:

```c++
class Base{
public
    virtual void foo();
    virtual void foo(int);
};
class Derived : public Base{
public:
    void foo();
};

...

Derived d;
d.foo(); // is fine
d.foo(3); // error
Base* b = new Derived();
b->foo(2); // is fine
```

## Why does an error occur? Don't we want to inherit the overloaded functions?

Well to first explain why we don't see the `Base::foo(int)`, we have some scoping issues:

```
scope global{
    scope Base{
        foo <-- only if we call Base::foo!
        scope Derived{
            foo <-- takes precedence!
        }
    }
}
```

But the reason it is implemented like this is **to prevent distant base classes being chosen over derived classes overloads.** Here's a conversation between me and [Jerry Coffin](https://stackoverflow.com/users/179910/jerry-coffin) from stack overflow (he knows his C++):

Me: 
> Oh geez, that seems like a language mistake. 
Any reason for why it's implemented like this?

Jerry Coffin:
> Though the route is rather long and twisted, it mostly comes down to implicit conversions.

> A derived class effectively defines an inner scope, with the base class at an outer scope. Searching always starts at the innermost scope, and proceeds outward until something by that required name is found... 
 
> Now, let's consider what could happen if we looked for the names at all the outer scopes, and picked the best "fit" for what's being done. Consider if I have something like: `int f() { double foo; foo = 1; }`

> Right now, it's clear that we're assigning 1 to `foo` (after converting it to 1.0).

> ... In this case, somebody could completely change the behavior of f by defining `int foo;` completely outside of f. Now the compiler would find that outer `foo`, and think: "the value being assigned is an int, and `::foo` is an int, so that's a better fit than a double. The next time `f` gets called, it blows up and does bad things because its `foo` hasn't been initialized.

Me:
> Ah I see, so you mean something like:

```c++
namespace global{
int foo;
namespace inner{
double foo;
foo = 1; // the global variable is modified! bad!
}
}
```

> So this is an inherent tradeoff between scoping variables and ignoring outer-scope overloads?

Jerry Coffin:
> Exactly.

## How do we fix it?

The way to fix this issue is to **drag the outer scope variables inside.** When you do something like `using namespace std`, you're dragging all variables from `std::` scope into the current scope.

When you do that, you call for trouble. But in this case, it's exactly what we need:

```c++
class Base{
public
    virtual void foo();
    virtual void foo(int);
};
class Derived : public Base{
public:
    using Base::foo; // drags all Base::foo's down!
    void foo();
};

...

Derived d;
d.foo(); // is fine
d.foo(3); // is fine, calls Base::foo(int)
```

**Now, if you actually don't want some overloaded functions to exist, then you're gonna need to be more picky:** We'll employ the tactic of a forwarding function:

```c++
class Base{
public
    virtual void foo(); // we want this overloaded
    virtual void foo(int); // we want this
    virtual void foo(std::string); // we don't want this!
};
class Derived : public Base{
public:
    void foo(); // different impl
    void foo(int); // forwarding function
};

...
void Derived::foo(int x){
    Base::foo(x); // forwarding!
}
...

Derived d;
d.foo(); // is fine
d.foo(3); // is fine, calls Derived::foo(int), which calls Base::foo(int)
d.foo(std::string("Hello world!")); // error! (which is good)
```

# Differentiate inheritance of interface/implementation

# Inheritance of interface

It's pretty much inheriting a **pure-virtual function**, like:

```c++
class Base{
    ...
    virtual void foo() = 0; // pure virtual
};
```

# Inheritance of implementation

It's pretty much inheriting a **simple-virtual function**, like:

```c++
class Base{
    ...
    // simple virtual
    virtual void foo(){ 
        ... // does something
    } 
};
```

This is used for when you need a default to fall back to.

Sometimes, the developer may be clumsy and _forget that there is a simple virtual implementation_:

```c++
class Derived{
    ...
    // Did NOT declare foo()
};

...

vector<shared_ptr<Base>> v;
for(auto ptr : v){
    // strange... why do some of these not work/default?
    // Oh that's right! I didn't define foo() in Derived!
    ptr->foo(); 
}
```

Then, their program behaves strangely because they either didn't know they call the function from the `Derived` class, or they call the function and it doesn't behave like how they wanted to.

Scott Meyers suggests the following if you wanted a default implementation:

```c++
class Base{
    ...
    // pure virtual
    virtual void foo() = 0;
protected:
    // notice how this isn't virtual;
    // it's because we need this to be *invariant* behavior.
    void defaultFoo(){
        ... // does something
    }
};

class Derived : public Base{
    ...
    // define
    virtual void foo(){
        defaultFoo();
    }
};
```

That way, we will have the default implementation, but we will let the compiler remind us.

This is a bit more code bloat though.

# Alternative Idioms to `virtual`

## NVI (Non-virtual interface)
This design revolves around the idea that **virtual functions should be private**.

For example:

```c++
class Base{
public:
    int foo(){
        ... // do something before
        int val = doFoo();
        ... // do something after
        return val;
    }
private:
    virtual int doFoo(){
        ... // do something
    }
}
```

The idea is similar to `setUp()` and `tearDown()` before and after the virtual `doFoo()` workhorse.

You're setting up the context, and then analyzing it and giving a result, while using simple virtuals in the middle to do most of the work.

What does private virtual even mean? It means this:

- The private means you cannot call `Base::doFoo()` from even the `Derived` class. 
- The virtual means you can call `foo()`, which is `Base::foo()`, which can access `Base::doFoo()`.

What do these two points amount to? **Yup, you cannot change the base implementation of `foo()`, but you can override `doFoo()` in the `Derived` class.**

## Strategy Pattern (Function Ptrs)

This is quite a dramatic design decision: **passing pointers to the actual functions to call.**

```c++
class Base;

int defaultFoo(const Base&);

class Base {
public:
    typedef int (*Foo)(const Base&);
    explicit Base(Foo f = defaultFoo) : _foo(f) {}
    ...
    int foo(){
        return _foo(*this);
    }
private:
    Foo _foo;
};
```

Here, we have a default function to execute `foo()`, passed in via the ctor. We keep a "strategy" as a function pointer in which we'll utilize inside of `void foo()`.

### Strategy Pattern Improved (`std::function`)

We can (and should) also do something with **`std::function<>`**, which generalizes the idea of a function:

```c++
...
class Base {
public:
    typedef std::function<int (const Base&)> Foo;
    explicit GameCharacter(Foo f = defaultFoo) : _foo(f) {}
    ...
};
```

This time, we can use any generalization of a function:

```c++
int defaultFoo1(const Base&); // works
float defaultFoo2(const Base&); // implicitly casts; works
struct defaultFoo3{ // generalized function; also works
    int operator()(const Foo&) const;
};
class defaultFoo4{ // A member function of a class.
    float foo(const Foo&) const; // uses this function
};
...

defaultFoo4 df4;

Base b(defaultFoo1);
Base b(defaultFoo2);
Base b(defaultFoo3());
Base b(std::bind(&defaultFoo4::foo, df4, _1));
```

#### A brief explanation of what the last example does

Inside of any member function of a class, we see something like:

`{return signature} {function name} (arg1, arg2, ... )`

This is not technically all that's happening underneath the hood. What's happening is that it's also taking in an implicit `*this` to know what the member function actually is. It looks more like:

`{return signature} {function name} (*this, arg1, arg2, ... )`

So what we need to do, when we want to use a specific object(in this case, df4's `foo()`)'s functions, then we need to implicitly bind the first argument to the instance that's holding the member function. `std::bind(&defaultFoo4::foo, df4, _1)` binds df4 to the first argument, so that any further arguments will be offset; starting at index 2 instead.

### Strategy Pattern Improved (hierarchical virtual functions)

When we have polymorphism on `foo()`, it will consult vtables on the object containing `foo()`. Normally, that object **is the object we're performing `foo()` on. In strategy pattern, you could delegate the inheritance of foo entirely to its own class:**

```c++
class Base;

class Foo{
public:
    virtual int hidden_foo(const Base&) const; // this can be inherited!
private:
};

class Base{
public:
    Base(Foo* foo = &defaultFoo) : _foo(foo) {};
    int foo(){
        return _foo->hidden_foo(*this);
    }
private:
    Foo* _foo;
};
```

This works too.

### Pros/Flexibilities:

1. Being able to supply different `foo()` for same class.
2. `foo()`'s behavior may be changed during runtime.

### Cons/Limitations:

1. No access to non-public interface of `Base`.
    - This may lead people to weaken the encapsulation of `Base` to achieve implementation goals.

# Never redefine an inherited non-virtual

You don't want this to happen:

```c++
Derived d;
Base* b = &d;

// These 2 are not the same...?
b->foo(); 
d->foo();
```

How could that happen? Does polymorphism allow this? 

Well, yeah if you have `virtual`, or don't override `Base::foo()`. If you don't have virtual, then do not redefine the function, because it's meant to be an **invariant**.

# Never redefine a function's inherited default parameter value

Two things: **virtual functions are dynamically bound, while default parameters are statically bound**. This means that default parameters are defined before virtual lookups.

This means:

```c++
class Base{
public:
    virtual void foo(int x = 1){
        std::cout << "Base! " << x << std::endl;
    }
};

class Derived : public Base{
public:
    void foo(int x = 2){
        std::cout << "Derived! " << x << std::endl;
    }
};

Base b;
Derived d;
Base* bp = &b;
Base* dp = &d;

bp->foo(); // Base! 1
dp->foo(); // Derived! 1
```

So you're going to be very confused. Don't do it. One way to prevent this is to use NVI design, and have the private virtual function not accept any defaults.

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
