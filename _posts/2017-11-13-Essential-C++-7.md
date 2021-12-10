---
published: true
title: Essential C++ - Templates and Generic Programming
category: dev
layout: default
---

Before we start - why did C++ create templates?

Templates were created because we needed containers that could safely contain
a multitude of different objects.

Now, we realize that templates are turing-complete, and that it can be used for
a whole new type of programming called **template meta-programming**,
which basically means a program that runs during compilation and finishes when
compilation ends.

Containers are just a small part of templates now. There are some nice uses like
`for_each`, `find`, and `merge`.

# Table of Contents

* TOC
{:toc}

# Templates are: implicit interfaces and compile-time polymorphism

In contrast, **inheritance is explicit interfaces and run-time polymorphism**.

Why is inheritance **explicit** while templates are **implicit**?

Suppose we have the following:

```c++
template <typename T>
void doSomething(T& w){
    if(w.size() > 10 && w != global_foo){
        T temp(w);
        temp.normalize();
        temp.swap(w);
    }
}
```

Well, we're not saying much about the specific type T. However,
we are implicitly defining its interface because we need these following
operations:

1. `size()` function.
2. `!=` comparison operator with `global_foo`.
3. `T(const T& t)` copy constructor.
4. `normalize()` function.
5. `swap(T& t)` function.

This is the **implicit** interface because we're not specifying anything
beforehand. Templates will deduce it for us.

Some people like to call it "duck typing".

To make sure these calls succeed, we will create templates during compilation.
The templates then create multiple copies of this function, allowing different
types to be inserted. This is **compile time polymorphism**.

However, **the requirements on these functions are lax.** 
You don't need `global_foo` to be of type T, if
`operator!=(const T& lhs, const U& rhs)` exists. Similarly, if `global_foo` is of type `V`
and can be implicitly casted into `U`.

# What is `typename`?

```c++
template <typename T> // any difference?
template <class T>
```

There is actually no difference above. This is a legacy feature added by Bjarne Stroustrup.

These two are exactly the same, but Scott Meyers suggests `typename` to suggest
it doesn't have to be a custom class name.

However, it is used specifically in 1 obscure situation:

```c++
template <typename C>
void foo(const C& c){
    C::const_iterator * x;
}
```

Does the above mean we have a **static member in C, and we are multiplying with x**,
or does it mean that we are **declaring a `const_iterator` pointer called x?**

According to C++ standards, **any nested dependent name is assumed a variable UNLESS
it's in a list of base classes(for inheritance) or a base class identifier in initializer lists**. 
`C::const_iterator` is a nested dependent name because it depends on the template type C.

If it's assumed a variable then we can't declare a pointer here! We would be multiplying.

To fix it, we use `typename`:

```c++
template <typename C>
void foo(const C& c){
    typename C::const_iterator * x; // fixed!
}

template <typename T>
// List of base classes:
class Derived : public Base<T>::Nested{ // typename NOT allowed here.
public:
    // initializer list
    explicit Derived(int x) : Base<T>::Nested(x){} // typename NOT allowed here.
};
```

A real life common use of this is to get a value pointed to by some iterator. This is
called `std::iterator_traits`. For a specific type of iterator, we would need:

```c++
// Creates an instance of whatever type IterT points to.
typename std::iterator_traits<IterT>::value_type val;
```

We don't want to type this, so we can typedef it:

```c++
// ugly but we're talking C++ here.
typedef typename std::iterator_traits<IterT>::value_type value_type;
value_type val;
```

# Deriving Templatized Classes

There's a big issue about this: **You cannot call base class methods in which the base class is a template!**

Why? The core of the problem is **template specialization**:

Suppose we have a class:

```c++
template <typename T>
class Foo{
    ...
    void bar(T t){ ... }
}

template <typename T>
class Bar : public Foo<T>{
    ...
    void far(T t){
        ...
        bar(t); // can't find it!
        ...
    }
}
```

Why can't we find it? It's clearly inherited, right?

The reason is because we can have **template specialization**:

```c++
template <>
class Foo<SpecialObj>{
    ... // we don't specify bar() here!
}
```

What if we were to instantiate `Bar<SpecialObj>`? Where would the `bar(SpecialObj t)` function go?

That's right! It was never defined! Our specialization really fucked us here.

