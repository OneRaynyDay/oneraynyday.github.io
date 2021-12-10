---
published: true
title: Memory Management And Garbage Collection
category: cs
layout: default
---
# Memory Management And Garbage Collection

# Table of Contents

* TOC
{:toc}

# Heaps

Many languages have unordered runtime memory allocation, and usually we hear this being called a "heap", rather than a "stack".

In C/C++, we hear "heap" and `new`, `malloc`, etc associated. However, **what exactly is the heap**?

The heap is **a pool of blocks of memory, with an interface for unordered alloc/dealloc**.

Heaps are not what you'd think of in a CS data structures class - those are binary/binomial/fibonacci heaps, etc. This is simply a
blob of memory being used in some policy. What's the policy? There are a ton out there:

## Example: First-fit

The pseudocode is:

```python 
def alloc(size)
    for i in range(0, N):
        if FREE_BLOCK[i] > size:
            # Sets these memory blocks unavailable. Splits the block.
            allocate(FREE_BLOCK[i], size)
            return i # starting pointer
    return 0 # returns NULL, as in unsuccessful.
```

We go from the beginning til the end until we see a block that's available, and then we split it.

Similarly, when we deallocate blocks, we need to coalesce the block with its neighboring free blocks. This way, we can permit allocation of bigger blocks(since available
 memory regions are like `[4,0,4,4]`, we can't allocate 8, but if we coalesce into `[4,0,8]`, we can.)

# Quick Lists

It's very common that small blocks get allocated and deallocated more often than larger blocks. Thus, we usually keep a **quick list**, in which all blocks are the same size(1024 for example), and allocate one object in each block. 
Although this leads to a lot of waste, since the objects are small this is usually not a big issue.
The most important result of this is that it's fast. It does not need coalescing, and all blocks are of the same size which is convenient. 

# Heap Links

These data structures are used for both **heap compaction** and **garbage collection**. 

Specifically, **a heap link is a memory location where a value is stored and the program will use it as a heap address**.

The heap links come from a base set, which are all pointers created by the main subroutine.
The links are populated recursively from objects that own its own heap pointers as members, etc.

However, heap links can be prone to errors(false positive, false negative, etc):

```c
union {
    char *p;
    char tag[4];
} x;
```

Is this a heap link if it's instantiated? It could either be a heap allocated block or a stack allocated array of 4 characters. 
To prevent any false negatives, we say that this is in fact a heap link.

# Heap Compaction

**Heap compaction moves all allocated blocks together at one end of the heap, and removes fragmentation in the heap.** To do this, we need to know what are
all the current heap links and then simply copy all the contents into the beginning of the heap, and update the pointer values on the heap links.

However, we can't guarantee that we can do heap compaction on all languages. For example C cannot avoid inclusion errors(false positives) via the union example,
and thus we could be moving values that are not actually alive, and fundamentally change the value of the variable.

# Garbage Collection

Garbage collection is a feature that exists in some languages for using heap links to determine the lifetime of a dynamically allocated variable. It removes any variables that are no in the root set, as in deallocates them.

There are 2 problems that garbage collection tackles:

1. Dangling pointer dereferencing - Creating a pointer that eventually points to an invalid address in heap space.
2. Memory leak - Allocating a resource on the heap and not freeing it after we are done using it.

As a result, some languages replace any semantics for freeing the objects with its own GC. There are 3 normal types of garbage collection:

## Mark-and-sweep GC

Mark and sweep carries the collection in 2 steps:

1. Garbage collector marks heap links that are currently being used.
2. Garbage collector passes over the heap and deallocates any blocks that are not marked.

As one can see, this GC does not reduce fragmentation, because it simply deallocates any blocks that are not marked, and leaves them in its original position.
However, it also means that we won't be accidentally changing contents of our values by moving the blocks(like the issue that heap compaction faces).

## Copying GC

Copying collector only uses half of its available memory at a time. It fills up one side of its memory, then it uses heap links to find non-garbage blocks to copy over to the other side of the memory. As a result, it can also perform heap compaction at the same time.

Obviously, this also suffers from the same issue of heap compaction, and needs to do extra work to copy non-garbage blocks.

## Reference Counting GC

All other GC's need heap links, but this one does not! In a refcount GC, each heap allocated object has # of how many resources are pointing to it.
When the count goes to 0, it is marked for deletion.

However, one issue with refcount GC's is that you can have cycles of garbage:

```
A - B
 \ /
  C
```

In this situation, all 3 objects could have no links except to themselves. In this case, none of these objects are actually GC'd.

There are ways to get around this. In C++ one can use `weak_ptr` and `shared_ptr` to simulate this refcounting relationship with loops.

## Going Meta: Generational GC

Generational GC's take into account that old objects tend to live long, and young objects tend to die fast.

As a result, they have divided heaps for young, midlife, and old objects, where some arbitrary GC algorithm can be conducted on the young heap more often than that of the older ones.

In addition, when an object has lived in the young heap for long enough, it can be copied over to the older heaps.

# Comprehensive comparison of GC's:

1. Mark and sweep:
    - does not support real time(locks entire program to do mark and sweep).
    - allows false positives.
    - does not manage fragmentation.
    - does 2 cycles through to mark and sweep.
2. Copying:
    - does not support real time(locks entire program to copy content).
    - requires there to be no false positives.
    - heap compaction supported.
    - Uses only half of available memory.
3. Refcounting:
    - does support real time.
    - allows false positives.
    - does not manage fragmentation.
    - Leads to garbage cycles.

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
