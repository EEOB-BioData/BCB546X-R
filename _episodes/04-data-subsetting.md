---
title: "Data Subsetting"
teaching: 30
exercises: 10
questions:
- "How can I work with subsets of data in R?"
objectives:
- "To be able to subset vectors, factors, lists, and dataframes"
- "To be able to extract individual and multiple elements: by index, by name, using comparison operations"
- "To be able to skip and remove elements from various data structures."
keypoints:
- "Access individual values by location using `[]`."
- "Access slices of data using `[low:high]`."
- "Access arbitrary sets of data using `[c(...)]`."
- "Use `which` to select subsets of data based on value."
---



**Disclaimer:** This tutorial is based on an excellent book [Advanced R.](https://github.com/hadley/adv-r)

# Subsetting  
R has many powerful subset operators and mastering them will allow you to
easily perform complex operations on any kind of dataset.

There are six different ways we can subset any kind of object, and three
different subsetting operators for the different data structures.

Let's start with the workhorse of R: atomic vectors.

## Subsetting atomic vectors


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

#### 1) Positive integers return elements at the specified positions

Getting a single element:


~~~
x[1]; x[4]
~~~
{: .r}



~~~
  a 
5.4 
~~~
{: .output}



~~~
  d 
4.8 
~~~
{: .output}

Getting multiple elements at once:


~~~
# Listing positions of interest:
x[c(1, 3)] #note c() function and try x[1,3]!
~~~
{: .r}



~~~
  a   c 
5.4 7.1 
~~~
{: .output}



~~~
# Or slices of the vector:
x[1:4]
~~~
{: .r}



~~~
  a   b   c   d 
5.4 6.2 7.1 4.8 
~~~
{: .output}



~~~
# We can ask for the same element multiple times:
x[c(1,1,3)]
~~~
{: .r}



~~~
  a   a   c 
5.4 5.4 7.1 
~~~
{: .output}

Note, that if we ask for a number outside of the vector, R will return a missing value(s) (a vector of length one containing an `NA`, whose name is also `NA`).


~~~
x[6]
~~~
{: .r}



~~~
<NA> 
  NA 
~~~
{: .output}

> ## Vector numbering in R starts at 1
>
> In many programming languages (C and python, for example), the first
> element of a vector has an index of 0. In R, the first element is 1.
{: .callout}

#### 2) Negative integers omit elements at the specified positions:

To skip one element:


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

We can skip a slice of the vector:


~~~
x[-(1:3)] #But note, that the following won't work:
~~~
{: .r}



~~~
  d   e 
4.8 7.5 
~~~
{: .output}



~~~
x[-1:3] #Why?
~~~
{: .r}



~~~
Error in x[-1:3]: only 0's may be mixed with negative subscripts
~~~
{: .error}

> ## Tip: Modifying a vector
>
> To modify a vector, we need to assign the results back into it:
>
> 
> ~~~
> x <- x[-4]
> x
> ~~~
> {: .r}
> 
> 
> 
> ~~~
>   a   b   c   e 
> 5.4 6.2 7.1 7.5 
> ~~~
> {: .output}
{: .callout}

#### 3) Subsetting by name

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

However, notice, what will happen if we have several elements named "a":


~~~
x <- c(5.4, 6.2, 7.1, 4.8, 7.5)
names(x) <- c('a', 'b', 'c', 'a', 'e')
x[c("a", "c")]
~~~
{: .r}



~~~
  a   c 
5.4 7.1 
~~~
{: .output}

To get all elements called "a" or to skip an element called "a" we have to use
the `names` function (you can also use `which(names())`):


~~~
x[names(x) == "a"] # This is the same as x[which(names(x) == "a")]
~~~
{: .r}



~~~
  a   a 
5.4 4.8 
~~~
{: .output}



~~~
x[names(x) != "a"] # This is the same as x[-which(names(x) == "a")]
~~~
{: .r}



~~~
  b   c   e 
6.2 7.1 7.5 
~~~
{: .output}

Because `names(x) == "a"` returns a logical vector, let's go to the next section to see how it works.

#### 4. Logical vectors select elements where the corresponding logical value is TRUE. 

This is probably the most useful type of subsetting because you write the expression that creates the logical vector:



~~~
x[c(TRUE, TRUE, FALSE, FALSE)]
~~~
{: .r}



~~~
  a   b   e 
5.4 6.2 7.5 
~~~
{: .output}

Note that in this case, the logical vector is also recycled to the
length of the vector we're subsetting!


~~~
x[c(TRUE, FALSE)]
~~~
{: .r}



~~~
  a   c   e 
5.4 7.1 7.5 
~~~
{: .output}

Since comparison operators evaluate to logical vectors, we can also
use them to succinctly subset vectors:


~~~
x[x > 7]
~~~
{: .r}



~~~
  c   e 
7.1 7.5 
~~~
{: .output}



~~~
x[names(x) != "a"]
~~~
{: .r}



~~~
  b   c   e 
6.2 7.1 7.5 
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

While you can combine logical conditions to search for elements with a set of names, a cleaner solution is to use `%in%` function:


~~~
x[names(x) == "a" | names(x) == "b"]
~~~
{: .r}



~~~
  a   b   a 
5.4 6.2 4.8 
~~~
{: .output}



~~~
x[names(x) %in% c("a","b")] # You can also use a vector with values of interest
~~~
{: .r}



~~~
  a   b   a 
5.4 6.2 4.8 
~~~
{: .output}

The `%in%` goes through each element of its left argument, in this case the
names of `x`, and asks, "Does this element occur in the second argument?". 

So what if want to exlude a set of elements by name?


~~~
x[names(x) != "a" & names(x) != "b"]
~~~
{: .r}



~~~
  c   e 
7.1 7.5 
~~~
{: .output}



~~~
x[!(names(x) %in% c("a","b"))] # You can also use a vector with values of interest
~~~
{: .r}



~~~
  c   e 
7.1 7.5 
~~~
{: .output}

> ## Tip: Getting help for operators
>
> Remember you can search for help on operators by wrapping them in quotes:
> `help("%in%")` or `?"%in%"`.
>
{: .callout}

### Handling special values

At some point you will encounter functions in R which cannot handle missing, infinite, or undefined data.

There are a number of special functions you can use to filter out this data:

 * `is.na` will return all positions in a vector, matrix, or data.frame
   containing `NA`.
 * likewise, `is.nan`, and `is.infinite` will do the same for `NaN` and `Inf`.
 * `is.finite` will return all positions in a vector, matrix, or data.frame
   that do not contain `NA`, `NaN` or `Inf`.
 * `na.omit` will filter out all missing values from a vector

You can read how to subset factors and matrices in the full version of this lesson at the [Software Carpentry](http://swcarpentry.github.io/r-novice-gapminder/06-data-subsetting/).

## Subsetting lists: three functions: `[`, `[[`, and `$`.

Using `[` will always return a list. If you want to *subset* a list, but not
*extract* an element, then you will likely use `[`.


~~~
lst <- list(1:3, "a", c(TRUE, FALSE, TRUE), c(2.3, 5.9))
names(lst) <- c("A","B","C","D")
lst[1]
~~~
{: .r}



~~~
$A
[1] 1 2 3
~~~
{: .output}



~~~
str(lst[1])
~~~
{: .r}



~~~
List of 1
 $ A: int [1:3] 1 2 3
~~~
{: .output}

To *extract* individual elements of a list, you need to use the double-square
bracket function: `[[`.


~~~
lst[[1]]
~~~
{: .r}



~~~
[1] 1 2 3
~~~
{: .output}



~~~
str(lst[[1]])
~~~
{: .r}



~~~
 int [1:3] 1 2 3
~~~
{: .output}

You **can't** extract more than one element at once **nor** use `[[ ]]` to skip elements:


~~~
lst[[1:3]]
~~~
{: .r}



~~~
Error in lst[[1:3]]: recursive indexing failed at level 2
~~~
{: .error}



~~~
lst[[-1]]
~~~
{: .r}



~~~
Error in lst[[-1]]: attempt to select more than one element in get1index <real>
~~~
{: .error}

You can use elements' names to both subset and extract elements:


~~~
lst[["D"]]
~~~
{: .r}



~~~
[1] 2.3 5.9
~~~
{: .output}



~~~
lst$D # The `$` function is a shorthand way for extracting elements by name
~~~
{: .r}



~~~
[1] 2.3 5.9
~~~
{: .output}

## Subsetting dataframes

Remember the data frames are lists underneath the hood, so similar rules
apply. However they are also two dimensional objects!

So `[` with one argument will act the same was as for lists, where each
element corresponds to a column. The resulting object will be a data frame:


~~~
df <- data.frame(
  x = 1:3,
  y = c("a", "b", "c"),
  z = c(TRUE,FALSE,TRUE),
  stringsAsFactors = FALSE)
str(df)
~~~
{: .r}



~~~
'data.frame':	3 obs. of  3 variables:
 $ x: int  1 2 3
 $ y: chr  "a" "b" "c"
 $ z: logi  TRUE FALSE TRUE
~~~
{: .output}



~~~
df[1]
~~~
{: .r}



~~~
  x
1 1
2 2
3 3
~~~
{: .output}

However, `[` can also take two arguments, exctracting rows (first argument) and columns (second argument) (either or both of them can be skipped!):


~~~
df[1:2,2:3]
~~~
{: .r}



~~~
  y     z
1 a  TRUE
2 b FALSE
~~~
{: .output}

If we subset a single row, the result will be a data frame (because the elements are mixed types):


~~~
df[3,]
~~~
{: .r}



~~~
  x y    z
3 3 c TRUE
~~~
{: .output}

But for a single column the result will be a vector (this can be changed with the third argument, `drop = FALSE`).


~~~
df[,3]
~~~
{: .r}



~~~
[1]  TRUE FALSE  TRUE
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

