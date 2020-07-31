# ECB Penguin demonstration

This project is a reproducible demonstration of the "ECB Penguin" concept.
This is for students who want to play with encryption tools to either
reproduce the results themselves, or play with the concept to create
variations.

## What is the ECB Penguin?

This is a demonstration of the different possible "modes" of encryption
algorithms, contrasting what's known as "electronic code book" mode to
"cipher block chaining" mode.

The most common encryption algorithm, AES, is a "block cipher". That means
it encrypts data a block at a time, or 128-bits at a time. Given the same
encryption key, that means the same input block will always encrypt to the
same output block.

Naively, that doesn't seem like a problem because that output is still
encrypted and insecure. That it actually is a problem can be demonstrated
by encrypting images, where we can see the content of an image even
though it's encrypted.

The famous demonstration of this is when encrypting the raw form of "Tux",
the Linux mascott penguin.

You can see that below. Even though the precise data is encrypted, you
can still see that this is the outline of a penguin.

![Tux-ecb](/Tux.ecb.png)

This mode is called "electronic code book". Historically, "codes" and "ciphers"
refered to slightly different things. A "cipher" was a tansformation of the
text, replacing every input letter with an output letter, like the classic
Caesar cipher or Enigma machine. A "code book" was a book of words, where
words like "army" or "attack" might be replaced in a message with words like
"flower" and "fly". Or, a code book can be demonstrated from that scene
in "Star Trek: Wrath of Kahn" where the word "hours" is replaced with "days".

Thus, instead of thinking of AES as a mathematical transformation of each
block as a cipher, you could think of it as a very large (*very* large) codebook
where you lookup the value of each 128-bit block in the book, and swap it with the
replacement listed there.

There are a number of ways of fixing this. The simplest and most straightforward
is known as *cipher block chaining* or *CBC* mode. It makes sure that the encryption
of the current block depends not only on the *key*, but also on the encryption of 
all previous blocks.

The simplest way to do that is to XOR the current plain-text 128-bit block with
the encrypted results of the previous block. This can easily be done in reverse
when decrypting the sequence.

When doing that, we get the following result for encrypting Tux. As you can
see, it looks like random noise and not like a penguin.

![Tux-cbc](/Tux.cbc.png)

It's not simply a matter of replacing ECB mode with CBC. For example, encrypted disk drives
need to do ramdom access, so they can't encrypt/decrypt in a stream. The recommended 
algorithm for SSL isn't CBC, either, because of reasons, but is instead GCM (Galois counter mode).

## Reproducing the results

The purpose of this project isn't to describe the results, but to help the student
reproduce them.

We start with the file `Tux.ppm`. The PPM image format has four white-space delimited fields,
a file type "P6", an integer indicating the width (in pixels), an integer indicating the height
(in pixels), followed lastly by a number of colors per pixel (often 256 for 8-bit pixels).

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
the more conventional PNG format (`Tux.ecb.png`), which you can view in the browser.
I've checked in the PNG format into this project, for display in this page.

For changing the mode to *cipher block chaining*, simply change the algorithm to
*aes-128-cbc", and change the file output to *Tux.body.cbc*.

    openssl enc -aes-128-cbc -nosalt -pass pass:"rob" -in Tux.body -out Tux.body.cbc
  
## Conclusion

All of this is discussed on Wikipedia https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation.
The purpose of this project is simply a 'show your work' allowing students
to do this themselves on the command-line.





