+++
categories = ["Development"]
date = "2017-06-09T16:45:40+01:00"
description = "Comparing sparse matrix data structures and algorithms in Golang for machine learning and large data sets"
draft = false
tags = ["development", "machine learning", "go", "algorithms", "data structures"]
title = "Optimising algorithms in Go for machine learning - Part 2"

image = "/post/programming-942487_1280.jpg"

+++

This is the second in a series of blog posts sharing my experiences working with algorithms and data structures for machine learning.  These experiences were gained whilst building out the [nlp project](http://github.com/james-bowman/nlp) for [LSA (Latent Semantic Analysis)]({{< ref "semantic-analysis-of-webpages-with-machine-learning-in-go.md" >}}) of text documents.  

In the [previous post]({{< ref "optimising-machine-learning-algorithms.md" >}}) in this series, I explored alternative approaches for representing and applying [TF-IDF]({{< ref "semantic-analysis-of-webpages-with-machine-learning-in-go.md#tf-idf-term-frequency-inverse-document-frequency" >}}) transforms for weighting term frequencies across document corpora.  We tested the approaches using Go's inbuilt benchmark functionality and found that our optimisations materially improved not just memory consumption but also performance (reducing memory consumption and processing time from 7 GB and 41 seconds to 250 KB and 0.8 seconds respectively).  In this blog post I shall explore other areas for optimisation, seeking to further reduce memory consumption and processing time.

## Matrix Sparsity

Machine learning applications typically model entities as vectors of numerical features so that they may be compared and analysed quantitively.  These vectors can be considered collectively as a matrix.  In the case of [Latent Semantic Analysis (LSA)]({{< ref "semantic-analysis-of-webpages-with-machine-learning-in-go.md" >}}) as implemented in the [nlp project](http://github.com/james-bowman/nlp), documents are the entities and the words/terms they contain are the features.  The elements within the matrix represent the frequency that each word appears in the associated document.  [For more detailed explanation of term document matrices and LSA please refer to this blog post]({{< ref "semantic-analysis-of-webpages-with-machine-learning-in-go.md" >}}).

The vocabulary of words occuring within a modestly sized document corpus of 3,000 documents could be hundreds of thousands of words.  However, each document probably only contains a couple of hundred unique terms.  To represent such a corpus as numerical feature vectors would therefore require a ~200,000 x 3,000 matrix (terms x documents) with 99% of the elements containing zeros.  Storing all the elements of such a matrix, using 64 bits per element, would require over 4GB of memory (64 x 200,000 x 3,000).

Thankfully, there are data structures and algorithms specifically designed for dealing with such sparse matrices that capitalise on the sparsity of the matrix by only storing the non-zero values.  This reduces the memory/storage requirements and processing effort to represent and process the matrix.

## Sparse Matrix Formats

Sparse matrix data structures can effectively be divided into 3 main categories:

1. __Creational__ - Sparse matrix formats suited to construction and random access updates of matrices.  Matrix formats in this category include [DOK (Dictionary Of Keys)]({{< ref "#dok-dictionary-of-keys-format" >}}) and [COO (COOrdinate)]({{< ref "#coo-coordinate-format" >}}).
1. __Operational__ - Sparse matrix formats suited to arithmetic operations e.g. multiplication or other operations requiring sequential access to elements.  Matrix formats in this category include [CSR (Compressed Sparse Row)]({{< ref "#csr-compressed-sparse-row-format" >}}) and [CSC (Compressed Sparse Column)]({{< ref "#csc-compressed-sparse-column-format" >}}).
1. __Specialised__ - Specialised matrix formats optimised for specific sparsity patterns.  Matrix formats in this category include [DIA (DIAgonal)]({{< ref "#dia-diagonal-format" >}}) for efficiently storing and manipulating symmetric diagonal matrices.

A common practice is to construct sparse matrices using a creational format e.g. DOK or COO and then convert them to an operational format e.g. CSR for arithmetic operations.

### 1. Creational Formats

#### DOK (Dictionary Of Keys) format

DOK format uses a dictionary data structure (a map in Go) as its backing store, mapping row/column pairs (i, j) to matrix elements containing non-zero values.  Only non-zero values and their row/column index pairs are stored so any items missing from the map are assumed to be zero.  Using a hash map as the underlying data structure means random access (reads and writes) is relatively fast (_O(1)_) but sequential iteration over the elements is relatively slow making this format a good choice for incrementally constructing or updating a matrix but poor for arithmetic operations.

#### COO (COOrdinate) format

Also known as Triplet format, the COO format matrix stores the row and column indices of non-zero values along with the values themselves.  Each row index, column index and data value tuple is stored in 3 respective slices such that `element(row[i], column[i]) = value[i]`.  Since the slices are unordered and duplicate elements are allowed, appending new non-zero elements to the end of the slices is a very fast operation (_O(1)_).  However, this also means that random access reads of elements is relatively slow (_O(n)_) and sequential iteration can also be slow (sorting the slices can improve access times).  These characteristics make this matrix format a good choice for initial construction of a matrix, adding new non-zero elements to an existing matrix or as an intermediate format for converting to CSR format but poor for arithmetic operations (assuming the slices are unsorted).

### 2. Operational Formats

#### CSR (Compressed Sparse Row) format

Also known as CRS (Compressed Row Storage), this format is similar to COO above except that the row index slice is compressed.  Specifically, the row index slice stores the cumulative count of non-zero elements in each row such that `row[i]` contains the index into both `column[]` and `data[]` of the first non-zero element of row `i`.  Thus ranging across data from `data[row[i]]` to `data[row[i+1]-1]` will yield all values within row `i`.  For a more detailed explanation of CSR format please refer [here](https://en.wikipedia.org/wiki/Sparse_matrix#Compressed_sparse_row_.28CSR.2C_CRS_or_Yale_format.29).

Relative to COO, the compression reduces storage requirements, allows faster random access reads of elements and row slicing but means making changes to the sparsity pattern is very slow (changing a zero value to non-zero).  These characteristics make this format a poor choice for random access updates, acceptable for random access reads and relatively good for arithmetic operations.

#### CSC (Compressed Sparse Column) format

Also known as CCS (Compressed Column Storage), this format is identical to CSR above except that the column index slice is compressed rather than the row index slice as with CSR.  The result is that CSC naturally stores values in column major order rather than row major order as with CSR.  CSC can be thought of as a natural transpose of CSR.

### 3. Specialised Formats

#### DIA (DIAgonal) format

DIA is a specialised format for storing symmetric diagonal matrices.  Symmetric diagonal matrices are square shaped and so have the same number of rows and columns, with only the values along the diagonal (top left to bottom right) having non-zero values.  The DIA format takes advantage of this fact by only storing the diagonal values which it stores in a single slice such that `element(i, j) = value[j] or 0 if i != j`.

## Putting it into Practice

Our implementation of Latent Semantic Analysis in the nlp project comprises 3 steps as follows:

1. [__Feature Extraction/Vectorisation__]({{< ref "#1-feature-extraction-vectorisation" >}}) - Converting raw text documents into vectors of numerical features to create a term document matrix
1. [__TF/IDF Weighting__]({{ ref "#2-tf-idf-weighting" }}) - Extracting weighting values based upon values in the term document matrix and then applying them to the matrix.
1. [__Truncated Singular Value Decomposition__]({{ ref "#3-truncated-singular-value-decomposition" }}) - reducing the dimensionality (to the top k most significant dimensions) of the weighted term document matrix.

Currently, each of these 3 steps outputs a dense matrix.  In addition, the TF-IDF weightings extracted and applied in step 2 are stored as a dense matrix to be multiplied with the term document matrix.

### 1. Feature Extraction/Vectorisation

Step 1 constructs a matrix incrementally based upon the terms encountered whilst parsing the document corpus.  We believe the matrix will be very sparse as most documents will only contain a couple of hundred unique terms from a corpus wide vocabulary of hundreds of thousands of terms.  The current dense matrix implementation will be very wasteful storing all of the zero values in addition to the non-zero values so a creational sparse matrix format may be the most appropriate format to use.  Specifically, the DOK (Dictionary Of Keys) format is most suited in this case as the matrix is incrementally constructed.

Here is a code snippet showing the current feature extraction using a Dense format matrix.

``` Go
func (v *CountVectoriser1) Transform(docs ...string) (*mat64.Dense, error) {
	mat := mat64.NewDense(len(v.Vocabulary), len(docs), nil)

	for d, doc := range docs {
		words := v.tokenise(doc)

		for _, word := range words {
			i, exists := v.Vocabulary[word]

			if exists {
				mat.Set(i, d, mat.At(i, d)+1)
			}
		}
	}
	return mat, nil
}
```

And here is the same code modified to use a Dictionary Of Keys (DOK) sparse format matrix.

``` Go
func (v *DOKCountVectoriser1) Transform(docs ...string) (*sparse.DOK, error) {
	mat := sparse.NewDOK(len(v.Vocabulary), len(docs))

	for d, doc := range docs {
		words := v.tokenise(doc)

		for _, word := range words {
			i, exists := v.Vocabulary[word]

			if exists {
				mat.Set(i, d, mat.At(i, d)+1)
			}
		}
	}
	return mat, nil
}
```

As you can see there is little difference to the code beyond which matrix type is constructed.  To test the two versions I used Go's built in benchmark functionality and a subset of the 20 Newsgroups dataset as used in Scikit Learn.  The subset of the dataset used for this benchmark comprises 2,001 documents representing a vocabulary of 33,552 unique terms.  This results in a a term document matrix of size 33,552 x 2,001 with only 385,944 non-zero values (99.5% of the matrix elements contain zero values).  Here are the two benchmark functions.

``` Go
// Benchmark feature extraction vectorisation into Dense vs Sparse matrices

// Baseline Dense matrix vectorisation
func BenchmarkDenseCountVectoriserTransform(b *testing.B) {
	files := Load("sci.space", "sci.electronics")

	vect := NewCountVectoriser1(false)
	vect.Fit(files...)

	b.ResetTimer()
	for n := 0; n < b.N; n++ {
		vect.Transform(files...)
	}
}

// Benchmark DOK Sparse matrix vectorisation
func BenchmarkDOKCountVectoriserTransform(b *testing.B) {
	files := Load("sci.space", "sci.electronics")

	vect := NewDOKCountVectoriser1(false)
	vect.Fit(files...)

	b.ResetTimer()
	for n := 0; n < b.N; n++ {
		vect.Transform(files...)
	}
}
```

The results are shown below.  It should be clear from the results that the new Sparse matrix implementation is slightly faster than the Dense matrix based implementation at 0.8 seconds vs 1.1 but most importantly uses considerably less memory (80 MB vs the 500 MB used by the Dense implementation).

```
Jamess-MacBook-Pro:nlpbench jbowman$ go test -bench=CountVectoriserTransform -benchmem
BenchmarkDenseCountVectoriserTransform-4  1	  1149735130 ns/op	 588033600 B/op	  688868 allocs/op
BenchmarkDOKCountVectoriserTransform-4    2	  849252150 ns/op	 83780632 B/op	  712011 allocs/op
PASS
ok  	github.com/james-bowman/nlpbench	6.640s
```

### 2. TF/IDF Weighting

In the [previous post]({{< ref "optimising-machine-learning-algorithms.md" >}}) in this series, we identified the TF-IDF weightings matrix to be a symmetric diagonal matrix and so switched to storing it as a simple slice containing just the diagonal values.  This resulted in material improvements to both storage requirements and processing time.  We will now switch back to using a matrix for the TF-IDF weightings but this time, rather than using a dense matrix as before, we will use the appropriate DIAgonal sparse matrix format.

Step 2, takes the term document matrix constructed during step 1, extracts weighting values and now stores them in a DIAgonal matrix and then multiplies it by the term document matrix.  The resulting product will have the same sparsity pattern of non-zero values as the input term document matrix.  As the product will clearly therefore also be a sparse matrix and will be the result of an arithmetic operation we should consider the CSR (Compressed Sparse Row) sparse matrix format for this matrix.  We should also consider converting the input matrix to CSR prior to the arithmetic operation.

Here is a code snippet showing the current TF-IDF using a Dense format matrix and a slice for the tf-idf weighting values.

``` Go
type TfidfTransformer3 struct {
	weights []float64
}

func (t *TfidfTransformer3) Fit(mat mat64.Matrix) Transformer {
	m, n := mat.Dims()

	t.weights = make([]float64, m)

	for i := 0; i < m; i++ {
		df := 0
		for j := 0; j < n; j++ {
			if mat.At(i, j) != 0 {
				df++
			}
		}
		idf := math.Log(float64(1+n) / float64(1+df))
		t.weights[i] = idf
	}

	return t
}

func (t *TfidfTransformer3) Transform(mat mat64.Matrix) (*mat64.Dense, error) {
	m, n := mat.Dims()
	product := mat64.NewDense(m, n, nil)

	// apply a function to every element of the matrix in turn which
	// multiplies the element value by the corresponding term weight
	product.Apply(func(i, j int, v float64) float64 {
		return (v * t.weights[i])
	}, mat)

	return product, nil
}

func (t *TfidfTransformer3) FitTransform(mat mat64.Matrix) (*mat64.Dense, error) {
	return t.Fit(mat).Transform(mat)
}
```

And here is the same code modified to use sparse format matrices for both the TF-IDF weighting values (DIAgonal format) and (CSR - Compressed Sparse Row format) for the input and output matrices.

``` Go
type SparseTfidfTransformer struct {
	transform mat64.Matrix
}

func (t *SparseTfidfTransformer) Fit(mat mat64.Matrix) *SparseTfidfTransformer {
	m, n := mat.Dims()

	weights := make([]float64, m)

	csr, isCsr := mat.(*sparse.CSR)

	for i := 0; i < m; i++ {
		df := 0
		if isCsr {
			// if matrix is CSR, take advantage of the RowNNZ() method to get 
			// number of documents in which term appears.
			df = csr.RowNNZ(i)
		} else {
			for j := 0; j < n; j++ {
				if mat.At(i, j) != 0 {
					df++
				}
			}
		}
		idf := math.Log(float64(1+n) / float64(1+df))
		weights[i] = idf
	}

	// build a diagonal matrix from array of term weighting values for subsequent
	// multiplication with term document matrics

	t.transform = sparse.NewDIA(m, weights)

	return t
}

func (t *SparseTfidfTransformer) Transform(mat mat64.Matrix) (mat64.Matrix, error) {
	product := &sparse.CSR{}

	// simply multiply the matrix by our idf transform (the diagonal matrix of term 
	// weights)
	product.Mul(t.transform, mat)

	return product, nil
}

func (t *SparseTfidfTransformer) FitTransform(mat mat64.Matrix) (mat64.Matrix, error) {
	return t.Fit(mat).Transform(mat)
}
```

This time, there is a little more difference between the 2 implementations.  If the input matrix is CSR format we take advantage of the RowNNZ() method to quickly determine the number of non-zero values in the row which is used to calculate the inverse document frequency (the number of documents each term occurs in).

To test the two versions I used Go's built in benchmark functionality and the same subset of the 20 Newsgroups dataset as used in Scikit Learn.  Here are the two benchmark functions.

``` Go
func BenchmarkDenseTfidfFitTransform(b *testing.B) {
	files := Load("sci.space", "sci.electronics")

	vect := NewCountVectoriser1(false)
	vect.Fit(files...)
	mat, _ := vect.Transform(files...)

	trans := &TfidfTransformer3{}

	b.ResetTimer()
	for n := 0; n < b.N; n++ {
		trans.FitTransform(mat)
	}
}

func BenchmarkSparseTfidfFitTransform(b *testing.B) {
	files := Load("sci.space", "sci.electronics")

	vect := NewDOKCountVectoriser1(false)
	vect.Fit(files...)
	mat, _ := vect.Transform(files...)

	trans := &SparseTfidfTransformer{}

	b.ResetTimer()
	for n := 0; n < b.N; n++ {
		trans.FitTransform(mat.ToCSR())
	}
}
```

The results are shown below.  It should be clear from the results that the new Sparse matrix implementation is considerably faster than the Dense matrix based implementation at 0.1 seconds vs 2.2 seconds but most importantly uses considerably less memory (16 MB vs the 500 MB used by the Dense implementation).

```
Jamess-MacBook-Pro:nlpbench jbowman$ go test -bench=TfidfFitTransform -benchmem
BenchmarkDenseTfidfFitTransform-4   1	2238133519 ns/op	537378960 B/op	       4 allocs/op
BenchmarkSparseTfidfFitTransform-4  10	 138069814 ns/op	16253248 B/op	      12 allocs/op
PASS
ok  	github.com/james-bowman/nlpbench	8.277s
```

### 3. Truncated Singular Value Decomposition

Step 3 outputs a matrix of much smaller dimensionality but as this matrix contains only the most significant dimensions, it is likely to be very dense.  Therefore, a dense matrix format is still the most appropriate choice of format for this step and so no changes will be made to this step at this point.

## Putting it all together

To test the impact of our changes to steps 1 & 2 above together we will run the following benchmark functions.

``` Go
func BenchmarkDenseEndToEndVectAndTrans(b *testing.B) {
	files := Load("sci.space", "sci.electronics")

	b.ResetTimer()

	vect := NewCountVectoriser1(false)
	trans := &TfidfTransformer3{}

	for n := 0; n < b.N; n++ {
		mat, _ := vect.FitTransform(files...)
		trans.FitTransform(mat)
	}
}

func BenchmarkSparseEndToEndVectAndTrans(b *testing.B) {
	files := Load("sci.space", "sci.electronics")

	b.ResetTimer()

	vect := NewDOKCountVectoriser1(false)
	trans := &SparseTfidfTransformer{}

	for n := 0; n < b.N; n++ {
		mat, _ := vect.FitTransform(files...)
		trans.FitTransform(mat.ToCSR())
	}
}
```

The results are shown below.  We can see that the current dense implementation of steps 1 & 2(with the optimisations made in the [previous post in this series]({{< ref "optimising-machine-learning-algorithms.md" >}})) took 3.3 seconds to complete and consumes over 1 GB of memory.  In contrast, the new sparse based implementation takes only 1.2 seconds to complete and consumes only 150 MB of memory.  

```
Jamess-MacBook-Pro:nlpbench jbowman$ go test -bench=EndToEndVect -benchmem
BenchmarkDenseEndToEndVectAndTrans-4    1	3373285711 ns/op	1180200344 B/op	 1379753 allocs/op
BenchmarkSparseEndToEndVectAndTrans-4   1	1232713457 ns/op	154805032 B/op	 1402819 allocs/op
PASS
ok  	github.com/james-bowman/nlpbench	5.110s
```

## Wrapping Up

Using Sparse matrix formats in the place of Dense formats can significantly reduce both memory consumption and processing time.  Not necessarily because the algorithms are cleverer but simply because they are doing less work by processing only the non-zero elements.  Switching to sparse matrix formats, we saw a reduction in memory consumption and processing time from 1 GB to 150 MB and 3.3 seconds to 1.2 seconds respectively.

The sparse matrix format implementations used in this article are [available on Github](http://github.com/james-bowman/sparse) along with all the [benchmarks and sample code](http://github.com/james-bowman/nlp-bench) used in this article.  The sparse matrix library is still quite basic in terms of features and available operations and I am hoping to extend it in due course with other operations (e.g. add, subtract, etc.) and optimisations (e.g. parallel matrix multiplication, BLAS integration, etc.).

I would love to hear other people's experiences of using or developing dense or sparse matrix implementations and any challenges they encountered and how they overcame them.  Please share your experiences, thoughts and suggestions in the comments below.