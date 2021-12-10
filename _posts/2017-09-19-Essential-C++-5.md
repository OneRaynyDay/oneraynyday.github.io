---
published: true
title: Essential C++ - Implementation
category: dev
layout: default
---

# Table of Contents

* TOC
{:toc}

# Postpone Variable Definitions

There is a cost to doing:

```c++
// V1
void foo(std::string s1){
    std::string s2;
    ... // do some conditional checks on s1
    s2 = s1;
}
```

versus:

```c++
// V2
void foo(std::string s1){
    ... // do some conditional checks on s1
    std::string s2 = s1;
}
```

versus:

```c++
// V3
void foo(std::string s1){
    ... // do some conditional checks on s1
    std::string s2(s1);
}
```

- V1 initialized an object that may not be used depending on if the function returns early
- V2 invokes both the default ctor and the assignment operator.
- V3 is the best; it initializes on time, and only calls the copy ctor.

## Within Loops

We are faced with 2 alternatives:

```c++
// V1
{
    Widget w; // ctor
    for(int i=0; i<n; i++){
        w = f(i); // assignment
    }
} // dtor
```

versus:

```c++
// V2
{
    for(int i=0; i<n; i++){
        Widget w(f(i)); // copy ctor
    } // dtor
}
```

- If initializing takes little time, then V2 should be done.
- If assignment takes little time, then V1 should be done.

# Minimize Casting

There are 2 ways of **old-school casting**:

```c++
// C-style cast:
(T) val;

// Function-style cast:
T(val)
```

They are **equivalent in effect**.

There are 4 ways of **new-school C++ style casting**:

```c++
// Casts away the const-ness
const_cast<T>(val);

// Casts Base->Derived
dynamic_cast<Derived*>(base_ptr);

// Reinterprets the bytes completely
reinterpret_cast<T>(val);

// Reverse-cast in many situations;
// Also forces implicit conversions.
static_cast<Base*>(derived_ptr);
static_cast<const T>(t_val);
```

We should **strive to not cast. C++ is a language designed so that type errors are impossible. The only way to break that rule is casting.**

Suppose we have the following:

```c++
class Base { ... };
class Derived : public Base { ... };
Derived d;
Base *pb = &d; // implicit conversion of Derived* -> Base*

assert(pb == &d); // could fail!
```

Why does that fail? Because **in C++, a single object could have more than 1 address.** 
It depends on what portion you're looking at.

For this example, by implicit static casting, at runtime the pointer might add some offset. 
We don't know this offset, and so these pointers won't be the same.

## You think you need cast, but you don't

In inheritance, calling super functions(functions from Base in the scope of the Derived) should not be done via casting and then calling:

```c++
class Base {
public:
    // Could modify object somehow
    virtual void foo() { ... }
};

class Derived : public Base {
    void foo() {
        static_cast<Base>(*this).foo(); // Not good! Subtly fails!
    }
};
```

Why does this fail? **It's because Base-casting `this` will generate a new object.** So **avoid casting for this subtle reason!**

Instead of doing that, we can simply do:

```c++
class Base {
public:
    // Could modify object somehow
    virtual void foo() { ... }
};

class Derived : public Base {
    void foo() {
        Base::foo(); // works fine!
    }
};
```

## Casts are slow

Especially `dynamic_cast`. How dynamic casting works in some cases, it compares the class names using `strcmp`.

Obviously, if you have a hierarchy like:

```
C1 -> C2 -> C3 ... -> CN
```

where `C1 -> C2` means C2 inherits C1, then `dynamic_cast<CN>(c1_instance)` will cost you N operations of `strcmp`. 

That's bad, so try not to do it.

Sometimes, you do it to call a function `cn()` that doesn't exist in `C1, C2, ... CN-1`.

This makes sense for large inheritance chains, but if your chain is only:

`C1 -> C2 -> C3`, then you should **probably** pull the function down to C1 to access:

```c++
class C1{
public:
    virtual void c3(){} // no-op
};
class C2 : public C1{
public:
    virtual void c3(){} // no-op again
};
class C3 : public C2{
public:
    void c3(){
        // do something
    }
};
```

Another easy win, is if you have a vector of:

```c++
std::vector<std::shared_ptr<C1> > v;
```

And you actually have all of the members be `C3`'s. Then just make it a vector of:

```c++
std::vector<std::shared_ptr<C3> > v;
```

The above was kind of obvious, but sometimes mistakes are made.

Sometimes, you are cornered into a situation where you have to use `dynamic_cast`. It might be impossible to do without it, but if you can redesign so you don't have it, chances are the program will run much faster.

