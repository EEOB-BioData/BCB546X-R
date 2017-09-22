---
title: "Data Structures"
teaching: 50
exercises: 30
questions:
- "How can I read data in R?"
- "What are the basic data types in R?"
- "How do I represent categorical information in R?"
- "How can I work with subsets of data in R?"
objectives:
- "To be aware of the different types of data."
- "To be able to ask questions from R about the type, class, and structure of an object."
- "To be able to subset vectors, factors, matrices, and lists"
- "To be able to extract individual and multiple elements: by index, by name, using comparison operations"
- "To be able to skip and remove elements from various data structures."
keypoints:
- "The basic data types in R are double, integer, complex, logical, and character."
- "Use factors to represent categories in R."
- "Indexing in R starts at 1, not 0."
- "Access individual values by location using `[]`."
- "Access slices of data using `[low:high]`."
- "Access arbitrary sets of data using `[c(...)]`."
- "Use `which` to select subsets of data based on value."
---



**Disclaimer:** This tutorial is based on an excellent book [Advanced R.](https://github.com/hadley/adv-r)


## Data structures {#data-structures}


R's base data structures can be organised by their dimensionality (1d, 2d, or nd) and whether they're homogeneous 
(all contents must be of the same type) or heterogeneous (the contents can be of different types): 

|    | Homogeneous   | Heterogeneous |
|----|---------------|---------------|
| 1d | Atomic vector | List          |
| 2d | Matrix        | Data frame    |
| nd | Array         |               |

Almost all other objects are built upon these foundations. 
Note that R has no 0-dimensional, or scalar types. Individual numbers or strings are vectors of length one. 

Given an object, the best way to understand what data structures it's composed of is to use `str()`. `str()` is short for structure and it gives a compact, human readable description of any R data structure.

## Vectors {#vectors}

The basic data structure in R is the vector. Vectors come in two flavours: atomic vectors and lists. They have three common properties:

* Type, `typeof()`, what it is.
* Length, `length()`, how many elements it contains.
* Attributes, `attributes()`, additional arbitrary metadata.

They differ in the types of their elements: all elements of an atomic vector must be the same type, whereas the elements of a list can have different types.

### Atomic vectors

There are four common types of atomic vectors : logical, integer, double (often called numeric), and character.

Atomic vectors are usually created with `c()`, short for combine.


~~~
dbl_var <- c(1, 2.5, 4.5)
# With the L suffix, you get an integer rather than a double
int_var <- c(1L, 6L, 10L)
# Use TRUE and FALSE (or T and F) to create logical vectors
log_var <- c(TRUE, FALSE, T, F)
chr_var <- c("these are", "some strings")
~~~
{: .r}

Atomic vectors are always flat, even if you nest `c()`'s: Try `c(1, c(2, c(3, 4)))`

Missing values are specified with `NA`, which is a logical vector of length 1. `NA` will always be coerced to the correct type if used inside `c()`.

#### Coercion

All elements of an atomic vector must be the same type, so when you attempt to combine different types they will be __coerced__ to the most flexible type. The coercion rules go: `logical` -> `integer` -> `double` -> `complex` -> `character`, where -> can be read as *are transformed into*. You can try to force coercion against this flow using the `as.` functions:


> ## Challenge 1
>
> Create the following vectors and predict their type:
> 
> ~~~
> a <- c("a", 1)
> b <- c(TRUE, 1)
> c <- c(1L, 10)
> d <- c(a, b, c)
> ~~~
> {: .r}
>
> > ## Solution to Challenge 1
> >
> > 
> > ~~~
> > typeof(a); typeof(b); typeof(c); typeof(d)
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> > [1] "character"
> > ~~~
> > {: .output}
> > 
> > 
> > 
> > ~~~
> > [1] "double"
> > ~~~
> > {: .output}
> > 
> > 
> > 
> > ~~~
> > [1] "double"
> > ~~~
> > {: .output}
> > 
> > 
> > 
> > ~~~
> > [1] "character"
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

When a logical vector is coerced to an integer or double, `TRUE` becomes 1 and `FALSE` becomes 0. This is very useful in conjunction with `sum()` and `mean()`


~~~
x <- c(FALSE, FALSE, TRUE)
as.numeric(x)
~~~
{: .r}



~~~
[1] 0 0 1
~~~
{: .output}



~~~
# Total number of TRUEs
sum(x)
~~~
{: .r}



~~~
[1] 1
~~~
{: .output}



~~~
# Proportion that are TRUE
mean(x)
~~~
{: .r}



~~~
[1] 0.3333333
~~~
{: .output}

Coercion often happens automatically. Most mathematical functions (`+`, `log`, `abs`, etc.) will coerce to a double or integer, and most logical operations (`&`, `|`, `any`, etc) will coerce to a logical. You will usually get a warning message if the coercion might lose information. If confusion is likely, explicitly coerce with `as.character()`, `as.double()`, `as.integer()`, or `as.logical()`. 

### Lists

Lists are different from atomic vectors because their elements can be of any type, including lists. You construct lists by using `list()` instead of `c()`:


~~~
x <- list(1:3, "a", c(TRUE, FALSE, TRUE), c(2.3, 5.9))
str(x)
~~~
{: .r}



~~~
List of 4
 $ : int [1:3] 1 2 3
 $ : chr "a"
 $ : logi [1:3] TRUE FALSE TRUE
 $ : num [1:2] 2.3 5.9
~~~
{: .output}

Lists are sometimes called __recursive__ vectors, because a list can contain other lists. This makes them fundamentally different from atomic vectors.


~~~
x <- list(list(list(list())))
str(x)
~~~
{: .r}



~~~
List of 1
 $ :List of 1
  ..$ :List of 1
  .. ..$ : list()
~~~
{: .output}



~~~
is.recursive(x)
~~~
{: .r}



~~~
[1] TRUE
~~~
{: .output}

`c()` will combine several lists into one. If given a combination of atomic vectors and lists, `c()` will coerce the vectors to lists before combining them. Compare the results of `list()` and `c()`:


~~~
x <- list(list(1, 2), c(3, 4))
y <- c(list(1, 2), c(3, 4))
str(x)
~~~
{: .r}



~~~
List of 2
 $ :List of 2
  ..$ : num 1
  ..$ : num 2
 $ : num [1:2] 3 4
~~~
{: .output}



~~~
str(y)
~~~
{: .r}



~~~
List of 4
 $ : num 1
 $ : num 2
 $ : num 3
 $ : num 4
~~~
{: .output}

The `typeof()` a list is `list`. You can test for a list with `is.list()` and coerce to a list with `as.list()`. You can turn a list into an atomic vector with `unlist()`. If the elements of a list have different types, `unlist()` uses the same coercion rules as `c()`.

Lists are used to build up many of the more complicated data structures in R. For example, both data frames and linear models objects (as produced by `lm()`) are lists:


~~~
mtcars
~~~
{: .r}



~~~
                     mpg cyl  disp  hp drat    wt  qsec vs am gear carb
Mazda RX4           21.0   6 160.0 110 3.90 2.620 16.46  0  1    4    4
Mazda RX4 Wag       21.0   6 160.0 110 3.90 2.875 17.02  0  1    4    4
Datsun 710          22.8   4 108.0  93 3.85 2.320 18.61  1  1    4    1
Hornet 4 Drive      21.4   6 258.0 110 3.08 3.215 19.44  1  0    3    1
Hornet Sportabout   18.7   8 360.0 175 3.15 3.440 17.02  0  0    3    2
Valiant             18.1   6 225.0 105 2.76 3.460 20.22  1  0    3    1
Duster 360          14.3   8 360.0 245 3.21 3.570 15.84  0  0    3    4
Merc 240D           24.4   4 146.7  62 3.69 3.190 20.00  1  0    4    2
Merc 230            22.8   4 140.8  95 3.92 3.150 22.90  1  0    4    2
Merc 280            19.2   6 167.6 123 3.92 3.440 18.30  1  0    4    4
Merc 280C           17.8   6 167.6 123 3.92 3.440 18.90  1  0    4    4
Merc 450SE          16.4   8 275.8 180 3.07 4.070 17.40  0  0    3    3
Merc 450SL          17.3   8 275.8 180 3.07 3.730 17.60  0  0    3    3
Merc 450SLC         15.2   8 275.8 180 3.07 3.780 18.00  0  0    3    3
Cadillac Fleetwood  10.4   8 472.0 205 2.93 5.250 17.98  0  0    3    4
Lincoln Continental 10.4   8 460.0 215 3.00 5.424 17.82  0  0    3    4
Chrysler Imperial   14.7   8 440.0 230 3.23 5.345 17.42  0  0    3    4
Fiat 128            32.4   4  78.7  66 4.08 2.200 19.47  1  1    4    1
Honda Civic         30.4   4  75.7  52 4.93 1.615 18.52  1  1    4    2
Toyota Corolla      33.9   4  71.1  65 4.22 1.835 19.90  1  1    4    1
Toyota Corona       21.5   4 120.1  97 3.70 2.465 20.01  1  0    3    1
Dodge Challenger    15.5   8 318.0 150 2.76 3.520 16.87  0  0    3    2
AMC Javelin         15.2   8 304.0 150 3.15 3.435 17.30  0  0    3    2
Camaro Z28          13.3   8 350.0 245 3.73 3.840 15.41  0  0    3    4
Pontiac Firebird    19.2   8 400.0 175 3.08 3.845 17.05  0  0    3    2
Fiat X1-9           27.3   4  79.0  66 4.08 1.935 18.90  1  1    4    1
Porsche 914-2       26.0   4 120.3  91 4.43 2.140 16.70  0  1    5    2
Lotus Europa        30.4   4  95.1 113 3.77 1.513 16.90  1  1    5    2
Ford Pantera L      15.8   8 351.0 264 4.22 3.170 14.50  0  1    5    4
Ferrari Dino        19.7   6 145.0 175 3.62 2.770 15.50  0  1    5    6
Maserati Bora       15.0   8 301.0 335 3.54 3.570 14.60  0  1    5    8
Volvo 142E          21.4   4 121.0 109 4.11 2.780 18.60  1  1    4    2
~~~
{: .output}



~~~
is.list(mtcars)
~~~
{: .r}



~~~
[1] TRUE
~~~
{: .output}



~~~
mod <- lm(mpg ~ wt, data = mtcars)
mod
~~~
{: .r}



~~~

Call:
lm(formula = mpg ~ wt, data = mtcars)

Coefficients:
(Intercept)           wt  
     37.285       -5.344  
~~~
{: .output}



~~~
is.list(mod)
~~~
{: .r}



~~~
[1] TRUE
~~~
{: .output}
> ## Callout
>
> Where does these data come from?
>
{: .callout}


> ## Discussion 1
>
>
> 1. What are the common types of atomic vector? How does a list differ from an atomic vector?
> 
> 1. What makes `is.vector()` and `is.numeric()` fundamentally different to `is.list()` and `is.character()`?
> 
> 1. Test your knowledge of vector coercion rules by predicting the output of the following uses of `c()`:
> 
>    
>    ~~~
>    c(1, FALSE)
>    c("a", 1)
>    c(list(1), "a")
>    c(TRUE, 1L)
>    ~~~
>    {: .r}
>
> 1.  Why do you need to use `unlist()` to convert a list to an 
>     atomic vector? Why doesn't `as.vector()` work? 
>
> 1. Why is `1 == "1"` true? Why is `-1 < FALSE` true? Why is `"one" < 2` false?
> 
> 1. Why is the default missing value, `NA`, a logical vector? What's special
>    about logical vectors? (Hint: think about `c(FALSE, NA_character_)`.)
{: .discussion}

## Attributes {#attributes}

All objects can have arbitrary additional attributes, used to store metadata about the object. Attributes can be thought of as a named list (with unique names). Attributes can be accessed individually with `attr()` or all at once (as a list) with `attributes()`.

The three most important attributes:

* Names, a character vector giving each element a name, described in 
  [names](#vector-names). 

* Dimensions, used to turn vectors into matrices and arrays, 
  described in [matrices and arrays](#matrices-and-arrays).

* Class, used to implement the S3 object system.
 
Each of these attributes has a specific accessor function to get and set values. When working with these attributes, use `names(x)`, `dim(x)`, and `class(x)`, not `attr(x, "names")`, `attr(x, "dim")`, and `attr(x, "class")`.

#### Names {#vector-names}

You can name elements in a vector in three ways:

* When creating it: 

~~~
x <- c(a = 1, b = 2, c = 3)
x
~~~
{: .r}



~~~
a b c 
1 2 3 
~~~
{: .output}



~~~
names(x)
~~~
{: .r}



~~~
[1] "a" "b" "c"
~~~
{: .output}

* By modifying an existing vector in place: 

~~~
x <- 1:3; names(x) <- c("a", "b", "c")
~~~
{: .r}

* By creating a modified copy of a vector: 

~~~
x <- 1:3
y <- setNames(x, c("a", "b", "c"))
~~~
{: .r}

Names don't have to be uniqueand not all elements of a vector need to have a name. If some names are missing, `names()` will return an empty string for those elements. If all names are missing, `names()` will return `NULL`.


~~~
y <- c(a = 1, 2, 3)
names(y)
~~~
{: .r}



~~~
[1] "a" ""  "" 
~~~
{: .output}



~~~
z <- c(1, 2, 3)
names(z)
~~~
{: .r}



~~~
NULL
~~~
{: .output}

You can create a new vector without names using `unname(x)`, or remove names in place with `names(x) <- NULL`.

### Factors

One important use of attributes is to define factors. A factor is a vector that can contain only predefined values, and is used to store categorical data. Factors are built on top of integer vectors using two attributes: the `class()`, "factor", which makes them behave differently from regular integer vectors, and the `levels()`, which defines the set of allowed values.


~~~
x <- c("a", "b", "b", "a")
x
~~~
{: .r}



~~~
[1] "a" "b" "b" "a"
~~~
{: .output}



~~~
x <- factor(x)
x
~~~
{: .r}



~~~
[1] a b b a
Levels: a b
~~~
{: .output}



~~~
class(x)
~~~
{: .r}



~~~
[1] "factor"
~~~
{: .output}



~~~
levels(x)
~~~
{: .r}



~~~
[1] "a" "b"
~~~
{: .output}



~~~
# You can't use values that are not in the levels
x[2] <- "c"
~~~
{: .r}



~~~
Warning in `[<-.factor`(`*tmp*`, 2, value = "c"): invalid factor level, NA
generated
~~~
{: .error}



~~~
x
~~~
{: .r}



~~~
[1] a    <NA> b    a   
Levels: a b
~~~
{: .output}



~~~
# NB: combining factors will produce unwanted results!
c(x, factor("b"))
~~~
{: .r}



~~~
[1]  1 NA  2  1  1
~~~
{: .output}



~~~
class(c(x, factor("b")))
~~~
{: .r}



~~~
[1] "integer"
~~~
{: .output}

Factors are useful when you know the possible values a variable may take. Using a factor instead of a character vector makes it obvious when some groups contain no observations:


~~~
sex_char <- c("m", "m", "m")
sex_factor <- factor(sex_char, levels = c("m", "f"))

table(sex_char)
~~~
{: .r}



~~~
sex_char
m 
3 
~~~
{: .output}



~~~
table(sex_factor)
~~~
{: .r}



~~~
sex_factor
m f 
3 0 
~~~
{: .output}

Factors crip up all over R, and occasionally cause headaches for new R users. We'll discuss why in the next lesson.

## Matrices and arrays {#matrices-and-arrays}

Adding a `dim()` attribute to an atomic vector allows it to behave like a multi-dimensional __array__. A special case of the array is the __matrix__, which has two dimensions. Matrices are used commonly as part of the mathematical machinery of statistics. Arrays are much rarer, but worth being aware of.

Matrices and arrays are created with `matrix()` and `array()`, or by using the assignment form of `dim()`:


~~~
# Two scalar arguments to specify rows and columns
a <- matrix(1:6, ncol = 3, nrow = 2)
# One vector argument to describe all dimensions
b <- array(1:12, c(2, 3, 2))

# You can also modify an object in place by setting dim()
c <- 1:12
dim(c) <- c(3, 4)
c
~~~
{: .r}



~~~
     [,1] [,2] [,3] [,4]
[1,]    1    4    7   10
[2,]    2    5    8   11
[3,]    3    6    9   12
~~~
{: .output}



~~~
dim(c) <- c(4, 3)
c
~~~
{: .r}



~~~
     [,1] [,2] [,3]
[1,]    1    5    9
[2,]    2    6   10
[3,]    3    7   11
[4,]    4    8   12
~~~
{: .output}



~~~
dim(c) <- c(2, 3, 2)
c
~~~
{: .r}



~~~
, , 1

     [,1] [,2] [,3]
[1,]    1    3    5
[2,]    2    4    6

, , 2

     [,1] [,2] [,3]
[1,]    7    9   11
[2,]    8   10   12
~~~
{: .output}

You can test if an object is a matrix or array using `is.matrix()` and `is.array()`, or by looking at the length of the `dim()`. `as.matrix()` and `as.array()` make it easy to turn an existing vector into a matrix or array.

Vectors are not the only 1-dimensional data structure. You can have matrices with a single row or single column, or arrays with a single dimension. They may print similarly, but will behave differently. The differences aren't too important, but it's useful to know they exist in case you get strange output from a function (`tapply()` is a frequent offender). As always, use `str()` to reveal the differences.


~~~
str(1:3)                   # 1d vector
~~~
{: .r}



~~~
 int [1:3] 1 2 3
~~~
{: .output}



~~~
str(matrix(1:3, ncol = 1)) # column vector
~~~
{: .r}



~~~
 int [1:3, 1] 1 2 3
~~~
{: .output}



~~~
str(matrix(1:3, nrow = 1)) # row vector
~~~
{: .r}



~~~
 int [1, 1:3] 1 2 3
~~~
{: .output}



~~~
str(array(1:3, 3))         # "array" vector
~~~
{: .r}



~~~
 int [1:3(1d)] 1 2 3
~~~
{: .output}

While atomic vectors are most commonly turned into matrices, the dimension attribute can also be set on lists to make list-matrices or list-arrays:


~~~
l <- list(1:3, "a", TRUE, 1.0)
dim(l) <- c(2, 2)
l
~~~
{: .r}



~~~
     [,1]      [,2]
[1,] Integer,3 TRUE
[2,] "a"       1   
~~~
{: .output}

These are relatively esoteric data structures, but can be useful if you want to arrange objects into a grid-like structure. For example, if you're running models on a spatio-temporal grid, it might be natural to preserve the grid structure by storing the models in a 3d array.

> ## Discussion 2
>
> 1.  What does `dim()` return when applied to a vector?
> 
> 1.  If `is.matrix(x)` is `TRUE`, what will `is.array(x)` return?
> 
> 1.  How would you describe the following three objects? What makes them
>     different to `1:5`?
>
>    
>    ~~~
>    x1 <- array(1:5, c(1, 1, 5))
>    x2 <- array(1:5, c(1, 5, 1))
>    x3 <- array(1:5, c(5, 1, 1))
>    ~~~
>    {: .r}
{: .discussion}

R has many powerful subset operators and mastering them will allow you to
easily perform complex operations on any kind of dataset.

There are six different ways we can subset any kind of object, and three
different subsetting operators for the different data structures.

Let's start with the workhorse of R: atomic vectors.


~~~
x <- c(5.4, 6.2, 7.1, 4.8, 7.5)
names(x) <- c('a', 'b', 'c', 'd', 'e')
x
~~~
{: .r}



~~~
  a   b   c   d   e 
5.4 6.2 7.1 4.8 7.5 
~~~
{: .output}

So now that we've created a dummy vector to play with, how do we get at its
contents?

## Accessing elements using their indices

To extract elements of a vector we can give their corresponding index, starting
from one:


~~~
x[1]
~~~
{: .r}



~~~
  a 
5.4 
~~~
{: .output}


~~~
x[4]
~~~
{: .r}



~~~
  d 
4.8 
~~~
{: .output}

It may look different, but the square brackets operator is a function. For atomic vectors
(and matrices), it means "get me the nth element".

We can ask for multiple elements at once:


~~~
x[c(1, 3)]
~~~
{: .r}



~~~
  a   c 
5.4 7.1 
~~~
{: .output}

Or slices of the vector:


~~~
x[1:4]
~~~
{: .r}



~~~
  a   b   c   d 
5.4 6.2 7.1 4.8 
~~~
{: .output}

We can ask for the same element multiple times:


~~~
x[c(1,1,3)]
~~~
{: .r}



~~~
  a   a   c 
5.4 5.4 7.1 
~~~
{: .output}

If we ask for a number outside of the vector, R will return missing values:


~~~
x[6]
~~~
{: .r}



~~~
<NA> 
  NA 
~~~
{: .output}

This is a vector of length one containing an `NA`, whose name is also `NA`.

If we ask for the 0th element, we get an empty vector:


~~~
x[0]
~~~
{: .r}



~~~
named numeric(0)
~~~
{: .output}

> ## Vector numbering in R starts at 1
>
> In many programming languages (C and python, for example), the first
> element of a vector has an index of 0. In R, the first element is 1.
{: .callout}

## Skipping and removing elements

If we use a negative number as the index of a vector, R will return
every element *except* for the one specified:


~~~
x[-2]
~~~
{: .r}



~~~
  a   c   d   e 
5.4 7.1 4.8 7.5 
~~~
{: .output}

We can skip multiple elements:


~~~
x[c(-1, -5)]  # or x[-c(1,5)]
~~~
{: .r}



~~~
  b   c   d 
6.2 7.1 4.8 
~~~
{: .output}

> ## Tip: Order of operations
>
> A common trip up for novices occurs when trying to skip
> slices of a vector. Most people first try to negate a
> sequence like so:
>
> 
> ~~~
> x[-1:3]
> ~~~
> {: .r}
>
> This gives a somewhat cryptic error:
>
> 
> ~~~
> Error in x[-1:3]: only 0's may be mixed with negative subscripts
> ~~~
> {: .error}
>
> But remember the order of operations. `:` is really a function, so
> what happens is it takes its first argument as -1, and second as 3,
> so generates the sequence of numbers: `c(-1, 0, 1, 2, 3)`.
>
> The correct solution is to wrap that function call in brackets, so
> that the `-` operator applies to the results:
>
> 
> ~~~
> x[-(1:3)]
> ~~~
> {: .r}
> 
> 
> 
> ~~~
>   d   e 
> 4.8 7.5 
> ~~~
> {: .output}
{: .callout}

To remove elements from a vector, we need to assign the results back
into the variable:


~~~
x <- x[-4]
x
~~~
{: .r}



~~~
  a   b   c   e 
5.4 6.2 7.1 7.5 
~~~
{: .output}

> ## Challenge 1
>
> Given the following code:
>
> 
> ~~~
> x <- c(5.4, 6.2, 7.1, 4.8, 7.5)
> names(x) <- c('a', 'b', 'c', 'd', 'e')
> print(x)
> ~~~
> {: .r}
> 
> 
> 
> ~~~
>   a   b   c   d   e 
> 5.4 6.2 7.1 4.8 7.5 
> ~~~
> {: .output}
>
> Come up with at least 3 different commands that will produce the following output:
>
> 
> ~~~
>   b   c   d 
> 6.2 7.1 4.8 
> ~~~
> {: .output}
>
> After you find 3 different commands, compare notes with your neighbour. Did you have different strategies?
>
> > ## Solution to challenge 1
> >
> > 
> > ~~~
> > x[2:4]
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> >   b   c   d 
> > 6.2 7.1 4.8 
> > ~~~
> > {: .output}
> > 
> > ~~~
> > x[-c(1,5)]
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> >   b   c   d 
> > 6.2 7.1 4.8 
> > ~~~
> > {: .output}
> > 
> > ~~~
> > x[c("b", "c", "d")]
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> >   b   c   d 
> > 6.2 7.1 4.8 
> > ~~~
> > {: .output}
> > 
> > ~~~
> > x[c(2,3,4)]
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> >   b   c   d 
> > 6.2 7.1 4.8 
> > ~~~
> > {: .output}
> >
> {: .solution}
{: .challenge}

## Subsetting by name

We can extract elements by using their name, instead of index:


~~~
x[c("a", "c")]
~~~
{: .r}



~~~
  a   c 
5.4 7.1 
~~~
{: .output}

This is usually a much more reliable way to subset objects: the
position of various elements can often change when chaining together
subsetting operations, but the names will always remain the same!

Unfortunately we can't skip or remove elements so easily.

To skip (or remove) a single named element:


~~~
x[-which(names(x) == "a")]
~~~
{: .r}



~~~
  b   c   d   e 
6.2 7.1 4.8 7.5 
~~~
{: .output}

The `which` function returns the indices of all `TRUE` elements of its argument.
Remember that expressions evaluate before being passed to functions. Let's break
this down so that its clearer what's happening.

First this happens:


~~~
names(x) == "a"
~~~
{: .r}



~~~
[1]  TRUE FALSE FALSE FALSE FALSE
~~~
{: .output}

The condition operator is applied to every name of the vector `x`. Only the
first name is "a" so that element is TRUE.

`which` then converts this to an index:


~~~
which(names(x) == "a")
~~~
{: .r}



~~~
[1] 1
~~~
{: .output}



Only the first element is `TRUE`, so `which` returns 1. Now that we have indices
the skipping works because we have a negative index!

Skipping multiple named indices is similar, but uses a different comparison
operator:


~~~
x[-which(names(x) %in% c("a", "c"))]
~~~
{: .r}



~~~
  b   d   e 
6.2 4.8 7.5 
~~~
{: .output}

The `%in%` goes through each element of its left argument, in this case the
names of `x`, and asks, "Does this element occur in the second argument?".

> ## Challenge 2
>
> Run the following code to define vector `x` as above:
>
> 
> ~~~
> x <- c(5.4, 6.2, 7.1, 4.8, 7.5)
> names(x) <- c('a', 'b', 'c', 'd', 'e')
> print(x)
> ~~~
> {: .r}
> 
> 
> 
> ~~~
>   a   b   c   d   e 
> 5.4 6.2 7.1 4.8 7.5 
> ~~~
> {: .output}
>
> Given this vector `x`, what would you expect the following to do?
>
>~~~
> x[-which(names(x) == "g")]
>~~~
>{: .r}
>
> Try out this command and see what you get. Did this match your expectation?
> Why did we get this result? (Tip: test out each part of the command on it's own - this is a useful debugging strategy)
>
> Which of the following are true:
>
> * A) if there are no `TRUE` values passed to `which`, an empty vector is returned
> * B) if there are no `TRUE` values passed to `which`, an error message is shown
> * C) `integer()` is an empty vector
> * D) making an empty vector negative produces an "everything" vector
> * E) `x[]` gives the same result as `x[integer()]`
>
> > ## Solution to challenge 2
> >
> > A and C are correct.
> >
> > The `which` command returns the index of every `TRUE` value in its
> > input. The `names(x) == "g"` command didn't return any `TRUE` values. Because
> > there were no `TRUE` values passed to the `which` command, it returned an
> > empty vector. Negating this vector with the minus sign didn't change its
> > meaning. Because we used this empty vector to retrieve values from `x`, it
> > produced an empty numeric vector. It was a `named numeric` empty vector
> > because the vector type of x is "named numeric" since we assigned names to the
> > values (try `str(x)` ).
> {: .solution}
{: .challenge}

