Dynamic dplyr
================
JerryC
8 May 2019

What is 'dynamic' dplyr and why do I need it?
---------------------------------------------

Truthfully, this is better known as *standard evaluation* and have actually been deprecated. This functionality comes in useful when you're incorporating dplyr verbs within your own custom functions and wish to pass an expandable list of arguments into the dplyr verbs.

Let's see an example with the mpg dataset from ggplot2 package

``` r
#Load packages
library(ggplot2)
library(tidyverse)
```

    ## -- Attaching packages ----------------------------------------------------------------------------------------------------------------------------------- tidyverse 1.2.1 --

    ## v tibble  1.4.2     v purrr   0.2.4
    ## v tidyr   0.8.0     v dplyr   0.7.4
    ## v readr   1.1.1     v stringr 1.2.0
    ## v tibble  1.4.2     v forcats 0.2.0

    ## -- Conflicts -------------------------------------------------------------------------------------------------------------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

Peek at mpg dataset

``` r
head(mpg)
```

    ## # A tibble: 6 x 11
    ##   manufacturer model displ  year   cyl trans drv     cty   hwy fl    class
    ##   <chr>        <chr> <dbl> <int> <int> <chr> <chr> <int> <int> <chr> <chr>
    ## 1 audi         a4     1.80  1999     4 auto~ f        18    29 p     comp~
    ## 2 audi         a4     1.80  1999     4 manu~ f        21    29 p     comp~
    ## 3 audi         a4     2.00  2008     4 manu~ f        20    31 p     comp~
    ## 4 audi         a4     2.00  2008     4 auto~ f        21    30 p     comp~
    ## 5 audi         a4     2.80  1999     6 auto~ f        16    26 p     comp~
    ## 6 audi         a4     2.80  1999     6 manu~ f        18    26 p     comp~

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
    ##  9 lincoln          17.0
    ## 10 mercury          18.0
    ## 11 nissan           24.6
    ## 12 pontiac          26.4
    ## 13 subaru           25.6
    ## 14 toyota           24.9
    ## 15 volkswagen       29.2

``` r
#My custom function
my_hwymean <- function(df, ...) {
  #... here allows us to pass any number of arguments to our function
  mydf <- df %>% group_by_(...) %>% summarise(hwy_mean = mean(hwy))
  return(mydf)
}

#Aggregate by both manufacturer and model
my_hwymean(mpg, 'manufacturer', 'model')
```

    ## # A tibble: 38 x 3
    ## # Groups:   manufacturer [?]
    ##    manufacturer model              hwy_mean
    ##    <chr>        <chr>                 <dbl>
    ##  1 audi         a4                     28.3
    ##  2 audi         a4 quattro             25.8
    ##  3 audi         a6 quattro             24.0
    ##  4 chevrolet    c1500 suburban 2wd     17.8
    ##  5 chevrolet    corvette               24.8
    ##  6 chevrolet    k1500 tahoe 4wd        16.2
    ##  7 chevrolet    malibu                 27.6
    ##  8 dodge        caravan 2wd            22.4
    ##  9 dodge        dakota pickup 4wd      17.0
    ## 10 dodge        durango 4wd            16.0
    ## # ... with 28 more rows

``` r
#Aggregate by transition type, in addition to manufacturer and model
my_hwymean(mpg, 'manufacturer', 'model', 'trans')
```

    ## # A tibble: 113 x 4
    ## # Groups:   manufacturer, model [?]
    ##    manufacturer model      trans      hwy_mean
    ##    <chr>        <chr>      <chr>         <dbl>
    ##  1 audi         a4         auto(av)       28.5
    ##  2 audi         a4         auto(l5)       27.5
    ##  3 audi         a4         manual(m5)     27.5
    ##  4 audi         a4         manual(m6)     31.0
    ##  5 audi         a4 quattro auto(l5)       25.0
    ##  6 audi         a4 quattro auto(s6)       26.0
    ##  7 audi         a4 quattro manual(m5)     25.5
    ##  8 audi         a4 quattro manual(m6)     26.5
    ##  9 audi         a6 quattro auto(l5)       24.0
    ## 10 audi         a6 quattro auto(s6)       24.0
    ## # ... with 103 more rows
