---
published: true
category: algorithms
layout: default
---
Okay - I know I already made a post about partitioning for quicksort, but there are so many bad points about it that I just **had** to create a new one. Not that what I wrote was bad - my blog's great, but rather the algorithm itself.

**Why is it sub-optimal?**

- Does not take care of many-duplicates situation very gracefully.
	- Search up Dutch Flag Problem.
- Does not run as fast as other partitioning algorithms
	- For example, this partition function is not even used in java/c++ libraries.
    
Here was the code we had before:

## Original partition ##

**Pseudocode**

```
int pivot(vec, l, r):
    pivot = vec[l]
    i = l, j = r;
    while i != j:
        while vec[i] < pivot and i <= j: increment i
        while vec[j] > pivot and j <= i: increment j
        if i == j: break
        if vec[i] == vec[j]: increment i (either is fine, but not both)
        swap(vec[i], vec[j])
    
    return i (either is fine)
```

**C++**

```c++
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
```

So as you can see, if you gave an array like:

> arr = {10, 3, 7, 4, 5, 16, 5, 23, 5, 5}

> Sorted: 3 4 5 5 5 7 10 16 23

You could see that if we were to run partition to get a quickSelect of the median, it could end up like:

> arr = {4, 5, 5, 3, 5, 7, 23, 10, 16} (median = 5)

Wait a second... What if we wanted the middle number to be in... well, the middle? What if we wanted something like:

> arr = {4, 3, 5, 5, 5, 7, 10, 23, 16} (median = 5)

This is very important for applications like the Wiggle Sort, which has a strict bound such that num[0] < num[1] > num[2] < num[3] ...

The obvious idea is to swap out every other number onto the other side of the array, so we get something like:

> arr = {4, 23, 5, 7, 5, 5, 10, 3, 16} (Wiggle sorted!... Sort of?)

Wait a second... If we focus on {... 7, 5, 5, ...}, we can see that this is NOT wiggle sorted! We want num[i] < num[i+1] or num[i] > num[i+1], not num[i] == num[i+1]! In this case, we want all the medians to be in the middle. It's VERY important.

## The 3-Way Partitioning Algorithm ##

Here comes our lord and savior. This algorithm will save our ass for problems like this. It's a tad different from the normal algorithm for pivoting.

**Pseudocode**

```
int pivot(vec, l, r):
    pivot = vec[l]
    lt = l, gt = r, i = l
    while i <= gt:
        if vec[i] < pivot: swap vec[i] and vec[lt], then increment i and lt
        else if vec[i] > pivot: swap vec[i] and vec[gt], then decrement gt
        else (vec[i] is equal to pivot): increment i
    return i 
```

If you stare at it long enough, you can see that the point is to keep track of the pivot, and swap any values that are less than it to behind the pivot. If any value is greater than the pivot, then swap it with the greatest indices. This way, *we keep the minimum values(less than pivot) at the left, and the maximum values(greater than pivot) at the right*. What about the values EQUAL to pivot? Well, this is where this algorithm shines: It's in the middle! The values between lt and gt at the very end will become the same values. To illustrate with the same example above:

    arr = {10(lt, i), 3, 7, 4, 5, 16, 5, 23, 5, 5(gt)} pivot = 10
    arr = {10(lt), 3(i), 7, 4, 5, 16, 5, 23, 5, 5(gt)} pivot = 10
    arr = {3, 10(lt), 7(i), 4, 5, 16, 5, 23, 5, 5(gt)} pivot = 10. 3 is less than 10.
    arr = {3, 7, 10(lt), 4(i), 5, 16, 5, 23, 5, 5(gt)} pivot = 10. 7 is less than 10.
    arr = {3, 7, 4, 10(lt), 5(i), 16, 5, 23, 5, 5(gt)} pivot = 10. 4 is less than 10.
    ...
    arr = {3, 7, 4, 5, 10(lt), 5(i), 5, 23, 5(gt), 16} pivot = 10. 16 is greater than 10.
    arr = {3, 7, 4, 5, 5, 10(lt), 5(i), 23, 5(gt), 16} ... And so on.
    