> ## Tip: Non-unique names
>
> You should be aware that it is possible for multiple elements in a
> vector to have the same name. (For a data frame, columns can have
> the same name --- although R tries to avoid this --- but row names
> must be unique.) Consider these examples:
>
>
>~~~
> x <- 1:3
> x
>~~~
>{: .r}
>
>
>
>~~~
>[1] 1 2 3
>~~~
>{: .output}
>
>
>
>~~~
> names(x) <- c('a', 'a', 'a')
> x
>~~~
>{: .r}
>
>
>
>~~~
>a a a 
>1 2 3 
>~~~
>{: .output}
>
>
>
>~~~
> x['a']  # only returns first value
>~~~
>{: .r}
>
>
>
>~~~
>a 
>1 
>~~~
>{: .output}
>
>
>
>~~~
> x[which(names(x) == 'a')]  # returns all three values
>~~~
>{: .r}
>
>
>
>~~~
>a a a 
>1 2 3 
>~~~
>{: .output}
{: .callout}


> ## Tip: Getting help for operators
>
> Remember you can search for help on operators by wrapping them in quotes:
> `help("%in%")` or `?"%in%"`.
>
{: .callout}


So why can't we use `==` like before? That's an excellent question.

Let's take a look at the comparison component of this code:


~~~
names(x) == c('a', 'c')
~~~
{: .r}



