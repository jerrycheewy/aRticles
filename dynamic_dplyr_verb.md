Dynamic dplyr
================
Jerry Chee
8 May 2019

Why did I look into this?
-------------------------

Have you been in a situation where you've written a custom function to summarise data, but find yourself creating several versions of this function depending on different levels of `group_by`? For instance in my work, I often find myself needing to analyse investments in groups or sub-groups such as 1)sectors OR geography, 2)sector AND geography.

What if it gets even dicier with multi-level groups such as sector + subsector + geography + holding period + portfolio weight and you cannot pre-determine before-hand how many combinations of these groups you might need in your work?

Is there such as thing as 'dynamic' `dplyr`?
--------------------------------------------

Truthfully, this is better known as *standard evaluation* and it has actually been deprecated. This functionality only comes in useful when you're incorporating dplyr verbs within your own custom functions and wish to pass an expandable list of arguments into the `dplyr` verbs.

Let's see an example with the mpg dataset from `ggplot2` package

``` r
#Load packages
library(ggplot2)
library(tidyverse)
```

Refresh ourselves of the `mpg` data...

``` r
head(mpg)
```

    ## # A tibble: 6 x 11
    ##   manufacturer model displ  year   cyl trans drv     cty   hwy fl    class
    ##   <chr>        <chr> <dbl> <int> <int> <chr> <chr> <int> <int> <chr> <chr>
    ## 1 audi         a4      1.8  1999     4 auto~ f        18    29 p     comp~
    ## 2 audi         a4      1.8  1999     4 manu~ f        21    29 p     comp~
    ## 3 audi         a4      2    2008     4 manu~ f        20    31 p     comp~
    ## 4 audi         a4      2    2008     4 auto~ f        21    30 p     comp~
    ## 5 audi         a4      2.8  1999     6 auto~ f        16    26 p     comp~
    ## 6 audi         a4      2.8  1999     6 manu~ f        18    26 p     comp~

##### Aggregate mean fuel consumption on highway roads for each car manufacturer

``` r
#Aggregate mean fuel consumption on highway for each car manufacturer
mpg %>% group_by(manufacturer) %>% summarise(hwy_mean = mean(hwy))
```

    ## # A tibble: 15 x 2
    ##    manufacturer hwy_mean
    ##    <chr>           <dbl>
    ##  1 audi             26.4
    ##  2 chevrolet        21.9
    ##  3 dodge            17.9
    ##  4 ford             19.4
    ##  5 honda            32.6
    ##  6 hyundai          26.9
    ##  7 jeep             17.6
    ##  8 land rover       16.5
    ##  9 lincoln          17  
    ## 10 mercury          18  
    ## 11 nissan           24.6
    ## 12 pontiac          26.4
    ## 13 subaru           25.6
    ## 14 toyota           24.9
    ## 15 volkswagen       29.2

But what if we'd like to do better and write a function that allows us to summarise `mpg` on a number of levels? Below is an example of passing additional arguments *(length undetermined beforehand)* with the ellipsis (this thing -&gt; ...)

##### Create a custom summarising function

``` r
#My custom function
my_hwymean <- function(df, ...) {
  #... here allows us to pass any number of arguments to our function
  mydf <- df %>% group_by_(...) %>% summarise(hwy_mean = mean(hwy))
  return(mydf)
}
```

##### Aggregate by any choice(s) of variables

``` r
#By manufacturer AND model
my_hwymean(mpg, 'manufacturer', 'model') %>% head()
```

    ## # A tibble: 6 x 3
    ## # Groups:   manufacturer [2]
    ##   manufacturer model              hwy_mean
    ##   <chr>        <chr>                 <dbl>
    ## 1 audi         a4                     28.3
    ## 2 audi         a4 quattro             25.8
    ## 3 audi         a6 quattro             24  
    ## 4 chevrolet    c1500 suburban 2wd     17.8
    ## 5 chevrolet    corvette               24.8
    ## 6 chevrolet    k1500 tahoe 4wd        16.2

``` r
#By year and class
my_hwymean(mpg, 'year', 'class') %>% head()
```

    ## # A tibble: 6 x 3
    ## # Groups:   year [1]
    ##    year class      hwy_mean
    ##   <int> <chr>         <dbl>
    ## 1  1999 2seater        24.5
    ## 2  1999 compact        27.9
    ## 3  1999 midsize        26.5
    ## 4  1999 minivan        22.5
    ## 5  1999 pickup         16.8
    ## 6  1999 subcompact     29

Additional tip
--------------

Sometimes you want to be using standardised references in all functional calls, regardless of what type of analysis is being done. For this example, imagine if all analysis you're doing requires 'manufacturer' to included minimally.

You can control for this mechanic by 'hard-coding' the required group into your function. As we're now using the NSE `group_by`, passing variable names into it requires that you enquote it.

``` r
my_mfgmean <- function(df, ...){
  mydf <- df %>% group_by_("manufacturer", ...) %>% summarise(hwy_mean = mean(hwy))
  return(mydf)
}
```

Now additional groupings provided through the ellipsis input will be **in addition** to `manufacturer`.

``` r
my_mfgmean(mpg, 'year') %>% head()
```

    ## # A tibble: 6 x 3
    ## # Groups:   manufacturer [3]
    ##   manufacturer  year hwy_mean
    ##   <chr>        <int>    <dbl>
    ## 1 audi          1999     26.1
    ## 2 audi          2008     26.8
    ## 3 chevrolet     1999     21.6
    ## 4 chevrolet     2008     22.1
    ## 5 dodge         1999     18.4
    ## 6 dodge         2008     17.6

``` r
my_mfgmean(mpg, 'class', 'trans') %>% head()
```

    ## # A tibble: 6 x 4
    ## # Groups:   manufacturer, class [2]
    ##   manufacturer class   trans      hwy_mean
    ##   <chr>        <chr>   <chr>         <dbl>
    ## 1 audi         compact auto(av)       28.5
    ## 2 audi         compact auto(l5)       26.2
    ## 3 audi         compact auto(s6)       26  
    ## 4 audi         compact manual(m5)     26.5
    ## 5 audi         compact manual(m6)     28  
    ## 6 audi         midsize auto(l5)       24