So now that you get the point, here's the C++ code:

**C++**

```c++
// The pair denotes the start of the equal median elements and the end of it.
pair<int, int> partition3Way(vector<int>& nums, int l, int r){
    int pivot = nums[l];
    int lt = l, gt = r;
    int i = l; 
    while(i <= gt){ // must be equal to so the partition would swap the undiscovered next greater.
        if(nums[i] < pivot){
            swap(nums[i++], nums[lt++]);
        }
        else if(nums[i] > pivot){
            swap(nums[i], nums[gt--]);   
        }
        else{ // (nums[i] == nums[lt])
            i++;
        }
    }
    return make_pair(lt, gt);
}
```

## The dual pivot partitioning ##

The dual pivot is also pretty nice. This is the actual pivoting algorithm being used in Java. It's developed by some russian guy called Yaroslavskiy. But whatever, let's get to the algorithm:

**Pseudocode**

```
int pivot(vec, left, right):
    lpiv, rpiv = vec[left], vec[right] := min(vec[left], vec[right]), max(...)

    l, i, g = left+1, left+1, right-1

    while i <= g:
        if vec[i] < lpiv:
            swap vec[i] and vec[l]
            l++
        elif vec[i] > rpiv:
            while vec[g] > rpiv and i <= g: decrement g
            swap vec[i] and vec[g]

            if vec[i] < lpiv:
                swap vec[i] and vec[l]
                l++
        i++

    swap vec[start] with vec[l-1]
    swap vec[end] with vec[g+1]

    return (l-1, g+1)
```

The c++ one has a ton of comments so you can just read that instead of me trying to explain.

**C++**

```c++
pair<int, int> partitionYaros(vector<int>& nums, int start, int end){
    // The left pivot <= right pivot
    if(nums[start] > nums[end])
        swap(nums[start], nums[end]);
    
    int leftpiv = nums[start], rightpiv = nums[end];
    // Initializing pointers:
    // l is the pointer that points to the element directly greater than leftpiv
    // g is the pointer that points to the element directly less than rightpiv
    int l = start+1, i = start+1, g = end - 1;
    while(i <= g){
        // If the current pointer at i is less than left pivot,
        // we need to put it in the first category.
        // This means we need to swap the current l(directly greater than leftpiv)
        // with this number, then increment l.
        if(nums[i] < leftpiv){
            swap(nums[i], nums[l]);
            l++;
        }
        else {
            // If the current pointer at i is greater than right pivot,
            // we need to put it in the third category.
            // This means we need to swap the current g(may be directly less than leftpiv)
            // with this number. However, the caveat is that
            // we have no information about g, so we need to scan until we find an element
            // that is less than rightpiv.
            //
            // Imagine this as we take the rightmost element that's less than rightpiv,
            // throwing it to our 2nd region, where leftpiv <= x <= rightpiv, checking
            // whether it belongs in the 2nd region, and if it doesn't, it throws the
            // number into the 1st region.
            if(nums[i] > rightpiv){
                while(nums[g] > rightpiv && i < g) g--;
                // Now our g is at an element where it's less than rightpiv, or
                // we got stuck at i.
                swap(nums[i], nums[g--]);
                // Now we need to check whether our new value at k is greater than
                // leftpiv. It's a slight optimization.
                if(nums[i] < leftpiv){
                    // same as the first condition in while loop.
                    swap(nums[i], nums[l++]);
                }
            }
        }
        i++;
    }
    swap(nums[start], nums[l-1]);
    swap(nums[end], nums[g+1]);
    return make_pair(l-1, g+1);
}
```

This one actually runs **a ton faster than the original single pivot, despite the single pivot behaving well to the binary-ness of our computers.** It runs asymptotically faster(It is log base 3 rather than log base 2, since now we have 3 regions.) and behaves better in certain conditions, such as duplicate elements. The 3-Way partition is, in a way, basically this but in a specific case.

I really enjoyed learning about these partitioning methods which improve the speed of algorithms. Pretty cool :)

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
