+++
date = "2017-03-31T16:31:29+01:00"
title = "Optimising algorithms in Go for machine learning"
categories = ["Development"]
tags = ["development", "machine learning", "go", "benchmark", "optimisation"]
draft = false
description = "Benchmarking and optimising algorithms in Golang for machine learning and large data sets"
image = "/post/optimising-algorithms.jpg"
imageURL = "https://www.flickr.com/photos/derekgavey/5528275910"
imageTitle = "Algorithmic Contaminations"
imageCreator = "Derek Gavey"
imageLicenceURL = "https://creativecommons.org/licenses/by/2.0"
imageLicenceName = "CC BY 2.0"

+++

In my last [blog post]({{< ref "semantic-analysis-of-webpages-with-machine-learning-in-go.md" >}}) I walked through the use of machine learning algorithms in Go to analyse the latent semantic meaning of documents.  These algorithms, like many others in data science, rely on linear algebra and vector space analysis.  By their nature, they often have to deal with large data sets, so any inefficiencies in the data structures used or algorithms themselves can result in a large impact on overall performance and/or memory usage.  Inefficiencies that are negligable when working with small data sets can have a huge cost applied across extremely large datasets.  As memory is a constrained resource, this could end up limiting the size of data sets that may be processed (certainly without having to resort to persistent storage and/or alternative algorithms) or the types of algorithms used.  To this end, I decided to see if I could optimise the algorithms I used to consume less memory and improve processing performance without sacrificing too much functionality or accuracy.

## Optimisation of the TF-IDF transform

As can be seen in the code snippet below, the TF-IDF transformer maintains a matrix internally,which is used to transform matrices containing raw term freqencies(tf) into weighted tf-idf values (term frequency - inverse document frequencies).  The transform is simply multiplied by the input tf matrix to produce the tf-idf matrix.  Please refer to the [previous blog post]({{< ref "semantic-analysis-of-webpages-with-machine-learning-in-go.md" >}}) for a more in depth description of TF-IDF.