# Avoid returning handles

A handle is any object type that is a *way to access other objects*. This means `int&, int*, std::vector<T>::iterator` are all handles.

## Accidentally expose non-constness

You wanted an object to behave like "read-only" under `const`, so you write it like this:

```c++
class Foo{
    Foo() : bar(0) {};
    void setBar(int x){
        bar = x;
    }
    int& getBar() const{
        return bar;
    }
};
```

Looks okay... except `int&` is a handle! Here is some magic to display how constness can mysteriously disappear:

```c++
const Foo f; // bar = 0
f.getBar() = 10; // f didn't modify bar, but we did! bar = 10
```

So we guaranteed that getBar() can be accessible during `const`, and returned a value of `int&`. 

We can then access this value and change it, making `const` completely pointless.

So our solution should be:

```c++
class Foo{
    Foo() : bar(0) {};
    void setBar(int x){
        bar = x;
    }
    const int& getBar() const{
        return bar;
    }
};
```

And that makes everything okay... not.

What if someone did this:

```c++
const Foo makeFoo(){ ... }

const int* bar = &(makeFoo().getBar()); // what are we pointing to...?
```

We got a const Foo temporary, got the reference of its member, **and then it destructed before our `const int* bar` was even able to capture it.**

Even though this is totally the user's fault for hacking through this mess, we don't want that to happen at all.

So try not to return a handle. It'll make debugging easier.

# Strive for exception-safe

Exception-safe functions offer 1 of these guarantees:

1. **Basic guarantee** : If an exception is thrown, the program is in **a valid state**. We don't know which one though.
    - A valid state by definition is one which no objects are corrupted(i.e. modified halfway)
2. **Strong guarantee** : If an exception is thrown, the program is in **the valid state it was in before the function call**. This is also called **atomic**.
    - If this type of function is called, a failure == no-op.
3. **No-throw guarantee** : Promises to never throw exceptions.

## `throw(...)` vs noexcept

TL;DR: Use noexcept instead of throw() when you can, and use noexcept sparingly.

If you define a function with the following signature:

```c++
int doSomething() throw();
```

It **does not mean that doSomething() will not throw an exception**. It means if it does throw an exception, it will go straight into `std::unexpected()`, which is a global handler for when `throw()` is violated.

In side of `std::unexpected()`, **it will beat your program to death**, calling `terminate` if you don't do anything to intervene.

What about:

```c++
int doSomething() noexcept;
```

It makes things a little bit more direct; we have a couple differences between `noexcept` and `throw`:

#### `throw()`

1. When exception occurs, `std::unexpected` is called, which then calls `std::terminate`.
2. When exception occurs, stack unwinds completely.
3. `throw(...)` may contain arguments, saying `throw(A)` means `A` is permitted to throw.

#### `noexcept`

1. When exception occurs, `std::terminate` is called.
2. There's no specification on whether the stack needs to unwind, thus allowing optimizations.
3. `noexcept` is binary, meaning it can either `noexcept` the entire function, or not.

### Which one to choose?

We should always choose `noexcept`. This is because of 2 minor improvements:

1. Some stl containers have a `std::move_if_noexcept` that will be used.
2. It will be easier to spot bugs.
    - Having 2 exceptions at once is not allowed in C++. Having 2 exceptions happen at once in development is very unlikely. Having 1 exception is likely. It can be used to detect errors at an earlier stage, and thus it's easier to predict during development, instead of having it occur in prod.

## What can throw?

The answer is a lot of things, which means not many things can be `noexcept`. 

The list includes(but not restricted to):

1. All STL containers can throw `bad_alloc` if not enough memory.
2. `bad_cast` can occur during `dynamic_cast<...>`.
3. `bad_typeid` can occur during `typeid(...)` where `...` is invalid. 
4. `system_error` can occur during any system calls, i.e. detaching a nonthread, file system errors, etc.

## A strategy for strong exception guarantee - Copy & Swap

Copy & swap goes like this:

1. You create a clone
2. Fill the clone up with the next state's values
3. Use some strong exception guarantee function to swap your current members and the clone's members.

The hardest part about this process is #3, since swapping all members might be difficult to do with the strong exception guarantee.

I should also note that **steps 1 and 2 must not edit the global state in any way(including the current instantiation that the member funcs are a part of)**. If they do, then all bets are off for strong guarantees.

However, the **PImpl** idiom saves the day here:

```c++
class Foo{
    ...
    void updateFoo();
private:
    class Impl{
        ...
    };
    std::unique_ptr<Impl> pimpl;
};
```

