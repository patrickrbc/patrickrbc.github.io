---
layout: post
title:  "Cryptopals - The basics"
date:   2019-04-06 16:42:00 -0300
comments: true
categories:
---

If you have **any** kind of interest in crypto you should probably check this out.
Recently, I started doing the **Cryptopals** which are a series of cryptography
challenges created by the formerly Matasano Security team. They are a
collection of 48 exercises. I have just finished the first set but I can
guarantee you that it is a lot of fun and learning.

**SPOILER ALERT**: This is post will give the solutions for the first set of
exercises, if you are planning about doing it you should definitely stop
reading and try them by yourself. However, if you are still unsure if this is
for you or not, maybe try skim reading through this post to see if you get the
taste for it.

Resolving the first set was mix of love and hate feelings. Sometimes I was
able to get the idea but ended up making a programming mistake that making me
stuck for while in one challenge or another. They stated in their website
that this is a “relatively easy” challenge with the except of one. I can’t
tell you about the further sets but one challenge in this set took me 2 days
to get it right

I decided to write my solutions in Node.js and this was a good way to learn
about some quirks of the language that you might not face in a common daily
programming session. You might want to choose a new programming language to
kill two birds with one stone.

# Challenge 1: Convert hex to base64

There is no mystery in here. If you were dealing with client-side JavaScript
you would be looking for atob and btoa, but in Node.js you will want to be
familiar with the Buffer API. Needless to 
say, you shouldn’t include any third-party libraries to solve the challenges.

```javascript
Buffer
.from('49276d206b696c6c696e6720796f757220627261696e206c696b65206120706f69736f6e6f7573206d757368726f6f6d','hex')
.toString('base64')
```

<br>
# Challenge 2: Fixed XOR

This challenge will introduce you to XOR operation for cryptographic purposes.
The simple XOR cipher is a type of additive cipher that was used in the past to
encrypt text, but it is clearly not safe since the ciphertext could be
decrypted using frequency analysis. However, the XOR operation is still
frequently used in more complex ciphers, due to its simplicity and performance.


```javascript
```


# Challenge 3: Single-byte XOR cipher

```javascript
```


# Challenge 4: Detect single-character XOR

```javascript
```


# Challenge 5: Implement repeating-key XOR

```javascript
```


# Challenge 6: Break repeating-key XOR

```javascript
```


# Challenge 7: AES in ECB mode

```javascript
```


# Challenge 8: Detect AES in ECB mode

```javascript
```


# What we have learned?


I am planning on doing the other sets soon and I will probably release the
solutions on Github and make my comments about them here. Stay tuned!

# References

 - [Tutte, William T. "Fish and I."][fish-19

[fish-1998]: https://uwaterloo.ca/combinatorics-and-optimization/sites/ca.combinatorics-and-optimization/files/uploads/files/corr98-39.pdf
