---
published: true
title: Essential C++ - Design
category: dev
layout: default
---


# Table of Contents

* TOC
{:toc}

# Easy Correct, Hard Incorrect

Seems easy, until you try it.

Consider the class `Date`:

```c++
class Date{
...
public:
    Date(int month, int day, int year);
};
```

This requires the user to look up the docs, which not 100% do. Furthermore, what if the user constructs something weird like `Date date(-1,40,300);`?

## Make Errors Occur Early & Obviously

It makes life easier for the user if he was given a clear type error, like:

```c++
class Date{
...
public:
    Date(Month m, Day d, Year y);
};
```

This requires the user to be fully aware that he's inputting a `Year` where a year should be.

## Restrict User Input Space

Furthermore, we can do something like this for `Month`:

```c++
class Month{
public:
    static Month Jan(){ return Month(1); }
    ...
private:
    Month(int month){ ... }
};
```

The `static` keyword allows us to return the same statically allocated object. Locally static variables are created the moment the function is called, remember?

Another thing we can do is use `const`, which should be pretty obvious by now. It prevents things like:

```c++
const val& operator*(const val& lhs, const val& rhs) { 
    ... 
}

// this is not allowed! (and rightfully so)
(a*b = c)
```

# Class Design is Type Design

It's ideal if the class design behaves the same way as a built-in type.

This is just a list of questions to ask myself:

- How to **create/delete the new type**?
- How should **initialization differ from assignment**?
	- should note that `operator=` is not a ctor.
- What does it mean to **pass by value**?
- What are the **restrictions on legal values**?
- Does **inheritance fit nicely here**?
- What **type conversions are allowed**?
- Choose which **operator overloads/functions** make sense.
- Which standard functions should be `private`?
	- or in general, any function? Member?
- How general is the new type? Is it actually a **class template**?
- **Do I really need this type**, or is some amalgamation of built-in types good enough?

# ALWAYS Prefer pass-by-`const&` to pass-by-value

## Inefficiency

It's extremely inefficient to copy by-value a large object, as it is likely to have many stl containers that also need to be recursively copied.

For example:

```c++
class Banker{
    ...
    std::vector<Client> clients;
};

Banker banker;
pass_value(banker);
```

This copies over banker, but also recursively copies over clients! What if the banker is good at his job? Then this function will be very slow.

## Subtle Incorrectness

This occurs during polymorphic functions:

```c++
class Base{
    ...
}

class Derived : Base{
    ...
}

void foo(Base b){
    ...
}

Derived derived;
foo(derived); // Slices the Derived portion off!
```

This will pass-by-copy, meaning it will call the constructor for `Base`, rather than `Derived` in this situation. The issue here is that **only the `Base` class will be created, thus any overloaded functions(thus useful applications) of inherited classes are rendered moot**.

The way to fix this is using a `const Base& b`:

```c++
class Base{
    ...
}

class Derived : Base{
    ...
}

void foo(const Base& b){
    ...
}

Derived derived;
foo(derived); // Totally OK!
```

Why does this work? **Because references are implemented as pointers underneath.** *You always pass by pointer when you want to support polymorphic behavior. OK?*

# DON'T Force `const&` when should return-by-value

Sometimes, `const&` just doesn't work. Consider the following `operator*` overload:

### Try 1 - Stack references

```c++
const Foo& operator*(const Foo& lhs, const Foo& rhs){
    Foo foo;
    return *foo;
}
```
Doesn't work. You just allocated something and returned the reference when the object just free'd itself. You are now looking at undefined behavior.

### Try 2 - Heap references

```c++
const Foo& operator*(const Foo& lhs, const Foo& rhs){
    Foo *foo = new Foo();
    return *foo;
}
```
Works... If you don't consider chaining statements. Who's going to delete this either? Who has the chance to either in this situation:

```c++
// captures the final result...
// But what happens to the `new` f1*f2?
Foo& foo = ((f1*f2)*f3); 
```
Doesn't work. Memory leak!

### Try 3 - Static references

```c++
const Foo& operator*(const Foo& lhs, const Foo& rhs){
	static Foo foo;
	foo = ...;
	return *foo;
}
```

This will return something that will be deleted at the very end of the program, and so there's no real memory leak. However, this could occur:

```c++
if((a*b) == (c*d)) // ALWAYS TRUE!
    ... // do something
```

Why always `true`? **Because both `(a*b)` and `(c*d)` return the SAME static reference**! Of course it would be equal - they're the same pointer.

# `private` your Data Members

## Consistency

The user shouldn't remember whether to get a result via `foo.val` or `foo.val()`. Because sometimes functions to retrieve processed values is inevitable, we can only move from members to functions.

## Granular Access

You can choose read/write access, or both with functions, like so:

```c++
class Foo{
   void setX(int _x){ x = _x; } // write access
   int getX(){ return x; } // read access
private:
   int x;
};
```

Can't do that with public data members.

## Encapsulation

You can imagine that we shouldn't store all the values if we can derive it somehow. This is important for low memory devices(embedded).

```c++
class Foo{
   int getY(){ return x*2; } // we don't have a y member!
private:
   int x;
};
```

Plus, **we can make sure that interfaces are not broken by the change of internals.** For this same reason, we do not want to use `protected` ever.

