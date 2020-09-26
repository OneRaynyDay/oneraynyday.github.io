---
published: true
category: algorithms
layout: default
---
So I was doing this leetcode question that asked for sorted odd/even lists. I misunderstood the question like always, and coded a harder version. Here's the question for reference: https://leetcode.com/problems/odd-even-linked-list/

Here's a version that sorts all even nodes and odd nodes together. It's a little tricky but it works :)

The idea is to run through the list with a before-even pointer and a before-odd pointer. We start at the before-even pointer to find the before-odd pointer, because we assume the odd will always come after(this isn't really true, but as we can see later we define it to be true). We connect the before-odd node to the even node(trivially, we can see that the node before the odd is an invariant - an odd node), then the even to the odd node, then the before even node to the after even node. 

It looks like this:
Denote before-odd as A, before-even as B, odd as A', even as B', after odd as A'', after even as A''.

[   2   ,   1   ,   3   ,   4   ,   5   ,   6   ]
    A       A'     B,A''    B'     B''
We connect A to B', B' to A', then B to B''. What a switcheroo...

Now... back to our assumption. **How can we say for sure that the odd one is always going to be the first one?** Well, going back to our question, I assumed that the groups just had to be separated, such that 2,4,6,1,3,5 and 1,3,5,2,4,6 were both valid answers. **So, I just took the first number, and called it the even node even if it was the odd node. All we need to know is whether something is "like" the first node, or not.**

Anyways, it's trivial that this algorithm is O(N). We gucci.

```c++
ListNode* getBeforeModuloNode(ListNode* head, int modulo){
	while(head && head->next){
		if(head->next->val % 2 == modulo){
			return head;
		}
		head = head->next;
	}
	return nullptr;
}

ListNode* oddEvenList(ListNode* head) {
	if(!head)
		return nullptr;
	int modulo = head->val % 2;
	ListNode* beforeOdd = getBeforeModuloNode(head, !modulo);
	ListNode* beforeEvenAfterOdd = getBeforeModuloNode(beforeOdd, modulo);
	while(beforeOdd && beforeEvenAfterOdd){
		ListNode* odd = beforeOdd->next;
		ListNode* even = beforeEvenAfterOdd->next;
		ListNode* afterEven = even->next;
		beforeOdd->next = even;
		even->next = odd;
		beforeEvenAfterOdd->next = afterEven;

		beforeOdd = getBeforeModuloNode(head, !modulo);
		beforeEvenAfterOdd = getBeforeModuloNode(beforeOdd, modulo);
	}
	return head;
}
```

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
