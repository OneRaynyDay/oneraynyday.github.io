---
published: true
title: Essential C++ - Objected Oriented Programming
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

