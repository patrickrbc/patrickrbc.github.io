---
layout: post
title:  "Cryptopals - The basics"
date:   2019-04-06 16:42:00 -0300
comments: true
categories:
---

If you have **any** kind of interest in crypto you should probably check this
out.  Recently, I started doing the **Cryptopals** which are a series of
cryptography challenges created by the formerly Matasano Security team. They
are a collection of 48 exercises divided in 8 sets. I have just finished the
first set but I can guarantee you that it is a lot of fun and learning.

**SPOILER ALERT**: This is post will give the solutions for the first set of
exercises, if you are planning to do it, stop reading this now and try them by
yourself. However, if you are still unsure if this is for you or not, maybe try
skim reading through this post to see if you get the taste for it.

Resolving the first set was mix of love and hate feelings. Sometimes I was able
to get the idea but ended up making a programming mistake that would keep me
stuck for while in one challenge or another. They stated in their website that
this is a “relatively easy” challenge with the except of one. I can’t tell you
about the further sets but one challenge in this set took me 2 days to get it
right.

I decided to write my solutions in Node.js and this was a good way to learn
about some quirks of the language that I did not face in a common daily
programming session. You might also want to choose a new programming language
to kill two birds with one stone.


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
The **simple XOR cipher** is a type of additive cipher that was used in the
past to encrypt text, but it is clearly not safe since the ciphertext could be
decrypted using frequency analysis. On the other hand, the XOR operation is
still frequently used in more complex ciphers, due to its simplicity and
performance.

The challenges asks for a function that takes two equal-length buffers and
produces their XOR combination. A solution for this exercise would be to loop
through one of the buffers doing a XOR with each correspondent byte from the
other buffer. The code that can do this will be shown further in this post.

<br>

# Challenge 3: Single-byte XOR cipher

Now that we already know how to cipher and decipher messages using XOR, things are going
to start becoming interesting. The challenge presents an hex encoded string
that was XOR'd against a single character and asks us to discover this key and
decrypt the message.



```javascript
/*
  This will receive a message and XOR every character in it with a byte which
  is our key. We will try all possible characters with ASCII code from 0 to 256
  (this range was arbitrary).
*/
function breakSingleByte (msg) {
  var plainText, tempBuffer
  var results = []

  for (let keyChar = 0, len = 256; keyChar < len; keyChar++) {

    tempBuffer = cipher(
      Buffer.from(msg, 'hex'),
      Buffer.from(String.fromCharCode(keyChar))
    )

    plainText = tempBuffer.toString()

/*
  If the resultant string has only printable characters, it will have its
  character frequency computed and it will be added to the results array
*/
    if (!/[^\x00-\x7E]/g.test(plainText))
      results.push({
        key: keyChar,
        message: plainText,
        score: parseFloat(Util.calculateFrequency(plainText).toFixed(6))
      })
  }

  // The string with the best score in the results array will be returned 
  return results.sort((x, y) => (x.score < y.score) ? 1 : -1).shift()
}
```

If everything works fine, we will get the decrypted message:

```
Cooking MC\'s like a pound of bacon
```

<br>

# Challenge 4: Detect single-character XOR

This time we were given a file containing a lot of encrypted strings. The Goal
was to find it, but we know from the previous exercise that decrypting it would
be really easy too.

```javascript
var strings = fs.readfilesync('../res/4.txt', 'utf-8').split('\n')

var decrypted = [], result

strings.foreach((string, index) => {
  result = xor.breaksinglebyte(string)

  if (result) {
    result.string = string
    result.line   = index + 1
    decrypted.push(result)
  }
})
```

If everything works fine, we will get the decrypted message:

```
Now that the party is jumping\n
```


<br>

# Challenge 5: Implement repeating-key XOR

This challenge is similar to the second one, but this time the message length
is bigger than the key length and we will use the [Vigenère
cipher](https://en.wikipedia.org/wiki/Vigenère_cipher). In this case, the key
to be repeated to cipher the whole message. Maybe this challenge could be
placed after the second one.

In order to reuse code some of the functions used in these challenges were
created in a separated file. Since XOR is a commonly used operation, I decided
to create a function that will XOR two buffers repeating the smallest one which
would be the key.

```javascript
function cipher (msg, key) {

  if (key.length > msg.length)
    [msg, key] = [key, msg]

  var result = Buffer.alloc(msg.length)

  for (var i = 0, j = 0, len = msg.length; i < len; i++, j++)
    result[i] = msg[i] ^ key[j % key.length]

  return result
}
```


<br>

# Challenge 6: Break repeating-key XOR

*"It is officially on, now."*
This one took me a while to make it work right. They give you a file containing
a message that is encrypted with repeating-xor and encoded in base64. This
where you put together everything you learned until now. Fortunately, they give
you the steps to do it.

The first thing you need to do is to figure out the size of the key. Unless you
have a crystal ball at home, you can start guessing. They suggest values from 2
to 40. 

#### The hamming distance

todo

```javascript
```

Now, using each of the guessed key sizes do the following steps:

 1. Divide the message in KEYSIZE worth of bytes
 2. Calculate the hamming distance between them and normalize dividing by the
   KEYSIZE
 3. Find the KEYSIZE with smallest normalized hamming distance

```javascript
```

Once you have the KEYSIZE, you can use the following strategy to discover the
key:

 1. Break the ciphertext into blocks of KEYSIZE length
 2. Transpose the blocks: make a block that is the first byte of every block,
    and a block that is the second byte of every block, and so on.
 3. Solve each block as it was single-character XOR
 4. For each block, the single-byte XOR key that produces the best output is
    the XOR key byte for that block.
 5. Put them together and you have the key


Using this key against the file will give you the lyrics of [this song]().

<br>

# Challenge 7: AES in ECB mode

After solving the previous challenge the last two are pretty seemed like a walk
in a park. This is an introduction to the block cipher AES.

A block cipher is...
https://en.wikipedia.org/wiki/Block_cipher#Rijndael_

Examples: DES, RC5, Blowfish, AES 

```javascript
function decrypt (input, key) {
  var decipher = crypto.createDecipheriv('aes-128-ecb', key, null)
  var out = decipher.update(input, 'base64', 'utf-8')
  out += decipher.final('utf-8')

  return out
}
```

If everything works fine, we will get the same lyrics of the previous
challenge.


<br>

# Challenge 8: Detect AES in ECB mode

This challenges requires you to find which one of the ciphertexts in a file was
encrytpted with AES in ECB mode. The ECB mode is...

```javascript
ciphertexts.forEach((ciphertext, index) => {
  var cipherTextBlocks = divideInBlocks(ciphertext, 16)

  cipherTextBlocks.forEach((block, i) => {
    results.push({
      line: index + 1,
      ciphertext: ciphertext,
      repetitions: countRepetitions(block, cipherTextBlocks)
    })
  })
})
```

<br>

# What we have learned?

 -
 -
 -

# Conclusion

I am planning on doing the other sets soon and I will probably release the
solutions on Github and make my comments about them here. Stay tuned!

<br>

# References

 - [Tutte, William T. "Fish and I."][fish-1998]

[fish-1998]: https://uwaterloo.ca/combinatorics-and-optimization/sites/ca.combinatorics-and-optimization/files/uploads/files/corr98-39.pdf
