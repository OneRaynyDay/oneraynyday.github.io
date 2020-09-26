---
published: true
layout: default
category: algorithms
publish: true
---
Used often in cryptography, especially RSA public key/private key systems, modular exponentiation is a bit more tricky than you think it would be.

The way RSA public key private key works in a very basic sense(we're not getting into this today), is that the private key can generate tons of public keys and can verify them easily. Authentication takes no time, but brute forcing the answer takes forever. **You have a number raised to the power of a large number, modulo'd by a larger, prime number.** If the resulting modulo is correct, then authentication is done. This entire pipeline of authentication has one very annoying problem, and that is:

## How do we take care of giant exponents? ##

If we had 3^10000, it would not be even close to fitting the normal scale of integers. If one recalls, 2147483647 is the largest 32 bit integer, and that has 10 digits in base 10. Modular exponentiation should be able to take care of situations much larger than just 10 digits in the exponent. Unfortunately, brute forcing it does not work:


```
const int MY_MOD = 100;
int i = 3;
pow(i, 10000);
return i%MY_MOD;
```

Yeah - just don't do that... First of all, pow() computes a floating point number, and the representation of an extremely large number that the floating point can't handle is inf. Positive/Negative inf, in floating point representation, is _011....1100....00_ and _111....1100....00_. Pow would just give you these values, and of course even after casting to int you would not get what you want. 


In addition, we also can't exponentiate it all at once for the purpose of the exponent possibly being bigger than an int. Pow would throw an error if you tried to pass in something like a long(or maybe even longer than a long, say a BigInteger in java). 


Anyways - what a pain, right? Well don't worry! Modular exponentiation is here!


There are two rules that is necessary for this to be true:

- For a real number x, y and z such that x * y = z, and a modulo a: **z mod a = ((x mod a) * (y mod a)) mod a = ((x mod a) * y) mod a**
- For a real number a, x, y, and z such that x + y = z, **a^z = a^x * a^y**.


An obvious method would be to factor the z into easier-to-swallow components and then perform modulo on them from rule #1. However, we go another step:


Recall that numbers can be represented in radix-2, or binary. For example, 11 would be 1011, or 1 + 2 + 8. This means we can break up the exponent(in rule 2). For example, if z = 11, then it can be represented as a^11 = a^(1+2+8) = a^1 * a^2 * a^8. Now this is where this gets fun: Recall that we can factor a number and get the modulo of the two smaller numbers. Well isn't a square VERY easy to factor? Square... binary... ^2... We're getting somewhere here!


You probably can already guess by now, but: 

	- a^2 mod b = ((a^1 mod b) * (a^1 mod b)) mod b
    - a^3 mod b = ((a^2 mod b) * (a^1 mod b)) mod b
    - a^4 mod b = ((a^2 mod b) * (a^2 mod b)) mod b OR ((a^3 mod b) * (a^1 mod b)) mod b
    - a^8 mod b = ((a^4 mod b) * (a^4 mod b)) mod b
    
... You see where we're getting at? You don't have to know the exponent a^4 to compute a^8 mod b! You only need to know a^4 mod b. This incremental, looping algorithm causes us to not cache any values(because we only need to know the previous a^(n/2) mod b value), and is very convenient with respect to bit shifting.


Now that we know the algorithm, here's the code:

```c++
int modExp(int a, int e) {
    int accumulate = 1, cache = a % MOD;
    while(e != 0){
        if(e & 0x1){
            accumulate = (accumulate * cache) % MOD;
        }
        cache = (cache * cache) % MOD;
        e >>= 1;
    }
    return accumulate;
}
```

See, this solves our first problem - overflow of numbers. Now what about the fact that we can't pass in numbers bigger than int/long? Well... We can actually call this again!


Recall how we can find out the modExp of a^2, a^4, a^8, etc? Well we can also find out a^10, and in general a^b where b in binary contains more than 1 one(we covered that in our algorithm, because we keep shifting the number and detects when the current least significant bit is 1). 


For example, if we have a^123, then that is a^(3+20+100)... Right? Can't we just do modular exponentiation here? We can calculate a^3 easy - **modExp(a, 3)**. What about a^20? Well couldn't we just do the following: **a = modExp(a, 10), and then modExp(a, 2)**? We sure can! All we did was raise a to the next digit, and then modExp'd it as if it was the first digit again. Here's the pseudocode using a vector representation:

```c++
int superPow(int a, vector<int>& b) {
    // b is a representation of the big integer. {1,0,0} = 100.
    int accumulator = 1;

    int cache = a % MOD, b_size = b.size();
    for(int i = b_size-1; i >= 0; i--){
        // Just doing our job here
        accumulator = (accumulator * modExp(cache, b[i])) % MOD;
        // Go to the next digit! If we raise (a^10)^10, then it becomes
        // a^100, then (a^100)^10 becomes a^1000 and so on...
        cache = modExp(cache, 10);
    }
    return accumulator % MOD;
}
```

This code runs relatively fast, with little to no memory overhead - neat!

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
