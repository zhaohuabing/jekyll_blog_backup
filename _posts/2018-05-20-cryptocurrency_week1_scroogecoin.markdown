---
layout:     post
title:      "Scrooge Coin"
subtitle:   "Bitcoin and Cryptocurrency Technologies-Week 1"
date:       2018-05-20 21:00:00
author:     "赵化冰"
header-img: "img/in-post/2018-05-06-cryptocurrency_week1/bitcoin_header.jpg"
published: true
tags:
    - Cryptocurrency
    - Bitcoin
category: [ note ]

---

> This series of articles are my notes of "Bitcoin and Cryptocurrency Technologies" online course.

## Table of Content 
{:.no_toc}

* Table of Content
{:toc}

Finally, I got to the most exciting part of week 1 lectures-the programming assignment!
 
I'm supposed to submit the assignment earlier because it was due a few weeks ago, however, I'd like to keep my pace relatively slow. I can't invest my full time to this course because I have a job to do, so I only take this course in my spare time. I also would like to digest all the information in one lesson before moving on to the next. Some fundamental technologies such as hash function, hash pointer, blockchain, Merkel tree and digital signature have been well-explained in week 1 lectures. In order to better understand these technologies, I also did some searches and programming practices, which can be found in my previous posts.  

It turns out that writing posts on my blog is a better way to learn, I have to fully understand the lessons before I can explain them in my posts.

## Scrooge Coin Transaction
Scrooge Coin programming assignment is a little bit tricky, the video of this lesson hasn't explained some implementation details. To help you understand the transaction data structure used in Scrooge Coin, I draw this diagram:
![Scrooge Coin](\img\in-post\2018-5-20-cryptocurrency_week1_scroogecoin\scroogecoin.png)

Every transaction has a set of inputs and a set of outputs. An input in a transaction must use a hash pointer to refer to its corresponding output in the previous transaction, and it must be signed with the private key of the owner because the owner needs to prove he/she agrees to spend his/her coins.  

Every output is correlated to the public key of the receiver, which is his/her ScroogeCoin address. 

In the first transaction, we assume that Scrooge has created 10 coins and assigned them to himself, we don't doubt that because the system-Scroogecoin has a building rule which says that Scrooge has right to create coins.

In the second transaction,  Scrooge transferred 3.9 coins to Alice and 5.9 coins to Bob. The sum of the two outputs is 0.2 less than the input because the transaction fee was 0.2 coin.

In the third transaction,  there were two inputs and one output, Alice and Bob transferred 9.7 coins to mike, and the transaction fee was 0.1 coin.

## Unclaimed transaction outputs pool
Another trick we need to note when doing the programming assignment is that an UTXOPool is introduced to track the unclaimed outputs (unspent coins), so we can know whether the corresponding output of an input of the transaction is available or not.

## Example Codes on GitHub
* [Scrooge Coin example in Java](https://github.com/zhaohuabing/scroogecoin)
