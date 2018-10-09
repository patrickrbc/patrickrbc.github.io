---
layout: post
title:  "Cryptopals: The basics"
date:   2019-04-06 16:42:00 -0300
comments: true
categories:
---

If you have **any** kind of interest in crypto you should probably check this
out.  Recently, I started doing the **Cryptopals**, which are a series of
cryptography challenges created by the formerly Matasano Security team. They
are a collection of 48 exercises divided in 8 sets. I have just finished the
first set but I can guarantee you that it was already a lot of fun and
learning.

**SPOILER ALERT**: This is post will give the solutions for the first set of
exercises, if you are planning to do it, stop reading this now and try them by
yourself. However, if you are still unsure if this is for you or not, maybe try
skim reading through this post to see if you get interested.

While resolving the first set I had mixed feelings of love and hatred.
Sometimes I was able to get the idea but ended up making a programming mistake
that would keep me stuck for while in one challenge or another. They stated in
their website that this is a “relatively easy” set of challenges with the
except of one. I can’t tell you about the further sets but one challenge in
this set took me 2 days to get it right.

I decided to write my solutions in Node.js and this was a good way to learn
about some quirks of the language that I was not used to face in a common daily
programming session. You might also want to choose a new programming language
to kill two birds with one stone.


# Challenge 1: Convert hex to base64

There is no mystery in here. If you were dealing with client-side JavaScript
you would be looking for atob and btoa, but in Node.js you will want to be
familiar with the Buffer API. Needless to 
say, you shouldn’t include any third-party libraries to solve the challenges.

{% highlight javascript %}
var result = Buffer
  .from('49276d206b696c6c696e6720796f757220627261696e206c696b65206120706f69736f6e6f7573206d757368726f6f6d','hex')
  .toString('base64')
{% endhighlight %}

<br>

# Challenge 2: Fixed XOR

This challenge will introduce you to XOR operation for cryptographic purposes.
The **simple XOR cipher** is a type of additive cipher that was used in the
past to encrypt text, but it is clearly not safe since the ciphertext could be
decrypted using character frequency analysis. On the other hand, the XOR
operation is still frequently used in more complex ciphers, due to its
simplicity and performance.

The challenge asks for a function that takes two equal-length buffers and
produces their XOR combination. A solution for this exercise would be to loop
through one of the buffers doing a XOR with each correspondent byte from the
other buffer. The code that can do this will be shown further in this post.

<br>

# Challenge 3: Single-byte XOR cipher

Now that we already know how to cipher and decipher messages using XOR, things
are going to become interesting. The challenge presents a hex encoded string
that was XOR'd against a single character and require us to discover this key
and decrypt the message. This can be achieved by brute-forcing the key and
analysing the character frequency of the output messages. The output with the
best score is probably the message decrypted.

{% highlight javascript %}
/**
 * Break a single-byte XOR cipher and returns the message with the best score
 * @param {String} hex encoded ciphertext
 * @return {Object} key, msg and score
 */
function breakSingleByte (ciphertext) {
  var plainText, tempBuffer
  var results = []

  for (let keyChar = 0, len = 256; keyChar < len; keyChar++) {

    tempBuffer = cipher(
      Buffer.from(ciphertext, 'hex'),
      Buffer.from(String.fromCharCode(keyChar))
    )

    plainText = tempBuffer.toString()

    if (!/[^\x00-\x7E]/g.test(plainText))
      results.push({
        key: keyChar,
        msg: plainText,
        score: parseFloat(Util.calculateFrequency(plainText).toFixed(6))
      })
  }

  return results.reduce((x, y) => (x && x.score > y.score) ? x : y, 0)
}
{% endhighlight %}


To calculate the score of a string according to the frequency of english
letters you can find the percentual values on the Internet and sum them. Don't
forget to add the space character to your list, it took me some time to notice
that.

{% highlight javascript %}
/**
* Computes the score of a string according to the frequency of letters
* @param {String} string
* @return {Number} score
*/
function calculateFrequency (string) {

  /* Source: http://www.oxfordmathcenter.com/drupal7/node/353 */
  const charFrequency = {
    'e': 0.12702, 't': 0.09056, 'a': 0.08167, 'o': 0.07507, 'i': 0.06966,
    'n': 0.06749, 's': 0.06327, 'h': 0.06094, 'r': 0.05987, 'd': 0.04253,
    'l': 0.04025, 'c': 0.02782, 'u': 0.02758, 'm': 0.02406, 'w': 0.02360,
    'f': 0.02228, 'g': 0.02015, 'y': 0.01974, 'p': 0.01929, 'b': 0.01492,
    'v': 0.00978, 'k': 0.00772, 'j': 0.00153, 'x': 0.00150, 'q': 0.00095,
    'z': 0.00074, ' ': 1
  }

  return string.toLowerCase().split('')
    .reduce((sum, value) => sum + (charFrequency[value] || 0), 0)
}
{% endhighlight %}

