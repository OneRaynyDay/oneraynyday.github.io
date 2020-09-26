---
published: true
category: algorithms
layout: default
---
When given a sorted array, say a vector\<int\>, how would one go about constructing the binary search tree that has "maximized bushiness", or in other words, the most close configuration to being a heap, aka complete BST? Well, this one is easy. However, let's talk about a linked list.

In an array, the idea is very easy - we just take the median of every single array and make that the head of the current depth. We perform a recursive dfs to construct the tree this way. On leetcode, I wrote this code in \<5 minutes and had it approve in 2 tries.

```c++
// Helper function
TreeNode* recurseBuild(vector<int>& nums, int start, int end){
	if(start > end)
		return nullptr;
	int med = (start+end+1)/2;
	TreeNode* node = new TreeNode(nums[med]);
	node->left = recurseBuild(nums, start, med-1);
	node->right = recurseBuild(nums, med+1, end);
	return node;
}

// Main function
TreeNode* sortedArrayToBST(vector<int>& nums) {
	if(nums.empty()){
		return nullptr;
	}
	int med = nums.size()/2;
	TreeNode* root = new TreeNode(nums[med]);
	root->left = recurseBuild(nums, 0, med-1);
	root->right = recurseBuild(nums, med+1, nums.size()-1);
	return root;
}
```
This code is so simple that I probably don't need to explain it.

**Cough cough, anyways - that's not the main attraction of this blog post. Let's carry on.**

However, I was completely humbled by the tree construction algorithm for a list. I thought about it, and one idea that I had was to have 1 pointer find the half point by moving only half as fast as the other pointer. This would be the "median". However, this is a O(nlogn) algorithm. Using the master theorem, we see that the equation is:

> T(N) = 2\*T(N/2)+O(1) 

Thus, we see that the algorithm is O(nlogn).

I saw this answer online, which was an in-order construction of the tree using the linkedlist. I was flabbergasted - how does he do this?

Here's the pseudocode just for those who tl;dr:

```
TreeNode* inOrderRecurse(ListNode*& node, start to end):
	if start > end: // means we passed a single node. A single node has start == end.
    	return null
    mid = avg of start and end
    // Left side
    TreeNode* left = inOrderRecurse(node, start to mid-1)
    // Middle side
    TreeNode* subroot = new TreeNode(node->val)
    node = node->next // Key point: We move this node(originally head) to the next node.
    // Right side
    TreeNode* right = inOrderRecurse(node, mid+1 to end)
    
    subroot->left = left
    subroot->right = right
    return subroot

TreeNode* sortedListToBST(ListNode* head):
	len := length of the list starting with head. Use a simple while loop.
    inOrderRecurse(head, 0 to len-1)

```

I never actually had much use of in-order traversal other than this before. The main point is that **even though it seems we're taking care of the middle node's value first, we're actually going all the way to the left, taking its value from head, and then going to the next node.**

The nature of an in-order traversal is of the following: 

- It takes care of the left first, then the middle, then the right
- It will execute the tree-building such that there are no disjoint sets created in the process.
- It executes the subroutine in order(as the name suggests). 
	- This causes our node = node-next code execute elegantly. It only is called in order, so the correct tree node will be given the correct node's value.
    
Anyways. This was pretty eye-opening. I thought in-order was only for printing, since I've never encountered a problem that really uses it.

Here's my C++ code:

```c++
TreeNode* inOrderRecurse(ListNode*& node, int start, int end){
	if(start > end)
		return nullptr;
	int mid = (start+end+1)/2;
	// Left side
	TreeNode* tleft = inOrderRecurse(node, start, mid-1);
    // Middle side
	TreeNode* tnode = new TreeNode(node->val);
	tnode->left = tleft;
	node = node->next;
	// Right side
	tnode->right = inOrderRecurse(node, mid+1, end);
	return tnode;
}

TreeNode* sortedListToBST(ListNode* head) {
	ListNode* node = head;
	int listLen = 0;
	while(node){
		node = node->next;
		listLen++;
	}
	return inOrderRecurse(head, 0, listLen-1);
}
```

Hope y'all enjoyed this one. :)

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
