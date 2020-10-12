# `const_cast`

`const_cast` is used to add or remove qualifiers such as `const` or `volatile` to an lvalue reference or a pointer type. Let's walk through the [standards](https://timsong-cpp.github.io/cppwp/n4140/expr.const.cast) and understand what each point means:

## Template argument `T` in `const_cast<T>(...)`

- `const_cast<T>(v)` will always return a value of type `T`.
- If `T` is an lvalue reference (i.e. `int&`), the result is an lvalue, likely the same object as `v` but reinterpreted to have different qualifiers (`reinterpret_cast` can't reinterpret qualifiers).
- If `T` is an rvalue reference (i.e. `int&&`), the result is an xvalue, which is an expiring value that may reside in memory.
- Otherwise (i.e. `int`, `int[2]` or `int*`), the result is a prvalue and the lvalue-to-rvalue (i.e. `v` is potentially copied), array-to-pointer (i.e. `int[2]` becomes `int*`), or function-to-pointer (i.e. we get a function pointer from `v`) are performed on `v`.

## Nestedness of pointers

The standard uses $T_1, T_2$ to denote two different pointer types. As we know, pointer types can be arbitrarily nested, i.e. `int***` is a triply nested pointer type. The standard formalizes this notion of qualifiers at every nested level with a sequence defined with $cv_{i,k}$ where $i$ is the $i$-th type, and $k$ is the nested level starting from 0. If we take $T_1 = $ `int***`, then $cv_{1,0}$ describes const-volatile qualifiers on `int***`, $cv_{1,1}$ describes `int**`, ... , $cv_{1,3}$ describes the qualifiers on `int`.

In `const_cast`'s, we only consider two types with the same nestedness, so that $T_1$ has $cv_{1,0...n}$ and $T_2$ has $cv_{2,0...n}$. If the types at every level are the same, we can perform a `const_cast` to change $cv_{1,0...n}$ to $cv_{2,0...n}$ respectively. 

Let's go through an example:

```c++
volatile int const * const volatile * * volatile * const x = nullptr;
const_cast<volatile int * * * volatile *>(x); // successful!

/**
cv1,0 = const (read it backwards)
cv1,1 = volatile
cv1,2 = none
cv1,3 = const volatile ... pointer of const volatile int
cv1,4 = const volatile int (we can put volatile and const on both sides)

cv2,0 = none
cv2,1 = volatile
cv2,2 = none
cv2,3 = none
cv2,4 = volatile
*/
```

... Note that this is **sufficient but not necessary in all cases**. If you only wanted to change the cv-qualifiers on $cv_{0,0} \to cv_{1,0}$, it is **unnecessary to use a `const_cast`**:

```c++
const int * const x = nullptr;
const int * y = x; // unnecessary to use const_cast!
int * z = const_cast<int *>(x); // necessary! cv1,1 and cv2,1 are different!
```

## Reference casts

If pointers of type $T_1$ and $T_2$ are `const_cast`'able to each other in the above situation, then we can also perform the following casts:

- lvalue reference casts between $T_1$ and $T_2$: `const_cast<T2&>(t1)`
- rvalue reference casts between $T_1$ and $T_2$: `const_cast<T2&&>(t1)`

## Casting away constness in general

In general, we may be performing a complicated cast with multiple steps, such as during a C-style cast. In this case, we can no longer assume that pointer types $T_1$ and $T_2$ have the same nested levels. We denote $T_1$ to have $N$ levels and $T_2$ to have $M$ levels below, and $K := min(N, M)$.

Casting $T_1 \to T_2$ **casts away constness** if the following expressions:

- $cv_{1,0}, cv_{1,1},..., cv_{1,K}$ ending at non-pointer type $T$
- $cv_{2,0}, cv_{2,1},...,cv_{2,K}$ ending at non-pointer type $T$

... are not implicitly convertible to each other. If we look at the previous example of `const int * const` vs. `const int *`, we can see that there exists an implicit conversion of [lvalue-to-rvalue](https://timsong-cpp.github.io/cppwp/n4140/conv#lval) which can be performed, which states:

> If T is a non-class type, the type of the prvalue is the cv-unqualified version of T. 

Since `int*` is a non-class type, `const int * const x` can be converted to a  prvalue of `const int*`. As a result, `y` did not need to perform an explicitly `const_cast`. Another type of implicit conversion is [qualification conversions](https://timsong-cpp.github.io/cppwp/n4140/conv#qual) which allows the user to make a prvalue of potentially nested pointers _more qualified_ than before. As a result, we can safely perform the following:

```c++
int **** x;
int const * const * const * const * const y = x; // qualification conversion is implicit!
```

An example of the above rule is the following:

```c++
volatile int* const (* * const x) [2] {}; // ptr -> ptr -> arr -> ptr -> int
reinterpret_cast<int * * const * * const>(x); // ptr -> ptr -> ptr -> ptr -> int

/**
cv1,0 = const
cv1,1 = none
cv1,2 = none (can't put qualifiers on arrays, only on its members)
cv1,3 = const
cv1,4 = volatile

cv2,0 = const
cv2,1 = none
cv2,2 = const
cv2,3 = none
cv2,4 = none
*/
```

The above example compiles because **arrays are not pointer types.** Therefore, $K = min(2, 4) = 2$, and we thus check whether $cv_{1,0...2} = cv_{2,0...2}$ elementwise. They are element-wise equal so therefore it's not considered casting away constness. However, if we were to change the above example to:

```c++
volatile int* const (* const * const x) [2] {}; // ptr -> ptr -> arr -> ptr -> int
reinterpret_cast<int * * const * * const>(x); // ptr -> ptr -> ptr -> ptr -> int

/**
cv1,0 = const
cv1,1 = const
cv1,2 = none (can't put qualifiers on arrays, only on its members)
cv1,3 = const
cv1,4 = volatile

cv2,0 = const
cv2,1 = none
cv2,2 = const
cv2,3 = none
cv2,4 = none
*/
```

Then we do have a compiler error! Specifically, we see that $cv_{1,1} \neq cv_{2,1}$. Let's do another example, where $M < N$:

```c++
volatile int* const * const * const x {}; // ptr -> ptr -> ptr -> int
reinterpret_cast<volatile double* const * const>(x); // ptr -> ptr -> double

/**
cv1,0 = const
cv1,1 = const
cv1,2 = const
cv1,3 = volatile

cv2,0 = const
cv2,1 = const
cv2,2 = volatile
*/
```

In this case, $K = 2$ and this does not compile because $cv_{1,2} \neq cv_{2,2}$.

For the cases where we're dealing with lvalue or rvalue references, we can determine whether the operation is casting away constness by asking the question: Does "pointer to $T_1$" $\to$ "pointer to $T_2$" casts away constness? For example, does the following cast away constness?

```c++
volatile int* const a {}; // ptr -> int
decltype(a) &x = a; // ref -> ptr -> int
reinterpret_cast<volatile int * const * const &>(x); // ref -> ptr -> ptr -> int

/**
We change the above ref to ptr's:
x: ptr -> ptr -> int
reinterpret_cast: ptr -> ptr -> ptr -> int

cv1,0 = none (pointer artificially added)
cv1,1 = const
cv1,2 = volatile

cv2,0 = none (pointer artificially added)
cv2,1 = const
cv2,2 = const
cv2,3 = volatile
*/
```

As we can see, $K = 2$ and this does not compile because $cv_{1,2} \neq cv_{2,2}$.

## Other Considerations

### Other qualifiers

Note that even though the cast operation is called `const_cast`, it can actually strip away other qualifiers such as `volatile` (which tells the compiler not to optimize access of the variable) and `__restrict`, which is an extension which is vendor-specific in its implementation. Some vendors like GCC [allow `__restrict` on references](https://stackoverflow.com/questions/12841279/should-i-use-restrict-on-references) but it's a concept from C99, which has no notion of references at all. As a result, we chose to **support `__restrict` but only for pointer types to avoid vendor implementation-specific oddities**. 

### Edge cases

The cast operation has one _very_ particular edge case, and that is *`const_cast` fails for function pointers (and in different ways).* This includes free functions and member functions, and we can see this below:

```c++
struct type { 
    void f(int v) const {}
};
void g() {}

...
    
decltype(g) ptr; // const on a free function pointer doesn't do anything
const_cast<decltype(g)>(ptr); // fails, const_cast can't handle func ptrs.

void (type::* f)(int) const = &type::f; // pointer to const member function
const_cast<void(type::*)(int) const>(f); // fails, const_cast can't handle func ptrs.

decltype(f)* fp;
const_cast<decltype(fp)>(fp); // successful, the top layer is not a func ptr
const_cast<void (type::** f)(int)>(fp); // fails, const_cast can't downcast func ptrs.

/**
Both fail with the error:

error: const_cast to '...', which is not a reference, pointer-to-object, or pointer-to-data-member
*/
```

According to the [standard](https://timsong-cpp.github.io/cppwp/n4140/expr.const.cast#12), we can't `const_cast` pointer-to-function types and we also can't downcast function pointers. Because non-member functions can't possibly be declared `const` or `volatile` or `__restrict`(we're talking C++ not C99) due to the semantics of those qualifiers being applied onto `this`, we are mostly concerned with member function pointers being downcasted. As a consequence of this, as long as we have a nested function pointer with no downcasting occurring at the base function pointer level, `const_cast` is successful. C++ is so weird sometimes.

## My implementation in Clang

We want to know when a C-style cast is performing a `const_cast`. In order to check for whether we are casting away constness, the general idea of our approach is to use [`QualType`'s in clang](https://clang.llvm.org/doxygen/classclang_1_1QualType.html#details) and find such $cv_{1,k}$'s in the `CanonicalCastType` and $cv_{2,k}$'s in the `CanonicalSubExprType` and perform a check and recurse if necessary. `QualType` is a type decorated with qualifiers, and one can get information about specific qualifiers with the functions `isConstQualified()`, `isRestrictQualified()`, `isVolatileQualified()`. Since we're dealing with [canonical types](https://clang.llvm.org/docs/InternalsManual.html#canonical-types), we don't discuss the local variants of the above since they're equivalent (which has to do with `typedef`s). In addition, we can get a single `unsigned int` mask that tells us about the qualifiers. *If we're dealing with pointers, `QualType` only tells us about the current pointer level and not the qualifiers on whatever it's pointing to*. 

### Case 0: Function pointers

This is an edge case we'll take care of at the beginning. In the case that we hit a pointer-to-function, we stop and say that we cannot perform the `const_cast` due to the edge case. There are two subtle things to note about this:

First, *the `const_cast` fails even when there is no downcasting.* This is because it can't handle function pointers regardless of what qualifiers are used.

```c++
void (*a)(void);
const_cast<void (*)(void)>(a); // fails
```

Second, *nested function pointers will be successful in a `const_cast`.* This is because these are no longer considered function pointers but rather pointers to pointers. Therefore, we should only check the top level and whether it's a function pointer.

### Case 1: References

In the case that the current type is a reference, we remove the reference and add a layer of pointer by using the `ASTContext`'s method `getPointerType(SomeReferenceQualType)`. Then, we delegate the modified type to case 1 or case 3.

### Case 2: Pointers

In the case that the current type is a pointer, we recursively compare the qualifiers until we hit case 3 (there's no such thing as a pointer to a reference). If the pointer happens to be a function pointer and we're downcasting, then we immediately return false. We then have a list of downcast checks for $cv_{1,0...K}$ and $cv_{2,0...K}$ and we need at least one downcast to be true.

### Case 3: POD types or arrays

In the case of an array, arrays can't have qualifiers on them unless it's in the function declaration [according to the standards](https://en.cppreference.com/w/c/language/array) (to be clear, it's fine to have qualifiers on the members). As a result, we strip the array layer during consideration and proceed to the POD-type underneath. Just to make sure, I tried performing a cast:

```c++
// error: type qualifier used in array declarator outside of function prototype
const double a[const 2] {1, 2};

// Even when we do define it in a function it seems to fail, regardless of standards.
// (clang 10.0.1 with -std=c++17)
// error: qualifier in array size is a C99 feature, not permitted in C++
void g(const double a[const 2]) {}
```

Thankfully, we can't ever deal with down-casting such a type, so we just skip its consideration. However, arrays are always an awkward member in C++, which comes with its [own quirks](http://lists.llvm.org/pipermail/cfe-dev/2020-October/066962.html) in clang's parsing engine. The reason is due to [the language specifying qualifiers on the array](http://eel.is/c++draft/basic.type.qualifier#3.), even when the qualifiers should semantically be on the element types themselves. Performing a `dyn_cast<ArrayType>()` will discard the elements' qualifiers, which means we need to use `ASTContext.getAsArrayType()` (note that there is no such `getAsPointerType()` etc).

In the case of a POD-type, we simply check the qualifiers a single time and return. This is our base case.

## Addendum

I have no idea why the standards state [section 3](https://timsong-cpp.github.io/cppwp/n4140/expr.const.cast#3) in the semantics of $cv_{1,0}$ being the top level pointer cv qualifiers, and then define $cv_{1,N}$ as the top level cv qualifiers in ["casting away constness" in section 8](https://timsong-cpp.github.io/cppwp/n4140/expr.const.cast#8). It caused a lot of confusion and warranted a complete rewrite of the `const_cast` section of the clang tool. Oh well...