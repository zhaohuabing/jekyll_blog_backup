---
layout:     post
title:      "Bitcoin and Cryptocurrency Technologies-Week 1"
subtitle:   "Hash Pointers and Data Structures"
date:       2018-05-12 10:00:00
author:     "赵化冰"
header-img: "img/in-post/2018-05-06-cryptocurrency_week1/bitcoin_header.jpg"
published: true
tags:
    - Cryptocurrency
    - Blockchain
    - Bitcoin
category: [ note ]

---

> This series of articles are my notes of "Bitcoin and Cryptocurrency Technologies" online course.

## Table of Content 
{:.no_toc}

* Table of Content 
{:toc}

## Hash Pointer
Hash Pointer is comprised of two parts:
* Pointer to where some information is stored
* Cryptographic hash of that information    
The pointer can be used to get the information, the hash can be used to verify that information hasn't been changed  

```
                           Hash Pointer
+----------------+      +----------------+
|                |   +--+Pointer of Data |
|                |   |  +----------------+
|                |   |  |  Hash of Data  |
|                |   |  +----------------+
|     Data       |   |
|                |   |
|                +<--+
|                |
|                |
+----------------+
```

## Build Data Structures with Hash Pointers
### Blockchain
Hash pointers can be used to build a linked list, which is also called a blockchain.

```
  Genines Block                                                                                      Head
+----------------+      +----------------+      +----------------+      +----------------+      +----------------+
|                |   +--+Pointer of Data4|   +--+Pointer of Data3|   +--+Pointer of Data2|   +--+Pointer of Data1|
|                |   |  +----------------+   |  +----------------+   |  +----------------+   |  +----------------+
|                |   |  |  Hash of Data4 |   |  |  Hash of Data3 |   |  |  Hash of Data2 |   |  |  Hash of Data1 |
|                |   |  +----------------+   |  +----------------+   |  +----------------+   |  +----------------+
|                |   |  |                |   |  |                |   |  |                |   |
|                |   |  |                |   |  |                |   |  |                |   |
|     Dtat4      +<--+  |     Dtat3      +<--+  |     Dtat2      +<--+  |     Dtat1      +<--+
|                |      |                |      |                |      |                |
|                |      |                |      |                |      |                |
+----------------+      +----------------+      +----------------+      +----------------+
```

