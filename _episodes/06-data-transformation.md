---
title: "Data Transformation"
teaching: 60
exercises: 30
questions:
- "How to use dplyr to transform data in R?"
- "How do I select columns"
- "How can I filter rows"
- "How can I join two data frames?"
objectives:
- "Be able to explore a dataframe"
- "Be able to add, rename, and remove columns."
- "Be able to filter rows"
- "Be able to append two data frames"
- "Be able to join two data frames"
keypoints:
- "Read in a csv file using `read_csv()`"
- "View a dataframe with `View`"
- "Use `filter()` to pick observations by their values"
- "Use `arrange()` to order the rows"
- "Use `select()` to pick variables by their names"
- "Use `mutate()` to create new variables with functions of existing variables"
- "Use `summarize()` to collapse many values down to a single summary"
---



# Data transformation {#transform}
<img src="../images/dplyr.png" align="right" hspace="20">

## Introduction

Every data analysis you conduct will likely involve manipulating dataframes at some point. Quickly extracting, 
transforming, and summarizing information from dataframes is an essential R skill. In this part we'll use Hadley 
Wickham’s `dplyr` package, which consolidates and simplifies many of the common operations we perform on dataframes. 
In addition, much of `dplyr` key functionality is written in C++ for speed.


~~~
if (!require("tidyverse")) install.packages("tidyverse")
library(tidyverse)
~~~
{: .r}


> ## Dataset
>
> Following chapter 8 of the Buffalo's book, we will use the dataset Dataset_S1.txt from the paper 
> "[The Influence of Recombination on Human Genetic Diversity](http://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.0020148)".
> The dataset contains estimates of population genetics statistics such 
> as nucleotide diversity (e.g., the columns Pi and Theta), recombination (column Recombination), and sequence 
> divergence as estimated by percent identity between human and chimpanzee genomes (column Divergence). 
> Other columns contain information about the sequencing depth (depth), and GC content 
> (percent.GC) for 1kb windows in human chromosome 20. We’ll only work with a few columns in our examples; 
> see the description of Dataset_S1.txt in the original paper for more detail.
>
> ### Downloading the dataset
>
> * You can download the Dataset_S1.txt file into a local folder of your choice using the `download.file` function.
> The `read_csv` function can then be executed to read the downloaded file:
> 
> ~~~
> download.file("https://raw.githubusercontent.com/vsbuffalo/bds-files/master/chapter-08-r/Dataset_S1.txt", destfile = "data/Dataset_S1.txt")
> dvst <- read_csv("data/Dataset_S1.txt")
> ~~~
> {: .r}
>
> * Alternatively, you can read files directly from the Internet with the `read_csv` function:
> 
> ~~~
> dvst <- read_csv("https://raw.githubusercontent.com/vsbuffalo/bds-files/master/chapter-08-r/Dataset_S1.txt")
> ~~~
> {: .r}
{: .callout}

So, let's read the file directly into a `tbl_df` or `tibble`:



Notice the parsing of columns in the file. Also notice that some of the columns names 
contain spaces and special characters. We'll replace them soon to facilitate typing.
Type `dvst` to look at the dataframe:



Because it’s common to work with dataframes with more rows and columns than fit in your screen, `tibble` 
wraps dataframes so that they don’t fill your screen when you print them 
(similar to using `head()`). Notice that the tibble only shows the first few rows and all the columns that 
fit on one screen. Run `View(dvst)`, which will open the dataset in the RStudio viewer, to see the whole dataframe. 
Also notice the row of three (or four) letter abbreviations under the column names. 
These describe the type of each variable:

| Abbreviation | Variable type            |
|:------------|:----------------------------|
| `int`       | integers                    |
| `dbl`       | doubles (real numbers)      |
| `fctr`      | factors (categorical)       |
| **Variables**| **not in the dataset**   |
| `chr`       | character vectors (strings) |
| `dttm`      | date-time                   |
| `lgl`       | logical                     |
| `date`      | dates                       |

Although the standard R operations work with tibbles (see below), we'll learn how to use dplyr functions instead


~~~
colnames(dvst)[12] <- "percent.GC" #rename X.GC columns
dvst$GC.binned <- cut(dvst$percent.GC, 5) # create a new column from existing data
~~~
{: .r}

---

## `dplyr` basics

There are **five key dplyr functions** that allow you to solve the vast majority of data manipulation challenges:

| Function     | Action                                               |
|:-------------|:-----------------------------------------------------|
| `filter()`   | Pick observations by their values                    |
| `arrange()`  | Reorder the rows                                     |
| `select()`   | Pick variables by their names                        |
| `mutate()`   | Create new variables with functions of existing variables |
| `summarise()`| Collapse many values down to a single summary        |

None of these functions perform tasks you can’t accomplish with R’s base functions. 
But dplyr’s advantage is in the added consistency, speed, and versatility of its data manipulation interface. 

