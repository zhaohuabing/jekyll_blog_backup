---
layout:     post
title:      "Digital Signature"
subtitle:   "Bitcoin and Cryptocurrency Technologies-Week 1"
date:       2018-05-15 20:00:00
author:     "赵化冰"
header-img: "img/in-post/2018-05-06-cryptocurrency_week1/bitcoin_header.jpg"
published: true
tags:
    - Cryptocurrency
    - Blockchain
    - Bitcoin
    - Digital Signature
category: [ note ]

---

> This series of articles are my notes of "Bitcoin and Cryptocurrency Technologies" online course.

## Table of Content 
{:.no_toc}

* Table of Content 
{:toc}

## Digital Signature
Just like a written signature of a document, but it's in digital form. The desired features:    
* Only you can sign your own signature
* Everyone can verify your signature
* A signature is tied to a certain document, it can't be copied and used with other documents

## How to Sign and Verify a Digital Signature
### Generate a Pair of Public Key and Secrete/Private Key  
**(sk, pk) := generates(keysize)**   

The mathematics feature of public/private key pair:  Messages encrypted with private can only be decrypted with the public key, and vice versa.     
You keep the private key to yourself and publish the public key to others.    
Because only you have the private key, so if anyone who receives an encrypted message and the message can be decrypted with your public key, they can make sure that the message is sent from you. 
### Sign a Message with the Private Key     
**signature: = sign(sk, message)**        

We usually sign the hash of the message rather than the message itself.  
It's because singing a fixed size hash(such as 256 bit) is much more efficient than signing a long message, and the collision-free feature of hash function can assure that no one can forge another message which has the same hash.    
So the sign function is like:   
**signature := encrypt(sk, hash(message))**
### Verify the Signature with Message
Others receive a message with a signature which is claimed to be sent by the signer.   
They verify the message with the public key of the signer, which has already been published to all.   
**isValid := verify(pk, message, signature)** 
  
Like I mentioned before, we usually sign the hash of the message, so the verify function is like:    
**isValid := isEqual(decrypt(pk, hash(message)), signature)** 
![digital signature](\img\in-post\2018-05-12-cryptocurrency_week1_digital_signature\digital-signatures.jpg)

### The Trust Issue of the Public Key 
How can we make sure the public key we received has not been modified by a middleman or it is not forged by an attacker?    
To solve the trust issue in the public key publishing process, we use the digital certificate, which is a document consists of a public key, user identity and a signature of a trusted authority. The public key of the trusted authority has already been planted into operating systems or browsers, it's called root certificate.

Obtain a digital certifacte from an authority
![digital certification](\img\in-post\2018-05-12-cryptocurrency_week1_digital_signature\digital-certificate.png)    
Verify a digital signature using a certificate issued by an authority    
![digital certification](\img\in-post\2018-05-12-cryptocurrency_week1_digital_signature\verify-signature.jpg)    

## Use Digital Signature with Cryptocurrency
Signing a hash pointer is identical to signing the whole structure of the data in the hash pointer points to.    
* Sign the head hash pointer of a blockchain(LinkedList) is identical to sign all the transaction data in the blockchain
* Sign the root of a Merkle tree is identical to sign all the transaction  data in the Merkle tree

Explanation:   
Because modification of any part in the data structure will result inconsistent at the head/root, so as long as we have verified the digital signature of the head/root, we can know for sure that the whole structure can't be forged because no one can create a fake data structure with the same head/root.

## Example Codes on GitHub
* [Digital Signature example in Java](https://github.com/zhaohuabing/digital-signature)