The compiler may be too fearful, so we can subvert its warning by doing this:

```c++
template <typename T>
class Bar : public Foo<T>{
    ...
    void far(T t){
        ...
        this->bar(t); // can find it! We are looking @ the vmt here.
        ...
    }
}
```

or this:

```c++
template <typename T>
class Bar : public Foo<T>{
    ...
    using Foo<T>::bar;
    void far(T t){
        ...
        bar(t); // Telling the compiler that it does exist. (We could be wrong)
        ...
    }
}
```

# Avoid Code Bloat via Refactoring Template Functions

This is actually pretty cool, because it resembles Eigen's syntax:

```c++
template<typename T, std::size n>
class SquareMatrix{
public:
    ...
    void invert();
};
```

This seems all nice and good, but what if we did:

```c++
SquareMatrix<double, 1> m1;
SquareMatrix<double, 2> m2;
SquareMatrix<double, 3> m3;
SquareMatrix<double, 4> m4;
...
m1.invert(); // we need a specific invert for this
m2.invert(); // and this...
...
```

Wow, so for each `n`, we need the same function signature being repeated? That is a lot of code bloat!

We can refactor it like:

```c++
template<typename T>
class SquareMatrixBase{
protected:
    ...
    void invert(std::size_t mSize);
}

template<typename T, std::size n>
class SquareMatrix : private SquareMatrixBase<T>{
public:
    ...
    void invert(){ invert(n); }
};
```

We would be making a ton of copies here, yes, but each copy is like, a single line of code! That's perfect for us.

*Note:* Notice how we did `private` inheritance. This is because we don't want to expose any internal details about the base, and this isn't an `is-a` paradigm, more like `is-implemented-in-terms-of`.

Seems good so far, except the `SquareMatrixBase` base class doesn't know what data to operate on.

We can either pass in the matrix object(1), or we can bind the object to the base class's ptr(2):

```c++
template<typename T>
class SquareMatrixBase{
protected:
    ...
    void invert(std::size_t mSize, T *pData); // (1) pass in
}
```

```c++
template<typename T>
class SquareMatrixBase{
protected:
    ...
    void invert(std::size_t mSize);
    void setDataPtr(T* ptr){ pData = ptr; } // (2) bind the object
private:
    T *pData;
}
```

**Very important**: Make sure `invert` is not inline though! If it's inline, then all of our work has gone to waste, since it will be copied into wherever it's called and we'll get humongous code bloat.

## Pros for Code Bloat

Will run faster(compile-time constants can be propagated through code, machine code uses literals)

## Cons for Code Bloat

It's code bloat.

# Use Member Function Templates to Accept "all implicit compatible types"

For example, smart pointers should allow explicit conversion, like in polymorphism:

```c++
class Base{
    ...
}
class Derived : public Base{
    ...
}

Derived* d = new Derived();
Base* b = d; // explicit conversion here!

DumbPtr<Derived> d(new Derived());
DumbPtr<Base> b = d; // this won't work if we're dumb!
```

By default, there's no clear conversion between `DumbPtr<Base>` and `DumbPtr<Derived>`.

We should add it. This is how `std::shared_ptr` does it:

```c++
template<class Y>
shared_ptr(const shared_ptr<Y>& otherPtr) 
: myPtr(otherPtr.get()) {} // generalized copy constructor
```

This is nice and all, but beware - **if you're generating generic fns for copy constructors or assignment operations, you're not going to automatically make one for the default.**

Therefore, if you want something special to happen for your copy constructor(non templatized), then you need to define them too:

```c++
shared_ptr(const shared_ptr& otherPtr);
template<class Y>
shared_ptr(const shared_ptr<Y>& otherPtr) 
: myPtr(otherPtr.get()) {} // generalized copy constructor
```

# Define Non-member Functions Inside Templates When We Want Type Conversions

If we wanted to multiply 2 `Rational<T>`'s together, how do we define the `operator*`?

We don't want it to be inside of the object. We want it to be a function outside:

```c++
// outside of class definition.
const Rational<T> operator*(const Rational<T>&, const Rational<T>&);
...

Rational<int> half(1,2);
Rational<int> result = half * 2; // error! won't work!
```

This works if we don't have templates, but it doesn't work because of the `<T>`.

Why does it not work? **Because the compiler's trying to figure out which `T` it should be using.**

