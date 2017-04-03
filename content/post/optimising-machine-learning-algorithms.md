+++
date = "2017-03-31T16:31:29+01:00"
title = "Optimising machine learning algorithms"
categories = ["DevOps","Development"]
tags = ["cd","devops","development"]
draft = true
description = ""

+++

In my last [blog post]({{< ref "semantic-analysis-of-webpages-with-machine-learnging-in-go.md" >}}) I walked through the implementation of machine learning algorithms in Go to perform analysis and comparison of the latent semantic meaning of documents.  These algorithms, like many in data science rely on linear algebra, matrix arithmetic and vector space analysis.  When dealing with large corpora some of the matrices used by the algorithms could contain millions of elements and memory will become a real constraint on the number of documents that can be analysed.  To this end, I decided to see if I could optimise the algorithms to use less memory without sacrificing too much performance, functionality or accuracy.

## Optimisation of the TF-IDF transform

As can be seen in the code snippet below, the TF-IDF transformer maintains a matrix internally used to transform matrices containing raw term freqencies(tf) into tf-idf (term frequency inverse document frequency) matrices.  The transform is simply multiplied by the tf matrix to produce the tf-idf matrix.  This transform matrix will have as many rows as there were unique terms/words in the training data set and as many columns as there were documents.  For a representative sample of 3000 articles from internet sites, I found the number of rows was typically around 30,000.  A 30,000 x 3000 matrix with elements of type float64 will require approximately 700MB of memory.  Ironically the majority of elements within this matrix will contain zeros, in fact, only values along the diagonal (from top left to bottom right) will be non-zero.  This is clearly an obvious candidate to optimise for memory use.

``` go
type TfidfTransformer struct {
    transform *mat64.Dense
}

func (t *TfidfTransformer) Fit(mat mat64.Matrix) *TfidfTransformer {
    m, n := mat.Dims()

    // build a diagonal matrix from array of term weighting values for subsequent
    // multiplication with term document matrics
    t.transform = mat64.NewDense(m, m, nil)

    for i := 0; i < m; i++ {
        df := 0
        for j := 0; j < n; j++ {
            if mat.At(i, j) != 0 {
                df++
            }
        }
        idf := math.Log(float64(1+n) / float64(1+df))
        t.transform.Set(i, i, idf)
    }

    return t
}

func (t *TfidfTransformer) Transform(mat mat64.Matrix) (*mat64.Dense, error) {
    m, n := mat.Dims()
    product := mat64.NewDense(m, n, nil)

    // simply multiply the matrix by our idf transform (the diagonal matrix of term weights)
    product.Product(t.transform, mat)

    return product, nil
}

func (t *TfidfTransformer) FitTransform(mat mat64.Matrix) (*mat64.Dense, error) {
    return t.Fit(mat).Transform(mat)
}
```

My initial thought was to use a sparse matrix instead of the dense matrix implementation I was using.  A sparse matrix would more efficiently store a matrix mostly comprised of zeros, storing only the non-zero values.  Unfortunately gonum does not currently have a sparse matrix implementation.

``` go
type OptimisedTfidfTransformer struct {
    weights []float64
}

func (t *OptimisedTfidfTransformer) Fit(mat mat64.Matrix) Transformer {
    m, n := mat.Dims()

    // simply capture term weights as an array
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

func (t *OptimisedTfidfTransformer) Transform(mat mat64.Matrix) (*mat64.Dense, error) {
    m, n := mat.Dims()
    product := mat64.NewDense(m, n, nil)

    // apply a function to every element of the matrix in turn which 
    // multiplies the element value by the corresponding term weight
    product.Apply(func(i, j int, v float64) float64 {
        return (v * t.weights[i])
    }, mat)

    return product, nil
}

func (t *OptimisedTfidfTransformer) FitTransform(mat mat64.Matrix) (*mat64.Dense, error) {
    return t.Fit(mat).Transform(mat)
}
```

``` go
func (t *Optimised2TfidfTransformer) Transform(mat mat64.Matrix) (*mat64.Dense, error) {
    m, n := mat.Dims()
    product := mat64.NewDense(m, n, nil)

    // iterate over every element of the matrix in turn and 
    // multiply the element value by the corresponding term weight
    for i := 0; i < m; i++ {
        for j := 0; j < n; j++ {
            product.Set(i, j, mat.At(i, j) * t.weights[i])
        }
    }

    return product, nil
}
```

``` go
func benchmarkTFIDFFitTransform(t Transformer, m, n int, b *testing.B) {
    mat := mat64.NewDense(m, n, nil)

    for n := 0; n < b.N; n++ {
        t.FitTransform(mat)
    }
}

func BenchmarkTFIDFFitTransformDiag2000x1000(b *testing.B) {
    benchmarkTFIDFFitTransform(TfidfTransformer{}, 2000, 1000, b)
}
func BenchmarkTFIDFFitTransformDiag20000x10000(b *testing.B) {
    benchmarkTFIDFFitTransform(TfidfTransformer{}, 20000, 10000, b)
}
func BenchmarkTFIDFFitTransformLoopOptimised2000x1000(b *testing.B) {
    benchmarkTFIDFFitTransform(Optimised2TfidfTransformer{}, 2000, 1000, b)
}
func BenchmarkTFIDFFitTransformLoopOptimised20000x10000(b *testing.B) {
    benchmarkTFIDFFitTransform(Optimised2TfidfTransformer{}, 20000, 10000, b)
}
func BenchmarkTFIDFFitTransformVisitorOptimised2000x1000(b *testing.B) {
    benchmarkTFIDFFitTransform(OptimisedTfidfTransformer{}, 2000, 1000, b)
}
func BenchmarkTFIDFFitTransformVisitorOptimised20000x10000(b *testing.B) {
    benchmarkTFIDFFitTransform(OptimisedTfidfTransformer{}, 20000, 10000, b)
}

```


```
$ go test -bench=.
BenchmarkTFIDFFitTransformDiag2000x1000-8                 20	  51173639 ns/op
BenchmarkTFIDFFitTransformDiag20000x10000-8                1  41549266838 ns/op
BenchmarkTFIDFFitTransformLoopOptimised2000x1000-8        30	  37288375 ns/op
BenchmarkTFIDFFitTransformLoopOptimised20000x10000-8       1	4483422906 ns/op
BenchmarkTFIDFFitTransformVisitorOptimised2000x1000-8     50	  24462948 ns/op
BenchmarkTFIDFFitTransformVisitorOptimised20000x10000-8    1	2622555890 ns/op
PASS
ok  	github.com/james-bowman/nlp	61.093s
```

## Use a Hashing Vectoriser ('the hashing trick')

## Word count removal

Hash map vs trie (Regex)