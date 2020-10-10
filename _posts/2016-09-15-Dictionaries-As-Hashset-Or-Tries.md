---
published: true
categories: algorithms
layout: default
---
Given many different situations, there are specific data structures which are appropriate for the occasion. If we have a dictionary, and we're trying to search words via wildcard(Like, what if we don't exactly remember the spelling of the word..? Is it spiderman or spoderman?), what kind of data structure should we use?

## One: Hashset - High memory O(N), O(1) on definitive searches, O(26^k) on wildcard searches. ##
The idea is to store every single word into the set as an entire string. In C++ we would use an ```unordered_set<string>```. This approach is super fast for finding the existence of "spiderman" inside of the dictionary. Hashset's find() is O(1). If we have one wildcard, we just have to do 26 find()'s. It looks like:

```
find(sp*derman) //* is wildcard
gives us:
find(spaderman) no
find(spbderman) no
find(spcderman) no
...
find(spiderman) yes!
...
find(spzderman) no

Total : 26 evaluations!

find(sp**erman) //Two wildcards this time
gives us:
find(spaaerman) no
find(spaberman) no
...
find(spazerman) no
find(spbaerman) no
...
find(spiderman) yes!
...
find(spzzerman) no

Total : 26 * 26 = 676 evaluations!
```
And yes, I know you can return early if you already see it. I'm just illustrating the point above. The c++ code below does do that optimization.

However, all this takes a ton of memory, because some redundancies could be reduced. For example, "spider" is inside of "spiderman". If we want to find spiderman, we could've started at "spider" and then just searched for "man" right afterwards, right?

Here's the code in C++:

```c++
class WordDictionary {
public:
    // Adds a word into the data structure.
    void addWord(string word) {
        // Write your code here
        dictionary.insert(word);
    }

    // Returns if the word is in the data structure. A word could
    // contain the dot character '.' to represent any one letter.
    bool search(string word) {
        // Write your code here
        queue<string> possibleWords;
        
        // We need to check whether it's an actual character or not.
        // word.substr(startPtr, endPtr) is where the characters are not '.'
        int startPtr = 0, endPtr = 0;
        possibleWords.push("");
        while(endPtr < word.size()){
            while(word[endPtr] != '.' && endPtr < word.size())
                endPtr++;
            //reached the end of the string or reaches a letter
            if(endPtr == word.size()){
                // In this case, it's not a wild card.
                int wordsInQueue = possibleWords.size();
                while(possibleWords.size() != 0 && wordsInQueue-- > 0){
                    possibleWords.push(possibleWords.front()+word.substr(startPtr));
                    possibleWords.pop();
                }
            }
            else{
                // In this case, it's a wild card.
                int wordsInQueue = possibleWords.size();
                while(possibleWords.size() != 0 && wordsInQueue-- > 0){
                    string w = possibleWords.front();
                    possibleWords.pop();
                    string subword = word.substr(startPtr, endPtr-startPtr);
                    for(int i = 0; i < 26; i++){
                        possibleWords.push(w+ subword + char('a'+i));
                    }
                }
                endPtr++;
                startPtr = endPtr;
            }
        }
        while(!possibleWords.empty()){
            string str = possibleWords.front();
            possibleWords.pop();
            if(dictionary.find(str) != dictionary.end()){
                return true;
            }
        }
        return false;
    }
private:
    unordered_set<string> dictionary;
}
```

## Two: Trie - Lower memory O(logN), slower speed: O(logN) on definitive searches, O(26^k\*logN) on wildcard searches. ##

The trie structure is really a tree. It will answer our plead for an optimization on the memory aspect.

The tree struct is very easy to make. We start off with the root node denoting "", and then we have a 26 letter array of pointers to the next nodes as its children. Each node has a value of bool hasWord, which means a word either exists there or not.

Here is the code:


```c++
class WordDictionary {
public:
    WordDictionary(){
        // root is a dummy node.
        root = new TrieNode();
    }
    // Adds a word into the data structure.
    void addWord(string word) {
        TrieNode* cur = root;
    
        for(int i = 0; i < word.size(); i++){
            int charIdx = word[i]-'a';
            if(!cur->children[charIdx]){
                cur->children[charIdx] = new TrieNode();
            }
            cur = cur->children[charIdx];
        }
        cur->isWord = true;
    }

    // Returns if the word is in the data structure. A word could
    // contain the dot character '.' to represent any one letter.
    // Let's perform a dfs on the search.
    bool search(string word) {
        return containsWord(root, word, 0);
    }
private:
    struct TrieNode{
        static const int NUM_LETTERS = 26;
        bool isWord;
        TrieNode* children[26]; // 26 letters
        TrieNode():isWord(false){
            for(int i = 0; i < 26; i++){
                children[i] = nullptr;
            }
        }
    };
    TrieNode* root;
    bool containsWord(TrieNode* node, string word, int curWordIdx){
        if(!node){
            return false;
        }
        if(curWordIdx == word.size()){
            return node && node->isWord;
        }
        int charIdx = word[curWordIdx] - 'a';
        if(word[curWordIdx] != '.'){
            return containsWord(node->children[charIdx], word, curWordIdx+1);
        }
        bool foundWord = false;
        for(int i = 0; i < TrieNode::NUM_LETTERS; i++){
            foundWord = foundWord || containsWord(node->children[i], word, curWordIdx+1);
        }
        return foundWord;
    }
};
```

I mean... This looks a little more elegant too. :) However, if I were to really care about security I would make this an actual stack to prevent stack overflow. Going character by character could really fuck with our stack limit on the computer.

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
