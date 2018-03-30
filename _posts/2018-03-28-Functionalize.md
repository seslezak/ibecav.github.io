---
layout: post
title: Using functions to be more efficient – March 28, 2018
tags: R dplyr excel lapply for-loop openxlsx
---

In yesterday’s post I focused on the task of using R to “drive” MS
Excel. I deliberately ended the post with a fully functioning (pun
intended) but very ugly set of code. Why “ugly”? Well, because the last
set of code wound up repeating 4 lines of code 12 times\!

``` r
### Manual and painful way
## Create a new workbook
wb <- createWorkbook()
# education by each of the other 4 variables
NameofSheet <- "CoverageByEducation"
TheData <- table(OfInterest$EDUCATION,OfInterest$NOTCOV)
addWorksheet(wb = wb, sheetName = NameofSheet)
writeData(wb = wb, sheet = NameofSheet, x = TheData, borders = "n")
NameofSheet <- "MedbillByEducation"
TheData <- table(OfInterest$EDUCATION,OfInterest$MEDBILL)
addWorksheet(wb = wb, sheetName = NameofSheet)
writeData(wb = wb, sheet = NameofSheet, x = TheData, borders = "n")
### many repeating lines removed ###
NameofSheet <- "PNMED12MByAge"
TheData <- table(OfInterest$AGE,OfInterest$PNMED12M)
addWorksheet(wb = wb, sheetName = NameofSheet)
writeData(wb = wb, sheet = NameofSheet, x = TheData, borders = "n")
#
saveWorkbook(wb, "BetterExcelExample.xlsx", overwrite = TRUE) ## save to working directory
```

Now, to be honest, in a nice modern IDE environment like `RStudio`,
cutting, pasting and making small edits 12 times is not all that
difficult. I did do exactly that my first pass through. But as I looked
at it I also knew I could do much better if I used automation. The code
would be easier to read or modify in the future and I’d be much less
likely to make a mistake.

## Background and catchup