~~~
Warning in names(x) == c("a", "c"): longer object length is not a multiple
of shorter object length
~~~
{: .error}



~~~
[1]  TRUE FALSE  TRUE
~~~
{: .output}

Obviously "c" is in the names of `x`, so why didn't this work? `==` works
slightly differently than `%in%`. It will compare each element of its left argument
to the corresponding element of its right argument.

Here's a mock illustration:


~~~
c("a", "b", "c", "e")  # names of x
   |    |    |    |    # The elements == is comparing
c("a", "c")
~~~
{: .r}

When one vector is shorter than the other, it gets *recycled*:


~~~
c("a", "b", "c", "e")  # names of x
   |    |    |    |    # The elements == is comparing
c("a", "c", "a", "c")
~~~
{: .r}

In this case R simply repeats `c("a", "c")` twice. If the longer
vector length isn't a multiple of the shorter vector length, then
R will also print out a warning message:


~~~
names(x) == c('a', 'c', 'e')
~~~
{: .r}



~~~
[1]  TRUE FALSE FALSE
~~~
{: .output}

This difference between `==` and `%in%` is important to remember,
because it can introduce hard to find and subtle bugs!

## Subsetting through other logical operations

We can also more simply subset through logical operations:


~~~
x[c(TRUE, TRUE, FALSE, FALSE)]
~~~
{: .r}



