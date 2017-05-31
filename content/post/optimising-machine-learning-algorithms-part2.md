+++
categories = ["Development"]
date = "2017-05-25T16:45:40+01:00"
description = "Sparse matrix data structures and algorithms in Golang for machine learning and large data sets"
draft = true
tags = ["development", "machine learning", "go", "algorithms", "data structures"]
title = "Optimising algorithms in Go for machine learning - Part 2"
image = "/post/optimising-algorithms.jpg"
imageURL = "https://www.flickr.com/photos/derekgavey/5528275910"
imageTitle = "Algorithmic Contaminations"
imageCreator = "Derek Gavey"
imageLicenceURL = "https://creativecommons.org/licenses/by/2.0"
imageLicenceName = "CC BY 2.0"

+++

This is the second in a series of blog posts sharing my experiences benchmarking and optimising the algorithms and data structures for machine learning used whilst building out the [nlp project](http://github.com/james-bowman/nlp).  In the [previous post]({{< ref "optimising-machine-learning-algorithms.md" >}}) in this series, I explored alternative approaches for representing and applying [TF-IDF]({{< ref "semantic-analysis-of-webpages-with-machine-learning-in-go.md#tf-idf-term-frequency-inverse-document-frequency" >}}) transforms for weighting term frequencies across document corpora.  We tested our hypothesis using Go's inbuilt benchmark functionality and found that our optimisations materially improved not just memory consumption but also performance.  For a dataset with 30,000 terms across 3,000 documents, we showed the original TF-IDF implementation executed in around 41 seconds, consuming around 7 GB.  Following optimisations, we processed the same dataset in just 0.8 seconds, consuming around 250 KB of memory.  In this blog post I shall explore other areas for optimisation, seeking to further reduce memory consumption and processing time.

## Matrix Sparsity

Machine learning applications typically model entities as vectors of numerical features so that they may be compared and analysed quantitively.  When considered in a group, these vectors form a 2D grid or matrix (features x entities).  In the case of the [Latent Semantic Analysis (LSA)]({{< ref "semantic-analysis-of-webpages-with-machine-learning-in-go.md" >}}) implemented in the [nlp project](http://github.com/james-bowman/nlp), our documents are the entities and the words/terms they contain are the features.  The elements within the matrix represent the frequency that each word appears in the associated document.  [Refer to the previous blog post on LSA for more details]({{< ref "semantic-analysis-of-webpages-with-machine-learning-in-go.md" >}}).

The vocabulary of words occuring within a modestly sized document corpus of 3,000 documents could be hundreds of thousands of words.  However, each document probably only contains a couple of hundred unique words.  To represent such a corpus as numerical feature vectors would therefore require a ~200,000 x 3,000 matrix (terms x documents) with 99% of the elements containing zeros.  Storing all the elements of such a matrix would require 64 bits for each element (64 x 200,000 x 3,000) which would consume over 4GB of memory.

Thankfully, there are data structures and algorithms specifically designed for dealing with such sparse matrices that capitalise on the sparsity of the matrix by only storing the non-zero values.  This reduces the memory/storage requirements to represent the matrix and also potentially the processing effort to manipulate the data.

Sparse matrix data structures can effectively be divided into 3 main categories:

1. Creational - Sparse matrix formats suited to construction and random access updates of matrices.  Matrix formats in this category include [DOK (Dictionary Of Keys)]({{< ref "#dok-dictionary-of-keys-format" >}}) and [COO (COOrdinate sometimes referred to as Triplet format)]({{< ref "#coo-coordinate-format" >}}).

2. Operational - Sparse matrix formats suited to arithmetic operations e.g. multiplication or other operations requiring sequential access to elements.  Matrix formats in this category include [CSR (Compressed Sparse Row sometimes referred to as CRS (Compressed Row Storage))]({{< ref "#csr-compressed-sparse-row-format" >}}) and CSC (Compressed Sparse Column sometimes referred to as CCS (Compressed Column Storage))

3. Specialised - Specialised matrix formats optimised for specific sparsity patterns.  Matrix formats in this category include [DIA (DIAgonal)]({{< ref "#dia-diagonal-format" >}}) for efficiently storing and manipulating symmetric diagonal matrices.

A common practice is to construct sparse matrices using a creational format e.g. DOK or COO and then convert them to an operational format e.g. CSR for arithmetic operations.

### DOK (Dictionary Of Keys) format

DOK format uses a dictionary data structure (a map in Go) as its backing store, mapping row, column pairs (i, j) to matrix elements containing non-zero values.  Only non-zero values and their row/column index pairs are stored so any items missing from the map are assumed to be zero.  Using a hash map as the underlying data structure means random access (reads and writes) is relatively fast (_O(1)_) but sequential iteration over the elements is relatively slow making this format a good choice for incrementally constructing or updating a matrix but poor for arithmetic operations or other operations using sequential iteration.

### COO (COOrdinate) format

Also known as Triplet format, the COO format matrix stores the row and column indices of non-zero values along with the values themselves.  Each row index, column index and data value tuple is stored in 3 respective slices such that element(row[i], column[i]) = value[i].  Since the slices are unordered and duplicate elements are allowed, appending new non-zero elements to the end of the slices is a very fast operation (_O(1)_).  However, this also means that random access reads of elements is relatively slow (_O(n)_) and sequential iteration is also very slow.  These characteristics make this matrix format a good choice for initial construction of a matrix, adding new non-zero elements to an existing matrix or as an intermediate format for converting to CSR format but poor for arithmetic operations or other operations using sequential or random access to elements (assuming the slices are unsorted).

### CSR (Compressed Sparse Row) format

Also known as CRS (Compressed Row Storage), this format is similar to COO above except that the rows slice is compressed.  Relative to COO, the compression reduces storage requirements, allows faster random access of elements and row slicing but means making changes to the sparsity pattern are very slow (changing a zero value to non-zero).  These characteristics make this format a poor choice for random access updates, acceptable for random access reads and relatively good for arithmetic operations or other operations using sequential iteration. 

### DIA (DIAgonal) format



Updating NLP
Vectoriser - DOK

Benchmarks

COO good intermediate format for converting to CSR

CSR good for multiplication

DIA?

Benchmarks

Overall benchmarks

Futher work
Parallel fast matrix multiplication
SVD?
Hashing Vectoriser (the hashing trick) is now feasible


TF-IDF