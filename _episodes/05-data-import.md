---
title: "Data Import"
teaching: 20
exercises: 10
questions:
- "How to read data in R?"
- "What are some potential problems with reading data in R?"
- "How to write data in files?"
objectives:
- "To understand how to read data into R"
- "To learn most commonly used functions in `readr` for reading the files"
- "To understand and be able to fix common problems in reading data files"
- "To know how to write data into a file"
keypoints:
- "Use `read_cvs` to read in CSV files"
- "Use `read_tvs` to read in TSV files"
- "Use `write_csv()` and `write_tsv()` to write such files"
- "Supply `col_types` to read functions in your script to insure consistancy"
- "Use `guess_encoding()` to guess encoding of strings in old docs"
---



# Data import
<img src="../img/readr.png" align="right" hspace="10">

## Introduction

Before transforming data we have to learn how to read them into R. We'll start by learning how to read 
plain-text rectangular files into R. We'll finish with a few pointers to packages that are useful for 
other types of data.

The tutorial is based on Chapter 10 of the [R for Data Science](R for Data Science) book by Garrett Grolemund 
and Hadley Wickham.  Check this book for more details.

### Prerequisites


We'll be using package __readr__ to load flat files into R. This packages is a part of the core tidyverse, 
an "an opinionated collection of R packages designed for data science".


~~~
if (!require("tidyverse")) install.packages("tidyverse")
~~~
{: .r}



~~~
Loading required package: tidyverse
~~~
{: .output}



~~~
── Attaching packages ────────────────────────────────── tidyverse 1.2.1 ──
~~~
{: .output}



~~~
✔ ggplot2 2.2.1     ✔ readr   1.1.1
✔ tibble  1.3.4     ✔ purrr   0.2.4
✔ tidyr   0.7.2     ✔ dplyr   0.7.4
✔ ggplot2 2.2.1     ✔ forcats 0.2.0
~~~
{: .output}



~~~
── Conflicts ───────────────────────────────────── tidyverse_conflicts() ──
✖ dplyr::filter() masks stats::filter()
✖ dplyr::lag()    masks stats::lag()
~~~
{: .output}



~~~
library(tidyverse)
~~~
{: .r}

## Getting started

The four most commonly used functions in `readr` are `read_csv`, `read_csv2`, `read_tsv`, and `read_delim`.

* `read_csv()` reads comma delimited files,
* `read_csv2()` reads semicolon separated files (common in countries where `,` is used as the decimal place),
* `read_tsv()` reads tab delimited files, and 
* `read_delim()` reads in files
  with any delimiter.

These functions all have similar syntax; we will use `read_csv` as an example. 

The first argument to `read_csv()` is the to the file to read.


~~~
heights <- read_csv("../data/heights.csv")
~~~
{: .r}



~~~
Parsed with column specification:
cols(
  earn = col_double(),
  height = col_double(),
  sex = col_character(),
  ed = col_integer(),
  age = col_integer(),
  race = col_character()
)
~~~
{: .output}

When you run `read_csv()` it prints out a column specification that gives the name and type of each column.

You can also supply an inline csv file. This is useful for experimenting with readr and for creating reproducible examples to share with others:


