+++
title = "Inverted Index"
date = "2024-02-19"
description = ""
+++

I’ve been working on inverted index these days. Hence, it’s a good time to do some summaries on it. I am going to split the notes into three parts: what is an inverted index, the possible file organization of it, and some of the optimizations to enhance its working efficiency.

## The Definition
As the name shows, it’s inverted, but what is the inversion meant to be? Let’s say we have a big set of documents and want to store them neatly. Each document is identified by an unique ID called DOCID. An index of such a whole bunch of data is the mappings from every single DOCID to its correspongding values. The value usually is a structured or semi-structured file which may contain the tokenized terms. Therefore, if we reverse it, the mappings will be from the terms to all the DOCIDs that contain the specific term. Although the structure of inverted index is faily simple, it’s been widely used in many sophisticated systems such as search engine and database.

## The File Organization
There are two basic parts of an inverted index: term dictionary and posting list. The dictionary helps locate the posting list of the designated term, where the posting list stores all the DOCIDs of the term. A basic possbile file organization of an inverted index looks like the following:

![file organization of inverted index](/images/tech/inv-idx-file-org.svg)

There are usually two files: a dictionary file that contains all the terms of the index and the corresponding offsets in a posting file. And, hence, the posting file is where the DOCID lists are stored. The files are split into blocks of the same size to be disk friendly. However, the structure is a vanilla one while the real implementation is way more complicated in the industry. Particularly, the skip list in the posting file is used to help traversing posting list efficiently. The posting list is sorted by DOCID by convention. Therefore, each item of the skip list captures at least the information of the last DOCID of a DOCID list block and the number of DOCIDs in that block. By leveraging the skip list, one can skip the unnecessary decoding of some file blocks when it only needs to seek some specific DOCID.

## Optimizations
There are many interesting optimizations on the performance of inverted index. For instance, Apache Lucene has done lots of relevant work on it: https://cwiki.apache.org/confluence/display/lucene/LucenePapers

### Cache
List caching: Disk IO is expensive, so caching is a good way get around it. This technique caches the block of posting list in memory so that when the frequently used terms are accessed, there won’t be any IOs to fetch the posting lists on disk.
Result caching: Ideally it’s possible as well to cache the result data at query layer. It has got a bigger granularity than list caching and it also requires no frequent updates upon the term.
Both of the two ideas are described in [1].

### Compression
As inverted index stores enormous amount of data, it usually consumes a lot of storage space. Therefore, an excellent compression algorithm will help saving hardware resources while, in the meantime, doesn’t hurt the query latency too much.

There are algorithms such as: Variable-Byte, Simple9/Simple16, and PForDelta in [1]. According to the reference paper, PForDelta is the optimal one.

### Smart Traversing
The common pattern of an inverted index query is first finding the right positing list. Then it is encapsulated as an iterator object with the ability to spit out a DOCID every time it gets invoked. Finally, the query starts to traverse the iterator. It usually passes a DOCID d_i to the iterator and expects it will return a DOCID d that is equal to or greater than d_i. When the iterator says there is no more documents, generally by a special DOCID, the whole process is finished.

If a posting list contains millions of documents, the whole searching process can be considerably slow. To encounter this inefficiency, some meta info are added to the file blocks of the posting list file, such as [2]. The paper introduces Block Max WAND (BMW) algorithm which stores the maximum score among all the documents in the current file block. The score can hint the query layer whether or not to decompress the block so that the decompression of lower scored blocks can be prevented and thus the query latency is improved.

## Reference
[1]. Performance of Compressed Inverted List Caching in Search Engines. Jiangong Zhang, Xiaohui Long, Torsten Suel. (2008)

[2]. Faster Top-k Document Retrieval Using Block-Max Indexes. Shuai Ding, Torsten Suel (2011)