---
layout: default
title: Median of Medians (Finding median-ish values)
category: algorithms
published: true
---
In the previous post we said that our quickSelectSort was O(N^2) worst case. This is super bad because if we simply used a heapsort algorithm, which is O(N) heapify(Might elaborate on this later), and O(klogN) to extract out k greatest elements, then the total is O(N+klogN) which is asymptotically lower than O(N^2) since we know k < n. We want to use the best algorithm to select k greatest elements right? Well, then we have to tweak the O(N^2) implementation of quickSelectSort a bit. The key is to **use a median-finding technique**.

With a naïve implementation, we could just say - sort the array and then find the floor(N/2)-th element. This will take O(NlogN) if we use a smart sorting algorithm like mergesort or heapsort. We already know that O(NlogN) is the typical upper bound efficiency for sorting via comparison, so we can't do anything more than O(NlogN) to find the median.

Imagine we are trying to find the median in O(NlogN) time, but our partitions that require this median for pivotting is in O(N). That means our algorithm's worst case time complexity will spike up to O(NlogN)! Our quickSelectSort should not change in performance as we do this. So what should we do?

## Algorithm

In a typical situation, we would do the following: {Mathematical notations without LaTeX incoming} 


- Given a set of numbers S. Denote N as cardinality(S).

- Partition S into floor(S/5) groups of size 5 + an extra leftover group if set not divisible by 5.
	- Denote each set as S1, S2, S3, ... S{n/5}.
    	- O(1) because we don't really need to do anything.
    - Find the median of the sets S1, S2, S3, ... S{n/5} and name them M1, M2, M3, ... M{n/5}. 
    	- O(N/5 * 1) = O(N)
    - We take these medians and then do the same thing to these medians again! As in, M1, M2, M3, ... M{n/5} is now the numbers S. Repeat from the start.
    	- Repeated iterations: O(N/5) + O((N/5)/5) + O(((N/5)/5)/5) ... Geometric series! --> O(N).
    
_Note_: Contrary to popular belief, _this is NOT O(NlogN)_! Just because we sorted the small lists of 5 does NOT mean the big O is O(NlogN). We can think about it as always being constant - requiring X amount of comparisons and swaps only. For example - if it takes O(NlogN) to sort 8 elements and pick the middle element, we just need 8\*log(8) = 8 * 3 = 24. 24 is a constant. It's not a variable in this case.

Okay, so you might not be sold on the fact that the median will indeed be a median. And you're right - you caught me. It's not going to be the "exact" median, but at least it's close enough(and that's the key point of this)!

C++ Code implementation:

```c++
/* In case someone wants to pass in the pivValue, I broke partition into 2 pieces.
 */
int pivot(vector<int>& vec, int pivot, int start, int end){
    
    /* Now we need to go into the array with a starting left and right value. */
    int left = start, right = end-1;
    while(left < right){
        /* Increase the left and the right values until inappropriate value comes */
        while(vec.at(left) < pivot && left <= right) left++;
        while(vec.at(right) > pivot && right >= left) right--;
        
        /* In case of duplicate values, we must take care of this special case. */
        if(left >= right) break;
        else if(vec.at(left) == vec.at(right)){ left++; continue; }
        
        /* Do the normal swapping */
        int temp = vec.at(left);
        vec.at(left) = vec.at(right);
        vec.at(right) = temp;
    }
    return right;
}


/* Returns the k-th element of this array. */
int MoM(vector<int>& vec, int k, int start, int end){
    /* Start by base case: Sort if less than 10 size
     * E.x.: Size = 9, 9 - 0 = 9.
     */
    if(end-start < 10){
        sort(vec.begin()+start, vec.begin()+end);
        return vec.at(k);
    }
    
    vector<int> medians;
    /* Now sort every consecutive 5 */
    for(int i = start; i < end; i+=5){
        if(end - i < 10){
            sort(vec.begin()+i, vec.begin()+end);
            medians.push_back(vec.at((i+end)/2));
        }
        else{
            sort(vec.begin()+i, vec.begin()+i+5);
            medians.push_back(vec.at(i+2));
        }
    }
    
    int median = MoM(medians, medians.size()/2, 0, medians.size());
    
    /* use the median to pivot around */
    int piv = pivot(vec, median, start, end);
    int length = piv - start+1;
    
    if(k < length){
        return MoM(vec, k, start, piv);
    }
    else if(k > length){
        return MoM(vec, k-length, piv+1, end);
    }
    else
        return vec[k];
}
```

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
