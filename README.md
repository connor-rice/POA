# POA
Educational Padding Oracle Demonstration

This repository provides an educational look at the Padding Oracle attack discovered in 2002.

A Padding Oracle Attack (POA) is a cryptograhpically based attack that manipulates the AES encrpytion process to peek into the encrypted plaintext. To encrpyt plaintext, a key, an initialization vector, and the AES crypto suite is needed. The key is what would be used by the user wishing to encrypt a message. The plaintext would then be XOR'd against the initialization vector, with the key encryption following creating the first block of ciphertext. Since this attack works against CBC mode of AES, that ciphertext block is XOR'd against the second block of plaintext, with the key encryption following. This process is repeated until the whole plaintext is encrypted.

However, not all plaintext is 'the perfect length'. This is where the PKCS #7 encryption standard comes in. AES-CBC can only encrypt in full byte blocks. If a plaintext block does not fill in the multiple of the block size required by AES-CBC, PKCS #7 outlines how to 'pad' the rest of the incomplete byte. Hence the name 'Padding' Oracle attack.

# The Oracle

In most systems, an oracle is a system that checks the padding of ciphertext by returning an error. This hint provided by the oracle can help attackers decrypt the cipher text. This padding validation process is normally hidden to prevent such attacks, but for this demonstration, the hint will be provided. 