If everything works fine, we will get the decrypted message:

```
Cooking MC\'s like a pound of bacon
```

<br>

# Challenge 4: Detect single-character XOR

This time we were given a file containing a lot of encrypted strings. The goal
was to find the string that was encrypted with single-character XOR, but we
know from the previous exercise that decrypting it would be really easy too. We
can use the function from the previous challenge to brute-force each string as
a single-byte XOR cipher and find out the one with the best score. 

{% highlight javascript %}
var decrypted = [], result

strings.forEach((string, index) => {
  result = XOR.breakSingleByte(string)

  if (result) {
    result.string = string
    result.line   = index + 1
    decrypted.push(result)
  }
})

result = decrypted.reduce((x, y) => x.score > y.score ? x : y)
{% endhighlight %}

If everything works fine, we will get the decrypted message:

```
Now that the party is jumping\n
```


<br>

# Challenge 5: Implement repeating-key XOR

This challenge is similar to the Challenge #2, but this time the message length
is bigger than the key length, so we will use the [Vigenère
cipher](https://en.wikipedia.org/wiki/Vigenère_cipher). In this case, the key
needs to be repeated to cipher the whole message. Maybe this challenge could be
positioned after the second one.

In order to reuse code, some of the functions were created in a separated file.
Since XOR is a commonly used operation, I decided to create a function that
will XOR two buffers repeating the smallest one which would be the key. This
function could be used with fixed or repeating-key XOR.

{% highlight javascript %}
/**
 * XOR two buffers repeating the smallest
 * @param {Buffer} msg
 * @param {Buffer} key
 * @return {Buffer} result
 */
function cipher (msg, key) {

  if (key.length > msg.length)
    [msg, key] = [key, msg]

  var result = Buffer.alloc(msg.length)

  for (var i = 0, j = 0, len = msg.length; i < len; i++, j++)
    result[i] = msg[i] ^ key[j % key.length]

  return result
}
{% endhighlight %}

Ciphering the message with the above function and converting the result to a
hex string will give the expected output.

<br>

# Challenge 6: Break repeating-key XOR

*"It is officially on, now."*
This one took me a while to make it work right. They give you a file containing
a message that is encrypted with repeating-xor and encoded in base64. This
challenge is when you put together everything you learned until now.
Fortunately, they give you the steps to do it.

In order to get this one right, you will need to use something called the
[Hamming distance](https://en.wikipedia.org/wiki/Hamming_distance), which is
the number of different position between two symbols. In this case, we need to
calculate the number of differing bits between the two strings.

{% highlight javascript %}
/**
* Calculates the Hamming distance between two strings
* @param {String} string1
* @param {String} string2
* @return {Number} distance 
*/
function calculateDistance (string1, string2) {
  var distance = 0
  var first, second

  string1 = Buffer.from(string1 || '0', 'ascii')
  string2 = Buffer.from(string2 || '0', 'ascii')

  // compose the binary representation of the strings 
  for (let i = 0, len = string1.length; i < len; i++)
    first += dec2bin(string1[i])

  for (let i = 0, len = string2.length; i < len; i++)
    second += dec2bin(string2[i])

  // count the number of different bits
  for (let i = 0, len = first.length; i < len; i++)
    if (first[i] ^ second[i])
      distance++

  return distance
}

/**
* Converts a decimal number to a 8 char binary representation
* @param {Number} decimal number
* @return {String} binary representation
*/
function dec2bin (dec) {
  var binaryNum = (dec >>> 0).toString(2);
  while (binaryNum.length < 8)
    binaryNum = '0' + binaryNum
  return binaryNum
}
{% endhighlight %}

The first thing you need to do is to figure out the size of the key. Unless you
have a crystal ball at home, you can start guessing. They suggest values from 2
to 40. 

{% highlight javascript %}
/**
* Test all key sizes from 2 to 40 and returns the best one
* @param {String} msg
* @return {Number} keysize
*/
function findKeySize (msg) {
  var results = []

  for (let keysize = 2, len = 40; keysize <= len; keysize++)
    results.push({
      keysize : keysize,
      score   : testKeySize(msg, keysize)
    })

  return results.reduce((x, y) => x.score < y.score ? x : y).keysize
}
{% endhighlight %}

If you are wondering what kind of test is that, no worries, they explain you
how to do it. It basically consists of getting the first two chunks of KEYSIZE
bytes from the ciphertext, computing the Hamming distance between them,
repeating this process a couple of times and normalizing the result. The
KEYSIZE which gives the least distance between the chunks of bytes in the
ciphertext is probably the right one.

{% highlight javascript %}
/**
* Given a message and the size of a key, returns the average hamming distance
* of between chunks of that message
* @param {String} msg
* @param {Number} keysize
* @return {Number} distance
*/
function testKeySize (msg, keysize) {
  var first, second
  const TESTS = 50 // arbitrary
  var score   = 0

  for (let i = 0, len = TESTS; i < len; i++) {
    first  = msg.slice(keysize * i, keysize * (i + 1))
    second = msg.slice(keysize * (i + 2), keysize * (i + 3))
    score += Util.calculateDistance(first, second)
  }

  return score / (TESTS * keysize)
}
{% endhighlight %}

Once you have the KEYSIZE, you can use the following strategy to solve the
challenge:

 1. Divide the ciphertext into blocks of KEYSIZE length;
 2. Transpose the blocks: make a block that is the first byte of every block,
    and a block that is the second byte of every block, and so on;
 3. Solve each block as it was single-character XOR;
 4. For each block, the single-byte XOR key that produces the best output is
    the XOR key byte for that block;
 5. Put them together and you have the key;
 6. Decipher the message with the key you obtained.

{% highlight javascript %}
var key, keysize, blocks, msg, result
var keyScores = []

msg     = Buffer.from(fs.readFileSync('../res/6.txt', 'utf-8'), 'base64')
keysize = XOR.findKeySize(msg)
blocks  = Util.divideInBlocks(msg, keysize)
blocks  = Util.transposeBlocks(blocks, keysize)
key     = findKey(blocks)
result  = XOR.cipher(msg, key).toString()
{% endhighlight %}

Using this key against the file will give you the lyrics of [this
song](https://www.youtube.com/watch?v=zNJ8_Dh3Onk).

<br>

# Challenge 7: AES in ECB mode

After solving the previous challenge the last two seemed like a walk in the
park. This is an introduction to the block cipher
[AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard). A [block
cipher](https://en.wikipedia.org/wiki/Block_cipher#Rijndael_) is a
deterministic algorithm which operates on fixed-length groups of bits. Examples
of block ciphers are DES, RC5, Blowfish, AES. The following code will do the
job of decrypting a base64 encoded ciphertext in Node.js:

{% highlight javascript %}
/**
* Decrypts a AES ciphertext with a given key and algorithm
* @param {String} ciphertext
* @param {String} key
* @param {String} algorithm
* @return {String} msg
*/
function decrypt (ciphertext, key, alg) {
  var decipher = crypto.createDecipheriv(alg, key, null)
  var out = decipher.update(ciphertext, 'base64', 'utf-8')
  return out += decipher.final('utf-8')
}
{% endhighlight %}

If everything works fine, we will get the same lyrics of the previous
challenge.


<br>

# Challenge 8: Detect AES in ECB mode

This challenges requires you to find which one of the ciphertexts in a file was
encrytpted with AES in ECB mode. The ECB mode is simplest mode of encryption in
which the message is divided into blocks and they are independently encrypted.
The problem with this mode is that a block is always encrypted into the same
ciphertext block. This is exactly the weakness you need to abuse in order to
detect which ciphertext was encrypted in ECB mode, you will count the
repetitions.

{% highlight javascript %}
ciphertexts.forEach((ciphertext, index) => {
  var cipherTextBlocks = Util.divideInBlocks(ciphertext, 16)

  cipherTextBlocks.forEach((block, i) => {
    results.push({
      line: index + 1,
      ciphertext: ciphertext,
      repetitions: Util.countRepetitions(block, cipherTextBlocks)
    })
  })
})
{% endhighlight %}

<br>

# Conclusion

The complete source code of the solutions you can find on my
[Github](https://github.com/patrickrbc/cryptopals). If you solved this first
set of exercises you have the knowledge of performing the following tasks:

 - Convert between data base64, hex, binary
 - Detect and decrypt single-character XOR ciphers 
 - Break repeating-key XOR ciphers
 - Detect and decipher AES in ECB mode

I am planning on doing the other sets soon. I will probably release the
solutions on Github and make my comments about them here. Stay tuned!

<br>

# References
 - [The cryptopals crypto challenges](https://cryptopals.com)
