---
title: "Data Structures"
teaching: 50
exercises: 30
questions:
- "What are the basic data types in R?"
- "How do I represent categorical information in R?"
objectives:
- "To be aware of the different types of data."
- "To begin exploring data frames, and understand how it's related to vectors, factors and lists."
- "To be able to ask questions from R about the type, class, and structure of an object."
keypoints:
- "The basic data types in R are double, integer, complex, logical, and character."
- "Use factors to represent categories in R."
- "Indexing in R starts at 1, not 0."
---



**Disclaimer:** This tutorial is based on an excellent book [Advanced R.](https://github.com/hadley/adv-r)


# Data structures {#data-structures}


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

There are four common types of atomic vectors : **logical**, **integer**, **double** (often called numeric), and **character**.

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
> Where do these data come from?
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

### Attributes {#attributes}

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
x <- setNames(x, c("a", "b", "c"))
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

#### Factors

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

Factors crip up all over R, and occasionally cause headaches for new R users (see below).

#### Matrices and arrays {#matrices-and-arrays}

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

## Data frames {#data-frames}

A data frame is the most common way of storing data in R, and if [used systematically](http://vita.had.co.nz/papers/tidy-data.pdf) makes data analysis easier. Under the hood, a data frame is a list of equal-length vectors. This makes it a 2-dimensional structure, so it shares properties of both the matrix and the list.

> ## Useful Data Frame Functions
>
> * `head()` - shows first 6 rows
> * `tail()` - shows last 6 rows
> * `dim()` - returns the dimensions of data frame (i.e. number of rows and number of columns)
> * `nrow()` - number of rows
> * `ncol()` - number of columns
> * `str()` - structure of data frame - name, type and preview of data in each column
> * `names()` - shows the `names` attribute for a data frame, which gives the column names.
> * `sapply(dataframe, class)` - shows the class of each column in the data frame
{: .callout}

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

Beware `data.frame()`'s default behaviour which turns strings into factors. Use `stringAsFactors = FALSE` to suppress this behaviour!


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

### Testing and coercion

Because a `data.frame` is an S3 class, its type reflects the underlying vector used to build it: the list. To check if an object is a data frame, use `class()` or test explicitly with `is.data.frame()`:


~~~
is.vector(df)
~~~
{: .r}



~~~
[1] FALSE
~~~
{: .output}



~~~
is.list(df)
~~~
{: .r}



~~~
[1] TRUE
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
