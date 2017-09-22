---
title: Functions
teaching: 20
exercises: 10
questions:
- "How functions are constructed and function :) in R?"
objectives:
- "Describe three main components of a function."
- "Understand how R finds values from names (lexical scoping)"
- "Understand that every operation in R is a function call"
- "Learn three ways of supplying arguments to a function"
- "Understand infix and replacement functions."
- "Understand how and when functions return values"
keypoints:
- "All R functions have three parts: the `body()`, the `formals()`, and the `environment()`"
- "The four basic principles behind R's implementation of lexical scoping are name masking, functions vs. variables, a fresh start, and dynamic lookup"
- "Load functions into programs using `source`."
---



This exercise is a slighly modified version of [Chapter 6 from Advanced R book](http://adv-r.had.co.nz/Functions.html) 
by Hadley Wickham.

# Functions

Functions are a fundamental building block of R: to master many of the more 
advanced techniques in R, you need a solid foundation in how functions work.

One important thing to understand about R is that functions are objects 
in their own right. You can work with them exactly the same way you work with 
any other type of object.

##### Outline

* [Function components](#function-components) describes the three main 
  components of a function.

* [Lexical scoping](#lexical-scoping) teaches you how R finds values from 
  names, the process of lexical scoping.

* [Every operation is a function call](#all-calls) shows you that everything
  that happens in R is a result of a function call, even if it doesn't look 
  like it.

* [Function arguments](#function-arguments) discusses the three ways of 
  supplying arguments to a function, how to call a function given a list of 
  arguments, and the impact of lazy evaluation.

* [Special calls](#special-calls) describes two special types of function: 
  infix and replacement functions.
  
* [Return values](#return-values) discusses how and when functions return
  values, and how you can ensure that a function does something before it
  exits.

##### Prerequisites

The only package you'll need is `pryr`, which is used to explore what happens 
when modifying vectors in place. Install it with `install.packages("pryr")`.

## Function components {#function-components}

All R functions have three parts:

* the `body()`, the code inside the function.
* the `formals()`, the list of arguments which controls how you can call the function.
* the `environment()`, the "map" of the location of the function's variables.

When you print a function in R, it shows you these three important components. 
If the environment isn't displayed, it means that the function was created in 
the global environment


~~~
f <- function(x) x^2
f
#> function(x) x^2

formals(f)
#> $x
body(f)
#> x^2
environment(f)
#> <environment: R_GlobalEnv>
~~~
{: .r}

The assignment forms of `body()`, `formals()`, and `environment()` can also be 
used to modify functions.

Like all objects in R, functions can also possess any number of additional 
`attributes()`. For example, you can set the `class()` and add a custom `print()` method.

### Primitive functions

There is one exception to the rule that functions have three components. 
Primitive functions, like `sum()`, call C code directly with `.Primitive()` 
and contain no R code. Therefore their `formals()`, `body()`, and 
`environment()` are all `NULL`:


~~~
sum
~~~
{: .r}



~~~
function (..., na.rm = FALSE)  .Primitive("sum")
~~~
{: .output}



~~~
formals(sum)
~~~
{: .r}



~~~
NULL
~~~
{: .output}



~~~
body(sum)
~~~
{: .r}



~~~
NULL
~~~
{: .output}



~~~
environment(sum)
~~~
{: .r}



~~~
NULL
~~~
{: .output}

Primitive functions are only found in the `base` package, and since they 
operate at a low level, they can be more efficient (primitive replacement 
functions don't have to make copies), and can have different rules for argument 
matching (e.g., `switch` and `call`).

### Exercises

1.  What function allows you to tell if an object is a function? What function
    allows you to tell if a function is a primitive function?

1.  This code makes a list of all functions in the base package. 
    
    
    ~~~
    objs <- mget(ls("package:base"), inherits = TRUE)
    funs <- Filter(is.function, objs)
    ~~~
    {: .r}

    Use it to answer the following questions:

    a. Which base function has the most arguments?
    
    a. How many base functions have no arguments? What's special about those
       functions?
       
    a. How could you adapt the code to find all primitive functions?

1. What are the three important components of a function?

1. When does printing a function not show what environment it was created in?

## Lexical scoping {#lexical-scoping}

Scoping is the set of rules that govern how R looks up the value of a symbol. 
In the example below, scoping is the set of rules that R applies to go from 
the symbol `x` to its value `10`:


~~~
x <- 10
x
~~~
{: .r}



~~~
[1] 10
~~~
{: .output}

R has two types of scoping: __lexical scoping__, implemented automatically 
at the language level, and __dynamic scoping__, used in select functions to 
save typing during interactive analysis. We discuss lexical scoping here.

Lexical scoping looks up symbol values based on how functions were nested when 
they were created, not how they are nested when they are called. The "lexical" 
in lexical scoping comes from the computer science term "lexing", which is part 
of the process that converts code represented as text to meaningful pieces that 
the programming language understands.

There are four basic principles behind R's implementation of lexical scoping:

* name masking
* functions vs. variables
* a fresh start
* dynamic lookup

### Name masking

The following example illustrates the most basic principle of lexical scoping, 
and you should have no problem predicting the output.


~~~
x <- 25
y <- 30
f <- function() {
  x <- 1
  y <- 2
  c(x, y)
}
f()
rm(f)
c(x, y)
rm(x, y)
~~~
{: .r}

If a name isn't defined inside a function, R will look one level up.


~~~
x <- 2
g <- function() {
  y <- 1
  c(x, y)
}
g()
rm(x, g)
~~~
{: .r}

The same rules apply if a function is defined inside another function: look inside the current function, then where that function was defined, and so on, all the way up to the global environment, and then on to other loaded packages.


~~~
x <- 1
h <- function() {
  y <- 2
  i <- function() {
    z <- 3
    c(x, y, z)
  }
  i()
}
h()
rm(x, h)
~~~
{: .r}

The same rules apply to closures, functions created by other functions. The 
following function, `j()`, returns a function.  What do you think this function 
will return when we call it? 


~~~
j <- function(x) {
  y <- 2
  function() {
    c(x, y)
  }
}
k <- j(1)
k
k()
rm(j, k)
~~~
{: .r}

This seems a little magical (how does R know what the value of `y` is after the 
function has been called). It works because `k` preserves the environment in 
which it was defined and because the environment includes the value of `y`.

### Functions vs. variables

The same principles apply regardless of the type of associated value --- finding 
functions works exactly the same way as finding variables:


~~~
l <- function(x) x + 1
m <- function() {
  l <- function(x) x * 2
  l(10)
}
m()
~~~
{: .r}



~~~
[1] 20
~~~
{: .output}



~~~
rm(l, m)
~~~
{: .r}

For functions, there is one small tweak to the rule. If you are using a name in 
a context where it's obvious that you want a function (e.g., `f(3)`), R will 
ignore objects that are not functions while it is searching. In the following 
example `n` takes on a different value depending on whether R is looking for 
a function or a variable.


~~~
n <- function(x) x / 2
o <- function() {
  n <- 10
  n(n)
}
o()
~~~
{: .r}



~~~
[1] 5
~~~
{: .output}



~~~
rm(n, o)
~~~
{: .r}

However, using the same name for functions and other objects will make for 
confusing code, and is generally best avoided.

### A fresh start {#fresh-start}

What happens to the values in between invocations of a function? What will 
happen the first time you run this function? What will happen the second time? 
(If you haven't seen `exists()` before: it returns `TRUE` if there's a variable 
of that name, otherwise it returns `FALSE`.)


~~~
j <- function() {
  if (!exists("a")) {
    a <- 1
  } else {
    a <- a + 1
  }
  a
}
j()
rm(j)
~~~
{: .r}

You might be surprised that it returns the same value, `1`, every time. This is 
because every time a function is called, a new environment is created to host 
execution. A function has no way to tell what happened the last time it was run; 
each invocation is completely independent. (We'll see some ways to get around 
this in [mutable state](#mutable-state).)

### Dynamic lookup

Lexical scoping determines where to look for values, not when to look for them. 
R looks for values when the function is run, not when it's created. This means 
that the output of a function can be different depending on objects outside its 
environment: 


~~~
f <- function() x
x <- 15
f()
~~~
{: .r}



~~~
[1] 15
~~~
{: .output}



~~~
x <- 20
f()
~~~
{: .r}



~~~
[1] 20
~~~
{: .output}

You generally want to avoid this behaviour because it means the function is no 
longer self-contained. This is a common error --- if you make a spelling mistake 
in your code, you won't get an error when you create the function, and you might 
not even get one when you run the function, depending on what variables are 
defined in the global environment.

One way to detect this problem is the `findGlobals()` function from `codetools`. 
This function lists all the external dependencies of a function: 


~~~
f <- function() x + 1
codetools::findGlobals(f)
~~~
{: .r}



~~~
[1] "+" "x"
~~~
{: .output}

Another way to try and solve the problem would be to manually change the 
environment of the function to the `emptyenv()`, an environment which contains 
absolutely nothing:


~~~
environment(f) <- emptyenv()
f()
~~~
{: .r}



~~~
Error in f(): could not find function "+"
~~~
{: .error}

This doesn't work because R relies on lexical scoping to find _everything_, 
even the `+` operator. It's never possible to make a function completely self-
contained because you must always rely on functions defined in base R or other 
packages.

### Exercises

1. What does the following code return? Why? What does each of the three `c`'s mean?

    
    ~~~
    c <- 10
    c(c = c)
    ~~~
    {: .r}

2. What are the four principles that govern how R looks for values?

3. What does the following function return? Make a prediction before 
   running the code yourself.

    
    ~~~
    f <- function(x) {
      f <- function(x) {
        f <- function(x) {
          x ^ 2
        }
        f(x) + 1
      }
      f(x) * 2
    }
    f(10)
    ~~~
    {: .r}

## Every operation is a function call {#all-calls}

> "To understand computations in R, two slogans are helpful:
>
> * Everything that exists is an object.
> * Everything that happens is a function call."
>
> --- John Chambers

Every operation in R is a function call, whether or not it looks like one. This 
includes infix operators like `+`, control flow operators like `for`, `if`, and 
`while`, subsetting operators like `[]` and `$`, and even the curly brace `{`. 
This means that each pair of statements in the following example is exactly 
equivalent.  Note that `` ` ``, the backtick, lets you refer to functions or 
variables that have otherwise reserved or illegal names: \index{reserved names} 
\indexc{`} \index{backticks|see{\texttt{`}}}


~~~
x <- 10; y <- 5
x + y
~~~
{: .r}



~~~
[1] 15
~~~
{: .output}



~~~
`+`(x, y)
~~~
{: .r}



~~~
[1] 15
~~~
{: .output}



~~~
for (i in 1:2) print(i)
~~~
{: .r}



~~~
[1] 1
[1] 2
~~~
{: .output}



~~~
`for`(i, 1:2, print(i))
~~~
{: .r}



~~~
[1] 1
[1] 2
~~~
{: .output}



~~~
if (i == 1) print("yes!") else print("no.")
~~~
{: .r}



~~~
[1] "no."
~~~
{: .output}



~~~
`if`(i == 1, print("yes!"), print("no."))
~~~
{: .r}



~~~
[1] "no."
~~~
{: .output}



~~~
x[3]
~~~
{: .r}



~~~
[1] NA
~~~
{: .output}



~~~
`[`(x, 3)
~~~
{: .r}



~~~
[1] NA
~~~
{: .output}



~~~
{ print(1); print(2); print(3) }
~~~
{: .r}



~~~
[1] 1
[1] 2
[1] 3
~~~
{: .output}



~~~
`{`(print(1), print(2), print(3))
~~~
{: .r}



~~~
[1] 1
[1] 2
[1] 3
~~~
{: .output}

It is possible to override the definitions of these special functions, but this 
is almost certainly a bad idea. It's more often useful to treat special functions 
as ordinary functions. For example, we could use `sapply()` to add 3 to every 
element of a list by first defining a function `add()`, like this: \indexc{sapply()}


~~~
add <- function(x, y) x + y
sapply(1:10, add, 3)
~~~
{: .r}



~~~
 [1]  4  5  6  7  8  9 10 11 12 13
~~~
{: .output}

But we can also get the same effect using the built-in `+` function.


~~~
sapply(1:5, `+`, 3)
~~~
{: .r}



~~~
[1] 4 5 6 7 8
~~~
{: .output}



~~~
sapply(1:5, "+", 3)
~~~
{: .r}



~~~
[1] 4 5 6 7 8
~~~
{: .output}

Note the difference between ``+`` and `"+"`.  The first one is the value of the 
object called `+`, and the second is a string containing the character `+`.  
The second version works because `sapply` can be given the name of a function 
instead of the function itself.

A more useful application is to combine `lapply()` or `sapply()` with subsetting:


~~~
x <- list(1:3, 4:9, 10:12)
sapply(x, "[", 2)
~~~
{: .r}



~~~
[1]  2  5 11
~~~
{: .output}



~~~
# equivalent to
sapply(x, function(x) x[2])
~~~
{: .r}



~~~
[1]  2  5 11
~~~
{: .output}

## Function arguments {#function-arguments}

It's useful to distinguish between the _formal_ arguments and the _actual_ arguments 
of a function. The formal arguments are a property of the function, whereas the 
actual or calling arguments can vary each time you call the function.

### Calling functions

When calling a function you can specify arguments by position, by complete name, 
or by partial name. Arguments are matched first by exact name (perfect matching), 
then by prefix matching, and finally by position. \index{functions!arguments}


~~~
f <- function(abcdef, bcde1, bcde2) {
  list(a = abcdef, b1 = bcde1, b2 = bcde2)
}
str(f(1, 2, 3))
~~~
{: .r}



~~~
List of 3
 $ a : num 1
 $ b1: num 2
 $ b2: num 3
~~~
{: .output}



~~~
str(f(2, 3, abcdef = 1))
~~~
{: .r}



~~~
List of 3
 $ a : num 1
 $ b1: num 2
 $ b2: num 3
~~~
{: .output}



~~~
# Can abbreviate long argument names:
str(f(2, 3, a = 1))
~~~
{: .r}



~~~
List of 3
 $ a : num 1
 $ b1: num 2
 $ b2: num 3
~~~
{: .output}



~~~
# But this doesn't work because abbreviation is ambiguous
str(f(1, 3, b = 1))
~~~
{: .r}



~~~
Error in f(1, 3, b = 1): argument 3 matches multiple formal arguments
~~~
{: .error}

If a function uses `...` (discussed in more detail below), you can only specify 
arguments listed after `...` with their full name.

### Calling a function given a list of arguments

Suppose you had a list of function arguments: \indexc{do.call()}


~~~
args <- list(1:10, na.rm = TRUE)
~~~
{: .r}

You then can send that list to `mean()` with `do.call()`:


~~~
do.call(mean, args)
~~~
{: .r}



~~~
[1] 5.5
~~~
{: .output}



~~~
# Equivalent to
mean(1:10, na.rm = TRUE)
~~~
{: .r}



~~~
[1] 5.5
~~~
{: .output}

### Default and missing arguments

Function arguments in R can have default values. \index{functions!default values}


~~~
f <- function(a = 1, b = 2) {
  c(a, b)
}
f()
~~~
{: .r}



~~~
[1] 1 2
~~~
{: .output}

You can determine if an argument was supplied or not with the `missing()` function. 
\indexc{missing()}


~~~
i <- function(a, b) {
  c(missing(a), missing(b))
}
i()
~~~
{: .r}



~~~
[1] TRUE TRUE
~~~
{: .output}



~~~
i(a = 1)
~~~
{: .r}



~~~
[1] FALSE  TRUE
~~~
{: .output}



~~~
i(b = 2)
~~~
{: .r}



~~~
[1]  TRUE FALSE
~~~
{: .output}



~~~
i(1, 2)
~~~
{: .r}



~~~
[1] FALSE FALSE
~~~
{: .output}

### Lazy evaluation {#lazy-evaluation}

By default, R function arguments are lazy --- they're only evaluated if they're actually used: 
\index{lazy evaluation} \index{functions!lazy evaluation}


~~~
f <- function(x) {
  10
}
f(stop("This is an error!"))
~~~
{: .r}



~~~
[1] 10
~~~
{: .output}

If you want to ensure that an argument is evaluated you can use `force()`: \indexc{force()}


~~~
f <- function(x) {
  force(x)
  10
}
f(stop("This is an error!"))
~~~
{: .r}



~~~
Error in force(x): This is an error!
~~~
{: .error}

### `...`

There is a special argument called `...` .  This argument will match any 
arguments not otherwise matched, and can be easily passed on to other functions.
`...` is often used in conjunction with S3 generic functions to allow individual 
methods to be more flexible. \indexc{...}

One relatively sophisticated user of `...` is the base `plot()` function. `plot()` 
is a generic method with arguments `x`, `y` and `...` . Most simple invocations 
of `plot()` end up calling `plot.default()` which has many more arguments, but 
also has `...` .  `...` accepts "other graphical parameters", which are listed in 
the help for `par()`.  This allows us to write code like:


~~~
plot(1:5, col = "red")
plot(1:5, cex = 5, pch = 20)
~~~
{: .r}

This illustrates both the advantages and disadvantages of `...`: it makes 
`plot()` very flexible, but to understand how to use it, we have to carefully 
read the documentation. Using `...` also comes at a price --- any misspelled 
arguments will not raise an error, and any arguments after `...` must be fully 
named.  This makes it easy for typos to go unnoticed:


~~~
sum(1, 2, NA, na.mr = TRUE)
~~~
{: .r}



~~~
[1] NA
~~~
{: .output}

It's often better to be explicit rather than implicit, so you might instead ask 
users to supply a list of additional arguments. That's certainly easier if 
you're trying to use `...` with multiple additional functions.

> ## Discussion 1
>
>
> 1.  Clarify the following list of odd function calls:
>     
>     ~~~
>     x <- sample(replace = TRUE, 20, x = c(1:10, NA))
>     y <- runif(min = 0, max = 1, 20)
>     cor(m = "k", y = y, u = "p", x = x)
>     ~~~
>     {: .r}
>
> 1.  What does this function return? Why? Which principle does it illustrate?
>
>     
>     ~~~
>     x <- sample(replace = TRUE, 20, x = c(1:10, NA))
>     y <- runif(min = 0, max = 1, 20)
>     cor(m = "k", y = y, u = "p", x = x)
>     ~~~
>     {: .r}
>
> What does this function return? Why? Which principle does it illustrate?
> 
>     
>     ~~~
>     f2 <- function(x = z) {
>     z <- 100
>     x
>     }
>     f2()
>     ~~~
>     {: .r}
{: .discussion}

## Special calls {#special-calls}

R supports two additional syntaxes for calling special types of functions: 
infix and replacement functions.

### Infix functions {#infix-functions}

Most functions in R are "prefix" operators: the name of the function comes 
before the arguments. You can also create infix functions where the function 
name comes in between its arguments, like `+` or `-`.  All user-created infix 
functions must start and end with `%`. R comes with the following infix 
functions predefined: `%%`, `%*%`, `%/%`, `%in%`, `%o%`,  `%x%`. 
\index{functions!infix} \index{infix functions} \indexc{\%\%}

> ## Callout
>
> The complete list of built-in infix operators that don't need `%` is: 
> `:, ::, :::, $, @, ^, *, /, +, -, >, >=, <, <=, ==, !=, !, &, &&, |, ||, ~, <-, <<-`
>
{: .callout}

For example, we could create a new operator that pastes together strings:


~~~
`%+%` <- function(a, b) paste0(a, b)
"new" %+% " string"
~~~
{: .r}



~~~
[1] "new string"
~~~
{: .output}

Note that you can use infix functions as prefix operators, but you have to put
its name in backticks because it's a special name:


~~~
"new" %+% " string"
~~~
{: .r}



~~~
[1] "new string"
~~~
{: .output}



~~~
`%+%`("new", " string")
~~~
{: .r}



~~~
[1] "new string"
~~~
{: .output}

This is true even for built-in infix operators: \indexc{`}


~~~
1 + 5
~~~
{: .r}



~~~
[1] 6
~~~
{: .output}



~~~
`+`(1, 5)
~~~
{: .r}



~~~
[1] 6
~~~
{: .output}

R's default precedence rules mean that infix operators are composed from left to right:


~~~
`%-%` <- function(a, b) paste0("(", a, " %-% ", b, ")")
"a" %-% "b" %-% "c"
~~~
{: .r}



~~~
[1] "((a %-% b) %-% c)"
~~~
{: .output}

Here is one infix function that can be used to provide a default value in case 
the output of another function is `NULL`:


~~~
`%||%` <- function(a, b) if (!is.null(a)) a else b
function_that_might_return_null() %||% default value
~~~
{: .r}

### Replacement functions {#replacement-functions}

Replacement functions _act like_ they modify their arguments in place, and have 
the special name `xxx<-`. They typically have two arguments (`x` and `value`), 
and they must return the modified object. For example, the following function 
allows you to modify the second element of a vector: 
\index{replacement functions} \index{functions!replacement}


~~~
`second<-` <- function(x, value) {
  x[2] <- value
  x
}
x <- 1:10
second(x) <- 5L
x
~~~
{: .r}



~~~
 [1]  1  5  3  4  5  6  7  8  9 10
~~~
{: .output}

When R evaluates the assignment `second(x) <- 5`, it notices that the left hand 
side of the `<-` is not a simple name, so it looks for a function named 
`second<-` to do the replacement. \index{assignment!replacement functions}

Note, replacement runctions actually create a modified copy. Only built-in 
functions that are implemented using `.Primitive()` will modify in place: \index{primitive functions}

If you want to supply additional arguments, they go in between `x` and `value`:


~~~
`modify<-` <- function(x, position, value) {
  x[position] <- value
  x
}
modify(x, 1) <- 10
x
~~~
{: .r}



~~~
 [1] 10  5  3  4  5  6  7  8  9 10
~~~
{: .output}

When you call `modify(x, 1) <- 10`, behind the scenes R turns it into:


~~~
x <- `modify<-`(x, 1, 10)
~~~
{: .r}

It's often useful to combine replacement and subsetting:


~~~
x <- c(a = 1, b = 2, c = 3)
names(x)
~~~
{: .r}



~~~
[1] "a" "b" "c"
~~~
{: .output}



~~~
names(x)[2] <- "two"
names(x)
~~~
{: .r}



~~~
[1] "a"   "two" "c"  
~~~
{: .output}

This works because the expression `names(x)[2] <- "two"` is evaluated as if you had written:


~~~
`*tmp*` <- names(x)
`*tmp*`[2] <- "two"
names(x) <- `*tmp*`
~~~
{: .r}

(Yes, it really does create a local variable named `*tmp*`, which is removed afterwards.)

> ## Exercises 2
>
>
> 1.  Create a list of all the replacement functions found in the base package. 
>     Which ones are primitive functions?
>
> 1.  What are valid names for user-created infix functions?
>
> 1.  Create an infix `xor()` operator.
> 
> 1.  Create infix versions of the set functions `intersect()`, `union()`, and `setdiff()`
> 
> 1.  Create a replacement function that modifies a random location in a vector.
> 
{: .discussion}

## Return values {#return-values}

The last expression evaluated in a function becomes the return value, the result of invoking the function. \index{functions!return value}


~~~
f <- function(x) {
  if (x < 10) {
    0
  } else {
    10
  }
}
f(5)
~~~
{: .r}



~~~
[1] 0
~~~
{: .output}



~~~
f(15)
~~~
{: .r}



~~~
[1] 10
~~~
{: .output}

It's good style to reserve the use of an explicit `return()` for when you are 
returning early, such as for an error, or a simple case of the function. This 
style of programming can also reduce the level of indentation, and generally 
make functions easier to understand because you can reason about them locally. 
\indexc{return()}


~~~
f <- function(x, y) {
  if (!x) return(y)

  # complicated processing here
}
~~~
{: .r}

Functions can return only a single object. But this is not a limitation because 
you can return a list containing any number of objects.

The functions that are the easiest to understand and reason about are *pure 
functions*: functions that always map the same input to the same output and have 
no other impact on the workspace. In other words, pure functions have no 
__side effects__*: they don't affect the state of the world in any way apart 
from the value they return. \index{pure functions}

R protects you from one type of side effect: most R objects have copy-on-modify 
semantics. So modifying a function argument does not change the original value: 
\index{copy-on-modify}


~~~
f <- function(x) {
  x$a <- 2
  x
}
x <- list(a = 1)
f(x)
~~~
{: .r}



~~~
$a
[1] 2
~~~
{: .output}



~~~
x$a
~~~
{: .r}



~~~
[1] 1
~~~
{: .output}

(There are two important exceptions to the copy-on-modify rule: environments 
and reference classes. These can be modified in place, so extra care is needed 
when working with them.)

This is notably different to languages like Java where you can modify the 
inputs of a function. This copy-on-modify behaviour has important performance 
consequences in R.

Most base R functions are pure, with a few notable exceptions:

* `library()` which loads a package, and hence modifies the search path.

* `setwd()`, `Sys.setenv()`, `Sys.setlocale()` which change the working 
  directory, environment variables, and the locale, respectively.

* `plot()` and friends which produce graphical output.

* `write()`, `write.csv()`, `saveRDS()`, etc. which save output to disk.

* `options()` and `par()` which modify global settings.

* S4 related functions which modify global tables of classes and methods.

* Random number generators which produce different numbers each time you 
  run them.

It's generally a good idea to minimise the use of side effects, and where 
possible, to minimise the footprint of side effects by separating pure from 
impure functions.

Functions can return `invisible` values, which are not printed out by 
default when you call the function. \indexc{invisible()} \index{functions!invisible results}


~~~
f1 <- function() 1
f2 <- function() invisible(1)

f1()
~~~
{: .r}



~~~
[1] 1
~~~
{: .output}



~~~
f2()
f1() == 1
~~~
{: .r}



~~~
[1] TRUE
~~~
{: .output}



~~~
f2() == 1
~~~
{: .r}



~~~
[1] TRUE
~~~
{: .output}

You can force an invisible value to be displayed by wrapping it in parentheses:


~~~
(f2())
~~~
{: .r}



~~~
[1] 1
~~~
{: .output}

The most common function that returns invisibly is `<-`: \index{assignment}


~~~
a <- 2
(a <- 2)
~~~
{: .r}



~~~
[1] 2
~~~
{: .output}

This is what makes it possible to assign one value to multiple variables:


~~~
a <- b <- c <- d <- 2
~~~
{: .r}

because this is parsed as:


~~~
(a <- (b <- (c <- (d <- 2))))
~~~
{: .r}



~~~
[1] 2
~~~
{: .output}

### On exit {#on-exit}

As well as returning a value, functions can set up other triggers to occur when 
the function is finished using `on.exit()`. This is often used as a way to 
guarantee that changes to the global state are restored when the function exits. 
The code in `on.exit()` is run regardless of how the function exits, whether 
with an explicit (early) return, an error, or simply reaching the end of the 
function body. \indexc{on.exit()}


~~~
in_dir <- function(dir, code) {
  old <- setwd(dir)
  on.exit(setwd(old))

  force(code)
}
getwd()
~~~
{: .r}



~~~
[1] "/Users/dlavrov/Documents/GitHub/BCB546_R-Skills/_episodes_rmd"
~~~
{: .output}



~~~
in_dir("~", getwd())
~~~
{: .r}



~~~
[1] "/Users/dlavrov"
~~~
{: .output}

The basic pattern is simple:

* We first set the directory to a new location, capturing the current location 
  from the output of `setwd()`.

* We then use `on.exit()` to ensure that the working directory is returned to 
  the previous value regardless of how the function exits.

* Finally, we explicitly force evaluation of the code. (We don't actually need 
  `force()` here, but it makes it clear to readers what we're doing.)

**Caution**: If you're using multiple `on.exit()` calls within a function, make 
sure to set `add = TRUE`. Unfortunately, the default in `on.exit()` is 
`add = FALSE`, so that every time you run it, it overwrites existing exit 
expressions.

### Exercises

1.  How does the `chdir` parameter of `source()` compare to `in_dir()`? Why 
    might you prefer one approach to the other?

1.  What function undoes the action of `library()`? How do you save and restore
    the values of `options()` and `par()`?

1.  Write a function that opens a graphics device, runs the supplied code, and 
    closes the graphics device (always, regardless of whether or not the plotting 
    code worked).

1.  We can use `on.exit()` to implement a simple version of `capture.output()`.

    
    ~~~
    capture.output2 <- function(code) {
      temp <- tempfile()
      on.exit(file.remove(temp), add = TRUE)
    
      sink(temp)
      on.exit(sink(), add = TRUE)
    
      force(code)
      readLines(temp)
    }
    capture.output2(cat("a", "b", "c", sep = "\n"))
    ~~~
    {: .r}
    
    
    
    ~~~
    [1] "a" "b" "c"
    ~~~
    {: .output}

    Compare `capture.output()` to `capture.output2()`. How do the functions 
    differ? What features have I removed to make the key ideas easier to see? 
    How have I rewritten the key ideas to be easier to understand?


## Quiz answers {#function-answers}

\enlargethispage*{\baselineskip}

1.  The three components of a function are its body, arguments, and environment.

1.  `f1(1)()` returns 11.

1.  You'd normally write it in infix style: `1 + (2 * 3)`.

1.  Rewriting the call to `mean(c(1:10, NA), na.rm = TRUE)` is easier to
    understand.
    
1.  No, it does not throw an error because the second argument is never used 
    so it's never evaluated.

1.  See [infix](#infix-functions) and 
    [replacement functions](#replacement-functions).

1.  You use `on.exit()`; see [on exit](#on-exit) for details.
[man]: http://cran.r-project.org/doc/manuals/r-release/R-lang.html#Environment-objects
[chapter]: http://adv-r.had.co.nz/Environments.html
[adv-r]: http://adv-r.had.co.nz/

