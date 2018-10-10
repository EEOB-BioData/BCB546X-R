---
title: Writing and Applying Functions to Data
teaching: 20
exercises: 10
questions:
- "What are `apply` functions in R?"
- "How can I write a new function in R?"
objectives:
- "Define a function that takes arguments."
- "Return a value from a function."
- "Test a function."
- "Set default values for function arguments."
- "Explain why we should divide programs into small, single-purpose functions."
keypoints:
- "Use `function` to define a new function in R."
- "Use parameters to pass values into functions."
- "Load functions into programs using `source`."
---



## Writing and Applying Functions to Data

In this section, we’ll cover one of the cornerstones of R: how to write and apply functions to data. 
This approach of applying functions to data rather than writing explicit loops follows from a 
functional-programming style that R inherited from one of its language influences, Scheme. 
Specifically, our focus will be on applying functions to R’s lists using `lapply()` and `sapply()`, but 
the same ideas extend to other R data structures through similar “apply” functions. Solving common 
data analysis problems with apply functions is tricky at first, but mastering it will serve you well in R.

### Using lapply() with lists

Suppose you have a list of numeric values (here, generated at random with rnorm()):


~~~
ll <- list(a=rnorm(6, mean=1), b=rnorm(6, mean=4), c=rnorm(6, mean=6))
ll
~~~
{: .r}



~~~
$a
[1]  1.13999868 -0.21058268  0.94240800  1.32890303 -0.03475052  3.05773592

$b
[1] 3.000209 3.225033 3.911813 3.477822 3.206598 3.228081

$c
[1] 4.757420 5.852658 4.976225 5.902675 4.277170 6.501490
~~~
{: .output}

How might we calculate the mean of each vector stored in this list? If you’re familiar with for loops in other languages, you may approach this problem using R’s for loops:


~~~
for (i in 1:length(ll)) {
  print(mean(ll[[i]]))
}
~~~
{: .r}



~~~
[1] 1.037285
[1] 3.341593
[1] 5.37794
~~~
{: .output}


However, this is not idiomatic R; a better approach is to use an `apply` function that applies another function to each list element.

> ## Apply functions
>
> * **apply** is used to apply a function to a matrix in row wise or column wise: `apply(x, margin, function)`
>
> > ## Example
> > 
> > 
> > ~~~
> > m <- matrix( c(1,2,3,4),2,2 )
> > apply(m,1,sum) #margin = 1 indicates rows, margin = 2 indicates columns
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> > [1] 4 6
> > ~~~
> > {: .output}
> {: .solution}
>
> * **lapply** takes a list as an input, and returns a list as the output
>
> > ## Example
> > 
> > 
> > ~~~
> > list <- list(a = c(1,1), b=c(2,2), c=c(3,3)) 
> > lapply(list,sum) 
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> > $a
> > [1] 2
> > 
> > $b
> > [1] 4
> > 
> > $c
> > [1] 6
> > ~~~
> > {: .output}
> {: .solution}
>
> * **sapply** is the same as `lapply`, but returns a vector instead of a list
>
> > ## Example
> > 
> > 
> > ~~~
> > list <- list(a = c(1,1), b=c(2,2), c=c(3,3)) 
> > sapply(list,sum) 
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> > a b c 
> > 2 4 6 
> > ~~~
> > {: .output}
> {: .solution}
>
> * **tapply** splits a vector based on factor levels and then applies a function to it:
>
> > ## Example
> > 
> > 
> > ~~~
> > tapply(mtcars$wt, mtcars$cyl, mean)
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> >        4        6        8 
> > 2.285727 3.117143 3.999214 
> > ~~~
> > {: .output}
> {: .solution}
>
> * **vapply** is the same as sapply except that you need to specify the type of return value
>
> > ## Example
> > 
> > 
> > ~~~
> > list <- list(a = c(1,1), b=c(2,2), c=c(3,3)) 
> > vapply(list, sum, FUN.VALUE=double(1)) 
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> > a b c 
> > 2 4 6 
> > ~~~
> > {: .output}
> {: .solution}
>
> * **mapply** is a multivariate version of sapply. It will apply the specified function to the first element of each argument first, followed by the second element, and so on.
>
> > ## Example
> > 
> > 
> > ~~~
> > mapply(rep, 1:4, 4:1) #same as list(rep(1, 4), rep(2, 3), rep(3, 2), rep(4, 1))
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> > [[1]]
> > [1] 1 1 1 1
> > 
> > [[2]]
> > [1] 2 2 2
> > 
> > [[3]]
> > [1] 3 3
> > 
> > [[4]]
> > [1] 4
> > ~~~
> > {: .output}
> {: .solution}
>
{: .callout}

For example, 
to calculate the mean of each list element, we’d want to apply the function `mean()` to each element. To do so, 
we can use the function `lapply()` (the l is for list):


~~~
lapply(ll, mean)
~~~
{: .r}



~~~
$a
[1] 1.037285

$b
[1] 3.341593

$c
[1] 5.37794
~~~
{: .output}

`lapply()` has several advantages over the loop shown above: it creates the output list for us, uses fewer lines of code, leads to clearer 
code, and is in some cases faster than using a for loop. We can also use `lapply()` in parallel (see p. 233 of the textbook for details)

