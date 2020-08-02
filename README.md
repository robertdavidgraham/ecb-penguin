# ECB Penguin demonstration

This project is a reproducible demonstration of the "ECB Penguin" concept
-- so that you can do the work yourself on the command-line to make your
own images.
This is for students who want to play with encryption tools to either
reproduce the results themselves, or play with the concept to create
variations.

## What is the ECB Penguin?

The most common encryption algorithm, AES, is a *block cipher* with
128-bit blocks.

A *block cipher* always encrypts the same contents the same way, given
the same key.

Naively, that doesn't seem like a problem because that output is still
encrypted, and hence "secure", but it reveals information. We can see that
two different blocks in the encrypted message are in fact the same, even
if we don't know their contents.

We can demonstrate this by encrypting cartoon images. Cartoons tend to have
regions where all the colors are the same, and hence, will encrypt the same
when using a block cipher.

The most famous cartoon for demonstrating this, from where this problem gets
its name, is the Linux penguin mascot known as "Tux". The following picture
shows Tux.

![Tux](/Tux.png)

Notice how large regions of the cartoon are either completely white (0xFFFFFF)
or completely black (0x000000).

In the image below, we've encrypted it.

![Tux-ecb](/Tux.ecb.png)

As you can see, the image is still recognizable as a penguin.

There are a number of ways of fixing this. Instead of straight encryption,
block by block, we do something slightly different to prevent to identical
blocks from encrypting the same way. This are known as *modes* -- different
ways of applying the basic block cipher.

One way is known as *counter* mode.
You assign a counter to each block (first block = 1, second block = 2, and so on).
Instead of encrypting the *block* with a *key*, you encrypt the *block* with
both the *key* and the *counter*. Now each input block will produce different output,
even when the key and the contents of the block are the same, because the counter will
be different.

Remember that the fundamental principle of cryptography is complete randomness. Even
though the counter is often only a single bit difference, it changes effectively all
the bits of the output. Thus, encrypting with the same key and some block contents
with a counter of 2 and 3, the outputs of these two blocks will be wholly unrelated
to each, even though the counter is only one bit different (0b0010 vs. 0b0011).

Another way of handling this is knowh as *cipher block chaining*.
Instead of encrypting a *block* with a *key*, you encrypt the current *block* with
both the *key* and the results of the previous block.

In the following example, we encrypt our image with AES, but this time, using
*cipher block chaining*. As you can see, now the contents of the file are complete
nonense. We have no clue here what the original contents might have been.

![Tux-cbc](/Tux.cbc.png)

The different ways of using AES here are called *modes*. Any block cipher can be
used in these modes. These modes were originally defined for the 56-bit DES standard
from the 1970s, and are now used for the modern AES standard. Presumably in the future
a new block cipher standard will be created that will still use these various modes.

The original naive mode is known as an *electronic code book* or *ECB*. This refers
to how a block cipher is what in older times would be refered to as a *code book*.

In the early
days of cryptography, a code book contained a list of words that were replaced by
other words. For example, you might replace "attack" with the word "paint", so 
the message "paint at dawn" would actually mean "attack at dawn". Given a key,
AES acts as a sort of code book -- each block of plaintext is replaced by 
the specified ciphertext. You could in theory create an improbably large book
consisting of every possible output block given an input block.

Because of this *ECB penguin* issue, nobody uses code book mode (AES-ECB) in 
practice.

Other modes have different properties. One useful feature of AES-CBC
("cipher block chaining" mode) is that any corruption near the start of the chain
will cause the rest of the chain to decrypt into nonsense. Indeed, this "chaining"
concept analogous to the "block chain" behind Bitcoin, such that the entire chain
of transactions cannot be altered in any place without altering the entire chain.

However, this chaining is useless for encrypted disk drives that have "random access",
which need to randomly change things in the middle of the drive. This is where
modes like AES-CTR (counter mode) might come into play. Every block on the disk
is assigned a different number, so that you can seek to that location on the disk
and decrypt just that block without having to decode all the blocks that came before
it first.

Chaining and counter modes are basic textbook examples. In practice, more complex
modes are chosen. When using AES on the Internet (such as with SSL), you'll probably
be using AES-GCM ("Galois Counter Mode"), to both encrypt and verify/authenticate
that it hasn't been corrupted.

Full disk encryption, like on your desktop or phone, might use AES-XTS.
XTS (as in AES-XTS)
stands for "*xor-encrypt-xor-based tweaked codebook mode with ciphertext stealing*".
This demonstrates how modes can get so complex that even simply expanding the
acronym can be difficult.

## Reproducing the results

The purpose of this project isn't to describe the ECB penguin but to help the student
reproduce it.

We start with the file `Tux.ppm`. This is in an uncompressed, raw format known as PPM.

The PPM image format has four text fields followed by binary data: a file type, the
width in pixels, the heigh in pixels, and the number of colors (often 255 or 256 for
8-bit color).

The remainder of the file is binary data containing the raw, uncompressed image.

We can retrieve the header of this file by doing:

    $ head -n 3 Tux.ppm
    P6
    265 314
    255

This means the rest of the binary data is 265x314 pixels by 255 colors.

To create our penguin, we want to do the following steps:
  1. separate the header from the body
  2. encrypt the body using the `openssl` command-line tool
  3. combine the header back with the body
  
I did this with the following commands;

    head -n 3 Tux.ppm > Tux.header
    tail -n +4 Tux.ppm > Tux.body
    openssl enc -aes-128-ecb -nosalt -pass pass:"rob" -in Tux.body -out Tux.body.ecb
    cat Tux.header Tux.body.ecb > Tux.ecb.ppm
  
Then, you want to view the results. Many image viewers can decode the PPM format.
On macOS, I used the Preview app. I also used that app to save the results in
the more conventional (compressed) PNG format (`Tux.ecb.png`), which you can view in the browser.
I've checked in the PNG format into this project, for display in this page.

For changing the mode to *cipher block chaining*, simply change the algorithm to
*aes-128-cbc", and change the file output to *Tux.body.cbc*.

    openssl enc -aes-128-cbc -nosalt -pass pass:"rob" -in Tux.body -out Tux.body.cbc
  
## Conclusion

All of this is discussed on Wikipedia https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation.
The purpose of this project is simply a 'show your work' allowing students
to do this themselves on the command-line.





