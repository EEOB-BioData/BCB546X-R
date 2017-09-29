---
title: "Exploring Data Frames"
teaching: 50
exercises: 30
questions:
- "How can I manipulate a data frame?"
- "How can I join two data frames?"
objectives:
- "Be able to add and remove rows and columns."
- "Be able to remove rows with `NA` values."
- "Be able to append two data frames"
- "Be able to articulate what a `factor` is and how to convert between `factor` and `character`."
- "Be able to find basic properties of a data frames including size, class or type of the columns, names, and first few rows."
- "Be able to match columns in two data frames"
- "Be able to join two data frames"
keypoints:
- "Use `cbind()` to add a new column to a data frame."
- "Use `rbind()` to add a new row to a data frame."
- "Remove rows from a data frame."
- "Use `na.omit()` to remove rows from a data frame with `NA` values."
- "Use `levels()` and `as.character()` to explore and manipulate factors"
- "Use `str()`, `nrow()`, `ncol()`, `dim()`, `colnames()`, `rownames()`, `head()` and `typeof()` to understand structure of the data frame"
- "Read in a csv file using `read.csv()`"
- "Understand `length()` of a data frame"
- "Use `x%in%y` or `match` to match columns in dataframes"
- "Use an index created by `match` or `merge` to merge two dataframes"
---



## Data frames {#data-frames}