The transform matrix is a [symmetric diagonal matrix](https://en.wikipedia.org/wiki/Diagonal_matrix#Symmetric_diagonal_matrices) and so will have as many rows and columns as there were unique terms/words in the training data set.  For a representative sample of 3000 articles from internet sites, I found the number of unique terms was typically around 30,000 which would require a 30,000 x 30,000 transform matrix.  Therefore the TF-IDF transform is a 30,000 x 30,000 matrix with elements of type float64 which will require approximately 7GB of memory.  Ironically the majority of elements within this matrix will contain zeros, in fact, only values along the diagonal (from top left to bottom right) will be non-zero.  There is clearly scope to optimise its memory use.

``` go
type TfidfTransformer1 struct {
    transform *mat64.Dense
}

func (t *TfidfTransformer1) Fit(mat mat64.Matrix) Transformer {
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

func (t *TfidfTransformer1) Transform(mat mat64.Matrix) (*mat64.Dense, error) {
    m, n := mat.Dims()
    product := mat64.NewDense(m, n, nil)

    // simply multiply the matrix by our idf transform (the diagonal matrix of term weights)
    product.Product(t.transform, mat)

    return product, nil
}

func (t *TfidfTransformer1) FitTransform(mat mat64.Matrix) (*mat64.Dense, error) {
    return t.Fit(mat).Transform(mat)
}
```

My initial thought was to use a sparse matrix instead of the dense matrix implementation I was using.  A sparse matrix could more efficiently store a matrix mostly comprised of zeros, storing only the non-zero values.  Unfortunately the gonum matrix library I am using does not currently have a sparse matrix implementation.

As an alternative approach, I considered simply storing the non-zero idf (term weights) values as an array of float64 values and then individually multiplying every element within the input raw tf matrix by its corresponding term weight from the array.  The code for this implementation called TfidfTransformer2 is shown below.  Note the differences in the `Fit()` method where we no longer build a symmetric diagonal transform matrix and simply keep the array of term weights.  Also, the `Transform()` method is now changed to simply iterate over every matrix element and multiply its value by the corresponding term weight from the array.

``` go
type TfidfTransformer2 struct {
    weights []float64
}

func (t *TfidfTransformer2) Fit(mat mat64.Matrix) Transformer {
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

func (t *TfidfTransformer2) Transform(mat mat64.Matrix) (*mat64.Dense, error) {
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

func (t *TfidfTransformer2) FitTransform(mat mat64.Matrix) (*mat64.Dense, error) {
    return t.Fit(mat).Transform(mat)
}
```

This second approach would be considerably more memory efficient but I had some concerns over performance, specifically how well iterating over every matrix element, using nested loops, outside of the heavily optimised LAPACK/BLAS libraries would perform.

Then I found a matrix `Apply()` method within the gonum/mat64 library that would apply a function to every element within the matrix.  Unlike the nested loop approach, this would allow the optimised LAPACK/BLAS libraries to handle the iteration so should theoretically be more performant.  This implementation would be identical to the `TfidfTransformer2` above save for the `Transform()` method where the matrix `Apply()` method is used instead of nested loops.  The differing `Transform()` method is shown below.

``` go
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
```

### Benchmarks

To confirm our hypothesis that the new implementations will outperform the current matrix transform based implementation we will run some benchmarks using the inbuilt benchmark functionality inside Go's `testing` package.

Implementing a benchmark in Go is much the same as implementing a test except the function name is prefixed with `Benchmark` rather than `Test`.  Here are separate benchmark implementations for the `Fit()`, `Transform()` and `FitTransform()` methods exercised for each of the 3 algorithm implementations.  As the `Fit()` method is identical between implementation 2 (TfidfTransformer2) and 3 (TfidfTransformer3) I have not bothered to benchmark both instead just benchmarking the `Fit()` method from implementation 2.

``` go
func benchmarkFit(t Transformer, m, n int, b *testing.B) {
    mat := mat64.NewDense(m, n, nil)

    b.ResetTimer()
    for n := 0; n < b.N; n++ {
        t.Fit(mat)
    }
}

func benchmarkFitTransform(t Transformer, m, n int, b *testing.B) {
    mat := mat64.NewDense(m, n, nil)

    b.ResetTimer()
    for n := 0; n < b.N; n++ {
        t.FitTransform(mat)
    }
}

func benchmarkTransform(t Transformer, m, n int, b *testing.B) {
    mat := mat64.NewDense(m, n, nil)
    t.Fit(mat)

    b.ResetTimer()
    for n := 0; n < b.N; n++ {
        t.Transform(mat)
    }
}

func BenchmarkTFIDF1Fit30000x3000(b *testing.B) {
    benchmarkFit(&TfidfTransformer1{}, 30000, 3000, b)
}
func BenchmarkTFIDF1Transform30000x3000(b *testing.B) {
    benchmarkTransform(&TfidfTransformer1{}, 30000, 3000, b)
}
func BenchmarkTFIDF1FitTransform30000x3000(b *testing.B) {
    benchmarkFitTransform(&TfidfTransformer1{}, 30000, 3000, b)
}
func BenchmarkTFIDF2Fit30000x3000(b *testing.B) {
    benchmarkFit(&TfidfTransformer2{}, 30000, 3000, b)
}
func BenchmarkTFIDF2Transform30000x3000(b *testing.B) {
    benchmarkTransform(&TfidfTransformer2{}, 30000, 3000, b)
}
func BenchmarkTFIDF2FitTransform30000x3000(b *testing.B) {
    benchmarkFitTransform(&TfidfTransformer2{}, 30000, 3000, b)
}
func BenchmarkTFIDF3Transform30000x3000(b *testing.B) {
    benchmarkTransform(&TfidfTransformer3{}, 30000, 3000, b)
}
func BenchmarkTFIDF3FitTransform30000x3000(b *testing.B) {
    benchmarkFitTransform(&TfidfTransformer3{}, 30000, 3000, b)
}
```

The benchmarks are run in the same way as tests except that the `-bench` option is used to express a regular expression matching the names of benchmark functions to run.  The regular expression `.` will match all benchmarks or as in the output below, `30000` will match and run all benchmarks with 30000 in the function name.  The `-benchmem` function will output memory allocations during the benchmark.  Here are the results of the benchmarks.

```
$ go test -bench=30000 -benchmem
BenchmarkTFIDF1Fit30000x3000-4              1  6025208569 ns/op     7200006208 B/op     2 allocs/op
BenchmarkTFIDF1Transform30000x3000-4        1  41773814068 ns/op    720005856 B/op      18 allocs/op
BenchmarkTFIDF1FitTransform30000x3000-4     1  50079258438 ns/op    7920011056 B/op     11 allocs/op
BenchmarkTFIDF2Fit30000x3000-4              3  513875302 ns/op      245760 B/op         1 allocs/op
BenchmarkTFIDF2Transform30000x3000-4        1  1669005488 ns/op     720003136 B/op      2 allocs/op
BenchmarkTFIDF2FitTransform30000x3000-4     1  1472562820 ns/op     720250752 B/op      6 allocs/op
BenchmarkTFIDF3Transform30000x3000-4        2  884655455 ns/op      720003136 B/op      2 allocs/op
BenchmarkTFIDF3FitTransform30000x3000-4     1  1029592374 ns/op     720248896 B/op      3 allocs/op
PASS
ok      github.com/james-bowman/nlpbench    121.167s
```

From the benchmark results, we can see that as we expected, the `Fit()` method on the original implementation is allocating around 7 GB of memory (first row).  In comparison we can see that the new array based `Fit()` implementation is allocating around 250 KB.  This is a significant improvement, but how does the performance stack up?

According to the benchmark, the original implementation took 41773814068 nano seconds to perform the transform (second row).  Comparing this with 1669005488 nano seconds for the second nested loop based implementation (TfidfTransformer2 - row 5) would suggest the nested loop implementation is around 20 times faster.  If we further compare the third implementation (TfidfTransformer3), using the `Apply()` method and delegating the iteration over matrix elements down into the optimised LAPACK/BLAS libraries (row 7) we can see this is about twice as fast as the second implementation executing in around half the time.  As we would expect memory usage is the same for both implementation 2 and 3.  

## Wrapping up

We started using a standard implementation of the TF-IDF algorithm (albeit using a Dense matrix implementation) and explored possible optimisations for memory usage.  We looked at 2 alternative methods and then tested our hypothesis using Go's inbuilt benchmark functionality.  The results showed that our optimisations materially improved not just memory consumption but also performance.  For a dataset with 30,000 terms across 3,000 documents, we showed the original implementation executed in around 41 seconds, consuming around 7 GB.  The new implementation processes the same dataset in just 0.8 seconds, consuming around 250 KB of memory.

All code used in this article is on [Github](http://github.com/james-bowman/nlpbench)

