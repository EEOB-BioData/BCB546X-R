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



**Disclaimer:** This tutorial is partially based on an excellent book [Advanced R.](https://github.com/hadley/adv-r).
Read [chapter 4](https://adv-r.hadley.nz/subsetting.html) to learn how to subset other data structures.

# Subsetting  
R has many powerful subset operators and mastering them will allow you to
easily perform complex operations on any kind of dataset.

There are six different ways we can subset any kind of object, and three
different subsetting operators for the different data structures.

Let's start with the workhorse of R: atomic vectors.

## Subsetting atomic vectors

### By position

| Code      | Meaning                |
|----------:|:-----------------------|
| `x[4]`    | The fourth element     |
| `x[-4]`   | All but the forth      |
| `x[2:4]`  | Elements two to four   |
| `x[-2:4]` | All except two to four |
| `x[c(1,5)`| Elements one and five  |

> ## Vector numbering in R starts at 1
>
> In many programming languages (C and python, for example), the first
> element of a vector has an index of 0. In R, the first element is 1.
{: .callout}

### By name

| Code      | Meaning                |
|----------:|:-----------------------|
| `x["b"]`  | An element named "b"   |


> ## Tip: Duplicated names
>
> Although inexing by name is usually a much more reliable way to subset objects, 
> notice, what will happen if we have several elements named "a":
>
> 
> ~~~
> y <- c(5.4, 6.2, 7.1, 4.8, 7.5, 6.2)
> names(y) <- c('a', 'b', 'c', 'a', 'e', 'a')
> y[c("a", "c")]
> ~~~
> {: .r}
> 
> 
> 
> ~~~
>   a   c 
> 5.4 7.1 
> ~~~
> {: .output}
>
> We'll solve this problem in the next section.
{: .callout}

### By position, value, or name using logical vectors

| Code                                 | Meaning                                            |
|-------------------------------------:|:---------------------------------------------------|
| `x[c(T,T,F)]`                        | Element at 1, 2, 4, 5, etc. postions               |
| `x[all.equal(x, 6.2)]`               | Element equal to 6.2                               |
| `x[x < 3]`                           | All elements less than three                       |
| `x[names(x) == "a"`                  | All elements with name "a"                         |
| `x[names(x) %in% c("a", "c", "d")`   | All elements with names "a", "c", or "d"           |
| `x[!(names(x) %in% c("a","c","d"))]` | All elements with names other than "a", "c", "d" |

> ## Discussion 1
>
>
> 1. Predict and check the results of the following operations:
> 
>    
>    ~~~
>    x <- c(5.4, 6.2, 7.1, 4.8, 7.5, 6.2)
>    names(x) <- c('a', 'b', 'c', 'd', 'e', 'f')
>    x[-(2:4)]
>    x[-2:4]
>    -x[2:4]
>    x[names(x) == "a"
>    ~~~
>    {: .r}
> 1. Discuss with your neighbor subsetting using logical values. How does each of these commands work?
>
> 1.  `x[names(x) == "a"]` `x[which(names(x) == "a")` produce the same results.
> How do these two commands differ from each other?
>
{: .discussion}

> ## Tip: Getting help for operators
>
> Remember you can search for help on operators by wrapping them in quotes:
> `help("%in%")` or `?"%in%"`.
>
{: .callout}

> ## Handling special values
>
> At some point you will encounter functions in R which cannot handle missing, infinite, or undefined data.
> 
> There are a number of special functions you can use to filter out this data:
>
> * `is.na` will return all positions in a vector, matrix, or data.frame containing `NA`.
> * likewise, `is.nan`, and `is.infinite` will do the same for `NaN` and `Inf`.
> * `is.finite` will return all positions in a vector, matrix, or data.frame that do not contain `NA`, `NaN` or `Inf`.
> * `na.omit` will filter out all missing values from a vector
{: .callout}


## Subsetting lists: three functions: `[`, `[[`, and `$`.

#### Use `[...]` to return a subset of a list as a list. 
If you want to *subset* a list, but not
*extract* an element, then you will likely use `[...]`.


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

#### Use `[[...]]` to *extract* an individual element of a list.


~~~
lst[[1]]
~~~
{: .r}



~~~
[1] 1 2 3
~~~
{: .output}



~~~
lst[["D"]]
~~~
{: .r}



~~~
[1] 2.3 5.9
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

#### Use `$...` to extract an element of a list by its name (you don't need "" for the name):


~~~
lst$D # The `$` function is a shorthand way for extracting elements by name
~~~
{: .r}



~~~
[1] 2.3 5.9
~~~
{: .output}

> ## Discussion 2
>
>
> 1. What is the difference between these two commands?  Did you get the results you expected?
> 
>    
>    ~~~
>    lst <- list(1:3, "a", c(TRUE, FALSE, TRUE), c(2.3, 5.9))
>    names(lst) <- c("A","B","C","D")
>    # command 1:
>    lst[1][2]
>    # command 2:
>    lst[[1]][2]
>    ~~~
>    {: .r}
{: .discussion}

## Subsetting dataframes

Remember the data frames are lists underneath the hood, so similar rules
apply. However they are also two dimensional objects!

#### Subset columns from a dataset using `[...]`. 
Remember the data frames are lists underneath the hood, so similar rules apply.  
However, the resulting object will be a data frame:


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
#### Subset cells from a dataset using `[...]` with two <sets of> numbers:
The first argument corresponds to rows and the second to columns (either or both of them can be skipped!):


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

#### Extract a single column using `[[...]]` or `$...` with the column name:


~~~
df$x
~~~
{: .r}



~~~
[1] 1 2 3
~~~
{: .output}



~~~
#or
df[["x"]]
~~~
{: .r}



~~~
[1] 1 2 3
~~~
{: .output}

> ## Discussion 3
>
> 1. What is the difference between:
> 
>    
>    ~~~
>    df[,2]
>    df[2]
>    df[[2]]
>    ~~~
>    {: .r}
> 1. How about
> 
>   
>   ~~~
>   df[3,2] #and
>   df[[2]][3]
>   ~~~
>   {: .r}
>
{: .discussion}