A data frame is the most common way of storing data in R, and if [used systematically](http://vita.had.co.nz/papers/tidy-data.pdf) makes data analysis easier. Under the hood, a data frame is a list of equal-length vectors. This makes it a 2-dimensional structure, so it shares properties of both the matrix and the list.  This means that a data frame has `names()`, `colnames()`, and `rownames()`, although `names()` and `colnames()` are the same thing. The `length()` of a data frame is the length of the underlying list and so is the same as `ncol()`; `nrow()` gives the number of rows.

You can subset a data frame like a 1d structure (where it behaves like a list), or a 2d structure (where it behaves like a matrix).

### Creation

You create a data frame using `data.frame()`, which takes named vectors as input:


~~~
cats <- data.frame(coat = c("calico", "black", "tabby"), weight = c(2.1, 5.0, 3.2), likes_string = c(1, 0, 1))
str(df)
~~~
{: .r}



~~~
function (x, df1, df2, ncp, log = FALSE)  
~~~
{: .output}

*Note that the `data.frame()`'s default behaviour which turns strings into factors. 
Use `stringAsFactors = FALSE` to suppress this behaviour!* 

### Testing and coercion

Because a `data.frame` is an S3 class, its type reflects the underlying vector used to build it: the list. To check if an object is a data frame, use `class()` or test explicitly with `is.data.frame()`:


~~~
typeof(cats)
~~~
{: .r}



~~~
[1] "list"
~~~
{: .output}



~~~
class(cats)
~~~
{: .r}



~~~
[1] "data.frame"
~~~
{: .output}



~~~
is.data.frame(cats)
~~~
{: .r}



~~~
[1] TRUE
~~~
{: .output}

You can coerce an object to a data frame with `as.data.frame()`:

* A vector will create a one-column data frame.  
* A list will create one column for each element; it's an error if they're not all the same length.  
* A matrix will create a data frame with the same number of columns and rows as the matrix.  

## Reading data into a dataframe

One of R's most powerful features is its ability to deal with tabular data - like what you might already have 
in a spreadsheet or a CSV. The `read.csv` function is used for reading in tabular data stored in a text file 
where the columns of data are delimited by commas (csv = comma separated values). CSV data are read into dataframes.

Let's create a file in the `data/` directory, called `feline-data.csv` with the same data as above:


~~~
coat,weight,likes_string
calico,2.1,1
black,5.0,0
tabby,3.2,1
~~~
{: .r}

> ## Tip: Editing Text files in R
>
> You can create `data/feline-data.csv` using a text editor (vi or Nano),
> or within RStudio with the **File -> New File -> Text File** menu item.
{: .callout}

We can load this into R via the following:


~~~
cats <- read.csv(file = "data/feline-data.csv")
cats
~~~
{: .r}



~~~
    coat weight likes_string
1 calico    2.1            1
2  black    5.0            0
3  tabby    3.2            1
~~~
{: .output}

We can begin exploring our dataset right away, pulling out columns by specifying
them using the `$` operator:


~~~
cats$weight
~~~
{: .r}



~~~
[1] 2.1 5.0 3.2
~~~
{: .output}



~~~
cats$coat
~~~
{: .r}



~~~
[1] calico black  tabby 
Levels: black calico tabby
~~~
{: .output}

We can do other operations on the columns:


~~~
## Say we discovered that the scale weighs two Kg light:
cats$weight + 2
~~~
{: .r}



~~~
[1] 4.1 7.0 5.2
~~~
{: .output}



~~~
paste("My cat is", cats$coat)
~~~
{: .r}



~~~
[1] "My cat is calico" "My cat is black"  "My cat is tabby" 
~~~
{: .output}

We can ask what type of data something is:


~~~
typeof(cats$weight)
~~~
{: .r}



~~~
[1] "double"
~~~
{: .output}

> ## IMPORTANT: Subsetting lists and dataframes
> ### Subsetting lists: three functions: `[`, `[[`, and `$`.
>
> Using `[` will always return a list. If you want to *subset* a list, but not
> *extract* an element, then you will likely use `[`.
> 
> ~~~~~~~~~~
> cats[1]
> ~~~~~~~~~~
> {: .source}
>
> To *extract* individual elements of a list, you need to use the double-square
> bracket function: `[[`.
> 
> ~~~~~~~~~~
> cats[[1]]
> ~~~~~~~~~~
> {: .source}
> 
> You **can't** extract more than one element at once:
> 
> ~~~~~~~~~~
> cats[[1:2]]
> ~~~~~~~~~~
> {: .source}
> 
> **Nor** use it to skip elements:
> 
> ~~~~~~~~~~
> cats[[-1]]
> ~~~~~~~~~~
> {: .source}
> 
> But you can use names to both subset and extract elements:
> 
> ~~~~~~~~~~
> d[["coat"]]
> ~~~~~~~~~~
> {: .source}
>
> The `$` function is a shorthand way for extracting elements by name:
> 
> ~~~~~~~~~~
> cats$coat
> ~~~~~~~~~~
> {: .source}
>
> ### Subsetting dataframes
>
> Remember the data frames are lists underneath the hood, so similar rules
> apply. However they are also two dimensional objects:
> 
> `[` with one argument will act the same was as for lists, where each list
> element corresponds to a column. The resulting object will be a data frame:
>
> With two arguments, `[` behaves the same way as for matrices, 
> exctracting raws (first argument) and columns (second argument):
> 
> ~~~~~~~~~~
> cats[1:3,]
> ~~~~~~~~~~
> {: .source}
>
> If we subset a single row, the result will be a data frame (because
> the elements are mixed types):
> 
> ~~~~~~~~~~
> cats[3,]
> ~~~~~~~~~~
> {: .source}
>
> But for a single column the result will be a vector (this can
> be changed with the third argument, `drop = FALSE`).
> 
> ~~~~~~~~~~
> cats[,3]
> ~~~~~~~~~~
> {: .source}
>
{: .callout}


## Adding columns and rows in data frame with `cbind` and `rbind`

We learned last time that the columns in a list (and data frame) were vectors, so that our
data are consistent in type throughout the column. As such, if we want to add a
new column, we need to start by making a new vector:


~~~
age <- c(2,3,5,12)
cats
~~~
{: .r}



~~~
    coat weight likes_string
1 calico    2.1            1
2  black    5.0            0
3  tabby    3.2            1
~~~
{: .output}

We can then add this as a column with the `cbind` function:


~~~
cats <- cbind(cats, age)
~~~
{: .r}



~~~
Error in data.frame(..., check.names = FALSE): arguments imply differing number of rows: 3, 4
~~~
{: .error}

OOPS! Why didn't this work? 
Let's try again:


~~~
cats
~~~
{: .r}



~~~
    coat weight likes_string
1 calico    2.1            1
2  black    5.0            0
3  tabby    3.2            1
~~~
{: .output}



~~~
age <- c(4,5,8)
cats <- cbind(cats, age)
cats
~~~
{: .r}



~~~
    coat weight likes_string age
1 calico    2.1            1   4
2  black    5.0            0   5
3  tabby    3.2            1   8
~~~
{: .output}

We can also add raws to the dataframe (as lists) (why?):


~~~
newRow <- list("tortoiseshell", 3.3, TRUE, 9)
cats <- rbind(cats, newRow)
~~~
{: .r}



~~~
Warning in `[<-.factor`(`*tmp*`, ri, value = "tortoiseshell"): invalid
factor level, NA generated
~~~
{: .error}

## Factors

The previous command produced a warning: "invalid factor level, NA generated".
This is because when R creates a factor, it only allows whatever is originally 
there when our data was first loaded, which was 'black', 'calico' and 'tabby'. 
Anything new that doesn't fit into one of these categories is rejected as nonsense (becomes NA).

The warning is telling us that we unsuccessfully added 'tortoiseshell' to our
*coat* factor, but 3.3 (a numeric), TRUE (a logical), and 9 (a numeric) were
successfully added to *weight*, *likes_string*, and *age*, respectfully, since
those values are not factors. To successfully add a cat with a
'tortoiseshell' *coat*, explicitly add 'tortoiseshell' as a *level* in the factor:


~~~
levels(cats$coat)
~~~
{: .r}



~~~
[1] "black"  "calico" "tabby" 
~~~
{: .output}



~~~
levels(cats$coat) <- c(levels(cats$coat), 'tortoiseshell')
cats <- rbind(cats, list("tortoiseshell", 3.3, TRUE, 9))
~~~
{: .r}

Alternatively, we can change a factor column to a character vector; we lose the
handy categories of the factor, but can subsequently add any word we want to the
column without babysitting the factor levels:


~~~
str(cats)
~~~
{: .r}



~~~
'data.frame':	5 obs. of  4 variables:
 $ coat        : Factor w/ 4 levels "black","calico",..: 2 1 3 NA 4
 $ weight      : num  2.1 5 3.2 3.3 3.3
 $ likes_string: int  1 0 1 1 1
 $ age         : num  4 5 8 9 9
~~~
{: .output}



~~~
cats$coat <- as.character(cats$coat)
str(cats)
~~~
{: .r}



~~~
'data.frame':	5 obs. of  4 variables:
 $ coat        : chr  "calico" "black" "tabby" NA ...
 $ weight      : num  2.1 5 3.2 3.3 3.3
 $ likes_string: int  1 0 1 1 1
 $ age         : num  4 5 8 9 9
~~~
{: .output}

We can also suppress conversion of character vectors into factors by setting stringsAsFactors=FALSE. This works for `data.frame()` as well as `read.csv()` and `read.delim()`.

## Removing rows

We now know how to add rows and columns to our data frame in R - but in our
first attempt to add a 'tortoiseshell' cat to the data frame we've accidentally
added a garbage row:


~~~
cats
~~~
{: .r}



~~~
           coat weight likes_string age
1        calico    2.1            1   4
2         black    5.0            0   5
3         tabby    3.2            1   8
4          <NA>    3.3            1   9
5 tortoiseshell    3.3            1   9
~~~
{: .output}

We can ask for a data frame minus this offending row:


~~~
cats[-4,]
~~~
{: .r}



~~~
           coat weight likes_string age
1        calico    2.1            1   4
2         black    5.0            0   5
3         tabby    3.2            1   8
5 tortoiseshell    3.3            1   9
~~~
{: .output}

Notice the comma with nothing after it indicates that we want to drop the entire fourth row.

Note: We could also remove both new rows at once by putting the row numbers
inside of a vector: `cats[c(-4,-5),]`

Alternatively, we can drop all rows with `NA` values:


~~~
na.omit(cats)
~~~
{: .r}



~~~
           coat weight likes_string age
1        calico    2.1            1   4
2         black    5.0            0   5
3         tabby    3.2            1   8
5 tortoiseshell    3.3            1   9
~~~
{: .output}


> ## What did the previous command do?
>
> It's important to notice that the previous command simply printed our `cats` dataframe excluding some rows
> In order to make changes in the dataframe itself, we want to assign them to the dataset itslef:
>
> ~~~~~~~~
> cats <- cats[-4,]
> ~~~~~~~~
>{: .source}
{:  .callout}


## Appending data frame

The key to remember when adding data to a data frame is that *columns are
vectors or factors, and rows are lists.* We can also glue two data frames
together with `rbind`:


~~~
cats <- rbind(cats, cats)
cats
~~~
{: .r}



~~~
            coat weight likes_string age
1         calico    2.1            1   4
2          black    5.0            0   5
3          tabby    3.2            1   8
4           <NA>    3.3            1   9
5  tortoiseshell    3.3            1   9
6         calico    2.1            1   4
7          black    5.0            0   5
8          tabby    3.2            1   8
9           <NA>    3.3            1   9
10 tortoiseshell    3.3            1   9
~~~
{: .output}
If you assigned row names in your dataframe, they may be unnecessarily complicated when we combine two dataframes. We can remove the rownames, and R will automatically re-name them sequentially:


~~~
rownames(cats) <- NULL
cats
~~~
{: .r}



~~~
            coat weight likes_string age
1         calico    2.1            1   4
2          black    5.0            0   5
3          tabby    3.2            1   8
4           <NA>    3.3            1   9
5  tortoiseshell    3.3            1   9
6         calico    2.1            1   4
7          black    5.0            0   5
8          tabby    3.2            1   8
9           <NA>    3.3            1   9
10 tortoiseshell    3.3            1   9
~~~
{: .output}


> ## Challenge 1
>
> Make a data frame that holds the following information for yourself:
>
> - first name
> - last name
> - lucky number
>
> Then use `rbind` to add an entry for the people sitting beside you.
> Finally, use `cbind` to add a column with each person's answer to the question, "Is it time for a break?"
>
> > ## Solution to Challenge 1
> > 
> > ~~~
> > df <- data.frame(first = c('Grace'),
> >                  last = c('Hopper'),
> >                  lucky_number = c(0),
> >                  stringsAsFactors = FALSE)
> > df <- rbind(df, list('Marie', 'Curie', 238) )
> > df <- cbind(df, coffeetime = c(TRUE,TRUE))
> > ~~~
> > {: .r}
> {: .solution}
{: .challenge}

After all this work, if you want to save your dataframe back to a file use:


~~~
write.csv(cats, file = "data/new_cats.csv")
~~~
{: .r}

## Realistic example

With a knowledge of basic R language essentials, we’re ready to start working with real data. 
We’ll work on a few datasets following examples in Chapter 8 of the Buffalo book. 
All files to load these datasets into R are available in Chapter 8 directory on GitHub.

We starts with Dataset_S1.txt on GitHub, which contains estimates of population genetics statistics such 
as nucleotide diversity (e.g., the columns Pi and Theta), recombination (column Recombination), and sequence 
divergence as estimated by percent identity between human and chimpanzee genomes (column Divergence). 
Other columns contain information about the sequencing depth (depth), and GC content 
(percent.GC). We’ll only work with a few columns in our examples; see the description 
of Dataset_S1.txt in the original paper (Spencer et al.2006) for more detail. 
Dataset_S1.txt includes these estimates for 1kb windows in human chromosome 20.

> ## Miscellaneous Tips
>
>
> * Files can be downloaded directly from the Internet into a local folder of your choice using the `download.file` function.
> The `read.csv` function can then be executed to read the downloaded file from the download location, for example,
> 
> ~~~
> download.file("https://raw.githubusercontent.com/vsbuffalo/bds-files/master/chapter-08-r/Dataset_S1.txt", destfile = "data/Dataset_S1.txt")
> d <- read.csv("data/Dataset_S1.txt")
> ~~~
> {: .r}
>
> * Alternatively, you can read files directly from the Internet into R by replacing the file paths with a web address in `read.csv`. In doing this no local copy of the csv file is first saved onto your computer. For example,
> 
> ~~~
> d <- read.csv("https://raw.githubusercontent.com/vsbuffalo/bds-files/master/chapter-08-r/Dataset_S1.txt")
> ~~~
> {: .r}
>
> * Finally, you can read directly from excel spreadsheets without
> converting them to plain text first by using the [readxl](https://cran.r-project.org/web/packages/readxl/index.html) package.
{: .callout}

Note that R’s read.csv() and read.delim() functions have numerous arguments, many of which will need to be adjusted for certain files you’ll come across in bioinformatics. See table below (Table 8-4 of the Buffalo book) for a list of some commonly used arguments, and/or consult help(read.csv) for full documentation.

image: ![Table 8.4](../fig/table-8.4.png)

Ok, let’s take a look at the dataframe we’ve loaded in `d` with `str`:


~~~
str(d)
~~~
{: .r}



~~~
'data.frame':	59140 obs. of  16 variables:
 $ start          : int  55001 56001 57001 58001 59001 60001 61001 62001 63001 64001 ...
 $ end            : int  56000 57000 58000 59000 60000 61000 62000 63000 64000 65000 ...
 $ total.SNPs     : int  0 5 1 7 4 6 7 1 1 3 ...
 $ total.Bases    : int  1894 6683 9063 10256 8057 7051 6950 8834 9629 7999 ...
 $ depth          : num  3.41 6.68 9.06 10.26 8.06 ...
 $ unique.SNPs    : int  0 2 1 3 4 2 2 1 1 1 ...
 $ dhSNPs         : int  0 2 0 2 0 1 1 0 0 1 ...
 $ reference.Bases: int  556 1000 1000 1000 1000 1000 1000 1000 1000 1000 ...
 $ Theta          : num  0 8.01 3.51 9.93 12.91 ...
 $ Pi             : num  0 10.35 1.99 9.56 8.51 ...
 $ Heterozygosity : num  0 7.48 1.1 6.58 4.96 ...
 $ X.GC           : num  54.8 42.4 37.2 38 41.3 ...
 $ Recombination  : num  0.0096 0.0096 0.0096 0.0096 0.0096 ...
 $ Divergence     : num  0.00301 0.01802 0.00701 0.01201 0.02402 ...
 $ Constraint     : int  0 0 0 0 0 0 0 0 58 0 ...
 $ SNPs           : int  0 0 0 0 0 0 0 0 1 1 ...
~~~
{: .output}

We can also look at the data in the databrame with the function `head`:

~~~
head(d, n=3)
~~~
{: .r}



~~~
  start   end total.SNPs total.Bases depth unique.SNPs dhSNPs
1 55001 56000          0        1894  3.41           0      0
2 56001 57000          5        6683  6.68           2      2
3 57001 58000          1        9063  9.06           1      0
  reference.Bases Theta     Pi Heterozygosity    X.GC Recombination
1             556 0.000  0.000          0.000 54.8096   0.009601574
2            1000 8.007 10.354          7.481 42.4424   0.009601574
3            1000 3.510  1.986          1.103 37.2372   0.009601574
   Divergence Constraint SNPs
1 0.003006012          0    0
2 0.018018020          0    0
3 0.007007007          0    0
~~~
{: .output}

> ## Challenge 2
>
> Use what you've learned about factors, lists and vectors,
> as well as the output of functions like `colnames` and `dim`
> to explain what everything that `str` prints out for this dataset means.
> If there are any parts you can't interpret, discuss with your neighbors!
>
> > ## Solution to Challenge 2
> >
> > The object `d` is a data frame with 16 columns (variables)
> > - all columns are vectors;
> > - some are integer vectors, other are numeric vectors.
> >
> {: .solution}
{: .challenge}

`str` shows you a lot of information. You can access specific information with functions: `nrow()` (number of rows), `ncol()` (number of columns), and `dim()` (returns both):


~~~
nrow(d) 
~~~
{: .r}



~~~
[1] 59140
~~~
{: .output}



~~~
ncol(d)
~~~
{: .r}



~~~
[1] 16
~~~
{: .output}



~~~
dim(d)
~~~
{: .r}



~~~
[1] 59140    16
~~~
{: .output}

We can also print the columns of this dataframe using colnames() (there’s also a row.names() function):


~~~
colnames(d)
~~~
{: .r}



~~~
 [1] "start"           "end"             "total.SNPs"     
 [4] "total.Bases"     "depth"           "unique.SNPs"    
 [7] "dhSNPs"          "reference.Bases" "Theta"          
[10] "Pi"              "Heterozygosity"  "X.GC"           
[13] "Recombination"   "Divergence"      "Constraint"     
[16] "SNPs"           
~~~
{: .output}

Note that R’s read.csv() function has automatically renamed some of these columns for us: spaces have been converted to periods and the percent sign in %GC has been changed to an “X.” “X.GC” isn’t a very descriptive column name, so let’s change this:


~~~
colnames(d)[12] # original name
~~~
{: .r}



~~~
[1] "X.GC"
~~~
{: .output}



~~~
colnames(d)[12] <- "percent.GC"
colnames(d)[12] # new name
~~~
{: .r}



~~~
[1] "percent.GC"
~~~
{: .output}

Because the `dataframe$column` command returns a vector, we can pass it to R functions like mean() or summary() to get an idea of what depth looks like across this dataset:


~~~
mean(d$depth)
~~~
{: .r}



~~~
[1] 8.183938
~~~
{: .output}



~~~
summary(d$depth)
~~~
{: .r}



~~~
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
  1.000   6.970   8.170   8.184   9.400  21.910 
~~~
{: .output}

As we saw above, the dollar sign operator is a syntactic shortcut for a more general bracket operator 
used to access rows, columns, and cells of a dataframe.

> ## Selecting multiple rows and/or columns (reminder)
>
> 1. The indexes we use for dataframes can be vectors to select multiple rows and columns simultaneously. 
> 1. Omitting the row index retrieves all rows, and omitting the column index retrieves all columns.
>
> ~~~~~~~~~~
> d[,1:2]
> d[, c("start", "end")]
> d[1, c("start", "end")]
> d[1,]
> d[2, 3]
> ~~~~~~~~~~
>
> {: .source}
{: .callout}

When accessing a single column from a dataframe using [,], R’s default behavior is to return this as a vector—not 
a dataframe with one column. Sometimes this can cause problems if downstream code expects to work with 
a dataframe. To disable this behavior, we set the argument drop to FALSE in the bracket operator:


~~~
d[, "start", drop=FALSE]
~~~
{: .r}

Now, let’s add an additional column to our dataframe that indicates whether a window is in the centromere region. The positions of the chromosome 20 centromere (based on Giemsa banding) are 25,800,000 to 29,700,000 (see this chapter’s README on GitHub to see how these coordinates were found). We can append to our d dataframe a column called cent that has TRUE/FALSE values indicating whether the current window is fully within a centromeric region using comparison and logical operations:


~~~
d$cent <- d$start >= 25800000 & d$end <= 29700000
~~~
{: .r}

> ## Challenge 3
>
> How many windows fall into this centromeric region? 
>
> > ## Solutions to challenge 3
> >
> > We can use `table()` 
> > 
> > ~~~
> > table(d$cent)
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> > 
> > FALSE  TRUE 
> > 58455   685 
> > ~~~
> > {: .output}
> > or sum()
> > 
> > 
> > ~~~
> > sum(d$cent)
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> > [1] 685
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

In the dataset we are using, the diversity estimate Pi is measured per sampling window (1kb) and scaled up by 10x (see supplementary Text S1 for more details). It would be useful to have this scaled as per basepair nucleotide diversity (so as to make the scale more intuitive). Hence our next challenge.

> ## Challenge 4
>
> *Create a new rescaled column called diversity, in which the nucleotide diversity is calculated per basepair 
> 
> > ## Solution
> >
> > 
> > ~~~
> > d$diversity <- d$Pi / (10*1000) # rescale, removing 10x and making per bp
> > ~~~
> > {: .r}
> {: .solution}
>
> *Use the `summary()` function to calculate the basic statistics for the nucleotide diversity
>
> > ## Solution
> >
> > 
> > ~~~
> > summary(d$diversity)
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> >      Min.   1st Qu.    Median      Mean   3rd Qu.      Max. 
> > 0.0000000 0.0005577 0.0010420 0.0012390 0.0016880 0.0265300 
> > ~~~
> > {: .output}
> >
> {: .solution}
{: .challenge}

Finally, to make sure our analysis is reproducible, we should put the code
into a script file so we can come back to it later.

> ## Challenge 5
>
> Go to file -> new file -> R script, and write an R script
> to load in the dataset we used and to create additional columns. Put it in the `scripts/`
> directory and add it to version control.
>
> Run the script using the `source` function, using the file path
> as its argument (or by pressing the "source" button in RStudio).
>
> > ## Solution to Challenge 5
> > The contents of `script/first_script.R`:
> > 
> > ~~~
> > d <- read.csv("https://raw.githubusercontent.com/vsbuffalo/bds-files/master/chapter-08-r/Dataset_S1.txt")
> > d$cent <- d$start >= 25800000 & d$end <= 29700000
> > d$diversity <- d$Pi / (10*1000) # rescale, removing 10x and making per bp
> > ~~~
> > {: .r}
> {: .solution}
{: .challenge}

## Exploring Data Through Slicing and Dicing: Subsetting Dataframes

The most powerful feature of dataframes is the ability to slice out specific rows by 
applying the same vector subsetting techniques we used before. Combined with R’s 
comparison and logical operators, this leads to an incredibly powerful method to 
query out rows in a dataframe.

Let’s start by looking at the total number of SNPs per window. From `summary()`, 
we see that this varies quite considerably across all windows on chromosome 20:


~~~
summary(d$total.SNPs)
~~~
{: .r}



~~~
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
  0.000   3.000   7.000   8.906  12.000  93.000 
~~~
{: .output}

Notice how right-skewed this data is: the third quartile is 12 SNPs, but the maximum is 93 SNPs. 
Often we want to investigate such outliers more closely. Let’s use data subsetting to select 
out some rows that have 85 or more SNPs (the number is arbitrary). We can create a logical 
vector containing whether each observation (row) has 85 or more SNPs using the following:


~~~
d$total.SNPs >= 85
~~~
{: .r}
We can use this logical vector to extract the rows of our dataframe that have a TRUE value for d$total.SNPs >= 85:


~~~
d[d$total.SNPs >= 85, ]
~~~
{: .r}



~~~
         start      end total.SNPs total.Bases depth unique.SNPs dhSNPs
2567   2621001  2622000         93       11337 11.34          13     10
12968 13023001 13024000         88       11784 11.78          11      1
43165 47356001 47357000         87       12505 12.50           9      7
      reference.Bases  Theta     Pi Heterozygosity percent.GC
2567             1000 43.420 50.926         81.589    43.9439
12968            1000 33.413 19.030         74.838    28.8288
43165            1000 29.621 27.108         69.573    46.7467
      Recombination Divergence Constraint SNPs  cent diversity
2567    0.000706536 0.01701702          0    1 FALSE 0.0050926
12968   0.000082600 0.01401401          0    1 FALSE 0.0019030
43165   0.000500577 0.02002002          0    7 FALSE 0.0027108
~~~
{: .output}

We can build more elaborate queries by chaining comparison operators. For example, suppose we wanted to see all windows where Pi (nucleotide diversity) is greater than 16 and percent GC is greater than 80.We’d use:


~~~
d[d$Pi > 16 & d$percent.GC > 80, ]
~~~
{: .r}



~~~
         start      end total.SNPs total.Bases depth unique.SNPs dhSNPs
58550 63097001 63098000          5         947  2.39           2      1
58641 63188001 63189000          2        1623  3.21           2      0
58642 63189001 63190000          5        1395  1.89           3      2
      reference.Bases  Theta     Pi Heterozygosity percent.GC
58550             397 37.544 41.172         52.784    82.0821
58641             506 16.436 16.436         12.327    82.3824
58642             738 35.052 41.099         35.842    80.5806
      Recombination Divergence Constraint SNPs  cent diversity
58550   0.000781326 0.03826531        226    1 FALSE 0.0041172
58641   0.000347382 0.01678657        148    0 FALSE 0.0016436
58642   0.000347382 0.01793722          0    0 FALSE 0.0041099
~~~
{: .output}

> ## Discussion time!
> What we just did is really cool, so take a minute to talk to your neighbor to make sure that you/him/her 
> understand how these commands work. So here are some questions to discuss:
> * Why do we need to have d$ before column names?  
> * Do we need a comma? Why?
> * Do we need the space?
> * How can we print only some but not all columns?
> You can try ommitting some of these elements and checking the results
{: .discussion}

Remember, columns of a dataframe are just vectors. If you only need the data from one column, 
just subset it as you would a vector:


~~~
d$percent.GC[d$Pi > 16]
~~~
{: .r}

Subsetting columns can be a useful way to summarize data across two different conditions. 
For example, we might be curious if the average depth in a window (the depth column) 
differs between very high GC content windows (greater than 80%) and all other windows:


~~~
for (i in (seq(10, 100, 10))) {
  print(summary(d$Recombination[d$percent.GC <= i & d$percent.GC > i-10])*100)
}
~~~
{: .r}



~~~
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
0.02510 0.03560 0.04609 0.04609 0.05658 0.06708 
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
0.00202 0.01980 0.03402 0.13860 0.12550 0.68640 
    Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
0.000056 0.006720 0.018920 0.120600 0.047580 4.091000 
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
0.00003 0.00793 0.02665 0.15060 0.08182 8.83400 
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
0.00003 0.01314 0.04527 0.19980 0.14450 9.10900 
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
0.00003 0.02392 0.07810 0.26870 0.27140 7.36100 
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
0.00003 0.03826 0.11480 0.26020 0.33860 4.08200 
    Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
0.000236 0.022190 0.072110 0.202600 0.238400 1.675000 
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
0.02013 0.03474 0.07018 0.23350 0.28510 0.76800 
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
                                                
~~~
{: .output}



~~~
summary(d$depth[d$percent.GC >= 80])
~~~
{: .r}



~~~
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
   1.05    1.89    2.14    2.24    2.78    3.37 
~~~
{: .output}



~~~
summary(d$depth[d$percent.GC < 80])
~~~
{: .r}



~~~
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
  1.000   6.970   8.170   8.185   9.400  21.910 
~~~
{: .output}



~~~
summary(d$depth[d$percent.GC <= i])
~~~
{: .r}



~~~
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
  1.000   6.970   8.170   8.184   9.400  21.910 
~~~
{: .output}



~~~
sum(d$SNPs[d$start >= 37492472 & d$end <= 37527931])
~~~
{: .r}



~~~
[1] 24
~~~
{: .output}



~~~
d$SNPs
~~~
{: .r}



~~~
    [1] 0 0 0 0 0 0 0 0 1 1 0 0 0 0 1 0 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0
   [35] 0 0 1 0 1 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0
   [69] 1 1 0 0 0 0 0 0 2 1 2 3 2 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
  [103] 1 0 0 0 0 0 2 0 2 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0
  [137] 0 1 0 0 0 0 0 0 0 0 1 0 1 1 3 1 0 1 0 0 1 0 1 3 0 1 3 0 1 1 0 0 1 0
  [171] 0 0 0 0 0 0 2 2 2 2 1 2 0 0 0 0 1 0 1 0 1 1 1 0 0 1 5 1 2 2 0 0 0 0
  [205] 1 2 0 0 0 1 1 0 0 0 0 1 0 3 0 1 0 0 0 1 0 1 0 0 2 2 2 1 0 0 0 0 1 0
  [239] 0 0 0 0 1 1 1 0 0 0 0 0 2 2 0 0 0 0 1 0 0 0 1 0 0 0 1 0 0 0 0 0 0 0
  [273] 0 0 0 1 0 0 0 0 0 1 0 0 0 1 1 0 0 0 0 3 1 2 0 0 0 1 0 0 2 0 0 0 0 0
  [307] 0 0 0 0 1 0 1 0 0 0 1 2 1 0 1 1 1 1 1 2 0 0 0 0 0 1 0 0 0 0 0 0 0 0
  [341] 0 0 4 0 1 0 0 0 1 0 0 2 0 1 1 0 1 0 2 0 0 0 1 0 1 3 2 0 2 0 0 1 0 0
  [375] 0 1 0 0 0 0 1 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 2 0 0 0 1 0 0 0 4
  [409] 1 0 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 1 2 1 0 0 0 2 0 0 1 0 0 0 0 0
  [443] 0 0 0 0 0 0 0 0 0 1 1 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0
  [477] 1 0 1 3 0 1 0 2 0 0 0 0 0 0 1 0 0 0 0 1 1 0 0 0 0 1 1 0 1 1 0 1 0 0
  [511] 0 0 1 1 0 0 0 0 0 0 0 0 0 1 0 0 1 0 1 0 0 0 2 0 0 0 0 0 2 0 1 1 0 1
  [545] 1 2 1 0 0 0 2 0 0 0 0 0 0 0 0 1 0 0 0 0 0 1 2 0 0 0 0 1 1 0 0 1 0 0
  [579] 0 0 0 1 0 2 0 1 0 0 1 0 0 1 1 0 0 0 0 0 0 2 1 1 0 0 0 1 0 0 0 1 0 0
  [613] 0 1 0 1 0 1 1 0 0 1 2 0 0 0 0 0 0 0 0 4 2 0 0 0 0 1 0 1 0 0 1 0 2 0
  [647] 1 2 0 1 0 0 0 1 1 2 0 2 1 2 2 0 0 2 0 0 0 0 0 0 0 0 0 1 0 0 0 1 0 1
  [681] 2 0 2 4 1 1 0 0 1 0 0 0 3 3 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
  [715] 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 1 0 1 0 1 2 0 1 1 1
  [749] 0 3 0 1 0 1 0 0 0 0 1 0 0 0 0 1 1 0 2 0 2 0 0 0 1 0 0 0 3 1 1 1 2 0
  [783] 0 0 1 1 0 3 1 1 0 1 0 1 3 0 0 1 0 1 0 0 0 1 4 0 0 0 0 0 1 2 1 2 0 0
  [817] 1 1 0 0 0 0 1 2 1 0 2 2 1 0 1 2 1 0 2 1 0 2 0 0 1 1 1 0 1 0 1 0 0 0
  [851] 0 1 2 1 0 2 0 0 0 1 2 1 0 0 0 0 4 2 1 0 1 1 2 0 0 0 2 0 2 3 1 1 1 1
  [885] 0 0 2 0 1 1 0 1 0 3 1 0 0 0 0 2 0 1 0 0 1 0 1 1 0 0 0 1 1 2 0 1 1 0
  [919] 0 0 0 1 0 0 0 2 1 0 0 0 0 0 1 0 0 1 1 0 1 2 0 2 1 1 0 0 0 0 0 4 0 0
  [953] 0 0 0 0 0 0 1 0 2 0 2 0 0 0 0 1 2 0 1 1 0 0 1 0 0 0 2 2 1 2 0 2 0 1
  [987] 1 0 1 0 0 2 0 1 1 0 0 0 0 1 1 1 2 1 0 0 2 1 1 1 0 0 0 0 0 0 0 1 0 1
 [1021] 0 0 0 0 0 0 2 0 0 2 1 0 0 0 2 0 1 0 1 3 1 0 1 0 3 0 0 0 1 0 0 2 1 0
 [1055] 0 0 0 0 0 0 0 0 0 0 0 1 2 0 1 0 0 0 0 0 1 1 0 0 0 0 1 0 0 0 0 0 1 2
 [1089] 2 1 0 1 1 1 0 0 0 0 1 1 0 2 0 0 1 0 1 0 0 0 0 1 0 0 0 0 1 2 0 0 1 0
 [1123] 0 1 1 2 1 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 1 1 0 0 1 0
 [1157] 0 1 0 0 0 1 0 0 0 0 1 0 1 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0
 [1191] 2 0 0 1 0 1 0 0 0 0 1 1 1 0 1 0 1 0 1 0 1 1 1 1 0 0 0 1 1 0 1 1 1 1
 [1225] 0 0 0 1 0 2 2 2 3 3 0 0 0 0 4 1 1 1 0 0 1 1 1 1 0 0 0 1 1 1 0 0 0 0
 [1259] 0 0 0 0 2 2 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1 0 0 1 1 0 1 0 0 2 2 0
 [1293] 0 0 0 0 1 1 2 0 2 0 0 2 1 1 0 0 1 3 0 1 0 0 0 0 2 0 0 0 0 0 0 2 2 0
 [1327] 1 1 3 0 0 0 0 1 0 0 1 1 1 1 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 2 0 0 1
 [1361] 1 1 1 1 0 1 3 0 1 0 1 0 0 0 0 0 0 0 0 0 0 1 1 0 0 1 0 0 0 0 0 1 0 1
 [1395] 0 1 0 2 1 1 1 0 2 0 0 0 0 0 0 0 3 2 1 0 2 1 0 0 0 0 0 0 0 0 0 3 2 1
 [1429] 0 1 2 0 0 0 2 1 1 0 0 1 0 3 3 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 4 0 1 0
 [1463] 0 0 0 0 0 0 0 0 0 0 1 2 2 0 0 0 2 2 0 1 1 0 0 0 1 0 0 0 0 2 1 1 1 0
 [1497] 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 1 0
 [1531] 0 0 0 0 0 0 0 0 0 0 1 0 2 0 0 0 0 0 0 1 3 0 2 0 1 1 2 0 1 0 0 0 0 0
 [1565] 0 0 1 0 1 1 1 0 0 1 0 0 0 0 1 1 0 1 1 0 0 2 1 0 0 1 0 1 0 1 0 0 0 1
 [1599] 1 2 1 0 1 0 0 1 0 0 2 0 0 0 0 0 0 1 1 0 1 0 0 0 0 1 0 0 0 0 2 4 2 0
 [1633] 0 0 0 0 0 0 0 0 0 1 1 1 0 0 0 1 0 0 0 0 0 0 0 1 0 0 0 0 0 1 1 0 0 0
 [1667] 2 2 1 1 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 1 1 2 1
 [1701] 1 0 2 0 1 0 0 1 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 2 1 0 2 1 0
 [1735] 0 0 1 0 0 0 0 0 0 0 1 0 0 0 1 0 0 0 1 0 0 0 1 0 2 0 0 0 0 1 0 1 0 1
 [1769] 0 0 0 0 1 0 2 1 3 0 1 0 1 1 0 1 1 0 1 0 0 0 0 0 1 0 0 0 0 0 2 0 0 0
 [1803] 0 1 1 0 0 3 1 0 1 0 1 1 1 0 0 2 0 2 0 1 0 0 1 1 1 1 0 3 3 1 0 2 2 0
 [1837] 0 1 0 1 2 1 0 0 0 0 0 1 0 1 1 0 1 2 1 2 1 3 0 1 0 1 3 0 1 1 1 2 0 1
 [1871] 2 0 1 0 2 0 0 1 0 0 0 1 1 1 1 1 3 2 0 1 0 0 0 0 1 0 1 0 0 1 1 3 0 1
 [1905] 0 0 0 1 0 1 2 1 1 0 2 1 0 0 1 0 0 0 1 0 0 1 0 0 2 3 0 0 0 0 0 0 0 0
 [1939] 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0
 [1973] 0 0 0 0 0 0 0 0 1 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 3 0 0 0 0 0 0
 [2007] 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 3 1 0 1 2 1 0 0 2 0 1 0 1 0 0 0
 [2041] 0 0 0 2 0 1 1 0 2 1 0 0 0 0 0 0 1 2 1 0 0 2 1 1 1 1 1 1 0 1 3 3 0 1
 [2075] 0 1 0 0 0 0 0 0 1 0 2 0 2 0 0 2 1 0 1 1 0 2 0 1 0 1 1 1 3 3 2 0 1 0
 [2109] 0 0 0 1 0 0 0 0 0 1 1 0 1 1 0 1 1 2 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0
 [2143] 0 0 2 0 0 0 0 1 0 2 0 1 1 0 0 3 0 0 0 0 0 1 0 0 0 1 0 1 1 1 1 0 0 0
 [2177] 2 0 0 1 1 0 0 0 1 0 0 0 0 0 0 0 0 0 1 0 0 0 3 2 0 0 0 1 0 0 0 0 1 0
 [2211] 3 2 1 1 1 3 0 0 3 2 2 1 1 0 3 0 3 1 1 0 0 1 2 0 0 0 0 0 0 1 1 1 2 0
 [2245] 0 0 0 0 1 0 2 1 0 2 0 2 1 2 1 2 0 2 1 0 0 0 0 2 1 0 2 0 1 0 0 0 0 1
 [2279] 2 2 0 0 0 0 0 0 0 1 0 0 0 1 1 2 3 3 0 2 1 1 2 2 1 0 1 1 0 0 1 1 0 0
 [2313] 3 0 0 1 1 1 0 0 1 0 0 0 2 0 0 1 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0
 [2347] 0 0 0 0 1 0 0 3 0 1 1 0 0 0 0 1 0 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0
 [2381] 0 0 1 2 1 0 0 0 2 0 1 0 0 0 0 0 0 1 2 1 0 0 1 1 0 0 0 0 0 0 0 1 0 0
 [2415] 1 0 1 0 0 1 0 2 0 0 0 0 0 0 2 0 0 1 0 0 0 1 1 0 0 1 0 1 2 0 0 0 0 0
 [2449] 1 1 1 0 0 0 1 0 0 4 2 3 3 0 0 0 0 0 0 2 1 0 0 0 0 0 0 0 0 0 0 1 2 1
 [2483] 0 1 0 1 2 0 0 3 1 0 3 0 0 2 2 0 0 2 1 0 5 0 0 1 0 0 0 2 1 0 1 0 1 0
 [2517] 0 0 0 2 2 0 0 0 0 0 0 0 0 2 2 3 0 2 1 0 1 1 1 1 1 1 0 0 0 1 0 0 0 0
 [2551] 0 0 0 0 1 3 3 4 3 0 1 2 2 4 1 3 1 3 0 0 2 3 3 0 1 1 1 1 1 1 1 0 0 0
 [2585] 3 3 1 2 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 1 0 1 2 1 3
 [2619] 0 0 2 2 0 0 1 2 1 0 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0
 [2653] 2 0 0 0 2 1 0 0 1 1 1 1 0 0 0 0 1 0 1 1 0 1 1 1 0 0 0 2 1 1 3 0 2 0
 [2687] 0 0 0 0 0 1 0 0 0 2 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 2 0 0 1 0 0 0 3
 [2721] 1 0 0 0 0 1 0 0 1 0 0 0 0 0 0 1 2 0 2 1 0 1 0 0 0 0 0 0 0 0 0 0 1 0
 [2755] 0 1 0 1 2 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 2 0 0 1 0 1 1 0 0 2 0
 [2789] 0 1 0 1 0 0 0 1 0 0 0 0 0 0 0 1 0 0 0 0 0 2 1 0 0 2 3 0 0 0 0 0 0 0
 [2823] 2 1 0 0 1 0 0 1 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 3 1 0 0 0 0
 [2857] 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
 [2891] 0 0 0 0 0 0 0 0 0 0 0 1 1 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 1 0 0 0 0 0
 [2925] 0 0 0 0 0 0 0 0 1 0 1 0 2 0 1 0 0 1 0 1 0 1 0 1 1 1 0 1 0 0 0 0 0 1
 [2959] 1 0 2 1 1 2 1 1 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0
 [2993] 0 2 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 1 0 0 0 0 0 3 0 0 0 0
 [3027] 0 0 1 0 2 2 1 0 2 0 0 0 0 0 0 0 0 1 1 0 0 1 2 0 0 0 0 1 0 0 0 0 0 2
 [3061] 0 0 0 0 0 1 0 0 0 0 1 2 0 0 0 0 1 1 1 0 0 1 0 1 0 0 0 0 0 0 1 1 1 0
 [3095] 0 2 1 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 1 0 3 1 0 0 0 1 0 0 1 0 0 0 0
 [3129] 0 0 1 2 0 3 0 0 0 0 0 2 0 1 0 1 2 0 0 1 0 0 0 1 0 0 2 0 1 2 1 0 0 0
 [3163] 0 1 2 0 0 0 1 0 0 3 2 2 0 1 0 2 0 1 0 1 0 0 1 0 0 0 0 1 0 3 0 0 0 1
 [3197] 0 3 2 0 2 0 1 1 2 2 1 4 0 0 0 1 0 0 2 1 1 2 0 0 0 0 0 0 2 0 2 1 0 0
 [3231] 0 0 0 1 1 0 3 0 0 2 0 0 0 0 1 2 0 0 0 0 0 0 0 1 3 1 0 0 0 0 1 1 0 0
 [3265] 0 0 0 0 2 0 2 0 1 0 3 1 0 0 0 0 0 0 1 0 0 0 0 0 0 0 1 1 1 0 0 2 0 0
 [3299] 0 4 0 1 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 2 0 0 0 1 1 1 0 1 0 0
 [3333] 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 1 0 1 1 0 0 0 0 0 0 0 0
 [3367] 0 0 0 0 0 2 1 0 0 1 1 2 2 1 1 0 0 0 0 0 0 0 1 0 1 0 2 0 1 1 1 1 0 1
 [3401] 0 0 1 1 0 1 0 0 0 0 2 2 0 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0
 [3435] 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 0 2 0 1 0 0 0 0 1
 [3469] 2 0 1 0 0 0 0 0 3 2 1 1 0 1 0 0 2 1 0 2 1 0 0 0 0 2 1 0 1 0 1 0 2 2
 [3503] 1 0 2 1 0 0 0 0 0 0 0 2 1 2 1 1 0 0 0 0 1 2 0 1 0 1 0 0 0 0 0 0 2 1
 [3537] 1 4 0 1 1 2 3 1 1 3 1 0 0 3 0 1 0 0 1 0 1 0 1 1 1 0 0 0 1 1 3 0 2 3
 [3571] 3 2 2 1 0 3 0 0 3 1 2 0 0 2 0 0 0 0 1 3 0 2 1 1 0 1 0 0 1 1 0 1 2 0
 [3605] 0 0 2 2 1 1 0 1 1 1 0 2 1 1 0 0 0 2 2 0 4 1 0 0 0 0 0 0 0 0 0 0 0 1
 [3639] 1 0 0 2 0 0 0 0 0 0 0 0 0 1 0 0 0 1 1 1 0 1 1 0 0 1 1 0 0 1 0 0 1 1
 [3673] 0 1 1 1 1 1 0 0 0 0 0 0 1 1 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0
 [3707] 0 0 0 0 0 1 1 0 0 0 2 2 1 0 0 0 0 1 2 1 2 3 1 0 0 0 2 0 1 0 0 0 0 2
 [3741] 1 0 0 1 1 0 1 1 0 0 0 0 0 0 0 1 0 1 0 0 1 0 0 0 0 0 0 0 0 0 0 0 1 0
 [3775] 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 1 0 0 0 0 0 2 0 0 0 0 0 1 0 0 0 0 0
 [3809] 0 0 1 0 0 0 0 0 0 0 1 0 0 0 0 0 0 2 0 0 0 1 0 1 0 0 0 0 0 1 0 0 0 0
 [3843] 0 0 1 0 0 0 0 0 0 1 2 1 2 1 0 3 0 0 0 1 0 0 1 2 2 0 0 0 0 0 0 0 0 0
 [3877] 0 0 0 0 1 0 0 0 0 2 1 2 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 2 0 0 0 0
 [3911] 0 0 1 1 0 2 0 0 0 0 0 0 0 1 1 1 2 1 0 0 0 0 0 2 1 0 0 0 1 0 1 1 0 0
 [3945] 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 2 2 0 0 0 0 0 0 2 2 0 0 1 1
 [3979] 5 1 2 1 1 0 2 2 3 3 2 4 0 0 1 3 3 3 2 0 0 0 0 0 0 0 1 3 0 1 0 3 0 2
 [4013] 1 2 0 2 0 0 0 0 0 1 0 0 0 0 0 1 0 1 1 0 1 1 0 1 0 0 1 0 0 2 0 0 2 0
 [4047] 0 0 0 2 1 2 0 0 0 0 1 1 0 0 0 1 1 0 0 0 0 1 1 2 0 0 1 1 1 2 3 2 1 0
 [4081] 2 0 0 2 1 0 0 2 0 1 2 0 1 0 0 1 1 1 1 1 0 0 2 4 1 1 0 1 0 2 1 2 1 1
 [4115] 1 1 0 0 2 0 0 2 0 0 0 0 0 0 0 0 0 1 1 0 0 1 1 0 1 0 1 2 2 0 0 0 2 0
 [4149] 1 2 0 0 0 1 0 1 0 1 0 1 0 0 1 1 1 0 0 0 0 0 0 1 0 1 2 2 1 4 2 2 0 0
 [4183] 0 2 0 0 0 1 0 0 0 1 0 0 0 2 0 1 1 3 1 1 1 0 0 0 0 0 0 0 1 1 0 0 0 1
 [4217] 1 0 0 0 1 2 0 0 1 4 1 1 0 1 0 0 0 0 0 3 0 0 0 0 0 1 0 3 2 0 2 1 2 0
 [4251] 1 0 3 2 1 4 1 0 1 0 1 1 1 0 1 0 0 0 0 0 0 0 0 0 4 2 0 1 0 1 2 0 0 0
 [4285] 0 0 1 1 5 1 0 3 6 2 0 0 0 0 0 0 0 0 2 0 1 0 0 0 0 0 0 0 0 3 0 2 0 0
 [4319] 0 2 1 2 2 3 1 4 1 1 1 3 2 0 1 4 2 1 1 0 0 0 0 0 2 2 2 1 0 2 1 0 0 0
 [4353] 3 1 1 2 1 1 3 0 1 1 1 0 0 1 2 3 0 0 2 0 0 1 0 0 0 0 0 1 0 2 1 0 2 2
 [4387] 0 0 2 2 2 0 1 1 1 0 0 0 0 0 3 2 0 0 0 0 0 0 2 2 1 3 0 2 0 0 0 0 0 0
 [4421] 0 0 0 1 2 0 0 1 2 3 0 2 0 1 0 1 2 0 0 2 1 0 0 0 0 0 0 0 0 0 0 0 0 2
 [4455] 0 0 0 0 0 0 1 2 1 0 0 0 0 0 0 2 2 0 0 0 2 1 0 0 1 0 0 1 0 1 2 0 0 0
 [4489] 0 2 1 0 3 0 0 0 0 0 1 2 3 0 1 1 0 1 3 1 0 1 1 1 3 1 0 2 2 0 0 0 0 0
 [4523] 1 3 0 0 0 1 1 0 1 2 1 1 2 2 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 2 0 0 1 1
 [4557] 0 2 1 2 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 1 0 2 0 0 0 0 0 0 0 0 0 1 0 1
 [4591] 0 0 0 0 0 2 0 2 0 0 0 1 0 0 0 1 0 0 1 1 1 1 2 0 0 1 0 2 0 0 0 0 0 0
 [4625] 1 2 1 1 0 2 2 1 0 1 5 1 0 6 3 0 2 2 3 4 1 2 1 2 2 2 1 0 0 1 1 1 3 2
 [4659] 1 3 6 1 2 0 0 1 1 2 0 2 1 1 1 0 0 0 0 0 0 0 0 2 1 0 2 1 2 1 3 1 0 1
 [4693] 0 0 0 0 1 1 0 2 0 1 1 1 2 1 1 1 2 2 1 0 0 1 2 1 3 3 3 1 1 0 1 0 0 1
 [4727] 2 1 1 0 2 1 2 2 1 5 0 1 2 1 1 1 2 1 0 3 1 1 0 0 0 0 0 1 0 1 0 1 2 1
 [4761] 1 2 0 0 0 0 1 1 0 0 0 0 0 1 1 0 0 0 0 0 1 0 1 2 0 1 0 1 0 0 0 2 0 2
 [4795] 2 1 1 0 0 0 0 0 0 0 2 0 1 1 0 1 3 2 0 0 0 1 1 0 2 2 1 1 1 1 0 0 0 1
 [4829] 0 0 0 2 0 0 0 1 1 0 1 0 1 1 0 1 0 0 2 0 1 1 2 0 0 0 1 0 0 2 0 0 1 0
 [4863] 0 1 0 0 1 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 2 2 0 1 0 1 2 0 0 0 1 1 0 0
 [4897] 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 1 1 0 1 0 1 2 0 1 0 2 0 1 0 0 0
 [4931] 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 1
 [4965] 3 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 2 0 0 0 0 0 0 2 0 0 0 0
 [4999] 0 0 0 1 0 0 1 0 1 1 0 1 0 0 2 0 0 1 2 2 0 0 0 1 0 0 0 0 1 0 0 0 0 0
 [5033] 1 0 0 3 0 0 0 2 0 0 0 0 0 0 0 0 0 2 0 1 0 0 1 2 1 0 0 0 0 1 0 0 0 1
 [5067] 1 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 1 1 0 0 2 2 0 0 1 0 0 0 0 1 3
 [5101] 1 0 0 1 0 1 0 0 0 0 0 0 2 0 3 1 1 0 1 3 0 0 0 0 1 0 2 0 1 1 0 0 1 2
 [5135] 2 2 0 0 1 0 0 0 0 2 0 0 0 0 0 0 0 2 1 0 0 0 1 0 0 0 0 0 0 2 2 0 0 0
 [5169] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 2 0 0 1 1 2 2 0 1 1 5 2 0 0 1 2 1
 [5203] 1 0 0 0 0 0 0 0 2 2 1 2 3 0 1 1 1 3 1 1 2 3 1 0 0 0 0 1 1 1 0 3 0 2
 [5237] 1 3 5 4 1 0 2 2 1 1 1 0 1 1 0 0 0 1 0 0 1 1 0 0 0 0 0 0 0 2 1 1 1 2
 [5271] 1 1 1 0 2 1 0 2 3 0 0 2 1 3 1 0 0 0 0 0 0 0 1 0 1 1 0 0 0 1 0 0 0 0
 [5305] 0 0 0 0 0 0 0 0 0 0 1 0 1 3 1 3 1 2 0 0 0 0 1 2 1 1 0 3 0 1 1 0 0 1
 [5339] 0 0 0 0 0 1 1 0 1 2 0 0 0 1 3 1 2 1 0 0 0 0 0 0 1 0 0 3 0 1 0 0 0 0
 [5373] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 2 2 4
 [5407] 2 4 2 0 0 0 1 0 2 2 2 1 3 2 2 0 0 0 1 0 0 0 1 0 2 0 1 1 0 1 0 0 0 0
 [5441] 0 0 0 0 0 1 2 1 0 1 1 1 0 0 0 1 0 0 0 0 1 0 0 0 1 1 3 0 1 0 0 0 0 1
 [5475] 0 0 0 0 0 1 0 1 4 1 2 0 0 0 1 0 1 2 2 0 3 1 0 1 1 0 1 0 1 0 0 1 1 0
 [5509] 1 1 0 1 0 1 1 0 0 1 0 0 0 0 2 1 1 0 0 1 0 1 1 2 0 0 0 0 0 2 1 0 0 1
 [5543] 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 1 0 1 1 0 0 0 1 2 0 1 0 0 1 0 3 0 0
 [5577] 1 0 2 0 0 0 0 0 0 0 0 1 0 1 1 0 0 0 1 0 0 0 0 1 0 1 0 0 1 0 0 2 2 1
 [5611] 0 0 0 0 0 1 2 1 0 0 0 0 0 0 2 0 0 0 0 0 1 1 0 0 0 2 1 1 1 2 1 2 0 0
 [5645] 1 1 0 1 0 0 1 1 0 0 1 1 1 1 1 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0
 [5679] 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 1 0 0 1 0 0 0 1 2 3 0 2 1 0 0 2 0 0 0
 [5713] 0 0 0 2 0 0 2 0 2 1 1 1 2 4 1 3 0 0 0 1 2 0 1 1 0 0 1 1 0 1 0 0 0 0
 [5747] 0 0 4 1 3 0 1 1 0 0 1 0 0 3 2 1 1 0 1 0 0 2 0 0 0 0 0 0 0 0 0 1 1 3
 [5781] 4 3 2 2 2 3 4 0 2 0 0 2 0 0 0 0 0 0 1 0 0 1 1 0 2 0 1 1 2 1 1 0 0 1
 [5815] 3 0 1 0 0 0 0 0 2 0 0 1 1 2 1 1 2 1 0 0 0 1 1 2 0 2 2 0 1 6 0 3 0 0
 [5849] 2 5 1 1 0 0 0 0 1 1 2 0 0 0 1 0 0 1 0 1 1 1 0 1 0 1 1 0 1 0 0 1 0 0
 [5883] 1 0 0 0 0 1 0 1 0 0 0 1 1 0 0 0 0 0 0 3 0 1 0 1 0 0 0 0 0 0 0 3 0 0
 [5917] 0 1 0 1 0 1 1 1 1 0 1 2 2 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 2
 [5951] 1 0 0 1 0 1 1 0 1 1 1 0 0 0 0 1 0 3 0 3 0 0 0 1 1 0 0 0 0 2 1 0 1 0
 [5985] 0 0 1 1 0 2 0 0 1 0 0 1 3 0 2 0 1 0 1 2 4 1 0 0 1 1 2 0 1 0 2 3 0 0
 [6019] 2 0 0 0 0 0 0 0 1 2 0 2 1 0 1 2 1 1 1 1 0 0 1 0 1 0 1 0 0 1 1 1 0 1
 [6053] 0 0 1 0 0 0 2 0 0 0 2 1 0 0 0 0 1 1 1 0 0 0 0 0 0 0 1 1 0 0 0 1 1 1
 [6087] 1 0 0 0 1 1 0 0 0 0 1 0 1 0 1 0 2 0 0 0 1 0 0 1 0 0 0 0 2 1 0 0 0 0
 [6121] 1 3 1 1 0 0 0 0 2 0 1 0 1 1 2 1 0 0 0 0 0 1 0 0 0 0 0 0 1 1 1 0 0 0
 [6155] 2 0 0 1 0 0 1 0 1 2 0 0 1 0 0 0 1 1 0 0 1 0 0 0 2 1 1 0 0 5 1 5 1 4
 [6189] 2 2 3 1 1 1 1 0 0 1 1 1 0 0 2 0 0 0 0 1 0 0 0 0 0 0 1 0 0 1 1 0 0 1
 [6223] 2 2 1 0 1 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 1
 [6257] 2 1 0 0 2 1 2 0 0 0 2 2 1 1 0 2 0 0 2 2 3 0 0 2 0 0 0 0 0 0 0 0 0 1
 [6291] 1 1 0 1 2 2 0 3 1 0 1 1 3 2 4 2 0 0 1 0 2 1 0 0 0 1 0 5 5 1 0 1 0 3
 [6325] 0 0 0 0 0 0 0 0 0 0 0 0 2 0 2 1 0 1 2 1 0 0 2 3 0 0 1 3 0 2 1 0 1 1
 [6359] 1 0 3 1 3 2 1 0 0 4 2 0 1 3 3 4 3 4 0 0 0 0 0 0 0 0 0 1 0 1 0 3 0 0
 [6393] 1 2 2 1 1 2 0 3 2 1 3 2 1 1 4 3 0 1 3 0 3 2 1 1 0 1 1 1 0 0 0 1 1 1
 [6427] 1 4 0 1 3 0 0 0 0 3 2 0 2 3 5 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 3
 [6461] 0 0 1 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 2 0 0 1 1 4 0 0 0 0 0 0
 [6495] 0 1 0 1 1 0 1 0 1 1 1 2 0 0 3 0 0 0 2 0 2 5 2 2 1 1 2 1 1 0 3 1 0 0
 [6529] 1 0 0 2 1 1 2 0 0 0 0 0 0 0 0 2 0 0 0 0 2 0 1 1 2 1 0 0 1 3 0 0 0 1
 [6563] 0 2 1 0 4 0 0 0 0 0 1 0 3 1 0 1 2 0 0 2 0 2 0 1 0 0 1 0 3 0 1 0 1 2
 [6597] 2 0 0 1 1 0 1 2 0 0 0 1 2 3 4 1 1 1 0 0 0 1 0 0 0 2 1 2 0 1 1 0 0 1
 [6631] 3 1 2 0 0 0 1 0 1 0 0 1 0 1 0 0 0 0 2 0 1 2 0 0 1 1 2 0 1 0 0 0 1 0
 [6665] 0 2 0 1 3 4 0 0 1 0 0 1 2 2 2 2 2 1 0 2 0 0 1 2 0 0 0 1 1 0 1 1 1 2
 [6699] 0 1 3 2 3 0 0 0 1 2 0 3 0 0 1 0 0 0 0 0 0 1 0 2 4 1 0 0 0 1 0 0 0 0
 [6733] 2 1 1 0 2 0 1 0 0 1 1 0 0 0 1 0 2 3 1 0 0 2 1 0 0 0 0 3 0 0 1 1 1 0
 [6767] 1 0 0 0 1 0 2 0 1 2 1 1 2 3 0 1 0 1 2 0 0 0 1 1 0 0 1 2 1 1 1 2 1 1
 [6801] 2 0 2 1 0 1 2 1 0 1 1 0 0 1 0 0 0 0 1 0 2 1 0 0 0 0 1 0 0 0 1 0 0 1
 [6835] 0 0 1 0 0 0 0 4 2 1 0 1 2 0 0 1 1 1 3 2 0 0 1 0 1 1 3 2 1 4 3 6 0 3
 [6869] 0 1 1 0 0 0 1 0 3 0 0 2 2 1 2 1 1 1 1 0 2 1 0 0 2 0 1 2 1 0 3 1 1 0
 [6903] 0 2 1 0 0 0 3 0 0 2 0 1 0 0 0 0 2 0 0 1 0 1 1 0 0 0 0 0 1 0 2 0 3 2
 [6937] 0 2 1 0 1 0 0 0 0 0 3 0 0 0 0 3 2 0 2 0 1 0 0 0 0 0 0 0 0 0 2 2 1 0
 [6971] 1 0 0 0 0 1 0 2 0 0 0 3 0 1 0 2 3 1 1 1 1 0 1 0 0 0 0 0 0 0 0 0 1 3
 [7005] 1 0 0 3 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 3 1 3 2 1 0
 [7039] 0 0 0 1 2 1 0 2 1 1 0 1 0 2 1 2 1 0 0 2 1 0 2 0 0 0 0 0 1 0 1 0 1 1
 [7073] 1 1 0 3 1 0 0 0 0 1 1 0 1 0 0 1 1 1 1 1 0 1 0 0 0 0 1 1 0 0 0 2 0 0
 [7107] 0 2 3 2 0 1 0 0 0 0 1 0 0 2 2 0 1 2 0 0 1 0 0 0 4 1 0 0 3 1 0 0 0 0
 [7141] 0 0 0 0 0 0 2 4 2 0 1 0 0 0 1 1 0 3 0 2 1 1 0 0 0 1 1 0 2 0 0 2 2 1
 [7175] 1 1 0 3 3 0 0 0 2 0 2 0 1 0 0 2 0 1 1 3 4 1 0 0 2 1 2 0 2 0 3 1 0 0
 [7209] 0 0 0 2 1 1 2 1 0 1 0 0 0 4 1 1 0 0 0 0 2 0 0 1 0 4 2 1 1 0 0 0 0 0
 [7243] 0 0 2 2 1 0 0 0 1 1 3 0 1 1 2 0 1 0 0 0 0 0 0 0 0 1 0 1 2 0 2 1 4 2
 [7277] 2 2 1 1 0 1 3 0 0 0 0 0 2 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
 [7311] 1 2 1 3 1 3 1 1 2 2 3 0 0 0 2 0 1 1 0 1 1 0 0 0 1 1 1 1 1 0 0 0 0 2
 [7345] 0 3 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 2 1 0 0 0 0 0 0 0 0 1 0 0 0 0
 [7379] 0 0 1 0 1 0 0 0 0 0 0 0 0 0 3 0 0 1 1 2 2 4 0 3 2 5 4 0 0 0 1 1 2 0
 [7413] 0 0 0 0 1 0 1 1 0 1 0 1 0 0 0 1 0 1 0 1 0 0 0 3 3 0 1 1 1 0 0 0 0 0
 [7447] 0 0 0 0 0 3 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 3 2 1 0 0 0 0 0 0 0
 [7481] 0 0 0 0 1 0 1 1 0 0 2 3 0 0 0 0 0 0 0 0 0 0 0 1 1 2 2 0 1 1 4 1 4 3
 [7515] 0 1 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 0 0 0 0 0 1 1 0 0 2 1 0 1 1
 [7549] 1 0 0 0 0 0 0 0 2 1 0 1 0 0 1 2 3 1 1 1 0 1 1 1 2 3 2 0 0 1 1 1 1 1
 [7583] 0 2 2 0 0 2 3 1 4 0 4 4 1 0 0 0 1 0 0 1 1 1 2 1 0 0 0 1 2 0 1 1 1 0
 [7617] 1 2 2 4 1 1 1 0 1 0 3 1 1 1 1 4 1 1 1 2 1 0 2 2 1 0 0 1 0 0 0 1 1 0
 [7651] 0 0 1 0 2 1 0 0 0 1 0 0 1 1 0 0 0 0 0 0 0 0 0 1 0 1 2 0 2 0 0 0 0 0
 [7685] 0 0 0 2 3 0 0 1 0 0 0 2 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 1 1 2 1 0 2 1
 [7719] 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 1 0 1 0 0 2 1 2 0 0 3 0 2
 [7753] 0 0 1 1 1 1 2 0 3 0 1 0 1 0 1 0 1 3 1 4 0 2 5 1 1 1 2 1 0 4 1 0 4 0
 [7787] 0 3 1 2 1 0 2 0 0 0 0 0 0 2 1 0 2 0 0 0 0 0 0 1 0 0 1 0 0 1 0 0 0 0
 [7821] 0 0 0 0 1 0 1 0 1 2 0 1 3 3 0 0 2 1 0 1 0 0 3 2 5 0 0 2 1 0 1 0 1 0
 [7855] 0 1 1 2 2 2 1 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1 0 0 0 0 0 1
 [7889] 1 0 3 0 1 2 1 1 1 0 1 0 1 1 1 0 0 2 0 0 0 0 0 0 0 2 0 0 0 0 1 1 0 0
 [7923] 0 0 0 0 0 0 0 0 0 1 1 1 1 0 0 1 2 0 0 0 0 1 2 1 1 0 0 0 0 1 1 0 0 2
 [7957] 1 1 1 0 1 2 0 0 0 0 0 3 0 0 0 1 0 0 1 1 2 0 0 1 1 2 3 1 0 0 1 1 0 0
 [7991] 2 3 4 0 0 1 0 1 1 0 0 1 0 1 0 0 0 0 0 0 2 1 1 3 3 2 1 1 1 1 0 0 0 0
 [8025] 0 0 1 0 0 0 0 0 0 0 0 3 1 0 3 0 0 1 1 0 1 2 1 1 0 1 1 1 0 2 0 1 0 2
 [8059] 0 2 1 1 1 2 2 0 0 3 1 2 0 0 1 1 1 0 0 2 0 0 0 0 0 2 0 0 0 0 1 2 1 0
 [8093] 0 0 0 0 1 0 0 0 0 1 2 2 0 0 2 1 2 0 4 1 3 1 0 0 0 0 3 1 1 1 0 1 0 0
 [8127] 0 0 0 1 0 1 4 0 0 0 1 1 0 2 0 0 1 1 1 2 0 2 1 1 2 1 2 1 0 2 0 1 1 1
 [8161] 2 2 1 0 0 1 0 0 0 0 2 0 3 0 1 0 0 1 2 1 1 3 2 0 0 2 1 0 0 2 0 0 3 2
 [8195] 0 0 0 1 0 0 1 2 0 1 1 1 2 4 3 1 0 0 0 0 0 1 2 0 2 0 1 0 1 0 0 1 0 0
 [8229] 0 0 0 0 0 1 1 0 2 1 2 0 0 1 2 0 1 1 0 2 0 2 2 3 0 0 1 1 2 1 1 1 0 0
 [8263] 1 1 5 1 0 0 0 1 0 0 0 0 1 1 0 1 0 1 0 0 0 0 0 0 1 1 0 0 2 0 0 0 2 0
 [8297] 1 1 0 0 0 0 0 0 0 0 0 0 1 2 1 0 0 1 0 1 0 0 0 0 0 0 2 1 0 0 1 0 1 0
 [8331] 0 2 0 2 1 2 1 0 1 2 1 2 0 1 0 1 1 0 2 2 0 0 0 1 2 0 0 0 2 1 0 1 1 0
 [8365] 0 0 1 1 2 1 0 1 2 0 0 0 1 0 1 0 0 1 1 1 0 1 0 0 0 1 0 1 1 2 0 0 0 1
 [8399] 0 1 0 1 4 1 0 0 1 1 2 1 0 0 4 2 2 0 0 0 2 0 2 0 0 1 0 0 1 1 2 1 0 1
 [8433] 0 0 0 0 0 1 0 0 0 1 0 1 1 0 0 3 1 0 0 1 0 0 0 1 0 0 1 1 1 2 0 0 3 0
 [8467] 1 1 0 0 1 0 0 0 1 0 0 0 0 0 2 2 0 0 0 0 2 0 0 1 0 0 0 1 0 2 1 0 2 0
 [8501] 0 0 0 2 2 3 6 0 0 0 0 1 0 0 0 0 0 0 0 0 0 1 1 0 0 0 1 1 2 1 0 0 0 0
 [8535] 0 0 0 1 0 1 0 1 0 2 2 1 2 0 0 0 1 1 0 1 0 3 4 0 1 2 2 3 1 1 0 1 1 1
 [8569] 1 1 0 2 4 1 2 0 0 4 1 1 1 2 0 0 0 2 4 2 2 2 1 2 0 1 3 0 0 0 4 0 0 1
 [8603] 0 0 1 0 0 0 0 0 0 1 4 4 1 0 0 3 0 0 0 0 1 0 0 1 0 0 1 0 0 0 0 0 2 2
 [8637] 4 0 2 2 2 0 2 1 2 1 1 2 0 1 1 0 0 0 1 1 0 1 0 1 1 0 0 1 1 0 2 1 1 1
 [8671] 0 2 0 0 1 0 2 2 0 0 0 0 2 0 0 1 2 2 0 1 0 0 3 3 2 2 0 0 3 1 3 0 1 1
 [8705] 2 4 0 1 3 4 0 2 4 0 2 1 0 1 2 1 0 0 2 0 2 0 3 1 0 2 1 1 0 1 2 2 2 1
 [8739] 1 1 1 1 0 1 1 0 3 5 4 4 1 3 0 1 1 2 0 0 0 2 3 2 1 2 1 2 2 0 3 0 1 1
 [8773] 4 3 0 2 0 1 3 4 4 0 1 1 2 9 3 3 0 0 2 3 2 1 2 1 1 3 2 0 1 0 0 1 3 0
 [8807] 0 0 0 2 0 3 0 0 3 1 0 0 0 0 2 0 0 0 0 3 2 1 2 1 1 0 0 1 0 1 0 0 0 1
 [8841] 1 2 0 0 0 2 1 2 1 0 0 0 0 0 1 0 0 1 1 2 1 0 1 1 1 0 1 1 0 2 1 0 1 1
 [8875] 1 0 1 0 0 3 2 1 0 2 0 1 0 0 2 0 1 0 2 0 1 0 0 1 0 1 0 1 0 1 0 0 1 1
 [8909] 0 0 0 0 0 1 3 1 0 0 0 0 2 0 1 2 0 0 0 1 0 0 1 0 0 3 2 2 3 1 1 1 0 3
 [8943] 0 0 1 0 1 0 1 0 2 2 0 1 0 0 0 0 0 1 3 0 0 1 1 1 0 0 0 0 0 2 3 1 0 0
 [8977] 0 0 1 0 0 0 1 3 0 1 0 0 0 0 1 0 2 2 1 2 0 1 0 4 4 3 4 5 3 4 0 4 4 3
 [9011] 4 2 1 2 0 2 1 0 0 1 2 1 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 2 0 1 0
 [9045] 0 1 0 0 0 0 2 1 1 0 0 0 0 0 0 1 0 0 1 1 2 0 1 0 0 0 0 0 0 2 0 0 1 0
 [9079] 0 0 1 0 2 0 1 0 0 0 1 0 1 1 0 0 0 1 0 0 0 0 2 0 0 1 0 0 0 2 0 0 1 0
 [9113] 0 1 0 0 1 1 0 0 1 0 2 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0
 [9147] 0 0 0 0 0 0 1 0 0 1 0 0 0 0 1 0 0 0 1 0 0 1 0 0 0 0 0 1 0 0 0 0 0 1
 [9181] 0 0 1 1 0 0 0 0 1 0 1 1 1 0 1 2 0 0 2 0 0 0 2 0 0 0 1 1 0 2 0 0 0 2
 [9215] 1 0 0 0 3 0 0 1 0 0 0 1 0 1 1 0 1 1 0 0 0 0 0 1 0 0 0 1 0 0 1 1 0 0
 [9249] 0 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 1 1 0 0 0 1 0 2 1 0 0 0 1 0 0 1 1 0
 [9283] 0 1 1 0 0 0 1 1 1 1 0 0 1 0 1 1 0 0 0 0 1 3 0 0 0 0 1 1 1 0 0 2 0 0
 [9317] 0 1 2 2 1 3 0 1 0 0 0 0 0 0 0 2 0 2 2 0 1 0 0 0 0 0 1 0 1 1 0 1 0 1
 [9351] 2 0 1 1 0 1 1 1 0 2 0 3 2 2 2 0 0 2 4 1 1 0 0 0 0 0 1 0 0 0 1 0 0 0
 [9385] 0 0 0 0 0 0 0 1 0 0 1 1 1 0 1 0 1 0 1 2 0 1 1 1 1 1 0 0 0 1 0 0 0 0
 [9419] 0 0 0 0 0 0 2 0 1 0 1 0 0 0 1 0 1 1 3 0 1 2 1 0 1 1 2 1 3 0 0 0 0 0
 [9453] 1 0 0 2 0 0 1 0 0 0 1 1 3 2 2 1 1 1 2 1 0 0 0 0 0 0 1 1 0 0 3 1 0 1
 [9487] 1 0 0 0 0 0 0 2 0 0 0 0 2 1 0 2 0 0 0 1 1 0 2 0 1 0 0 0 0 2 0 1 0 0
 [9521] 1 2 2 0 4 0 0 0 2 1 1 1 0 0 0 0 1 1 0 0 3 3 1 0 1 0 0 0 1 0 1 0 1 0
 [9555] 0 0 1 0 1 0 0 0 1 1 2 0 2 0 1 1 1 1 3 0 2 1 1 0 1 1 0 0 2 1 0 0 1 3
 [9589] 2 1 0 1 1 1 3 1 2 0 0 0 0 0 2 0 1 0 1 0 2 3 0 2 4 1 0 1 0 1 0 2 3 0
 [9623] 2 0 2 1 1 0 1 2 0 1 0 0 1 0 0 0 1 1 0 0 0 1 0 0 1 0 0 0 2 1 3 2 0 0
 [9657] 0 0 3 2 0 1 2 0 1 2 0 0 0 1 2 2 1 1 1 2 0 1 1 1 2 0 0 0 2 1 0 3 2 1
 [9691] 2 1 2 2 0 0 0 1 1 2 2 0 0 3 2 0 1 1 2 0 1 1 0 0 2 1 0 2 1 2 0 0 4 3
 [9725] 0 0 0 1 1 0 1 0 0 0 0 0 0 0 1 1 1 0 0 0 0 0 3 2 1 1 0 0 0 1 2 2 0 0
 [9759] 0 0 1 0 0 0 0 0 0 0 0 0 0 2 0 1 1 2 0 0 0 0 0 0 0 1 0 0 1 2 0 0 1 1
 [9793] 2 0 0 0 1 0 1 2 0 0 0 0 2 0 1 0 0 1 0 2 1 0 0 1 1 2 1 0 1 0 0 1 3 0
 [9827] 1 2 1 0 0 1 3 0 0 0 1 0 1 0 1 0 0 0 0 0 1 2 5 1 0 0 0 2 0 0 0 0 2 1
 [9861] 0 0 0 0 0 0 0 0 0 1 1 1 0 1 0 0 0 0 0 0 0 1 0 0 0 0 0 1 1 0 0 0 0 1
 [9895] 0 1 0 1 2 0 0 1 0 0 1 0 1 2 1 0 0 2 0 1 2 1 0 0 0 0 0 1 1 0 2 1 2 0
 [9929] 2 1 3 0 2 0 0 3 0 0 0 1 2 0 0 2 3 2 4 0 0 1 3 0 0 0 0 1 0 1 1 1 0 0
 [9963] 2 1 0 1 1 1 0 2 2 2 0 2 0 2 1 0 1 1 1 1 0 1 2 0 2 2 0 0 2 1 3 1 0 0
 [9997] 0 1 0 0 1 0 0 2 0 1 0 2 0 3 4 1 1 1 2 2 0 0 0 0 0 1 0 0 2 4 3 1 2 2
[10031] 1 2 1 1 2 5 0 0 0 0 0 1 0 0 0 2 1 3 0 1 2 3 3 1 5 0 0 0 1 0 0 1 1 0
[10065] 0 0 0 0 1 1 0 0 1 0 0 1 0 0 1 0 0 0 0 0 0 1 1 1 2 2 0 1 0 2 0 0 0 0
[10099] 2 0 1 0 0 0 1 0 0 0 2 0 3 1 1 0 0 0 0 0 2 0 0 1 1 0 0 0 0 0 0 0 0 0
[10133] 0 1 2 0 2 1 0 0 0 1 1 0 1 3 0 1 0 0 0 1 2 1 0 2 2 2 3 1 1 0 3 0 0 3
[10167] 1 1 0 1 1 0 0 1 0 1 0 0 0 0 0 2 1 1 1 2 0 1 1 0 0 1 4 2 0 1 1 1 0 1
[10201] 1 1 0 1 1 2 1 0 1 1 2 1 1 0 0 2 3 2 1 3 0 1 3 1 0 1 2 0 0 1 1 2 1 2
[10235] 3 2 1 0 1 1 1 1 1 1 0 1 0 0 3 2 3 1 2 2 1 0 0 1 2 1 1 0 0 1 1 1 4 0
[10269] 1 2 0 0 1 3 1 1 1 0 0 1 0 0 2 0 0 0 1 0 1 0 1 1 0 0 0 0 0 1 1 2 2 0
[10303] 1 1 2 2 2 2 0 0 0 0 0 3 0 0 0 0 1 1 1 2 1 3 1 2 0 4 3 1 0 3 0 0 0 2
[10337] 0 0 0 1 0 3 1 0 0 1 0 0 1 2 4 0 0 2 2 1 1 2 0 1 0 3 1 1 0 0 0 0 3 2
[10371] 1 1 0 0 0 0 0 0 0 1 1 1 0 0 2 1 2 2 3 3 3 0 0 1 2 3 2 1 0 0 0 2 1 0
[10405] 0 1 2 4 2 2 6 3 1 0 0 0 2 3 1 0 2 3 0 0 0 1 0 1 1 0 0 0 0 0 1 1 2 1
[10439] 0 0 0 2 0 3 1 1 0 0 0 0 1 1 1 0 1 1 0 0 1 0 0 0 0 2 0 1 2 3 0 0 0 1
[10473] 0 0 0 0 0 1 3 0 0 0 0 0 0 1 0 0 1 0 0 1 0 0 0 0 1 0 0 0 0 1 0 0 0 0
[10507] 0 0 0 0 1 0 0 0 0 1 0 1 1 1 0 1 0 1 0 0 1 1 0 0 1 0 0 0 0 0 0 1 0 1
[10541] 0 0 1 0 1 0 0 0 0 0 0 0 0 1 0 0 1 0 1 2 3 0 0 2 0 0 0 0 1 0 0 0 2 0
[10575] 0 0 0 1 3 2 0 0 0 2 1 1 1 0 1 2 0 0 0 0 0 1 0 0 0 0 0 0 0 1 1 0 1 1
[10609] 1 1 1 0 0 0 1 0 1 0 0 1 0 1 0 0 0 2 1 2 1 0 1 3 0 2 0 0 0 0 0 0 2 1
[10643] 1 0 1 1 0 0 2 2 0 0 0 0 1 1 0 1 1 2 2 0 1 2 1 1 0 0 2 3 0 1 1 1 0 0
[10677] 1 1 0 0 1 1 1 2 0 0 0 1 2 0 1 1 1 0 0 0 0 0 2 0 0 0 0 0 1 1 2 0 1 0
[10711] 0 0 1 0 2 1 0 0 0 0 5 0 0 0 0 0 0 0 0 0 0 2 5 1 0 0 1 0 0 0 0 0 0 0
[10745] 0 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 1 2 1 0 2 4 0 0 0
[10779] 2 0 0 1 2 0 1 0 2 1 0 0 1 1 2 0 0 0 0 0 0 2 0 0 1 2 0 2 0 0 0 1 0 0
[10813] 1 0 0 0 0 0 1 0 0 3 2 0 0 0 1 0 0 0 1 2 0 0 0 0 2 0 0 2 1 3 1 1 0 0
[10847] 1 0 3 0 0 0 0 0 0 1 0 2 2 1 0 1 1 0 1 0 0 1 4 1 1 2 0 1 1 2 1 1 0 1
[10881] 1 1 0 0 0 1 1 0 0 0 0 0 0 0 2 0 3 2 4 1 1 1 1 0 1 1 3 1 0 0 0 0 1 2
[10915] 3 2 2 1 1 2 1 1 1 2 0 0 1 4 2 1 1 0 1 1 1 1 3 4 0 0 0 0 2 0 0 1 1 1
[10949] 0 0 1 0 1 1 3 0 0 1 1 0 1 1 0 0 1 1 4 2 0 0 2 1 1 0 0 0 2 0 1 1 0 2
[10983] 1 2 0 1 1 1 1 0 1 1 3 4 0 2 2 2 2 1 0 0 1 0 0 0 1 0 0 0 1 2 2 1 1 2
[11017] 0 2 0 0 0 0 0 0 0 1 2 0 1 0 2 1 1 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0
[11051] 1 0 0 1 1 2 0 0 0 0 0 0 0 1 2 1 0 0 0 0 0 0 0 1 1 2 0 0 0 1 0 1 2 1
[11085] 1 0 1 1 0 1 3 1 0 0 1 0 0 1 1 1 1 0 1 0 0 0 1 0 0 1 1 0 1 0 0 0 1 0
[11119] 2 1 1 0 0 0 1 3 1 3 1 2 1 0 0 1 0 0 1 0 3 0 0 0 0 1 0 0 3 1 0 0 0 0
[11153] 2 1 0 1 1 3 0 2 1 2 0 3 2 2 1 2 3 4 2 4 1 3 1 2 0 0 1 0 0 2 1 1 0 0
[11187] 1 0 0 2 0 0 1 1 2 1 4 2 0 1 1 3 1 0 0 1 0 0 0 0 1 1 1 1 1 0 2 0 0 0
[11221] 0 1 0 0 3 1 1 0 0 0 0 0 0 1 0 0 0 0 0 1 1 0 1 0 0 0 0 0 0 2 0 1 0 0
[11255] 2 0 0 1 0 0 0 0 0 2 0 0 1 1 0 1 2 4 1 1 0 4 0 2 0 0 0 0 0 0 0 0 1 1
[11289] 2 0 3 2 2 0 0 3 2 0 1 0 0 0 0 1 4 1 0 0 0 2 1 0 1 2 0 2 1 0 0 0 2 0
[11323] 0 0 2 1 0 0 1 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 3 1 1 0 1 0
[11357] 0 3 0 0 1 1 1 2 2 1 2 1 1 2 0 0 0 1 1 0 1 1 1 2 1 1 0 1 0 0 0 0 1 0
[11391] 1 2 0 0 0 0 0 0 0 0 0 0 0 1 1 0 1 1 0 0 2 1 1 1 0 1 5 0 3 1 2 0 1 0
[11425] 0 0 0 0 0 0 0 2 0 0 3 1 3 0 0 1 2 1 0 0 0 0 0 1 1 1 0 0 0 0 0 3 0 1
[11459] 0 2 1 0 2 0 0 2 1 0 1 0 0 3 3 2 1 1 0 0 1 0 1 2 0 1 0 2 0 0 0 0 0 0
[11493] 1 2 1 1 3 0 1 0 2 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2
[11527] 0 1 1 1 0 1 0 0 0 2 0 1 1 0 0 2 0 0 1 1 2 1 0 0 2 2 0 0 0 0 0 0 0 0
[11561] 3 3 2 0 0 0 0 0 0 0 0 0 0 2 0 2 0 0 1 0 0 0 0 0 1 0 1 0 2 1 0 0 0 0
[11595] 2 4 0 0 1 0 1 0 0 0 1 0 1 0 1 1 0 0 0 0 0 0 0 0 0 0 1 1 0 1 1 0 0 0
[11629] 0 0 1 0 0 2 0 1 0 0 3 3 0 0 0 0 0 1 3 2 0 0 0 0 0 0 0 1 0 1 2 0 0 2
[11663] 1 3 0 1 1 1 0 3 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[11697] 0 1 1 1 1 2 3 1 2 1 0 1 0 0 0 0 1 2 4 1 0 0 1 0 0 3 3 0 0 0 0 0 1 0
[11731] 4 3 0 0 0 1 1 0 0 0 1 0 1 0 0 1 0 0 1 0 1 1 0 0 1 0 0 0 0 1 2 0 0 0
[11765] 0 1 0 0 0 0 2 1 0 0 0 0 0 1 0 1 0 1 1 0 0 1 1 0 1 0 2 0 0 2 1 2 0 0
[11799] 0 3 1 3 0 0 1 1 2 0 0 0 0 0 1 1 0 0 0 0 0 1 0 1 0 0 0 2 0 1 0 1 1 1
[11833] 0 1 1 2 2 0 0 0 2 1 0 1 1 1 2 3 0 1 0 0 1 0 0 0 0 1 0 0 4 2 3 2 1 0
[11867] 0 0 1 0 1 1 0 0 0 0 2 1 2 0 1 0 0 0 1 1 0 2 1 1 0 1 1 0 3 0 1 2 1 3
[11901] 4 1 1 1 0 0 2 1 0 0 0 1 2 0 0 2 1 0 0 0 1 4 0 0 0 0 2 1 0 1 1 0 0 1
[11935] 2 2 1 0 3 0 2 2 1 1 0 0 0 0 1 1 2 0 0 0 0 0 1 1 1 1 0 0 0 1 1 0 0 0
[11969] 0 0 0 0 0 0 0 0 0 2 0 0 0 1 2 0 0 0 1 1 3 2 2 1 1 0 0 0 1 1 0 0 0 0
[12003] 0 0 0 2 0 2 3 0 0 1 1 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 0 1
[12037] 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 1 1 0 1 0 0 0 0 0 0 0 0 1 1 0 0 0 0
[12071] 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 2 0 3 0 0 0 0
[12105] 0 0 0 0 0 0 0 0 0 0 3 1 2 1 0 3 1 0 0 1 0 0 0 1 0 0 2 1 0 0 0 0 0 1
[12139] 1 0 1 1 0 1 3 0 0 0 0 0 0 0 2 4 2 0 1 0 1 1 1 3 2 1 1 0 4 1 0 0 0 0
[12173] 0 1 0 0 1 0 1 0 0 2 0 1 2 1 3 3 2 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1
[12207] 0 0 3 0 0 0 3 2 0 2 0 0 0 1 2 0 0 0 1 2 2 0 0 0 0 3 0 2 0 0 0 0 1 0
[12241] 0 1 0 0 0 0 1 0 0 0 1 0 0 0 0 3 0 0 0 0 0 0 0 0 0 0 0 1 1 1 1 1 1 2
[12275] 1 0 0 0 1 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0 0 0 1 1 0 0 0 0 1 0 2 0 1
[12309] 1 0 0 0 0 0 1 1 1 0 1 2 0 1 0 0 0 2 1 0 3 2 0 4 0 1 3 2 0 1 1 2 2 0
[12343] 0 0 0 0 0 0 2 0 2 0 4 1 1 2 1 4 0 2 0 0 3 0 0 0 0 0 0 0 0 0 1 2 1 0
[12377] 0 0 0 0 0 0 0 1 1 0 0 4 2 0 2 2 0 0 0 0 0 0 0 0 0 0 1 3 1 0 0 1 1 0
[12411] 1 0 0 0 0 0 0 0 1 3 1 3 4 2 3 1 1 1 0 1 1 0 0 2 1 0 0 0 0 0 1 2 0 0
[12445] 0 0 2 0 0 3 3 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 2 1 2 0 0 0
[12479] 0 0 0 0 0 0 1 1 0 1 2 1 1 1 3 1 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 2
[12513] 3 0 4 0 2 0 1 0 0 0 0 0 0 0 2 0 2 1 0 1 0 2 1 3 0 0 0 0 0 2 0 0 0 0
[12547] 0 0 0 0 0 3 0 0 0 0 1 0 1 0 1 1 1 0 1 2 1 1 0 1 3 5 0 1 2 0 2 1 2 4
[12581] 1 1 1 0 0 0 0 0 3 0 2 1 1 3 3 3 2 2 3 0 1 3 1 1 2 1 1 4 4 0 0 0 0 0
[12615] 0 1 0 2 2 2 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 1 2 1 1 0 3 1 2
[12649] 1 1 0 0 1 1 0 0 1 1 0 1 1 1 1 2 2 0 1 0 1 1 0 0 0 0 0 0 0 0 0 0 2 1
[12683] 4 1 1 0 0 1 0 0 0 0 0 0 1 3 2 1 0 0 0 0 4 0 2 0 0 2 1 2 1 0 0 0 0 0
[12717] 0 1 1 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 1 2 1 2 0 0 0 1 0 1 1 0 1 0 2 1
[12751] 0 2 1 1 0 1 0 5 1 3 0 1 0 0 0 3 0 0 0 1 2 1 0 0 0 0 0 0 0 0 0 0 1 0
[12785] 1 0 3 1 1 1 2 0 2 1 0 2 1 1 1 0 2 0 0 0 0 0 0 0 0 0 0 0 0 4 0 1 0 1
[12819] 0 1 2 1 0 0 0 0 0 1 2 1 0 0 1 1 0 1 1 1 1 2 2 0 0 1 1 0 0 4 0 1 2 0
[12853] 0 1 1 0 1 2 0 0 0 0 1 0 0 0 1 2 0 3 0 0 0 0 0 3 0 1 1 0 1 3 1 3 0 0
[12887] 4 0 1 0 0 0 0 0 0 1 1 0 2 2 1 1 0 0 2 2 0 0 2 1 2 0 3 2 0 0 3 1 0 3
[12921] 4 4 1 0 0 1 4 3 1 6 3 0 1 1 1 0 1 0 4 3 1 0 0 1 0 0 1 0 0 1 0 1 0 1
[12955] 4 1 1 3 0 0 0 0 2 1 2 2 2 1 2 1 0 0 0 1 3 2 0 1 2 1 0 0 0 0 2 0 4 4
[12989] 3 1 0 2 0 2 1 2 0 2 3 1 0 2 1 3 2 4 2 0 0 0 1 0 2 0 0 0 1 1 0 2 1 2
[13023] 1 2 1 2 1 0 0 0 2 1 0 0 1 0 1 0 3 1 0 6 1 1 1 1 1 1 0 1 1 0 0 0 1 0
[13057] 0 0 0 1 1 2 0 0 1 0 0 0 0 2 0 1 0 2 0 1 0 0 1 2 0 2 0 0 0 1 0 2 0 0
[13091] 1 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 2
[13125] 2 1 1 0 0 1 0 2 2 0 0 0 0 0 1 0 0 2 0 3 1 0 0 0 0 1 1 1 0 0 0 1 0 0
[13159] 0 3 0 1 3 2 2 1 0 2 3 2 1 0 2 2 1 0 0 0 0 0 0 5 3 3 0 0 2 1 0 0 1 0
[13193] 1 1 2 1 2 2 1 3 3 1 0 2 1 1 1 2 1 1 0 0 0 0 1 1 0 1 2 1 1 1 2 2 0 0
[13227] 1 0 0 0 0 0 1 0 0 0 0 0 0 2 1 1 0 0 0 1 0 0 0 0 1 0 1 3 0 0 2 0 0 2
[13261] 1 0 0 3 0 0 1 0 1 1 0 0 0 1 0 1 2 0 0 0 0 0 0 2 2 0 1 0 2 1 1 0 0 0
[13295] 0 0 0 0 0 0 0 0 0 0 0 1 0 2 0 0 0 0 0 1 0 1 1 1 0 0 1 3 3 0 0 2 0 0
[13329] 0 0 0 0 0 1 0 1 4 2 0 2 1 0 1 2 1 2 1 0 2 1 0 0 0 0 1 0 1 0 0 0 0 0
[13363] 0 2 3 2 2 1 0 0 0 0 1 2 2 0 0 1 0 0 0 2 1 0 0 0 0 1 0 1 2 2 0 0 0 0
[13397] 1 0 1 1 0 0 1 1 0 0 0 4 1 1 1 0 2 0 4 3 1 0 0 1 0 0 1 0 0 0 0 0 0 0
[13431] 2 0 1 0 0 1 2 1 1 0 0 0 0 0 0 0 1 0 1 0 0 0 2 3 0 2 0 0 0 0 0 0 0 0
[13465] 0 2 3 0 1 0 1 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 1 0
[13499] 1 0 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0
[13533] 1 1 1 1 1 0 0 0 0 0 0 0 0 0 1 1 1 0 0 1 3 1 0 0 0 0 1 0 0 0 0 0 0 0
[13567] 1 0 0 1 1 0 0 0 1 0 0 0 0 0 1 1 1 0 0 0 0 0 0 0 1 3 1 0 1 1 0 1 0 0
[13601] 1 0 0 0 2 0 0 0 0 0 0 0 1 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0
[13635] 2 1 0 2 1 2 0 1 1 1 1 0 1 2 1 0 0 0 0 0 1 0 0 2 0 0 0 0 0 0 0 0 2 1
[13669] 0 1 2 1 0 0 2 1 0 0 2 1 0 1 1 0 0 0 1 2 1 0 0 0 0 1 1 1 0 0 0 0 1 1
[13703] 1 2 1 0 0 2 1 2 0 0 0 0 0 0 0 1 3 1 1 1 0 0 0 0 1 2 0 0 0 2 1 1 2 2
[13737] 0 2 0 0 0 0 1 0 1 0 0 1 1 0 0 0 0 1 0 0 0 0 3 0 1 0 0 0 0 0 6 0 1 2
[13771] 0 0 2 1 0 0 1 1 1 0 0 0 1 1 1 2 2 1 0 0 0 0 1 1 0 0 0 1 1 2 0 0 0 0
[13805] 0 0 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 2 1 0 0 0 0 0 0 0 0 0 0 0 0 1
[13839] 1 1 0 1 0 0 0 0 0 2 0 0 2 0 0 0 0 0 0 1 0 1 0 2 1 0 0 0 0 0 0 0 0 0
[13873] 0 0 0 0 0 1 0 0 0 2 1 0 0 0 0 0 0 1 0 0 0 2 1 0 1 0 0 0 0 0 1 1 0 0
[13907] 0 0 0 0 1 0 1 0 0 0 2 1 0 0 0 0 1 0 0 0 0 3 0 0 0 0 0 1 0 0 0 0 0 0
[13941] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 1 0 1 0 0 0 0 0 1 0 1 0 4 0 0 1
[13975] 1 0 0 0 0 1 0 2 0 0 2 0 0 2 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0
[14009] 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 1 0 0 0 0 0 0
[14043] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 2 0 0 1 0 0 0 0 0
[14077] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 1 1 1 0 0 0 0 0 0 0 0 1
[14111] 2 0 0 0 0 0 0 0 0 0 3 0 0 0 0 0 0 2 2 0 3 0 0 1 0 0 0 0 0 0 0 0 0 0
[14145] 2 1 0 0 1 0 0 0 0 0 0 0 1 0 0 0 0 0 0 1 3 0 2 1 0 0 0 0 0 0 0 0 1 0
[14179] 0 0 1 0 0 3 1 2 1 2 1 0 0 0 0 0 0 0 2 2 2 1 0 0 0 1 1 2 1 0 1 1 0 1
[14213] 0 0 0 0 0 0 0 0 2 0 0 1 0 3 0 1 0 1 0 0 1 1 0 0 1 0 0 0 0 0 0 1 1 2
[14247] 0 0 2 1 2 1 0 0 0 0 3 0 2 1 3 0 1 1 0 0 2 2 1 0 0 0 0 0 0 0 0 2 0 0
[14281] 5 2 1 0 0 0 0 0 0 3 2 0 3 1 1 0 0 1 0 0 0 0 2 1 1 2 0 0 0 0 0 0 1 1
[14315] 0 0 0 0 1 0 0 1 0 0 1 0 0 0 0 1 1 2 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0
[14349] 1 1 0 1 1 4 1 1 1 0 0 2 3 1 0 0 0 1 1 1 0 0 0 1 1 1 0 0 0 1 0 0 0 0
[14383] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 2 0 0 0 0 2 1 1 0 2 0 2 1 0 1 2
[14417] 1 2 1 0 0 1 4 1 2 1 0 1 0 0 3 2 3 1 1 0 0 2 0 0 2 0 2 1 1 2 0 0 3 2
[14451] 2 1 1 2 1 3 1 0 0 3 3 3 1 3 0 2 1 2 0 0 0 2 0 1 1 2 0 0 1 0 1 0 0 0
[14485] 0 0 0 1 1 1 2 1 1 2 0 1 1 0 2 0 0 4 4 2 2 0 2 1 1 2 0 3 2 0 1 0 0 0
[14519] 1 0 0 0 0 1 0 1 1 1 0 2 0 0 1 1 0 2 0 0 0 0 0 0 1 1 0 0 1 0 2 1 0 0
[14553] 0 0 2 1 0 2 1 0 0 0 0 0 1 1 0 0 1 0 1 1 1 0 0 1 0 1 1 0 0 0 0 1 0 0
[14587] 0 0 1 1 1 0 0 0 1 0 0 1 0 1 1 2 0 0 0 0 0 1 0 0 1 2 0 1 0 0 4 1 0 0
[14621] 2 1 2 0 2 1 1 0 0 0 0 0 1 1 0 2 2 1 1 1 0 2 1 0 1 2 1 1 2 1 0 0 2 1
[14655] 0 0 0 1 0 1 1 0 0 0 1 2 1 0 2 1 1 0 0 3 1 1 0 0 3 1 1 1 2 0 2 2 2 0
[14689] 1 0 0 0 2 2 0 2 0 2 2 0 0 1 1 1 1 1 0 1 0 0 3 1 0 2 0 1 2 0 0 2 1 1
[14723] 0 1 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 1 1 1 0 0 1 0 1 0 1 1 0 0 0 1 1
[14757] 2 2 1 0 0 2 1 0 1 0 0 0 1 1 2 0 2 0 1 1 3 0 1 3 1 1 0 1 0 2 3 2 4 1
[14791] 0 0 1 0 1 0 0 1 1 2 1 3 1 0 0 0 2 1 0 0 0 1 0 0 0 1 1 2 0 0 1 2 2 0
[14825] 2 0 2 2 0 2 0 1 0 1 2 0 2 1 1 1 3 1 1 1 3 2 3 4 1 0 1 2 0 0 1 0 3 0
[14859] 0 0 0 0 0 1 2 1 2 1 3 0 2 1 0 0 0 1 0 0 0 0 0 0 0 0 0 2 0 1 1 0 1 0
[14893] 1 0 0 0 1 1 0 1 0 0 0 0 1 0 0 1 1 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0
[14927] 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 2 1 0 0 1 0 0 1
[14961] 1 0 1 0 1 1 0 1 1 0 1 1 0 0 0 0 4 5 1 0 1 1 2 2 2 0 0 0 0 0 1 2 0 1
[14995] 0 1 2 1 0 0 0 2 1 2 1 0 1 0 1 4 2 0 0 2 0 1 1 3 0 0 1 2 1 0 2 1 1 0
[15029] 1 1 0 3 0 0 0 2 0 1 4 1 1 2 0 0 1 1 0 0 1 3 1 3 2 1 0 0 0 0 1 3 2 5
[15063] 0 2 5 3 4 1 0 0 0 0 0 0 1 1 2 1 2 4 0 4 1 0 0 1 0 1 0 0 1 0 1 2 0 1
[15097] 0 0 0 0 1 0 1 0 0 1 0 2 0 2 0 0 1 2 0 0 4 1 1 1 1 0 2 1 0 0 0 0 0 3
[15131] 2 0 0 1 0 0 1 1 0 2 0 1 2 0 0 0 1 1 1 1 1 0 2 0 2 0 0 1 4 1 0 0 0 0
[15165] 0 1 0 2 1 0 0 1 0 1 0 0 2 0 0 0 2 1 0 0 0 3 0 1 2 0 0 1 0 3 1 0 0 1
[15199] 1 0 1 0 0 1 0 0 2 0 0 0 0 0 1 1 0 0 0 3 2 0 2 1 1 0 0 0 2 0 2 1 1 0
[15233] 0 0 0 0 0 1 0 0 0 0 0 1 1 1 1 0 0 0 4 2 2 4 2 2 2 1 2 0 2 2 2 2 1 1
[15267] 0 1 1 1 4 1 0 0 1 1 1 0 0 0 0 0 1 2 0 2 0 0 0 0 1 0 1 1 0 1 0 0 3 1
[15301] 0 0 3 0 2 0 1 0 0 0 0 0 0 0 2 0 0 1 1 1 0 2 0 1 1 1 1 1 1 3 1 1 3 1
[15335] 2 1 3 1 3 3 0 0 1 0 0 1 1 1 1 0 1 0 1 0 0 2 1 0 1 1 0 4 0 1 0 1 2 0
[15369] 0 1 0 0 2 1 3 3 2 2 0 0 0 0 1 1 1 2 3 1 0 1 1 0 1 2 3 0 0 0 0 3 0 2
[15403] 0 0 0 0 1 1 2 0 0 0 0 2 2 1 1 2 0 3 1 0 0 1 0 2 2 1 3 3 3 3 2 2 0 0
[15437] 1 2 1 3 0 1 3 4 2 0 0 0 0 2 1 1 0 2 3 1 1 0 1 1 0 1 3 0 0 0 0 2 0 0
[15471] 0 0 0 1 1 1 4 2 0 0 0 1 1 0 2 0 1 2 0 0 0 2 2 1 2 0 1 0 0 0 0 1 1 1
[15505] 1 0 0 0 0 0 2 1 1 1 1 0 2 1 1 2 2 0 1 1 2 3 1 1 2 1 0 3 1 1 1 1 3 3
[15539] 2 1 0 1 0 0 1 1 1 0 0 2 2 0 1 2 2 2 2 1 4 2 1 1 0 1 1 1 0 1 0 3 0 2
[15573] 0 7 2 1 2 2 1 2 1 0 1 3 3 1 0 0 0 1 3 1 0 2 3 1 2 0 0 1 1 0 0 0 3 3
[15607] 0 1 1 0 0 2 0 0 0 0 0 0 0 0 0 1 0 2 1 0 3 0 0 0 3 2 2 2 1 2 0 0 1 0
[15641] 1 1 0 0 0 0 0 0 0 0 0 0 0 1 0 2 0 0 0 1 0 0 0 0 1 0 0 0 2 0 0 0 0 1
[15675] 0 3 1 1 0 4 0 0 0 0 0 1 1 1 0 0 0 0 0 2 1 0 2 0 2 1 1 0 1 2 1 1 0 0
[15709] 1 0 1 1 0 0 0 4 3 3 1 0 0 0 0 2 0 1 2 0 0 0 0 0 1 0 1 2 1 0 0 0 1 0
[15743] 0 2 0 0 1 1 0 0 0 5 4 2 1 3 0 2 1 1 0 3 5 1 0 3 1 0 3 0 1 0 2 0 1 0
[15777] 2 2 1 0 2 2 1 1 1 0 0 2 0 1 0 2 0 0 0 0 1 3 1 0 0 0 2 2 0 0 2 0 3 2
[15811] 0 2 1 3 2 1 0 1 1 2 0 0 1 0 0 2 0 1 1 0 0 0 0 1 0 1 1 3 1 2 3 1 1 0
[15845] 0 2 0 2 2 1 2 2 0 2 0 1 1 0 0 0 1 1 0 0 0 2 2 0 0 1 1 0 1 0 1 1 0 3
[15879] 2 0 0 0 1 2 2 1 1 2 0 1 0 0 0 0 1 3 4 0 1 2 0 0 0 0 0 1 2 1 2 3 1 2
[15913] 1 0 1 0 0 0 0 1 3 0 0 1 0 0 0 0 0 0 0 0 0 1 0 1 0 1 1 0 1 0 2 0 1 0
[15947] 0 1 0 1 0 0 0 1 1 0 1 1 1 1 2 3 0 0 0 0 0 0 0 0 1 0 1 2 0 0 0 0 0 1
[15981] 1 0 2 0 1 0 1 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[16015] 1 1 0 1 0 4 0 1 0 0 0 0 1 0 1 0 0 1 0 0 0 2 0 0 0 0 0 2 0 1 1 2 1 1
[16049] 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 2 0 2 2 5 3
[16083] 1 1 0 1 1 2 0 0 1 1 1 1 0 0 0 2 0 1 0 0 0 0 1 1 1 2 0 1 0 1 1 3 0 2
[16117] 0 3 0 2 0 0 1 0 2 3 2 2 2 0 2 1 0 2 2 1 1 1 1 0 0 2 1 0 1 2 0 0 1 1
[16151] 0 0 2 2 1 1 3 2 1 5 1 1 0 4 2 0 1 2 1 0 3 4 0 6 0 0 1 1 0 2 2 2 2 1
[16185] 0 1 3 3 0 3 1 0 1 0 1 3 1 5 0 0 2 3 0 0 0 1 4 0 1 1 4 2 1 3 2 0 1 0
[16219] 0 0 0 0 0 0 0 0 2 0 1 1 0 0 0 1 0 1 2 1 0 0 0 0 2 0 0 0 0 1 0 2 0 0
[16253] 0 0 0 0 0 0 0 0 2 1 1 3 1 2 0 0 0 1 0 0 1 0 1 0 1 0 1 1 1 0 0 1 3 3
[16287] 0 1 0 1 0 1 1 1 0 0 0 0 1 1 1 0 2 0 2 1 1 0 2 1 0 0 1 2 0 0 1 1 0 0
[16321] 1 0 1 1 1 0 0 0 0 0 0 1 0 1 0 0 1 1 1 3 1 1 0 1 0 1 0 1 1 0 0 1 2 2
[16355] 1 0 0 0 1 0 0 0 1 3 1 1 0 0 1 0 0 0 0 0 0 1 1 0 1 0 0 0 1 1 2 0 0 0
[16389] 0 3 1 2 0 3 4 1 2 0 0 0 0 0 2 3 0 1 1 1 1 1 3 1 0 3 0 1 2 1 5 1 1 0
[16423] 0 2 2 1 1 0 0 0 1 1 1 2 1 5 1 3 1 0 0 0 0 0 2 4 1 1 1 0 0 3 1 0 0 1
[16457] 1 1 0 2 0 1 1 1 1 0 1 0 0 0 2 1 1 0 3 0 0 1 0 2 0 0 0 1 2 1 0 0 0 0
[16491] 2 1 1 2 1 1 1 1 2 4 0 0 1 0 0 1 0 0 2 4 1 0 0 0 1 0 4 0 0 0 0 0 0 2
[16525] 1 0 2 2 0 1 1 0 0 0 0 1 0 0 0 1 0 3 1 2 0 1 0 0 0 0 1 3 0 2 1 1 1 0
[16559] 1 0 2 5 1 1 0 0 2 1 1 0 1 1 0 1 0 0 0 0 0 0 0 4 0 2 0 5 1 2 5 0 2 4
[16593] 1 1 1 0 2 0 1 1 0 2 0 2 2 0 3 0 2 0 1 1 0 0 0 2 1 0 1 3 0 1 1 1 0 0
[16627] 1 1 1 1 0 0 1 1 0 1 1 0 1 0 0 0 0 1 0 1 1 1 1 1 1 1 0 0 2 0 1 1 2 1
[16661] 0 0 1 1 2 3 2 2 1 1 0 1 3 0 0 0 0 3 5 2 1 1 0 4 1 4 4 0 0 0 0 0 0 0
[16695] 0 0 2 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1 0 0 3 1 0 1 0 2 1 0 1 0
[16729] 1 0 1 1 3 2 1 0 3 0 0 0 0 0 0 0 0 0 0 3 0 2 2 0 0 0 0 2 0 0 0 0 1 0
[16763] 1 1 0 4 1 0 0 2 1 0 0 0 0 1 1 1 1 0 0 2 0 1 2 0 0 0 0 0 0 0 1 1 2 2
[16797] 0 1 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 1 2 0 2 0 0 0 0 0 0 0
[16831] 0 0 0 0 0 0 0 0 3 1 2 1 0 1 2 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 2 1 0 0
[16865] 0 2 1 1 1 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 1 2 1 0 0 2 0 0 0 0 0 0 2
[16899] 0 0 3 0 0 0 0 0 1 1 1 0 0 0 0 0 0 3 2 1 2 1 1 0 0 0 0 0 0 0 0 0 1 0
[16933] 0 0 0 0 3 1 0 1 2 1 0 0 0 0 0 2 2 0 1 0 0 1 0 0 1 0 0 0 0 0 4 1 2 0
[16967] 1 1 0 2 0 0 0 0 0 1 1 1 0 0 1 2 0 1 0 0 0 0 1 0 2 1 0 0 0 0 0 2 2 0
[17001] 0 2 2 0 0 1 2 0 1 3 2 1 3 2 1 1 2 0 1 0 0 0 2 2 3 0 1 0 0 0 0 0 0 0
[17035] 0 0 0 0 0 0 0 0 0 0 0 0 0 2 1 2 0 0 1 1 1 2 1 0 0 0 0 1 0 0 1 0 1 0
[17069] 2 0 0 3 1 0 0 0 1 0 1 0 1 3 2 1 0 1 0 1 2 0 1 4 3 0 0 1 0 0 2 1 0 0
[17103] 0 0 0 0 0 1 0 1 0 0 0 0 0 0 1 0 2 2 0 1 1 0 1 0 0 0 1 2 0 1 0 2 1 1
[17137] 0 0 0 0 1 1 0 1 0 0 0 0 1 2 0 2 2 0 0 2 0 1 0 1 2 1 0 1 2 1 1 0 0 0
[17171] 0 0 0 0 0 0 0 1 1 0 2 0 0 0 3 3 2 0 0 1 3 0 1 0 2 0 0 0 1 0 2 0 1 0
[17205] 0 0 1 0 0 0 0 0 0 3 0 1 0 2 1 1 0 1 0 0 3 0 1 0 0 0 3 0 0 2 2 1 0 0
[17239] 0 0 0 0 0 0 0 2 1 0 1 2 0 2 0 2 2 3 0 0 2 1 2 0 2 0 2 3 1 2 1 2 0 0
[17273] 1 2 0 2 0 2 2 1 3 0 0 0 1 1 1 1 1 2 0 0 2 0 0 2 3 1 0 0 0 0 2 0 0 1
[17307] 1 0 0 0 2 0 0 2 0 2 0 1 2 0 1 1 1 0 0 0 0 1 1 1 0 0 0 0 0 0 1 0 1 0
[17341] 3 0 0 1 0 1 0 0 1 0 1 0 2 3 1 2 1 2 1 2 1 2 4 1 0 1 2 2 0 1 1 4 1 2
[17375] 1 1 3 0 1 0 0 1 0 1 0 0 1 2 1 2 0 3 2 1 3 1 1 3 0 1 0 1 0 1 0 3 1 1
[17409] 1 1 1 1 3 1 3 4 1 0 1 1 0 0 2 2 1 0 2 2 1 2 2 0 0 0 0 0 0 0 1 0 0 0
[17443] 2 1 0 0 0 0 0 0 0 0 1 0 0 0 0 0 1 0 0 1 1 1 0 0 1 0 0 0 0 0 0 0 0 1
[17477] 0 0 0 0 1 0 0 0 0 0 1 0 0 1 1 0 0 0 1 0 1 0 0 1 0 0 0 0 0 2 0 1 0 0
[17511] 2 0 0 0 0 1 0 0 0 1 0 0 0 2 0 0 2 0 1 2 0 1 2 0 0 4 1 0 2 0 0 0 1 0
[17545] 0 0 1 2 1 1 0 0 0 0 2 1 2 1 1 0 3 1 1 0 0 2 1 1 1 0 1 1 0 0 0 0 0 0
[17579] 0 1 0 2 2 0 1 1 0 0 0 0 4 0 0 0 0 0 0 1 1 0 0 0 1 0 0 0 0 0 1 0 0 0
[17613] 2 0 0 1 0 2 3 5 2 2 0 1 2 1 0 0 2 2 0 2 0 2 1 0 1 1 0 0 2 1 2 4 4 1
[17647] 1 1 0 2 1 1 3 0 1 0 0 0 0 2 1 0 3 0 0 1 0 0 1 0 0 0 0 0 2 1 0 0 0 1
[17681] 0 1 1 1 0 0 1 2 1 0 0 1 1 0 0 1 1 0 0 0 0 1 2 0 1 4 0 0 0 2 1 1 0 1
[17715] 1 1 1 0 1 2 0 0 0 0 0 0 0 2 1 0 1 1 0 0 0 1 2 0 0 0 0 0 0 0 3 1 3 2
[17749] 1 0 2 0 1 1 1 2 2 1 1 0 2 0 2 1 0 0 2 3 0 0 0 0 0 1 2 2 0 0 2 2 0 3
[17783] 0 1 1 2 1 1 1 2 0 3 2 0 2 0 0 1 2 2 1 1 1 2 2 3 1 2 2 0 0 0 0 1 1 1
[17817] 0 2 3 0 1 1 0 0 1 0 0 0 0 1 1 0 1 0 1 0 0 0 0 0 0 0 2 0 0 0 0 1 1 0
[17851] 0 1 1 0 0 0 1 3 1 3 0 0 0 1 0 0 0 0 1 1 3 2 3 0 1 1 1 1 0 1 3 0 1 4
[17885] 0 1 1 1 0 1 2 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 2 0 3 0 0 2 1 0 0
[17919] 0 0 1 0 2 0 1 0 0 0 0 0 1 0 1 0 1 0 0 0 0 0 1 0 0 2 1 1 1 0 0 1 0 1
[17953] 2 2 1 0 3 1 0 1 2 0 1 3 2 1 0 0 0 1 0 0 1 1 0 0 0 0 0 0 0 0 1 0 0 1
[17987] 1 0 0 0 0 0 1 2 0 0 1 0 1 0 0 0 0 0 0 1 0 0 0 0 0 1 0 0 0 0 1 0 0 0
[18021] 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1
[18055] 0 0 0 2 1 0 0 1 2 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0
[18089] 0 0 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 1 2 1 1 1 1 1 0 0 2 0 1 0 0 0 0 0
[18123] 0 0 0 2 1 0 0 1 0 1 0 1 2 2 2 3 3 0 0 0 2 1 0 0 1 0 0 0 0 0 0 0 0 0
[18157] 1 0 0 0 0 0 0 2 0 0 0 0 0 1 0 0 0 0 0 2 0 0 0 0 0 0 0 2 0 3 1 2 2 2
[18191] 2 2 1 0 0 0 0 1 1 0 1 2 0 1 2 2 0 0 0 0 2 0 0 0 0 2 0 0 0 0 0 0 0 2
[18225] 0 0 3 0 1 0 1 3 0 0 2 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[18259] 0 0 0 0 1 0 2 1 1 0 1 1 1 1 0 0 0 0 0 1 2 1 1 0 0 0 0 2 0 0 0 0 0 0
[18293] 0 0 0 1 0 0 1 0 0 1 0 0 0 1 1 1 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0
[18327] 0 1 0 0 1 0 0 1 1 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 1 1 1 0 0 0 0 0 2 0
[18361] 1 1 0 0 0 0 0 0 0 0 1 0 3 0 0 1 2 1 0 0 0 0 1 0 0 0 0 0 0 1 1 1 0 0
[18395] 0 0 0 0 0 1 0 1 2 0 0 0 1 0 0 0 1 0 0 0 1 1 0 0 1 0 0 0 1 1 2 2 0 1
[18429] 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 4 0 0 0 0 0 2 1 1 0 1 0 0 0 0 0 1 3 1
[18463] 1 0 0 2 0 0 0 1 1 0 0 1 0 0 0 0 0 0 0 0 0 0 1 0 1 0 3 0 0 0 0 0 0 0
[18497] 2 0 0 1 0 1 0 0 0 1 1 1 1 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0
[18531] 2 2 2 1 1 1 0 0 0 0 0 0 0 4 1 3 0 0 2 2 3 1 1 1 1 0 1 0 0 1 0 2 1 0
[18565] 1 0 0 1 2 1 2 0 1 1 0 2 0 0 0 2 0 3 0 1 1 0 1 0 1 2 0 0 0 1 1 2 0 2
[18599] 1 0 0 0 1 0 0 2 0 0 1 0 0 2 1 0 0 0 0 1 4 1 1 0 2 0 1 0 3 0 1 1 0 2
[18633] 0 0 0 0 0 0 1 0 2 0 2 1 2 2 3 1 1 2 0 2 1 4 1 2 1 0 0 0 0 3 0 0 1 3
[18667] 1 2 0 1 1 3 0 0 0 0 0 0 2 2 4 0 0 2 1 2 0 0 0 0 0 0 0 0 0 3 0 0 0 3
[18701] 1 1 0 0 0 0 0 1 1 1 1 0 1 1 1 0 0 0 0 0 0 1 0 0 0 2 0 0 0 0 1 2 0 1
[18735] 2 0 0 1 0 1 1 1 2 1 0 0 1 0 0 0 0 0 0 0 0 0 1 1 2 0 0 1 0 0 0 0 0 0
[18769] 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[18803] 1 0 0 2 1 0 0 0 0 0 0 0 0 0 0 0 3 1 1 3 2 1 0 1 1 0 0 1 1 0 0 0 1 0
[18837] 0 0 2 0 1 1 0 2 1 2 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2
[18871] 2 0 1 3 1 4 0 1 0 0 0 0 0 0 1 1 1 1 2 0 0 0 1 0 2 1 0 2 0 2 0 1 0 0
[18905] 0 0 0 1 0 0 0 0 0 1 3 4 1 0 1 3 2 1 2 1 2 0 1 1 0 2 0 0 1 0 0 0 0 0
[18939] 2 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1 0 0 0 0 0 0 0 1 0 0 0 0 0 0
[18973] 0 0 0 0 0 1 0 0 0 2 0 0 0 3 0 2 1 0 0 1 0 1 0 0 0 0 1 0 0 1 0 1 2 0
[19007] 0 0 1 1 1 0 0 2 2 0 0 1 0 2 0 2 2 1 3 1 0 0 1 1 2 5 0 2 2 2 0 0 0 0
[19041] 0 0 0 0 2 0 0 1 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0
[19075] 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0
[19109] 0 0 0 0 0 0 0 0 0 0 2 2 1 3 0 1 2 0 0 0 2 1 0 2 0 0 1 0 0 0 3 0 1 1
[19143] 0 0 1 1 1 1 1 2 0 1 0 1 1 1 1 2 0 0 0 0 0 0 1 0 0 0 1 0 0 1 0 0 2 0
[19177] 1 0 0 1 0 0 0 0 0 0 0 1 1 2 0 0 2 0 1 0 0 0 1 1 0 1 1 3 3 4 3 0 0 0
[19211] 0 1 0 1 0 0 0 2 2 1 0 0 1 0 1 1 0 0 1 0 1 0 0 0 0 0 0 1 0 0 0 0 1 1
[19245] 0 0 1 1 1 0 0 1 0 0 1 0 0 2 0 0 0 0 2 1 1 1 1 0 2 1 0 5 0 2 4 1 2 1
[19279] 1 1 1 3 0 0 2 0 4 3 0 1 1 1 1 0 0 0 3 0 0 0 0 0 0 0 0 2 1 0 0 0 0 0
[19313] 0 0 1 0 1 0 2 0 0 2 0 0 0 0 1 0 0 2 0 0 0 1 0 0 2 1 0 0 0 2 2 2 1 3
[19347] 1 2 0 3 2 1 1 4 2 0 0 0 0 1 2 0 1 0 1 1 1 0 0 0 2 2 1 5 5 0 1 0 0 2
[19381] 0 0 2 0 4 0 1 2 1 4 4 0 1 1 0 1 3 0 0 1 0 2 4 1 0 1 0 1 2 7 1 1 0 0
[19415] 2 0 1 6 0 1 1 2 0 1 1 0 0 1 0 0 2 5 3 3 2 2 2 1 2 2 1 1 1 1 1 0 1 0
[19449] 0 1 0 0 0 0 1 2 3 0 0 3 2 3 1 1 1 4 2 0 0 3 0 0 0 1 1 0 3 0 0 1 2 1
[19483] 0 0 0 0 0 0 0 0 0 1 2 1 1 1 3 1 0 0 2 1 0 2 1 0 1 2 1 2 1 0 1 1 0 1
[19517] 1 2 3 1 3 1 2 0 2 2 0 1 2 0 1 1 1 1 0 1 1 0 0 0 2 0 1 0 0 0 1 1 0 0
[19551] 0 1 0 0 0 0 0 0 0 1 0 0 0 1 0 0 1 0 0 0 1 2 1 0 0 0 0 0 0 0 1 3 0 1
[19585] 0 0 2 1 2 1 1 0 0 0 1 2 0 1 1 0 2 3 2 1 2 0 1 2 3 0 1 0 1 4 0 2 2 2
[19619] 1 0 0 1 1 1 1 0 3 1 2 2 0 1 0 0 0 3 2 1 1 1 0 1 0 0 0 1 0 1 0 4 0 1
[19653] 1 3 0 1 1 1 0 0 3 0 3 0 1 0 0 0 0 0 1 1 1 0 1 1 1 1 1 0 0 0 1 1 1 0
[19687] 0 0 0 1 0 1 2 1 0 0 0 0 1 0 3 0 0 0 0 2 0 1 0 1 2 0 0 1 0 0 0 0 0 0
[19721] 0 2 0 0 0 1 2 2 0 1 1 0 1 3 0 0 2 1 1 1 0 2 2 2 1 2 0 0 2 4 2 2 1 0
[19755] 2 0 0 1 0 0 0 2 2 1 0 2 2 0 0 0 0 0 1 0 0 0 1 0 2 0 0 0 2 0 1 2 0 0
[19789] 0 0 1 0 0 1 0 1 1 0 0 0 1 2 4 1 0 0 1 0 2 2 0 0 0 0 2 2 0 0 2 0 2 3
[19823] 1 3 0 1 1 0 0 1 1 0 2 2 0 0 0 1 1 0 3 0 0 0 2 0 0 1 0 1 1 0 0 0 1 2
[19857] 2 0 4 0 0 0 0 3 0 0 2 1 0 0 1 0 0 1 1 1 0 0 0 2 2 2 2 0 3 0 0 1 0 0
[19891] 3 2 1 0 4 4 3 0 0 1 0 2 0 0 4 0 1 1 1 0 1 0 1 1 0 1 1 1 1 1 0 2 1 0
[19925] 0 0 0 0 0 3 0 0 2 0 0 0 1 0 1 1 1 2 0 0 0 0 0 1 0 0 0 0 0 0 2 1 3 0
[19959] 1 0 0 1 1 2 1 3 4 3 0 2 1 1 0 2 0 2 0 0 0 0 1 2 2 0 1 0 0 1 0 2 2 0
[19993] 0 1 1 1 0 1 1 0 2 2 2 0 1 0 0 0 1 1 1 1 2 1 0 2 2 4 3 0 0 0 0 0 0 0
[20027] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0 0 0 1 0 0 0 1
[20061] 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0 0 2 0 1 0 0 1 0 0 2 0 0
[20095] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 1 1 1 1 0 3 0 1 0 2 0 1
[20129] 2 1 0 1 1 2 0 0 0 1 1 0 0 0 1 0 1 0 1 1 0 0 1 0 0 2 0 1 0 0 0 0 0 0
[20163] 2 0 1 0 0 1 1 0 1 1 0 1 2 1 0 0 0 3 1 0 0 1 0 1 1 2 0 3 1 0 0 0 0 0
[20197] 0 0 0 0 1 0 2 1 0 0 0 0 0 0 0 1 0 0 0 0 1 2 1 1 1 3 2 0 1 0 0 1 0 0
[20231] 0 2 0 0 0 1 1 1 1 0 0 2 0 1 0 0 2 1 1 1 0 2 0 0 2 0 0 2 0 0 0 0 0 0
[20265] 0 0 0 0 0 0 3 0 0 1 0 1 0 1 0 0 0 0 0 0 1 0 0 2 0 0 0 1 0 0 0 0 0 0
[20299] 0 1 5 0 0 0 1 1 0 0 0 2 0 0 0 1 0 0 1 2 0 1 0 1 0 0 1 0 1 0 0 1 3 2
[20333] 1 0 1 1 0 1 1 3 2 0 2 1 2 1 0 1 0 0 1 0 0 0 1 0 0 0 1 0 0 0 1 0 0 2
[20367] 0 0 0 0 0 0 0 1 3 1 1 0 0 2 1 0 0 1 1 1 0 0 0 0 0 0 0 1 0 3 0 0 0 0
[20401] 0 0 0 0 0 0 1 1 1 0 0 0 0 0 1 2 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0
[20435] 0 2 1 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0
[20469] 1 1 1 1 0 0 1 1 0 2 0 0 1 0 1 0 1 0 0 0 0 2 0 0 0 0 0 0 0 1 0 0 0 0
[20503] 0 2 0 1 1 2 0 0 1 0 0 0 0 0 0 1 0 1 0 0 1 1 0 0 1 0 1 0 1 2 0 0 1 0
[20537] 0 1 1 0 0 0 0 0 1 1 0 0 0 1 2 1 1 0 0 0 1 0 1 0 0 1 0 1 0 2 0 0 0 0
[20571] 1 1 0 0 0 0 1 1 2 0 0 1 0 0 0 0 1 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0
[20605] 0 1 1 0 0 1 0 0 1 0 0 1 0 0 0 1 0 1 1 0 0 0 1 0 0 0 0 1 0 0 0 0 1 1
[20639] 0 0 0 0 2 0 5 4 2 1 0 0 0 0 2 2 0 1 1 0 0 1 0 2 1 4 2 0 0 1 1 1 1 0
[20673] 1 2 0 0 1 0 0 0 0 0 0 1 3 1 0 0 1 2 0 0 1 0 1 0 0 0 1 1 0 0 0 0 1 0
[20707] 0 0 0 1 1 1 1 0 0 0 2 3 4 2 0 0 0 1 0 1 0 0 1 0 0 1 3 1 1 2 1 1 3 0
[20741] 0 0 0 0 0 2 0 1 3 0 0 0 0 2 1 3 4 0 1 1 1 1 0 0 0 0 1 0 2 1 2 0 0 2
[20775] 2 0 0 1 2 3 1 1 0 1 0 1 0 3 2 0 0 1 1 1 0 0 4 1 2 3 3 1 0 0 0 0 0 0
[20809] 0 1 0 0 0 0 0 1 0 0 0 0 0 0 1 0 1 3 0 0 0 0 0 1 1 0 0 0 0 1 1 1 0 1
[20843] 0 0 2 0 0 1 0 0 0 0 0 1 2 1 0 1 0 1 1 0 1 0 0 0 1 1 1 0 2 0 0 0 0 0
[20877] 0 0 2 2 1 1 0 1 0 1 1 0 0 0 0 0 1 0 1 0 1 0 2 0 0 1 1 2 0 1 0 0 2 1
[20911] 1 1 0 1 0 2 1 0 1 0 0 1 1 0 3 0 3 4 5 3 1 1 0 0 0 1 1 0 0 0 0 0 1 0
[20945] 1 0 0 0 1 0 0 0 0 0 0 2 1 3 2 1 2 0 1 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0
[20979] 0 0 0 0 0 0 0 1 3 0 0 1 1 0 0 0 0 0 0 0 0 0 1 2 0 1 0 2 0 0 2 1 1 3
[21013] 2 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0 1 0 0 1 0 0 0 0 1 0 0 1 0 0 0
[21047] 1 1 1 0 0 1 2 1 0 2 0 0 0 1 0 0 0 0 2 1 2 0 0 1 2 0 0 3 0 0 0 0 1 0
[21081] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 1 0 0 0 1 1 1 0 1 1 0 1
[21115] 0 0 0 0 0 0 0 0 1 0 0 1 1 3 0 0 1 0 0 1 1 0 0 0 1 0 0 0 0 0 1 0 0 0
[21149] 0 2 0 0 0 2 2 0 0 0 0 0 0 1 1 0 1 0 0 0 0 0 0 0 0 0 0 0 1 1 1 0 1 1
[21183] 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 2 2 0 0 1 2 0 1 0 0 0 0 0 2 0
[21217] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1 1 0 0 0 0 0 1 0 1 0 0
[21251] 0 0 1 0 0 2 1 0 0 2 1 0 2 0 0 0 1 1 0 0 2 1 0 1 1 0 1 0 0 1 0 0 1 2
[21285] 0 0 0 0 0 0 2 1 0 1 1 1 2 1 1 0 0 0 1 0 0 0 3 0 0 0 0 1 1 0 0 1 0 0
[21319] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 1 2 0 0 0 0 0 0
[21353] 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 1 0 1 0 0 0 0 0 1 1 0 0 0 0 2 1 0 0
[21387] 0 0 0 0 0 1 0 0 0 0 1 0 0 0 0 2 0 1 0 0 1 0 0 1 0 0 0 1 0 0 2 0 0 1
[21421] 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0 1 0 0 1 0 0 0 0 0 0 0 0 0 0
[21455] 0 0 0 0 2 1 1 1 0 1 1 0 1 1 0 0 0 0 1 0 0 0 1 0 0 0 1 1 1 1 1 1 1 1
[21489] 0 0 1 0 0 0 2 1 2 2 0 0 0 1 1 0 1 0 3 0 0 0 0 0 0 1 0 0 0 0 1 0 0 1
[21523] 1 1 1 0 0 2 1 0 0 0 0 1 1 0 0 1 0 3 0 1 1 0 0 0 0 0 0 1 2 2 1 1 1 1
[21557] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 2 0 0 0 0 0 0 0 1 0 1 3 0
[21591] 0 0 0 1 0 0 0 0 1 0 0 0 1 0 0 1 4 0 0 0 1 2 0 0 0 0 0 0 0 0 0 0 0 0
[21625] 1 0 0 0 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0
[21659] 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 1 0 0 1 0 0 0 0 1 1 0 0 1 1 1 0
[21693] 0 0 0 0 0 1 0 0 1 3 5 0 1 3 0 1 0 0 0 1 3 1 0 1 2 0 1 2 0 2 2 0 1 0
[21727] 0 1 0 0 0 3 2 0 0 0 0 0 1 1 0 0 0 2 0 0 1 0 0 3 2 0 3 0 0 0 0 0 0 0
[21761] 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 2 0 0 2 0 1 1 0 1 1 0 1 1 0 0 0 0 0 0
[21795] 1 0 0 3 0 0 0 0 0 0 0 0 2 0 1 1 0 2 1 1 0 1 2 3 0 1 1 2 1 0 0 0 0 0
[21829] 1 0 4 3 3 0 0 0 0 0 1 0 0 0 0 0 0 1 1 1 0 0 0 0 0 0 0 1 0 1 0 0 1 0
[21863] 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 1 1 0 0 0 1 1 0 0 0 0 0 1 0 2 2 0 1 0
[21897] 0 0 1 1 0 1 1 1 1 1 0 0 0 0 0 0 0 1 0 0 1 0 1 1 0 0 0 0 0 0 0 0 1 0
[21931] 1 0 0 1 0 0 0 0 0 0 1 1 0 1 2 2 3 0 0 0 0 0 0 0 0 0 0 0 2 1 0 0 0 0
[21965] 0 1 0 2 1 0 1 0 1 0 1 2 0 0 1 0 2 1 1 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0
[21999] 1 0 0 0 0 0 0 0 0 0 1 1 1 0 3 0 1 0 0 1 3 1 1 1 0 0 0 0 0 0 0 0 0 0
[22033] 0 1 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 1 0 0 2 0 0 1 0 1 2 2 0 2 0 0 2
[22067] 1 3 0 0 2 1 1 4 0 3 1 2 1 2 1 0 2 1 0 0 0 0 0 0 0 1 2 0 0 0 0 0 0 0
[22101] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 1 0 0 0 0 2 1 2 1 1 3 1 1 1 2 0
[22135] 2 0 0 1 1 2 1 1 2 0 0 1 0 1 1 0 1 1 0 0 2 2 0 0 0 1 0 1 1 1 0 1 1 0
[22169] 0 0 2 0 2 1 1 1 2 1 0 0 0 0 0 2 0 1 0 1 1 0 0 1 0 1 3 1 1 0 1 0 1 0
[22203] 2 1 0 0 0 3 1 0 0 0 0 1 2 3 0 1 1 1 0 0 0 0 0 2 1 1 1 2 3 2 0 3 1 3
[22237] 3 4 4 2 2 0 3 3 1 1 1 2 3 0 0 2 0 1 1 0 1 2 2 0 0 0 0 0 1 0 1 0 0 0
[22271] 0 0 0 0 0 0 0 0 1 0 1 1 1 1 1 0 1 0 1 0 0 1 2 0 0 1 2 2 0 1 0 0 0 0
[22305] 2 5 0 5 1 3 4 1 1 1 0 0 3 0 0 0 0 0 2 1 1 2 0 0 2 0 0 0 0 0 0 1 0 0
[22339] 0 0 1 0 1 1 0 0 2 3 1 0 1 1 0 0 0 1 0 0 1 0 1 0 0 1 0 0 2 2 1 1 1 1
[22373] 3 2 2 3 2 0 3 0 0 0 0 0 2 1 1 0 0 0 0 0 0 1 0 0 0 1 0 0 0 1 1 0 0 0
[22407] 0 1 2 1 0 0 0 0 1 2 2 1 1 2 1 0 1 0 0 1 0 1 1 0 1 1 1 0 0 0 1 1 1 0
[22441] 2 0 2 2 1 3 4 1 0 1 0 0 2 2 0 2 0 0 2 1 0 0 1 1 0 1 1 0 2 1 1 2 3 1
[22475] 4 3 1 3 2 3 3 1 0 0 2 0 0 1 1 2 1 2 0 0 0 0 0 1 2 1 0 0 2 0 0 0 0 1
[22509] 0 1 0 0 2 0 0 0 2 1 1 2 0 2 0 0 1 2 1 0 1 0 1 1 0 0 0 0 1 0 1 1 3 0
[22543] 2 0 0 0 0 0 0 0 0 0 0 1 4 0 0 1 0 0 0 0 0 0 2 0 1 0 1 1 2 1 2 0 2 1
[22577] 2 0 0 1 0 1 0 0 1 1 0 1 0 0 1 2 2 2 1 0 0 0 0 4 0 0 0 2 0 0 0 0 0 0
[22611] 1 0 2 1 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 1 2 1 0 0 2 3 0 0 0 4 1 1 1 0
[22645] 0 0 0 0 0 0 0 0 0 0 0 0 3 1 0 0 0 2 1 1 0 1 0 1 1 3 0 1 0 1 2 1 2 4
[22679] 3 1 2 0 0 1 2 0 1 0 2 3 0 0 0 0 0 0 0 0 0 1 1 1 0 2 1 0 2 4 0 1 1 2
[22713] 3 0 1 0 3 2 1 0 4 1 2 1 1 0 1 1 1 0 0 2 1 0 1 3 1 2 2 0 0 0 0 0 0 0
[22747] 0 0 2 0 0 1 1 1 0 1 0 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 2 1 1 0 0 0 0 0
[22781] 1 0 0 0 0 0 0 1 0 1 2 0 0 2 0 0 0 2 0 0 0 0 0 0 1 0 0 1 1 0 0 1 0 0
[22815] 1 0 0 0 1 0 0 0 2 1 0 0 0 0 0 0 1 0 2 1 0 0 1 1 0 0 1 2 2 1 0 0 0 0
[22849] 0 1 0 3 1 0 0 2 1 1 0 0 1 0 1 0 1 0 1 0 1 2 1 0 0 0 2 0 0 0 0 0 0 0
[22883] 0 0 0 0 0 0 0 0 0 0 0 0 2 2 1 1 0 0 0 0 0 0 0 3 1 1 1 2 4 1 1 1 0 1
[22917] 0 0 0 1 2 2 0 1 0 1 1 1 1 1 0 1 0 1 0 0 0 0 1 0 2 0 0 2 0 1 1 0 3 1
[22951] 4 1 1 1 1 0 1 0 2 2 0 0 1 0 0 1 2 1 0 0 1 0 2 0 0 1 1 1 0 1 0 0 0 0
[22985] 1 0 2 0 1 0 1 0 0 2 0 0 0 0 0 0 1 0 1 0 1 3 1 1 1 1 1 2 1 2 2 0 4 4
[23019] 0 3 0 2 0 2 1 1 2 2 2 1 3 1 2 2 2 2 1 1 1 0 2 2 1 1 3 1 0 0 0 0 0 0
[23053] 0 0 0 0 0 0 1 1 0 2 2 1 1 1 0 0 1 1 0 1 1 4 0 2 1 0 0 0 3 0 1 0 0 0
[23087] 0 0 0 1 3 1 2 0 0 0 0 0 0 1 1 1 0 0 0 2 1 0 0 0 2 1 1 1 0 0 0 0 1 0
[23121] 0 0 1 3 2 0 1 2 1 0 0 0 0 0 2 0 0 0 2 0 1 0 2 0 0 0 0 2 2 0 0 0 0 3
[23155] 1 0 1 1 3 1 0 2 0 0 1 1 0 1 2 0 1 1 1 0 0 0 2 1 0 3 0 2 1 0 1 1 2 1
[23189] 0 0 0 2 1 0 1 2 0 0 3 3 0 0 1 0 0 2 0 0 0 0 4 0 0 0 0 0 2 2 0 0 0 0
[23223] 0 1 0 2 0 3 2 3 0 0 0 1 0 0 1 0 0 0 1 3 1 1 2 0 0 1 0 0 1 1 1 0 1 0
[23257] 1 0 2 2 1 2 1 1 0 0 0 2 1 0 0 0 1 1 1 3 2 0 1 0 0 1 1 2 0 2 1 1 1 2
[23291] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 3 0 1 3 1 4 0 0 2 2 1 1 0 0 1 2 2 4 2 1
[23325] 0 1 0 0 0 1 1 1 0 2 1 0 0 2 0 0 0 0 1 1 1 2 1 0 0 2 1 3 0 0 1 0 1 1
[23359] 1 3 1 0 0 1 0 1 1 0 1 0 0 1 1 2 2 0 0 0 0 0 0 0 0 3 1 1 2 1 3 2 1 2
[23393] 0 0 0 1 2 0 2 0 0 2 1 1 0 0 1 0 1 2 2 1 0 2 2 0 7 0 1 1 1 1 1 0 0 1
[23427] 0 1 0 0 1 0 1 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0 0 3 0 2 0 0 2 3 7 2 3
[23461] 0 1 0 0 0 0 0 0 0 0 0 1 1 5 1 4 5 2 3 0 0 0 0 0 0 0 0 0 0 0 0 2 2 2
[23495] 4 2 2 0 0 0 0 0 2 1 0 1 3 2 2 1 1 4 0 2 0 2 2 2 1 3 0 0 0 0 0 0 0 0
[23529] 0 0 0 0 2 1 1 1 1 0 0 0 0 2 0 0 0 0 1 2 0 1 2 1 1 0 0 0 0 0 1 1 1 1
[23563] 0 0 0 0 1 1 1 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[23597] 0 0 1 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 1 0 0 2 1 2 0 0 0 0 1 0 0 1 1 1
[23631] 1 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 2 2 0 0 0 0 1 2 4 1 1 0 0 0
[23665] 1 1 0 0 0 0 2 0 3 2 0 1 1 1 1 0 2 2 0 0 0 0 3 3 0 1 2 1 0 0 0 0 0 0
[23699] 2 0 0 3 1 1 0 0 0 2 0 2 0 0 0 0 0 0 1 0 0 0 0 0 0 0 3 0 1 1 0 1 0 0
[23733] 0 3 0 3 2 1 0 2 1 0 0 0 0 0 1 3 0 0 0 0 0 1 1 1 0 1 0 2 1 0 0 1 0 0
[23767] 0 0 0 0 0 2 0 0 2 0 1 0 1 1 0 1 2 0 2 1 1 1 0 2 0 1 1 0 0 0 0 0 0 0
[23801] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 1 1 0 1 0 1 1 0 1 1 0 2 2
[23835] 1 0 0 0 0 0 0 0 1 0 0 0 0 0 0 2 1 1 0 0 0 1 1 2 0 0 0 0 2 1 2 1 0 0
[23869] 0 0 0 0 2 3 1 0 2 1 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1
[23903] 0 1 0 0 1 1 1 0 0 2 0 0 0 2 0 0 0 1 0 0 0 1 0 0 1 1 0 1 1 0 0 0 1 0
[23937] 2 1 1 0 0 0 1 3 0 0 0 2 0 1 0 1 0 0 0 2 0 1 0 2 1 0 0 0 1 1 0 0 2 2
[23971] 1 0 0 0 0 0 0 1 2 2 0 1 0 0 0 2 3 2 2 3 0 1 0 0 0 0 0 1 0 1 2 3 1 0
[24005] 1 3 2 2 3 2 1 1 0 1 0 0 1 2 0 0 2 0 2 0 3 2 4 0 2 1 0 0 0 0 0 0 0 0
[24039] 1 1 1 0 1 0 0 1 3 2 1 1 0 1 2 0 1 3 3 1 2 2 0 4 0 1 1 0 1 0 0 3 0 1
[24073] 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 1 0 0 1 0 1 0 0 0 0 0 0 1 1 1 1 0
[24107] 2 3 1 3 2 0 0 1 2 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 5 0
[24141] 2 1 2 4 3 2 3 0 1 0 1 1 2 0 0 0 0 1 2 2 0 1 1 2 0 3 1 0 0 1 1 2 0 0
[24175] 0 0 0 0 0 0 2 0 1 0 0 0 0 0 0 2 0 0 0 0 1 0 0 1 1 2 2 4 0 0 0 0 4 1
[24209] 4 2 0 0 2 1 2 1 3 4 3 0 1 0 0 1 0 1 1 0 1 0 2 1 0 0 0 0 1 1 0 0 0 0
[24243] 0 0 0 0 0 0 0 0 0 0 2 1 0 0 1 0 2 1 1 0 0 0 0 1 0 1 1 1 2 0 0 1 0 2
[24277] 2 1 0 0 1 1 0 0 1 0 0 0 2 3 1 0 0 0 0 1 4 2 0 0 0 0 0 0 0 0 1 1 0 0
[24311] 2 0 0 0 0 0 2 2 1 2 0 0 1 0 1 0 1 0 2 0 1 2 0 0 0 1 0 0 2 2 3 2 4 0
[24345] 1 4 1 1 3 1 2 5 0 1 0 2 0 1 1 0 0 1 0 0 0 0 0 0 0 2 0 0 0 1 0 0 1 0
[24379] 1 0 0 0 0 0 0 0 1 2 1 0 0 0 2 0 1 2 2 0 1 0 1 0 1 0 0 0 1 0 0 1 0 2
[24413] 2 0 0 0 2 0 0 0 0 0 0 1 1 0 0 0 1 0 1 0 0 0 0 0 0 1 1 2 0 1 0 0 0 0
[24447] 3 0 2 0 3 1 1 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1 0 3 2 0 0 2 0 0 0 1 0 0
[24481] 1 2 2 0 0 0 0 0 1 0 1 1 0 0 0 0 1 0 0 0 0 3 2 0 0 0 1 0 1 0 0 0 1 1
[24515] 0 1 0 1 0 0 1 1 0 0 0 1 0 0 0 0 0 0 1 2 0 1 1 1 1 0 2 0 2 0 0 0 0 0
[24549] 0 0 0 1 0 1 0 0 0 0 0 0 0 0 1 1 0 1 0 1 3 1 0 0 2 1 3 0 3 1 1 1 1 1
[24583] 2 3 3 0 0 1 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 0 1 1 1 1 0 0 0 0 0 0 0
[24617] 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0 0 0 1 2 1 2 3 3 1 3 2 2 1 1 0
[24651] 2 4 0 3 1 0 1 0 1 4 0 0 0 0 0 0 3 1 0 4 2 1 3 4 2 3 0 1 0 1 1 2 3 2
[24685] 0 1 0 1 0 1 0 0 0 1 3 5 1 0 0 2 1 1 2 1 1 2 0 1 0 0 1 1 0 1 1 0 0 1
[24719] 1 0 1 0 1 0 1 0 2 0 0 0 1 0 1 0 1 1 1 3 1 0 0 0 1 1 2 1 1 1 0 0 0 0
[24753] 0 0 0 1 0 0 0 1 1 0 1 0 1 0 0 0 0 0 1 1 0 1 0 2 2 1 1 3 0 0 0 1 0 1
[24787] 0 0 0 0 0 0 0 0 0 0 3 1 2 2 0 2 0 0 1 0 0 0 0 0 0 0 0 1 3 2 2 1 1 0
[24821] 1 3 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 3 0 2 0 0 0 0 0 3 1 0 0 0 0 0
[24855] 0 2 1 0 0 4 1 1 1 1 3 4 1 0 2 0 0 0 3 0 2 1 0 1 0 0 0 0 1 0 1 0 0 0
[24889] 4 1 0 0 0 1 1 2 0 3 4 2 3 2 0 2 2 1 0 0 1 1 0 2 0 0 0 0 0 0 1 0 1 1
[24923] 0 0 3 1 1 0 0 0 0 0 0 0 0 1 1 0 1 1 0 0 2 1 3 0 0 0 1 1 1 2 0 0 0 0
[24957] 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0 1 0 0 1 1 2 3 1 4 0 2 0 0 0 1 1 1
[24991] 0 0 0 1 1 2 1 0 0 0 0 0 0 0 0 1 2 0 1 0 0 0 0 0 0 0 0 0 1 0 1 2 0 0
[25025] 0 1 0 0 0 0 0 0 0 0 1 2 0 4 1 2 0 0 1 0 1 1 1 1 1 3 2 1 2 0 0 0 0 2
[25059] 0 0 0 0 0 0 0 4 0 0 0 0 0 0 0 0 0 0 0 0 0 1 2 1 0 0 1 1 0 0 0 0 0 0
[25093] 0 0 3 1 0 0 0 0 0 2 3 0 0 0 0 0 0 0 2 0 0 0 2 1 0 1 1 2 0 2 0 1 2 0
[25127] 2 0 1 1 2 3 1 0 2 0 0 0 1 0 0 0 0 0 0 2 0 1 0 0 0 0 0 1 0 2 3 0 0 3
[25161] 0 2 0 1 1 1 0 0 0 0 0 0 1 0 0 1 2 0 1 1 2 0 0 3 2 0 0 3 2 3 2 1 1 1
[25195] 2 2 2 0 1 2 1 1 1 1 1 3 1 1 2 1 0 0 1 0 1 0 0 0 0 1 1 1 0 0 0 2 1 0
[25229] 0 1 0 0 2 0 0 0 0 0 0 0 0 2 0 1 1 0 0 0 0 0 0 0 0 1 0 2 0 1 2 1 0 1
[25263] 0 2 0 0 0 0 0 0 0 1 0 0 3 0 0 0 0 0 2 0 0 1 1 0 0 2 0 1 1 0 0 2 0 0
[25297] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 1 0 2 2 0 1 1 2 0 0 0
[25331] 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 1 0 1 0 0 0 1 0 2 1 1 1 0 1 2
[25365] 2 1 1 0 1 1 0 1 0 0 0 1 2 0 0 0 0 0 1 1 1 0 0 0 1 0 0 2 1 0 0 0 1 0
[25399] 1 0 0 0 1 1 1 3 0 0 1 0 0 1 0 1 0 1 2 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0
[25433] 0 0 2 1 1 1 0 0 0 1 1 2 1 1 0 0 0 0 0 1 1 1 1 0 1 0 1 0 0 0 0 3 0 0
[25467] 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 1 2 0 0 1 1 0 0 1 0 0 0 0 0 0 0 1 1 0
[25501] 0 0 0 0 1 0 0 1 3 0 0 0 1 0 0 1 0 1 2 1 1 1 1 0 2 1 0 0 0 0 1 0 0 0
[25535] 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1
[25569] 0 0 0 0 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 4 0 0 0 0
[25603] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[25637] 1 0 0 3 0 0 0 0 0 0 0 2 0 1 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0
[25671] 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[25705] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0
[25739] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[25773] 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0
[25807] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[25841] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[25875] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[25909] 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[25943] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0 0 0 0
[25977] 0 0 0 0 0 0 1 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 1 0 3 0 0 0
[26011] 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[26045] 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[26079] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[26113] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[26147] 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[26181] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[26215] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[26249] 0 1 0 1 1 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0
[26283] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[26317] 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[26351] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0
[26385] 0 0 1 0 0 0 2 0 1 0 1 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0 0
[26419] 0 0 0 0 1 0 0 0 0 0 2 1 0 0 0 0 0 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0
[26453] 2 1 1 3 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 1 2 0 1 2 0 0 1 0 2 0 0 2 0 0
[26487] 0 1 0 0 1 1 0 0 0 1 1 0 0 1 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 1 0 0
[26521] 0 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0 1 1 0 1 1 0 0 0 0 0 3 0 2 0 0
[26555] 0 2 3 2 0 0 0 1 0 0 0 0 2 1 0 1 0 0 0 0 1 0 0 2 1 0 0 0 1 1 0 0 1 0
[26589] 0 1 1 1 1 0 0 0 0 0 0 0 1 0 0 0 1 0 0 2 2 0 1 0 0 0 0 0 1 0 0 0 1 1
[26623] 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 1 1 0 0 0 0 0
[26657] 0 0 0 0 1 0 1 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 1 0 1
[26691] 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 1 0 1 0 1 0 1 0 0
[26725] 0 1 2 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 0 0 0 0 0 0 0 1
[26759] 0 1 0 1 1 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0 0 0 0 0 1 0 0 1 0 0 0 0 0 0
[26793] 0 0 0 1 0 1 1 0 0 0 0 1 0 0 0 0 0 1 3 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0
[26827] 1 0 0 0 0 0 0 0 0 1 1 0 0 1 1 1 0 0 1 0 1 0 0 0 0 0 0 1 1 0 0 0 0 0
[26861] 1 0 0 0 0 1 1 0 0 0 1 0 1 1 0 1 1 0 0 0 0 0 0 1 1 1 0 0 0 0 0 0 0 0
[26895] 1 0 0 0 0 0 0 0 0 0 1 0 0 0 1 1 0 1 0 0 0 0 0 1 0 0 0 0 0 1 0 1 0 0
[26929] 0 0 0 0 1 0 0 0 0 0 0 0 0 2 1 0 0 0 0 0 0 0 0 0 0 0 1 0 2 1 0 1 2 0
[26963] 0 2 0 0 0 0 1 0 1 0 1 0 0 2 1 0 1 0 1 0 0 0 2 0 1 0 1 1 0 1 1 1 0 0
[26997] 0 0 0 1 0 1 0 0 1 0 0 0 0 2 0 1 0 1 0 0 0 0 0 0 0 1 1 0 1 0 0 0 0 1
[27031] 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 1 0 1 1 0 0 0 0 0 0
[27065] 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 1 0 0 0 1 0
[27099] 0 0 0 0 0 0 0 1 0 0 0 0 2 0 0 0 0 0 1 0 0 1 0 0 2 0 0 0 0 1 0 0 1 0
[27133] 0 0 0 0 1 1 2 1 1 0 0 0 0 1 0 1 0 0 0 0 0 1 0 0 2 0 0 0 0 1 0 1 1 0
[27167] 1 1 1 1 2 0 0 1 0 0 0 0 1 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 1 0
[27201] 1 1 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0
[27235] 0 1 0 0 1 0 2 0 2 0 1 1 0 0 0 0 0 1 0 0 0 0 1 0 1 1 0 0 0 0 0 0 1 0
[27269] 1 1 1 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 1 1 0 1 2 0 0 1 2 0 1 0 0
[27303] 0 0 1 4 1 0 1 2 1 1 0 1 0 0 2 1 0 0 0 0 0 1 1 0 0 0 0 0 0 0 1 0 0 0
[27337] 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0 2 0 0 1 0 0 0 1 1 1 1
[27371] 1 0 0 1 0 0 0 0 0 1 0 0 0 0 0 0 2 1 0 1 1 0 1 0 0 0 0 2 1 0 0 0 0 0
[27405] 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0
[27439] 1 0 0 0 0 0 0 0 0 1 0 1 1 0 0 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 1 1 0 0
[27473] 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 1 1 1 1 0 0 0 0 1 0 0
[27507] 0 0 0 0 0 0 0 1 0 0 1 0 1 1 0 0 0 0 0 1 0 0 0 0 1 1 0 2 0 0 0 0 0 0
[27541] 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[27575] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 1 0 0 0 0 0 1 0 1 1 0 1
[27609] 0 1 1 0 1 1 1 0 2 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0
[27643] 0 1 0 1 0 0 0 0 1 1 1 0 0 2 1 0 1 1 0 0 2 0 2 0 0 0 1 0 0 0 0 0 0 1
[27677] 1 0 0 0 0 0 1 0 1 0 0 1 1 0 0 1 0 0 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 1
[27711] 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 1 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0 1 0
[27745] 0 0 0 0 0 0 0 0 1 0 2 0 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0
[27779] 0 0 0 0 1 0 1 0 0 0 0 0 1 0 0 1 1 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0
[27813] 1 0 0 0 0 0 1 0 0 0 0 1 0 0 1 1 0 0 0 1 1 0 0 0 0 0 0 1 1 0 0 0 0 0
[27847] 0 0 0 0 0 1 0 0 0 0 1 0 0 0 0 0 1 1 0 0 2 0 0 0 0 2 0 0 0 1 0 1 0 0
[27881] 0 1 0 0 0 3 1 1 2 0 0 2 1 1 0 0 2 0 0 0 1 1 1 0 0 0 1 1 2 1 1 1 0 2
[27915] 0 0 1 0 0 0 0 0 2 2 0 1 3 2 0 1 2 2 2 0 0 0 1 0 0 2 1 2 0 2 1 1 1 1
[27949] 0 1 0 2 0 0 0 0 1 0 0 0 1 0 1 0 0 1 0 0 0 0 0 0 2 0 0 2 0 0 0 0 0 0
[27983] 0 0 1 0 0 2 0 0 0 0 1 0 0 0 0 0 1 1 0 0 0 0 0 2 0 1 0 0 0 0 1 1 0 0
[28017] 0 0 0 1 0 1 3 0 1 1 2 0 1 1 1 0 0 3 1 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0
[28051] 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0
[28085] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0 1 0 1 2 2 1 0 0 0
[28119] 0 0 1 1 0 0 1 0 0 2 2 0 0 2 2 3 0 2 0 0 2 2 1 0 0 0 2 0 1 1 0 0 0 0
[28153] 1 0 0 1 0 0 0 1 1 0 0 2 0 2 1 2 0 1 2 1 1 0 2 0 1 2 3 0 0 1 0 1 0 0
[28187] 0 2 0 0 0 2 2 0 0 0 0 0 0 3 1 2 0 0 2 0 0 1 3 0 1 0 2 2 0 0 0 1 0 2
[28221] 1 1 2 1 1 0 3 1 0 1 2 0 0 1 1 1 0 0 0 1 0 2 0 0 2 0 0 0 0 0 0 0 0 0
[28255] 0 0 0 0 0 0 0 1 1 0 0 1 0 0 3 1 0 0 1 0 1 0 2 0 0 0 1 0 1 1 0 0 0 0
[28289] 1 0 0 0 0 1 0 0 0 0 0 0 1 1 0 0 1 0 0 0 0 1 0 0 0 1 0 1 1 1 1 0 0 0
[28323] 1 0 0 0 0 2 0 0 0 2 0 1 2 0 1 2 2 1 3 0 0 0 0 1 0 5 0 1 0 0 1 0 0 0
[28357] 0 1 0 0 1 1 1 1 0 1 0 1 0 4 2 2 3 0 1 2 2 0 2 0 0 0 0 0 0 1 1 0 0 0
[28391] 0 0 1 0 1 0 1 0 2 1 0 1 1 0 1 1 0 0 1 1 0 1 0 1 1 3 0 1 1 0 1 0 0 0
[28425] 0 1 0 0 0 0 0 0 2 1 0 0 2 1 1 0 0 0 0 0 0 2 1 0 1 2 1 0 1 1 0 0 1 0
[28459] 0 1 1 0 0 2 0 2 0 2 0 1 0 0 2 1 1 0 0 0 0 0 0 0 0 0 1 1 0 1 1 1 0 0
[28493] 0 2 0 0 0 0 0 0 0 0 1 1 0 1 1 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0
[28527] 0 0 0 2 0 0 1 0 0 1 0 0 0 0 0 0 0 0 0 0 0 1 1 0 1 1 0 0 0 0 0 1 0 0
[28561] 0 0 0 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0
[28595] 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 1
[28629] 0 0 1 0 1 0 1 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 1 1 0
[28663] 0 0 0 0 0 1 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 1 1 1 0 0 0 0 0 0
[28697] 0 0 1 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 2 0 0 0 0 0 1 0 0 0 0 0 0
[28731] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0
[28765] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0
[28799] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[28833] 0 0 0 0 0 0 0 0 1 0 1 2 0 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[28867] 1 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 1 0 2 0 1 0 0 0 0 0 0 0 1 0 0
[28901] 0 0 1 1 2 1 0 0 0 0 0 0 0 0 0 2 0 1 1 0 0 0 2 2 0 0 1 0 0 0 0 0 2 0
[28935] 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 1 1 2 0 1 0 0 0 0 0 0 0 0 1 1 0 0 0
[28969] 0 0 0 1 0 1 0 0 0 0 0 0 1 0 1 0 0 1 0 0 1 0 1 1 1 0 1 1 0 0 0 1 0 1
[29003] 2 1 1 0 1 0 0 1 0 0 1 1 0 0 0 0 0 0 0 0 1 0 1 1 1 0 0 2 1 1 0 2 0 0
[29037] 0 0 0 0 0 0 0 2 1 0 1 0 1 1 3 0 0 0 1 0 0 1 0 1 0 2 2 1 0 1 2 3 0 0
[29071] 0 0 0 0 1 1 0 0 0 0 0 0 1 0 0 1 0 0 1 2 1 0 0 0 0 0 1 0 1 0 1 0 0 0
[29105] 3 1 1 0 0 0 0 1 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 2 0 1 1 0
[29139] 0 0 0 0 0 0 1 1 0 0 0 1 0 1 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 0 0 0 1 1
[29173] 0 1 1 0 2 0 1 1 0 1 0 0 0 0 0 2 0 1 0 1 1 1 0 0 2 0 0 0 0 0 0 0 1 1
[29207] 0 0 0 1 1 0 0 0 1 1 0 0 0 0 0 0 1 2 0 0 0 0 0 1 0 1 0 0 1 0 0 0 1 0
[29241] 0 0 0 1 0 2 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[29275] 0 0 0 0 0 0 0 0 0 0 0 0 3 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0
[29309] 0 0 0 0 1 0 0 0 1 0 0 0 0 1 0 0 0 0 0 1 0 1 0 3 0 0 0 0 0 0 0 0 0 0
[29343] 0 0 0 0 0 0 0 1 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[29377] 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 2 1 0 2 1 0 0 0 0 0 0 0 0 0 0 0 1 2 0
[29411] 1 0 2 1 2 1 1 0 1 0 0 0 0 0 0 0 0 2 0 0 0 0 0 0 0 1 1 2 0 0 0 0 0 0
[29445] 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 1 0 0 1 1 1 0 0 0 0
[29479] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0
[29513] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0 0 0 1 0 0 0 1 0 0
[29547] 1 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0 2 0 1 0 1 0 0 1 0 0 0 0 0 0 0 0 0
[29581] 0 0 0 0 1 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[29615] 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1
[29649] 1 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 1 2 0 0
[29683] 0 0 0 0 1 0 1 1 0 0 2 0 0 0 0 1 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0 0 1 0
[29717] 0 2 2 0 1 0 0 0 0 0 1 0 0 0 0 0 1 1 0 1 0 0 0 1 0 0 0 0 1 0 0 0 0 0
[29751] 0 0 1 0 0 0 0 0 1 0 0 0 0 0 1 0 0 0 0 1 0 0 0 0 0 0 0 1 0 0 0 0 0 0
[29785] 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0
[29819] 0 0 0 0 0 1 1 0 0 0 1 0 0 0 0 0 0 1 0 0 0 1 0 2 0 0 0 0 0 0 0 0 0 0
[29853] 0 1 0 0 0 0 0 0 2 0 0 0 1 0 0 0 0 0 0 1 1 0 1 0 0 0 1 0 0 0 1 0 1 0
[29887] 0 0 1 2 0 0 1 0 1 0 1 0 1 0 0 1 0 1 1 1 0 1 1 0 0 1 0 0 0 0 0 0 0 0
[29921] 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 3 1 1 1
[29955] 0 0 1 0 2 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0
[29989] 1 1 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 1 2 0 1 0 1
[30023] 0 0 1 0 1 0 0 0 0 1 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 3 0 2
[30057] 1 0 0 0 0 0 1 1 0 0 0 0 1 2 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0 1 1 0 0 1
[30091] 1 0 0 0 0 0 0 1 0 0 0 0 0 1 1 0 0 0 0 0 0 2 1 0 1 0 0 0 3 0 1 1 0 0
[30125] 1 0 1 0 1 1 0 0 1 0 0 0 0 2 1 2 3 0 1 1 0 1 1 1 1 0 1 0 1 0 1 0 0 0
[30159] 2 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 1 0 0 2 0 0 0 0 1 1 1 0 0 1 0 0
[30193] 1 0 0 0 0 0 0 2 1 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 1 0 2 0
[30227] 0 1 0 0 0 0 0 0 0 0 2 0 0 0 0 2 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0
[30261] 2 0 0 0 0 2 2 0 0 1 1 0 0 0 0 0 0 2 0 2 1 1 0 1 0 0 1 0 0 0 0 1 0 0
[30295] 0 0 0 0 0 1 0 1 0 0 0 1 0 0 0 0 1 1 2 0 1 0 0 1 1 0 0 0 0 0 2 1 0 0
[30329] 0 1 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 1 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0
[30363] 1 0 2 1 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 2 0 0 0 2 0 0 0 0 1 0 1 0 2
[30397] 0 0 0 1 0 1 2 1 2 1 0 0 0 0 1 0 0 2 2 1 0 1 0 0 0 0 0 0 0 1 0 1 1 0
[30431] 0 0 0 0 0 0 1 0 1 0 1 0 1 0 0 0 0 0 0 1 0 1 2 1 1 0 1 0 0 1 1 1 0 1
[30465] 0 0 0 1 0 1 0 0 1 1 0 0 0 1 0 0 0 0 1 0 0 0 1 0 0 1 0 0 0 1 1 2 0 0
[30499] 1 0 0 0 1 0 1 0 1 1 0 0 0 0 0 0 0 0 0 1 2 0 1 1 1 0 0 0 1 1 1 1 0 0
[30533] 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 1 1 0 0 0 1 0 0 0 0 0 0 0 2 1
[30567] 4 0 0 1 1 1 1 0 0 0 0 0 1 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 2 0 0 1 0
[30601] 0 0 0 0 0 1 2 0 1 1 0 0 0 1 1 0 0 0 0 0 0 1 1 1 0 1 0 0 0 0 0 0 0 0
[30635] 0 1 0 0 0 0 1 0 0 2 1 0 1 1 3 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[30669] 0 0 0 0 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 0 0 0 0 2 0 0 1 0 0 0 0 1 0 0
[30703] 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 1 0 0 0 0 0 2 0 0 0
[30737] 0 1 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 2 0 0
[30771] 0 1 0 0 2 2 0 1 0 1 0 1 0 2 0 1 0 0 0 1 0 0 0 0 2 3 0 0 0 0 0 0 0 0
[30805] 0 0 0 0 0 0 1 1 0 1 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 2 0
[30839] 0 0 1 0 1 0 0 1 2 0 0 0 0 2 0 1 0 0 2 0 1 1 0 1 1 0 0 0 0 0 2 0 4 0
[30873] 0 0 1 2 0 0 0 1 0 0 0 1 0 1 0 0 0 0 0 0 1 0 0 0 1 1 0 0 0 0 0 2 0 0
[30907] 1 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0
[30941] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0 2 0 0 0 0
[30975] 0 0 0 0 0 0 1 0 0 0 2 1 1 0 0 1 0 1 1 1 0 2 0 0 0 0 0 0 1 2 0 1 0 0
[31009] 0 0 0 0 1 0 0 0 1 0 0 0 0 1 0 0 0 1 1 2 1 0 0 0 0 0 0 0 0 3 0 0 1 0
[31043] 0 0 0 0 0 0 0 0 0 1 2 1 0 0 0 0 0 0 0 0 0 0 1 0 0 0 1 0 1 0 0 0 0 0
[31077] 3 0 1 0 0 1 1 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0 0 1 0 1 1 0 0 1 0 1 0
[31111] 1 0 0 0 1 0 0 1 1 1 0 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 3 0
[31145] 0 1 0 0 0 0 0 0 1 1 1 2 0 0 0 0 1 1 0 1 0 1 0 0 0 0 1 0 0 0 0 0 1 0
[31179] 1 0 1 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 2 0 0 0 1 0 0
[31213] 0 0 0 0 0 0 0 2 1 0 0 1 0 0 0 0 0 1 0 0 1 1 2 0 0 0 0 0 0 0 0 0 0 0
[31247] 0 0 0 0 0 0 1 1 0 1 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0
[31281] 1 0 1 0 0 0 1 0 0 1 0 0 0 0 0 0 1 0 0 0 1 0 1 1 0 0 0 1 1 0 0 0 0 0
[31315] 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 1 1 0 0 0 0 0 0
[31349] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 1 1 1 0 0 0 0 0
[31383] 0 0 0 0 0 0 0 1 0 0 0 0 1 0 2 0 1 0 1 0 0 1 0 1 0 0 0 2 0 0 0 1 1 0
[31417] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0
[31451] 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 0 0 1 1 0 1 0 0 0 0 0 0 0 0 1 1 1
[31485] 0 1 0 0 0 0 0 0 0 0 1 0 1 0 2 0 0 0 0 1 1 1 0 1 1 1 0 1 0 1 0 0 1 0
[31519] 1 1 0 1 0 1 1 0 0 2 0 0 1 0 0 1 0 1 0 2 0 0 0 0 0 0 2 0 1 0 2 1 0 0
[31553] 0 0 0 0 0 0 0 0 1 0 1 0 1 2 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0
[31587] 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 0 0 0
[31621] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 1 0 0 0 0 0 0 0 1 0 1
[31655] 0 0 1 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 1 0 2 1 1 0 0 0 0
[31689] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 2 0 0 0 0 0 1 0 0 0
[31723] 1 1 1 0 0 0 0 0 1 0 2 0 3 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0
[31757] 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1 1 0 1 0 0 1 0 0 0 0 0 0
[31791] 0 0 1 0 0 0 2 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0
[31825] 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0 0 0
[31859] 1 1 0 0 0 0 0 0 0 1 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[31893] 0 0 0 1 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[31927] 0 0 0 0 0 2 0 0 0 0 1 0 0 0 0 0 0 0 0 1 0 3 0 1 0 0 0 0 0 0 0 0 0 0
[31961] 0 0 0 0 0 0 0 0 0 0 0 1 0 3 0 1 1 1 0 0 0 0 0 0 1 2 1 0 0 0 0 1 0 0
[31995] 1 0 0 1 0 2 0 0 0 0 0 2 2 0 0 0 0 0 0 1 0 1 1 0 0 0 0 0 1 0 0 1 0 0
[32029] 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 0 0 0 1 0 0 1 0 1 0 0 0
[32063] 0 0 0 0 0 1 0 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0 0 0
[32097] 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1
[32131] 0 0 0 0 1 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0
[32165] 0 0 0 0 0 0 1 0 0 1 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 1 0 1 0 0 0 0 1 0
[32199] 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0
[32233] 0 0 0 0 2 0 1 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 1 1 0 0 0 1 0 0 0 0 0
[32267] 0 2 0 0 1 0 0 0 0 1 0 0 0 0 1 0 0 2 1 1 0 1 3 0 0 0 0 1 0 0 0 0 0 0
[32301] 0 0 1 0 0 0 0 0 1 0 1 0 0 0 1 2 0 0 1 0 0 0 0 0 1 0 1 0 0 0 0 0 1 1
[32335] 1 1 0 0 0 0 0 0 0 0 0 0 0 0 1 2 1 2 1 4 0 0 0 1 1 4 1 1 1 3 0 1 0 2
[32369] 1 1 2 0 0 0 0 0 0 0 0 0 1 0 0 2 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[32403] 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0
[32437] 0 0 0 0 0 0 2 0 1 0 0 0 0 0 0 1 0 0 0 2 0 0 0 0 0 1 0 1 0 1 1 0 1 0
[32471] 0 0 0 2 1 0 0 0 2 0 2 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 1
[32505] 0 0 0 1 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 1 1 1 0 0 0
[32539] 0 0 0 0 0 0 0 1 0 0 1 2 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 1 1 0 0 0
[32573] 0 0 0 0 0 1 3 2 0 0 1 0 0 0 1 0 0 0 1 0 1 0 0 1 0 1 0 0 0 0 1 1 0 1
[32607] 0 0 1 0 0 1 0 0 0 0 0 0 0 1 1 0 2 0 0 1 0 1 0 2 0 0 0 1 1 0 1 0 1 3
[32641] 2 1 0 1 0 1 3 0 0 0 1 2 0 0 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 1 2 0 0 0
[32675] 0 0 0 0 0 0 0 0 0 2 0 1 0 0 1 0 1 0 0 0 1 0 0 0 1 1 1 0 0 1 1 0 1 4
[32709] 0 0 1 0 0 0 0 1 0 1 4 2 0 0 3 0 4 0 3 2 3 0 2 0 1 0 0 1 1 1 0 2 2 2
[32743] 0 0 1 0 3 2 1 0 0 5 5 0 4 3 3 1 2 3 2 1 1 1 0 0 0 1 0 1 1 0 1 2 0 1
[32777] 1 1 2 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 3 5 0 0 0 0 1 1 1
[32811] 0 2 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 1 3 0 1 1 0 0 0 0 0 3 2 0 2 2 1 2
[32845] 0 1 1 1 3 2 2 0 1 0 0 0 1 0 1 1 0 1 1 1 2 0 0 0 0 0 1 1 2 0 1 0 1 0
[32879] 2 0 1 1 0 0 1 0 0 0 2 0 0 0 0 0 0 0 0 0 0 1 0 0 3 1 0 0 0 0 0 0 0 0
[32913] 0 0 0 0 2 0 0 0 1 0 0 0 0 0 1 0 0 1 0 0 0 0 0 0 1 0 0 1 0 0 0 0 0 0
[32947] 0 0 0 0 0 0 0 0 0 1 0 1 1 1 0 1 1 2 1 0 1 2 1 4 1 2 2 0 1 0 1 1 3 2
[32981] 1 0 2 0 0 0 1 2 0 1 0 0 0 1 2 1 0 1 0 1 1 0 3 1 0 0 1 3 1 0 0 1 1 3
[33015] 0 0 1 1 1 0 0 0 1 0 1 1 0 2 0 0 1 1 1 0 2 0 0 0 0 0 0 0 0 0 0 0 2 0
[33049] 0 0 1 0 0 0 0 0 2 0 1 1 1 1 0 2 1 0 1 1 0 1 0 0 0 0 0 0 0 1 0 0 0 1
[33083] 0 1 1 0 0 0 0 0 0 0 0 0 0 0 2 1 1 0 0 0 1 0 0 0 0 0 0 0 0 1 0 0 1 0
[33117] 2 0 0 0 0 1 1 0 0 1 0 0 0 0 1 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0
[33151] 0 0 0 0 0 0 0 0 2 1 1 0 0 1 0 1 1 0 0 0 1 0 0 1 1 1 0 0 0 1 0 0 1 2
[33185] 1 3 1 0 0 0 3 1 0 0 1 1 2 0 3 1 0 1 0 0 0 0 0 0 3 1 0 0 0 0 1 0 0 0
[33219] 0 0 1 0 0 0 0 0 0 0 0 0 0 0 1 0 0 2 2 1 0 0 1 1 0 0 0 0 0 0 0 0 0 0
[33253] 0 1 1 0 1 0 1 0 0 0 0 1 1 1 0 1 0 0 0 0 1 1 1 0 0 0 0 0 0 0 1 1 0 1
[33287] 0 0 0 1 1 3 0 1 1 0 1 0 0 1 0 1 0 0 2 1 0 1 0 0 0 0 1 0 1 2 0 3 2 1
[33321] 0 1 0 0 0 1 0 1 1 1 0 0 0 1 2 2 0 1 1 1 4 1 0 1 0 0 1 0 0 0 0 0 1 0
[33355] 0 0 0 0 0 1 0 0 0 1 1 2 1 1 1 0 2 0 1 1 1 2 1 0 1 0 0 0 1 0 0 0 1 0
[33389] 0 0 0 0 0 0 0 0 0 2 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 4 0 1 0
[33423] 0 0 0 0 0 0 0 1 2 3 1 1 2 1 1 2 1 0 0 0 1 1 3 2 0 1 0 1 1 1 1 0 0 1
[33457] 0 0 1 0 1 0 0 3 0 0 2 3 3 1 1 3 2 0 0 2 4 0 1 0 1 0 0 1 0 1 0 0 0 1
[33491] 1 1 4 1 0 2 0 1 0 3 0 0 1 0 0 0 2 2 2 0 1 0 0 0 0 0 0 1 1 1 0 0 0 0
[33525] 1 0 1 1 1 0 0 0 0 0 0 1 0 0 0 1 1 0 1 1 0 0 1 1 1 0 0 1 2 1 0 0 2 0
[33559] 0 0 2 0 1 0 0 0 0 0 0 0 1 0 1 0 0 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1
[33593] 0 0 0 0 0 3 0 0 1 1 0 1 1 1 2 0 0 2 1 0 0 0 1 1 0 0 0 0 0 0 0 1 1 0
[33627] 1 0 0 1 0 0 1 0 0 0 0 0 0 0 0 0 2 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0 0 0
[33661] 0 0 0 1 0 1 0 0 0 0 0 1 0 0 1 0 0 1 1 0 0 0 1 0 0 0 0 0 1 0 0 1 0 0
[33695] 1 0 0 0 0 1 0 0 0 0 0 1 1 1 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 1
[33729] 1 0 0 1 2 0 1 0 1 0 0 0 0 1 0 2 0 0 0 2 0 0 0 1 0 0 0 0 0 0 0 0 0 0
[33763] 0 2 1 0 1 0 0 1 0 1 1 0 0 1 2 0 0 0 2 1 1 0 0 0 0 0 0 0 2 0 1 0 1 0
[33797] 0 0 1 0 0 0 0 0 2 1 0 2 1 0 0 2 2 2 2 1 0 1 1 1 4 0 0 1 1 1 3 0 1 1
[33831] 0 1 0 1 0 1 0 2 0 1 0 0 0 3 1 0 0 2 0 0 0 0 0 0 0 1 1 1 1 2 2 0 1 0
[33865] 0 0 0 0 3 0 0 0 1 0 0 1 0 4 1 1 0 2 1 2 0 3 0 2 2 2 1 0 1 2 1 2 2 1
[33899] 0 0 0 1 1 1 0 1 0 0 0 1 1 0 0 1 1 0 2 0 0 0 1 0 0 0 0 0 0 0 0 0 0 1
[33933] 0 0 0 0 1 0 1 0 0 0 0 0 1 0 0 0 0 0 1 1 2 2 2 1 0 1 0 0 0 1 0 0 0 0
[33967] 0 0 0 0 0 0 0 0 2 0 0 0 0 0 2 0 2 0 0 0 0 1 0 1 1 3 0 0 0 0 1 0 2 0
[34001] 1 0 0 1 0 1 0 1 1 1 1 0 1 3 0 1 0 0 0 0 0 0 0 0 0 1 0 0 0 1 0 1 0 0
[34035] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 0 1 1
[34069] 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0
[34103] 0 0 1 1 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 1 0 1 0 1 0 0 0 0 0 0 0 0 0
[34137] 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0 1 0 0 0 1
[34171] 0 0 0 0 0 0 0 0 1 0 1 0 2 0 1 1 2 0 0 0 2 1 0 0 0 0 2 0 0 1 2 0 0 0
[34205] 0 0 1 0 1 0 0 2 1 0 0 0 0 0 0 1 0 0 0 0 1 1 0 0 0 1 0 1 0 0 0 0 0 0
[34239] 0 0 0 0 0 0 2 1 0 0 0 1 0 1 1 0 0 2 0 1 1 0 0 0 0 0 0 0 0 0 1 0 0 0
[34273] 0 1 0 0 1 0 1 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 1 1 1 0 0 0
[34307] 0 0 0 0 0 0 0 0 0 0 0 0 3 1 0 2 0 0 0 0 0 1 0 0 0 0 0 0 0 1 1 0 0 0
[34341] 0 0 1 0 0 0 1 2 1 0 2 0 0 3 0 0 1 0 0 0 0 1 0 0 0 1 1 0 0 0 0 1 1 1
[34375] 2 0 1 1 1 0 0 1 0 0 0 0 0 0 0 0 1 0 1 0 1 1 0 1 0 1 0 1 0 1 0 0 2 1
[34409] 0 2 0 1 0 0 0 1 0 0 1 0 0 0 0 0 1 1 1 1 1 0 0 0 0 0 0 0 0 0 0 1 0 1
[34443] 0 1 0 0 3 0 1 0 0 1 0 0 0 1 3 0 1 0 0 3 3 0 0 1 1 1 1 1 0 0 0 0 0 0
[34477] 0 0 0 0 0 0 0 1 1 0 0 1 0 0 2 2 1 0 0 0 0 0 0 0 1 0 1 0 1 1 0 1 0 1
[34511] 2 0 0 1 1 0 1 1 0 0 2 0 1 0 0 1 1 0 3 1 0 0 0 0 1 0 0 0 1 0 0 2 0 1
[34545] 1 0 0 0 2 0 1 0 3 0 0 2 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 0 1 1 0 1 0 1
[34579] 5 1 2 0 0 0 0 2 0 0 4 1 1 1 1 0 1 1 0 0 2 0 0 0 0 1 0 0 0 0 0 0 0 0
[34613] 0 0 0 3 0 0 3 0 1 0 1 0 0 0 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 1 0
[34647] 1 1 0 1 1 0 0 2 0 1 0 1 1 0 1 1 1 0 0 1 0 0 0 1 0 0 0 0 1 0 0 2 2 1
[34681] 1 1 1 1 1 4 3 1 0 1 0 0 0 1 0 1 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 0 1 0
[34715] 1 0 0 0 2 0 0 1 0 1 2 0 0 0 0 0 0 0 0 1 1 0 0 2 1 1 0 0 3 0 0 0 1 0
[34749] 0 1 0 1 0 2 3 3 3 1 2 4 1 1 3 0 2 0 1 0 1 5 1 0 2 1 1 3 2 1 2 0 2 1
[34783] 0 0 2 2 0 0 1 2 1 2 1 0 1 1 0 1 1 3 4 2 0 2 3 0 3 1 2 1 2 1 0 0 0 0
[34817] 1 1 0 2 1 0 0 1 1 1 0 1 2 1 1 2 0 0 4 0 2 0 0 1 1 2 0 0 1 0 1 0 0 0
[34851] 0 0 0 3 0 0 0 0 1 0 0 1 0 3 0 2 0 0 0 1 0 2 1 0 0 0 0 1 1 1 2 1 0 2
[34885] 0 2 1 2 0 0 0 0 0 0 0 0 0 1 1 1 1 1 1 1 1 0 0 0 0 1 0 0 0 0 1 1 0 0
[34919] 1 1 0 1 0 1 0 0 1 1 0 0 0 0 2 0 1 0 0 0 1 1 1 2 0 0 1 0 0 0 0 0 0 0
[34953] 1 1 0 0 0 0 1 0 0 2 0 2 0 1 1 1 0 1 3 0 2 4 1 1 3 2 0 1 0 0 0 0 1 1
[34987] 0 0 1 2 2 0 1 0 1 0 0 0 0 0 0 0 1 0 0 0 2 1 0 3 0 0 0 0 1 2 0 0 0 1
[35021] 0 0 1 0 1 0 0 0 2 0 0 1 1 0 1 2 2 0 0 2 1 0 0 0 0 0 0 0 0 3 0 0 1 1
[35055] 1 0 0 0 0 0 0 0 1 1 0 1 2 3 0 1 1 0 1 2 0 0 1 0 0 2 0 0 2 0 0 2 0 1
[35089] 0 1 0 1 0 0 1 1 0 0 0 0 0 0 2 0 0 1 0 0 1 0 2 0 1 0 2 2 0 0 0 1 1 0
[35123] 0 0 0 1 0 1 1 1 2 1 2 1 0 1 2 2 0 0 0 0 3 3 0 3 2 2 2 0 2 3 1 1 1 0
[35157] 2 1 2 0 1 0 0 0 0 0 2 0 1 0 1 1 1 1 1 0 1 1 2 2 2 3 3 1 2 1 0 0 1 0
[35191] 1 0 0 0 0 0 0 0 0 1 2 2 2 0 0 2 3 0 0 0 0 2 0 1 0 1 2 0 0 0 0 0 0 0
[35225] 0 1 2 0 0 0 0 1 1 0 0 3 0 1 1 0 0 1 0 0 0 2 0 0 1 0 0 0 2 0 0 1 1 0
[35259] 1 2 0 1 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 1 1 1 1 0 3 4 2 0 1 1 0 2 0 0
[35293] 1 3 0 5 0 0 0 0 0 1 2 0 0 0 1 0 0 0 1 0 0 0 0 1 0 0 1 0 0 0 0 2 2 0
[35327] 1 2 0 0 0 1 3 1 1 0 0 2 0 1 1 1 2 1 1 1 1 1 0 0 0 0 1 0 1 2 0 0 0 0
[35361] 1 0 1 0 0 0 1 1 1 2 2 1 1 1 0 0 0 0 0 3 0 2 1 2 1 1 0 1 1 0 4 3 4 0
[35395] 2 0 0 0 0 0 1 3 0 0 0 0 1 1 0 0 0 0 0 0 1 1 0 0 0 0 2 0 0 0 0 1 0 0
[35429] 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 2 0 0 0 2 0 0 0 2
[35463] 1 2 1 2 3 3 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 2 0 2 4 2 1 0 2 3 0
[35497] 1 1 1 0 0 2 0 0 0 1 0 0 0 0 1 1 0 2 1 1 0 0 0 0 0 0 0 0 0 0 0 0 1 0
[35531] 0 0 0 2 0 0 2 1 0 1 0 0 0 1 0 0 0 0 2 1 0 0 0 0 0 1 0 0 0 0 1 0 0 0
[35565] 2 0 0 0 0 0 1 0 0 0 0 2 0 0 0 1 1 0 0 0 1 0 0 0 1 0 0 1 1 1 0 0 0 0
[35599] 0 0 1 1 1 1 1 1 0 0 1 0 3 0 1 2 1 0 0 0 0 0 1 3 2 0 1 0 1 1 1 0 2 1
[35633] 2 1 0 1 0 0 0 0 1 0 0 0 2 1 1 0 0 1 0 1 2 0 1 1 1 0 0 1 0 1 3 0 0 1
[35667] 0 0 0 0 1 3 3 1 1 1 1 2 2 0 0 0 0 0 2 1 1 0 0 1 1 0 1 0 0 1 0 0 2 1
[35701] 0 0 1 0 0 0 0 1 0 0 1 0 0 1 1 0 2 1 0 1 0 0 1 0 0 0 1 1 0 0 0 1 0 0
[35735] 0 0 0 0 1 0 0 0 1 1 1 0 0 0 0 0 0 0 1 0 0 0 0 0 1 1 0 0 0 0 1 1 0 0
[35769] 1 0 2 0 1 2 1 1 0 0 1 2 0 0 0 1 1 1 0 0 2 1 1 0 0 0 0 2 1 3 0 0 0 1
[35803] 0 0 0 2 0 0 1 0 0 0 1 0 0 0 0 0 1 0 0 0 2 1 0 1 0 1 0 3 3 2 0 1 0 0
[35837] 0 2 1 0 4 1 3 5 1 2 7 0 3 2 3 0 2 2 1 2 3 1 0 2 1 2 0 1 4 4 3 1 0 0
[35871] 0 2 2 2 4 0 1 1 0 3 2 2 1 3 1 4 1 1 1 0 3 1 3 0 1 1 0 3 1 3 2 3 0 1
[35905] 0 1 0 1 1 1 1 0 3 2 0 0 0 0 0 0 0 1 0 1 0 0 1 0 0 0 2 1 2 0 0 3 0 0
[35939] 0 2 0 0 1 0 0 1 2 1 0 1 0 0 0 0 1 0 2 1 0 1 1 0 1 2 1 0 0 0 1 0 0 0
[35973] 1 0 1 0 0 0 1 0 2 1 0 0 2 1 0 1 1 1 0 0 0 1 1 2 1 0 1 0 0 3 0 0 0 0
[36007] 0 0 0 0 1 2 0 0 0 2 1 1 0 2 0 0 0 1 1 0 1 0 0 0 1 0 0 2 1 0 0 0 1 0
[36041] 0 0 1 0 0 2 0 0 0 0 0 0 0 0 0 1 2 3 1 0 0 0 0 0 0 1 0 1 0 1 0 0 0 1
[36075] 1 2 0 3 1 0 1 1 0 0 0 1 2 1 2 2 0 1 1 0 1 0 1 0 0 0 1 0 0 0 0 0 0 0
[36109] 0 0 0 0 2 1 0 0 0 1 1 0 1 0 1 0 0 0 0 0 0 0 0 2 1 0 1 0 0 2 1 0 0 0
[36143] 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 0 1 0 0 2 0 1
[36177] 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 1 0 0 0
[36211] 1 2 1 0 1 0 0 0 3 0 1 1 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 1 1 0 0 1 0 1
[36245] 0 0 0 0 0 0 0 1 1 0 0 0 0 1 0 0 1 0 0 0 0 1 0 0 0 1 0 1 0 1 0 0 0 0
[36279] 0 0 0 0 1 1 0 2 1 0 0 0 0 1 0 1 1 1 1 0 2 0 0 0 0 0 1 2 1 0 1 0 1 0
[36313] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0
[36347] 0 0 0 0 1 0 0 1 0 0 0 0 0 1 1 0 0 0 1 0 0 0 0 0 0 0 0 1 1 1 0 0 2 1
[36381] 0 0 0 0 0 1 0 1 1 1 1 0 0 0 1 1 1 1 0 0 0 0 0 0 1 0 2 1 2 0 0 0 2 0
[36415] 0 3 0 0 2 0 0 0 1 1 1 0 1 0 0 1 0 0 0 2 0 0 0 0 0 0 0 1 0 0 1 0 1 0
[36449] 0 0 0 0 0 0 0 3 1 0 1 0 0 1 0 0 0 1 1 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0
[36483] 0 0 0 2 1 2 0 0 0 0 0 0 0 1 0 1 0 1 1 1 0 0 1 4 1 0 0 0 0 0 0 0 0 1
[36517] 0 0 1 1 0 0 0 0 1 0 1 1 1 0 0 0 0 1 0 1 2 4 2 2 0 0 0 1 0 0 2 0 0 3
[36551] 2 1 0 0 1 0 0 0 0 0 0 0 0 1 0 1 0 0 0 2 0 0 0 3 0 2 0 1 0 1 0 1 0 1
[36585] 0 0 0 0 1 0 0 1 0 1 0 1 6 1 0 2 0 0 0 0 0 0 0 1 0 2 0 1 1 1 0 1 0 1
[36619] 2 0 1 1 2 0 0 0 2 0 0 1 1 0 0 0 0 1 3 0 0 1 1 2 3 1 1 0 0 1 0 1 0 1
[36653] 2 0 1 2 2 1 0 0 3 0 0 0 0 2 0 1 2 0 0 1 2 0 1 0 2 2 0 0 1 0 1 0 0 1
[36687] 2 0 1 0 1 0 0 1 1 1 0 1 1 0 0 0 1 0 1 0 1 0 0 0 0 0 0 0 1 0 0 0 2 1
[36721] 0 0 0 0 0 0 0 2 1 0 0 1 0 0 0 2 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 1 1
[36755] 0 0 0 0 0 1 0 0 0 1 0 2 0 1 0 0 0 0 1 2 2 2 3 1 0 0 0 0 1 0 0 1 0 1
[36789] 2 1 2 0 2 1 2 0 0 2 1 0 1 0 1 0 0 0 1 0 0 0 0 2 0 0 0 0 0 0 1 1 0 0
[36823] 0 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0
[36857] 0 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 3 1 0 0 0 0 0 0 0 0 0 0
[36891] 1 0 1 1 1 1 1 1 1 0 0 0 1 1 0 0 0 1 0 1 0 0 0 0 0 0 0 0 1 1 1 0 0 0
[36925] 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0 1 0 1 0 0 1 0 0 0 0 0 0 1
[36959] 0 0 0 1 0 1 0 0 2 0 0 0 0 1 0 0 0 1 2 1 0 1 1 0 0 0 2 0 0 0 0 0 0 1
[36993] 0 1 0 0 0 0 0 1 2 1 5 0 0 1 0 1 0 2 0 0 0 0 1 0 1 3 0 0 0 1 2 1 0 0
[37027] 0 1 1 1 0 2 1 0 0 3 2 1 1 0 2 0 0 1 3 0 1 0 0 2 1 3 1 3 0 0 0 1 0 1
[37061] 1 3 1 0 3 1 1 0 1 0 1 1 0 0 0 1 3 1 3 1 0 1 2 3 1 1 0 0 1 0 0 0 0 0
[37095] 0 0 0 0 3 0 3 1 3 1 0 0 3 1 1 2 1 0 0 0 1 0 1 0 2 0 2 0 0 1 0 0 0 0
[37129] 0 1 0 1 2 1 0 0 0 0 1 0 0 1 1 1 2 0 1 1 2 0 2 1 0 0 3 3 0 2 0 0 0 0
[37163] 0 1 3 1 1 4 3 0 2 0 1 3 0 1 2 3 1 1 2 2 1 2 0 1 0 0 1 1 1 0 2 2 1 1
[37197] 3 1 3 1 2 2 0 4 1 1 0 2 3 0 1 5 1 0 0 3 2 3 3 5 4 1 1 4 3 2 0 1 1 1
[37231] 0 0 0 1 3 2 1 1 1 1 1 2 0 1 0 0 0 0 0 0 0 0 0 0 1 0 1 2 0 0 1 0 2 3
[37265] 0 1 0 2 1 0 0 0 1 1 3 1 4 1 2 1 1 0 0 1 0 2 0 2 3 2 2 0 1 0 1 1 0 0
[37299] 2 0 1 1 4 0 1 0 5 6 1 4 2 2 0 2 1 1 1 0 0 2 2 0 0 0 1 1 1 0 1 2 2 0
[37333] 0 0 0 0 0 0 0 0 0 2 3 0 0 0 0 0 0 1 1 0 3 1 0 1 3 0 0 1 2 0 0 0 0 0
[37367] 2 0 2 3 0 2 0 0 1 1 1 1 0 1 2 1 3 0 1 3 0 1 0 0 0 1 1 0 0 0 1 1 0 1
[37401] 0 0 1 1 0 0 0 0 1 1 0 0 0 0 0 1 0 0 0 1 0 1 1 0 0 0 0 0 2 0 1 0 3 0
[37435] 1 1 0 1 1 0 1 1 1 1 1 2 0 1 0 0 2 1 0 4 1 2 2 1 0 0 0 3 1 1 1 0 1 1
[37469] 2 1 1 0 0 0 0 0 2 1 0 3 0 0 0 0 3 1 3 0 0 0 3 1 1 3 2 3 0 1 1 1 2 1
[37503] 0 2 0 1 1 1 1 2 0 0 3 1 0 1 0 1 1 1 2 0 0 0 1 0 1 1 0 2 0 0 0 0 2 0
[37537] 0 0 0 2 1 3 0 1 1 0 1 0 0 0 2 3 1 1 5 3 2 2 0 0 1 1 1 0 4 0 0 2 0 1
[37571] 1 0 1 1 2 1 2 2 1 1 0 1 0 0 1 1 1 1 0 1 2 0 0 1 1 1 0 0 2 2 1 0 1 0
[37605] 1 3 0 2 0 0 0 2 1 0 1 0 3 0 2 2 2 2 0 0 0 0 0 1 0 2 1 1 0 2 0 2 1 1
[37639] 0 0 0 0 1 0 0 0 0 2 1 0 0 0 0 1 0 1 0 0 3 0 0 1 2 0 2 0 0 2 0 0 4 2
[37673] 1 4 0 0 1 0 0 3 1 2 4 1 0 2 1 1 3 1 1 1 0 1 0 0 0 0 0 0 4 1 0 1 0 0
[37707] 0 1 1 1 0 0 0 1 0 2 3 1 0 2 4 3 0 0 2 2 1 0 1 2 2 3 1 0 2 0 1 0 0 1
[37741] 1 0 0 0 0 0 0 0 0 3 5 2 3 3 5 1 1 0 0 1 1 1 0 0 0 0 1 1 1 1 0 1 0 1
[37775] 4 2 2 0 1 1 1 3 2 2 1 1 0 2 2 2 0 0 1 2 1 1 0 0 0 0 2 0 1 4 1 0 1 2
[37809] 3 2 1 2 0 4 2 0 2 4 1 1 0 2 1 4 2 3 0 3 2 1 2 0 1 0 3 0 0 0 0 2 0 1
[37843] 1 0 0 0 1 2 2 0 1 0 0 1 0 0 4 0 1 0 3 2 1 0 1 1 1 3 4 3 0 1 2 0 0 2
[37877] 1 0 1 0 0 0 1 1 2 0 3 0 0 0 1 2 2 1 0 0 2 0 0 2 3 0 0 1 1 0 0 0 1 2
[37911] 4 2 0 1 1 0 0 1 0 0 1 1 1 4 1 0 0 1 0 0 1 1 0 2 1 0 2 1 0 2 2 2 1 1
[37945] 0 0 0 0 0 1 2 2 0 0 0 0 0 0 1 0 0 0 3 2 1 1 0 1 3 0 2 1 0 2 0 0 3 1
[37979] 0 0 2 0 1 0 0 0 1 1 0 1 1 1 2 0 0 0 1 0 2 0 1 1 2 2 1 1 0 0 1 0 0 3
[38013] 0 0 1 1 1 1 0 0 1 0 1 3 1 1 0 0 1 2 1 0 0 0 1 1 1 0 0 0 0 0 0 0 0 0
[38047] 0 1 0 1 0 0 1 0 2 0 0 4 0 0 2 0 0 0 0 0 0 0 1 0 2 0 1 0 0 0 0 0 1 0
[38081] 1 2 1 0 1 2 0 0 1 0 0 0 0 0 1 0 1 1 0 0 0 1 0 0 0 1 2 0 0 2 2 2 1 0
[38115] 2 2 3 0 1 1 3 0 3 1 2 0 3 1 0 1 3 0 2 0 0 1 0 0 0 1 0 0 0 1 2 0 3 2
[38149] 0 2 1 1 0 0 1 0 2 0 1 1 2 1 0 1 0 5 2 1 5 2 0 4 1 1 1 4 0 1 0 0 0 0
[38183] 3 0 0 0 0 0 0 1 0 1 1 1 0 0 0 0 1 1 1 0 1 2 3 2 0 0 3 0 0 1 2 0 1 0
[38217] 1 0 0 0 1 1 1 0 1 3 0 4 2 1 4 1 3 2 0 1 1 1 3 0 2 1 1 1 2 1 0 0 0 1
[38251] 2 1 2 1 0 4 3 1 3 1 1 1 2 1 0 2 0 3 0 1 0 0 1 0 1 0 1 0 0 2 0 0 0 1
[38285] 1 1 1 0 1 0 2 0 0 1 0 0 0 0 0 0 0 0 1 0 2 1 0 0 0 0 0 1 0 2 1 0 0 0
[38319] 0 0 0 0 0 1 1 2 0 3 1 0 0 0 1 0 1 1 2 1 2 1 1 0 1 1 0 2 0 0 1 1 0 2
[38353] 0 0 0 0 0 0 0 1 0 0 1 1 0 0 1 0 0 0 0 0 1 1 0 0 0 1 0 0 0 0 0 1 0 0
[38387] 2 0 0 1 2 0 0 0 1 0 0 0 1 0 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0
[38421] 3 0 0 0 0 1 0 0 0 1 1 2 0 0 0 0 0 0 0 1 0 0 1 0 2 2 2 1 1 0 0 0 0 0
[38455] 2 0 0 1 0 0 0 0 0 1 0 0 0 0 0 1 1 0 0 0 0 1 1 0 2 0 1 0 0 1 2 1 0 0
[38489] 2 3 1 1 0 0 0 0 0 0 3 1 0 0 0 0 0 1 0 0 2 4 2 0 1 1 0 2 0 1 1 3 1 0
[38523] 0 4 1 2 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 1 1 0 0 1
[38557] 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 0 0 0 1 0 1 0 1 1 0 1 0
[38591] 1 2 1 0 1 0 0 0 0 0 2 1 0 1 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 3 0 0 0 0
[38625] 0 0 0 0 1 0 0 0 0 1 1 0 0 0 1 1 0 0 1 2 2 0 0 1 0 0 0 0 0 2 1 0 3 0
[38659] 2 0 0 2 0 0 1 2 0 0 0 1 0 0 2 3 0 2 1 1 0 0 0 0 1 1 0 1 0 0 0 0 0 0
[38693] 2 3 2 0 0 0 0 0 2 0 0 0 1 1 0 0 0 1 1 1 0 0 2 0 0 1 1 0 1 0 1 0 1 0
[38727] 0 0 2 1 0 0 0 0 0 0 0 0 1 1 2 0 0 0 0 0 1 1 0 0 1 0 0 0 0 0 1 2 0 0
[38761] 1 1 0 2 0 0 0 0 2 0 0 0 1 0 1 0 0 1 0 3 0 0 0 1 1 0 0 1 1 1 0 1 2 1
[38795] 1 0 0 1 0 1 1 0 0 0 1 1 0 2 1 0 0 0 3 0 0 0 0 1 0 0 1 1 0 0 0 0 3 0
[38829] 1 0 1 0 0 1 1 1 0 0 1 1 2 0 0 0 0 1 0 0 1 0 0 0 0 1 0 0 2 0 0 0 2 0
[38863] 1 0 0 1 0 0 0 0 1 0 1 0 0 0 0 1 1 2 0 1 0 0 0 1 0 0 1 0 0 0 0 0 1 0
[38897] 0 1 0 0 0 0 3 1 2 1 3 0 0 1 0 0 0 2 0 0 0 1 1 2 1 1 2 0 1 1 3 1 1 0
[38931] 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 3 1 0 0 0 0 3 0 0 1 1 1 0 1
[38965] 1 2 1 1 1 0 1 2 0 0 1 0 0 0 0 0 0 0 1 0 0 1 1 0 1 0 0 0 1 0 0 0 0 0
[38999] 0 0 0 0 0 0 0 0 1 0 1 0 0 1 2 0 0 0 0 0 2 0 0 0 0 0 1 1 1 4 0 0 3 3
[39033] 1 0 1 0 0 0 0 0 0 0 0 2 0 2 1 0 1 2 1 0 3 1 1 0 1 0 0 1 0 0 0 3 2 1
[39067] 0 2 1 0 0 2 1 1 2 0 0 0 1 1 1 3 1 2 0 0 0 1 0 1 1 0 0 0 0 1 0 0 0 0
[39101] 0 0 0 1 1 0 1 0 0 0 2 0 0 2 0 0 1 1 1 1 0 3 0 1 1 1 1 3 0 0 1 1 0 0
[39135] 1 1 0 1 1 0 0 1 0 0 0 0 1 0 1 0 1 2 0 1 1 4 1 2 1 0 0 0 1 1 0 1 0 0
[39169] 0 0 0 0 0 2 0 1 1 0 1 0 0 1 0 0 0 0 0 0 0 0 0 1 1 1 1 0 1 1 0 0 5 3
[39203] 1 0 0 0 2 1 1 1 0 1 1 0 1 3 1 1 3 0 2 0 0 0 0 0 2 0 1 2 1 2 1 1 1 3
[39237] 2 3 1 0 1 0 2 0 0 0 1 1 3 2 1 0 1 0 1 3 3 0 0 0 0 1 0 1 1 0 3 0 0 0
[39271] 0 1 1 3 1 0 0 1 1 0 1 0 0 0 1 0 0 2 1 1 1 1 1 2 3 0 1 1 0 0 3 1 1 2
[39305] 0 0 1 1 0 0 2 0 3 0 0 0 0 0 0 0 0 0 1 2 1 0 1 3 0 1 1 1 0 1 4 2 2 1
[39339] 0 1 0 0 2 2 0 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 1 0 0 0 0 2 1 2 0 0 0 1
[39373] 0 0 0 0 1 0 0 1 0 0 0 0 0 1 1 1 0 1 0 3 0 2 1 0 0 0 1 0 0 1 0 0 1 0
[39407] 0 0 0 0 0 1 0 0 1 2 1 0 0 1 0 0 1 0 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 1
[39441] 0 0 0 0 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 0 2 0 1 1 1 0 0 0 0 1 0 0 0 0
[39475] 0 0 0 2 0 1 0 0 1 1 1 0 0 2 0 0 0 0 0 1 0 0 0 0 1 0 1 0 0 0 0 0 0 0
[39509] 0 0 0 2 0 0 0 0 0 0 0 0 0 2 0 2 1 1 1 0 2 0 1 0 3 0 0 0 2 0 0 1 0 1
[39543] 0 1 1 0 0 0 1 1 0 1 1 1 1 1 0 0 0 0 0 0 2 0 0 0 1 0 0 0 0 0 0 0 0 0
[39577] 0 0 1 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0
[39611] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 1
[39645] 0 0 0 0 2 0 0 0 0 0 0 0 2 0 0 1 0 1 1 1 0 0 0 1 1 1 0 0 0 0 0 0 0 0
[39679] 1 0 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 2 0 1 0 1 0 0
[39713] 0 2 0 0 0 1 0 0 0 0 1 2 0 0 3 0 0 1 0 0 0 1 0 0 0 0 0 0 0 0 1 0 0 0
[39747] 0 1 2 0 3 0 1 0 0 0 0 0 1 1 0 1 2 0 3 0 0 0 3 0 0 2 1 2 1 1 1 0 0 0
[39781] 1 1 1 2 0 1 1 1 3 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 3 6 1 0 0
[39815] 0 1 2 1 0 0 0 0 1 1 0 0 1 1 0 0 1 1 2 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[39849] 0 1 1 0 1 2 1 0 0 1 0 1 1 0 1 0 0 1 0 1 0 0 0 1 0 0 0 1 0 1 0 0 0 1
[39883] 0 1 0 0 0 1 0 0 0 0 0 1 0 0 1 1 0 1 1 1 0 0 0 0 1 0 0 0 1 0 2 0 0 0
[39917] 0 0 0 0 0 0 0 0 0 1 0 1 1 1 1 0 0 2 0 0 0 1 0 0 0 0 1 0 0 0 0 0 0 0
[39951] 0 0 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 1 1 0 0 0 0 0
[39985] 0 0 1 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 1 0 0 0 0
[40019] 1 2 1 1 1 1 1 4 1 1 0 1 0 0 2 1 1 2 0 1 1 0 0 2 1 0 1 1 1 0 0 1 0 0
[40053] 0 0 0 1 1 1 2 1 0 0 1 2 2 0 0 0 0 0 1 0 1 0 0 1 0 1 0 0 0 0 0 0 0 0
[40087] 0 0 0 0 0 0 0 0 0 0 1 0 0 1 2 2 1 1 0 1 3 0 1 1 0 1 1 1 1 0 0 0 0 1
[40121] 0 2 0 0 0 3 2 1 2 0 0 0 2 0 0 0 0 1 0 1 2 3 0 1 2 2 2 2 0 2 0 0 0 1
[40155] 1 0 2 0 0 0 1 0 1 2 0 0 0 2 1 0 1 1 0 0 0 1 1 0 1 2 0 0 1 0 1 1 2 1
[40189] 0 0 0 0 2 1 1 1 0 0 0 1 1 0 1 1 0 1 2 0 1 0 0 1 2 0 2 0 0 1 1 0 0 0
[40223] 1 1 0 0 0 2 1 0 0 1 0 1 0 0 0 2 2 0 0 0 0 2 0 0 0 0 1 4 0 3 2 1 1 0
[40257] 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0
[40291] 0 0 0 0 0 0 0 0 0 0 0 2 1 1 1 0 0 0 0 0 0 0 0 0 1 2 2 0 0 0 2 0 0 0
[40325] 0 0 0 1 0 0 0 0 4 3 1 0 0 1 0 0 0 0 0 0 0 1 1 3 2 1 2 2 2 0 0 0 0 0
[40359] 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 0 0 0 0 0 2 2 0 0 0 1 0 2 0 0 0 1 3 3
[40393] 0 1 1 1 1 0 2 0 0 2 2 1 0 0 0 0 0 0 4 0 0 0 0 0 0 2 0 1 1 0 1 1 1 0
[40427] 0 1 2 2 0 1 0 3 0 1 0 1 1 1 1 0 1 0 2 0 0 0 1 1 1 0 0 2 2 1 4 0 0 1
[40461] 1 2 3 3 1 0 0 1 0 2 0 2 4 2 1 0 0 0 1 0 1 2 0 0 0 1 0 1 1 0 2 0 1 4
[40495] 3 1 0 1 1 0 1 0 3 2 1 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[40529] 1 0 1 1 0 1 0 1 1 1 1 1 1 1 1 1 0 0 1 1 3 0 3 1 0 2 1 0 2 0 0 0 1 0
[40563] 0 1 0 0 0 0 0 1 1 0 0 1 0 0 1 1 1 4 0 1 3 1 0 0 0 2 0 0 0 1 1 2 2 0
[40597] 1 0 0 0 0 0 0 0 0 0 0 3 0 0 0 0 0 0 0 0 2 1 1 0 0 0 1 1 1 0 0 1 2 1
[40631] 2 0 0 0 0 0 0 0 2 0 0 1 0 0 1 1 0 0 0 0 0 1 0 0 0 0 0 1 1 0 2 2 0 2
[40665] 0 2 0 0 4 2 4 5 0 2 1 2 3 0 0 4 0 5 0 0 0 3 1 3 1 5 0 3 1 3 3 1 2 0
[40699] 0 0 0 1 2 0 1 1 1 0 2 0 0 0 0 0 3 2 3 0 2 0 0 0 5 1 3 1 0 1 0 0 0 0
[40733] 1 1 2 3 1 1 0 1 6 0 0 0 0 3 4 1 1 3 0 0 0 0 0 3 6 2 2 0 0 0 3 0 0 0
[40767] 2 0 0 0 0 0 0 0 0 1 2 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 3 0 2 0 0 0 0
[40801] 0 0 0 0 1 2 0 0 3 1 3 1 2 0 0 0 0 0 0 0 1 0 0 1 1 0 0 0 1 2 3 1 2 1
[40835] 3 1 1 0 1 0 0 0 0 0 0 0 0 1 6 1 2 1 2 5 1 0 0 0 0 0 1 0 0 0 1 4 4 0
[40869] 2 3 2 1 1 0 5 3 0 5 0 1 0 0 0 0 0 0 0 1 0 0 0 1 3 0 0 0 0 1 1 1 1 1
[40903] 0 0 1 0 2 0 1 2 1 1 0 0 2 2 1 1 1 1 1 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[40937] 1 0 0 0 0 0 0 3 0 0 0 0 0 0 1 1 3 1 0 0 1 0 0 1 0 0 1 1 0 0 1 0 0 0
[40971] 0 1 3 0 0 0 0 2 1 1 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2
[41005] 0 0 0 0 1 0 0 0 1 1 1 0 1 1 0 1 3 1 0 0 1 0 0 0 3 0 0 0 0 0 1 1 1 1
[41039] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 2
[41073] 0 1 2 0 0 0 0 0 1 0 1 0 0 2 0 0 0 1 1 0 0 1 0 0 0 0 0 0 0 0 1 0 0 0
[41107] 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0 0 1 3 3 1 1 0 0 1 0 2
[41141] 1 2 5 1 0 0 0 0 0 0 2 0 1 1 0 1 1 1 1 1 0 1 0 1 0 1 0 0 0 0 0 1 2 0
[41175] 0 0 0 0 0 0 0 0 1 0 0 2 1 0 1 0 1 0 0 0 3 0 0 1 0 3 0 1 0 0 1 0 1 0
[41209] 0 0 0 2 2 0 0 1 0 0 0 0 0 1 0 0 0 1 0 0 1 0 3 1 0 1 1 0 2 0 1 2 0 0
[41243] 0 0 1 1 0 1 0 0 0 0 0 0 0 0 0 0 0 1 1 0 1 0 0 1 0 0 0 0 0 3 0 1 2 0
[41277] 0 0 0 1 1 0 1 0 1 0 0 2 1 2 0 0 0 0 2 3 0 1 1 0 0 0 0 0 0 1 0 0 0 0
[41311] 0 0 0 1 1 0 0 0 0 0 0 1 1 0 0 0 0 0 0 1 0 3 1 1 0 0 0 0 1 0 1 0 0 0
[41345] 2 1 0 0 2 0 0 0 2 1 0 1 0 1 0 0 0 0 0 0 1 1 0 0 1 1 3 1 1 0 2 1 0 0
[41379] 2 0 0 0 0 0 0 1 0 0 1 1 0 1 3 0 1 2 2 0 0 3 0 1 3 0 0 6 2 1 1 0 2 1
[41413] 2 1 0 0 1 0 1 0 1 0 0 0 0 0 0 0 0 1 0 0 0 0 0 2 1 1 1 0 0 0 0 0 1 1
[41447] 0 1 2 0 0 1 0 1 0 0 0 0 1 0 1 0 1 0 1 2 0 1 0 1 0 0 1 0 0 1 0 0 0 0
[41481] 0 0 0 0 1 0 0 0 1 0 0 0 0 1 0 1 1 1 0 1 0 1 1 1 1 0 1 0 0 0 0 0 0 0
[41515] 0 0 0 0 0 0 1 0 0 1 1 0 0 0 0 0 1 0 1 0 0 1 0 0 2 1 1 0 3 3 0 2 0 0
[41549] 0 0 0 2 0 0 0 0 0 1 0 0 0 2 0 0 3 2 1 0 0 0 0 0 0 0 3 0 1 2 3 2 0 1
[41583] 0 0 1 0 1 1 1 0 4 1 0 1 2 1 1 0 0 2 2 0 0 0 0 0 0 1 1 1 1 2 2 3 0 0
[41617] 1 1 0 1 1 0 0 0 0 0 0 0 0 1 0 3 0 2 0 0 1 0 1 1 1 0 0 2 0 0 1 1 0 2
[41651] 2 0 0 1 0 1 0 0 0 1 0 0 0 1 0 0 1 1 0 0 1 2 0 0 0 2 0 1 3 0 0 0 0 5
[41685] 2 1 3 1 0 1 3 2 1 0 1 2 0 2 0 0 0 3 2 3 0 1 2 0 1 1 2 0 0 1 1 0 0 0
[41719] 1 0 2 4 0 1 0 0 0 0 0 0 1 1 0 1 2 3 1 2 1 1 0 0 2 2 0 1 0 0 0 0 1 0
[41753] 0 1 2 1 1 0 1 0 1 1 0 1 1 1 3 0 1 1 0 0 1 1 0 1 0 0 0 0 0 0 0 0 0 1
[41787] 0 1 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0
[41821] 0 0 1 1 0 3 0 3 2 0 0 1 1 0 1 0 0 0 1 1 0 0 0 0 0 0 1 1 1 0 0 0 0 0
[41855] 2 3 2 1 1 1 0 1 3 3 2 1 0 1 1 0 0 0 0 0 0 0 1 0 2 2 0 0 0 1 0 1 0 1
[41889] 1 0 3 2 2 1 4 2 0 2 0 0 3 0 1 0 0 0 0 0 1 0 0 0 0 1 0 0 1 1 1 0 1 0
[41923] 0 2 1 1 3 0 4 0 1 1 3 1 1 0 5 0 1 0 0 4 0 1 0 0 0 1 0 2 0 0 2 4 3 1
[41957] 0 0 1 3 1 0 1 1 2 0 1 0 0 1 0 1 2 1 2 4 5 1 0 1 2 0 2 4 1 0 0 0 0 3
[41991] 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 1 1 0 0 0 0 0 1 0 1 0 0 1 2
[42025] 2 3 1 1 1 1 1 1 0 0 2 1 1 0 0 0 3 1 2 1 0 0 0 2 3 0 0 1 2 1 1 3 3 2
[42059] 0 0 0 1 0 0 0 1 2 3 1 1 4 2 0 0 2 3 1 1 0 3 1 1 0 1 2 5 0 0 2 0 2 0
[42093] 1 0 0 1 0 0 0 0 1 1 0 0 1 1 1 0 0 2 0 1 0 0 1 0 2 0 1 1 1 1 1 1 1 3
[42127] 1 0 0 1 1 0 3 0 0 0 0 1 3 0 0 2 0 0 2 0 1 0 0 0 0 2 1 4 0 3 0 2 0 1
[42161] 1 0 1 0 0 1 1 0 0 1 3 4 0 1 0 0 1 2 1 0 0 1 1 1 0 1 1 0 0 0 0 0 2 2
[42195] 0 1 0 0 2 0 0 1 0 1 0 2 0 0 0 0 0 0 1 1 1 1 1 2 3 0 1 0 0 2 0 1 0 2
[42229] 2 0 0 0 3 1 0 2 2 3 1 2 1 0 0 0 0 0 1 0 0 0 1 0 1 0 0 0 0 4 0 0 1 1
[42263] 1 0 1 0 1 1 0 0 2 0 0 1 0 0 0 0 0 2 0 0 1 1 0 0 0 4 1 0 2 2 1 1 0 2
[42297] 0 0 0 0 1 3 0 0 0 0 3 3 0 1 0 4 1 1 3 0 0 2 0 0 0 0 0 0 1 0 0 0 0 0
[42331] 0 0 0 0 1 0 0 1 3 2 2 1 0 0 0 0 1 1 0 0 1 1 0 1 1 1 0 0 0 0 0 0 1 1
[42365] 0 1 0 0 0 0 0 1 0 1 1 1 0 2 0 1 1 0 0 0 0 1 0 0 1 0 1 0 0 0 1 1 0 1
[42399] 0 1 1 0 0 1 0 0 0 1 1 0 0 0 1 0 0 0 0 0 0 0 2 0 1 0 0 0 0 0 0 0 0 0
[42433] 1 1 0 0 0 0 1 0 0 0 0 1 0 2 0 0 0 0 0 1 0 0 1 0 1 0 0 0 0 0 0 0 0 0
[42467] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 1 0 0 0 0 0 0 1 0 0 0 0 1
[42501] 0 0 0 0 0 0 1 0 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 1 0 0 0 1 0 0
[42535] 0 0 0 0 1 0 0 1 1 2 0 0 0 0 0 1 0 1 0 0 1 0 1 1 1 0 0 1 0 0 0 0 0 0
[42569] 0 0 0 0 0 0 1 0 0 3 0 0 1 1 0 0 0 1 0 0 0 0 1 2 0 0 0 1 1 0 0 1 1 0
[42603] 0 0 0 1 0 0 0 0 0 0 0 1 0 0 0 0 1 1 0 2 0 0 1 0 0 0 0 1 0 0 0 1 1 1
[42637] 1 1 0 0 0 0 1 0 1 1 0 0 0 0 0 0 0 0 2 0 0 1 0 0 0 1 0 0 0 0 2 2 0 0
[42671] 0 1 1 1 0 0 1 0 0 0 0 0 0 3 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0
[42705] 1 3 2 1 0 0 0 0 0 0 1 0 1 1 1 0 1 0 1 0 0 0 1 1 0 1 2 0 1 0 0 0 0 3
[42739] 0 1 0 0 0 3 0 0 0 1 0 0 0 0 0 2 1 1 0 0 0 0 3 0 1 0 1 3 0 2 1 1 0 3
[42773] 1 2 0 0 0 0 0 0 0 2 0 1 0 1 0 2 1 0 0 0 0 1 0 1 0 0 0 0 1 2 0 2 1 0
[42807] 0 1 2 0 1 0 3 0 1 0 0 2 0 2 1 0 0 2 1 1 0 0 0 1 1 1 2 0 0 1 0 1 0 1
[42841] 1 0 1 1 0 0 1 0 0 0 1 0 0 1 1 0 0 0 0 1 0 1 2 0 0 0 1 1 0 1 0 2 0 1
[42875] 1 0 0 2 0 0 0 1 0 0 1 0 0 1 0 2 0 0 0 0 0 0 0 1 1 1 0 0 0 1 2 0 0 1
[42909] 2 0 1 0 3 1 1 3 0 2 1 0 1 0 1 0 0 0 3 2 0 1 0 2 0 0 0 1 2 2 0 0 0 1
[42943] 1 4 4 1 1 0 0 2 2 1 0 1 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0 1 0 1 0 2
[42977] 1 0 0 0 0 1 0 0 0 1 0 0 0 1 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0 1
[43011] 1 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 0 2 0 1 3 2 0 1 1 1 1 1 0 1 0 0 0
[43045] 2 4 0 2 2 1 2 2 1 0 2 3 0 0 1 2 1 2 2 1 1 2 1 1 1 0 1 2 1 0 0 2 0 3
[43079] 1 1 1 0 0 0 0 3 2 0 1 0 0 0 0 0 1 2 0 0 0 1 0 0 0 0 0 0 1 1 1 1 1 2
[43113] 0 2 0 0 0 0 0 0 3 1 0 0 0 0 0 1 0 1 0 0 1 0 1 0 0 1 0 2 1 2 1 0 3 0
[43147] 0 0 1 1 3 3 0 2 0 0 0 0 1 1 0 1 0 7 7 0 2 4 2 2 0 0 2 2 0 0 1 0 0 2
[43181] 0 0 1 0 0 2 1 1 0 0 0 0 0 1 0 0 0 0 0 1 0 0 1 0 1 0 0 1 0 0 0 0 0 0
[43215] 0 0 0 0 1 0 0 2 1 0 0 1 0 2 0 2 0 1 0 1 1 1 0 2 0 0 1 0 1 0 0 2 0 0
[43249] 0 0 0 0 4 0 0 0 0 1 0 0 1 0 0 1 1 1 0 0 1 1 1 0 2 1 1 1 3 0 3 0 5 3
[43283] 2 2 0 0 1 4 4 4 3 0 0 1 3 1 6 5 3 1 3 0 2 1 0 1 1 1 1 0 0 0 0 0 1 0
[43317] 0 1 0 2 1 2 0 1 0 1 1 2 0 0 5 2 1 3 4 0 0 4 0 1 0 0 2 1 0 2 1 2 2 0
[43351] 1 2 0 1 1 1 4 0 0 0 2 0 0 1 0 0 1 1 1 1 0 0 2 0 2 0 0 0 0 2 0 2 2 0
[43385] 1 1 1 1 0 2 1 0 0 0 0 1 0 0 0 1 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0
[43419] 1 1 0 0 1 0 0 0 0 0 1 1 1 0 1 0 1 0 1 1 1 0 0 1 0 0 0 2 1 2 1 0 2 2
[43453] 2 1 1 0 2 2 1 0 0 2 1 2 1 0 1 0 0 0 0 0 1 1 1 1 0 1 0 0 1 0 1 1 0 1
[43487] 0 0 0 1 0 0 0 1 0 0 0 2 1 1 0 1 0 0 1 1 1 1 1 2 1 2 0 1 1 0 0 1 0 0
[43521] 0 2 0 0 0 2 1 2 0 0 0 1 0 1 2 4 1 4 0 0 0 0 0 0 1 0 0 0 1 0 0 0 2 0
[43555] 0 0 1 1 2 1 0 0 0 0 3 0 0 1 0 2 0 0 1 2 3 1 2 2 1 2 0 2 2 0 0 0 0 2
[43589] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 2 0 0 0 0 0 0 0 0 1
[43623] 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 1 0 1 1 1 0 0 0 0 0 1 1 3 0
[43657] 1 0 1 0 0 2 0 0 1 2 1 0 0 2 0 2 0 2 0 0 3 1 0 0 3 0 2 0 1 0 2 0 1 2
[43691] 1 0 2 0 3 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 2 0 0 1 0 1 0 0 2 2 1 0 3 2
[43725] 1 3 1 0 0 1 0 0 1 0 1 1 0 0 0 2 4 1 2 1 0 0 1 0 0 3 0 0 1 0 2 1 0 0
[43759] 0 0 1 0 1 2 0 0 0 2 0 1 0 0 3 1 0 0 0 0 0 0 0 0 0 3 2 0 0 0 0 1 2 0
[43793] 0 1 0 0 1 1 0 0 0 0 0 1 1 0 0 1 0 1 0 0 1 2 3 0 3 0 0 2 1 0 1 0 0 0
[43827] 3 0 1 1 0 0 0 3 0 1 1 1 0 0 0 0 0 0 1 1 1 1 2 1 1 1 0 1 1 0 1 2 1 0
[43861] 0 0 1 0 0 1 1 1 2 0 0 0 0 0 0 1 0 1 2 0 0 3 0 0 0 1 1 3 0 0 0 0 0 1
[43895] 1 0 0 0 0 0 1 0 0 0 1 0 1 0 0 1 1 0 0 0 1 0 1 2 0 0 0 0 0 1 1 0 2 0
[43929] 0 1 0 0 0 0 0 0 0 0 0 1 0 1 1 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0
[43963] 0 0 0 1 1 0 1 0 1 2 0 0 0 0 1 0 0 0 0 0 2 0 0 1 0 0 0 0 0 0 0 0 0 1
[43997] 0 0 0 0 3 0 0 0 1 0 0 0 0 0 0 2 0 0 1 1 1 0 0 1 0 0 0 1 1 0 0 1 1 0
[44031] 1 0 0 0 0 1 0 0 1 0 0 0 0 0 0 0 0 0 0 0 2 0 1 0 1 0 1 0 0 1 0 0 1 0
[44065] 1 3 0 1 0 0 0 0 0 1 0 0 1 1 0 0 0 0 1 1 1 0 0 0 0 0 0 0 0 0 1 2 1 0
[44099] 1 0 0 0 0 1 0 1 0 0 0 1 1 0 0 0 2 0 1 0 0 3 2 1 0 1 0 1 1 0 0 1 2 0
[44133] 0 3 0 1 0 0 0 0 1 1 0 0 1 2 0 2 0 0 0 0 0 1 0 0 1 1 1 0 0 0 1 1 1 0
[44167] 0 0 1 0 0 0 0 1 0 0 1 0 1 2 0 0 0 0 2 0 1 0 0 0 0 0 0 0 3 2 0 0 3 1
[44201] 0 0 0 0 0 0 1 0 0 2 0 1 0 3 2 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 3 0
[44235] 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 1 0 0 1 1 0 1 0 0 0 0 1 0 1 1
[44269] 0 2 0 0 1 0 0 0 0 0 1 2 1 1 0 0 1 1 1 0 0 0 0 0 0 0 1 0 0 0 0 2 0 1
[44303] 0 0 0 0 0 0 0 0 0 0 0 1 2 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0 0 0 1 0 0
[44337] 2 1 1 0 2 1 2 1 0 1 2 2 0 0 0 0 2 0 1 0 0 0 1 0 2 0 1 0 1 2 2 0 0 0
[44371] 0 1 1 0 0 1 1 1 0 0 1 0 0 2 0 0 1 0 1 1 0 0 0 1 0 0 0 2 3 0 1 0 1 2
[44405] 0 0 2 1 0 0 0 0 1 3 0 1 1 1 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 2 0 0 0 0
[44439] 0 1 0 1 0 0 0 0 0 0 1 0 1 0 0 1 0 0 1 2 0 0 0 0 0 1 0 0 0 0 1 0 0 0
[44473] 1 0 1 2 0 0 0 0 0 0 2 0 0 0 0 0 1 0 2 0 0 0 0 2 0 1 0 1 2 0 0 2 5 1
[44507] 1 0 0 1 1 0 0 0 0 0 0 1 1 1 2 2 0 0 2 1 3 2 1 0 0 0 2 0 0 1 0 0 0 1
[44541] 1 0 2 1 1 1 1 0 0 1 1 0 0 0 1 1 0 0 0 1 0 1 1 0 1 0 0 0 0 1 1 1 0 0
[44575] 0 1 0 0 1 1 0 1 0 0 2 2 1 1 1 0 2 0 0 1 1 0 1 0 0 0 1 0 0 2 0 1 2 0
[44609] 1 0 0 0 0 1 0 2 0 1 0 0 1 0 0 1 2 1 1 2 0 0 0 0 1 0 0 0 0 0 0 0 0 0
[44643] 1 1 0 0 1 0 0 0 1 0 1 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 3 0 1 0 0
[44677] 0 0 0 0 0 1 0 3 0 0 0 1 0 0 0 0 1 1 1 1 0 1 2 1 0 3 0 1 1 1 1 0 0 1
[44711] 0 1 0 0 1 2 0 0 3 1 0 0 1 0 1 1 1 0 0 0 0 0 0 1 0 0 1 0 1 0 0 1 0 0
[44745] 1 0 2 1 1 0 1 0 1 1 0 0 1 0 0 2 2 0 0 1 0 0 2 0 0 4 0 0 1 0 0 0 1 0
[44779] 1 0 1 0 0 0 0 1 0 1 1 0 0 0 0 0 0 0 1 1 1 0 1 0 0 0 0 0 0 0 1 0 0 1
[44813] 0 1 0 0 1 0 2 1 0 1 0 0 1 0 0 0 2 0 0 0 1 0 0 0 0 1 0 1 0 1 0 0 0 0
[44847] 0 1 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 2 1 1 0 2 0 1 0 0 1 0 1 0 1 0 0 2
[44881] 5 2 0 0 1 2 0 1 0 0 0 0 0 0 0 0 1 0 1 1 0 1 0 1 1 0 0 2 2 3 0 0 1 2
[44915] 0 0 1 0 0 0 4 1 1 0 0 0 0 1 3 0 0 2 0 1 2 0 1 0 0 1 1 2 1 1 1 0 0 0
[44949] 1 0 0 0 0 1 0 0 1 0 2 3 2 1 0 0 0 1 2 1 1 1 1 0 1 0 0 1 0 0 0 0 0 2
[44983] 2 0 1 0 1 3 1 1 0 0 2 1 1 0 0 0 1 0 1 0 1 0 1 0 0 0 0 0 0 0 0 0 1 0
[45017] 0 0 2 1 3 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 2 1
[45051] 0 0 0 0 0 0 0 0 0 0 0 1 2 0 0 0 0 1 0 0 0 0 0 0 1 1 1 0 0 0 1 1 0 0
[45085] 0 3 1 0 2 0 0 1 0 0 2 2 0 2 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[45119] 0 0 0 0 1 0 2 0 0 0 1 2 0 2 0 0 0 1 0 1 0 1 0 0 1 1 0 0 0 0 1 0 0 0
[45153] 0 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 1 0 0 1 1 0 0 0 0 0 0 0 0 0 0 1 0 1
[45187] 0 2 0 0 0 1 1 2 0 0 0 2 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0
[45221] 0 1 0 1 0 1 0 2 1 0 0 0 0 1 0 0 0 0 0 1 0 0 0 1 0 0 0 0 1 0 0 0 0 0
[45255] 0 0 0 1 1 0 1 0 1 0 2 1 0 1 0 2 1 0 1 1 0 2 0 0 1 1 0 1 0 0 0 0 1 0
[45289] 1 1 0 0 0 0 0 0 2 0 0 1 0 0 0 0 0 2 1 0 0 1 1 0 0 0 0 0 0 1 0 0 1 0
[45323] 0 0 0 0 0 0 0 0 1 0 1 0 3 0 1 0 0 0 0 0 0 1 1 0 0 0 2 0 0 0 0 0 1 0
[45357] 1 1 0 0 1 1 1 0 0 0 2 2 0 0 0 0 0 0 0 1 1 1 0 0 1 0 0 0 0 0 0 1 0 0
[45391] 0 0 0 0 0 0 1 1 0 1 0 3 3 1 0 0 1 1 0 1 1 1 3 0 3 0 1 0 1 1 0 1 1 2
[45425] 0 0 0 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 1 1 0 0 0 0 1 0 0 0 1 2 2 0 1 0
[45459] 1 0 1 0 0 0 1 0 1 3 0 1 0 0 0 2 2 0 0 1 0 2 1 0 0 0 0 0 1 0 0 1 0 0
[45493] 0 1 1 2 0 0 0 0 2 1 0 1 1 0 1 1 2 0 0 0 3 0 0 0 0 1 0 0 0 0 1 0 0 1
[45527] 1 0 0 0 1 0 0 1 0 1 0 0 0 1 0 1 0 0 0 0 1 0 1 0 1 2 1 1 1 0 1 0 0 3
[45561] 2 0 0 0 0 0 0 0 0 1 0 1 1 0 1 1 0 0 0 0 0 0 0 1 0 0 0 1 2 1 1 1 0 0
[45595] 1 0 0 0 0 0 0 0 1 1 1 2 0 1 0 0 0 0 2 1 0 0 0 0 0 0 0 2 0 1 0 0 1 0
[45629] 3 2 0 1 0 0 0 1 0 0 0 1 0 0 0 0 0 0 1 0 1 1 0 0 2 0 0 1 1 0 0 0 2 0
[45663] 0 0 0 0 1 0 1 0 0 0 1 1 0 0 0 0 1 2 0 0 0 1 0 1 0 0 2 0 3 0 0 0 0 0
[45697] 0 0 0 1 1 1 0 0 1 1 0 1 0 0 1 0 0 1 0 0 0 0 0 1 0 0 1 0 0 1 0 0 0 0
[45731] 2 1 2 0 0 2 0 0 0 1 0 2 1 0 0 0 1 1 0 1 0 0 1 0 0 2 0 0 0 1 1 1 0 0
[45765] 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 1 0 1 0 0 1 2 0 0 0 1 0 1
[45799] 0 0 3 0 0 1 0 2 0 0 0 0 0 1 0 1 0 0 0 2 0 0 0 0 0 1 0 0 0 1 2 0 1 1
[45833] 0 1 0 0 0 0 0 0 1 0 0 0 0 1 1 1 0 0 0 0 0 1 0 0 1 2 0 0 1 0 2 0 0 0
[45867] 0 0 1 0 1 0 1 0 0 1 0 2 0 0 0 1 1 0 0 1 0 0 1 0 0 0 0 0 0 0 0 1 1 0
[45901] 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0
[45935] 1 1 0 0 0 2 0 1 0 1 1 0 1 0 1 0 1 0 1 1 0 1 0 1 2 0 0 0 0 0 0 0 0 1
[45969] 1 2 0 1 1 1 0 0 0 0 0 1 0 0 0 3 1 1 3 4 1 1 1 0 1 0 0 0 1 0 0 2 0 1
[46003] 1 0 0 0 0 0 0 0 0 0 1 2 0 0 1 0 0 0 0 1 0 1 0 0 1 0 0 2 2 1 0 1 0 1
[46037] 1 1 0 0 1 0 0 0 0 0 1 0 0 1 0 0 0 0 1 0 0 0 0 1 0 0 0 1 0 0 0 1 0 1
[46071] 0 0 0 0 0 0 1 0 1 0 0 0 1 0 0 0 0 1 1 1 0 0 1 0 2 1 1 2 0 0 0 0 0 1
[46105] 1 0 0 1 0 0 0 0 0 2 0 0 1 2 4 1 0 0 1 0 1 0 0 0 2 0 0 0 0 1 0 0 3 0
[46139] 1 0 0 0 0 2 1 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 2 1 1 0 0 0 0 0 0 0
[46173] 0 0 0 0 0 0 0 0 1 3 2 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 2 1 0 1 0
[46207] 2 1 2 1 0 0 0 0 0 0 0 1 1 1 1 1 2 0 0 0 3 0 0 0 0 1 1 1 1 0 1 1 3 4
[46241] 0 0 1 0 0 0 0 0 0 0 2 1 2 1 3 0 0 0 0 0 0 0 1 3 0 0 0 1 1 1 2 0 3 0
[46275] 2 2 2 0 0 0 0 1 2 1 4 1 4 0 1 3 0 1 0 2 0 1 1 2 0 1 2 1 3 0 0 0 1 0
[46309] 1 0 0 0 2 0 0 0 0 0 1 4 0 0 0 0 0 1 1 0 3 0 1 0 0 0 0 0 1 0 0 0 0 1
[46343] 0 2 1 0 1 0 0 0 0 0 0 0 1 1 0 0 2 0 0 0 2 1 2 2 1 0 0 0 2 0 0 3 0 0
[46377] 3 1 1 1 0 0 1 0 1 2 0 0 0 0 0 0 0 0 2 0 0 2 0 0 0 1 1 0 0 0 1 0 1 0
[46411] 0 0 0 1 0 0 2 0 0 1 1 1 1 0 2 0 0 0 0 1 0 0 1 1 1 1 2 1 0 1 0 0 0 1
[46445] 2 1 0 0 0 2 2 1 0 1 0 1 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 1 1 0 0 1
[46479] 1 0 1 0 2 3 1 3 0 1 1 1 1 0 0 3 0 1 1 0 3 2 0 0 3 0 0 0 0 1 1 0 1 1
[46513] 0 0 0 2 0 3 1 0 0 0 0 0 0 0 0 0 0 2 1 1 0 0 2 1 0 0 1 4 2 0 1 0 0 0
[46547] 0 1 0 0 0 0 0 1 0 0 0 2 0 1 0 0 1 0 2 0 1 0 0 0 1 2 0 1 0 0 0 0 1 0
[46581] 0 0 1 0 0 2 0 0 1 0 0 2 1 0 1 3 1 2 1 0 0 1 0 1 0 1 0 0 0 0 0 0 1 0
[46615] 0 0 0 1 0 0 0 0 1 1 0 1 1 0 1 0 1 0 1 0 3 0 0 0 0 1 0 0 0 2 0 1 2 0
[46649] 1 0 0 0 0 0 1 0 1 0 0 0 1 0 1 0 2 0 1 1 1 0 0 0 0 1 1 0 3 0 0 0 0 1
[46683] 0 0 0 1 1 0 2 1 0 0 1 0 1 1 1 0 1 0 0 0 0 0 0 0 2 1 1 2 0 1 0 0 0 0
[46717] 0 2 1 2 4 3 3 2 0 0 0 0 2 0 0 2 2 1 1 1 1 2 0 1 1 3 1 1 2 1 1 1 1 1
[46751] 0 0 1 0 0 0 1 0 2 1 4 0 2 0 0 0 0 2 0 1 2 2 1 1 1 0 1 0 2 5 1 2 2 1
[46785] 1 1 0 1 1 2 0 1 0 0 0 0 0 2 1 1 2 0 1 1 1 1 1 2 0 0 2 1 0 2 0 1 0 3
[46819] 0 1 1 0 1 1 0 1 0 0 1 0 0 0 0 0 0 1 0 1 1 1 0 0 1 0 1 1 0 1 0 1 0 0
[46853] 0 1 0 0 0 1 1 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 0 0 0
[46887] 0 0 0 0 0 0 1 1 0 0 0 0 0 1 1 0 0 1 1 0 0 0 2 0 0 1 0 0 2 1 2 0 0 0
[46921] 1 0 3 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 1 1 0 0 0 0 0 0 1 0 0 0 0 0 0
[46955] 1 0 1 0 1 1 0 0 1 0 0 0 3 0 1 0 0 2 0 1 1 1 1 0 0 0 0 1 0 0 0 1 0 0
[46989] 1 0 0 0 0 0 1 0 0 2 0 1 0 0 0 0 0 0 1 1 1 0 0 0 0 0 1 0 1 0 0 0 0 0
[47023] 0 0 1 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 1 0 1 2 0 1 0 0 0 0 0 0 1 0 0
[47057] 0 0 0 1 0 1 0 0 0 0 1 0 1 1 1 3 1 1 0 0 0 0 1 2 2 1 0 0 2 0 0 1 0 0
[47091] 0 1 0 0 0 0 0 0 0 1 1 0 0 0 0 1 1 0 0 0 0 1 0 0 1 0 0 1 0 2 2 2 2 0
[47125] 2 1 0 0 1 1 0 1 0 0 0 1 0 1 0 0 0 0 0 0 0 0 1 1 0 0 0 1 1 0 1 0 2 0
[47159] 1 0 0 2 0 0 2 1 1 1 1 1 1 0 0 0 0 1 0 0 1 1 0 0 1 0 0 0 1 0 0 2 0 1
[47193] 0 1 0 0 2 0 1 1 1 0 1 0 3 1 0 0 0 1 1 0 0 0 1 0 2 0 0 0 0 0 0 0 0 0
[47227] 0 0 1 0 0 0 0 0 1 1 2 1 1 1 0 0 0 0 0 0 0 2 0 0 0 1 0 0 0 0 1 0 0 1
[47261] 0 0 0 1 1 1 3 0 0 1 1 0 1 1 1 2 1 0 2 0 0 1 1 0 1 0 0 0 1 0 0 0 0 1
[47295] 0 0 1 0 0 0 0 0 1 1 0 0 0 0 0 0 1 1 0 1 1 0 0 1 1 0 0 0 0 0 1 0 0 0
[47329] 0 0 0 0 0 0 0 2 1 1 0 0 3 0 1 0 0 0 0 1 1 0 0 1 1 1 0 0 0 0 1 1 0 0
[47363] 0 0 0 1 0 1 0 1 0 1 0 1 1 1 0 1 2 0 0 1 0 0 0 1 1 1 1 0 0 1 1 0 1 1
[47397] 0 0 1 0 2 0 0 0 0 1 1 0 1 0 0 0 1 0 1 1 0 0 2 0 1 0 0 1 1 2 0 2 2 0
[47431] 0 0 0 1 3 1 0 0 0 0 2 0 0 0 2 1 0 0 1 1 0 1 1 0 0 0 1 0 1 1 1 0 0 0
[47465] 0 0 1 0 0 0 1 0 1 1 0 2 1 0 2 0 2 1 0 1 2 0 0 0 1 1 1 0 1 1 0 2 0 1
[47499] 0 0 1 2 0 1 1 2 0 1 0 2 2 3 0 2 1 0 0 0 1 0 3 2 0 0 1 0 2 2 0 0 0 1
[47533] 2 0 2 1 0 0 0 1 0 1 1 1 0 1 0 1 1 0 2 0 1 0 0 0 1 1 0 3 0 3 1 1 0 0
[47567] 1 0 2 1 0 1 0 0 1 1 0 0 0 0 0 1 0 0 0 0 0 0 1 0 1 1 1 1 1 0 2 1 0 1
[47601] 0 0 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 1 3 0 0 0 1 0 1 0 0 0 0
[47635] 0 0 2 0 0 1 0 0 0 0 1 0 0 0 0 2 2 3 2 0 0 0 0 0 0 0 2 0 1 1 0 0 0 0
[47669] 0 0 0 1 0 1 0 1 0 1 0 1 1 0 0 0 0 1 0 0 1 0 1 0 0 0 2 1 0 0 0 0 0 0
[47703] 1 0 1 0 0 1 0 1 1 1 1 0 3 2 2 0 0 0 0 0 1 0 1 0 0 0 0 1 1 1 1 1 0 0
[47737] 0 0 0 1 1 1 1 1 1 0 0 0 2 1 0 0 2 0 0 0 0 2 2 1 2 1 0 3 5 0 1 1 2 0
[47771] 0 1 1 0 0 0 0 0 0 0 1 3 0 0 0 2 1 3 2 2 0 1 1 2 0 0 2 0 1 4 1 0 0 1
[47805] 1 1 0 0 0 2 1 2 2 0 1 0 1 0 0 0 0 1 0 1 0 1 1 1 1 0 1 1 3 1 0 4 2 0
[47839] 0 2 0 0 1 0 0 1 0 2 2 0 0 0 1 1 2 0 0 0 0 0 0 3 1 1 0 1 0 0 0 1 2 2
[47873] 1 0 1 0 0 0 1 2 0 2 1 0 0 0 2 0 0 1 3 0 2 0 0 3 0 0 2 0 2 2 1 1 0 2
[47907] 0 0 1 0 1 1 0 2 0 0 1 0 1 0 0 1 1 2 1 1 1 1 1 1 1 0 2 2 0 1 0 0 2 0
[47941] 0 1 0 0 0 0 0 1 0 0 0 2 1 2 0 2 0 1 1 0 1 0 1 2 0 1 0 1 1 2 1 0 1 0
[47975] 0 0 1 1 0 0 0 1 1 1 0 0 2 0 1 1 3 1 0 0 0 0 0 0 0 1 1 0 1 3 0 0 0 1
[48009] 1 0 0 0 2 0 1 1 0 1 2 0 1 1 1 0 1 1 1 0 0 2 1 0 0 2 3 0 0 1 0 0 0 0
[48043] 0 2 1 1 0 2 0 0 2 0 0 2 0 0 0 0 2 0 1 0 1 0 0 0 1 0 0 4 2 0 2 0 2 0
[48077] 0 0 1 1 0 2 0 1 0 1 1 0 0 0 0 1 1 1 0 0 0 0 1 0 1 0 0 1 0 1 1 0 0 0
[48111] 1 1 0 0 1 3 1 0 0 0 0 0 2 1 0 2 3 2 2 0 1 1 3 0 0 0 0 0 1 1 1 1 3 3
[48145] 0 0 1 2 0 2 2 1 0 0 1 0 0 3 0 1 0 0 0 1 1 0 0 0 1 1 3 2 0 1 0 3 0 0
[48179] 0 1 0 1 0 0 1 2 1 1 0 0 0 2 0 1 1 0 2 0 2 1 0 4 1 1 1 0 0 1 2 0 0 2
[48213] 2 1 2 1 0 3 1 1 1 1 0 2 1 0 0 3 1 0 0 2 2 1 0 0 1 0 0 0 1 0 1 2 2 0
[48247] 0 0 0 0 1 0 1 0 1 1 1 0 0 0 0 0 2 0 3 0 0 3 1 2 2 0 2 0 0 1 0 2 1 1
[48281] 3 3 1 0 2 1 2 1 1 0 3 1 0 1 1 1 0 1 1 1 0 1 2 1 1 4 0 0 0 0 2 0 0 0
[48315] 0 0 1 0 1 0 0 0 1 2 0 1 0 2 2 0 1 0 1 0 1 0 0 0 1 0 0 2 0 0 0 1 0 1
[48349] 2 0 1 2 1 1 0 1 0 0 1 0 2 1 0 1 1 0 0 1 1 2 0 0 0 0 0 1 2 1 1 0 3 2
[48383] 0 0 2 2 0 0 2 0 1 2 2 1 0 1 0 0 0 1 0 0 4 0 2 1 2 0 1 0 2 0 0 0 0 0
[48417] 2 0 2 1 0 1 2 0 0 0 2 2 0 1 0 1 1 0 0 0 2 1 1 1 1 2 2 0 0 1 1 2 1 1
[48451] 1 0 2 2 3 0 1 1 0 0 1 1 1 0 1 0 0 1 0 1 0 1 0 0 1 0 2 0 1 0 1 0 1 0
[48485] 0 0 2 0 0 0 0 0 1 1 1 0 1 1 2 1 0 2 0 1 1 0 0 0 1 3 2 1 1 0 0 0 0 1
[48519] 0 0 1 0 1 0 0 0 0 0 1 0 0 2 0 0 0 0 0 1 1 1 0 0 0 0 1 0 0 0 0 0 0 0
[48553] 3 3 0 2 1 0 1 0 1 1 0 1 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 2 1 2 0 0 0
[48587] 0 0 0 0 0 0 2 2 0 0 1 2 0 1 0 2 0 0 2 2 0 0 1 0 0 0 0 1 0 0 0 0 0 0
[48621] 0 0 0 0 0 0 0 0 1 0 2 1 1 0 1 0 0 0 0 0 0 0 0 0 2 0 2 1 1 0 0 0 1 0
[48655] 0 0 0 1 0 0 0 1 1 1 1 2 1 1 0 1 1 1 0 0 0 1 0 1 1 0 2 0 2 0 0 0 1 0
[48689] 0 0 1 0 1 1 0 0 0 3 0 0 0 1 0 0 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 0 0 0
[48723] 0 0 0 0 0 0 0 0 0 0 0 1 1 0 1 0 0 0 1 0 0 0 1 1 0 0 0 0 0 0 0 0 0 1
[48757] 0 0 0 0 0 0 2 1 0 1 2 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0
[48791] 0 0 0 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[48825] 1 0 0 1 0 0 1 0 0 0 0 1 1 0 0 0 0 1 0 1 1 0 1 1 0 0 4 2 0 1 0 0 0 1
[48859] 2 0 0 2 1 1 0 0 1 0 0 0 0 1 0 0 2 0 1 3 2 0 0 1 0 0 0 0 0 2 1 1 1 0
[48893] 0 1 0 1 0 0 0 0 1 0 1 0 1 1 0 1 0 0 0 0 1 0 0 1 0 0 2 2 1 2 0 1 0 0
[48927] 0 0 0 0 1 0 0 0 2 2 0 0 1 1 1 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 1 1 3 0
[48961] 0 0 1 0 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0 1 0 1 2 1 0 1 1 0 0 0 0
[48995] 0 2 1 0 0 0 1 0 0 0 0 1 4 1 1 0 0 0 0 0 0 2 0 0 0 0 1 1 0 0 0 1 1 0
[49029] 0 1 0 1 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 2 1 1 3 0 3 1 0 1 1 1 0 0
[49063] 0 0 0 2 0 0 3 1 1 2 2 1 3 0 0 0 0 1 0 0 0 2 1 1 0 0 0 0 2 0 0 2 3 1
[49097] 3 2 1 2 2 0 1 3 3 2 3 1 1 0 0 0 1 1 1 0 0 1 0 0 0 0 0 0 1 0 0 1 1 0
[49131] 1 0 2 2 0 1 2 3 0 1 1 0 0 0 0 0 2 2 3 1 2 0 2 1 1 1 0 0 0 2 1 1 2 0
[49165] 3 0 2 1 3 0 0 3 1 4 1 1 1 2 2 3 1 3 0 1 0 1 1 0 0 0 2 2 0 1 0 1 0 1
[49199] 3 0 0 0 0 1 1 0 0 0 3 2 0 1 0 0 2 2 1 3 2 0 0 0 1 1 0 0 1 2 0 0 1 1
[49233] 1 1 0 0 1 3 4 1 4 3 1 3 0 0 0 2 0 0 1 0 1 2 0 0 2 1 1 3 1 0 1 2 0 0
[49267] 1 2 2 1 4 1 2 1 1 1 1 1 0 2 0 2 1 0 1 1 0 0 1 0 2 2 3 1 1 3 0 1 0 1
[49301] 2 1 1 1 0 2 2 5 0 1 3 1 0 0 1 2 1 0 0 2 1 1 0 0 1 0 0 0 0 1 0 1 0 0
[49335] 0 0 0 2 0 1 1 1 0 0 0 0 0 1 0 1 0 0 0 0 0 0 1 1 2 0 1 1 0 0 0 1 1 0
[49369] 0 1 0 2 0 2 1 2 0 0 0 0 0 0 0 1 0 0 0 0 1 1 0 0 2 1 1 1 1 0 0 0 0 0
[49403] 1 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 1 1 1 0 1 2 1 1 1 1 2 0 1 3 0 0
[49437] 0 0 0 0 0 1 0 1 0 1 0 1 0 1 0 2 3 1 1 1 1 3 1 2 0 1 1 1 0 0 0 0 2 1
[49471] 1 1 0 1 0 0 0 1 0 1 0 2 1 0 0 2 1 0 0 1 0 0 0 0 1 0 0 1 1 0 0 0 0 2
[49505] 0 0 0 1 0 1 0 0 2 0 2 0 0 0 0 0 0 0 0 0 2 1 0 0 0 0 0 0 0 0 0 0 0 0
[49539] 0 0 0 1 2 0 0 2 1 0 2 0 0 1 0 3 0 1 0 2 0 2 0 0 0 0 1 0 0 0 2 0 1 0
[49573] 0 1 0 1 2 2 0 1 0 1 1 2 2 0 1 1 0 0 0 1 0 2 4 4 0 2 0 2 0 1 1 2 2 1
[49607] 0 0 1 0 0 1 1 0 0 0 0 0 0 2 0 1 0 0 1 5 3 4 1 1 0 2 1 0 0 1 2 1 2 2
[49641] 1 2 2 0 2 0 3 3 1 0 1 2 2 2 0 1 0 2 2 1 1 0 1 3 3 2 2 1 0 1 3 0 0 0
[49675] 0 1 0 0 0 2 3 1 1 0 1 0 1 1 0 2 0 0 1 1 0 0 0 1 1 0 0 0 0 0 0 0 0 0
[49709] 0 0 0 0 0 0 0 0 0 1 2 0 0 0 1 0 0 1 0 0 0 1 2 0 1 0 2 0 0 0 0 0 0 0
[49743] 0 0 0 0 0 0 0 0 0 1 0 0 1 0 2 1 3 1 1 2 2 0 1 0 0 2 1 0 0 0 0 1 0 2
[49777] 4 2 2 2 0 0 3 0 1 2 2 0 1 0 0 1 1 1 0 0 1 2 3 2 0 3 1 0 1 1 1 1 1 0
[49811] 0 0 1 0 0 2 1 1 0 2 1 2 0 2 0 0 1 1 1 0 1 2 3 1 1 1 0 0 0 0 2 2 0 1
[49845] 1 1 1 0 0 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 2 0 1 0 1 0 0 0 0 2 0
[49879] 2 0 0 0 2 1 1 0 0 1 0 0 1 1 3 1 0 2 1 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0
[49913] 0 0 2 2 2 1 3 3 1 2 1 1 0 2 2 0 0 1 2 0 0 1 0 0 1 1 0 1 0 1 0 0 1 1
[49947] 1 0 1 2 1 0 0 1 0 3 1 3 1 2 3 1 3 1 0 2 0 0 5 2 0 1 2 0 1 0 0 1 0 1
[49981] 0 0 0 0 0 0 0 0 0 0 0 2 2 0 0 0 0 0 0 1 0 0 3 2 5 0 1 3 1 2 0 1 0 0
[50015] 0 0 0 0 0 0 0 0 0 1 0 0 2 0 1 0 0 0 0 1 1 1 0 0 0 2 0 2 0 2 4 1 1 0
[50049] 1 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 1 0 0 0 1 1 0
[50083] 0 0 1 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 1 0 1 1 0 0 0 0 1
[50117] 0 0 0 1 0 3 1 0 0 0 1 0 1 0 0 0 0 2 1 1 0 0 0 0 1 0 2 1 0 0 0 1 1 0
[50151] 1 0 1 1 1 0 2 0 0 0 0 0 0 0 0 1 1 1 0 2 3 3 1 0 0 0 0 0 0 0 0 2 0 1
[50185] 2 1 0 0 0 0 0 2 0 0 1 0 1 2 0 0 0 0 0 0 1 1 1 1 0 0 0 2 0 0 2 2 1 1
[50219] 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 1 0 1 2 2 0 2 0 0 0 0
[50253] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 1 0 0 0 1 0 0 0 0 0 0 1
[50287] 0 0 2 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 1 1
[50321] 0 1 1 0 0 0 0 2 0 1 0 0 0 0 0 0 0 0 0 1 1 1 0 0 1 0 0 0 0 0 0 0 0 0
[50355] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 3 2 3 0 1 3 1 1 2 2 2 0 0 0 0 0
[50389] 0 0 0 0 0 0 0 0 1 0 2 0 0 1 0 0 1 2 0 0 0 0 1 1 0 1 3 0 2 0 0 0 0 1
[50423] 1 1 1 3 0 1 1 0 1 0 0 0 0 2 3 1 1 2 3 2 2 1 0 0 0 0 0 0 0 0 0 0 0 0
[50457] 1 1 0 1 0 0 3 1 0 1 1 0 3 0 0 0 2 1 0 0 0 2 1 1 3 1 1 2 3 1 1 0 0 0
[50491] 1 2 1 1 2 1 2 2 2 1 1 0 0 3 2 3 1 0 0 4 1 2 0 0 1 2 1 2 0 0 0 3 0 2
[50525] 0 0 1 0 0 0 0 0 1 1 1 0 1 0 0 2 0 1 2 2 1 0 0 0 0 0 1 0 0 0 0 2 0 0
[50559] 0 1 3 3 1 1 0 4 0 1 4 3 2 0 3 2 1 0 0 1 2 0 0 1 1 0 3 2 1 3 2 1 1 2
[50593] 1 1 2 3 4 1 1 0 3 0 0 1 0 0 2 1 0 0 1 0 0 0 0 0 1 2 2 3 7 1 1 0 1 4
[50627] 1 1 4 1 1 3 0 3 1 1 0 0 0 1 0 0 0 1 1 1 0 0 0 0 0 0 1 0 0 1 0 0 0 0
[50661] 1 1 2 0 0 1 2 1 0 2 2 1 0 2 0 1 0 0 2 1 0 2 1 1 2 1 1 0 3 0 1 0 1 1
[50695] 0 2 0 0 1 1 4 2 2 1 0 0 0 0 0 1 1 0 1 1 4 0 1 5 2 1 0 0 1 0 0 0 0 0
[50729] 1 2 3 1 2 2 2 0 1 2 0 2 0 0 1 1 1 2 1 0 2 0 0 0 0 0 0 0 0 0 3 2 0 0
[50763] 0 0 0 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 2 1 2 0 2 0 0 3 2 0 1 0 0 0
[50797] 2 1 0 2 1 0 0 2 0 0 0 0 0 1 0 0 0 0 0 1 2 1 1 0 0 0 0 0 4 0 0 1 0 2
[50831] 1 0 1 1 1 1 1 1 1 1 0 0 2 0 0 2 0 0 0 0 0 0 0 1 0 0 0 0 0 2 0 0 0 0
[50865] 0 1 0 0 2 1 1 1 0 1 2 1 1 4 0 2 1 1 2 0 1 0 2 0 0 0 2 0 1 0 0 0 0 2
[50899] 1 1 0 0 0 1 1 0 2 0 1 0 0 1 2 2 0 0 1 0 0 0 2 0 2 0 1 0 1 0 0 0 1 1
[50933] 0 0 0 1 0 0 1 0 1 0 0 0 0 1 1 1 1 1 0 0 2 0 2 0 2 0 0 0 0 0 0 0 0 0
[50967] 1 2 1 0 1 0 0 0 0 0 0 1 1 0 2 2 0 1 0 0 1 0 1 0 0 0 0 0 0 0 0 0 1 0
[51001] 0 1 0 0 0 0 0 0 1 1 0 1 1 0 1 0 1 0 0 0 2 0 0 0 0 1 0 0 0 0 0 0 1 1
[51035] 0 0 1 1 0 0 0 0 1 0 0 0 0 0 1 1 0 0 1 0 0 0 0 1 0 1 0 0 0 0 0 0 3 0
[51069] 1 1 0 0 1 3 3 1 0 0 2 0 2 1 0 1 2 1 2 0 0 0 1 0 1 2 1 0 1 2 0 0 1 1
[51103] 1 3 2 1 0 1 1 1 1 1 1 2 0 0 1 0 1 0 0 0 0 1 2 0 0 1 0 0 0 0 2 0 1 2
[51137] 1 1 2 0 1 4 0 1 2 1 0 3 2 1 0 0 1 1 0 2 2 1 0 0 2 2 2 2 1 5 2 1 1 2
[51171] 3 3 0 1 0 2 1 0 2 0 2 3 1 0 0 0 0 1 1 0 0 0 1 0 3 1 2 1 0 1 0 0 0 2
[51205] 1 1 2 1 0 2 1 1 0 1 1 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 2 0
[51239] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 2 1 1 0 1 0 0 0 1 1 0 2 0 0 0 0 0 1
[51273] 1 0 1 1 0 1 0 0 0 0 0 0 0 0 1 0 0 0 2 0 1 0 3 1 3 1 0 3 1 0 2 0 0 0
[51307] 0 1 1 0 0 0 1 0 2 1 0 1 0 1 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0
[51341] 1 0 0 1 0 0 0 1 1 2 3 0 0 0 1 0 0 0 1 1 1 0 1 2 0 1 0 1 1 3 0 0 0 0
[51375] 5 0 1 0 1 1 1 2 0 1 1 1 0 0 0 0 1 0 0 0 0 1 0 0 1 0 1 0 1 0 3 1 0 0
[51409] 0 0 1 1 0 2 0 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 2 1 1 1 0 0 0 0 0 0 0 0
[51443] 1 0 0 3 0 0 0 0 3 1 0 0 0 1 0 0 0 0 1 0 0 0 0 1 0 1 0 0 0 0 1 0 0 0
[51477] 0 0 0 0 1 0 2 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 1 0 2 0 0 0 2 0 0 0 0 0
[51511] 0 1 0 0 0 0 0 1 2 0 0 0 0 0 0 0 1 0 0 1 1 0 0 0 0 0 1 0 0 0 1 0 0 1
[51545] 0 0 1 1 0 0 1 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 0 0 1 0 0
[51579] 0 0 0 1 1 0 0 1 0 0 2 2 0 0 0 1 1 0 0 0 1 2 1 1 1 1 0 0 1 2 0 1 2 1
[51613] 1 0 1 3 1 1 0 0 3 0 0 0 0 1 1 3 0 1 1 1 0 1 0 0 0 2 1 1 2 0 0 2 0 1
[51647] 0 1 2 0 1 0 0 0 0 0 0 2 1 0 1 1 1 0 0 0 2 1 1 3 0 0 0 0 1 1 0 0 0 0
[51681] 0 1 0 1 1 0 0 0 1 0 0 0 0 0 0 1 2 1 0 1 1 1 2 1 0 1 0 0 0 0 0 2 0 0
[51715] 0 1 2 0 1 2 2 0 0 0 1 0 0 0 3 1 1 1 0 0 2 0 1 1 0 0 0 0 1 0 2 1 0 0
[51749] 0 1 1 0 0 0 0 0 0 0 0 1 2 0 0 1 2 1 0 0 1 0 1 1 2 2 1 0 0 0 0 0 1 0
[51783] 0 3 1 0 0 2 2 0 0 0 0 1 2 0 2 1 1 0 0 1 2 0 1 4 1 1 0 0 0 2 2 1 1 0
[51817] 1 1 1 1 0 0 0 1 0 0 0 0 0 0 1 0 0 0 1 2 1 2 1 2 0 1 0 3 0 1 2 0 0 0
[51851] 1 0 3 1 1 1 1 0 0 0 1 1 1 0 0 2 0 0 1 1 0 0 1 1 1 3 1 0 0 4 1 1 1 0
[51885] 1 1 1 2 0 1 0 1 0 1 0 0 2 1 4 1 1 0 3 1 1 0 1 1 1 0 2 0 0 2 1 0 0 2
[51919] 1 3 1 0 2 0 1 1 1 1 1 0 1 2 1 1 1 1 1 0 1 1 1 1 4 1 0 1 1 1 1 1 2 3
[51953] 2 1 0 3 1 1 1 0 1 2 1 0 1 0 0 0 0 1 1 1 2 3 2 2 2 3 1 3 0 0 0 3 0 1
[51987] 1 0 1 1 1 0 0 1 1 0 1 1 0 0 1 3 0 1 0 0 1 0 1 0 2 0 0 1 1 2 1 1 1 0
[52021] 2 0 0 2 0 0 1 0 0 0 0 0 0 0 1 1 0 1 0 1 2 0 0 0 2 0 1 1 3 2 1 0 0 0
[52055] 0 0 2 1 0 0 0 0 1 1 0 0 0 1 0 0 0 0 0 0 0 0 0 4 0 2 0 1 1 2 0 2 2 2
[52089] 0 0 0 1 0 1 0 1 2 1 1 0 0 1 0 0 1 0 0 1 0 3 0 0 2 0 0 0 0 0 0 1 1 0
[52123] 0 0 1 2 2 0 2 2 1 1 0 0 1 1 2 3 2 0 1 1 2 0 1 1 0 0 0 0 1 0 1 1 0 0
[52157] 1 1 2 0 0 0 0 1 0 1 0 0 1 0 1 0 0 0 0 0 0 1 0 1 0 0 0 1 0 1 0 0 0 0
[52191] 0 2 0 1 1 0 0 0 0 0 0 0 0 1 0 2 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0
[52225] 1 0 0 1 0 0 0 0 0 0 0 0 0 0 1 0 2 0 0 0 1 0 0 1 0 0 0 3 2 0 0 0 2 0
[52259] 0 0 1 0 0 2 1 0 0 0 2 1 1 0 1 1 2 0 0 0 1 0 0 0 0 0 1 0 3 1 1 0 0 0
[52293] 1 0 1 0 1 0 0 0 0 0 1 3 1 0 1 1 0 1 0 0 2 2 0 0 0 1 0 1 0 0 1 2 0 0
[52327] 1 0 0 0 0 0 0 0 0 0 0 0 1 0 1 1 1 0 0 0 0 0 0 0 0 2 2 2 1 0 0 0 1 1
[52361] 0 1 0 1 1 1 1 0 1 0 0 0 0 1 1 0 0 2 0 1 0 0 0 1 1 1 1 3 0 1 2 1 1 1
[52395] 0 0 0 0 0 0 0 0 0 1 0 0 0 0 2 0 0 0 1 0 0 0 1 0 0 0 0 0 2 0 1 0 1 0
[52429] 0 0 0 0 2 0 0 0 1 0 0 1 0 0 0 2 0 1 1 2 0 3 2 3 4 1 1 1 1 2 1 2 0 1
[52463] 1 0 1 0 0 2 2 1 3 1 1 2 1 0 2 0 2 0 0 1 2 1 1 0 0 2 0 1 1 0 0 0 0 0
[52497] 2 2 0 0 1 0 1 0 0 0 1 1 0 2 0 0 1 0 0 1 0 2 1 1 0 0 0 0 1 0 0 0 1 0
[52531] 0 0 1 1 2 1 0 1 1 2 0 0 0 0 1 1 1 0 0 1 0 0 0 0 0 2 3 2 3 3 0 1 2 3
[52565] 0 0 1 0 0 0 0 1 0 2 1 1 0 2 1 0 0 0 2 1 2 1 0 0 1 0 2 1 1 1 1 0 0 2
[52599] 0 2 0 0 0 0 0 1 2 0 0 0 1 0 0 2 2 1 0 0 1 0 2 0 2 1 1 0 1 0 0 1 0 1
[52633] 1 0 1 0 0 0 0 0 1 0 1 0 1 1 2 0 0 0 0 0 0 1 1 0 0 0 2 0 0 0 0 0 0 1
[52667] 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 3 0 0 0 1 0 0 0 0 1 0 0
[52701] 0 3 1 1 1 0 1 0 1 0 0 0 1 2 0 1 2 3 0 0 0 0 3 3 1 0 1 1 3 0 1 1 0 3
[52735] 1 1 2 2 0 0 0 2 0 1 2 2 2 0 2 3 0 1 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0
[52769] 1 1 1 2 1 0 0 0 0 2 1 1 1 1 3 0 0 0 0 0 0 0 0 0 0 1 0 0 0 1 0 2 0 0
[52803] 1 1 0 1 0 0 1 1 0 0 2 0 0 2 0 0 0 1 1 1 2 1 0 0 0 3 1 4 1 0 1 0 1 2
[52837] 3 0 1 0 0 0 0 1 0 2 1 2 2 3 1 3 1 0 2 4 2 2 0 1 2 3 0 1 1 3 1 2 0 0
[52871] 1 3 2 1 0 2 0 0 0 0 0 0 2 2 1 2 0 1 1 2 1 0 0 1 1 0 2 0 2 3 1 0 0 0
[52905] 1 0 0 0 0 0 2 2 0 0 0 1 1 0 0 2 0 0 0 0 0 2 1 2 3 0 2 0 0 1 0 0 0 0
[52939] 0 0 1 1 0 0 0 0 1 1 0 0 0 1 3 2 0 0 0 0 0 0 0 2 2 0 1 1 0 0 0 0 0 0
[52973] 0 1 0 0 1 3 1 0 0 1 1 0 1 0 0 0 0 0 0 1 0 0 0 2 1 0 0 1 0 0 1 1 0 0
[53007] 1 1 0 0 1 1 0 0 0 1 0 2 1 0 1 0 0 0 0 0 0 0 0 1 0 0 0 0 1 0 0 1 1 0
[53041] 0 1 1 0 4 1 0 1 2 1 1 1 3 2 2 2 1 1 1 0 1 0 0 0 0 0 3 1 0 1 2 0 0 1
[53075] 0 0 2 3 2 1 1 0 0 1 0 0 1 1 0 0 0 0 1 0 0 1 1 1 2 0 2 0 1 1 1 0 1 0
[53109] 0 1 0 1 0 0 0 0 2 0 0 1 0 1 0 3 1 1 0 2 1 3 1 1 0 0 0 2 2 1 0 0 2 2
[53143] 0 1 0 0 0 0 2 1 1 1 1 0 2 0 1 1 1 1 2 0 0 0 0 0 0 0 0 0 1 0 0 0 0 1
[53177] 1 0 0 0 1 1 0 0 0 0 0 0 1 0 1 2 0 1 2 2 2 0 3 0 0 1 0 2 1 0 1 0 0 0
[53211] 1 3 6 2 1 1 0 1 0 1 1 0 0 1 4 1 0 0 0 0 0 1 0 0 0 0 1 1 0 0 1 0 0 0
[53245] 0 0 0 1 1 2 0 1 1 0 0 0 0 1 0 0 0 0 0 1 1 0 0 0 0 0 2 3 0 1 0 4 0 0
[53279] 0 0 0 0 0 0 0 0 0 0 2 0 0 0 1 0 0 0 0 0 0 0 2 0 0 0 0 0 0 1 0 1 1 2
[53313] 2 1 1 0 0 2 0 0 0 0 0 1 1 1 1 3 1 0 1 0 0 0 0 1 0 2 1 0 0 1 2 1 0 0
[53347] 0 0 1 0 1 1 1 1 0 0 0 1 0 1 3 0 0 0 0 0 1 0 0 0 0 0 2 2 2 0 0 1 1 1
[53381] 2 2 1 0 0 0 0 1 3 0 2 1 1 0 0 0 0 0 0 0 0 0 0 2 0 2 1 0 0 0 1 0 0 0
[53415] 0 0 0 0 0 1 3 2 1 1 0 1 3 0 0 0 0 0 1 1 2 0 0 0 0 0 0 0 1 0 0 0 0 0
[53449] 1 0 1 1 0 0 1 1 0 2 1 1 1 0 0 0 2 0 2 1 0 0 0 0 0 0 0 1 1 1 1 0 1 1
[53483] 1 0 1 1 4 0 1 0 0 0 1 0 0 1 0 1 2 1 1 1 2 0 0 1 2 0 1 0 0 0 2 1 0 0
[53517] 1 1 1 0 0 0 1 0 2 0 1 0 1 1 1 0 0 1 0 1 2 2 4 2 2 0 1 2 1 3 0 0 0 0
[53551] 0 0 1 0 2 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 2 0 0 0 0 0 0
[53585] 0 0 1 0 1 0 0 0 0 1 0 0 0 0 0 1 1 0 1 1 0 1 3 1 0 0 0 0 1 0 0 0 1 1
[53619] 0 0 0 0 0 0 1 1 1 0 1 0 0 1 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 2 1 1
[53653] 0 0 2 2 1 1 1 2 0 1 0 2 2 0 0 0 0 0 1 0 3 1 0 0 0 1 2 2 1 2 1 2 0 0
[53687] 1 0 1 1 3 0 0 1 1 2 1 0 1 1 0 2 0 1 0 0 0 0 1 0 2 1 0 2 0 0 1 1 0 0
[53721] 2 0 1 0 0 0 0 2 1 1 1 0 0 0 0 0 1 0 0 0 0 1 0 0 1 0 1 2 1 0 1 1 0 0
[53755] 1 1 0 0 0 0 0 0 0 0 0 1 1 0 0 1 0 1 1 0 0 0 0 3 1 3 0 1 1 0 0 0 1 0
[53789] 0 1 0 0 0 1 0 1 0 0 0 0 0 1 0 0 0 2 0 0 1 0 1 1 2 1 2 0 0 2 0 1 0 1
[53823] 1 1 0 0 0 0 0 0 1 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1
[53857] 0 0 0 0 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 0 1 0 0 1 0 2 0 0 0 2 1 1 0 0
[53891] 0 0 0 0 0 0 0 2 0 1 1 0 0 3 0 1 1 0 1 1 1 3 0 2 0 0 0 0 1 1 0 0 2 0
[53925] 1 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 1 0 1 1 0 0 0 0 0 0 0 0 1 0 2 1 1 0
[53959] 0 0 1 0 0 0 0 0 0 1 0 2 0 1 3 0 0 0 0 1 2 1 0 2 0 1 1 0 1 0 2 0 1 0
[53993] 1 0 1 0 1 0 0 1 0 1 0 2 0 1 1 0 0 0 0 0 0 0 0 0 0 1 2 0 0 0 1 0 1 0
[54027] 1 1 1 1 0 1 3 0 1 1 1 0 0 0 0 1 1 0 0 1 0 1 0 0 0 0 2 0 0 0 2 0 0 1
[54061] 0 1 3 0 0 0 0 0 1 0 0 0 1 1 1 0 0 0 0 1 0 1 0 0 1 0 0 0 0 0 0 0 1 0
[54095] 1 0 0 0 0 0 0 0 0 2 0 1 1 0 0 0 0 1 0 0 0 1 1 2 0 0 1 2 1 0 1 1 0 1
[54129] 1 0 0 0 0 1 3 0 0 0 1 0 2 0 0 1 1 0 2 0 0 1 1 1 0 0 1 0 1 1 3 0 1 0
[54163] 1 0 0 0 2 3 0 1 0 4 2 0 0 0 0 2 2 1 2 2 1 0 0 1 0 2 0 3 0 0 0 1 0 0
[54197] 1 0 1 0 1 2 0 0 0 0 1 0 1 1 0 1 2 1 1 0 0 2 1 1 1 1 0 0 0 1 1 0 0 0
[54231] 0 0 0 2 1 0 0 0 1 0 2 0 0 0 0 4 0 0 0 0 0 2 2 1 0 0 0 3 0 1 1 0 1 0
[54265] 1 1 1 0 2 1 0 2 0 1 1 0 1 0 3 2 0 0 0 0 2 2 1 1 1 0 1 0 0 0 1 0 0 1
[54299] 0 0 1 1 0 2 0 0 1 0 2 1 0 0 0 2 1 1 0 0 1 1 1 0 0 1 0 1 2 1 3 2 0 0
[54333] 0 0 1 2 1 0 1 1 1 1 1 2 0 3 1 0 1 0 0 0 2 0 2 0 0 1 1 0 2 2 0 2 2 2
[54367] 1 0 1 2 0 1 1 0 0 1 0 0 0 0 1 0 0 0 0 0 1 1 0 1 0 0 1 1 1 1 0 2 1 0
[54401] 0 0 0 1 0 1 1 1 0 0 0 1 0 0 1 0 0 0 0 0 0 1 1 0 1 0 2 3 1 1 0 0 0 0
[54435] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 1 0 0 0 0 0 0 0 0 0 0 1
[54469] 1 0 0 1 1 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 1 2 2 0 2 0 0
[54503] 0 0 0 1 0 3 0 0 1 2 0 0 0 1 0 1 0 1 0 1 0 1 2 0 0 0 0 1 0 1 0 0 0 0
[54537] 1 2 0 5 0 0 1 1 2 1 1 1 1 1 0 0 0 0 3 0 0 0 0 0 1 0 0 0 0 0 0 0 0 2
[54571] 1 0 0 0 1 0 2 2 0 1 2 0 0 0 1 1 3 1 1 1 0 0 0 0 0 0 1 1 1 1 0 3 0 1
[54605] 0 1 1 1 0 1 0 2 0 0 1 0 0 1 0 0 0 2 0 0 1 1 2 1 4 0 0 0 1 1 2 0 0 0
[54639] 3 1 0 4 0 1 0 0 2 1 0 2 1 0 0 1 1 1 0 3 1 1 1 0 0 3 0 0 1 1 0 0 1 1
[54673] 0 2 0 1 1 0 2 0 0 0 0 2 3 1 3 2 1 3 0 2 1 0 0 0 0 1 1 0 0 0 0 0 0 0
[54707] 0 0 0 0 0 0 1 0 1 0 0 1 1 4 3 1 0 0 0 0 1 0 0 0 0 0 0 0 0 2 1 0 0 1
[54741] 1 1 0 0 0 0 2 0 0 1 1 0 0 1 1 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 0
[54775] 0 0 1 0 0 3 0 1 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0 0 0 1
[54809] 1 0 1 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0
[54843] 0 1 0 0 0 0 0 2 1 0 0 0 0 0 0 1 0 0 0 0 0 1 1 0 0 0 1 1 0 0 1 0 0 0
[54877] 0 0 1 0 1 2 0 0 0 1 0 0 0 0 0 2 1 0 0 0 1 0 0 0 1 2 1 0 1 0 1 1 0 1
[54911] 0 2 0 0 0 2 0 0 1 2 0 1 1 0 0 0 0 0 0 0 1 0 1 1 1 0 1 1 0 0 1 0 1 0
[54945] 1 0 0 0 0 0 0 1 0 0 0 0 0 1 1 0 2 0 0 1 0 1 2 2 0 3 1 0 1 0 1 0 1 2
[54979] 1 0 1 1 0 0 1 0 0 0 2 1 2 0 2 0 0 0 0 0 2 1 2 0 2 2 0 3 1 1 2 5 2 1
[55013] 0 0 2 2 1 0 3 2 3 0 0 0 1 2 1 1 1 1 2 4 2 2 1 0 0 0 0 1 1 0 0 0 0 1
[55047] 1 0 1 1 0 0 1 2 0 0 0 0 2 1 0 0 1 0 0 0 0 1 1 2 3 0 0 0 0 1 0 0 0 2
[55081] 2 0 4 0 0 0 1 2 0 1 1 1 0 0 1 3 0 3 0 2 1 1 1 2 0 0 0 0 0 0 0 1 0 3
[55115] 1 0 0 1 0 0 1 1 1 3 1 1 0 0 2 1 0 0 2 0 2 2 0 1 0 0 1 1 1 1 0 0 0 1
[55149] 3 1 0 0 1 0 0 0 0 1 1 1 1 1 0 0 0 0 0 0 1 0 1 0 0 2 0 2 1 3 0 0 1 1
[55183] 1 2 0 0 0 2 0 0 1 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 1 0 0 0 0 1 0 2
[55217] 0 1 0 2 0 2 0 1 1 0 2 1 0 0 3 1 0 1 0 1 0 1 2 1 0 0 0 0 0 1 2 1 0 2
[55251] 0 0 1 0 1 2 0 1 1 3 0 2 1 2 0 1 0 0 0 1 1 1 0 1 0 0 0 0 0 1 1 0 0 0
[55285] 1 1 0 0 0 1 0 0 0 0 0 0 0 0 1 0 1 1 0 1 1 1 0 1 0 0 0 2 0 1 0 0 1 2
[55319] 0 0 0 0 0 2 0 0 0 0 0 0 1 0 0 0 2 1 4 0 1 0 1 2 0 2 3 1 0 0 0 0 1 0
[55353] 0 0 0 1 1 1 0 1 1 0 1 2 0 1 3 4 1 0 0 0 1 1 0 2 1 1 2 0 1 2 2 0 0 1
[55387] 0 1 0 4 2 1 3 1 1 1 1 0 3 0 1 0 0 0 1 3 2 1 2 1 0 1 0 1 0 0 0 0 1 1
[55421] 2 0 0 0 0 1 1 1 1 0 0 0 1 0 0 1 1 0 0 0 0 1 1 1 0 0 0 0 0 1 0 0 0 0
[55455] 0 0 0 0 0 0 1 0 1 0 0 0 0 0 1 0 0 0 0 0 1 1 1 2 0 1 1 1 1 0 1 1 0 0
[55489] 0 0 1 0 0 0 0 1 2 4 1 0 1 0 0 1 1 2 1 0 0 0 1 2 3 1 1 0 0 0 1 0 0 0
[55523] 1 1 0 0 0 1 0 0 1 2 1 1 0 0 0 0 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1
[55557] 0 0 1 0 0 0 0 0 0 1 1 0 2 0 0 0 0 0 0 0 0 0 2 1 0 0 0 2 1 0 0 0 0 2
[55591] 0 2 1 1 2 0 1 2 0 2 1 1 4 1 1 1 1 4 4 4 1 0 0 0 0 0 0 0 0 0 0 1 1 1
[55625] 0 0 1 0 0 4 0 1 0 2 0 0 0 1 0 1 2 2 1 1 1 0 0 0 1 0 0 0 0 1 0 1 2 1
[55659] 0 1 0 0 1 0 1 0 0 1 0 1 0 2 1 1 3 1 0 0 1 4 2 3 0 1 1 0 1 1 1 1 2 2
[55693] 5 2 2 2 3 1 0 0 1 0 0 0 1 0 1 0 0 1 0 1 1 3 1 1 0 0 2 0 1 3 0 0 0 1
[55727] 2 0 1 0 2 1 1 0 0 0 0 0 0 0 1 0 0 2 0 0 2 0 0 0 0 0 1 1 1 0 2 0 0 4
[55761] 1 0 0 1 0 0 0 1 1 3 0 1 0 2 0 1 1 1 0 3 0 1 1 0 5 1 2 2 1 1 1 1 0 1
[55795] 1 1 0 2 1 2 2 1 3 0 0 1 0 0 0 1 1 0 0 1 0 0 0 0 0 1 1 1 1 0 0 0 0 1
[55829] 2 2 2 0 0 0 1 0 0 0 1 0 1 1 1 0 0 0 0 1 0 0 2 1 2 1 0 0 0 1 1 2 1 0
[55863] 0 0 1 0 0 0 2 2 2 0 0 0 0 1 1 0 0 1 0 0 1 0 1 1 1 0 0 0 0 0 1 1 0 1
[55897] 0 1 3 0 0 0 0 1 1 0 1 0 1 1 0 0 3 0 1 0 0 0 0 0 0 1 0 1 1 2 0 1 0 1
[55931] 1 1 0 2 0 0 0 1 1 3 3 3 0 0 0 0 0 0 2 1 1 3 1 0 0 2 2 2 2 1 0 0 1 1
[55965] 2 2 1 3 2 4 3 0 0 1 2 0 2 2 1 1 1 0 0 0 2 0 1 0 0 1 0 1 2 1 1 1 1 0
[55999] 0 1 0 0 0 0 0 0 0 0 1 1 0 0 1 1 0 1 0 1 1 0 1 0 1 0 2 1 2 0 0 1 0 1
[56033] 1 1 1 2 0 0 2 0 1 0 1 1 0 1 1 0 0 0 0 1 0 0 0 0 1 1 1 0 0 0 0 1 0 0
[56067] 0 0 2 1 0 1 0 2 1 1 1 0 0 2 0 0 0 0 0 0 3 1 0 2 2 0 1 1 0 1 1 0 0 0
[56101] 0 1 0 3 1 2 0 3 0 0 0 1 0 0 0 0 1 1 3 3 1 0 0 0 0 0 0 2 0 2 1 1 1 2
[56135] 0 1 1 1 0 0 0 0 0 3 0 0 0 2 0 0 0 0 0 1 0 0 1 1 1 1 0 0 1 2 0 0 1 0
[56169] 1 1 3 0 0 0 0 0 0 1 1 1 1 0 1 0 0 3 1 0 0 0 1 0 3 2 2 3 0 0 0 2 1 2
[56203] 0 2 1 0 1 1 1 1 2 1 2 1 0 1 2 3 0 1 0 1 0 1 2 0 1 1 0 1 2 1 1 0 0 0
[56237] 0 0 0 1 1 2 0 0 1 1 0 2 0 0 1 3 3 3 2 3 0 2 0 2 0 0 0 1 0 2 0 0 0 2
[56271] 2 2 1 0 0 2 2 0 0 0 1 0 0 0 0 1 0 0 2 3 0 4 1 1 2 2 2 0 0 0 0 0 0 0
[56305] 1 1 3 1 2 0 1 0 1 0 2 0 1 0 0 0 0 1 2 0 0 5 1 0 2 2 2 1 1 1 2 0 1 0
[56339] 3 1 0 2 2 2 1 1 0 0 0 0 0 2 0 2 2 1 1 2 2 0 0 0 0 0 0 1 0 1 1 0 0 0
[56373] 2 4 1 0 0 0 0 1 0 2 1 0 0 0 2 0 1 0 1 0 2 1 1 0 0 0 2 0 0 0 3 2 1 0
[56407] 0 2 1 0 0 2 1 1 1 2 0 0 1 0 2 3 1 1 2 1 0 0 1 2 0 0 1 4 0 0 0 0 0 0
[56441] 0 0 0 0 0 0 1 2 0 1 0 0 1 1 1 0 0 0 0 0 2 0 1 0 1 0 1 3 1 1 2 1 0 0
[56475] 0 2 0 1 1 0 1 2 1 0 0 0 1 3 1 0 1 0 0 2 1 0 1 1 1 1 1 0 0 1 1 1 0 0
[56509] 1 0 1 0 1 2 0 0 0 0 1 1 2 3 2 1 0 4 0 1 2 4 1 1 1 0 1 1 0 2 0 1 0 1
[56543] 0 0 0 0 1 0 0 4 1 1 1 1 0 0 0 3 0 0 0 1 1 1 1 1 0 1 1 2 0 1 3 0 1 0
[56577] 0 0 2 0 0 0 0 0 1 1 2 0 0 0 0 2 1 1 0 1 0 2 2 1 0 1 0 0 2 0 0 0 1 0
[56611] 3 3 2 1 1 0 2 0 0 0 0 0 0 1 1 1 0 0 0 2 0 2 1 0 2 0 1 0 0 0 0 0 1 0
[56645] 0 1 0 1 0 2 0 0 2 2 3 1 0 0 0 1 0 1 0 0 0 1 0 0 1 0 0 1 0 0 0 0 0 3
[56679] 1 2 1 0 0 0 0 0 1 1 0 0 0 0 1 0 1 1 1 0 1 0 0 0 0 0 0 0 1 0 0 0 1 0
[56713] 0 1 0 0 0 0 1 0 0 0 1 0 1 0 0 0 0 0 0 0 1 1 0 1 0 0 0 0 0 1 0 0 0 0
[56747] 0 1 0 0 0 1 0 0 1 0 1 0 1 0 2 1 0 1 0 0 1 1 0 0 1 0 1 0 0 0 0 0 0 0
[56781] 0 0 0 0 0 0 0 0 0 2 0 1 1 0 0 0 0 1 1 1 0 0 2 1 1 1 3 0 0 1 1 0 0 0
[56815] 1 0 2 1 0 0 0 0 1 0 0 0 2 1 2 1 0 1 0 1 0 0 0 0 0 0 0 0 0 0 1 0 0 0
[56849] 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 2 0 1
[56883] 0 0 0 0 0 1 2 2 0 1 0 0 0 2 2 0 1 3 1 2 0 1 0 0 1 0 0 0 1 0 0 1 0 0
[56917] 2 2 1 1 0 1 0 1 0 0 0 0 1 0 0 2 0 0 1 0 2 0 1 0 0 0 0 0 0 0 1 1 0 0
[56951] 1 0 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 1 1 2 0 0 0 0 0 0
[56985] 0 0 0 0 1 0 1 0 0 0 1 1 0 1 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 2 0 0 0
[57019] 0 2 1 0 1 0 1 0 1 2 1 1 0 0 0 0 0 0 0 1 1 1 0 0 0 1 1 0 0 0 0 1 0 0
[57053] 0 0 0 1 1 1 0 2 0 0 0 0 0 2 1 0 0 0 0 0 1 0 0 0 1 1 0 0 0 0 0 2 0 0
[57087] 0 0 0 1 3 1 1 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0
[57121] 0 1 0 0 2 1 2 0 1 0 1 2 0 0 0 1 0 0 0 1 1 0 0 0 0 0 1 0 0 1 0 1 0 2
[57155] 0 0 0 0 0 0 0 0 0 2 1 2 0 1 0 0 1 0 0 1 2 1 2 1 1 1 0 0 1 1 1 0 0 0
[57189] 1 0 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 0 1 1 0 0 0
[57223] 0 0 0 0 0 0 0 0 0 0 0 1 0 1 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[57257] 0 0 0 1 1 1 0 0 0 0 0 0 1 0 0 0 1 0 0 0 1 0 2 3 1 0 0 1 0 1 1 0 0 0
[57291] 0 1 2 0 1 0 1 0 0 0 0 1 1 0 0 1 0 1 0 0 0 0 1 0 0 0 0 0 1 0 0 0 1 0
[57325] 2 0 0 0 0 0 1 1 1 1 1 2 1 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1
[57359] 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 2 0 0 0 0 0 0 0 0 0 0 0 0
[57393] 0 0 0 0 0 0 2 1 1 0 1 0 0 0 1 3 3 1 2 1 2 3 0 2 3 0 1 3 1 1 0 0 0 2
[57427] 1 0 1 0 0 1 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 1 2 2 0 0 1 0 0 0 0 1 0
[57461] 1 0 0 0 1 1 1 0 2 0 0 0 1 1 0 1 1 2 0 1 0 0 3 1 0 1 0 2 0 1 0 1 1 0
[57495] 0 2 2 0 0 0 2 0 1 2 1 1 0 2 0 1 2 1 1 3 5 1 0 2 0 1 0 2 1 0 0 1 1 0
[57529] 0 0 0 1 1 4 0 0 0 1 3 0 0 0 0 0 1 1 1 1 1 1 0 0 1 0 0 0 1 0 0 0 0 1
[57563] 2 2 0 1 0 1 2 0 0 0 1 2 0 1 0 0 0 0 1 3 3 2 1 1 1 0 1 0 2 0 0 0 1 0
[57597] 1 1 0 1 1 0 1 1 0 0 0 1 0 2 1 2 0 0 0 0 1 0 0 1 0 1 0 0 2 1 1 0 0 0
[57631] 1 1 1 0 0 0 2 1 1 1 0 3 3 0 1 0 1 0 0 1 4 0 0 0 0 0 1 2 0 1 0 1 0 2
[57665] 0 0 0 2 1 0 1 1 1 0 0 0 0 0 0 0 1 1 0 0 1 1 1 0 0 0 0 1 1 0 1 1 0 3
[57699] 1 3 0 0 2 0 1 3 1 2 1 3 1 3 0 3 0 0 0 0 1 1 1 0 0 0 0 1 0 0 0 2 1 0
[57733] 2 0 0 1 0 1 0 1 0 0 0 1 0 2 0 0 0 0 0 1 0 2 0 4 1 0 1 0 1 0 0 2 0 0
[57767] 0 0 0 4 0 0 1 1 0 0 0 0 1 0 1 0 0 0 1 1 1 1 0 1 0 1 1 1 1 1 2 1 0 0
[57801] 1 2 2 0 0 1 0 2 1 2 0 2 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1
[57835] 3 4 2 2 0 3 1 3 1 2 2 2 0 0 0 0 0 0 0 0 1 1 1 0 0 0 0 0 1 1 1 0 0 0
[57869] 1 1 3 1 0 0 1 0 2 0 0 0 0 1 0 1 1 0 0 0 0 0 1 1 1 0 0 0 1 1 1 2 1 0
[57903] 1 0 0 2 1 1 1 2 1 0 0 1 0 2 1 2 0 0 0 1 3 0 2 0 0 2 0 0 0 4 3 1 3 1
[57937] 2 2 0 1 1 1 2 1 0 1 0 0 0 1 0 1 0 0 1 0 0 1 0 0 0 0 0 0 1 1 0 1 0 0
[57971] 0 0 1 0 2 0 0 0 0 2 2 1 0 2 0 0 0 0 0 0 1 0 0 0 0 0 0 2 0 1 1 0 0 0
[58005] 1 0 0 0 0 2 1 2 0 1 1 1 4 1 0 1 1 1 0 0 0 0 1 2 1 1 1 0 1 1 3 1 0 0
[58039] 0 1 0 0 0 0 0 0 0 0 1 2 1 0 0 0 0 0 0 1 2 2 0 0 0 1 0 2 0 0 0 1 1 0
[58073] 0 0 0 1 3 1 1 0 1 0 0 0 2 2 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 2
[58107] 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 1 1 1 0 1 2 2 0 1 1 0 2
[58141] 0 2 1 1 1 1 0 3 0 0 0 0 1 0 0 1 0 1 2 1 1 2 1 0 0 1 0 2 1 0 0 0 1 1
[58175] 0 0 1 0 1 2 1 0 0 0 0 0 0 1 0 0 0 1 0 2 0 0 0 0 0 0 0 0 0 0 0 1 0 0
[58209] 0 0 0 3 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 1 0 0 0 1 0 3 2 0 0 0 0 0 0 0
[58243] 1 2 0 0 0 1 1 1 1 0 0 0 2 1 0 2 1 0 0 2 0 2 0 0 0 0 0 0 1 0 1 1 1 1
[58277] 0 4 2 1 0 0 0 0 0 0 0 2 1 1 0 0 0 0 0 0 1 1 0 1 0 1 0 1 1 2 2 0 1 1
[58311] 0 3 0 0 0 1 0 2 0 0 0 1 0 0 0 0 1 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[58345] 0 1 0 0 0 1 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 1 0 0 1 0 1 0 0 0 1
[58379] 0 1 1 1 0 0 0 0 1 1 3 1 0 1 0 0 0 0 1 0 2 0 0 1 1 1 0 2 0 1 2 2 0 0
[58413] 2 0 0 0 1 0 0 0 1 0 0 0 1 0 0 1 0 1 0 0 1 0 0 1 0 1 0 0 0 0 1 1 0 0
[58447] 1 1 2 1 1 0 1 0 0 2 1 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 1 2 0 0 1 1 0 0
[58481] 1 0 0 1 1 0 0 0 2 3 0 0 0 0 0 0 1 2 0 1 0 1 0 0 0 1 1 2 0 0 3 1 1 0
[58515] 1 1 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 1 0 0 0 0
[58549] 1 1 1 1 1 1 0 0 0 0 1 0 0 0 0 0 0 1 2 0 0 1 0 0 0 0 0 0 0 1 1 0 1 0
[58583] 1 1 1 2 0 0 2 0 0 0 1 1 0 1 1 0 0 1 0 0 0 2 1 0 0 0 0 0 0 0 0 1 0 0
[58617] 0 1 0 2 2 0 0 0 3 0 1 1 0 3 0 0 0 0 0 1 0 1 1 2 0 0 0 0 0 0 1 0 0 1
[58651] 1 0 3 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 1 1 0 1 0 0 0 0 0 0 1 0 0 0
[58685] 0 0 1 1 0 0 1 0 1 0 0 2 0 1 0 0 1 0 1 0 2 0 0 0 3 1 1 2 0 0 1 0 0 0
[58719] 0 1 1 0 2 2 2 1 0 0 1 1 2 1 0 0 0 0 0 0 1 0 1 0 0 1 0 0 0 1 1 0 0 0
[58753] 0 0 1 1 0 2 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 1 0 0 0 4 2 0 0 0 1 0 0 0
[58787] 0 0 2 1 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 1 0 0 1
[58821] 1 0 0 0 0 0 1 0 0 3 2 0 0 1 0 1 0 1 0 1 0 2 0 0 0 0 1 2 0 0 1 1 0 0
[58855] 1 0 0 0 0 0 0 0 1 0 0 1 1 1 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 1 1 0 0 0
[58889] 0 0 0 2 2 0 0 0 1 0 0 0 0 1 0 0 0 0 1 0 0 1 0 0 0 0 0 0 0 0 0 0 1 0
[58923] 2 1 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 1 2 1 1 1 1 0 0 2 0 1
[58957] 0 1 1 2 2 0 0 1 3 0 0 1 1 1 0 1 2 0 1 0 0 0 1 2 0 0 0 0 0 1 0 1 1 0
[58991] 0 0 1 0 1 0 0 0 0 1 0 0 1 0 0 0 0 2 1 2 1 1 4 0 0 1 0 5 1 0 1 0 5 1
[59025] 0 0 0 0 1 1 0 1 1 1 1 2 1 1 0 1 0 1 0 1 1 0 0 1 0 0 0 0 1 0 4 0 0 0
[59059] 0 0 0 0 0 0 1 1 0 1 0 2 0 0 0 0 0 1 1 0 0 1 0 0 1 0 0 1 2 0 0 0 0 0
[59093] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
[59127] 0 0 0 0 0 0 0 0 0 0 0 0 0 0
~~~
{: .output}

This is a fairly large difference, but it’s important to consider how many windows this includes. 
Indeed, there are only nine windows that have a GC content over 80%:


~~~
sum(d$percent.GC >= 80)
~~~
{: .r}



~~~
[1] 9
~~~
{: .output}

> ## Challenge 6
>
> As another example, consider looking at Pi by windows that fall in the centromere and those that do not.
> Does the centromer have higher nucleotide diversity than other regions in these data?
>
> > ## Solutions to challenge 6
> >
> > Because d$cent is a logical vector, we can subset with it directly (and take its complement 
> > by using the negation operator, !):
> > 
> > 
> > ~~~
> > summary(d$Pi[d$cent])
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> >    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
> >    0.00    7.95   16.08   20.41   27.36  194.40 
> > ~~~
> > {: .output}
> > 
> > 
> > 
> > ~~~
> > summary(d$Pi[!d$cent])
> > ~~~
> > {: .r}
> > 
> > 
> > 
> > ~~~
> >    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
> >   0.000   5.557  10.370  12.290  16.790 265.300 
> > ~~~
> > {: .output}
> > Indeed, the centromere does appear to have higher nucleotide diversity than other regions in this data. 
> >
> {: .solution}
{: .challenge}

In addition to using logical vectors to subset dataframes, it’s also possible to subset 
rows by referring to their integer positions. The function which() takes a vector of 
logical values and returns the positions of all TRUE values. For example:


~~~
d$Pi>3
which(d$Pi > 3)
~~~
{: .r}

Thus, `d[d$Pi > 3, ]` is identical to `d[which(d$Pi > 3), ]`; subsetting operations can be expressed using either method. 
In general, you should omit which() when subsetting dataframes and use logical vectors, 
as it leads to simpler and more readable code. Under other circumstances, which() is 
necessary—for example, if we wanted to select the four first TRUE values in a vector:


~~~
which(d$Pi > 10)[1:4]
~~~
{: .r}



~~~
[1]  2 16 21 23
~~~
{: .output}

which() also has two related functions that return the index of the first minimum or maximum element of a vector: 
which.min() and which.max(). For example:


~~~
d[which.min(d$total.Bases),]
~~~
{: .r}



~~~
         start      end total.SNPs total.Bases depth unique.SNPs dhSNPs
25689 25785001 25786000          0         110     1           0      0
      reference.Bases Theta Pi Heterozygosity percent.GC Recombination
25689             110     0  0              0    38.8388      4.63e-05
      Divergence Constraint SNPs  cent diversity
25689 0.04946043          0    0 FALSE         0
~~~
{: .output}



~~~
d[which.max(d$depth),]
~~~
{: .r}



~~~
       start     end total.SNPs total.Bases depth unique.SNPs dhSNPs
8718 8773001 8774000         58       21914 21.91           7      4
     reference.Bases  Theta     Pi Heterozygosity percent.GC Recombination
8718            1000 17.676 14.199         26.581    39.3393   0.001990459
     Divergence Constraint SNPs  cent diversity
8718 0.01601602          0    1 FALSE 0.0014199
~~~
{: .output}

Sometimes subsetting expressions inside brackets can be quite redundant (because each column must be 
specified like `d$Pi`, `d$depth`, etc). A useful convenience function (intended primarily for interactive use) 
is the R function `subset()`. `subset()` takes two arguments: the dataframe to operate on, and then conditions 
to include a row. With `subset()`, `d[d$Pi > 16 & d$percent.GC > 80, ]` can be expressed as:


~~~
subset(d, Pi > 16 & percent.GC > 80)
~~~
{: .r}



~~~
         start      end total.SNPs total.Bases depth unique.SNPs dhSNPs
58550 63097001 63098000          5         947  2.39           2      1
58641 63188001 63189000          2        1623  3.21           2      0
58642 63189001 63190000          5        1395  1.89           3      2
      reference.Bases  Theta     Pi Heterozygosity percent.GC
58550             397 37.544 41.172         52.784    82.0821
58641             506 16.436 16.436         12.327    82.3824
58642             738 35.052 41.099         35.842    80.5806
      Recombination Divergence Constraint SNPs  cent diversity
58550   0.000781326 0.03826531        226    1 FALSE 0.0041172
58641   0.000347382 0.01678657        148    0 FALSE 0.0016436
58642   0.000347382 0.01793722          0    0 FALSE 0.0041099
~~~
{: .output}

Optionally, a third argument can be supplied to specify which columns (and in what order) to include:


~~~
subset(d, Pi > 16 & percent.GC > 80, c(start, end, Pi, percent.GC, depth))
~~~
{: .r}



~~~
         start      end     Pi percent.GC depth
58550 63097001 63098000 41.172    82.0821  2.39
58641 63188001 63189000 16.436    82.3824  3.21
58642 63189001 63190000 41.099    80.5806  1.89
~~~
{: .output}

Note that we (somewhat magically) don’t need to quote column names. This is because `subset()` follows 
special evaluation rules, and for this reason, `subset()` is best used only for interactive work.

## Merging and Combining Data: Matching Vectors and Merging Dataframes

Bioinformatics analysis often involves connecting datasets: sequencing data, genomic features 
(e.g., gene annotation), functional genomics data, population genetic data, and so on. As data piles 
up in repositories, the ability to connect different datasets together to tell a cohesive story will 
become an increasingly more important analysis skill. In this section, we’ll look at some canonical 
ways to combine datasets together in R.

Before we do, we'll download two datasets:


~~~
download.file("https://raw.githubusercontent.com/vsbuffalo/bds-files/master/chapter-08-r/chrX_rmsk.txt.gz", destfile = "data/chrX_rmsk.txt.gz")
reps <- read.delim("data/chrX_rmsk.txt.gz")
~~~
{: .r}

Let's look at these files:


~~~
head(reps, 3)
~~~
{: .r}



~~~
  bin swScore milliDiv milliDel milliIns genoName genoStart genoEnd
1 585     342        0        0        0     chrX         0      38
2 585     392      109        0        0     chrX        41     105
3 585     302      240       31       20     chrX       105     203
    genoLeft strand   repName      repClass     repFamily repStart repEnd
1 -154824226      + (CCCTAA)n Simple_repeat Simple_repeat        3     40
2 -154824159      +    LTR12C           LTR          ERV1     1090   1153
3 -154824061      +     LTR30           LTR          ERV1      544    642
  repLeft id
1       0  1
2    -425  2
3     -80  3
~~~
{: .output}



~~~
head(mtfs, 3)
~~~
{: .r}



~~~
Error in head(mtfs, 3): object 'mtfs' not found
~~~
{: .error}

The first file shows the repeats on human chromosome X found by Repeat Masker.  Here repClass is an example of a factor column — try `class(reps$repClass)` and `levels(reps$repClass)`. Suppose we wanted to select out rows for some common repeat classes: DNA, LTR, LINE, SINE, and Simple_repeat. We can construct a statement to select these values using logical operators: 


~~~
reps$repClass == "SINE" | reps$repClass == "LINE" | reps$repClass == "LTR" | reps$repClass == "DNA" | reps$repClass == "Simple_repeat"
~~~
{: .r}

However, this approach is tedious and error prone.

Instead, we can use the  R’s %in% operator, which returns a logical vector indicating which of the values of x are in y. E.g.,:


~~~
c(3,4,-1)%in%c(1,3,4,8)
~~~
{: .r}



~~~
[1]  TRUE  TRUE FALSE
~~~
{: .output}

In our case:


~~~
common_repclass <- c("SINE", "LINE", "LTR", "DNA", "Simple_repeat") 
reps[reps$repClass %in% common_repclass, ]
~~~
{: .r}

Note, that we can also create common_repclass vector programmatically:


~~~
top5_repclass <- names(sort(table(reps$repClass), decreasing=TRUE)[1:5])
top5_repclass
~~~
{: .r}



~~~
[1] "LINE"          "SINE"          "LTR"           "Simple_repeat"
[5] "DNA"          
~~~
{: .output}

The `%in%` operator is a simplified version of another function, `match()`. `x %in% y` returns TRUE/FALSE for 
each value in x depending on whether it’s in y. In contrast, `match(x, y)` returns the first occurrence of 
each of x’s values in y. Back to our previous example:


~~~
match(c(3,4,-1),c(1,3,4,8))
~~~
{: .r}



~~~
[1]  2  3 NA
~~~
{: .output}

Note, the vector returned will always have the same length as the first argument and contains positions in the second argument.

Because match() returns where it finds a particular value, match()’s output can be used to join two 
data frames together by a shared column. Here we’ll merge two datasets to explore recombination 
rates around a degenerate sequence motif that occurs in repeats.  The first dataset contains 
estimates of the recombination rate for all windows within 40kb of each motif (for two motif variants).
The second dataset (motif_repeats.txt) contains which repeat each motif occurs in. Our goal is to merge 
these two datasets so that we can look at the local effect of recombination of each motif on specific repeat backgrounds.

Let’s start by loading in both files and peeking at them with head():


~~~
mtfs <- read.delim("https://raw.githubusercontent.com/vsbuffalo/bds-files/master/chapter-08-r/motif_recombrates.txt", header=TRUE)
rpts <- read.delim("https://raw.githubusercontent.com/vsbuffalo/bds-files/master/chapter-08-r/motif_repeats.txt", header=TRUE)
head(mtfs, 3)
~~~
{: .r}



~~~
   chr motif_start motif_end    dist recomb_start recomb_end  recom
1 chrX    35471312  35471325 39323.0     35430651   35433340 0.0015
2 chrX    35471312  35471325 36977.0     35433339   35435344 0.0015
3 chrX    35471312  35471325 34797.5     35435343   35437699 0.0015
          motif           pos
1 CCTCCCTGACCAC chrX-35471312
2 CCTCCCTGACCAC chrX-35471312
3 CCTCCCTGACCAC chrX-35471312
~~~
{: .output}



~~~
head(rpts, 3)
~~~
{: .r}



~~~
   chr     start       end name motif_start
1 chrX  63005829  63006173   L2    63005830
2 chrX  67746983  67747478   L2    67747232
3 chrX 118646988 118647529   L2   118647199
~~~
{: .output}

> ## Discussion time!
> Examine these two files and discuss the following questions with your neighbor
> * How long is/are the motif(s) we are interested in?
> * Why do we see the same motif on multiple lines in the dataset1?
> * Why do we have two "start" columns in the second dataset?
> * Can we have two columns with the same start/end in the dataset2?
> * What column(s) can we use to combine these two datasets?
> * What is our goal, again?
{: .discussion}

OK, so what we are trying to do is to add the colun "name" from the second dataset to the first one, 
to see whether and in which repeat each motif is contained.

Because we are dealing with multiple chromosomes, we start by merging two column, `chr` and `motif_start`:


~~~
mtfs$pos <- paste(mtfs$chr, mtfs$motif_start, sep="-")
rpts$pos <- paste(rpts$chr, rpts$motif_start, sep="-")
~~~
{: .r}

Now, this pos column functions as a common key between the two datasets.
Let's validate that our keys overlap in the way we think they do before merging:


~~~
table(mtfs$pos %in% rpts$pos)
~~~
{: .r}



~~~

FALSE  TRUE 
10832  9218 
~~~
{: .output}

Now, we use match() to find where each of the `mtfs$pos` keys occur in the `rpts$pos`. 
We’ll create this indexing vector first before doing the merge:


~~~
i <- match(mtfs$pos, rpts$pos)
~~~
{: .r}

All motif positions without a corresponding entry in rpts are NA; our number of NAs
is exactly the number of `mts$pos` elements not in `rpts$pos`:


~~~
table(is.na(i))
~~~
{: .r}



~~~

FALSE  TRUE 
 9218 10832 
~~~
{: .output}

Finally, using this indexing vector we can select out the appropriate elements of rpts $name and merge these into mtfs:


~~~
mtfs$repeat_name <- rpts$name[i]
~~~
{: .r}

Often in practice you might skip assigning match()’s results to i and use this directly:


~~~
mtfs$repeat_name <- rpts$name[match(mtfs$pos, rpts$pos)]
~~~
{: .r}

Let's check our result:


~~~
head(mtfs[!is.na(mtfs$repeat_name), ], 3)
~~~
{: .r}



~~~
     chr motif_start motif_end    dist recomb_start recomb_end  recom
99  chrX    63005830  63005843 37772.0     62965644   62970485 1.4664
100 chrX    63005830  63005843 34673.0     62970484   62971843 0.0448
101 chrX    63005830  63005843 30084.5     62971842   62979662 0.0448
            motif           pos repeat_name
99  CCTCCCTGACCAC chrX-63005830          L2
100 CCTCCCTGACCAC chrX-63005830          L2
101 CCTCCCTGACCAC chrX-63005830          L2
~~~
{: .output}

Great, we’ve combined the `rpts$name` vector directly into our mtfs dataframe. Not all motifs have 
entries in rpts, so some values in `mfs$repeat_name` are NA. We could easily remove these NAs with:


~~~
mtfs_inner <- mtfs[!is.na(mtfs $repeat_name), ]
nrow(mtfs_inner)
~~~
{: .r}



~~~
[1] 9218
~~~
{: .output}

In this case, only motifs in mtfs contained in a repeat in rpts are kept (technically, this type of join is called an inner join). Inner joins are the most common way to merge data. 

We’ve learned match() first because it’s a general, extensible way to merge data in R. 
However, R does have a more user-friendly merging function: merge(). 
Merge can directly merge two datasets:


~~~
recm <- merge(mtfs, rpts, by.x="pos", by.y="pos")
head(recm, 2)
~~~
{: .r}



~~~
             pos chr.x motif_start.x motif_end    dist recomb_start
1 chr1-101890123  chr1     101890123 101890136 34154.0    101855215
2 chr1-101890123  chr1     101890123 101890136 35717.5    101853608
  recomb_end  recom         motif repeat_name chr.y     start       end
1  101856736 0.0700 CCTCCCTAGCCAC       THE1B  chr1 101890032 101890381
2  101855216 0.0722 CCTCCCTAGCCAC       THE1B  chr1 101890032 101890381
   name motif_start.y
1 THE1B     101890123
2 THE1B     101890123
~~~
{: .output}



~~~
nrow(recm)
~~~
{: .r}



~~~
[1] 9218
~~~
{: .output}

`merge()` takes two dataframes, x and y, and joins them by the columns supplied by by.x and by.y. 
If they aren’t supplied, merge() will try to infer what these columns are, but it’s much safer 
to supply them explicitly. If you want to keep all rows in both datasets, you can specify `all=TRUE`. 
See `help(merge)` for more details on how merge() works. 

> ## Vince Buffalo's guidelines for combining data:
>
> 1. Carefully consider the structure of both datasets;
> 2. Validate that your keys overlap in the way you think they do before merging;
> 3. Validate, validate, validate!
>
{: .callout}
