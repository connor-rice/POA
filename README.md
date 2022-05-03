# POA
Educational Padding Oracle Demonstration

This repository provides an educational look at the Padding Oracle attack discovered in 2002.

A Padding Oracle Attack (POA) is a cryptograhpically based attack that manipulates the AES encrpytion process to peek into the encrypted plaintext. To encrpyt plaintext, a key, an initialization vector, and the AES crypto suite is needed. The key is what would be used by the user wishing to encrypt a message. The plaintext would then be XOR'd against the initialization vector, with the key encryption following creating the first block of ciphertext. Since this attack works against CBC mode of AES, that ciphertext block is XOR'd against the second block of plaintext, with the key encryption following. This process is repeated until the whole plaintext is encrypted. An illustration of the CBC mode of AES is provided below.

![image](https://user-images.githubusercontent.com/82915527/166330389-38547110-f0fc-42ec-b18c-985c2ce6d565.png)

However, not all plaintext is 'the perfect length'. This is where the PKCS #7 encryption standard comes in. AES-CBC can only encrypt in full byte blocks. If a plaintext block does not fill in the multiple of the block size required by AES-CBC, PKCS #7 outlines how to 'pad' the rest of the incomplete byte. Hence the name 'Padding' Oracle attack.

# The Oracle

In most systems, an oracle is a system that checks the padding of ciphertext by returning an error. This hint provided by the oracle can help attackers decrypt the cipher text. This padding validation process is normally hidden to prevent such attacks, but for this demonstration, the hint will be provided. 

# The Setup

For this demonstration, a few things will be required:

1.) Oracle VM Virtual Box

2.) Seed Labs Ubuntu 16.04 Virtual Machine

3.) Python 2.7.14 AT MOST (Seed Labs Ubuntu 16.04 contains 2.7.12 but still works)

Items 1 and 2 are not mandatory. Those were used to create the environment and test this attack. As long you can run a Linux terminal with Python 2.7.14 installed, the attack should work.

# The Demonstration

Using the color-coded CBC illustration above, each part of the CBC process is as follows:

Initialization Vector (IV) - Blue

Plaintext - Green

Ciphertext - Yellow

As mentioned previously under the explanation behind a POA, CBC mode encrypts in multiples of 16 bytes. However, not everything being encrypted is an exact multiple of sixteen, so padding is introduced at the end of the block. For example, say 47 bytes were encrypted. That leaves one byte of padding (colored purple) to added to the last block of the plaintext shown in the image below.

![image](https://user-images.githubusercontent.com/82915527/166332230-2475cde7-daf7-4a6a-b6ff-2b36cb692a62.png)

This is all involved in the ecnryption of AES in CBC mode. For the reverse, decryption, the first block of ciphertext gets decrypted with the key and then XOR'd against the IV, producing the original block of plaintext. Before the decryption with the key takes place, the first block of ciphertext is XOR'd against the second block of ciphertext, producing the second block of plaintext. This process is completed until all blocks are decrypted. Staying true to the padding example above, the padding (colored purple) of the last block of plaintext remains.

![image](https://user-images.githubusercontent.com/82915527/166332954-1d57a4cc-fe45-40b2-a2a3-8966236bb4fa.png)

Inside your Linux machine, open a terminal and enter ```python2```. This will put you into Python interactive mode where instructions can be sent and have responses in real time, almost similar to an IDE. To show how the padding works, enter these commands into the terminal:

```
from Crypto.Cipher import AES
key = "0000111122223333"
iv = "aaaabbbbccccdddd"
cipher = AES.new(key, AES.MODE_CBC, iv)
a = "This simple sentence is forty-seven bytes long."
```

These commands will set the intial values AES-CBC will use to encrypt. Again, CBC needs a key and an IV, both of which are 16 bytes long. It will encrypt the value set for ```a```. Since the plaintext is 47 bytes long, padding will be introduced. 

Next, to show how tinkering with the ciphertext changes the only certain parts of the plaintext, the original ciphertext and a modified ciphertext will be compared against each other. To create these two ciphertexts, enter these follwing commands:

```
ciphertext = cipher.encrypt(a + chr(1))
mod = ciphertext[0:31] + chr(255) + ciphertext[32:]
```

The first command adds a single padding character at the end of the plaintext ```a``` and encrypts it. The second command creates the value ```mod``` that changes the last bit of the ciphertext block 16:32, resulting in a different plaintext block 16:32 and final byte of plaintext block 32:38. An illustration is added below to explain the process of modifying the ciphertext resulting in different plaintext.

![image](https://user-images.githubusercontent.com/82915527/166391282-84cce946-d1f0-4c57-a8be-a4150b49383a.png)

Inside your terminal, enter the following commands to view the modified ciphertext next to the original, intact ciphertext:

```
print ciphertext.encode("hex")
print mod.encode("hex")
```
The output from those two commands should look like this:

![image](https://user-images.githubusercontent.com/82915527/166391491-9e3f0251-ef0d-4197-82b0-6d3b3519af3b.png)

Notice the highlighted bytes between ```ciphertext``` and ```mod```. Since you changed the last byte of the ciphertext, when showing the ciphertext in hexadecimal, that byte is different.

Enter these commands:

```
print cipher.decrypt(ciphertext).encode("hex")
print cipher.decrypt(mod).encode("hex")
```

When decrypting ```ciphertext``` and ```mod```, you can clearly see the differences between the ciphertext block 16:32. But also notice that the rest of the rest of the decrypted ciphertext remains mostly the same. This is due to how CBC works in conjuction with the 16 byte long blocks. Changing the last byte of the ciphertext block 16:32 results in a completely different plaintext block.

![image](https://user-images.githubusercontent.com/82915527/166392047-bda7b70a-f508-49d1-b92e-ad17bbbab87b.png)

