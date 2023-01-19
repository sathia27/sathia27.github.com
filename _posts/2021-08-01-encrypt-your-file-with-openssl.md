---
layout: post
title: Encrypt your file with openssl
date: '2021-08-01'
categories: posts
published: true
tags: [openssl, crpyto, bash]
---

In today's world, we rely on multiple software like LastPass, Lockwise, Dashlane etc for managing our passwords or Securing our files. Most of this software has few limitations / or we might need to pay for their services. The cost would be based on Storage / Usage.

Consider if you have 30GB or more of data that you need to keep secure (encrypted), you might have to pay more money to one of these third-party services extra to keep them secure.

As a programmer, I don't want to pay for these services to keep my files secure. Also, That does not mean I need to implement the SSL protocol from scratch. I could use the OpenSSL library or any programming language which has OpenSSL library support. You can write your own wrapper for securing your information in the cloud.

OpenSSL command-line tool is a Swiss army knife for cryptographic tasks, testing and analyzing. It can be used for

1. Creating a secret key/password
2. creation of SSL certificates.
3. Encryption and decryption
etc.

### Wikipedia definitions on OpenSSL.

> **OpenSSL** is a software library for applications that secure communications over computer networks against eavesdropping or need to identify the party at the other end. It is widely used by Internet servers, including the majority of HTTPS websites. 

> OpenSSL contains an open-source implementation of the SSL and TLS protocols. The core library, written in the C programming language, implements basic cryptographic functions and provides various utility functions. Wrappers allowing the use of the OpenSSL library in a variety of computer languages are available. 

In the cryptography world, we should always consider open source for good security. Public security is always more secure than proprietary security.

Note: There are few free, open-source password managers like KeyPass which can also be considered.

There are many encryption algorithm strategies which is supported by OpenSSL.

### Symmetric vs Asymmetric

Symmetric encryption uses one key and Asymmetric encryption works with two keys. Asymmetric encryption is slower than symmetric encryption. It is impractical for encrypting larger files with Asymmetric algorithm. 

Some system uses both asymmetric and symmetric encryption. Asymmetric encryption for sharing secret keys between two parties, once two parties agreed upon the secret key then the rest of communication would be using Symmetric encryption.

For the scope of this blog, I have added command-line scripts for the RSA algorithm. RSA algorithm is an asymmetric cryptography algorithm.

### Few Openssl commands.
Below are a few commands that you can play around to encrypt or decrypt your files.

1. #### Create private key using `openssl`

	```bash
	openssl genrsa -aes128 -out sathia_private.pem 1024
	```
	This generates an RSA private key. you can find description for each option that can be passed [here](https://www.openssl.org/docs/man1.0.2/man1/genrsa.html)

2. #### Create public key using `openssl`

	```bash
	openssl rsa -in sathia_private.pem -pubout > sathia_public.pem
	```

    Public keys are generated using private keys. You can get to know more options for generating public key from [here](https://www.openssl.org/docs/man1.0.2/man1/openssl-rsa.html)

3. #### Encrypt your file
	
	Created new file, which will be encrypted using openssl
	```bash
	echo "tesing my key" > secret.txt
	```

	Generating new file `secret.enc` using the public key which was generated in Step 2. More details about options [here](https://www.openssl.org/docs/man1.0.2/man1/rsautl.html)
	```bash
	openssl rsautl -encrypt -inkey sathia_public.pem -pubin -in secret.txt -out secret.enc
	```

	Deleted un-encrypted file from file system.
	```bash
	rm secret.txt
	```

5. #### Decrypt your file
	
	Decrypting a file can be done only using a private key.

	```bash
	openssl rsautl -decrypt -inkey sathia_private.pem -in secret.enc
	```

References:
1. Openssl website
2. Wikipedia