Then a swap is simply:

```c++
void Foo::updateFoo(){
    // STEP 1
    using std::swap; // remember swap overloading?
    std::shared_ptr<Foo> temp = std::make_shared<Foo>();

    // STEP 2
    ... // do something with temp

    // STEP 3
    pimpl = temp; // assignment deletes old and adds refcount to temp.
}
```

# Inlining

**Inlining is the act of replacing function calls with their function bodies during compile time.** This could lead to reduced # of function calls thus faster performance. However, it could also lead to object code bloat, if the inlined functions are too large. **So inline with caution, and understand what it's doing**.

## Misconception: Inline vs. Templates

Some people think templates must be inline. That's not true.

### Templates

1. Templates are in header files because compilers need to know what a template looks like before instantiating. Templates can be inlined, but it's not necessary.
2. Templates are a command to the compiler. It's mandatory.

To elaborate on #1, suppose you have `x.h` and `x.cpp`. Compiling its object code where template functions are defined inside of the .cpp file will generate all the template types the compiler **sees**. The compiler **cannot see the template types needed for y.cpp**. Thus, it will cause a linker error.

### Inlines

1. Inlines are in header files because compilers need to know what it looks like before replacing function calls with the code.
2. Inlines are **a request** to the compiler. They can choose whether or not to inline.

## When does inlining fail/When not to inline?

### Function Pointers - fail

If you have a function pointer to the inlined function, it must reside in code section memory!

How else can we get an address...? So the inlining will be **partially applied**:

```c++
inline void f() {...}
void (*pf)() = f;
...
f(); // inlined!
pf(); // not inlined!
```

### Ctor/Dtors for inherited classes - don't

Suppose we have:

```c++
class Base{
...
};

class Derived : public Base{
public:
    Derived() {} // empty... right?
private:
    std::string s1, s2, s3,...;
};
```

It's actually more like this when the compiler's done with it:

```c++
class Base{
...
};

class Derived : public Base{
public:
    Derived() {
        // Create the base class
        Base::Base();
        // Initialize the members
        try{ s1.std::string::string(); }
        catch (...) {
            Base::~Base();
            throw;
        }

        try{ s2.std::string::string(); }
        catch (...) {
            s1.std::string::~string();
            Base::~Base();
            throw;
        }

        try{ s3.std::string::string(); }
        catch (...) {
            s1.std::string::~string();
            s2.std::string::~string();
            Base::~Base();
            throw;
        }

        ...
        
        try{ sN.std::string::string(); }
        catch (...) {
            s1.std::string::~string();
            s2.std::string::~string();
            ...
            sN-1.std::string::~string();
            Base::~Base();
            throw;
        }

        ...

    }
private:
    std::string s1, s2, s3,...;
};
```

NOTE: The above is only for illustration. This isn't exactly what happens.

Now if you try to inline this secretly huge function, you could end up with large code bloat. Especially if the member functions have inlined constructors/destructors too!

### As a library designer - warning!

If you want to change the header file, you must recompile everything.

If you want to change the cpp files, you just need to relink.

A user probably only wants to relink.

## Rule of thumb : 80/20 rule

Your program spends 80% of its execution time in 20% of its code. Inlining may not be a part of that 20%, so don't bother making things inline unless you need to.

Try to start with no inlining first, and then hand-apply your optimization after.

# Minimize compilation dependencies

Sometimes, when you change some private stuff about the implementation, compilation should be fast.

If you're not changing the interface, then you should be able to recompile easily... right?

Well no. In C++, it's hard to distinguish between implementation and interface.

Consider the following:

```c++
class Foo{
public:
    void f1(); // interfaces
    void f2();
    void f3();
private:
    std::string s1, s2, s3; // implementation details
};
```

In this example, you can't compile `Foo` in your translation unit without including implementation specific details, **which is bad**. This is the prime culprit that leads to long compilation times.

Suppose instead of `std::string` members, you had special members:

```c++
#include "Bar1.h"
#include "Bar2.h"
#include "Bar3.h"

...

class Foo{
public:
    Foo(Bar1, Bar2, Bar3);
    void f1(); // interfaces
    void f2();
    void f3();
private:
    Bar1 s1; // implementation details
    Bar2 s2;
    Bar3 s3; 
};
```

This creates a dependency, like `Bar1 -> Foo`, `Bar2 -> Foo`, `Bar3 -> Foo`.

This means if any `Bar{1,2,3}` changed their header files(hence recompile), then `Foo` must ALSO recompile. This could get **really bad**.

## Why not forward declaration?

