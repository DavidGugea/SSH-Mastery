# SSH Mastery by Michael W. Lucas

## 0. Introduction
## 1. Encryption, Algorithms, and Keys
## 2. Common Configuration
## 3. The OpenSSH Server
## 4. Verifying Server Keys
## 5. SSH Clients
## 6. Copying Files over SSH
## 7. SSH Keys
## 8. X Forwarding
## 9. Port Forwarding
## 10. Keeping SSH Connections Open
## 11. Key Distribution
## 12. Automation
## 13. Virtual Private Networks
## 14. Certificate Authorities
## 15. OpenSSH Scraps

---
---

# 1. Encryption, Algorithms, and Keys

## Introduction to Encryption

OpenSSH encrypts traffic. What does that mean, and how does it work?

Encryption transforms readable plaintext into unreadable ciphertext that attackers cannot understand. Decryption reverses the transformation, producing readable text from apparent gibberish. An encryption algorithm is the exact method for performing this transformation. Most children discover the code that substitutes numbers for letters, so that A equals one, B equals two, Z equals 26, and so on. This is a simple encryption algorithm. Modern computer-driven encryption algorithms work on chunks of text at a time and perform far more complicated transformations.

Most encryption algorithms use a key; a chunk of text, numbers, symbols, or data used to encrypt messages. A key can be chosen by the user or randomly generated. (People habitually choose easily-guessed keys, so OpenSSH doesn’t even give users an option to create your own.) The encryption algorithm uses the key to encrypt the text, making it more difficult for an outsider to decrypt. Even if you know the encryption algorithm, you cannot decrypt the message without the secret encryption key.

Think of the encryption algorithm as a type of lock, and the key is a specific key. Locks come in many different types: house doors, bicycles, factories, and so on. Each uses a certain type of key—your door key is probably the wrong shape to fit into any vehicle ignition. But even a key of the proper type won’t work in the wrong lock. Your front door key unlocks your front door, and only your front door. Encryption keys work similarly.

## Algorithm Types

Encryption algorithms come in two varieties, symmetric and asymmetric.

***A symmetric algorithm*** uses the same key for both encryption and decryption. Symmetric algorithms include, but are not limited to, the Advanced Encryption Standard (AES) and ChaCha20, as well as older but now insecure algorithms like 3DES and Blowfish. A child’s substitution code is a symmetric algorithm. Once you know that A equals one and so on, you can encrypt and decrypt messages. Symmetric algorithms (more sophisticated than simple substitution) can be very fast and secure, so long as only authorized people have the key. And that’s the problem: an outsider who gets the key can read your messages or replace them with his own. You must protect the key. Sending a key unencrypted across the Internet is like standing on the playground shouting, “A is one, B is two.” Anyone who hears the key can read your private message.

***An asymmetric algorithm*** uses different keys for encryption and decryption. You encrypt a message with one key, and then decrypt it with another. This works because the keys are very large numbers, and multiplying very large numbers is much easier than figuring out how to divide them. (There are very good explanations out on the Internet, if you want the details.) Asymmetric encryption became popular only with the wide availability of computers that can handle the very difficult math, and is much, much slower and more computationally expensive than symmetric encryption.

Having two separate keys creates interesting possibilities. Make one key public. Give it away. Broadcast it to the entire world. Keep the other key private, and protected at all costs. Anyone who has the public key can encrypt a message that only someone who knows the private key can read. Someone who has the private key can encrypt a message and send it out into the world. Anyone can use the public key to decrypt that message, but the fact that the public key can decrypt the message assures recipients that the message sender had the private key. This is the basis of public key encryption. The public key and its matching private key are called a key pair. Again, think of the lock on your front door. The lock itself is public; anyone can touch it. The key is private. You must have both to get into your home. (You can learn more by researching Diffie-Hellman key exchange.)

## How SSH Uses Encryption

Symmetric encryption is fast, but offers no way for hosts to securely exchange keys. Asymmetric encryption lets hosts exchange public keys, but it’s slow and computationally expensive. How can you efficiently encrypt the session between two hosts that have never previously communicated?

Every SSH server has a key pair. Whenever a client connects, the server and the client use this key pair to negotiate a temporary key pair shared only between these two hosts. The client and the server both use this temporary key pair to derive a symmetric key that they will use to exchange data during this session, as well as related keys to provide connection integrity. If the session runs for a long time or exchanges a lot of data, the computers will intermittently negotiate a new temporary key pair and new symmetric key. The SSH protocol is more complicated than this, and include safeguards to prevent many different cryptographic attacks, but cryptographic key exchange is the heart of the protocol.

SSH supports many symmetric and asymmetric encryption algorithms. The client and server negotiate mutually agreeable algorithms at every connection. While OpenSSH offers options to easily change the algorithms supported and its preference for each, don’t! Programmers with more cryptography experience than both of us together arrived at OpenSSH’s encryption preferences after much hard thought, troubleshooting, and suffering. Gossip, rumor, and innuendo might crown Blowfish as the awesome encryption algorithm du jour, but that doesn’t mean you should tweak your OpenSSH server to use that algorithm and no other.

The most common reason people offer for changing the encryption algorithms is to improve speed. SSH’s primary purpose is security, not speed. Do not abandon security to improve speed. You might encounter a device that only speaks older encryption algorithms. We’ll cope with those in Chapter 15, “OpenSSH Scraps.”

Now that you understand how SSH encryption works, leave the encryption settings alone.