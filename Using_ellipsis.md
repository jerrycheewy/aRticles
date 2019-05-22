Using ellipsis in a function
================
Jerry Chee
May 22, 2019

Ever wondered what the three dots `...` is when you're keying in arguments to a function?

`...` is called the ellipsis, and is a handy way of passing additional arguments to a function without constraints on the length/order.

An immediate use-case for something like this is with the dplyr verbs.

We'll work with the `mtcars` dataset for a simple demonstration.

``` r
# Basic set up
library(dplyr)
library(tidyr)

# Remind ourselves of what mtcars dataset looks like
head(mtcars)
```

    ##                    mpg cyl disp  hp drat    wt  qsec vs am gear carb
    ## Mazda RX4         21.0   6  160 110 3.90 2.620 16.46  0  1    4    4
    ## Mazda RX4 Wag     21.0   6  160 110 3.90 2.875 17.02  0  1    4    4
    ## Datsun 710        22.8   4  108  93 3.85 2.320 18.61  1  1    4    1
    ## Hornet 4 Drive    21.4   6  258 110 3.08 3.215 19.44  1  0    3    1
    ## Hornet Sportabout 18.7   8  360 175 3.15 3.440 17.02  0  0    3    2
    ## Valiant           18.1   6  225 105 2.76 3.460 20.22  1  0    3    1

Traditional way of counting the number of observations: How many cars within each cylinder group?

``` r
mtcars %>% group_by(cyl) %>% tally
```

    ## # A tibble: 3 x 2
    ##     cyl     n
    ##   <dbl> <int>
    ## 1     4    11
    ## 2     6     7
    ## 3     8    14

What if we wanted the ability to do a count based on a varying set of dimensions?

Let's write function to do just that!

``` r
mycount <- function(x, ...){
  x %>% group_by(...) %>% tally
}
```

Now let's ask: How many transmission (`am`) type per gear type?

``` r
mycount(mtcars, gear, am)
```

    ## # A tibble: 4 x 3
    ## # Groups:   gear [3]
    ##    gear    am     n
    ##   <dbl> <dbl> <int>
    ## 1     3     0    15
    ## 2     4     0     4
    ## 3     4     1     8
    ## 4     5     1     5

Looks like 3-geared cars is the most populous and all of them being automatic (am = 0) cars!

We could easily ask different questions by dynamically passing the list of dimensions to the `group_by` within our `mycount` function.

``` r
mycount(mtcars, gear, carb, am)
```

    ## # A tibble: 13 x 4
    ## # Groups:   gear, carb [11]
    ##     gear  carb    am     n
    ##    <dbl> <dbl> <dbl> <int>
    ##  1     3     1     0     3
    ##  2     3     2     0     4
    ##  3     3     3     0     3
    ##  4     3     4     0     5
    ##  5     4     1     1     4
    ##  6     4     2     0     2
    ##  7     4     2     1     2
    ##  8     4     4     0     2
    ##  9     4     4     1     2
    ## 10     5     2     1     2
    ## 11     5     4     1     1
    ## 12     5     6     1     1
    ## 13     5     8     1     1

``` r
mycount(mtcars, cyl, carb)
```

    ## # A tibble: 9 x 3
    ## # Groups:   cyl [3]
    ##     cyl  carb     n
    ##   <dbl> <dbl> <int>
    ## 1     4     1     5
    ## 2     4     2     6
    ## 3     6     1     2
    ## 4     6     4     4
    ## 5     6     6     1
    ## 6     8     2     4
    ## 7     8     3     3
    ## 8     8     4     6
    ## 9     8     8     1

But what if we wanted to always specify the top level group by gear group?

Simple! Just pre-specify that group within your custom function and it will always be there *in addition* to the additional dimensions you provided via the `...`!

``` r
mycount_gear <- function(x, ...){
  x %>% group_by(gear, ...) %>% tally
}

#Notice that the result here is exactly the same as in 2 sections above.
mycount_gear(mtcars, am)
```

    ## # A tibble: 4 x 3
    ## # Groups:   gear [3]
    ##    gear    am     n
    ##   <dbl> <dbl> <int>
    ## 1     3     0    15
    ## 2     4     0     4
    ## 3     4     1     8
    ## 4     5     1     5