dplyr functions can be used in conjunction with `group_by()`, which changes the scope of each function from operating 
on the entire dataset to operating on it group-by-group. These six functions provide the verbs for a language of 
data manipulation.

All verbs work similarly: 

1.  The first argument is a data frame (actually, a `tibble`).

1.  The subsequent arguments describe what to do with the data frame,
    using the variable names (without quotes).
    
1.  The result is a new data frame.

Together these properties make it easy to chain together multiple simple steps to achieve a complex result. 
Let's dive in and see how these verbs work.

===

## Use `filter()` to filter rows

Let's say we used `summary` to get an idea about the total number of SNPs in different windows:


~~~
summary(dvst$`total SNPs`)
~~~
{: .r}



~~~
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
  0.000   3.000   7.000   8.906  12.000  93.000 
~~~
{: .output}



~~~
# or with dplyr
summary(select(dvst,`total SNPs`)) #this does not look simpler, but wait...
~~~
{: .r}



~~~
   total SNPs    
 Min.   : 0.000  
 1st Qu.: 3.000  
 Median : 7.000  
 Mean   : 8.906  
 3rd Qu.:12.000  
 Max.   :93.000  
~~~
{: .output}

We see that this number varies considerably across all windows on chromosome 20 and the data are right-skewed: 
the third quartile is 12 SNPs, but the maximum is 93 SNPs. Often we want to investigate such outliers more 
closely. We can use `filter()` to extract the rows of our dataframe based on their values. The first argument 
is the name of the data frame. The second and subsequent arguments are the expressions that filter the data frame.


~~~
filter(dvst,`total SNPs` >= 85)
~~~
{: .r}



~~~
# A tibble: 3 x 17
     start      end `total SNPs` `total Bases` depth `unique SNPs` dhSNPs
     <int>    <int>        <int>         <int> <dbl>         <int>  <int>
1  2621001  2622000           93         11337  11.3            13     10
2 13023001 13024000           88         11784  11.8            11      1
3 47356001 47357000           87         12505  12.5             9      7
# ... with 10 more variables: `reference Bases` <int>, Theta <dbl>,
#   Pi <dbl>, Heterozygosity <dbl>, percent.GC <dbl>, Recombination <dbl>,
#   Divergence <dbl>, Constraint <int>, SNPs <int>, GC.binned <fct>
~~~
{: .output}

We can build more elaborate queries by using multiple filters. For example, suppose we wanted to see all windows 
where Pi (nucleotide diversity) is greater than 16 and percent GC is greater than 80. We’d use:


~~~
filter(dvst, Pi > 16, percent.GC > 80)
~~~
{: .r}



~~~
# A tibble: 3 x 17
     start      end `total SNPs` `total Bases` depth `unique SNPs` dhSNPs
     <int>    <int>        <int>         <int> <dbl>         <int>  <int>
1 63097001 63098000            5           947  2.39             2      1
2 63188001 63189000            2          1623  3.21             2      0
3 63189001 63190000            5          1395  1.89             3      2
# ... with 10 more variables: `reference Bases` <int>, Theta <dbl>,
#   Pi <dbl>, Heterozygosity <dbl>, percent.GC <dbl>, Recombination <dbl>,
#   Divergence <dbl>, Constraint <int>, SNPs <int>, GC.binned <fct>
~~~
{: .output}

When you run that line of code, dplyr executes the filtering operation and returns a new data frame. 
Note, **dplyr functions never modify their inputs**, so if you want to save the result, you'll need to use the 
assignment operator, `<-`:


~~~
new_df <- filter(dvst, Pi > 16, percent.GC > 80)
~~~
{: .r}

> ## Miscellaneous Tips
>
> R either prints out the results, or saves them to a variable. If you want to do both, you can wrap the 
assignment in parentheses:
> 
> 
> ~~~
> (new_df <- filter(dvst, Pi > 16, percent.GC > 80))
> ~~~
> {: .r}
> 
> 
> 
> ~~~
> # A tibble: 3 x 17
>      start      end `total SNPs` `total Bases` depth `unique SNPs` dhSNPs
>      <int>    <int>        <int>         <int> <dbl>         <int>  <int>
> 1 63097001 63098000            5           947  2.39             2      1
> 2 63188001 63189000            2          1623  3.21             2      0
> 3 63189001 63190000            5          1395  1.89             3      2
> # ... with 10 more variables: `reference Bases` <int>, Theta <dbl>,
> #   Pi <dbl>, Heterozygosity <dbl>, percent.GC <dbl>, Recombination <dbl>,
> #   Divergence <dbl>, Constraint <int>, SNPs <int>, GC.binned <fct>
> ~~~
> {: .output}
{: .callout}

### Comparisons

To use filtering effectively, you have to know how to select the observations that you want using the 
comparison operators. R provides the standard suite: `>`, `>=`, `<`, `<=`, `!=` (not equal), and `==` (equal). 
Note, that because `==` does not work well with real numbers, use dplyr `near` function instead!

