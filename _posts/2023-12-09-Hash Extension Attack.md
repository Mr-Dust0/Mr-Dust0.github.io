---
title: Hash Extension Attack
date: 2023-12-09
published: true
categories: ["Crypto"]
tags: ["sha1",  "Merkle Damgard construction", "Hash Extension"]
---
# Hash extension

This attack allows an attacker to append arbitrary data to the text being hashed (with sha1, sha2 and md5) as long as the users data is at the end 
and know a valid hash. This can allow to an attacker to create a valid hash with malicious data even if a secret is prepended to
the data being hashed.

## Background on hashes
![img_8.png](/assets/img/Hash_ext/img_8.png)
This an attack works on hashes Merkle Damgard construction this splits the message into identical blocks and then mix's
these blocks with an internal state using a compression function. The initialization vector (iv) is H<sub>0</sub> and M<sub>i</sub>
is a message bock and then the output on the compression function is then passed as input to the next compression and repeats this until 
all the bytes in the message have been compressed. Then when no more data needs to be compressed the output of the hash
function is just the internal state at the end.

## How it works
As the hash that is outputted is just the internal state of the hash function when it does not need to compress anymore data we know the internal state of the hash function that will be used if a new block is added. 
So the output of the original data without the appended data is the 2nd  to last internal state passed into the function with the appended data
and hence we can create the hash with the appended data because we know the internal state that will be used with the appended data as long as padding the 
data is correct so that the new appended data is in a new block.

![img_10.png](/assets/img/Hash_ext/img_10.png)
In the image above H = Hash(M<sub>1</sub> || M<sub>2</sub>) is the original hash without the appended data, so we know that 
will be the internal state passed to the last compresses function if the padding is correct. The padding needs to be correct
because the hash functions works in blocks so if the appended data is not in a new block it will change the original internal state because the data that is being 
compresses has changed because it is not in a new block. The padding that is used on most algorithms is the text in the last block that isn't a multiple of 64 is padded until it is 8 less than a full 64 byte block.
The data in the uncompleted block is followed by a 1 then null bytes until it is 8 bytes less than a full 64 byte block which is where length of the message goes in hex. 
However, we might not know the length of the message there might be a secret prepended to the hash data that we don't know the length to, so we will have to brute force the padding
until we get valid padding for the length of the secret data that is being prepended to the message. We could also create a program that sets the internal state of the hash function to the hash without the appended data and update the hash function to have a new block with the append data and thus get a valid hash.

## More information
The images come from the [Serious Cryptography: A Practical Introduction to Modern Encryption](https://www.amazon.com.be/-/en/Jean-Philippe-Aumasson/dp/1593278268) and there is 
a good blog post of this by the creator of the hash_extender tool that can be used to exploit this [Blog post](https://www.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks).
The ippsec video extension shows an example of exploiting this attack to get rce on the box [Extension](https://www.youtube.com/watch?v=qNsbf3EmLrA).
