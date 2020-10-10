---
published: true
categories: dev
layout: default
---
Just a list of things to take mindful consideration of when designing C++ objects. This blog post will grow as I keep making mistakes.

# Assignment Operator #
- Remember to deep clone everything, because you don't want to release an object on 1 object that another one is pointing to!
	- This extends to the idea of "use less pointers".
    - When you're deep cloning, make sure you check whether the pointers are nullptrs or not!
- Remember to check whether you're copying a version of yourself!
	- Use an if-statement like "if(this == &object) return \*this;"

# Destructor #
- Remember destructors are very useful if you don't want a memory leak! For those recursively created data structures like trees, you can do the same thing to destroy them recursively(but backwards).
	- Constructors construct the elements FIRST, and then goes into the scope of whatever code you want to inject. Usually, you construct the root element, and then you build the children.
    - Destructors destroys the elements LAST, after executing the code inside the destructor. Usually, you destroy the children, and then destroy itself.
    
``` c++
/* The initializer list executes first! */
SomeObject(int arg1, std::string arg2) : m_arg1(arg1), m_arg2(arg2) { 
    /* Some code here */
    /* This gets executed second! */
}

~SomeObject(){
	/* This gets executed first! */
} /* Destruction happens second! */
```

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