My colleague wanted to be able to do some simple analysis around health
care using the Centers for Disease Control and Prevention
(<https://www.cdc.gov>), National Center for Health Statistics
(<https://www.cdc.gov/nchs/index.htm>), National Health Interview Survey
(<https://www.cdc.gov/nchs/nhis/nhis_2016_data_release.htm>). They
wanted a series of cross tabulated sets of summary data for variable
pairings (for example whether or not the respondent had a formal health
care provider by region of the country). They wanted one Excel
“workbook” with 12 worksheets each one of which was the summary of
counts for a pair of variables. From there they could use Excel’s native
plotting tools to make the graphs they needed.

You can review everything that happened yesterday (which I recommend) or
you can pick up here. To join us in progress make sure you load the
right libraries and grab the dataset we wound up on which is called
`OfInterest`.

``` r
knitr::opts_chunk$set(echo = TRUE, warning = FALSE)
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(ggplot2)
theme_set(theme_bw()) # set theme to my personal preference
# install.packages("openxlsx")
require(openxlsx)
```

    ## Loading required package: openxlsx

``` r
OfInterest <- read.csv("ofinterest.csv")
### OfInterest <- read.csv("https://raw.githubusercontent.com/ibecav/ibecav.github.io/master/Rmdfiles/ofinterest.csv") available through Github about 9Mb
str(OfInterest)
```

    ## 'data.frame':    103789 obs. of  9 variables:
    ##  $ AGE      : Factor w/ 3 levels "19 to 60","Less than 18",..: 1 1 1 2 2 1 1 1 3 1 ...
    ##  $ REGION   : Factor w/ 4 levels "Midwest","Northeast",..: 3 4 4 4 4 4 3 3 4 3 ...
    ##  $ SEX      : Factor w/ 2 levels "Female","Male": 2 1 2 2 1 1 1 2 2 2 ...
    ##  $ EDUCATION: Factor w/ 3 levels "Bachelor's degree or higher",..: 3 2 1 2 NA 1 2 2 1 3 ...
    ##  $ EARNINGS : Factor w/ 3 levels "$01-$34,999",..: 1 2 2 NA NA 3 1 NA 2 3 ...
    ##  $ PDMED12M : Factor w/ 2 levels "No","Yes": 1 2 1 1 1 1 1 1 1 1 ...
    ##  $ PNMED12M : Factor w/ 2 levels "No","Yes": 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ NOTCOV   : Factor w/ 2 levels "Covered","Not covered": 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ MEDBILL  : Factor w/ 2 levels "No","Yes": 1 2 2 2 2 1 1 1 1 1 ...

## Breaking the task down

So as my tagline indicates I don’t consider myself a “programmer”. I
love analyzing data and I love `R` but I approach programming slowly and
cautiously. What I’m about to explain as my method will likely seem
quaint and even antiquated to some but has the advantage of being very
methodical and very practical. There are lots of places on the web to
read about this stuff, I’m simply making the case I hope mine is slow
enough and methodical enough for a beginner.

What do we have there in those 12 iterations? Well what we have are 3
variables `EDUCATION`, `EARNINGS`, and `AGE` that we want to **“cross”**
with 4 other variables `NOTCOV`, `MEDBILL`, `PDMED12M`, and `PNMED12M`
to give us the total of 12. In my discipline we would call the first
three the independent variables and the second four the dependent
variables.

So as a first step in automating our work lets just make two lists that
acknowledge that fact. We’ll even name the list elements so we can make
use of those names in future steps. That means we can use the short hand
`depvars$Coverage` instead of `depvars[[1]]` which I find difficult to
keep track of. Notice that the list contains the actual data and is not
just a pointer at the
dataframe.

``` r
depvars <- list(Coverage = OfInterest$NOTCOV, ProbPay = OfInterest$MEDBILL, CareDelay = OfInterest$PDMED12M, NeedNotGet = OfInterest$PNMED12M)
indvars <- list(Education = OfInterest$EDUCATION, Earnings = OfInterest$EARNINGS, Age = OfInterest$AGE)
# these two are identical use head because the list is more than 100,000 entries long
head(depvars$Coverage)
```

    ## [1] Covered Covered Covered Covered Covered Covered
    ## Levels: Covered Not covered

``` r
head(depvars[[1]])
```

    ## [1] Covered Covered Covered Covered Covered Covered
    ## Levels: Covered Not covered

Okay, so far so good\! Now what? Well `R` has a wonderful function
called `lapply` which as the documentation `?lapply` and numerous
websites will tell you applies a function to a list. It sequentially
walks its way through a list and applies the function you tell it to
use.

Let’s take a small step. We want tables. 12 of them. Each table is of
the form `table(indvars,depvars)` like “education” by “coverage”. Let’s
try `lapply` for just part of that process. The command becomes
`lapply(depvars, function (x) table(OfInterest$EDUCATION,x))` which you
can read as “Take the **list** of dependent variables **depvars**.
**Apply** the function called **table** and wherever you see an **x**
substitute the current value of **depvars**”. So the very first thing it
would do is `table(OfInterest$EDUCATION,OfInterest$NOTCOV))` and it
would do it for all four variables in the list.

``` r
lapply(depvars, function (x) table(OfInterest$EDUCATION,x))
```

    ## $Coverage
    ##                                   x
    ##                                    Covered Not covered
    ##   Bachelor's degree or higher        20802         895
    ##   High School Grad or less           43404        6846
    ##   Some college or Associate degree   20667        2313
    ## 
    ## $ProbPay
    ##                                   x
    ##                                       No   Yes
    ##   Bachelor's degree or higher      19994  1789
    ##   High School Grad or less         41375  9217
    ##   Some college or Associate degree 19231  3899
    ## 
    ## $CareDelay
    ##                                   x
    ##                                       No   Yes
    ##   Bachelor's degree or higher      20583  1219
    ##   High School Grad or less         47576  3155
    ##   Some college or Associate degree 21001  2185
    ## 
    ## $NeedNotGet
    ##                                   x
    ##                                       No   Yes
    ##   Bachelor's degree or higher      21108   694
    ##   High School Grad or less         48191  2532
    ##   Some college or Associate degree 21545  1645

Perfect\! Just what we were looking for. We get back a list of 4 tables.
Progress\! No surprise it works the other way around as well. We can
hold the second part of the table command constant and just vary the
independent variable via `indvars`\!

``` r
lapply(indvars, function (y) table(y,OfInterest$NOTCOV))
```

    ## $Education
    ##                                   
    ## y                                  Covered Not covered
    ##   Bachelor's degree or higher        20802         895
    ##   High School Grad or less           43404        6846
    ##   Some college or Associate degree   20667        2313
    ## 
    ## $Earnings
    ##                   
    ## y                  Covered Not covered
    ##   $01-$34,999        16148        4121
    ##   $35,000-$74,999    12116         783
    ##   $75,000 and over    6253         158
    ## 
    ## $Age
    ##               
    ## y              Covered Not covered
    ##   19 to 60       47258        8623
    ##   Less than 18   24193        1312
    ##   More than 60   20730         571

What we need, of course, is both of those things. A **“nested”** set of
calls to `lapply` to walk through both lists and give us 12 tables not 3
or 4. So the next command is a bit ugly to read but hopefully if you
have been following along it will make perfectly good sense. We’re going
to call `lapply` and the function we will tell it to run is `lapply`\!
The second `lapply` will in turn call `table` and all we have to do is
to keep our x’s and y’s correct\!

``` r
lapply(depvars, function (x) lapply(indvars, function (y) table(y,x)))
```

    ## $Coverage
    ## $Coverage$Education
    ##                                   x
    ## y                                  Covered Not covered
    ##   Bachelor's degree or higher        20802         895
    ##   High School Grad or less           43404        6846
    ##   Some college or Associate degree   20667        2313
    ## 
    ## $Coverage$Earnings
    ##                   x
    ## y                  Covered Not covered
    ##   $01-$34,999        16148        4121
    ##   $35,000-$74,999    12116         783
    ##   $75,000 and over    6253         158
    ## 
    ## $Coverage$Age
    ##               x
    ## y              Covered Not covered
    ##   19 to 60       47258        8623
    ##   Less than 18   24193        1312
    ##   More than 60   20730         571
    ## 
    ## 
    ## $ProbPay
    ## $ProbPay$Education
    ##                                   x
    ## y                                     No   Yes
    ##   Bachelor's degree or higher      19994  1789
    ##   High School Grad or less         41375  9217
    ##   Some college or Associate degree 19231  3899
    ## 
    ## $ProbPay$Earnings
    ##                   x
    ## y                     No   Yes
    ##   $01-$34,999      16229  4139
    ##   $35,000-$74,999  11321  1596
    ##   $75,000 and over  6069   341
    ## 
    ## $ProbPay$Age
    ##               x
    ## y                 No   Yes
    ##   19 to 60     46924  9504
    ##   Less than 18 20841  4747
    ##   More than 60 19331  2031
    ## 
    ## 
    ## $CareDelay
    ## $CareDelay$Education
    ##                                   x
    ## y                                     No   Yes
    ##   Bachelor's degree or higher      20583  1219
    ##   High School Grad or less         47576  3155
    ##   Some college or Associate degree 21001  2185
    ## 
    ## $CareDelay$Earnings
    ##                   x
    ## y                     No   Yes
    ##   $01-$34,999      17937  2447
    ##   $35,000-$74,999  12049   877
    ##   $75,000 and over  6239   174
    ## 
    ## $CareDelay$Age
    ##               x
    ## y                 No   Yes
    ##   19 to 60     51621  5020
    ##   Less than 18 25058   612
    ##   More than 60 20307  1112
    ## 
    ## 
    ## $NeedNotGet
    ## $NeedNotGet$Education
    ##                                   x
    ## y                                     No   Yes
    ##   Bachelor's degree or higher      21108   694
    ##   High School Grad or less         48191  2532
    ##   Some college or Associate degree 21545  1645
    ## 
    ## $NeedNotGet$Earnings
    ##                   x
    ## y                     No   Yes
    ##   $01-$34,999      18481  1907
    ##   $35,000-$74,999  12403   522
    ##   $75,000 and over  6327    86
    ## 
    ## $NeedNotGet$Age
    ##               x
    ## y                 No   Yes
    ##   19 to 60     52841  3794
    ##   Less than 18 25262   402
    ##   More than 60 20638   782

Please note that it took me quite some time to get that nested lapply
correct\! A lot of searching on Stack Overflow, a lot of trial and
error, but I won’t forget it now\! If you want to test yourself try on
your own to make the change necessary to invert the output to have the
columns and rows the other way around.

Now that we have demonstrated we can do it, lets put our 12 tables
someplace safe. Let’s call it `TablesList` and to ensure we know how to
get these tables back out again let’s pull just one of them from our
list of 12, this is where using names not numbers
helps.

``` r
TablesList <- lapply(depvars, function (x) lapply(indvars, function (y) table(y,x)))
TablesList$Coverage$Education
```

    ##                                   x
    ## y                                  Covered Not covered
    ##   Bachelor's degree or higher        20802         895
    ##   High School Grad or less           43404        6846
    ##   Some college or Associate degree   20667        2313

``` r
TablesList$Coverage
```

    ## $Education
    ##                                   x
    ## y                                  Covered Not covered
    ##   Bachelor's degree or higher        20802         895
    ##   High School Grad or less           43404        6846
    ##   Some college or Associate degree   20667        2313
    ## 
    ## $Earnings
    ##                   x
    ## y                  Covered Not covered
    ##   $01-$34,999        16148        4121
    ##   $35,000-$74,999    12116         783
    ##   $75,000 and over    6253         158
    ## 
    ## $Age
    ##               x
    ## y              Covered Not covered
    ##   19 to 60       47258        8623
    ##   Less than 18   24193        1312
    ##   More than 60   20730         571

Okay we made a list of tables now let’s work on putting those tables
where we want them in a series of sheets in a workbook.

## For a little more fun

Pardon the pun (for those who got it) but yes we’re now going to use
`for loops` to pull the tables out of the list and put them somewhere in
an organized fashion. So the name of our list is `TablesList` and inside
that list are sublists with names like `TablesList$Coverage` and the
individual tables have names like `TablesList$Coverage$Education`.

So why don’t we walk down `TablesList$Coverage` and extract the 3 tables
in there one by one? Many ways to do it but I’ll use a `for loop` with
`for (i in seq_along(TablesList$Coverage))
{print(TablesList$Coverage[[i]])}` which says sequence along the list
`TablesList$Coverage` using `i` as a placeholder for where we are in the
list. Then for each item in the list `i` `print` the
table.

``` r
for (i in seq_along(TablesList$Coverage)) {print(TablesList$Coverage[[i]])}
```

    ##                                   x
    ## y                                  Covered Not covered
    ##   Bachelor's degree or higher        20802         895
    ##   High School Grad or less           43404        6846
    ##   Some college or Associate degree   20667        2313
    ##                   x
    ## y                  Covered Not covered
    ##   $01-$34,999        16148        4121
    ##   $35,000-$74,999    12116         783
    ##   $75,000 and over    6253         158
    ##               x
    ## y              Covered Not covered
    ##   19 to 60       47258        8623
    ##   Less than 18   24193        1312
    ##   More than 60   20730         571

Good. Once again though we want all 12 not just three so we need to nest
again. So let’s walk down just `EDUCATION` first…

``` r
for (j in seq_along(TablesList)) {print(TablesList[[j]][[1]])}
```

    ##                                   x
    ## y                                  Covered Not covered
    ##   Bachelor's degree or higher        20802         895
    ##   High School Grad or less           43404        6846
    ##   Some college or Associate degree   20667        2313
    ##                                   x
    ## y                                     No   Yes
    ##   Bachelor's degree or higher      19994  1789
    ##   High School Grad or less         41375  9217
    ##   Some college or Associate degree 19231  3899
    ##                                   x
    ## y                                     No   Yes
    ##   Bachelor's degree or higher      20583  1219
    ##   High School Grad or less         47576  3155
    ##   Some college or Associate degree 21001  2185
    ##                                   x
    ## y                                     No   Yes
    ##   Bachelor's degree or higher      21108   694
    ##   High School Grad or less         48191  2532
    ##   Some college or Associate degree 21545  1645

## Driving Excel through automation

Okay now we’re ready to make magic. Scroll back to the top of this post
or review the last post and you’ll see the logic is:

1.  Create a new empty workbook object `wb <- createWorkbook()` **once**
2.  Invent a name for the **tab** or worksheet inside the workbook
    `NameofSheet` **12 times**
3.  Make a `table` for a pair of variables like `TheData` **12 times**
4.  Add a worksheet (tab) into the workbook `addWorksheet` **12 times**
5.  Write the table we made onto the worksheet with `writeData` **12
    times**
6.  Save the workbook with the 12 sheets in it **once**

So steps 1 & 6 occur once and steps 2-5 need to occur in our loop
structure

1.  `wb <- createWorkbook()`
2.  `for loop`
    1)  get table `TheData`
    2)  create `NameofSheet`
    3)  make an empty worksheet named `NameofSheet` with `addWorksheet`
    4)  `writeData` called `TheData` into `NameofSheet`