<!-- > ## Common pitfalls -->
<!-- > -->
<!-- > When you're starting out with R, the easiest mistake to make is to use `=` instead of `==` when testing for equality. When this > happens you'll get an informative error: -->
<!-- >  -->
<!-- > ```{r, error = TRUE} -->
<!-- > filter(dvst, `total SNPs` = 0) -->
<!-- > ``` -->
<!-- > -->
<!-- > There's another common problem you might encounter when using `==`: floating point numbers.  -->
<!-- > These results might surprise you! -->
<!-- >  -->
<!-- > ```{r} -->
<!-- > sqrt(2) ^ 2 == 2 -->
<!-- > 1/49 * 49 == 1 -->
<!-- > ``` -->
<!-- >  -->
<!-- > Computers use finite precision arithmetic (they obviously can't store an infinite number of digits!)  -->
<!-- > so remember that every number you see is an approximation. Instead of relying on `==`, use `near()` from the dplyr package: -->
<!-- >  -->
<!-- > ```{r} -->
<!-- > near(sqrt(2) ^ 2,  2) -->
<!-- > near(1 / 49 * 49, 1) -->
<!-- > ``` -->
<!-- > -->
<!-- {: .callout} -->

### Logical operators

Multiple arguments to `filter()` are combined with "and": every expression must be true in order for a 
row to be included in the output. For other types of combinations, you'll need to use Boolean operators 
yourself: `&` is "and", `|` is "or", and `!` is "not". The figure below shows the complete set 
of Boolean operations.

<img src="../images/transform-logical.png" title="Complete set of boolean operations. `x` is the left-hand circle, `y` is the right-hand circle, and the shaded region show which parts each operator selects." alt="Complete set of boolean operations. `x` is the left-hand circle, `y` is the right-hand circle, and the shaded region show which parts each operator selects." style="display: block; margin: auto;" />

If you are looking for multiple values within a certain column, use the `x %in% y` shortcut. This will 
select every row where `x` is one of the values in `y`:


~~~
filter(dvst, total.SNPs %in% c(0,1,2))
~~~
{: .r}

Finally, whenever you start using complicated, multipart expressions in `filter()`, consider making them explicit 
variables instead. That makes it much easier to check your work. We'll learn how to create new variables shortly.

>  ## Missing values
>  
>  One important feature of R that can make comparison tricky are missing values, or `NA`s ("not availables"). 
>  `NA` represents an unknown value so missing values are "contagious": almost any operation involving an unknown 
>  value will also be unknown.
>  
>  
>  ~~~
>  NA > 5
>  ~~~
>  {: .r}
>  
>  
>  
>  ~~~
>  [1] NA
>  ~~~
>  {: .output}
>  
>  
>  
>  ~~~
>  10 == NA
>  ~~~
>  {: .r}
>  
>  
>  
>  ~~~
>  [1] NA
>  ~~~
>  {: .output}
>  
>  
>  
>  ~~~
>  NA + 10
>  ~~~
>  {: .r}
>  
>  
>  
>  ~~~
>  [1] NA
>  ~~~
>  {: .output}
>  
>  
>  
>  ~~~
>  NA / 2
>  ~~~
>  {: .r}
>  
>  
>  
>  ~~~
>  [1] NA
>  ~~~
>  {: .output}
>  
>  
>  
>  ~~~
>  NA == NA
>  ~~~
>  {: .r}
>  
>  
>  
>  ~~~
>  [1] NA
>  ~~~
>  {: .output}
{: .callout}

`filter()` only includes rows where the condition is `TRUE`; it excludes both `FALSE` and `NA` values. If you want 
to preserve missing values, ask for them explicitly:


~~~
filter(dvst, is.na(Divergence))
~~~
{: .r}

> ## Your turn!
>
> 1.  Find all windows ...
> - with the lowest/highest GC content
> - that have 0 total SNPs
> - that are incomplete (<1000 bp)
> - that have the highest divergence with the chimp
> - that have less than 5% GC difference from the mean
>
> 2.  One useful dplyr filtering helper is `between()`. 
> - Can you figure out what does it do?
> - Use it to simplify the code needed to answer the previous challenges.
>
{: .discussion}

===

## Use `mutate() to add new variables`

It's often useful to add new columns that are functions of existing columns (notice, how we used `dvst$GC.binned <- cut(dvst$percent.GC, 5)` above). That's the job of `mutate()`. 

`mutate()` always adds new columns at the end of your dataset. Remember that when you're in RStudio, the easiest 
way to see all the columns is with `View()`.

Here we'll add an additional column that indicates whether a window is in the centromere region (nucleotides 25,800,000 to 29,700,000, based on Giemsa banding; see [README](https://github.com/vsbuffalo/bds-files/blob/master/chapter-08-r/README.md) for details). 

We'll call it `cent` and make it logical (with TRUE/FALSE values):


~~~
mutate(dvst, cent = start >= 25800000 & end <= 29700000)
~~~
{: .r}



~~~
# A tibble: 59,140 x 18
   start   end `total SNPs` `total Bases` depth `unique SNPs` dhSNPs
   <int> <int>        <int>         <int> <dbl>         <int>  <int>
 1 55001 56000            0          1894  3.41             0      0
 2 56001 57000            5          6683  6.68             2      2
 3 57001 58000            1          9063  9.06             1      0
 4 58001 59000            7         10256 10.3              3      2
 5 59001 60000            4          8057  8.06             4      0
 6 60001 61000            6          7051  7.05             2      1
 7 61001 62000            7          6950  6.95             2      1
 8 62001 63000            1          8834  8.83             1      0
 9 63001 64000            1          9629  9.63             1      0
10 64001 65000            3          7999  8.00             1      1
# ... with 59,130 more rows, and 11 more variables: `reference
#   Bases` <int>, Theta <dbl>, Pi <dbl>, Heterozygosity <dbl>,
#   percent.GC <dbl>, Recombination <dbl>, Divergence <dbl>,
#   Constraint <int>, SNPs <int>, GC.binned <fct>, cent <lgl>
~~~
{: .output}



~~~
dvst
~~~
{: .r}



~~~
# A tibble: 59,140 x 17
   start   end `total SNPs` `total Bases` depth `unique SNPs` dhSNPs
   <int> <int>        <int>         <int> <dbl>         <int>  <int>
 1 55001 56000            0          1894  3.41             0      0
 2 56001 57000            5          6683  6.68             2      2
 3 57001 58000            1          9063  9.06             1      0
 4 58001 59000            7         10256 10.3              3      2
 5 59001 60000            4          8057  8.06             4      0
 6 60001 61000            6          7051  7.05             2      1
 7 61001 62000            7          6950  6.95             2      1
 8 62001 63000            1          8834  8.83             1      0
 9 63001 64000            1          9629  9.63             1      0
10 64001 65000            3          7999  8.00             1      1
# ... with 59,130 more rows, and 10 more variables: `reference
#   Bases` <int>, Theta <dbl>, Pi <dbl>, Heterozygosity <dbl>,
#   percent.GC <dbl>, Recombination <dbl>, Divergence <dbl>,
#   Constraint <int>, SNPs <int>, GC.binned <fct>
~~~
{: .output}

> ## Challenge 1
>
> * Why don't we see the column in the dataset? 
>
> > ## Solution
> > As discussed above, dplyr functions never modify their inputs
> {: .solution}
>
> * What standard R command can we use to create a new column in an existing dataframe?
> > ## Solution
> > 
> > ~~~
> > d_df$cent <- d_df$start >= 25800000 & d_df$end <= 29700000
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> > Error in eval(expr, envir, enclos): object 'd_df' not found
> > ~~~
> > {: .error}
> {: .solution}
>
> * How many windows fall into this centromeric region? 
>
> > ## Solution
> >
> > Use `filter()` 
> > 
> > ~~~
> > filter(d_df,cent==TRUE)
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> > Error in filter(d_df, cent == TRUE): object 'd_df' not found
> > ~~~
> > {: .error}
> > or sum()
> > 
> > 
> > ~~~
> > sum(d_df$cent)
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> > Error in eval(expr, envir, enclos): object 'd_df' not found
> > ~~~
> > {: .error}
> {: .solution}
{: .challenge}

In the dataset we are using, the diversity estimate Pi is measured per sampling window (1kb) and scaled 
up by 10x (see supplementary Text S1 for more details). It would be useful to have this scaled as per 
basepair nucleotide diversity (so as to make the scale more intuitive).


~~~
mutate(dvst, diversity = Pi / (10*1000)) # rescale, removing 10x and making per bp
~~~
{: .r}



~~~
# A tibble: 59,140 x 18
   start   end `total SNPs` `total Bases` depth `unique SNPs` dhSNPs
   <int> <int>        <int>         <int> <dbl>         <int>  <int>
 1 55001 56000            0          1894  3.41             0      0
 2 56001 57000            5          6683  6.68             2      2
 3 57001 58000            1          9063  9.06             1      0
 4 58001 59000            7         10256 10.3              3      2
 5 59001 60000            4          8057  8.06             4      0
 6 60001 61000            6          7051  7.05             2      1
 7 61001 62000            7          6950  6.95             2      1
 8 62001 63000            1          8834  8.83             1      0
 9 63001 64000            1          9629  9.63             1      0
10 64001 65000            3          7999  8.00             1      1
# ... with 59,130 more rows, and 11 more variables: `reference
#   Bases` <int>, Theta <dbl>, Pi <dbl>, Heterozygosity <dbl>,
#   percent.GC <dbl>, Recombination <dbl>, Divergence <dbl>,
#   Constraint <int>, SNPs <int>, GC.binned <fct>, diversity <dbl>
~~~
{: .output}

Notice, that we can combine the two operations in a single command (and also save the result):


~~~
dvst <- mutate(dvst, 
       diversity = Pi / (10*1000), 
       cent = start >= 25800000 & end <= 29700000
)
~~~
{: .r}

If you only want to keep the new variables, use `transmute()`:


~~~
transmute(dvst,
  diversity = Pi / (10*1000), 
  cent = start >= 25800000 & end <= 29700000
)
~~~
{: .r}



~~~
# A tibble: 59,140 x 2
   diversity cent 
       <dbl> <lgl>
 1  0        F    
 2  0.00104  F    
 3  0.000199 F    
 4  0.000956 F    
 5  0.000851 F    
 6  0.000912 F    
 7  0.000806 F    
 8  0.000206 F    
 9  0.000188 F    
10  0.000541 F    
# ... with 59,130 more rows
~~~
{: .output}

> ## Useful creation functions
> 
> There are many functions for creating new variables that you can use with `mutate()`. The key property is that 
> the function must be vectorised: it must take a vector of values as input, return a vector with the same number 
> of values as output. There's no way to list every possible function that you might use, but here's a selection 
> of functions that are frequently useful:
> 
> *   Arithmetic operators: `+`, `-`, `*`, `/`, `^`. These are all vectorised,
>     using the so called "recycling rules".
>     
> *   Modular arithmetic: `%/%` (integer division) and `%%` (remainder), where
>     `x == y * (x %/% y) + (x %% y)`. Modular arithmetic is a handy tool because 
>     it allows you to break integers up into pieces.
>   
> *   Logs: `log()`, `log2()`, `log10()`. Logarithms are an incredibly useful
>     transformation for dealing with data that ranges across multiple orders of
>     magnitude. They also convert multiplicative relationships to additive.
> 
> *   Offsets: `lead()` and `lag()` allow you to refer to leading or lagging 
>     values. This allows you to compute running differences (e.g. `x - lag(x)`) 
>     or find when values change (`x != lag(x))`. They are most useful in 
>     conjunction with `group_by()`, which you'll learn about shortly.
>   
> *   Cumulative and rolling aggregates: R provides functions for running sums,
>     products, mins and maxes: `cumsum()`, `cumprod()`, `cummin()`, `cummax()`; 
>     and dplyr provides `cummean()` for cumulative means.
> 
> *   Logical comparisons, `<`, `<=`, `>`, `>=`, `!=`, which you learned about
>     earlier. If you're doing a complex sequence of logical operations it's 
>     often a good idea to store the interim values in new variables so you can
>     check that each step is working as expected.
> 
> *   Ranking: `min_rank()` does the most usual type of ranking 
>     (e.g. 1st, 2nd, 2nd, 4th). The default gives smallest values the small
>     ranks; use `desc(x)` to give the largest values the smallest ranks. 
>     If `min_rank()` doesn't do what you need, look at the variants
>     `row_number()`, `dense_rank()`, `percent_rank()`, `cume_dist()`,
>     `ntile()`.  See their help pages for more details.
> 
{: .callout}

===

## Use `arrange()` to arrange rows

`arrange()` works similarly to `filter()` except that instead of selecting rows, it changes their order. 
It takes a data frame and a set of column names (or more complicated expressions) to order by. If you 
provide more than one column name, each additional column will be used to break ties in the values of 
preceding columns:


~~~
arrange(dvst, cent, percent.GC)
~~~
{: .r}



~~~
# A tibble: 59,140 x 19
      start      end `total SNPs` `total Bases` depth `unique SNPs` dhSNPs
      <int>    <int>        <int>         <int> <dbl>         <int>  <int>
 1 43448001 43449000            5           690  2.66             5      0
 2 14930001 14931000           19          3620  4.57             7      1
 3 37777001 37778000            3           615  1.12             3      0
 4 54885001 54886000            1          2298  3.69             1      0
 5 54888001 54889000           15          3823  3.87            11      1
 6 41823001 41824000           15          6950  6.95            10      4
 7 50530001 50531000           10          7293  7.29             2      2
 8 39612001 39613000           13          8254  8.25             5      3
 9 55012001 55013000            7          7028  7.03             7      0
10 22058001 22059000            4           281  1.20             4      0
# ... with 59,130 more rows, and 12 more variables: `reference
#   Bases` <int>, Theta <dbl>, Pi <dbl>, Heterozygosity <dbl>,
#   percent.GC <dbl>, Recombination <dbl>, Divergence <dbl>,
#   Constraint <int>, SNPs <int>, GC.binned <fct>, diversity <dbl>,
#   cent <lgl>
~~~
{: .output}

Use `desc()` to re-order by a column in descending order:


~~~
arrange(dvst, desc(cent), percent.GC)
~~~
{: .r}



~~~
# A tibble: 59,140 x 19
      start      end `total SNPs` `total Bases` depth `unique SNPs` dhSNPs
      <int>    <int>        <int>         <int> <dbl>         <int>  <int>
 1 25869001 25870000           10          7292  7.29             6      2
 2 29318001 29319000            2          6192  6.19             2      0
 3 29372001 29373000            5          7422  7.42             3      2
 4 26174001 26175000            1          7816  7.82             1      0
 5 26182001 26183000            2          6630  6.63             1      1
 6 25924001 25925000           30          7127  7.13            15      8
 7 26155001 26156000            0          9449  9.45             0      0
 8 25926001 25927000           21          8197  8.20            13      6
 9 29519001 29520000           15          8135  8.14            13      2
10 25901001 25902000            9          9126  9.13             5      2
# ... with 59,130 more rows, and 12 more variables: `reference
#   Bases` <int>, Theta <dbl>, Pi <dbl>, Heterozygosity <dbl>,
#   percent.GC <dbl>, Recombination <dbl>, Divergence <dbl>,
#   Constraint <int>, SNPs <int>, GC.binned <fct>, diversity <dbl>,
#   cent <lgl>
~~~
{: .output}

Note, that missing values are always sorted at the end!

### Exercises

1.  How could you use `arrange()` to sort all missing values to the start?
    (Hint: use `is.na()`).

===    

## Use `select()` to select columns and `rename()` to rename them

It's not uncommon to get datasets with hundreds or even thousands of variables. In this case, the first 
challenge is often narrowing in on the variables you're actually interested in. `select()` allows you to 
rapidly zoom in on a useful subset using operations based on the names of the variables.

`select()` is not terribly useful with our data because they only have a few variables, but you can still 
get the general idea:


~~~
# Select columns by name
select(dvst, start, end, Divergence)
~~~
{: .r}



~~~
# A tibble: 59,140 x 3
   start   end Divergence
   <int> <int>      <dbl>
 1 55001 56000    0.00301
 2 56001 57000    0.0180 
 3 57001 58000    0.00701
 4 58001 59000    0.0120 
 5 59001 60000    0.0240 
 6 60001 61000    0.0160 
 7 61001 62000    0.0120 
 8 62001 63000    0.0150 
 9 63001 64000    0.00901
10 64001 65000    0.00701
# ... with 59,130 more rows
~~~
{: .output}



~~~
# Select all columns between depth and Pi (inclusive)
select(dvst, depth:Pi)
~~~
{: .r}



~~~
# A tibble: 59,140 x 6
   depth `unique SNPs` dhSNPs `reference Bases` Theta    Pi
   <dbl>         <int>  <int>             <int> <dbl> <dbl>
 1  3.41             0      0               556  0     0   
 2  6.68             2      2              1000  8.01 10.4 
 3  9.06             1      0              1000  3.51  1.99
 4 10.3              3      2              1000  9.93  9.56
 5  8.06             4      0              1000 12.9   8.51
 6  7.05             2      1              1000  7.82  9.12
 7  6.95             2      1              1000  8.60  8.06
 8  8.83             1      0              1000  4.01  2.06
 9  9.63             1      0              1000  3.37  1.88
10  8.00             1      1              1000  4.16  5.41
# ... with 59,130 more rows
~~~
{: .output}



~~~
# Select all columns except those from start to percent.GC (inclusive)
select(dvst, -(start:percent.GC))
~~~
{: .r}



~~~
# A tibble: 59,140 x 7
   Recombination Divergence Constraint  SNPs GC.binned   diversity cent 
           <dbl>      <dbl>      <int> <int> <fct>           <dbl> <lgl>
 1       0.00960    0.00301          0     0 (51.6,68.5]  0        F    
 2       0.00960    0.0180           0     0 (34.7,51.6]  0.00104  F    
 3       0.00960    0.00701          0     0 (34.7,51.6]  0.000199 F    
 4       0.00960    0.0120           0     0 (34.7,51.6]  0.000956 F    
 5       0.00960    0.0240           0     0 (34.7,51.6]  0.000851 F    
 6       0.00960    0.0160           0     0 (34.7,51.6]  0.000912 F    
 7       0.00960    0.0120           0     0 (34.7,51.6]  0.000806 F    
 8       0.00960    0.0150           0     0 (34.7,51.6]  0.000206 F    
 9       0.00960    0.00901         58     1 (34.7,51.6]  0.000188 F    
10       0.00958    0.00701          0     1 (17.7,34.7]  0.000541 F    
# ... with 59,130 more rows
~~~
{: .output}

There are a number of helper functions you can use within `select()`:

* `starts_with("abc")`: matches names that begin with "abc".

* `ends_with("xyz")`: matches names that end with "xyz".

* `contains("ijk")`: matches names that contain "ijk".

* `matches("(.)\\1")`: selects variables that match a regular expression.
   This one matches any variables that contain repeated characters. You'll 
   learn more about regular expressions in [strings].
   
*  `num_range("x", 1:3)` matches `x1`, `x2` and `x3`.
   
See `?select` for more details.

`select()` can also be used to rename variables, but it's rarely useful because it drops all of the variables 
not explicitly mentioned. Instead, use `rename()`, which is a variant of `select()` that keeps all the 
variables that aren't explicitly mentioned:


~~~
dvst <- rename(dvst, total.SNPs = `total SNPs`,
       total.Bases = `total Bases`,
       unique.SNPs = `unique SNPs`,
       reference.Bases = `reference Bases`) #renaming all the columns with spaces!
colnames(dvst)
~~~
{: .r}



~~~
 [1] "start"           "end"             "total.SNPs"     
 [4] "total.Bases"     "depth"           "unique.SNPs"    
 [7] "dhSNPs"          "reference.Bases" "Theta"          
[10] "Pi"              "Heterozygosity"  "percent.GC"     
[13] "Recombination"   "Divergence"      "Constraint"     
[16] "SNPs"            "GC.binned"       "diversity"      
[19] "cent"           
~~~
{: .output}

Another option is to use `select()` in conjunction with the `everything()` helper. This is useful if you 
have a handful of variables you'd like to move to the start of the data frame.


~~~
select(dvst, cent, everything())
~~~
{: .r}



~~~
# A tibble: 59,140 x 19
   cent  start   end total.SNPs total.Bases depth unique.SNPs dhSNPs
   <lgl> <int> <int>      <int>       <int> <dbl>       <int>  <int>
 1 F     55001 56000          0        1894  3.41           0      0
 2 F     56001 57000          5        6683  6.68           2      2
 3 F     57001 58000          1        9063  9.06           1      0
 4 F     58001 59000          7       10256 10.3            3      2
 5 F     59001 60000          4        8057  8.06           4      0
 6 F     60001 61000          6        7051  7.05           2      1
 7 F     61001 62000          7        6950  6.95           2      1
 8 F     62001 63000          1        8834  8.83           1      0
 9 F     63001 64000          1        9629  9.63           1      0
10 F     64001 65000          3        7999  8.00           1      1
# ... with 59,130 more rows, and 11 more variables: reference.Bases <int>,
#   Theta <dbl>, Pi <dbl>, Heterozygosity <dbl>, percent.GC <dbl>,
#   Recombination <dbl>, Divergence <dbl>, Constraint <int>, SNPs <int>,
#   GC.binned <fct>, diversity <dbl>
~~~
{: .output}

===

## Use `summarise()` to make summaries

The last key verb is `summarise()`. It collapses a data frame to a single row:


~~~
summarise(dvst, GC = mean(percent.GC, na.rm = TRUE), averageSNPs=mean(total.SNPs, na.rm = TRUE), allSNPs=sum(total.SNPs))
~~~
{: .r}



~~~
# A tibble: 1 x 3
     GC averageSNPs allSNPs
  <dbl>       <dbl>   <int>
1  44.1        8.91  526693
~~~
{: .output}

`summarise()` is not terribly useful unless we pair it with `group_by()`. This changes the unit of analysis from the complete dataset to individual groups. Then, when you use the dplyr verbs on a grouped data frame they'll be automatically applied "by group". For example, if we applied exactly the same code to a data frame grouped by position related to centromere:


~~~
by_cent <- group_by(dvst, cent)
summarise(by_cent, GC = mean(percent.GC, na.rm = TRUE), averageSNPs=mean(total.SNPs, na.rm = TRUE), allSNPs=sum(total.SNPs))
~~~
{: .r}



~~~
# A tibble: 2 x 4
  cent     GC averageSNPs allSNPs
  <lgl> <dbl>       <dbl>   <int>
1 F      44.2        8.87  518743
2 T      39.7       11.6     7950
~~~
{: .output}

Subsetting columns can be a useful way to summarize data across two different conditions. 
For example, we might be curious if the average depth in a window (the depth column) 
differs between very high GC content windows (greater than 80%) and all other windows:


~~~
by_GC <- group_by(dvst, percent.GC >= 80)
summarize(by_GC, depth=mean(depth))
~~~
{: .r}



~~~
# A tibble: 2 x 2
  `percent.GC >= 80` depth
  <lgl>              <dbl>
1 F                   8.18
2 T                   2.24
~~~
{: .output}

This is a fairly large difference, but it’s important to consider how many windows this includes. 
Whenever you do any aggregation, it's always a good idea to include either a count (`n()`), 
or a count of non-missing values (`sum(!is.na(x))`). That way you can check that you're not 
drawing conclusions based on very small amounts of data:


~~~
summarize(by_GC, mean_depth=mean(depth), n_rows=n())
~~~
{: .r}



~~~
# A tibble: 2 x 3
  `percent.GC >= 80` mean_depth n_rows
  <lgl>                   <dbl>  <int>
1 F                        8.18  59131
2 T                        2.24      9
~~~
{: .output}


> ## Challenge 6
>
> As another example, consider looking at Pi by windows that fall in the centromere and those that do not.
> Does the centromer have higher nucleotide diversity than other regions in these data?
>
> > ## Solutions to challenge 6
> >
> > Because cent is a logical vector, we can group by it directly:
> > 
> > 
> > ~~~
> > by_cent <- group_by(dvst, cent)
> > summarize(by_cent, nt_diversity=mean(diversity), min=which.min(diversity), n_rows=n())
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> > # A tibble: 2 x 4
> >   cent  nt_diversity   min n_rows
> >   <lgl>        <dbl> <int>  <int>
> > 1 F          0.00123     1  58455
> > 2 T          0.00204     1    685
> > ~~~
> > {: .output}
> > Indeed, the centromere does appear to have higher nucleotide diversity than other regions in this data. 
> >
> {: .solution}
{: .challenge}

> ## Useful summary functions
> 
> Just using means, counts, and sum can get you a long way, but R provides many other useful summary functions:
> 
> *   Measures of location: we've used `mean(x)`, but `median(x)` is also useful. 
> *   Measures of spread: `sd(x)`, `IQR(x)`, `mad(x)`. The mean squared deviation,
>     or standard deviation or sd for short, is the standard measure of spread.
>     The interquartile range `IQR()` and median absolute deviation `mad(x)`
>     are robust equivalents that may be more useful if you have outliers.
> *   Measures of rank: `min(x)`, `quantile(x, 0.25)`, `max(x)`. Quantiles
>     are a generalisation of the median. For example, `quantile(x, 0.25)`
>     will find a value of `x` that is greater than 25% of the values,
>     and less than the remaining 75%.
> *   Measures of position: `first(x)`, `nth(x, 2)`, `last(x)`. These work 
>     similarly to `x[1]`, `x[2]`, and `x[length(x)]` but let you set a default 
>     value if that position does not exist (i.e. you're trying to get the 3rd
>     element from a group that only has two elements).
> *   Counts: You've seen `n()`, which takes no arguments, and returns the 
>     size of the current group. To count the number of non-missing values, use
>     `sum(!is.na(x))`. To count the number of distinct (unique) values, use
>     `n_distinct(x)`.
> *   Counts and proportions of logical values: `sum(x > 10)`, `mean(y == 0)`.
>     When used with numeric functions, `TRUE` is converted to 1 and `FALSE` to 0. 
>     This makes `sum()` and `mean()` very useful: `sum(x)` gives the number of 
>    `TRUE`s in `x`, and `mean(x)` gives the proportion.
>
{: callout}

Together `group_by()` and `summarise()` provide one of the tools that you'll use most commonly when working with dplyr: grouped summaries. But before we go any further with this, we need to introduce a powerful new idea: the pipe.

### Combining multiple operations with the pipe

It looks like we have been creating and naming several intermediate files in our anlysis, even though we didn't care about them. Naming things is hard, so this slows down our analysis. 

There's another way to tackle the same problem with the pipe, `%>%`:


~~~
dvst %>%
  rename(GC.percent = percent.GC) %>%
  group_by(GC.percent >= 80) %>%
  summarize(mean_depth=mean(depth, na.rm = TRUE), n_rows=n())
~~~
{: .r}



~~~
# A tibble: 2 x 3
  `GC.percent >= 80` mean_depth n_rows
  <lgl>                   <dbl>  <int>
1 F                        8.18  59131
2 T                        2.24      9
~~~
{: .output}

This focuses on the transformations, not what's being transformed. 
You can read it as a series of imperative statements: group, then summarise, then filter, 
where `%>%` stands for "then".

Behind the scenes, `x %>% f(y)` turns into `f(x, y)`, and `x %>% f(y) %>% g(z)` turns into 
`g(f(x, y), z)` and so on.

Working with the pipe is one of the key criteria for belonging to the tidyverse. The only exception is 
ggplot2: it was written before the pipe was discovered. However, the next iteration of ggplot2, 
ggvis, will use the pipe. 

> ## RStudio Tip
> A shortcut for %>% is available in the newest RStudio releases under the keybinding 
> CTRL + SHIFT + M (or CMD + SHIFT + M for OSX).  
> You can also review the set of available keybindings when within RStudio with ALT + SHIFT + K.
> ```
{: .callout}


> ## Missing values
>
> You may have wondered about the `na.rm` argument we used above. What happens if we don't set it? 
> We may get a lot of missing values! That's because aggregation functions obey the usual rule of 
> missing values: if there's any missing value in the input, the output will be a missing value. Fortunately, 
> all aggregation functions have an `na.rm` argument which removes the missing values prior to computation:
> We could have also removed all the rows that have uknown position value prior to the analysis with the 
> `filter(!is.na())` command
>
{: .callout}

Finally, one of the best features of dplyr is 
that all of these same methods also work with database connections. For example, you can manipulate a SQLite 
database with all of the same verbs we’ve used here.

