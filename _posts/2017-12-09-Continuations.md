---
published: true
title: What Are Continuations?
category: cs
layout: default
---
# What Are Continuations

# Table of Contents

* TOC
{:toc}

Continuations in scheme are quite tricky. Here we will try to delve deep into how they work.

# What is a Continuation?

A continuation, is "just about to return from call-with-current-continuation", and it represents the rest of the computations.

In other words, a continuation is an **arbitrary point in a program where a value is expected**.

Continuations essentially save the state of the current program(registers, ip/ep, etc), and halts execution.

# Using Continuations as Currying

```scheme
(+ a b) ; we need to fill in a b with values! These are the "arbitrary points"
```

How do we fill in these points? We can either pass in values, or more interestingly, we can kind of curry this function by passing in 2 initially.

```scheme
(define handle #f) ; define a variable bound to continuation.

; pass in 2 as the first argument always
(+ 2
    ; this call/cc routine substitutes the 2nd 
    ; argument of the function.
    ; We expect the argument to be a procedure that 
    ; takes 1 argument - the continuation.
    (call/cc (lambda (k) ; k is the continuation
        ; We bind handle to the continuation point.
        ; and the we return the value 2.
        (set! handle k) 2)    
    )
)
```

Here, k is the continuation point, and we can set handle to k. 
We don't yet evaluate `(+ 2 ...)`, and so we get back the handle that, once evaluated, will give us the value.

We could've done something like:

```scheme
(define handle
    (lambda (x) (+ 2 x)))
```

but there are more uses for continuations.

# Using Continuations as Returns

In Scheme, we don't have "returns", and so for example if we wanted to search for an element in a list that matches a criteria,
we would have to recurse down to that element, and the recurse back up the function stack to return the value. Would it be nice
if we had a short-circuit?

```scheme
;;; Does not compile!
(define (search want? lst)
    (for-each (lambda (e)
        (if (want? e) (return e))) lst) ; return the element early!
    #f ; no results
    )
```

This would be really nice, because for-each will go through every single element and return the last element.

We can use `call/cc` to emulate the return though!

```scheme
;;; Does compile!
(define (search want? lst)
    (call/cc
        (lambda (return) ; here we define the return value.
            (for-each (lambda (e)
                (if (want? e) (return e))) lst) ; return the element early!
        #f)
    )
)
```

What happens here is that the moment we find the value via `want? e`, 
we send the `e` into `return`, which is a procedure that takes a continuation.

The entire subroutine just stops right there, and `call/cc` just returns the element.
All the registers and instruction pointers are saved at that snapshot, with the `%rax` 
register holding the value of `e`.

# Green Threads and Generators

A green thread is a thread that's on the user-level, and cannot reap the benefits of parallelism.
Its usage is usually for multiplexing between coroutines.

One can implement green threads using continuations! A master clock can call a ton of coroutines using continuations, by aggregating
a list of them, and calling each one in order.

```scheme
(thread1 1) ; Each thread is a call/cc return value.
(thread2 2)
...
(threadN N)
(thread1 1) ; back to the beginning! Multiplex N threads.
```

Generators are also similar - in fact, in python, there is very little difference between the asyncio coroutines and
python generators. You can iterate through a list without having to hold all of it:

```scheme
;;; Nice try! But not there.
(define (iter lst)
    (call/cc (lambda (return)
        (return (car lst))
        (iter (cdr lst)))))

(define x (list 1 2 3))

(iter x) ; 1
(iter x) ; 1
(iter x) ; 1
```

This will give us a non-stateful generator.

To capture state, we need to have a state variable.

```scheme
;;; Stolen from http://danielfm.me/posts/why-are-continuations-so-darn-cool.html
(define (iter lst)
    ;; Defines `state` as being a function that starts the
    ;; iteration via `for-each`
    (define (state return)
        (for-each
            (lambda (item)
            ;; Here, we capture the continuation that represents the
            ;; current state of the iteration
            (let/cc item-cc
                ;; Before the item is yielded, we update `state` to
                ;; `item-cc` so the computation is resumed the next
                ;; time the generator is called
                (set! state item-cc)
                ;; Yields the current item to the caller
                (return item)))
         lst)

    ;; Yields 'done when the list is exhausted
    (return 'done))

    ;; Returns a function that calls the stored `state` with the
    ;; current continuation so we can yield one item at a time
    (define (generator)
        (call/cc state))
    generator)

(define x (list 1 2 3))
(define next (iter x))
(next) ; 1
(next) ; 2
(next) ; 3
```

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
