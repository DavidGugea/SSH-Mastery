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

# 7. SSH Keys

## Introduction

An SSH host key identifies a server. SSH also supports authenticating users with keys. Using keys to authenticate users requires more setup ahead of time than passwords, but when correctly done is both far more secure and much more convenient. We’ll consider both server and user keys.

## Manually Creating Server Keys

If an intruder compromises your server, the server’s private key is no longer private. You must replace it. This requires generating a new key pair. While most operating systems automatically create missing host keys, others don’t. Use ssh-keygen (1) to manually create server keys.

If your server runs a recent OpenSSH version, run ssh-keygen -A as root to automatically generate all supported but missing host keys.

You might need to create host keys for a host other than the local host, such as when deploying new installs using some orchestration systems. You can create key files manually using the -t and -f arguments to ssh-keygen.

```bash
$ ssh-keygen -t ecdsa -f ssh_host_ecdsa_key -N ''
$ ssh-keygen -t ed25519 -f ssh_host_ed25519_key -N ''
```

The -t flag specifies the type of key to create. Here we create two different types of keys, ECDSA, and ED25519. The -f flag gives the file name of the private key file. The public key for each key pair is in a file of the same name with .pub added to the end. Finally, -N lets you specify a passphrase on the command line. Host keys have no passphrase. The two single quotes indicate an empty passphrase.

Whenever you generate host keys, be sure to get the key fingerprints as discussed in chapter 4. Your users will need the fingerprints to verify the host keys.

## Passphrases

What’s this passphrase thing I just mentioned? A passphrase is like a password, but longer. It includes spaces, words, special characters, numbers, and anything else you can type. The passphrase is used to encrypt and decrypt the private key. A key with a passphrase cannot be used until someone enters the correct passphrase.

Passphrases are most often used with user authentication keys. A user with a key pair can access the system without providing a password for that system. Desktop and laptop systems are usually less secure than servers, and get infected, hijacked, or outright stolen depressingly often. If a user’s authentication key pair is stolen, the intruder can use that key pair to access servers just as if he was the legitimate user. Encrypting the private key with the passphrase means that even if the user’s private key file is stolen, the intruder cannot use the key without the passphrase. If an intruder gets either your private key file or your passphrase, but not both, the damage is contained. Make the passphrase too long to guess by brute force and sufficiently complex to discourage casual eavesdropping.

Can a passphrase be a single word, like a password? Yes, but it’s a really bad idea. Computers are now so fast that they can quickly discover short passwords by trying all possible passwords in succession. Using a short passphrase considerably reduces your private key’s security.

A passphrase should be at least several words long, something you can easily remember, and shouldn’t be obvious to others—even to people who know you. It should include special characters such as #, !, ~, and so on. Peculiar words from specialized non-computing vocabularies are useful. Substitute numbers for letters. Never use anything from pop culture, and never use any of your own personal catchphrases. Anything you’ve said to friends or coworkers that was catchy enough to repeat is a poor choice. If your imagination completely fails, Diceware (http://www.diceware.com) is a tool for randomly generating mostly-memorable passphrases from real words using ordinary dice. While intruders can ruin your week, a coworker with your private key and a sense of humor can be even more aggravating.

Host keys do not use passphrases, because the SSH service must start when the system boots. You could use a passphrase with the server key, but SSH would not start until someone entered the passphrase at the server console. This is unacceptable in most environments.

## User Keys

User key pairs provide stronger authentication than passwords. Combined with agents (see “SSH Agents” later this chapter), user keys eliminate the need to type any authentication credentials into remote machines. Cryptographically, user keys are identical to host keys. The only difference is where the keys are used.

Speaking very generally, a computer can identify you based on something you are, something you know, or something you have. Iris scanners and fingerprint readers verify your physical body, something you are. A password verifies that you know a secret. Getting into a house requires that you have the door key. Key-based authentication combines two of these: you must have the file containing the private key and you must know the passphrase for that key. Admittedly, a private key file is easier to reproduce than a physical key—it’s only copying the file—but it’s more difficult to reproduce than an 8-character password. This additional layer of security provides extra protection against unauthorized use of an account.

