---
published: true
category: algorithms
layout: default
---
So segment trees are kind of hype now. I guess we can use it to find the largest increasing subsequence type questions, or to find largest subset sum of an array kind of thing. 

## What they are: Just trees! ##
Not too complicated I would say. Instead of containing a single value like how we're used to in a BST or something, we get an "interval". The tree doesn't have to be balanced but if we were to create a continuous segment tree, it would look something like:


```
          [ 4, 8 ]
          /      \
      [4, 6]    [7, 8]
      /    \    /    \
   [4,5] [6,6][7,7][8,8]
   /   \
[4,4][5,5]
```

# class/struct definition #
```c++
class SegmentTreeNode {
public:
    int start, end, max;
    SegmentTreeNode *left, *right;
    SegmentTreeNode(int start, int end, int max=0) {
        this->start = start, this->end = end, this->max = max;
        this->left = this->right = NULL;
    }
}
```
I believe according to cpp guidelines by bjarne stroustrup, these tree nodes should often be structs. But whatever, I'm a fkin' rebel tho

# Construction #
```c++
SegmentTreeNode * build(int start, int end) {
    if(start > end){
        return nullptr;
    }
    SegmentTreeNode* root = new SegmentTreeNode(start, end);
    root->left = start == end ? nullptr : build(start, (start+end)/2);
    root->right = build((start+end)/2 + 1, end);
    return root;
}
```
Again - prone to stack overflow. It's okay tho.
One thing to look out for is that if you don't use the 

>start == end ? nullptr

Part of the logic, you'll run into an infinite loop when you have an interval of size 1(e.g. [3,3]).

# Query for something #
In this, we can query for a member called max. This basically is the maximum subsequence problem.

```c++
int query(SegmentTreeNode *n, int start, int end) {
    int n_start = n->start;
    int n_end = n->end;
    int mid = (n_start + n_end)/2;

    // If the current node's range is within range then we return it
    if(n_start >= start && n_end <= end){
        return n->max;
    }
    // If the start range is greater than this node's middle,
    // We throw away the left side.
    if(start > mid){
        return query(n->right, start, end);
    }
    else if(end <= mid){ // Same logic to the end range
        return query(n->left, start, end);
    }
    else{ // This is the special case of n_start <= start <= end <= n_end
        return max(query(n->right, start, end), query(n->left, start, end));
    }
}
```

This one was a litle trickier than the construction. But here we know that the maximum number between, say, (1,4) is obviously going to be greater than or equal to (1,1), or (2,3), or (2,4). So we exit early.

But anyways, the maximum sum interval is easy in this respect. And our query function can define a specific range's max as well. We can solve in O(1) the entire range, and we can solve in O(nlogn) for some range that's within the root's range.

Sounds good? Sounds good!

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