3.  `saveWorkbook` or save it

<!-- end list -->

``` r
## Create a new empty workbook
wb <- createWorkbook()
## nested for loop
for (j in seq_along(TablesList)) { #top list with depvars
  for (i in seq_along(TablesList[[j]])) { #for each depvar walk the indvars
    TheData <- TablesList[[j]][[i]]
    NameofSheet <- paste0(names(TablesList[j]), "By", names(TablesList[[j]][i]))
    addWorksheet(wb = wb, sheetName = NameofSheet)
    writeData(wb = wb, sheet = NameofSheet, x = TheData, borders = "n")
    }
  }
## Save our new workbook
saveWorkbook(wb, "newversion.xlsx", overwrite = TRUE) ## save to working directory
```

Breaking it down:

These two lines create the nested for loops to walk the 12 tables. `j`
is the outer loop of dependent variables and `i` is the inner loop of
independent variables.

``` r
for (j in seq_along(TablesList)) { #top list with depvars
  for (i in seq_along(TablesList[[j]])) { #for each depvar walk the indvars
```

`NameofSheet` is nice because since our list items are “named” we can
put those names together with the word “By” to make the name of the
worksheet sensible to a human.

``` r
    TheData <- TablesList[[j]][[i]]
    NameofSheet <- paste0(names(TablesList[j]), "By", names(TablesList[[j]][i]))
    addWorksheet(wb = wb, sheetName = NameofSheet)
    writeData(wb = wb, sheet = NameofSheet, x = TheData, borders = "n")
```

## All done (not yet\!)

Hopefully this post helps you understand how to put automation in the
guise of `lapply` and `for` for you. In my next post I’ll build on this
scaffolding to discuss how to make these very same graphs in `ggplot2`
(which IMHO runs circles around Excel for scientific plotting), as well
as making this all more efficient through the use of our own functions
to take care of some of the more repetitive chores.

I hope you’ve found this useful. I am always open to comments,
corrections and suggestions.

Chuck (ibecav at gmail dot
com)

### License

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />This
work is licensed under a
<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative
Commons Attribution-ShareAlike 4.0 International License</a>.
