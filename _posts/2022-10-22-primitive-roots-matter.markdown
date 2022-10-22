---
layout: post
title:  "primitive roots matter"
date:   2022-10-22 12:46:00 +0300
categories: rust
---

The other day I happened upon a nice writeup about modular [exponentiation in Rust](https://rob.co.bb/posts/2019-02-10-modular-exponentiation-in-rust).  After reading halfway through the article however, I found some things that didn't make sense.  I've emailed the author to notify him of the error, however I wanted to document my findings for others.

Modular exponention is used [Diffie–Hellman–Merkle key exchange](https://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange) to compute public, private and shared keys.  The problem I discovered is how the authr of [exponentiation in Rust](https://rob.co.bb/posts/2019-02-10-modular-exponentiation-in-rust) describes the process of creating shared keys.


The problem setup:
```
    prime 1 (modulus): 941
    prime 2 (base): 631
    secret: I choose 289
    shareable number: (631 ^ 289) % 941 = 854

So, I would send over 941, 631, and 854 to you, and even with those numbers, a snoop couldn’t figure out that my secret was 289.

You would do something similar:

    prime 1 (modulus): 941
    prime 2 (base): 631
    secret (exponent): you choose 523
    shareable number: (631 ^ 523) % 941 = 124

and send back 124.
```

And the mistake:
```
Me:

    modulus: same old prime, 941
    base: your secret, 124
    exponent: my secret, 289
    shared key: (124 ^ 289) % 941 = 349

You:

    modulus: same old prime, 941
    base: my secret, 289
    exponent: your secret, 124
    shared key: (289 ^ 124) % 941 = 349
```

The issue is that `289` is the secret of party A, so party B can't possibly know this.

Instead, I it should read:

Party A computes: (124 ^ 289) % 941
Party B computes: (854 ^ 523) % 941

But hold on, now if we compute `(124 ^ 289) % 941` we get `934` and `(854 ^ 523) % 941` results in `933`.  The issue here is that each party should compute the shared key seperatly and arrive at the same result (even though they use different values during the computation).  The problem here is that g is not a primitive root of p.  To quote [wiki](https://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange):

```
..the protocol uses the multiplicative group of integers modulo p, where p is prime, and g is a primitive root modulo p
```

In the above example, `g=631` and `p=941`, which means that in order for the math to work out, 631 must be a primitive root of p.  listing all primitive roots of `941` using the tool [here](http://bluetulip.org/2014/programs/primitive.html) shows that `631` is indeed not a primitie root.  However, `632` and `629` are both primitive roots of `941`.  Using `632` in place of `631` gives us the expected results.


The problem setup:
```
    prime 1 (modulus): 941
    prime 2 (base): 632
    secret: I choose 289
    shareable number: (632 ^ 289) % 941 = 703

So, I would send over 941, 631, and 854 to you, and even with those numbers, a snoop couldn’t figure out that my secret was 289.

You would do something similar:

    prime 1 (modulus): 941
    prime 2 (base): 631
    secret (exponent): you choose 523
    shareable number: (632 ^ 523) % 941 = 699

and send back 124.
```

```
Me:

    modulus: same old prime, 941
    base: your public, 699
    exponent: my secret, 289
    shared key: (699 ^ 289) % 941 = 839

You:

    modulus: same old prime, 941
    base: my secret, 523
    exponent: your public, 703
    shared key: (703 ^ 523) % 941 = 839
```

Pretty cool, we now arrive at the same shared secret while using or different secret keys which are not revealed.  All made possible by using `g` `631` which is a primitive root of `p` `941`.
