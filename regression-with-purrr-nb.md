Regression with purrr
================
Jerry Chee
5/8/2020

In portfolio analysis, often times we are required to regress
instruments against a benchmark or reference index. In this short and
simple example, we explore how we could tackle this use-case of
obtaining the beta coefficient of several indexes against a benchmark.

Taking a look at the dataset. As observed there are four indexes across
various sectors and a `Market` benchmark.

``` r
indexes <- data.table::fread("temp/indexes.csv")
head(indexes)
```

    ##           id       date         return
    ## 1: Utilities 2002-01-01  0.00003519062
    ## 2: Utilities 2002-01-02  0.00448078073
    ## 3: Utilities 2002-01-03 -0.00143632860
    ## 4: Utilities 2002-01-04 -0.00679436804
    ## 5: Utilities 2002-01-05  0.00000000000
    ## 6: Utilities 2002-01-06  0.00000000000

``` r
unique(indexes$id)
```

    ## [1] "Utilities" "Airlines"  "Biotech"   "Banks"     "Market"

Traditionally, one might run a loop to pair each index against the
benchmark for a linear regression.

Below is an example how one might use the *split-apply-combine*
technique often used in other instances of data wrangling.

Because the operation we’re doing is to obtain a linear regression
model, this task is not so easily solved with a `group_by()` method.

``` r
#Reshape so Market is in one column, the rest in another column. 
#This is so that we can regress all indexes against Market respectively and extract the model stats.
indexes.long <- indexes %>%
  filter(date >= ymd(20110101), date <= ymd(20191231)) %>%
  distinct %>%
  pivot_wider(names_from = "id", values_from = "return") %>%
  pivot_longer(names_to = "index", values_to = "return", cols = -c("Market", "date"))

#Perform regression against ACWI -------------
models <- indexes.long %>%
  #Here's the equivalent of a group_by() with purrr
  split(.$index) %>%
  map(~lm(return ~ Market, data = .))

#Extract the beta of each index
models %>%
  map(summary) %>%
  #The beta coefficient we're looking for is in the second element
  map_dfr(~.$coefficients[2])
```

    ## # A tibble: 1 x 4
    ##   Airlines Banks Biotech Utilities
    ##      <dbl> <dbl>   <dbl>     <dbl>
    ## 1    0.975  1.13   0.986     0.655

And there you have it\! This is definitely more readable than a loop and
neater for extracting the beta coefficient quickly.

``` r
sessionInfo()
```

    ## R version 3.6.1 (2019-07-05)
    ## Platform: x86_64-w64-mingw32/x64 (64-bit)
    ## Running under: Windows 10 x64 (build 17134)
    ## 
    ## Matrix products: default
    ## 
    ## locale:
    ## [1] LC_COLLATE=English_Singapore.1252  LC_CTYPE=English_Singapore.1252   
    ## [3] LC_MONETARY=English_Singapore.1252 LC_NUMERIC=C                      
    ## [5] LC_TIME=English_Singapore.1252    
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ## [1] lubridate_1.7.4 purrr_0.3.3     tidyr_1.0.0     dplyr_0.8.3    
    ## [5] FactSet3_3.0   
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_1.0.3        knitr_1.26        magrittr_1.5      tidyselect_0.2.5 
    ##  [5] R6_2.4.1          rlang_0.4.1       fansi_0.4.0       stringr_1.4.0    
    ##  [9] tools_3.6.1       data.table_1.12.6 xfun_0.11         utf8_1.1.4       
    ## [13] cli_1.1.0         htmltools_0.4.0   yaml_2.2.0        assertthat_0.2.1 
    ## [17] digest_0.6.22     tibble_2.1.3      lifecycle_0.1.0   crayon_1.3.4     
    ## [21] vctrs_0.2.0       zeallot_0.1.0     glue_1.3.1        evaluate_0.14    
    ## [25] rmarkdown_1.17    stringi_1.4.3     compiler_3.6.1    pillar_1.4.2     
    ## [29] backports_1.1.5   pkgconfig_2.0.3