~~~
a a 
1 2 
~~~
{: .output}

Note that in this case, the logical vector is also recycled to the
length of the vector we're subsetting!


~~~
x[c(TRUE, FALSE)]
~~~
{: .r}



~~~
a a 
1 3 
~~~
{: .output}

Since comparison operators evaluate to logical vectors, we can also
use them to succinctly subset vectors:


~~~
x[x > 7]
~~~
{: .r}



~~~
named integer(0)
~~~
{: .output}

> ## Tip: Combining logical conditions
>
> There are many situations in which you will wish to combine multiple logical
> criteria. For example, we might want to find all the countries that are
> located in Asia **or** Europe **and** have life expectancies within a certain
> range. Several operations for combining logical vectors exist in R:
>
>  * `&`, the "logical AND" operator: returns `TRUE` if both the left and right
>    are `TRUE`.
>  * `|`, the "logical OR" operator: returns `TRUE`, if either the left or right
>    (or both) are `TRUE`.
>
> The recycling rule applies with both of these, so `TRUE & c(TRUE, FALSE, TRUE)`
> will compare the first `TRUE` on the left of the `&` sign with each of the
> three conditions on the right.
>
> You may sometimes see `&&` and `||` instead of `&` and `|`. These operators
> do not use the recycling rule: they only look at the first element of each
> vector and ignore the remaining elements. The longer operators are mainly used
> in programming, rather than data analysis.
>
>  * `!`, the "logical NOT" operator: converts `TRUE` to `FALSE` and `FALSE` to
>    `TRUE`. It can negate a single logical condition (eg `!TRUE` becomes
>    `FALSE`), or a whole vector of conditions(eg `!c(TRUE, FALSE)` becomes
>    `c(FALSE, TRUE)`).
>
> Additionally, you can compare the elements within a single vector using the
> `all` function (which returns `TRUE` if every element of the vector is `TRUE`)
> and the `any` function (which returns `TRUE` if one or more elements of the
> vector are `TRUE`).
{: .callout}

