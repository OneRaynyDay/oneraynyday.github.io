---
published: true
title: Essential C++ - Constructors, Destructors, Assignments
category: dev
layout: default
---

# Table of Contents

* TOC
{:toc}

# Default Implementations

C++ fills in default implementations for you if you don't write anything. For example, an empty class:

```c++
class Foo{};
``` 

Will be equivalent to:

```c++
class Foo{
public:
    Foo(){}; // Calls each member's default constructor
    Foo(const Foo& f){...}; // Calls each member's copy constructors
    Foo& operator=(const Foo& rhs){...}; // Calls each member's = operator
};
```

## Limitations

C++ has these default implementations but sometimes it's just impossible. For example:

```c++
class Foo{

    std::string& s; // cannot change reference
    const int x; // cannot change const
};
```

If we wanted to call the assignment operator of `Foo`, as in `f2 = f1;`, it would be illegal. There would be a compiler error in this case. 

# Disallow Unnecessary Functions

## Uncopyable

Achieve linker error upon copy using:

```c++
class Foo{

private:
    Foo(const Foo&); // we don't need a var name
    Foo& operator=(const Foo&); // this neither
};
```

## Uninitializable (Singleton)

Privatize the constructor too:

```c++
class Foo{
public:
    // 'static' means you don't need
    // an instance to call this getFoo()
    static Foo& getFoo(){
       // static variable. Contrary to
       // intuition, this only gets initialized
       // once ever. Any subsequent calls will
       // NOT create any new instances.
        static Foo foo;
        return foo;
    };
    
    Foo(const Foo&)               = delete;
    void operator=(const Foo&)  = delete;

    // Note: Scott Meyers mentions in his Effective Modern
    //       C++ book, that deleted functions should generally
    //       be public as it results in better error messages
    //       due to the compilers behavior to check accessibility
    //       before deleted status

private:
    Foo(){}
    Foo(const Foo&);
    Foo& operator=(const Foo&);
};
```

# Declare Destructors `virtual` in Base Class

If you have a base class, `Base`, and a derived class, `Derived`, and you use polymorphism like:

```c++
Base* derived = new Derived(); // polymorphic pointer

delete derived; // UH OH!
```

If you don't have `virtual`, then the vtable will not be initialized. This means `Base` portion of `derived` will be deleted, but there will be UB in general for whatever is left. Memory leaks ensue.

```c++
class Base{
public:
    ...
    virtual ~Base(); // Important virtual!
};
```
## Common screw-up

Sometimes, people try to derive the stl library in C++:

```c++
// DON'T DO THIS
class DerivedString : public std::string{
...
};
```

stl library is not implemented with inheritance in mind, so don't do this. In case you have polymorphic designs, you will only be able to delete part of the object via deleting the `std::string` portion.

## Aside: `public/protected/private` inheritance

From a very intuitive stack overflow answer:

```c++
class A 
{
public:
    int x;
protected:
    int y;
private:
    int z;
};

class B : public A
{
    // x is public
    // y is protected
    // z is not accessible from B
};

class C : protected A
{
    // x is protected
    // y is protected
    // z is not accessible from C
};

class D : private A    // 'private' is default for classes
{
    // x is private
    // y is private
    // z is not accessible from D
};
```

# How to Handle Exceptions in Destructor

Usually, you shouldn't be getting one in the first place. Having exceptions inside destructors is really dangerous because:

```c++
{
std::vector<Foo> v; // suppose Foo has a destructor that throws
} // scope leaves, destructor called on all Foo's
// ...
// BAM, 10 exceptions at once, undefined behavior!
```

C++ apparently can **only handle one active exception at once.** So how should we handle this?

1. Try your best to NOT throw inside a destructor.
2. If all else fails, and it's appropriate, then delegate the task to a `close()` function:

```c++
{
Foo foo; // suppose Foo has a destructor that throws
...
try{
    foo.close(); // close it ourselves
} catch (std::exception& e){
    std::cout << "Error : " << e.what() << std::endl;
}
}
```

Now, if **the user doesn't respect this `close()` function, then they're asking for us to kill their program or absorb the error.** It wouldn't be surprising if they didn't listen:

```c++
Foo::~Foo(){
    ...
    catch (bad){
        log.write("Error occured. Aborting.");
        std::abort(); // kill it with fire
    }
}
```

# Don't Call `virtual` Functions in Constructors/Destructors

When you call `virtual` inside of a constructor, hoping it would reference the derived class's implementation, you **are making a huge mistake**. 

Why? Because when you call a constructor, you first begin by creating the base class. **Before the base class is fully constructed, you cannot possibly have a notion of derived class.**

Calling `virtual` will refer to the base class's implementation, which is NOT what you want! 

```c++
class Base{
public:
    Base(){
        foo(); // Error - virtual!
    }
    virtual void foo() = 0;
}
```

Sometimes, a compiler error won't even show up due to indirection:

```c++
class Base{
public:
    Base(){
        bar(); // No error - not a virtual!
    }
    void bar(){
        foo();
    };
    virtual void foo() = 0;
}
```

Which is **extremely tricky**.

## A Meh Workaround

You can call the base class's `explicit` function using a helper function to help shape your input first:

```c++
class Base{
public:
    Base(const std::string& s){
        log(s);
    };
private:
    void log(const std::string& s){
        ...
    };
};
```

# Assignment Operators return `*this`

For all operators, `+, -, /, *, +=, -=, =`, just return `*this` at the end of the function:

```c++
Foo& operator+=(const Foo& rhs){
   ...
   return *this; // do this or else won't compile sometimes!
}
```

# Handle `*this = *this`

Suppose we had a class:

```c++
class Foo{
public:
    ...
    Foo& operator=(const Foo& rhs){
        ...
        delete s; // deleting itself on both rhs and lhs :(
        s = new char[5];
    }
private:
    char* s;
};

Foo f1;
Foo* fp1 = &f1; // secretly the same object!
Foo* fp2 = &f1;

*fp1 = *fp2; // uh oh...
```

So prevent this by a simple pointer check:

```c++
if (&rhs == this) return *this;
```

# Copy Constructor - Base Class Mistake

You may think that:

```c++
class Derived : Base{
public:
    Derived(const Derived& rhs) : x(rhs.x){};
private:
    int x;
};
```

is sufficient for copying. **It's not.**

You copied all the parts of the `Derived` class, but none of the `Base`. You **called the default constructor of `Base` in this case**, and so you won't have any of its values. 

**Always initialize the base class in copy ctor:**

```c++
class Derived : Base{
public:
    Derived(const Derive& rhs) : Base(rhs), x(rhs.x){};
private:
    int x;
};
```

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
