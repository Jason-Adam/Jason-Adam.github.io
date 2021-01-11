---
layout: post  
mathjax: true  
comments: true  
title: Data Structures of Databases  
tags: [Data Structures]  
---  
What are the data structures that power our databases?  

I've admittedly become minorly obsessed with studying data structures and algorithms lately, and I've decided to pair that study with my second read through of Martin Kleppmann's book, *Designing Data-Intensive Applications* [1]. The book is chalk full of great material as it relates to databases, application design, and distributed systems. Anyways, the reason I decided to pair these two courses of study is due to the fact that I see a large volume of folks online that study algorithms and data structures through the singular focus of a site like LeetCode, and I've always felt like the concepts don't stick unless you see them in action. A good example of this relates to how different databases work. In an algorithms book one will most likely encounter trees at some point. Binary Trees, Binary Search Trees (BSTs), etc will be taught, their Big O time space complexity for traversal will be memorized, and all will be right in the world. However, many people write SQL queries every day and don't understand how they can retrieve data so quickly due to it's declarative nature. This leads me to the point of this post. I aim to simplify and condense some of Kleppmann's work in his book to perhaps shed some light on how powerful these data structures are and how they are the backbone of data retrieval for most modern day databases.  

I'll focus on the 3 main indexes described in the book:  
* Hash Indexes  
* LSM-Trees  
* B-Trees  

## Hash Indexes  
Most of us are familiar with the swiss army knife of data structures, the [hash map](https://jason-adam.github.io/hash-tables/). It boasts O(1) lookup time which makes it an excellent choice for things like simple caching, de-duplicating records, and just about anything else we want. Kleppmann describes a simple example in the book whereby we keep an in-memory hash map where the key is mapped to a byte offset in the data file [1]. This translates to the location in which a record is found. We can leverage the lightning fast lookup time to quickly retrieve a record from our file, and voila, we're done. This setup is essentially how [BitCask](https://github.com/basho/bitcask) works (the default storage engine for the NoSQL database [Riak](https://riak.com/index.html)). Below is an illustration of this concept from Kleppmann's book.  

![](../imgs/2021-01-10-db-data-structures/hash-index.png)  

The main limitation of the hash index is its need to be in memory. If you have a small number of keys, and you don't require any sort of range queries, then they can be a very viable choice. Range queries are inefficient due to needing to retrieve each individual key within the range.

## SSTables & LSM-Trees  
Apart from the hash map, I find graphs and trees (just a type of graph) to be some of the coolest data structures. I think most people can intuit how to traverse an array or linked list due to linear visual they evoke, but they fall short in usefullness when we start talking about indexing a database (all the cool kids study graphs).  

In the first example I purposely left out the data being indexed. It was an append only log file (zzzzzz.... boring), but I'll state it briefly to set the stage for the next portion. SSTable stands for *Sorted String Table*. This differs from an append only log file by requiring the sequence of key-value pairs to sorted (woah, mind blowing!). They have several advantages over log segments with hash indexes, and I'll do my best to condense down the material into a human readable format [1]:  

1. Merging segments is simple and efficient (think mergesort -> I smell a future post coming). Below is the visual from the book whereby we see several segments compacted and merged.  

![](../imgs/2021-01-10-db-data-structures/sstable-compact-merge.png)  

2. You no longer need to keep all keys in memory! This is due to the sorted nature. We can now keep some of the keys in our in-memory hash map with their offsets and find records near them by retrieving their offset, then scanning until we find what we need. This is displayed below.  

![](../imgs/2021-01-10-db-data-structures/sstable-memory-index.png)  

3. Since read requests need to scan several key-value pairs in a range, those records can be grouped and compressed before writing to disk [1]. This concept is displayed above in the grayed out compressible block section (along the right).

### Reference  
[1] Kleppmann, Martin. Designing Data-Intensive Applications: The Big Ideas Behind Reliable, Scalable, and Maintainable Systems. 1st ed., O’Reilly Media, 2017.
