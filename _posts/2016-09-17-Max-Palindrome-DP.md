---
published: true
category: algorithms
layout: default
---
So I just got the news that doing DP for palindromes is cool now. Anyways, so given a question like: "Given this string S, find the biggest palindrome it has." Now... Disregarding the fact that we'll actually never use this stupid algorithm in our real life situations... How do we answer it?

# The naive DP (still better than retarded approach) #

Well, my first idea(and the one I used most often) is the classical dp approach. I know for sure that IF a palindrome exists, it can only become a BIGGER palindrome if its left and right characters are also the same.

> Given that s(i,j) is a palindrome, s(i-1, j+1) is a palindrome iff s[i-1] == s[j+1].

That's fairly straight forward right? Well, we just have to memoize s(i,j) for any 0 <= i <= j < n where n is the size of the string.

**This is an O(N^2) operation**. It's pretty straight forward and takes O(N^2) space as well.
Here's the code. No real pseudocode is really needed for this. It's simple as hell.

```c++
string longestPalindrome(string s) {
    if(s.size() == 0)
        return "";
    bool isPalindrome[s.size()][s.size()];
    for(int i = 0; i < s.size(); i++){
        for(int j = 0; j < s.size(); j++){
            if(i == j || i == j+1)
                isPalindrome[i][j] = true;
            else
                isPalindrome[i][j] = false;
        }
    }
    
    // i denotes the length of the palindrome.
    int max_start = 0, max_len = 1;
    for(int i = 1; i < s.size(); i++){
        // j denotes the starting index of the palindrome.
        for(int j = 0; j < (s.size()-i); j++){
            if(isPalindrome[j+1][j+i-1] && s[j] == s[j+i]){
                isPalindrome[j][j+i] = true;
                max_start = j;
                max_len = i;
            }  
        }
    }
    return s.substr(max_start, max_len+1);
}
```

# The real algorithm #
So the real algorithm is much smarter. It even has a name. It's called **Manacher's algorithm**. It's probably named after a guy called Manacher but that's an ugly name, so I'll just call it O(N) DP. (lol like i dont even kno how to pronounce that name wtf)

So O(N) DP utilizes the property of symmetry of a palindrome. Recall that:

> A palindrome like s = "ababcbaba" has symmetrical counts of palindromes. 

> For example, count for each letter in order: {1,3,3,1,9,1,3,3,1}. 

So the basic idea is to say: If we found the count for 1 side, we can just copy paste it to the other side... right?

Well, there's a small caveat: 

> Given the situation: "abacababa".

> This count is NOT: {1,3,1,7,1,3,**1**,..}.

> It is: {1,3,1,7,1,3,**5**,..}

> Try it with "ababacaba". It's the same idea.

So how do we take care of these situations? Well, we can see that another less-long palindrome is actually using the SAME characters. That's why the characters are actually greater, like the 1->5.

The key idea is to say:

L     C     R

a b a c a b a b a

1 3 1 7 1 3 **1** ? ? 

Is **INCORRECT!** We can see that, after some time of analyzing..

L     C     R

a b a c a b a b a

1 3 1 7 1 3 **5** 3 1

Should be the correct answer.

Given the boundaries, we can see that we should START at the minimum index, which is R-i, where i is the current index of the string, and it should try to advance forwards. The fact that we gave the other half of the palindrome a starting index(in this case, 1) already decreases the traversal time. In the extreme cases, where we have:

1. "abcdefg" - palindrome of size 1
	- we do not actually traverse for each index past itself. This is O(N).
2. "aaaaaaa" - entire string is palindrome
	- The traversal looks like the following:
    
> 1 ? ? ? ? ? ?

> 1 3 1 ? ? ? ?

> 1 3 5 3 1 ? ?

> 1 3 5 7 5 3 1

If we accumulate the number of traversals, we see: 1 + 2 + 4 + 4. In fact, the 4 constant is the maximum for this given scenario if you work out the proof.
    
As handwavy as that is, we can see the intuitive idea of this algorithm, which is to accelerate the next indices with a head start on what the minimum palindrome size is. This greatly improves performance. 

**If you think I'm a piece of shit at explaining, look at this: http://manacher-viz.s3-website-us-east-1.amazonaws.com/#/. It's got GUI's and stuff!**

Here's the pseudocode:

```
f(s):
    c := 0, r := 0
    s := augment(s)
    for i = 0...s.size:
        if i > r:
            curLen = 0
        else 
            curLen = min(r-i, s[2*c-i]) //2*c-i is the other side of the center
        while s[i+(curLen+1)] == s[i-(curLen+1)] curLen++

        if i+curLen > r
            r = i+curlen
            c = i
```

The augment function turns the string into one that's easier to work with regarding this algorithm. We put a character in between every character. The standard one I guess is like the '#' character.

> e.g. "abcba" becomes "$#a#b#c#b#a#^", where # is the filler and $ and ^ are the start and end characters respectively.

Anyways, here's the juicy C++ code. (It's not actually that juicy it's pretty short)

```c++
string augmentManacher(const string& s){
    string manacher("$");
    for(int i = 0; i < s.size(); i++){
        manacher+='#';
        manacher+=s[i];
    }
    manacher+='#';
    manacher+='^';
    return manacher;
}

string longestPalindrome(string _s_) {
    // Perform manacher's algorithm on this:
    string s = augmentManacher(_s_);
    vector<int> v(s.size(), 0);
    int C = 0, R = 0, maxIdx = 0;
    for(int i = 1; i < s.size()-1; i++){
        int curLen = (i > R) ? 0 : min(R-i, v[2*C-i]);
        while(s[i-(curLen+1)] == s[i+(curLen+1)]) curLen++;
        v[i] = curLen;
        if(i+curLen > R){
            R = i+curLen;
            C = i;
        }
        if(curLen > v[maxIdx])
            maxIdx = i;
    }
    return _s_.substr((maxIdx-v[maxIdx])/2, v[maxIdx]);
}
```

# How can we use this for other palindrome questions? #

For example, we can use it for this question: https://leetcode.com/problems/shortest-palindrome/

How? Well, we just set an extra limitation on the manachers algorithm to keep the current max palindrome starting from the very beginning. How do we check for that? We check if the curLen == C, or we check if curLen * 2 == R, or if R - 2C == 0. A lot of ways. Anyways, in my version I used the last part. Assuming we have the same manacher string appending process, we have:

```c++
string shortestPalindrome(string _s_) {
    // We know the shortest character is just the first character.
    string s = augmentManacher(_s_);
    vector<int> v(s.size(), 0);
    int C = 0, R = 0;
    int maxLen = 0;
    for(int i = 1; i < s.size()-1; i++){
        int curLen = (i > R) ? 0 : min(R-i, v[2*C-i]);
        while(s[i-(curLen+1)] == s[i+(curLen+1)]) curLen++;
        v[i] = curLen;
        if(i+curLen > R){
            R = i+curLen;
            C = i;
        }
        if(curLen > maxLen && (R-1) - 2*(C-1) == 0){ // making sure it's on the left side.
            maxLen = curLen;
        }
    }
    string append = _s_.substr(maxLen);
    reverse(append.begin(), append.end());
    return append + _s_;
}
```

Anyways. This was a pretty lengthy blog. I'm gonna go to the gym bye

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