Why can't you do something like this:

```c++
class Bar1; // forward declaration
class Bar2;
class Bar3;

class Foo{
public:
    Foo(Bar1, Bar2, Bar3);
    void f1(); // interfaces
    void f2();
    void f3();
};
```

... and ignore the implementation details along the way?

Sometimes, you **just can't**. For example, suppose `Bar1` was not actually a class name, but rather a typedef:

```c++
typedef Some_Other_Class Bar1;
...
class Bar1; // won't work!
...
class Foo{
...
};
```

Then it's not a valid class. Sometimes, you'll never be able to predict whether it's a typedef or not, so you shouldn't. In that case, you have to use a header file.

Another, more important issue with this is that **to instantiate an object, you need to consult its class definition, which must be already defined**:

```c++
class Bar1;

int main(){
    int x; // c++ knows it's 4 bytes.
    Bar1 b; // how many bytes is this? We don't know!
}
```

However, you can do this:

```c++
class Bar1;

int main(){
    int x; // c++ knows it's 4 bytes.
    Bar1* b; // how many bytes is this? It's 8 bytes on 64 bit!
}
```

Which leads us to the solution...

## Solution 1: PImpl

```c++
class FooImpl;
class Bar1; // assuming Bar{1,2,3} are classes
class Bar2; // ... and not typedefs
class Bar3;

class Foo{
public:
    Foo(Bar1, Bar2, Bar3);
    void f1(); // interfaces
    void f2();
    void f3();
private:
    std::shared_ptr<FooImpl> pimpl;
};
```

With this, clients know exactly how many bytes `Foo` is: it's the size of a `shared_ptr`, usually 16 bytes, and that should be it.

This, is a true separation of implementation and interface.

## Solution 2: Pure Abstract

Another solution is pure abstract classes, which are just interfaces in java/.NET. However, this idea of a pure abstract class is not enforced:

```c++
class Foo{
public:
    Foo(Bar1, Bar2, Bar3);
    virtual ~Foo();
    virtual void f1() = 0;
    virtual void f2() = 0;
    virtual void f3() = 0;
};
```

We cannot possibly instantiate a class of type `Foo`. Polymorphically, we can though, using pointers:

```c++
Foo f(b1, b2, b3); // ERROR!
std::shared_ptr<Foo> fp = std::make_shared<FooImpl>(b1, b2, b3); // OK!
```

This is a little inconvenient, but it does the job too.

## Some Guidelines

1. **Avoid using objects when object reference/ptrs will do**. You necessitate the existence of the definition of the object when you use the object.
2. **Depend on class declarations instead of definitions**. You can do something like this:

```c++
class Bar;
Bar getBar();
void setBar(Bar); // compiles!
```

Why does this compile? Because when someone calls `getBar()`, the definition of the class `Bar` must've already been loaded in. So it's okay if we do this.

3. **Provide separate header files for declaration/definitions**. For example, in the pimpl idiom, we should do something like this:

**Headers:**
```c++
// "Foo.h"
class FooImpl;
class Bar1; // assuming Bar{1,2,3} are classes
class Bar2; // ... and not typedefs
class Bar3;

class Foo{
public:
    Foo(Bar1, Bar2, Bar3);
    void f1(); // interfaces
    void f2();
    void f3();
private:
    std::shared_ptr<FooImpl> pimpl;
};
```

```c++
// "FooImpl.h"
class Bar1; // assuming Bar{1,2,3} are classes
class Bar2; // ... and not typedefs
class Bar3;

class FooImpl{
public:
    FooImpl(Bar1, Bar2, Bar3);
    void _f1();
    void _f2();
    void _f3();
private:
    Bar1 s1;
    Bar2 s2;
    Bar3 s3;
};
```

**Cpp:**
```c++
// "Foo.cpp"
Foo::Foo(...) : pimpl(std::make_shared<FooImpl>(b1, b2, b3)) {};
...
void Foo::f3(){
    pimpl->_f3();
}
void FooImpl::_f3(){
    b3.doSomething(); // do something
}
...
```

## The costs of decoupling

We should be well aware of the cons as well. **It's obvious that if we take less time in compilation, we should take more time during runtime.** It's very rare that you get free wins.

1. You need to perform dynamic allocation, which has a penalty.
2. You have an extra indirection for pimpl via pointer access.
3. You have an extra indirection for interface via vtable pointer.
4. You can't get much use out of inline functions, since definitions must be in the header.

So what should you do?

The **best possible answer is to use decoupling during development, and replace it with concrete classes with inlining for production.**

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
