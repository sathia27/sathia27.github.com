---
layout: post
title: Encrypt your file with openssl
date: '2021-08-01'
categories: posts
published: true
tags: [openssl, crpyto, bash]
---

In today's world, we rely on multiple software like LastPass, Lockwise, Dashlane etc for managing our password or Securing our file. Most of these software have few limitations / or we might need to pay to their services. Cost would be based on Storage / Usage.

Consider if you have 30GB or more of data which you need to keep it secure (encrypted), you might have to more pay money to one of these third party services extra to keep them secure.

As a programmer, I don't want to pay for these services to keep my files secure. Also That does not mean I need to implement the SSL protocol from scratch. I can simply use OpenSSL library. Even all programing language have openssl library. If you have your cloud, you can take care of securing your file.

OpenSSL command line tool is Swiss army knife for cryptographic tasks, testing and analyzing. It can be used for

1. Creating secret key / password
2. creation of SSL certificates.
3. Encryption and decryption
etc.

### Wikipedia definitions on OpenSSL.

> **OpenSSL** is a software library for applications that secure communications over computer networks against eavesdropping or need to identify the party at the other end. It is widely used by Internet servers, including the majority of HTTPS websites. 

> OpenSSL contains an open-source implementation of the SSL and TLS protocols. The core library, written in the C programming language, implements basic cryptographic functions and provides various utility functions. Wrappers allowing the use of the OpenSSL library in a variety of computer languages are available. 

In the cryptography world, we should consider always open source for good security. Public security is always more secure than proprietary security.

Note: There are few free, open source password managers like KeyPass which can also be considered.

### Few Openssl commands.
Below are few commands that you can play around to encrypt or decrypt your files.

1. #### Create private key using `openssl`

	```bash
	openssl genrsa -aes128 -out sathia_private.pem 1024
	```
	This generates RSA private key. you can find description for each option that can be passed [here](https://www.openssl.org/docs/man1.0.2/man1/genrsa.html)

2. #### Create public key using `openssl`

	```bash
	openssl rsa -in sathia_private.pem -pubout > sathia_public.pem
	```

	Public keys are generated using private key. You can get to know more options for generating public key from [here](https://www.openssl.org/docs/man1.0.2/man1/openssl-rsa.html)

3. #### Encrypt your file
	
	Created new file, which will be encrypted using openssl
	```bash
	echo "tesing my key" > secret.txt
	```

	Generating new file `secret.enc` using public key which was generated in Step 2. More details about options [here](https://www.openssl.org/docs/man1.0.2/man1/rsautl.html)
	```bash
	openssl rsautl -encrypt -inkey sathia_public.pem -pubin -in secret.txt -out secret.enc
	```

	Deleted un-encrypted file from file system.
	```bash
	rm secret.txt
	```

5. #### Decrypt your file
	
	Decrypting file can be done only using private key.

	```bash
	openssl rsautl -decrypt -inkey sathia_private.pem -in secret.enc
	```

References:
1. Openssl website
2. Wikipedia