# Prefer non-member non-friend functions to member functions

Pretty much, if you can move a member function out as a non-member non-friend function, **you are moving a convenience function out of the core functionalities**. This is good, because there should be as little interface to the data from the core functionalities as possible. 

## Task Splitting

Further compositions can be just a list of convenience functions. **Convenience functions can be moved to their own `.cpp` and `.h` files**.

```c++
// file1.h
class Foo{
public:
    ...
    int getA(){...}
    int getB(){...}
    int getAplusB(){...} // a convenience function.
private:
    int a,b;
};
```

This will lead to monolithic `.cpp` files. In retrospect, you could've done:

```c++
// file1.h
class Foo{
public:
    ...
    int getA(){...}
    int getB(){...}
private:
    int a,b;
};

// file2.h
int getAplusB(const Foo& foo){...} // a convenience function.
```

## Operator Overloads

When you overload the member function:

```c++
class Foo{
public:
    ...
    const Foo operator+(const Foo& rhs){
        ...
    };
private:
    int a,b;
};
```

**This just doesn't work with implicit type conversions. This requires the lhs expression to be a foo as well.**

If you wanted it to be a numeric type, then this is terrible. consider: `3 + foo`. This is `3.operator+(foo)`. That's not what we wanted :(

So the natural fix is to move it as a non-member **non-friend** function. Non-friend because a friend function will decrease the encapsulation.

# Consider non-throwing `std::swap` overload


## PImpl

Sometimes, `std::swap` is slow and unnecessary. Under the hood, it looks like the most simple impl:

```c++
namespace std{
	template<typename T>
	void swap(T& lhs, T& rhs){
		T temp(lhs); // SLOW!
		lhs = rhs;
		rhs = temp;
	};
}
```

Now, we can introduce an idea of a **pimpl**. This idiom allows us to add a level of indirection by pointing our resources in a heap allocated ptr.

This also comes with the added benefit of not having to initialize a new object:

```c++
class Foo{
public:
    ...
private:
    class Impl;
    // everything in here!
    std::unique_ptr<Impl> pimpl; 
};
```

When we swap:

```c++
class Foo{
public:
    ...
private:
    class Impl;
    // everything in here!
    std::unique_ptr<Impl> pimpl; 
    // This MUST be here, since pimpl is private.
    // We are also specializing.
    friend std::swap<>(Foo& lhs, Foo& rhs);
};

... 

namespace std{
	template <> // template specialization syntax
	void swap<Foo>(Foo& lhs, Foo& rhs){
	    // FAST! Swapping a pointer is almost free
	    std::swap(lhs.pimpl, rhs.pimpl);
	};
}
```

## Best Practices for Writing Swap

### Non-templated Class

As we just saw above, the swap function needed to be `friend`'d. That's not good, because **it decreases the encapsulation of the class `Foo`.** So how do we alleviate this?

We can define our own member `swap()` function inside of Foo, and direct the specialized `std::swap` function to use that interface!

```c++
class Foo{
public:
    ...
private:
    class Impl;
    // everything in here!
    std::unique_ptr<Impl> pimpl; 
    void swap(Foo& rhs){
        using std::swap;
        swap(pimpl, rhs.pimpl);
    }
};

... 

namespace std{
    template <> // template specialization syntax
    void swap<Foo>(Foo& lhs, Foo& rhs){
        // FAST! Swapping a pointer is almost free
        lhs.swap(rhs);
    };
}
```

### Templated Class

Suppose `Foo`'s declaration is now:

```c++
template <typename T>
class Foo{
    ...
};
```

Then how do you exactly specialize this? If we just replace `swap<Foo>` with `swap<Foo<T>>`, it won't compile. **This is a partial template specialization, which is not allowed for functions.**

Why are partial template specializations not allowed? Because you can achieve the same thing by **function overloading**:

```c++
// WILL NOT WORK
namespace std{
    // This is a specialization
    template <typename T> 
    void swap<Foo<T>>(Foo& lhs, Foo& rhs){
        lhs.swap(rhs);
    };
}

// WILL WORK (BUT ACTUALLY DOESNT)
namespace std{
    // This is an overload
    template <typename T> 
    void swap(Foo<T>& lhs, Foo<T>& rhs){
        lhs.swap(rhs);
    };
}
```

***MMM... But it doesn't actually work...*** You see, **std namespace does NOT allow for new functions/classes/definitions**. Specializations are OK, but new functions to overload are not. So what can we do?


```c++
// WILL WORK(BUT UGLY)
namespace foo_namespace{
    class Foo { ... };
    
    // This is an overload
    template <typename T> 
    void swap(Foo<T>& lhs, Foo<T>& rhs){
        lhs.swap(rhs);
    };
}

```

Is the best thing we can do. It's ugly, but refer to next section on how to use it effectively.

## Best Practices for Using Swap

The specialization & overloading kind of forces you to keep track of the 3 functions you could face:

- The `std::swap` default
- The specialized `std::swap`
- The overloaded `foo_namespace::swap`

You don't know which one's going to be used, so what are you gonna do?

```c++
{
    // calling swap() w/o overload will give us this.
    using std::swap; 
    ...
    // all 3 functions are taken into account.
    swap(foo1, foo2);
    ...
}
```

Is the cleanest way.

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
