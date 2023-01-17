---
layout: post
title:  "Primitive Roots Matter"
date:   2022-10-22 12:46:00 +0300
categories: rust
---
The other day, I happened upon a nice writeup titled [Modular Exponentiation in Rust](https://rob.co.bb/posts/2019-02-10-modular-exponentiation-in-rust). However, after reading halfway through the article, I found some things that didn’t jive. I’ve emailed the author to notify him of the error, so the content may be fixed by now.

Modular exponention is used in [Diffie–Hellman key exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange) to compute public, private, and shared keys. Below, I describe
 the subtle problems that occur if one of the primes is not a primitive root of the other.

Before I delve into the details, it's important to understand what makes a number a primitive root of a prime number n modulo n.  A primitive root of a prime will generate all the numbers le
ss than a second prime using modular exponentiation.  Said another way, if we compute `(g ^ a) % p` where `g` is a primitive root of `p`, than for all `a` `[0, p-2]`, the result is a range of
 all numbers `[1, p-1]`.  Further explanation and examples can be found [here](https://www.geeksforgeeks.org/primitive-root-of-a-prime-number-n-modulo-n/).
\
\
\
__Setup__
---

Alice:
- prime 1 (modulus): `941`
- prime 2 (base): `631`
- chossen secret (exponent): `289`
- shareable number: `(631 ^ 289) % 941 = 854`

Alice sends `941`, `631`, and `854` to Bob, and Eve cant figure out the secret `289`.


Bob:
- prime 1 (modulus): `941`
- prime 2 (base): `631`
- chossen secret (exponent): you choose `523`
- shareable number: `(631 ^ 523) % 941 = 124`

Bob replys with `124`.

\
Next, Alice and Bob independently compute a shared key.

Alice:
- modulus: `941`
- base: `124`
- exponent: `289`
- shared key: `(124 ^ 289) % 941 = 349`

Bob:
- modulus: `941`
- base: `289`
- exponent: `124`
- shared key: `(289 ^ 124) % 941 = 349`

\
The issue is that `289` is Alice's secret, so Bob can't possibly know this.

Instead, it should read:

Alice computes: `(124 ^ 289) % 941`
Bob computes: `(854 ^ 523) % 941`

But hold on, now if we compute `(124 ^ 289) % 941` we get `934` and `(854 ^ 523) % 941` results in `933`.  Each party should compute the shared key separately and arrive at the same result (even though they use different values during the computation).  The problem here is that `g` is not a primitive root of `p`.  To quote [wiki](https://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange):

> ..the protocol uses the multiplicative group of integers modulo p, where p is prime, and g is a primitive root modulo p

In the above example, `g=631` and `p=941`, which means that in order for the math to work out, 631 must be a primitive root of `p`.  Listing all primitive roots of `941` using the tool [here](http://bluetulip.org/2014/programs/primitive.html) shows that `631` is indeed not a primitive root.  However, `632` and `629` are both primitive roots of `941`.  Using `632` in place of `631` gives us the expected results.

\
Alice:
- prime 1 (modulus): `941`
- prime 2 (base): `632`
- choosen secret (exponent): `289`
- shareable number: `(632 ^ 289) % 941 = 703`

Bob:
- prime 1 (modulus): `941`
- prime 2 (base): `632`
- choosen secret (exponent): `523`
- shareable number: `(632 ^ 523) % 941 = 699`

\
When Alice and Bob compute the shared key, they get the same answer.

Alice:
- modulus: `941`
- base: `699`
- exponent: `289`
- shared key: `(699 ^ 289) % 941 = 839`

Bob:
- modulus: `941`
- base: `523`
- exponent: `703`
- shared key: `(703 ^ 523) % 941 = 839`

\
Pretty cool. We now arrive at the same shared secret while using different secret keys that are never revealed.  It's all made possible by using `g` `631` which is a primitive root of `p` `941`.