> ## Challenge 3
>
> Given the following code:
>
> 
> ~~~
> x <- c(5.4, 6.2, 7.1, 4.8, 7.5)
> names(x) <- c('a', 'b', 'c', 'd', 'e')
> print(x)
> ~~~
> {: .r}
> 
> 
> 
> ~~~
>   a   b   c   d   e 
> 5.4 6.2 7.1 4.8 7.5 
> ~~~
> {: .output}
>
> Write a subsetting command to return the values in x that are greater than 4 and less than 7.
>
> > ## Solution to challenge 3
> >
> > 
> > ~~~
> > x_subset <- x[x<7 & x>4]
> > print(x_subset)
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> >   a   b   d 
> > 5.4 6.2 4.8 
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

## Handling special values

At some point you will encounter functions in R which cannot handle missing, infinite,
or undefined data.

There are a number of special functions you can use to filter out this data:

 * `is.na` will return all positions in a vector, matrix, or data.frame
   containing `NA`.
 * likewise, `is.nan`, and `is.infinite` will do the same for `NaN` and `Inf`.
 * `is.finite` will return all positions in a vector, matrix, or data.frame
   that do not contain `NA`, `NaN` or `Inf`.
 * `na.omit` will filter out all missing values from a vector

You can read how to subset factors and matrices in the full version of this lesson at the [Software Carpentry](http://swcarpentry.github.io/r-novice-gapminder/06-data-subsetting/).

We will discuss subsetting lists and dataframes in the next lesson.

## Data frames {#data-frames}

A data frame is the most common way of storing data in R, and if [used systematically](http://vita.had.co.nz/papers/tidy-data.pdf) makes data analysis easier. Under the hood, a data frame is a list of equal-length vectors. This makes it a 2-dimensional structure, so it shares properties of both the matrix and the list.  This means that a data frame has `names()`, `colnames()`, and `rownames()`, although `names()` and `colnames()` are the same thing. The `length()` of a data frame is the length of the underlying list and so is the same as `ncol()`; `nrow()` gives the number of rows.

As described in [subsetting](#subsetting), you can subset a data frame like a 1d structure (where it behaves like a list), or a 2d structure (where it behaves like a matrix).

### Creation

You create a data frame using `data.frame()`, which takes named vectors as input:


~~~
df <- data.frame(x = 1:3, y = c("a", "b", "c"))
str(df)
~~~
{: .r}



~~~
'data.frame':	3 obs. of  2 variables:
 $ x: int  1 2 3
 $ y: Factor w/ 3 levels "a","b","c": 1 2 3
~~~
{: .output}

Beware `data.frame()`'s default behaviour which turns strings into factors. Use `stringAsFactors = FALSE` to suppress this behaviour! Compare:


~~~
df <- data.frame(
  x = 1:3,
  y = c("a", "b", "c"),
  stringsAsFactors = FALSE)
str(df)
~~~
{: .r}



~~~
'data.frame':	3 obs. of  2 variables:
 $ x: int  1 2 3
 $ y: chr  "a" "b" "c"
~~~
{: .output}


~~~
df <- data.frame(
  x = 1:3,
  y = c("a", "b", "c"))
str(df)
~~~
{: .r}



~~~
'data.frame':	3 obs. of  2 variables:
 $ x: int  1 2 3
 $ y: Factor w/ 3 levels "a","b","c": 1 2 3
~~~
{: .output}

### Testing and coercion

Because a `data.frame` is an S3 class, its type reflects the underlying vector used to build it: the list. To check if an object is a data frame, use `class()` or test explicitly with `is.data.frame()`:


~~~
typeof(df)
~~~
{: .r}



~~~
[1] "list"
~~~
{: .output}



~~~
class(df)
~~~
{: .r}



~~~
[1] "data.frame"
~~~
{: .output}



~~~
is.data.frame(df)
~~~
{: .r}



~~~
[1] TRUE
~~~
{: .output}

You can coerce an object to a data frame with `as.data.frame()`:

* A vector will create a one-column data frame.

* A list will create one column for each element; it's an error if they're 
  not all the same length.
  
* A matrix will create a data frame with the same number of columns and rows as the matrix.

> ## Discussion 3
>
> 1.  What attributes does a data frame possess?
> 
> 1.  What does `as.matrix()` do when applied to a data frame with 
>     columns of different types?
> 
> 1.  Can you have a data frame with 0 rows? What about 0 columns?
{: .discussion}
