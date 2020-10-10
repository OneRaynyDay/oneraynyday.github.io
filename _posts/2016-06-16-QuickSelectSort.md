---
layout: default
title: Quick Select Sort (Partial Sorting)
category: algorithms
published: true
---

Let's skip the boring stuff of algorithms, like the ones we already learned in school - Quicksort, mergesort, etc. What's interesting is the different ways to spin these algorithms just a tiny bit to get a good result. Lately I've been trying to hone my algorithmic skills for the upcoming career fair season and I came across an algorithm problem to _**find all k least/greatest elements in a list**_(in a nutshell).

So, the basic idea of this Quick Select Sort is that we incorporate **Quicksort** and **Binary Search** in the same algorithm. What do I mean by this? 

To find the k greatest elements in a list, we just need to find the k-th greatest element and do a pivot partitioning(remember this from quicksort?) around it to get the correct answer. 

The idea of pivotting is simple but very tricky when you get to the nitty gritty of repeating values and returning the correct value of the pivot. We'll write some pseudo-code to cover this idea:

```
partition(vector vec, int pivotIndex, int start, int end){

	pivot = vec[pivotIndex]
    int left = start, right = end;
    while left <= right 
    	while left < pivot and left <= right: increment left
        while right > pivot and right >= left: decrement right
        // Now we have the right swapping points! But there are some catches...
        /* Case 1: Repeated value will lead to infinite loop */
        if vec[left] == vec[right]: increment right and continue
        /* Case 2: We end early */
        if left == right: break
        
        swap vec[left], vec[right]
    
    return left // or right (it doesn't matter)
    
}
```

Now that we have the partitioning done, we should be able to have a result that slightly resembles this:

> Initial configuration:
1
1
5
2
4
2
0
2
5
1
0
5
0
2
3
5
1
1
4
4

>pivot value: 2

>pivot, after, is at : 11

>Final configuration:
1
1
1
1
2
2
0
2
0
1
0
2
5
5
3
5
4
5
4
4

Sounds good? Awesome. Now that we have the basics of quicksort mechanics down, we just need to add a bit more of the binary sorting portion into the mix.

```
binarySearch(vector vec, int k, int start, int end){
	
    int piv = partition(vec, (start+end)/2, start, end); // but any pivot index is fine
    
    /* IMPORTANT! */
    /* We need to have an established offset practice.
       The k-th is actually the k-1 because arrays are offset by 1.
       This means we should make a new variable. Check the cpp version! */
    
    if k > offset_piv: binarySearch(vec, k-offset_piv, piv+1, end); // Search the right side. 
    if k < offset_piv: binarySearch(vec, k, start, piv-1); // Search the left side. 
    if k == piv: return; // We've correctly partitioned yay!
}
```

That's all there is to it!

## Algorithmic Complexity

This is fairly easy to see. The partitioning step in quicksort is not very safe if we have a ton of duplicates(Think about it for a bit). This kind of behavior leads to O(N^2) in a normal quicksort.

Trivial fact: The tight bound on 1 partition is O(N).

Now, on an average case the pivot will be in [N/2] position, given N is the size of the array. This means we've eliminated N/2 elements from the list! Wow that's pretty good right? If we write it out, it seems like we have a geometric series:

>N + N/2 + N/4 + N/8 ... 

This looks like the standard C\*1/(1-ratio). What does it evaluate to asymptotically? We get N\*1/(1-1/2) = 2N!

That means this algorithm on average runs O(N).

What about worst case? The partitions would be so unlucky that we only eliminate one value, which is the pivot. Well, then we would say it would be no longer geometric but rather arithmetic:

>N + N-1 + N-2 + N-3 ... + 1

This is the standard sum of i from 1 to N, which has a asymptotic value of N\*(N+1)/2.

That means this algorithm, at worst case, runs O(N^2). There is a secondary algorithm called the "Median of medians", which helps decrease the complexity of worst case to O(N) as well. We might discuss this later. For now just think of it as O(N) average and O(N^2) if we're super unlucky.

Now we're able to find the k-th element, and a cool niche with this algorithm is that we also get the group of elements from k to most largest in a subvector/subarray for free!

Pretty neat huh. :^)

Here's the code in C++ for the implementation if anyone is interested:

```c++
#include <iostream>
#include <vector>
#include <stdlib.h>
using namespace std;

int pivot(vector<int>& vec, int pInd, int start, int end){
    int pivot = vec[pInd];
    int i = 0, j = end;
    while(i <= j){
        while(vec[i] < pivot && i <= j){ i++; }
        while(vec[j] > pivot && j >= i){ j--; }
        if(i == j)
            break;
        else if(vec[i] == vec[j]){
            i++;
            continue;
        }
        int temp = vec[i];
        vec[i] = vec[j];
        vec[j] = temp;
    }
    return j;
}

void partialQuicksort(vector<int>& vec, int k, int start, int end){
    int piv = pivot(vec, (start+end)/2, start, end); 
    int length = piv - start + 1;
    if(k < length) 
        partialQuicksort(vec, k, start, piv-1);
    else if(k > length)       
        partialQuicksort(vec, k-length, piv+1, end);
    else
        return;
}
```

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
