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

## Inside the Terminal

Inside your Linux machine, open a terminal and enter ```python2```. This will put you into Python interactive mode where instructions can be sent and have responses in real time, almost similar to an IDE. To show how the padding works, enter these commands into the terminal:

```
from Crypto.Cipher import AES
key = "0000111122223333"
iv = "aaaabbbbccccdddd"
cipher = AES.new(key, AES.MODE_CBC, iv)
a = "This simple sentence is forty-seven bytes long."
```

These commands will set the intial values AES-CBC will use to encrypt. Again, CBC needs a key and an IV, both of which are 16 bytes long. It will encrypt the value set for ```a```. Since the plaintext is 47 bytes long, padding will be introduced. 

## Modifying and Viewing the Ciphertext

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

## The Code

Included in this repository is a Python script called ```poa.py```. It is located in the ```main``` branch. Copy this script into the home directory of your Linux machine so the function with ```poa.py``` can be called. Be sure the extension of the file is ```.py``` or the interactive mode will not be able to read it.

## Encrypting Non-Multiple Byte Length Plaintext

To keep things simple, the same value for ```a``` will remain the same. Enter these commands:

```
from poa import encr, decr
a = "This simple sentence is forty-seven bytes long."
c = encr(a)
print c.encode("hex")
```

The function ```encr``` is called to encrypt the value ```a``` and store it in ```c```. It will then print the ciphertext in hexadecimal which will look like this:

![image](https://user-images.githubusercontent.com/82915527/166393407-2294e94a-456a-48d0-b80f-8c2155fd3b80.png)

To decrypt it, enter these following commands:

```
d = decr(c)
print d
print d.encode("hex")
```

The output will look like this:

![image](https://user-images.githubusercontent.com/82915527/166393517-791f507f-1689-4cbc-b1de-3ae85e356510.png)

Notice how the last byte is ```01```. This means there is a single byte of padding due to the fact that the plaintext is 47 bytes long which is not a perfect multiple of the needed 16 byte blocks. After the command ```print d``` is executed, the output has a strange character at the end of the decrypted ciphertext. This is because the byte is not printable in the ASCII version, but it is printable in hexadecimal.

Since only one byte of padding is needed, per Public Key Cryptography Standard #7, or PKCS#7, use ```01``` to fill in the rest of the block. This padding is incremented depending on how many bytes of padding are needed. For example, if two bytes of padding are needed, us ```0202```. If three bytes of padding are needed, use ```030303```. This continues all the way up to needing potentially 15 bytes, which is ```0f0f0f0f0f0f0f0f0f0f0f0f0f0f0f```. Since CBC works in 16 byte blocks, no more padding is needed beyond that point.

## The Crutch

Here is where things become interesting. Recall the term "oracle". If you need, an explanation of what an Oracle does is provided near the top of this file.

To see what an Oracle does in action, enter the following commands:

```
mod = c[0:47] + chr(65)
decr(mod)
```
Notice the output after entering these commands:

![image](https://user-images.githubusercontent.com/82915527/166394635-287313b7-df38-4064-a34a-449339a65396.png)

This is the crutch that is the Oracle. 

When an attacker has the ability to exploit the padding validation check, it gives them the ability to find the original plaintext. When the Oracle returns a valid padding check, this tells the attacker that the last byte of the last ciphertext block XOR'd against the last byte of the modified ciphertext block is 0x01, meaning the last byte of the decrypted cipher block equals the value of the modified ciphertext byte XOR'd against 0x01. Yeah, that's a lot to take in. Here is an illustration of the process of conducting a POA that might help explain what you just read a little better:

![image](https://user-images.githubusercontent.com/82915527/166400772-811a03d2-a83a-4774-9eae-ca3a1bc64a05.png)

# Performing the Attack

In your terminal session, enter the following commands:

```
a = "This sentence cleaarly says I'M A LOSER."
original = encr(a)
print original.encode("hex")
```
The output should look like this:

![image](https://user-images.githubusercontent.com/82915527/166401442-c51e7314-e45f-493b-bef8-93af85465a63.png)

This ciphertext is what you are going to tinker with to eventually inject the word "WIN" into the plaintext. Neat.

Now, per the process of conducting a POA, you need to attempt any values that will render a valid padding check from the Oracle. To do this, you need to attempt this padding check 256 times (one guess for each possible byte) until the Oracle returns a match. Enter the following commands to do so (you may need to prese enter twice to receive output):

```
for i in range(256):
  mod = original[0:31] + chr(i) + original[32:]
  if decr(mod) != "PADDING ERROR":
    print i, "is correctly padded"
```

These commands run a for loop changing the last byte of the ciphertext[16:32] block for each byte representation. If the Oracle sees a correct padding scheme, it will return the original byte that returns the correct padding scheme. In this case, two padding schemes are correct.

![image](https://user-images.githubusercontent.com/82915527/166402015-1bd84961-e3c1-4c45-ac6e-fa6892b5f5ff.png)

Now of these two values, one of them will return correct padding, while the other one returns the final byte of 01. Since you modified the value for ciphertext[31] from the code you entered for the value ```mod```, the following command is what you will enter to see what the original ciphertext byte was:

```
print ord(original[31])
```

![image](https://user-images.githubusercontent.com/82915527/166402172-0174d3a0-a0e4-4c85-80b1-6a08bfe4de2c.png)

Although, this process is a bit inefficient. We want to be able to find a single correct padding scheme. Let's try filling the ciphertext block [16:31] with the following (you may have to press enter twice to receive output):

```
prefix = original[0:16] + "AAAAAAAAAAAAAAA"
for i in range(256):
  mod = prefix + chr(i) + original[32:]
  if decr(mod) != "PADDING ERROR":
    print i, "is correctly padded"
```

![image](https://user-images.githubusercontent.com/82915527/166402952-f65e00e6-a616-4f82-b605-ae2ea8286f42.png)

Here, the value of 147 is found. From filling ciphertext[16:31] with "A" characters and testing all 256 possibilites for ciphertext[31], only one value is returned. This is because the modified ciphertext creates random bytes of cleartext which won't end in valid padding unless the final byte is 01. In fact, filling ciphertext[16:31] with any values will return the correct value of 147. Try it with all "B"s instead.

## Calculating the Intermediate State

To complete a POA, an attacker must find the values of the intermediate state. The intermediate state is the part between the decryption and the XOR operation against the previous block.

To calcuate the intermediate state, refer to the diagram below to see how the last byte of plaintext is calculated:

![image](https://user-images.githubusercontent.com/82915527/166403550-3d0ff22b-ac38-46b0-a6de-a16f285d44bb.png)

So now for the fun part: logic gate math.

To make things easier to read, the character ```^``` will replace the XOR symbol. Based on the image above, it is known that ```ciphertext[31] ^ intermediate[47] = plaintext[47]```. We need to find the value of ```intermediate[47]```. Ready for some algebra?

We add ```ciphertext[31]``` to both side of the equation, resulting in ```intermediate[47] ^ ciphertext[31] ^ ciphertext[31] = cipertext[31] ^ plaintext[47]```.

On the left side, ```intermediate[47]``` is XOR'd twice, meaning those two ```ciphertext[31]``` on the left side of the equation cancel. This is due to the nature of how XOR works; it is its own inverse - XOR'ing twice with the same byte gets you back where you started. Now we have ```intermediate[47] = ciphertext[31] ^ plaintext[47]```.

We know that ``plaintext[47]`` must equal 1 since the Oracle returned a valid padding scheme, which turns the equation into ```intermediate[47] = ciphertext[31] ^ 1```. Do the XOR math to solve for ```intermediate[47]```, and you have ```intermediate[47] = 146```.


The idea of a padding oracle attack rests on this small hint given to the attacker. This demonstration is simply a demonstration to a complex attack involving cryptography, an already complext topic. This demonstration does not reflect how POAs are done in the real world, it just provides a conceptual foundation behind how they are done.