~~~
read_csv("a,b,c
1,2,3
4,5,6")
~~~
{: .r}



~~~
# A tibble: 2 x 3
      a     b     c
  <int> <int> <int>
1     1     2     3
2     4     5     6
~~~
{: .output}

In both cases `read_csv()` uses the first line of the data for the column names, which is a very common convention. There are two cases where you might want to tweak this behaviour:

1.  Sometimes there are a few lines of metadata at the top of the file. You can
    use `skip = n` to skip the first `n` lines; or use `comment = "#"` to drop
    all lines that start with (e.g.) `#`.
    
    
    ~~~
    read_csv("The first line of metadata
      The second line of metadata
      x,y,z
      1,2,3", skip = 2)
    ~~~
    {: .r}
    
    
    
    ~~~
    # A tibble: 1 x 3
          x     y     z
      <int> <int> <int>
    1     1     2     3
    ~~~
    {: .output}
    
    
    
    ~~~
    read_csv("# A comment I want to skip
      x,y,z
      1,2,3", comment = "#")
    ~~~
    {: .r}
    
    
    
    ~~~
    # A tibble: 1 x 3
          x     y     z
      <int> <int> <int>
    1     1     2     3
    ~~~
    {: .output}
    
1.  The data might not have column names. You can use `col_names = FALSE` to
    tell `read_csv()` not to treat the first row as headings, and instead
    label them sequentially from `X1` to `Xn`. Alternatively you can pass `col_names` 
    a character vector which will be used as the column names:
    
    
    ~~~
    read_csv("1,2,3\n4,5,6", col_names = c("x", "y", "z"))
    ~~~
    {: .r}
    
    
    
    ~~~
    # A tibble: 2 x 3
          x     y     z
      <int> <int> <int>
    1     1     2     3
    2     4     5     6
    ~~~
    {: .output}

Another option that commonly needs tweaking is `na`: this specifies the value (or values) 
that are used to represent missing values in your file:


~~~
read_csv("a,b,c\n1,2,.", na = ".")
~~~
{: .r}



~~~
# A tibble: 1 x 3
      a     b     c
  <int> <int> <chr>
1     1     2  <NA>
~~~
{: .output}

### Compared to base R

If you've used R before, you might wonder why we're not using `read.csv()`. 
There are a few good reasons to favour readr functions over the base equivalents:

* They are typically much faster (~10x) than their base equivalents.
  There is even a faster version, `data.table::fread()`, but it doesn't fit 
  quite so well into the tidyverse.

* They produce tibbles, they don't convert character vectors to factors,
  use row names, or munge the column names. These are common sources of
  frustration with the base R functions.

* They are more reproducible. Base R functions inherit some behaviour from
  your operating system and environment variables, so import code that works 
  on your computer might not work on someone else's.

## Parsing a vector

Before we get into the details of how readr reads files from disk, we need to take a little 
detour to talk about the `parse_*()` functions. These functions take a character vector and 
return a more specialised vector like a logical, integer, or date. These functions are useful 
in their own right, but are also an important building block for `readr`. Like all functions 
in the tidyverse, the `parse_*()` functions are uniform: the first argument is a character 
vector to parse, and the `na` argument specifies which strings should be treated as missing:


~~~
parse_integer(c("1", "231", ".", "456"), na = ".")
~~~
{: .r}



~~~
[1]   1 231  NA 456
~~~
{: .output}

If parsing fails, you'll get a warning:


~~~
x <- parse_integer(c("123", "345", "abc", "123.45"))
~~~
{: .r}



~~~
Warning in rbind(names(probs), probs_f): number of columns of result is not
a multiple of vector length (arg 1)
~~~
{: .error}



~~~
Warning: 2 parsing failures.
row # A tibble: 2 x 4 col     row   col               expected actual expected   <int> <int>                  <chr>  <chr> actual 1     3    NA             an integer    abc row 2     4    NA no trailing characters    .45
~~~
{: .error}

And the failures will be missing in the output:


~~~
x
~~~
{: .r}



~~~
[1] 123 345  NA  NA
attr(,"problems")
# A tibble: 2 x 4
    row   col               expected actual
  <int> <int>                  <chr>  <chr>
1     3    NA             an integer    abc
2     4    NA no trailing characters    .45
~~~
{: .output}

There are severeal important parsers, but we will look just at two types:

1.  number parsers: `parse_double()` is a strict numeric parser, and `parse_number()` 
    is a flexible numeric parser. 
1.  character parser: `parse_character()`.

The following sections describe these parsers in more detail.

### Numbers

It seems like it should be straightforward to parse a number, but three problems make it tricky:

1. People write numbers differently in different parts of the world.
   
1. Numbers are often surrounded by other characters that provide some
   context, like "$1000" or "10%".

1. Numbers often contain "grouping" characters to make them easier to read, 
   like "1,000,000", and these grouping characters vary around the world.

To address the first problem, readr has the notion of a "locale", an object that specifies 
parsing options that differ from place to place. When parsing numbers, the most important 
option is the character you use for the decimal mark. You can override the default value 
of `.` by creating a new locale and setting the `decimal_mark` argument:


~~~
parse_double("1.23")
~~~
{: .r}



~~~
[1] 1.23
~~~
{: .output}



~~~
parse_double("1,23", locale = locale(decimal_mark = ","))
~~~
{: .r}



~~~
[1] 1.23
~~~
{: .output}

`parse_number()` addresses the second problem: it ignores non-numeric characters before and 
after the number. This is particularly useful for currencies and percentages, but also works 
to extract numbers embedded in text.


~~~
parse_number("$100")
~~~
{: .r}



~~~
[1] 100
~~~
{: .output}



~~~
parse_number("20%")
~~~
{: .r}



~~~
[1] 20
~~~
{: .output}



~~~
parse_number("It cost $123.45")
~~~
{: .r}



~~~
[1] 123.45
~~~
{: .output}

The final problem is addressed by the combination of `parse_number()` and the locale as `parse_number()` 
will ignore the "grouping mark":


~~~
# Used in America
parse_number("$123,456,789")
~~~
{: .r}



~~~
[1] 123456789
~~~
{: .output}



~~~
# Used in many parts of Europe
parse_number("123.456.789", locale = locale(grouping_mark = "."))
~~~
{: .r}



~~~
[1] 123456789
~~~
{: .output}



~~~
# Used in Switzerland
parse_number("123'456'789", locale = locale(grouping_mark = "'"))
~~~
{: .r}



~~~
[1] 123456789
~~~
{: .output}

### Strings {#readr-strings}

It seems like `parse_character()` should be really simple but it's not, as there are multiple ways to 
represent the same string. To understand what's going on, we need to dive into the details of how 
computers represent strings. In R, we can get at the underlying representation of a string using `charToRaw()`:


~~~
charToRaw("Hadley")
~~~
{: .r}



~~~
[1] 48 61 64 6c 65 79
~~~
{: .output}

Each hexadecimal number represents a byte of information: `48` is H, `61` is a, and so on. The mapping 
from hexadecimal number to character is called the encoding, and in this case the encoding is called ASCII. 
ASCII does a great job of representing English characters, because it's the __American__ Standard Code for 
Information Interchange.

Things get more complicated for languages other than English as there are many competing standards for encoding 
non-English characters. Fortunately, today there is one standard that is supported almost everywhere: UTF-8. 
UTF-8 can encode just about every character used by humans today, as well as many extra symbols (like emoji!).

readr uses UTF-8 everywhere: it assumes your data is UTF-8 encoded when you read it, and always uses it when 
writing. This is a good default, but will fail for data produced by older systems that don't understand UTF-8. 
If this happens to you, your strings will look weird when you print them. Sometimes just one or two characters 
might be messed up; other times you'll get complete gibberish. For example:


~~~
x1 <- "El Ni\xf1o was particularly bad this year"
x2 <- "\x82\xb1\x82\xf1\x82\xc9\x82\xbf\x82\xcd"

x1
~~~
{: .r}



~~~
[1] "El Ni\xf1o was particularly bad this year"
~~~
{: .output}



~~~
x2
~~~
{: .r}



~~~
[1] "\x82\xb1\x82\xf1\x82\u0242\xbf\x82\xcd"
~~~
{: .output}

To fix the problem you need to specify the encoding in `parse_character()`:


~~~
parse_character(x1, locale = locale(encoding = "Latin1"))
~~~
{: .r}



~~~
[1] "El Niño was particularly bad this year"
~~~
{: .output}



~~~
parse_character(x2, locale = locale(encoding = "Shift-JIS"))
~~~
{: .r}



~~~
[1] "こんにちは"
~~~
{: .output}

How do you find the correct encoding? If you're lucky, it'll be included somewhere in the data documentation. If not, 
readr provides  `guess_encoding()` to help you figure it out. It's not foolproof, and it works better when you have 
lots of text (unlike here), but it's a reasonable place to start.


~~~
guess_encoding(charToRaw(x1))
~~~
{: .r}



~~~
# A tibble: 2 x 2
    encoding confidence
       <chr>      <dbl>
1 ISO-8859-1       0.46
2 ISO-8859-9       0.23
~~~
{: .output}



~~~
guess_encoding(charToRaw(x2))
~~~
{: .r}



~~~
# A tibble: 1 x 2
  encoding confidence
     <chr>      <dbl>
1   KOI8-R       0.42
~~~
{: .output}

The first argument to `guess_encoding()` can either be a path to a file, or, as in this case, a raw vector 
(useful if the strings are already in R).


## Parsing a file

readr uses a heuristic to figure out the type of each column: it reads the first 1000 rows and uses some 
(moderately conservative) heuristics to figure out the type of each column. You can emulate this process 
with a character vector using `guess_parser()`, which returns readr's best guess, and `parse_guess()` which 
uses that guess to parse the column:


~~~
guess_parser("2010-10-01")
~~~
{: .r}



~~~
[1] "date"
~~~
{: .output}



~~~
guess_parser("15:01")
~~~
{: .r}



~~~
[1] "time"
~~~
{: .output}



~~~
guess_parser(c("TRUE", "FALSE"))
~~~
{: .r}



~~~
[1] "logical"
~~~
{: .output}



~~~
guess_parser(c("1", "5", "9"))
~~~
{: .r}



~~~
[1] "integer"
~~~
{: .output}



~~~
guess_parser(c("12,352,561"))
~~~
{: .r}



~~~
[1] "number"
~~~
{: .output}



~~~
str(parse_guess("2010-10-10"))
~~~
{: .r}



~~~
 Date[1:1], format: 
~~~
{: .output}



~~~
Warning in format.POSIXlt(as.POSIXlt(x), ...): unknown timezone 'zone/tz/
2019b.1.0/zoneinfo/America/Chicago'
~~~
{: .error}



~~~
"2010-10-10"
~~~
{: .output}

The heuristic tries each of the following types, stopping when it finds a match:

* logical: contains only "F", "T", "FALSE", or "TRUE".
* integer: contains only numeric characters (and `-`).
* double: contains only valid doubles (including numbers like `4.5e-5`).
* number: contains valid doubles with the grouping mark inside.
* time: matches the default `time_format`.
* date: matches the default `date_format`.
* date-time: any ISO8601 date.

If none of these rules apply, then the column will stay as a vector of strings.

### Problems

These defaults don't always work for larger files. There are two basic problems:

1.  The first thousand rows might be a special case, and readr guesses
    a type that is not sufficiently general. For example, you might have 
    a column of doubles that only contains integers in the first 1000 rows. 

1.  The column might contain a lot of missing values. If the first 1000
    rows contain only `NA`s, readr will guess that it's a character 
    vector, whereas you probably want to parse it as something more
    specific.

readr contains a challenging CSV that illustrates both of these problems:


~~~
head(read_csv(readr_example("challenge.csv")))
~~~
{: .r}



~~~
Parsed with column specification:
cols(
  x = col_integer(),
  y = col_character()
)
~~~
{: .output}



~~~
Warning in rbind(names(probs), probs_f): number of columns of result is not
a multiple of vector length (arg 1)
~~~
{: .error}



~~~
Warning: 1000 parsing failures.
row # A tibble: 5 x 5 col     row   col               expected             actual expected   <int> <chr>                  <chr>              <chr> actual 1  1001     x no trailing characters .23837975086644292 file 2  1002     x no trailing characters .41167997173033655 row 3  1003     x no trailing characters  .7460716762579978 col 4  1004     x no trailing characters   .723450553836301 expected 5  1005     x no trailing characters   .614524137461558 actual # ... with 1 more variables: file <chr>
... ................. ... ....................................................... ........ ....................................................... ...... ....................................................... .... ....................................................... ... ....................................................... ... ....................................................... ........ ....................................................... ...... .......................................
See problems(...) for more details.
~~~
{: .error}



~~~
# A tibble: 6 x 2
      x     y
  <int> <chr>
1   404  <NA>
2  4172  <NA>
3  3004  <NA>
4   787  <NA>
5    37  <NA>
6  2332  <NA>
~~~
{: .output}



~~~
challenge <- read_csv(readr_example("challenge.csv"))
~~~
{: .r}



~~~
Parsed with column specification:
cols(
  x = col_integer(),
  y = col_character()
)
~~~
{: .output}



~~~
Warning in rbind(names(probs), probs_f): number of columns of result is not a multiple of vector length (arg 1)
Warning in rbind(names(probs), probs_f): 1000 parsing failures.
row # A tibble: 5 x 5 col     row   col               expected             actual expected   <int> <chr>                  <chr>              <chr> actual 1  1001     x no trailing characters .23837975086644292 file 2  1002     x no trailing characters .41167997173033655 row 3  1003     x no trailing characters  .7460716762579978 col 4  1004     x no trailing characters   .723450553836301 expected 5  1005     x no trailing characters   .614524137461558 actual # ... with 1 more variables: file <chr>
... ................. ... ....................................................... ........ ....................................................... ...... ....................................................... .... ....................................................... ... ....................................................... ... ....................................................... ........ ....................................................... ...... .......................................
See problems(...) for more details.
~~~
{: .error}

(Note the use of `readr_example()` which finds the path to one of the files included with the package)

There are two printed outputs: the column specification generated by looking at the first 1000 rows, 
and the first five parsing failures. It's always a good idea to explicitly pull out the `problems()`, 
so you can explore them in more depth:


~~~
problems(challenge)
~~~
{: .r}



~~~
# A tibble: 1,000 x 5
     row   col               expected             actual
   <int> <chr>                  <chr>              <chr>
 1  1001     x no trailing characters .23837975086644292
 2  1002     x no trailing characters .41167997173033655
 3  1003     x no trailing characters  .7460716762579978
 4  1004     x no trailing characters   .723450553836301
 5  1005     x no trailing characters   .614524137461558
 6  1006     x no trailing characters   .473980569280684
 7  1007     x no trailing characters  .5784610391128808
 8  1008     x no trailing characters  .2415937229525298
 9  1009     x no trailing characters .11437866208143532
10  1010     x no trailing characters  .2983446326106787
# ... with 990 more rows, and 1 more variables: file <chr>
~~~
{: .output}

A good strategy is to work column by column until there are no problems remaining. Here we can see that there 
are a lot of parsing problems with the `x` column - there are trailing characters after the integer value. That 
suggests we need to use a double parser instead.

To fix the call, start by copying and pasting the column specification into your original call. Then you can 
tweak the type of the `x` column:


~~~
challenge <- read_csv(
  readr_example("challenge.csv"), 
  col_types = cols(
    x = col_double(),
    y = col_character()
  )
)
~~~
{: .r}

That fixes the first problem, but if we look at the last few rows, you'll see that they're dates stored in a character vector:


~~~
tail(challenge)
~~~
{: .r}



~~~
# A tibble: 6 x 2
          x          y
      <dbl>      <chr>
1 0.8052743 2019-11-21
2 0.1635163 2018-03-29
3 0.4719390 2014-08-04
4 0.7183186 2015-08-16
5 0.2698786 2020-02-04
6 0.6082372 2019-01-06
~~~
{: .output}

You can fix that by specifying that `y` is a date column:


~~~
challenge <- read_csv(
  readr_example("challenge.csv"), 
  col_types = cols(
    x = col_double(),
    y = col_date()
  )
)
tail(challenge)
~~~
{: .r}



~~~
# A tibble: 6 x 2
          x          y
      <dbl>     <date>
1 0.8052743 2019-11-21
2 0.1635163 2018-03-29
3 0.4719390 2014-08-04
4 0.7183186 2015-08-16
5 0.2698786 2020-02-04
6 0.6082372 2019-01-06
~~~
{: .output}

Every `parse_xyz()` function has a corresponding `col_xyz()` function. You use `parse_xyz()` when the data is in a character vector in R already; you use `col_xyz()` when you want to tell readr how to load the data.

It's a good idea to supply `col_types` from the print-out provided by readr in your `read_csv` call. This 
ensures that you have a consistent and reproducible data import script. If you rely on the default guesses 
and your data changes, readr will continue to read it in. If you want to be really strict, use 
`stop_for_problems()`: that will throw an error and stop your script if there are any parsing problems.


## Writing to a file

readr also comes with two useful functions for writing data back to disk: `write_csv()` and `write_tsv()`. Both functions increase the chances of the output file being read back in correctly by:

* Always encoding strings in UTF-8.
  
* Saving dates and date-times in ISO8601 format so they are easily
  parsed elsewhere.

If you want to export a csv file to Excel, use `write_excel_csv()` --- this writes a special character (a "byte order mark") at the start of the file which tells Excel that you're using the UTF-8 encoding.

The most important arguments are `x` (the data frame to save), and `path` (the location to save it). You can also specify how missing values are written with `na`, and if you want to `append` to an existing file.


~~~
write_csv(challenge, "challenge.csv")
~~~
{: .r}

Note that the type information is lost when you save to csv:


~~~
challenge
~~~
{: .r}



~~~
# A tibble: 2,000 x 2
       x      y
   <dbl> <date>
 1   404     NA
 2  4172     NA
 3  3004     NA
 4   787     NA
 5    37     NA
 6  2332     NA
 7  2489     NA
 8  1449     NA
 9  3665     NA
10  3863     NA
# ... with 1,990 more rows
~~~
{: .output}



~~~
write_csv(challenge, "challenge-2.csv")
read_csv("challenge-2.csv")
~~~
{: .r}



~~~
Parsed with column specification:
cols(
  x = col_integer(),
  y = col_character()
)
~~~
{: .output}



~~~
# A tibble: 2,000 x 2
       x     y
   <int> <chr>
 1   404  <NA>
 2  4172  <NA>
 3  3004  <NA>
 4   787  <NA>
 5    37  <NA>
 6  2332  <NA>
 7  2489  <NA>
 8  1449  <NA>
 9  3665  <NA>
10  3863  <NA>
# ... with 1,990 more rows
~~~
{: .output}

This makes CSVs a little unreliable for caching interim results---you need to recreate the column specification every time you load in. There are two alternatives:

1.  `write_rds()` and `read_rds()` are uniform wrappers around the base 
    functions `readRDS()` and `saveRDS()`. These store data in R's custom 
    binary format called RDS:
    
    
    ~~~
    write_rds(challenge, "challenge.rds")
    read_rds("challenge.rds")
    ~~~
    {: .r}
    
    
    
    ~~~
    # A tibble: 2,000 x 2
           x      y
       <dbl> <date>
     1   404     NA
     2  4172     NA
     3  3004     NA
     4   787     NA
     5    37     NA
     6  2332     NA
     7  2489     NA
     8  1449     NA
     9  3665     NA
    10  3863     NA
    # ... with 1,990 more rows
    ~~~
    {: .output}
  
1.  The feather package implements a fast binary file format that can
    be shared across programming languages:
    
    
    ~~~
    library(feather)
    write_feather(challenge, "challenge.feather")
    read_feather("challenge.feather")
    #> # A tibble: 2,000 x 2
    #>       x      y
    #>   <dbl> <date>
    #> 1   404   <NA>
    #> 2  4172   <NA>
    #> 3  3004   <NA>
    #> 4   787   <NA>
    #> 5    37   <NA>
    #> 6  2332   <NA>
    #> # ... with 1,994 more rows
    ~~~
    {: .r}

Feather tends to be faster than RDS and is usable outside of R. However, RDS supports list-columns while feather currently does not.



## Other types of data

To get other types of data into R, we recommend starting with the tidyverse packages listed below. They're certainly 
not perfect, but they are a good place to start. For rectangular data:

* __haven__ reads SPSS, Stata, and SAS files.

* __readxl__ reads excel files (both `.xls` and `.xlsx`).

* __DBI__, along with a database specific backend (e.g. __RMySQL__, 
  __RSQLite__, __RPostgreSQL__ etc) allows you to run SQL queries against a 
  database and return a data frame.

For hierarchical data: use __jsonlite__ (by Jeroen Ooms) for json, and __xml2__ for XML. Jenny Bryan has some excellent worked examples at <https://jennybc.github.io/purrr-tutorial/>.

For other file types, try the [R data import/export manual](https://cran.r-project.org/doc/manuals/r-release/R-data.html) and the [__rio__](https://github.com/leeper/rio) package.

> ## Homework
>
> Parse the UNIX assignment files (fang_et_al_genotypes.txt and snp_position.txt) 
> into R.
>
> - What problems did you encounter?
> - Did R guessed the data type correctly?
> - If not, how did you fix it?
>
> Try to use the `t` function to transpose the first file
>
> - Did it work?
> - Can you figure out how to make it work?
>
{: .discussion}


