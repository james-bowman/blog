+++
categories = ["Development"]
date = "2017-07-07T18:45:40+01:00"
description = "Using feature hashing to avoid training vocabularies in Golang for Natural Language Processing (NLP) and machine learning"
draft = false
tags = ["development", "machine learning", "go", "algorithms"]
title = "Optimising algorithms in Go for machine learning - Part 3: The hashing trick"
image = "/post/juggler-1216853_1280.jpg"

+++

This is the third in a series of blog posts sharing my experiences working with algorithms and data structures for machine learning.  These experiences were gained whilst building out the [nlp project](http://github.com/james-bowman/nlp) for [LSA (Latent Semantic Analysis)]({{< ref "semantic-analysis-of-webpages-with-machine-learning-in-go.md" >}}) of text documents.

In Part 2 of this series, I explored [sparse matrix formats]({{< ref "optimising-machine-learning-algorithms-part2.md" >}}) as a set of data structures for more efficiently storing and manipulating sparsely populated matrices (matrices where most elements contain zero values).  We tested the impact of using sparse formats, over the originally implemented dense matrix formats, using Go's inbuilt benchmark functionality and found that our optimisations led to a reduction in memory consumption and processing time from 1 GB to 150 MB and 3.3 seconds to 1.2 seconds respectively.

The [Golang sparse matrix format implementations](http://github.com/james-bowman/sparse) used in the article are available on [Github](http://github.com/james-bowman/sparse) along with all the [benchmarks and sample code](http://github.com/james-bowman/nlpbench) used in this series.

In this article we will use the sparse matrix formats from the previous article to implement 'the hashing trick'.

## Feature Extraction (Vectorisation)

In machine learning applications we frequently encode 'things' as vectors of numerical features to allow quantative analysis and comparison.  For natural language processing, these 'things' are usually text documents encoded as vectors of term frequencies i.e. the frequency with which each unique word (term) appears in the document.  When vectorising features in this way, it is important that all vectors are encoded consistently i.e. that frequencies for each specific term always map to the same row from one vector to another.  If featurers were not encoded consistently, with different positions in vectors or vectors of different length, then they could not be compared to one another: it would be like comparing apples and oranges.

Usually, consistent vectorisation is achieved using a predefined vocabulary of words, with each unique word in the vocabulary corresponding to the same specific vector row.  This can be implemented using a dictionary/map structure to map each unique word of the vocabulary to a vector row index.  The vocabulary can either be manually created, or more usually, it is trained from a representative set of training documents.  This approach has 3 main disadvantages as follows:

1. The memory required to store the vocabulary and associated row index mappings (especially as the vocabulary gets large - see point 2 below)
1. Any words missing from the vocabulary will not be represented as features in vectors encoded from the vocabulary.  To overcome this limitation, we usually train the vocabulary with a large and fully representative training data set to increase the chances that the model is exposed to any and all potentially required terms during training.
1. The requirement to train the vocabulary can make this algorithm unsuitable for some streaming data applications.

## Feature Hashing ('the hashing trick')

Feature hashing (also known as 'the hashing trick') uses a hash function to map values to indices in feature vectors.  This removes the reliance on a pre-defined or trained vocabulary, as features are directly mapped based upon their hash value, meaning the same index can be repeatedly and consistenly reproduced given the same word and hash function.  This has the effect of reducing memory requirements and making it highly suitable for use in streaming applications where a pre-defined vocabulary might not be available or practical.

When using feature hashinig, the number of unique words across the corpus is unknown, so the dimensionality of feature vectors must be set in advance (so all vectors are in the same dimensional space).  This length is entirely arbitary but should be sufficiently large to support all required words from the language being used.  For an English language document corpora, this number is likely to be around 1,000,000 although in many applications, much smaller values can be sufficient.

Once the vector dimensionality is fixed, we can apply a hash function to each word in the documents to be vectorised.  We then calculate the modulus of the resulting hash value with the vector length.  This calculation will yield the row index for the word applicable for all vectors within the defined dimensional space (using the same hash function)  - `index = hash(word) % vectorLength`.

It should be clear that this approach will create vectors of extremely high dimensionality and so without [sparse matrix formats]({{< ref "optimising-machine-learning-algorithms-part2.md" >}}), as discussed in the previous article, this approach is not feasible because of the memory requirements necessary to store all the elements in dense matrix format.

One potential downside of using the hashing trick is that multiple, different words may translate to the same index in the vector space (hash collisions) resulting in a loss of precision.  One way of reducing the effect of collisions is to choose a dimensionality sufficiently high enough so as to lessen the likelihood of collisions.  Another approach sometimes adopted is to use a second hash function that will indicate the sign (+/-) to apply to the value being updated within the vector.  If 2 words equate to the same index in the vector space - one may result in +1 (or +_n_ where _n_ is the number of occurances of the word within the corresponding document) and the other may result in -1 (or -_n_ where _n_ is the number of occurances of the word within the corresponding document).  In this way, the collision does not compound the error but rather the two word occurances cancel each other out.

Another disadvantage of feature hashing is that, once vectors are created, it is difficult to determine which frequency values relate to which terms.  When using a vocabulary, with associated index mappings, it is easier to analyse frequencies for particular words.  This can make feature hashing unsuitable for some applications.

## Implementation

Here is the code for the `HashingVectoriser` that implements the hashing trick.

``` Go
type HashingVectoriser struct {
	NumFeatures   int
	wordTokeniser *regexp.Regexp
	stopWords     map[string]bool
}

func NewHashingVectoriser(removeStopwords bool, numFeatures int) *HashingVectoriser {
	var stop map[string]bool

	if removeStopwords {
		stop = make(map[string]bool)
		for _, word := range stopWords {
			stop[word] = true
		}
	}
	return &HashingVectoriser{
		NumFeatures:   numFeatures,
		wordTokeniser: regexp.MustCompile("\\w+"),
		stopWords:     stop,
	}
}

func (v *HashingVectoriser) Fit(train ...string) *HashingVectoriser {
	// Do nothing - the HashingVectoriser is stateless and does not
	// require training.  Method included for compatibility with
	// other vectorisers
	return v
}

func (v *HashingVectoriser) Transform(docs ...string) (mat64.Matrix, error) {
	mat := sparse.NewDOK(v.NumFeatures, len(docs))

	for d, doc := range docs {
		words := v.tokenise(doc)

		for _, word := range words {
			if v.stopWords != nil {
				if v.stopWords[word] {
					continue
				}
			}
			h := murmur3.Sum32([]byte(word))
			i := int(h) % v.NumFeatures

			mat.Set(i, d, mat.At(i, d)+1)
		}
	}
	return mat, nil
}

func (v *HashingVectoriser) FitTransform(docs ...string) (mat64.Matrix, error) {
	return v.Transform(docs...)
}

func (v *HashingVectoriser) tokenise(text string) []string {
	// convert content to lower case
	c := strings.ToLower(text)

	// match whole words, removing any punctuation/whitespace
	words := v.wordTokeniser.FindAllString(c, -1)

	return words
}
```

We shall do some basic benchmarking of the `HashingVectoriser` against our exising `CountVectoriser` implementation.  Whilst the main advantage of the HashingVectoriser is that it removes the dependency on the training data set to train the model with a complete vocabulary, we want to ensure the performance and memory consumption are at least comparable to the `CountVectoriser`.

We shall benchmark the `Transform()` method of the `HashingVectoriser` and compare this against benchmarks for both the `Transform()` and `FitTransform()` methods of `CountVectoriser`.  This will allow comparison of the raw transforms for both algorithms but also to see any savings achieved by not having to train the vocabulary (Fitting).

Here is the benchmarking code which uses [Golang's inbuilt benchmarking functionality](https://golang.org/pkg/testing/#hdr-Benchmarks).  The benchmarks make use of the 20 Newsgroups dataset as used in Scikit Learn.  In this case, we are using the entire dataset rather than a subset as in the last article.  This dataset contains 19,998 documents comprising a vocabulary of 209,412 unique words.  Our CountVectoriser will create a term document matrix 209,412 rows x 19,998 columns and a density of 0.1%.  We shall set the vector dimensionality for the `HashingVectoriser` to 1,000,000 rows.

``` Go
// Benchmark CountVectoriser vs HashingVectoriser
func BenchmarkNLPCountVectoriserTransform(b *testing.B) {
	files := Load()
	vect := nlp.NewCountVectoriser(false)
	vect.Fit(files...)

	b.ResetTimer()
	for n := 0; n < b.N; n++ {
		mat, _ := vect.Transform(files...)
	}
}

func BenchmarkNLPCountVectoriserFitTransform(b *testing.B) {
	files := Load()
	vect := nlp.NewCountVectoriser(false)

	b.ResetTimer()
	for n := 0; n < b.N; n++ {
		mat, _ := vect.FitTransform(files...)
	}
}

func BenchmarkNLPHashingVectoriserFitTransform(b *testing.B) {
	files := Load()
	vect := nlp.NewHashingVectoriser(false, 1000000)

	b.ResetTimer()
	for n := 0; n < b.N; n++ {
		mat, _ := vect.FitTransform(files...)
	}
}
```

And here are the results.

```
Jamess-MacBook-Pro:nlpbench jbowman$ go test -bench=NLP -benchmem
BenchmarkNLPCountVectoriserTransform-4       1	7364512097 ns/op	1098524336 B/op	 8181853 allocs/op
BenchmarkNLPCountVectoriserFitTransform-4    1	11587444509 ns/op	1717760288 B/op	16137998 allocs/op
BenchmarkNLPHashingVectoriserFitTransform-4  1	6578351432 ns/op	1098525672 B/op	 8182272 allocs/op
PASS
ok  	github.com/james-bowman/nlpbench	35.133s
```

From the results of the benchmarks we can see that the `HashingVectoriser` is slightly faster at transforming the data than the `CountVectoriser`.  However, the real saving is that the `HashingVectoriser` does not require training.  This can save time - using the same sized corpus for training and transforming, both take similar lengths of time - but also memory consumption.  The `HashingVectoriser` does not need to store the vocabulary and associated index mappings in memory (over 600 MB using our training data).

## Wrapping Up

Using a `HashingVectoriser` can remove the need to pre-train a vocabulary better supporting streaming applications and also reducing memory consumption (by over 600 MB in our benchmarks).  However, the `HashingVectoriser` is not without its drawbacks.  Using a `HashingVectoriser`, one trades precision and accuracy for utility and memory consumption.

The [benchmarks and sample code](http://github.com/james-bowman/nlpbench) used in this article are all on [Github](http://github.com/james-bowman/nlpbench).
