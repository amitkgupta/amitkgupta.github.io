---
layout: post
title: "k-NN Algorithm in Golang and Haskell"
date: 2014-06-21 17:46
comments: true
categories: 
---

There's been a recent string of blog posts featuring implementations of the [k-Nearest Neighbour algorithm](http://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm) in several languages (for k = 1), and it's been cool to see how the solutions are expressed differently in different languages, as well as the vast differences in performance.  The k-Nearest Neighbour algorithm is cool in itself, it's a dead simple technique for classifying (or labelling) a collection of observed data by searching the previously observed collections of data whose classifications/labels are already known, and finding the one which most nearly resembles our not-yet-classified collection (for some definition of "nearly resembles").  It's an example of a [machine learning](http://en.wikipedia.org/wiki/Machine_learning) classification algorithm, one of those things that lives in the fun intersection of math and programming.

We're gonna take a look at a concrete application of the k-NN algorithm, compare the performance of the implementations from those aforementioned blog posts with new implementations in [Golang](http://golang.org) and [Haskell](http://www.haskell.org/), and take a look at an optimized version which takes a logical shortcut and also leverages Golang's built-in support for [concurrency](http://www.golang-book.com/10/index.htm).

All the code and datasets can be found on [Github](https://github.com/amitkgupta/nearest_neighbour).  The Golang and Haskell code is also at the bottom of this post.

**TL;DR**: Golang wins, or, in honor of the World Cup: ***GOOOOOOOOOOLLLLLLLang!!!***

## The problem

In this particular example, we've got 5000 pixelated (28x28) greyscale (0-255) "drawings" of the digits 0 through 9.  Some of them might look like this:

![](https://onlinecourses.science.psu.edu/stat857/sites/onlinecourses.science.psu.edu.stat857/files/image_01.png)

*<sub><sup>Source: https://onlinecourses.science.psu.edu/stat857/node/186</sup></sub>*

These 5000 digit drawings constitute our **training set**.  We're then given a bunch of new drawings where (let's pretend for a moment) we don't know what digits they're supposed to represent, but we know the greyscale values at each pixel.  Given any such unclassified drawing, our goal is to make a reasonable guess as to what digit it's supposed to represent.  The way this algorithm works is to find the drawing in the training set which most nearly resembles our unclassified drawing, then our reasonable guess is that the unclassified drawing in question represents the same digit as the nearest drawing in the training set.  At this point, we can say that we've **classified** our previously unclassified drawing.

But what does "nearly resemble" mean in this case?  Roughly, we want to look at how different a pair of drawings is, pixel by pixel, and aggregate those differences for all the pixels.  The smaller the aggregate pixel difference, the nearer the resemblance. The standard measure of distance here is the Euclidean metric: Given two vectors $\vec{x}, \vec{y}$ of length $28 \times 28 = 784$ consisting of 8-bit unsigned integers $0\dots 255$, we define their distance to be:

$$d(\vec{x}, \vec{y}) = \sqrt{\sum_{i=0}^{783} (x_i - y_i)^2}$$


In this problem we're given 500 drawings to classify, and they form our **validation set**.  After running the algorithm against all 500, we can see what percentage of them we classified correctly (because we actually are given their labels, we just pretend not to know them when doing the classification), and how long it took to do them all.

The data is given to us as a couple of CSV files, one for the training set, one for the validation set.  Each row corresponds to a drawing.  The first column is the label (i.e. what digit the drawing represent), and the next 784 columns are the greyscale values of each pixel in the drawing.

Note that the above describes the k-Nearest Neighbour classification in the case $k=1$.  If we wanted to do it for $k > 0$, we would take an unclassified drawing and find the $k$ nearest drawings in the training set, and then classify the drawing according to whichever digit is represented most amongst those $k$ nearest drawings.

## Blog Chain

This post inspired by a chain of blog posts, each of which contains implementations of the algorithm in a different language (or two).  All the implementations are *naive*, in that they pretty much do the simplest thing possible, and take hardly any shortcuts to speed up or skip calculations:
  
* [The most recent one](http://re-factor.blogspot.ca/2014/06/comparing-k-nn-in-factor.html) implemented it in [Factor](http://factorcode.org/) 
* [The one before that](http://huonw.github.io/2014/06/10/knn-rust.html) did it in [Rust](http://www.rust-lang.org/).    
* That one was inspired by [a blog post](http://philtomson.github.io/blog/2014/05/29/comparing-a-machine-learning-algorithm-implemented-in-f-number-and-ocaml/) which had it in [F#](http://fsharp.org/) and [OCaml](http://ocaml.org/), and [a follow-up](http://philtomson.github.io/blog/2014/05/30/stop-the-presses-ocaml-wins/) which improves the first OCaml implementation.  

I work for [Pivotal](http://www.gopivotal.com/) on the [Cloud Foundry](http://cloudfoundry.org/index.html) project and recently joined the [Diego](https://www.youtube.com/watch?v=1OkmVTFhfLY) team where I was introduced to Golang.  I thought it'd be fun to add naive and optimized implementations in Golang to the comparison.  Then I came across an [awesome primer on Haskell](http://learnyouahaskell.com/) so the incomparable [@alexsuraci](https://twitter.com/alexsuraci) and I paired on adding Haskell to the mix.

## Comparison

Performance comparisons between the naive implementations in each language were performed on a freshly spun up c3.xlarge EC2 instance as follows:

1. Install Golang, Haskell, Rust, F#, and OCaml. Download Factor.
2. Write the (naive) code for Golang and Haskell. Copy-paste the code for Rust, F#, OCaml, and Factor.
3. Compile executables for Haskell, Rust, F#, and OCaml.
4. Run and time the executables with `time ./<executable-name>`.  Run the Golang code with `time go run golang-k-nn.go`.  Run the Factor code in the `scratchpad` REPL with `[k-nn] time`.

### Results
1. Golang: 4.701s
1. Factor: 6.358s
1. OCaml: 12.757s
1. F#: 23.507s
1. Rust: 78.138s
1. Haskell: 91.581s

#### Golang
```
$ time go run golang-k-nn.go

0.944

real	0m4.701s
user	0m4.582s
sys	0m0.136s
```

#### Haskell
```
$ time ./haskell-k-nn

Percentage correct: 472

real  1m31.581s
user  1m29.191s
sys   0m2.384s
```

#### Rust
```
$ time ./rust-k-nn

Percentage correct: 94.4%

real	1m18.138s
user	1m17.980s
sys	    0m0.155s
```

#### F\# 
```
$ time ./fsharp-k-nn.exe

start...
Percentage correct:94.400000

real	0m23.507s
user	0m22.751s
sys	    0m0.798s
```

#### OCaml
```
$ time ./ocaml-k-nn

Percentage correct:94.400000

real	0m12.757s
user	0m12.500s
sys	    0m0.257s
```

#### Factor
```
$ mkdir -p $FACTOR_HOME/work/k-nn
$ cp factor/factor-k-nn.factor $FACTOR_HOME/work/k-nn
$ cp *.csv $FACTOR_HOME/work/k-nn
$ $FACTOR_HOME/factor

IN: scratchpad USE: factor-k-nn
Loading resource:work/k-nn/factor-k-nn.factor
Loading resource:basis/formatting/formatting.factor
Loading resource:basis/formatting/formatting-docs.factor

IN: scratchpad gc [ k-nn ] time
Percentage correct: 94.400000
Running time: 6.357621145 seconds
```

## Optimized implementation in Golang

The Golang implementation gets a major performance boost involves two optimizations:

1. Short-circuit distance calculations between a test case and a training case that are necessarily suboptimal.  In other words, if you know the distance to one potential nearest neighbour is 100, and half-way through calculating the distance to another potential nearest neighbour you already have a distance-so-far of 105, stop calculating and move on to the next candidate for nearest neighbour.
2. Use goroutines to parallelize the computations.  The way this was done was not ideal, because the parallelism isn't in the classification algorithm itself, instead it parellelizes the classification of the members of the validation sample.  However, it's is easy enough to "do it right", and what's currently there is good enough to see how significant the gains are when firing on all your cores.

```
$ time go run golang-k-nn-speedup.go

0.944

real	0m1.375s
user	0m3.314s
sys	0m0.117s
```

## Code

#### Golang (naive)

```go
package main

import (
	"bytes"
	"fmt"
	"io/ioutil"
	"strconv"
)

type LabelWithFeatures struct {
	Label    []byte
	Features []float64
}

func NewLabelWithFeatures(parsedLine [][]byte) LabelWithFeatures {
	label := parsedLine[0]
	features := make([]float64, len(parsedLine)-1)

	for i, feature := range parsedLine {
		// skip label
		if i == 0 {
			continue
		}

		features[i-1] = byteSliceTofloat64(feature)
	}

	return LabelWithFeatures{label, features}
}

var newline = []byte("\n")
var comma = []byte(",")

func byteSliceTofloat64(b []byte) float64 {
	x, _ := strconv.ParseFloat(string(b), 32)
	return x
}

func parseCSVFile(filePath string) []LabelWithFeatures {
	fileContent, _ := ioutil.ReadFile(filePath)
	lines := bytes.Split(fileContent, newline)
	numRows := len(lines)

	labelsWithFeatures := make([]LabelWithFeatures, numRows-2)

	for i, line := range lines {
		// skip headers
		if i == 0 || i == numRows-1 {
			continue
		}

		labelsWithFeatures[i-1] = NewLabelWithFeatures(bytes.Split(line, comma))
	}

	return labelsWithFeatures
}

func squareDistance(features1, features2 []float64) (d float64) {  
	for i := 0; i < len(features1); i++ {
    d += (features1[i] - features2[i]) * (features1[i] - features2[i])
  }

  return
}

var trainingSample = parseCSVFile("trainingsample.csv")

func classify(features []float64) (label []byte) {
	label = trainingSample[0].Label
	d := squareDistance(features, trainingSample[0].Features)

	for _, row := range trainingSample {
		dNew := squareDistance(features, row.Features)
		if dNew < d {
			label = row.Label
			d = dNew
		}
	}

	return
}

func main() {
	validationSample := parseCSVFile("validationsample.csv")

	totalCorrect := 0

	for _, test := range validationSample {
		if string(test.Label) == string(classify(test.Features)) {
			totalCorrect++
		}
	}

	fmt.Println(float64(totalCorrect) / float64(len(validationSample)))
}
```

#### Golang (optimized)
```go
package main

import (
	"bytes"
	"fmt"
	"io/ioutil"
	"math"
	"runtime"
	"strconv"
)

type LabelWithFeatures struct {
	Label    []byte
	Features []float32
}

func NewLabelWithFeatures(parsedLine [][]byte) LabelWithFeatures {
	label := parsedLine[0]
	features := make([]float32, len(parsedLine)-1)

	for i, feature := range parsedLine {
		// skip label
		if i == 0 {
			continue
		}

		features[i-1] = byteSliceTofloat32(feature)
	}

	return LabelWithFeatures{label, features}
}

var newline = []byte("\n")
var comma = []byte(",")

func byteSliceTofloat32(b []byte) float32 {
	x, _ := strconv.ParseFloat(string(b), 32) //10, 8)
	return float32(x)
}

func parseCSVFile(filePath string) []LabelWithFeatures {
	fileContent, _ := ioutil.ReadFile(filePath)
	lines := bytes.Split(fileContent, newline)
	numRows := len(lines)

	labelsWithFeatures := make([]LabelWithFeatures, numRows-2)

	for i, line := range lines {
		// skip headers
		if i == 0 || i == numRows-1 {
			continue
		}

		labelsWithFeatures[i-1] = NewLabelWithFeatures(bytes.Split(line, comma))
	}

	return labelsWithFeatures
}

func squareDistanceWithBailout(features1, features2 []float32, bailout float32) (d float32) {
	for i := 0; i < len(features1); i++ {
		x := features1[i] - features2[i]
		d += x * x

		if d > bailout {
			break
		}
	}

	return
}

var trainingSample = parseCSVFile("trainingsample.csv")

func classify(features []float32) (label []byte) {
	label = trainingSample[0].Label
	d := squareDistanceWithBailout(features, trainingSample[0].Features, math.MaxFloat32)

	for _, row := range trainingSample {
		dNew := squareDistanceWithBailout(features, row.Features, d)

		if dNew < d {
			label = row.Label
			d = dNew
		}
	}

	return
}

func main() {
	runtime.GOMAXPROCS(runtime.NumCPU())

	validationSample := parseCSVFile("validationsample.csv")

	var totalCorrect float32 = 0
	successChannel := make(chan float32)

	for _, test := range validationSample {
		go func(t LabelWithFeatures) {
			if string(t.Label) == string(classify(t.Features)) {
				successChannel <- 1
			} else {
				successChannel <- 0
			}
		}(test)
	}

	for i := 0; i < len(validationSample); i++ {
		totalCorrect += <-successChannel
	}

	fmt.Println(float32(totalCorrect) / float32(len(validationSample)))
}
```

#### Haskell (naive)
```haskell
{-# LANGUAGE BangPatterns #-}

import Data.Csv
import qualified Data.ByteString.Lazy as BS
import qualified Data.Vector as V

type Label = Field

type Feature = Integer

data Observation = Observation { label :: !Label
                               , features :: !(V.Vector Feature)
                               } deriving (Show, Eq)

instance FromRecord Observation where
  parseRecord !v = do
    V.mapM parseField (V.tail v) >>=
      return . Observation (V.head v)

parseRecords :: BS.ByteString -> Either String (V.Vector Observation)
parseRecords = decode HasHeader

dist :: (V.Vector Feature) -> (V.Vector Feature) -> Integer
dist !x !y = V.sum $ V.map (^2) $ V.zipWith (-) x y

closerTo :: (V.Vector Feature) -> Observation -> Observation -> Ordering
closerTo !target !o1 !o2 = compare (dist target (features o1)) (dist target (features o2))

classify :: (V.Vector Observation) -> (V.Vector Feature) -> Label
classify !os !fs = label where
  (Observation label _) = V.minimumBy (closerTo fs) os

checkCorrect :: (V.Vector Observation) -> Observation -> Int
checkCorrect !training (Observation label features)
  | label == classify training features = 1
  | otherwise = 0

main = do
  validationSample <- BS.readFile "validationsample.csv"
  trainingSample <- BS.readFile "trainingsample.csv"

  let Right validation = parseRecords validationSample
  let Right training = parseRecords trainingSample
  
  print (V.sum (V.map (checkCorrect training) validation))
```