> ## Tips for `apply` functions
>
> 1) Don’t call the function you pass to lapply()—for example, don’t do `lapply(ll, mean(x))` or 
> `lapply(ll, mean()). 
>
> 2) In some cases you will need to specify additional arguments to the function you’re passing 
> to `lapply()`. For example, mean() returns NA if some data are missing.  We ignore NA values by
> calling `mean()` with the argument `na.rm=TRUE`:
>
> 
> ~~~
> ll[[c(3,2)]] <- NA
> lapply(ll, mean)
> ~~~
> {: .r}
> 
> 
> 
> ~~~
> $a
> [1] 1.037285
> 
> $b
> [1] 3.341593
> 
> $c
> [1] NA
> ~~~
> {: .output}
> 
> 
> 
> ~~~
> lapply(ll, mean, na.rm=TRUE)
> ~~~
> {: .r}
> 
> 
> 
> ~~~
> $a
> [1] 1.037285
> 
> $b
> [1] 3.341593
> 
> $c
> [1] 5.282996
> ~~~
> {: .output}
>
{: .callout}

### Writing functions.

> ## What is a function?
>
> Functions gather a sequence of operations into a whole, preserving it for ongoing use. Functions provide:
>
> * a name we can remember and invoke it by
> * relief from the need to remember the individual operations
> * a defined set of inputs and expected outputs
> * rich connections to the larger programming environment
>
> As the basic building block of most programming languages, user-defined
> functions constitute "programming" as much as any single abstraction can. If
> you have written a function, you are a computer programmer.
{: .callout}

Sometimes, we don't have an exact function that we want to apply to our data. But we can just write it :) 
R makes writing functions very easy, both because you should be writing lots of them to organize your code and 
applying functions is such a common operation in R. For example, we could write a simple version of R’s mean() 
function with na.rm=TRUE and apply it to the list:


~~~
meanRemoveNA <- function(x) mean(x, na.rm=TRUE)
lapply(ll, meanRemoveNA)
~~~
{: .r}



~~~
$a
[1] 1.037285

$b
[1] 3.341593

$c
[1] 5.282996
~~~
{: .output}

The syntax for meanRemoveNA() is a common shortened version of the general syntax for R functions:

~~~
fun_name <- function(args) {
# body, containing R expressions return(value)
}
~~~
{: .r}

Function definitions consist of arguments, a body, and a return value. Functions that contain only one line 
in their body can omit the braces (as the meanRemoveNA() function does). Similarly, using return() to specify 
the return value is optional; R’s functions will automatically return the last evaluated expression in the body. 
These syntactic shortcuts are commonly used in R, as we often need to quickly write functions to apply to data.

Alternatively, we could forgo creating a function named meanRemoveNA() in our global environment altogether 
and instead use an *anonymous function*. Anonymous functions are useful when we only need a function once for 
a specific task:


~~~
lapply(ll, function(x) mean(x, na.rm=TRUE))
~~~
{: .r}



~~~
$a
[1] 1.037285

$b
[1] 3.341593

$c
[1] 5.282996
~~~
{: .output}

In other cases, we might need to create polished functions that we’ll use repeatedly throughout our code. 
In theses cases, it pays off to carefully document your functions and add some extra features.


~~~
meanRemoveNAVerbose <- function(x, warn=TRUE) {
# A function that removes missing values when calculating the mean # and warns us about it.
if (any(is.na(x)) && warn) { warning("removing some missing values!")
}
mean(x, na.rm=TRUE)
}
~~~
{: .r}

Note, that functions over one line should be kept in a file and sent to the interpreter. RStudio conveniently 
allows you to send a whole function definition at once with Command-Option-f on a Mac and Control-Alt-f on Windows.

If you've been writing these functions down into a separate R script
(a good idea!), you can load in the functions into our R session by using the
`source` function:


~~~
source("functions/functions-lesson.R")
~~~
{: .r}


> ## Function tips
> **Pass by value**
>
> Functions in R almost always make copies of the data to operate on
> inside of a function body. When we modify these data inside the function
> we are modifying the copy of them,
> not the original variable we gave as the first argument.
>
> This is called "pass-by-value" and it makes writing code much safer:
> you can always be sure that whatever changes you make within the
> body of the function, stay inside the body of the function.
>
> **Function scope**
>
> Another important concept is scoping: any variables (or functions!) you
> create or modify inside the body of a function only exist for the lifetime
> of the function's execution. Even if we
> have variables of the same name in our interactive R session, they are
> not modified in any way when executing a function.
>
> **Additional resources**
>
> R has some unique aspects that can be exploited when performing
> more complicated operations. We will not be writing anything that requires
> knowledge of these more advanced concepts. In the future when you are
> comfortable writing functions in R, you can learn more by reading the
> [R Language Manual][man] or this [chapter][] from
> [Advanced R Programming][adv-r] by Hadley Wickham. For context, R uses the
> terminology "environments" instead of frames.
{: .callout}

[man]: http://cran.r-project.org/doc/manuals/r-release/R-lang.html#Environment-objects
[chapter]: http://adv-r.had.co.nz/Environments.html
[adv-r]: http://adv-r.had.co.nz/


> ## Tip4: Testing and documenting
>
> It's important to both test functions and document them:
> Documentation helps you, and others, understand what the
> purpose of your function is, and how to use it, and its
> important to make sure that your function actually does
> what you think.
>
> When you first start out, your workflow will probably look a lot
> like this:
>
>  1. Write a function
>  2. Comment parts of the function to document its behaviour
>  3. Load in the source file
>  4. Experiment with it in the console to make sure it behaves
>     as you expect
>  5. Make any necessary bug fixes
>  6. Rinse and repeat.
>
> Formal documentation for functions, written in separate `.Rd`
> files, gets turned into the documentation you see in help
> files. The [roxygen2][] package allows R coders to write documentation alongside
> the function code and then process it into the appropriate `.Rd` files.
> You will want to switch to this more formal method of writing documentation
> when you start writing more complicated R projects.
>
> Formal automated tests can be written using the [testthat][] package.
{: .callout}

[roxygen2]: http://cran.r-project.org/web/packages/roxygen2/vignettes/rd.html
[testthat]: http://r-pkgs.had.co.nz/tests.html
