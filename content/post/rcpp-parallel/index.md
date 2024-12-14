+++
title = "Getting Started with Rcpp Parallel"

date = 2020-03-23T21:00:00
draft = false

authors = ["Gabriel Teotonio"]

tags = ["C++", "Parallel", "Benchmark"]

summary = ""

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Otherwise, set `projects = []`.

#projects = ["grants", "CEEMID"]

# Featured image
# To use, add an image named `featured.jpg/png` to your project's folder. 
[image]
# Caption (optional)
caption = ""

# Focal point (optional)
# Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
focal_point = ""

# Show image only in page previews?
preview_only = false

+++
  
My first contact with C/C++ was in Data Structures and Algorithms course in the begging of my undergrad, by the time I learned the 
classical data structures as list, queue, stack, binary tree, binary search tree, among others. In most part of my projects at 
university I used to program in R and wasn't aware that the computational power of C++ could be integrated to R and make libraries
like `Armadillo`available. This reality change in my life when I was presented to `Rcpp` R library, which let us to import functions 
from C++ to R session. Since then I've been using C++ integrated with R in many problems as my undergraduate thesis, which I developed 
a K-means algorithm for Statistical Shape Analysis with 3-dimensional data about the cortical surface. Similar to the thesis problem,
most of my projects integrating both languages needed a good computational performance for calculating operations with big 
vector/matrices structures. Now I am trying to reach a new level of this integration, by using the `RcppParallel` library to run C++ 
code in R with a parallel backend.  
`RcppParallel` is a R package that provides implementation of high level parallel algorithms. The main functions are:  
  
* `parallelFor`  
* `parallelReduce`  

Throughout this post we are going to run some examples and understand how this package works and its benefits. You can install the package with the following command  

```r
install.packages("RcppParallel")
```

A good reminder before starting to build your parallel functions is that R is single-thread, so you code should not call the R or Rcpp API in any fashion. It can cause crashes and other undefined behavior, as pointed by documentation.  

