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

This is the second in a series of blog posts sharing my experiences benchmarking and optimising the algorithms and data structures for machine learning used whilst building out the [nlp](http://github.com/james-bowman/nlp) project.  In the [previous post]({{< ref "optimising-machine-learning-algorithms.md" >}}) in this series, I explored alternative approaches for representing and applying a TF-IDF transform to apply weightings to vectors of raw term frequencies for a corpus of documents.  We tested our hypothesis using Go's inbuilt benchmark functionality and found that our optimisations materially improved not just memory consumption but also performance.  For a dataset with 30,000 terms across 3,000 documents, we showed the original TF-IDF implementation executed in around 41 seconds, consuming around 7 GB.  Following optimisations, we processed the same dataset in just 0.8 seconds, consuming around 250 KB of memory.  In this blog post I shall explore further areas for optimisation, seeking to reduce memory consumption and processing time.

## Matrix Density

Machine learning applications typically model entities as vectors of numerical features so that they may be compared and analysed quantitively.  These vectors can be combined to form a 2D grid or matrix (features x entities).  In the case of the Latent Semantic Analysis (LSA) implemented in the [nlp](http://github.com/james-bowman/nlp) project, our documents are the entities and the words/terms they contain are the features.  The elements within the matrix represent the frequency that each word appears in the associated document.  [Refer to the previous blog post on LSA for more details]({{< ref "semantic-analysis-of-webpages-with-machine-learning-in-go.md" >}}).

The vocabulary of words occuring within a modestly sized document corpus of 3,000 documents could be hundreds of thousands of words.  However, each document probably only contains a couple of hundred unique words.  To represent such a corpus as numerical feature vectors would therefore require a ~200,000 x 3,000 matrix (terms x documents) with only as little as 1% of the elements containing non-zero values.  Storing all the elements of such a matrix would require 64 bits for each element (64 x 200,000 x 3,000) which would require over 4GB of memory.

There are published data structures and algorithms for dealing with such sparse matrices implementations that capitalise on the sparsity of the matrix by only storing the non-zero values.  This reduces the memory/storage requirements to represent the matrix and also potentially the processing effort to manipulate the data.

Sparse matrix data structures can effectively be divided into 3 main categories:

1. Creational - Sparse matrix formats suited to construction and random access updates of matrices.  Matrix formats in this category include DOK (Dictionary Of Keys) and COO (COOrdinate sometimes referred to as Triplet format).

2. Operational - Sparse matrix formats suited to arithmetic operations e.g. multiplication or other operations requiring sequential access to elements.  Matrix formats in this category include CSR (Compressed Sparse Row sometimes referred to as CRS (Compressed Row Storage)) and CSC (Compressed Sparse Column sometimes referred to as CCS (Compressed Column Storage))

3. Specialised - Specialised matrix formats optimised for specific sparsity patterns.  Matrix formats in this category include DIA (DIAgonal) for efficiently storing and manipulating symmetric diagonal matrices.

A common practice is to construct sparse matrices using a creational format e.g. DOK or COO and then convert them to an operational format e.g. CSR for arithmetic operations.

### DOK (Dictionary Of Keys) format

DOK format uses a dictionary data structure (using a map in Go) as its backing store, mapping row, column pairs (i, j) to elements containing non-zero values.  Any keys (row, column pairs) missing from the map are assumed to be zero.

``` Go
type key struct {
	i, j int
}

type DOK struct {
	r        int
	c        int
	elements map[key]float64
}

func (d *DOK) At(i, j int) float64 {
	return d.elements[key{i, j}]
}
```

### COO (COOrdinate) format

Also known as Triplet format, the COO format matrix stores the row and column indices of non-zero values along with the values themselves.  Each row index, column index and data value tuple is stored in 3 slices such that element(row[i], column[i]) = value[i].  The slices can be sorted for faster access times.

``` Go
type COO struct {
	r             int
	c             int
	rows          []int
	cols          []int
	data          []float64
}

func (c *COO) At(i, j int) float64 {
	result := 0.0
	for k := 0; k < len(c.data); k++ {
		if c.rows[k] == i && c.cols[k] == j {
            // as duplicate elements are allowed in COO format, sum them together
			result += c.data[k]
		}
	}

	return result
}
```

### CSR (Compressed Sparse Row) format

Also known as CRS (Compressed Row Storage), this format is similar to COO above except that the rows slice is compressed.  The compression system provides the following important differences COO format:

1. Reduced storage requirements - rather than storing the row index of every non-zero value (length = NNZ (the Number of Non-Zero values)), the cumulative number of non-zero values is stored for each matrix row (length = m (the number of matrix rows)) such that it can be used as an offset into the column index or data value slices.  This will use less memory if m < NNZ i.e. assuming that on average, each row contains more than one non-zero value.
2. Faster random access of elements - with COO format, elements within a row can only be found by iterating through all non-zero values.  With CSR format, elements within a specific row can be found by looking at the corresponding value in the compressed row pointer slice (indptr) to find the offset into the column index slice for that row (see At() method below).
3. Slower changes to sparsity pattern (changes of zero values to non-zero values or vice versa) - due to the way the compressed row pointer slice is formatted with cumulative counts, changing the number of non-zero values on a row requires subsequent row counts to be updated too.

``` Go
type CSR struct {
	i, j   int
	indptr []int
	ind    []int
	data   []float64
}

func (c *CSR) At(i, j int) float64 {
	for k := c.indptr[i]; k < c.indptr[i+1]; k++ {
		if c.ind[k] == j {
			return c.data[k]
		}
	}

	return 0
}
```

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