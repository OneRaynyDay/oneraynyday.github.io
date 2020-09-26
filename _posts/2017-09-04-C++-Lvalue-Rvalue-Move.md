---
published: true
title: C++ - Lvalue/Rvalue, and Move Semantics
category: dev
layout: default
---

# Table of Contents

* TOC
{:toc}

# Rvalue and Lvalue

### Simplified definition:

- **lvalue** - An object that occupies some address in memory.
	- most things are lvalues.
	
```c++
int i = 1;
int* p = &i; // address exists!
i = 2; // memory is modified!

class foo;
foo f; // also resides in memory!
```


- **rvalue** - Anything that's not an lvalue.

```c++
int x = 2;
int* p = &(x+2); // error! not a memory location!

i+2 = 4; // error! cannot assign to rvalue!

auto sum = [](int x, int y){return x + y;};

sum(3,4) = 2; // error! rvalue!

Foo f;
f = foo(); // foo() is an rvalue!
```

The "l" stands for "left hand side", and "r" is "right hand side".

## References

The classic reference type is denoted by `{type}&`, like `int& x = y`.

We **cannot assign a reference an rvalue!**

```c++
int x = 3;
int& y = x; // OK!
int& z = 2; // ERROR!
```

### Small (Confusing) Exception
**There is a small exception, which is constant references!**

```c++
int x = 3;
int& y = 2; // ERROR!
const int& z = 2; // OK!
```

Why does this work? during compile time, you declare something to be `const` allows the right hand side to exist as an lvalue temporarily, so then the reference can capture it properly.

## Functions Can Yield Lvalues

You would think that most functions just return by value, giving you all rvalues.

That's not true.

Consider the following:

```c++
int foo(){ return 3; }

int global = 3;
int& bar(){ return global; }
```

Here, `foo()` returns an rvalue. However, `bar()` returns an lvalue.

## Rvalue References

We know `int& x` as a reference. In C++11, we introduce the idea of an **rvalue reference**, denoted by `int&& x`. 

```c++
void foo(int& i){ ... }
void foo(int&& i){ ... }

int a = 1;
foo(a); // calls (int& i)
foo(5); // calls (int&& i)
```

### Small Aside: function overloading ambiguity

As we said above, you can have two different arguments. However, can we add this?

```c++
void foo(int i){ ... }
void foo(int& i){ ... }
void foo(int&& i){ ... }

int a = 1;
foo(a); // ERROR!
```

The answer is no. It's extremely ambiguous, because you can pass by value or pass by reference in general.

# Move Semantics

In CS32, I've learned the idea of a constructor and a copy constructor. However, there is a third type in c++11: **move constructor**.

A move constructor takes in an rvalue, instead of a const reference or a bunch of inputs:

```c++
class Foo{
public:
    ...
    Foo(const Foo&& rhs){ // move ctor
       ...
    }
}
```

This constructor takes in an rvalue, which is a temporary object residing on the registers(not in memory!)

So now that you've seen it, how do you use it?

## `std::move`

We use `std::move` to invoke the move constructor of the object. An example:

```c++
void bar(Foo&& foo){
    ...
}

bar(std::move(foo)); // works!
```

A general survey can be shown as:

```c++
bar_by_ref(foo); // no constructor calls
bar_by_val(foo); // calls copy constructor
bar_by_move(std::move(foo)); // calls move constructor
```

So as you can see the ranking of **speed is : reference > move > value.** Generally.

## So what can you `std::move` efficiently?

If you wanted to move an lvalue, you would likely have to use an RAII container that does this for you. Refer to the Essential C++ blog for RAII.

```c++
std::auto_ptr<Foo> foo(new Foo()); // auto_ptrs are deprecated btw
bar(std::move(foo)); // changed ownership. foo now is null.
```

If you wanted to move an rvalue, you're in luck! You can `std::move()` as much as you want.

In fact, in C++11 and above, the stl containers implemented `std::move` internally for rvalues(not sure about lvalues), and the performance went up by a noticeable margin.

If you've got a large object that you want to `std::move`, aka swap pointers with large memory members, then implement your own move constructor!

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
