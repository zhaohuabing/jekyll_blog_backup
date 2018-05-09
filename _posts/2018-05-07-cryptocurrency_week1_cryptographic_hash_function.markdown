---
layout:     post
title:      "Lecture Notes: Bitcoin and Cryptocurrency Technologies-Week 1"
subtitle:   "Cryptographic Hash Function"
date:       2018-05-09 22:00:00
author:     "赵化冰"
header-img: "img/in-post/2018-05-06-cryptocurrency_week1/bitcoin_header.jpg"
published: hide
tags:
    - Cryptocurrency
    - Blockchain
    - Bitcoin
category: [ note ]
---

> This series of articles are my notes of "Bitcoin and Cryptocurrency Technologies" online course.
Cryptographic hash function

## Hash function
A mathematical function: hash(input string)=output

* input: any string
* output: a fix-size bit(Bitcoin uses 256)

## Cryptographic properties

A hash function which is used for cryptographic purposes should have these properties:

### collision free

It's impossible to find two inputs which can produce the same outputs. 

The collision does exist because the inputs can be any data and the outputs are only 2 to 256 possibilities. 

But for a good hash function, it's just impossible to find them in an acceptable time frame even use all the computers to solve this together on the earth.

We can use this property of hash functions to create a digest for a given data.  By comparing the hash digests, we can tell if a big file is modified or corrupted during a transmission, which is often used in downloading a software.

### Hiding

We want a hash function that It's infeasible to find out the input throughout the output, which is called the hiding property of hash functions.

The problem is that if there are only a few values of inputs, it will be very easy to figure out what the input is by the output by simply trying all the possible values of inputs and see if they match the output.

Solution: concatenate input with a random r which is chosen from a highly spread out distribution.  Hash(input\|r). Now it's infeasible to figure out what input is by traversing all the values because there're too many possibilities.

Ris used to hide the input, by using r, the Hash function can hide the input while exposing the output.