Keys are more complicated than passwords, however. Just as you wouldn’t leave your front door key hanging from the doorknob, you must protect your private keys. If the computer is lost or stolen, any private keys on that machine should be considered lost as well. While it’s possible to remember a password, most people won’t put in the time or energy to remember the thousands of characters in a private key. Yes, you should have backups… but if your laptop is stolen, the private keys on that laptop should be considered compromised anyway.

## SSH Agents

Replacing a password with a passphrase and a private key has one obvious flaw: typing passwords is an annoyance. Why replace an annoying password with an even more annoying passphrase? It might be more secure, but are you and your users really going to bother?

That’s where an SSH agent comes in. An SSH agent is a small program that runs in the background. When you start a desktop session, you enter your passphrase to decrypt your private key. The decrypted private key is loaded into the SSH agent. The agent stores the key in memory, never on disk. The agent processes all private key operations for the SSH client. When the SSH client needs to decrypt something with the private key, it asks the agent to handle it. When you log off for the day, the SSH agent shuts down. The decrypted private key disappears from memory. In other words, with an SSH agent, you type your passphrase once per work session, no matter how many SSH sessions you open that day.

On a typical day I log into my workstation, activate my SSH agent, and type my passphrase once. I then open innumerable SSH sessions to servers and routers all over my network, without typing a passphrase or password again. When I log off for the day, my agent shuts down. The memory used by the agent is wiped and returned to the operating system. My private key is once again available only in the encrypted file.

Agents do not guarantee security. Anyone who can read your computer’s memory while you are logged in can access the decrypted key. This includes the root account. If you don’t trust the system administrator on your desktop, don’t use an SSH agent.1 If you suspend your laptop, the decrypted private key remains in memory. Anyone who can wake your laptop and login can use the key as their access rights permit. A random thief interested in swapping your laptop for a quick buck probably won’t know or understand what he has, but a thief who is specifically targeting you and/or your employer will probably check for a live private key. More commonly, if you don’t lock your desktop before going to lunch, a coworker might take advantage of your unsecured terminals. These problems are best solved by emptying or shutting down your agent when you’re not actively using the system.

Agent security is also a problem on multiuser machines. Anyone who has administrative or superuser privileges on the system can access any SSH agent running on the host. If other people have root or Administrator access on your desktop, they can access your agent and masquerade as you. Using an agent would be unwise.

## Creating an OpenSSH User Key

If you have a Unix-like desktop, generate a key using ssh-keygen(1). Don’t use any arguments and the program will walk you through generating a user authentication key.

```bash
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/mwlucas/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/mwlucas/.ssh/id_rsa.
Your public key has been saved in /home/mwlucas/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:LK+1dKbb/PtN8KjiXLDdZzOK1fivRkMZIn8YoOrmEvA mwlucas@zfs1
The key's randomart image is:
+---[RSA 2048]----+
…
```

You’ll be asked where the new key should be saved. The various OpenSSH programs expect to find key files in the default locations, so take the suggestion. You’ll then be asked to enter a passphrase twice, to verify that you can type it more than once. Your private key will be encrypted with this passphrase. Always use a passphrase, as discussed just a few pages previous.

SSH uses identical key formats for hosts and users. When you generate a user key, you get a key fingerprint and a randomart image. Neither is particularly useful for user authentication keys.

You’ll find your new private key in $HOME/.ssh/id_rsa and your new public key in $HOME/.ssh/id_rsa.pub. Immediately backup your new key pair on offline media, such as a flash drive or CD-ROM. If you destroy your workstation, you’ll want the ability to recover your key pair.

## Key Algorithms

Like host keys, user keys can use a few different encryption algorithms. If you don’t specify an algorithm, the OpenSSH tools use the recommended one—at this time, 2048-bit RSA. You can specify a different algorithm with the -t flag.

```bash
$ ssh-keygen -t ecdsa
```

Why create multiple keys? Cryptographers have this distressing habit of finding weaknesses in cryptographic algorithms. One day the unthinkable will happen and someone will discover a flaw in a widely used and broadly trusted algorithm. All keys that use that algorithm will immediately become untrustworthy. If you have user keys with different algorithms, you can disable the broken algorithm on your SSH server and still have server access.

Our examples assume that you’re using an RSA key, but they’re just as applicable for keys made with other algorithms.