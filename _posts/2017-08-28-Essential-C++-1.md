---
published: true
title: Essential C++ - C++ Basics
category: dev
layout: default
---

# Table of Contents

* TOC
{:toc}

# Enums, Consts, Inlines & Macros

## Pros and Cons

### Macros
(e.g. `#define C 1.5`)

+ *pro*: Does not allow for dereferencing.
+ *con*: Symbol table will not include macro "variables", no scope to macros as it is globally defined,  crazy behavior when not written properly.

**When should we use macros?** (Not often)

+ When we don't want something to be dereferenced.
+ Is a _very_ simple constant. (But even then, refer to **enums**)

**When should we _not_ use macros?** (Most of the time)

+ Trying to define a quick function:
```c++
#define MAX(a, b) f((a) > (b) ? (a) : (b))
```
    + If passing in `MAX(++a, b)`, `a` _could be incremented twice!_
    + A good alternative is an **inline function**: 

```c++
template <typename T>
inline void max(const T& a, const T& b){
    return f(a > b ? a : b);
}
```

+ If we ever want to inspect the _symbol name_.

---

### Consts

(e.g. `const int x = 3;`)
+ *pro*: Is in the symbol table, has scope.
+ *con*: Allows for dereferencing.

**When should we use consts?**

+ When we know that it won't be dereferenced ever
+ Nowadays is a pretty safe choice in general

**When should we not use constants?**

+ When we have a lot of _related_ variables. We should then use an enum.

---

### Enums

(e.g. enum { x = 3 };)
+ *pro*: Has the same properties as const, except doesn't allow dereferencing
+ *con*: None really?

**When should we use enums?**

+ All the time? I don't see why not. Maybe the syntax is a little clunkier than `const`'s.

---

# Use `Const`'s Often

Here are some ways to use them:

## `const` Variables

### C-Style

```c++
char *p; // non const ptr, data
const char *p; // non-const pointer, const data
char * const p; // const pointer, non-const data
const char * const p; // const pointer, data
```

Rule of thumb: 

+ `const` before the star modifies the type. 
+ `const` after the star modifies the pointer.

### C++97+ Style

```c++
typedef std::vector<int> v;

const v::iterator iter <= const pointer, non-const data

v::const_iterator iter <= non-const pointer, const data
```

Rule of thumb:

+ `const` iterator modifies the pointer.
+ `const_iterator` modifies the data.

#### Example

```c++
const Rational operator*(const Rational& lhs, const Rational& rhs);
```

Means constant Rational returned from the `*` operator. Why does the result need to be constant? Because we could avoid:

```c++
(a*b) = c;
```

## `const` Functions

`const` functions are useful for the purpose of overloading:

```c++
class Bar{
    const char& foo() const; //const function, returns const char&.
    
    char& foo(); //function, returns char&
};
```

`const` functions inside classes cannot change the class members. By creating a class `Foo f`, we can call the non-const version of `foo()`, meanwhile `const Foo f` uses the const `foo()`.

A way to allow for **reduced code duplication between const and non-const equivalents**, one can do the following:

```c++
class Bar{
private:
    char c;
public:
    const char& foo() const{
        return c;
    }
    char& foo(){
        /* Reuse code from foo() const: */
        // removes const'ness
        return const_cast<char&>( 
            // calls foo() const
            static_cast<const Bar&>(*this).foo() 
        );
    }
};
```

## Physical/Bitwise `const` vs. Logical `const`

**Physical**:

+ declared constant if the member functions do not modify any members.

**Logical(Recommended)**:

+ allowed to change elements, but must be invisible to the client. `const`ness is only perceived by the user.
+ uses keyword `mutable` in front of variables, like `mutable int x;` to allow changeable class members even when `const` class instance.

# Initialize Before You Use

For **non-member, built-in types**, manually initialize:

```
int x = 0; // manual
char* cs = "hello world!"; // manual
```

For **anything else, the onus is on the constructor**. 

### Initialization vs. Assignments

```
/* Example of ASSIGNMENTS */
Foo::Foo(int x, char* s){
    // Assigned DURING constructor
    x_ = x;
    s_ = s;
}
```
Here, it's using the assignment operator `=`, to call the copy constructor. It calls this function: `Foo& operator=(const Foo& rhs) {  ... ; return *this; }`

```
/* Example of INITIALIZATION */
Foo::Foo(int x, char* s) : 
x_(x), s_(s)
{
    // Initialization BEFORE constructor
}
```

Here, it's directly passing in x and s as arguments of the copy constructors, which is this: `Foo::Foo(const Foo& rhs) : ... { ... }`

The crucial difference here is that **assignments call the constructor of those objects first, and then we re-assign. This is a waste of resources compared to initialization with the arguments.**

In addition, `const` variables cannot be **assigned**. They must be **initialized**, forcing them to have a value inside the initializer list.

## Issue with `static` & Singleton Design

If you have two files compiled in 2 different `.so` files like:

```c++
//f1.cpp
class Foo{
...
}

extern Foo foo; // allows any other cpp file to use it. Static non-local.
```

```c++
//f2.cpp
class Bar{
int x = foo.num();
...
}

Bar bar(); // uses foo.num()! Static non-local.
```

We have an issue here. We don't know which order these objects are loaded.

Thus, we should implicitly define an order. We obviously want foo to **exist** when we call `foo.num()`, so we should define it in a static function:

```c++
//f1.cpp
Foo& foo(){ // Remind you of singleton by any chance?
    static Foo foo;
    return foo;
}
```

```c++
//f2.cpp
...
int x = foo().num();
```
This way, calling the function will be first, then we will be forced to create `foo` before `bar` is initialized statically.

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