One thing that does **not** occur during template argument deduction is that they **do not consider implicit type conversions.**

Before you implicitly cast `int` to `Rational`, you need to actually **create** it. 
And creating it, thus is impossible.

One thing we can do is **declare the template function when we instantiate a `Rational<int>`**. To do this, we will write:

```c++
template <typename T>
class Rational{
...
    friend Rational operator*(const Rational& lhs, const Rational& rhs);
    /**
     * Can also be declared as:
     * friend Rational<T> operator*(const Rational<T>& lhs, const Rational<T>& rhs);
     * The two are equivalent.
    **/
};

const Rational<T> operator*(const Rational<T>&, const Rational<T>&);
Rational<int> half(1,2); // we will generate the code for friend function!
Rational<int> result = half * 2; // We will use the friend function.
```

However, the issue with the above is that it **won't link**. Why will it not link?

Because we only **declared** the function, and we didn't **define** it. Defining it outside the
function will not give us the solution because we declare a bunch of functions inside `Rational`
but there's no guarantee at all that the template friend function will be generated if it's
outside of our class.

One solution is just make it inline:

```c++
template <typename T>
class Rational{
...
    friend Rational operator*(const Rational& lhs, const Rational& rhs){
        ... // long definition
    }
};
```

And another solution is to call a helper:

```c++
template <typename T>
class Rational{
...
    friend Rational operator*(const Rational& lhs, const Rational& rhs){
        return doMultiply(lhs, rhs); // short definition
    }
};

template <typename T>
Rational<T> doMultiply(const Rational& lhs, const Rational& rhs){
    ... // long definition
}
```

The latter helps with inline code bloat.

# Type Traits

Here's a case study:

```c++
template <typename IterT, typename DistT>
void advance(IterT& iter, DistT d); // moves iterator d steps forward.
```

There are 5 STL iterator types (in increasing power):

1. **Input iterators** only move forward, one step at a time, reads only once.
    - e.x. `istream_iterator`
2. **Output iterators** only move forward, one step at a time, can only write once.
    - e.x. `ostream_iterator`
3. **Forward iterators** only move forward, one step at a time, can read/write more than once.
4. **Bidirectional iterators** move forward and backwards, one step at a time, and can read/write more than once.
5. **Random access iterators** can move multiple steps at once, and does everything #4 can do.

We want to write something like this when implementing `advance`:

```c++
template <typename IterT, typename DistT>
void advance(IterT& iter, DistT d){
    if(**iter is a random access**)
        iter += d
    else{
        if(d >= 0){while (d--) ++iter;}
        else{ while(d++) --iter; }
    }
}
```

Traits is a way to do this, so we can get information **about the type during compilation**.

```c++
template <typename IterT>
struct iterator_traits;
```

Inside of classes, we would stick a `typedef` of `iterator_category`:

```c++
struct random_access_iterator_tag : public bidirectional_iterator_tag {};

template <...>
class deque{
public:
    class iterator{
    public:
        // Now we will define iterator_category
        typedef random_access_iterator_tag iterator_category;
    };
};
```

And the `iterator_traits` will just refer back to the typedef:

```c++
template <typename IterT>
struct iterator_traits{
    // Remember, typename is needed because
    // We assume iterator_category is a member variable of IterT.
    typedef typename IterT::iterator_category iterator_category;
};
```

This works well for iterators that are **NOT** pointers. In this case, we need to do a **partial template specialization.**

A partial template specialization is specifying only a part of the qualifier of the type:

```c++
template <typename T>
struct iterator_traits<T*> //built in pointer types
{
    typedef random_access_iterator_tag iterator_category;
    ...
};
```

Now to go back to our original example with the `if` statement...

We don't actually want an if statement, per se. All of this information
is determined during compile time. After compile time, we should theoretically
have what we want, rather than having if-statements check something that's
already determined.

For this, we use an old friend called **function overloading**:

```c++
template <typename IterT, typename DistT>
void advance(IterT& iter, DistT d){
    doAdvance(iter, d,
        typename std::iterator_traits<IterT>::iterator_category()
    );
}

template <typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::bidirectional_iterator_tag){
    if(d >= 0){while (d--) ++iter;}
    else{ while(d++) --iter; }
}

... and many other overloads of doAdvance().
```

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
