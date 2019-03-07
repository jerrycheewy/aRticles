Tidy Data
================
Jerry Chee
7 March 2019

What is 'tidy' data?
--------------------

If you've been working in Excel sheets for analytical work, learning how to see your data in a 'tidy' format is one of the most important concepts to start with. If you'd like to understand in detail the definition of 'tidy data' and a more comprehensive coverage on methods to achieve it, pls read Hadley Wickham's (Chief Data Scientist at RStudio) [research paper](https://vita.had.co.nz/papers/tidy-data.pdf).

Let's get started
-----------------

Here's how a typical dataset in Excel looks like. Likely so because it makes the lives of those keying data much easier and also more readable.

``` r
# Let's quickly load some standard R packages here so that you can follow this article. 
# If tidyverse is not yet installed, you may do so like such: > install.packages('tidyverse')
library(tidyverse)

# Creating some dummy data about exam results across different students and terms
myresults <- data.frame(
  Names = c("Angela", "Bryce", "Charlie"),
  Term1 = c(20, 80, 10),
  Term2 = c(30, 70, 50),
  Term3 = c(90, 80, 30))
```

The shape of the data here is helpful because we can quickly assess results either by student (by rows) or by term (by columns).

``` r
myresults
```

    ##     Names Term1 Term2 Term3
    ## 1  Angela    20    30    90
    ## 2   Bryce    80    70    80
    ## 3 Charlie    10    50    30

Conducting some simple analysis the Excel-way...
------------------------------------------------

OKAY, hold your horses. We're getting there!

Let's first calculate a simple statistic such as the **average result** per student *and* per term.

This is how you'd have likely done it in Excel:

1.  Calculate average per student in the column next to Term3
2.  Calculate average per term in the row below Charlie

<!-- -->

    ##     Names Term1 Term2 Term3 StudentAVG
    ## 1  Angela    20    30    90       46.7
    ## 2   Bryce    80    70    80       76.7
    ## 3 Charlie    10    50    30       30.0

    ##     Names Term1 Term2 Term3
    ## 1  Angela  20.0    30  90.0
    ## 2   Bryce  80.0    70  80.0
    ## 3 Charlie  10.0    50  30.0
    ## 4 TermAVG  36.7    50  66.7

It is unlikely for anything to go wrong if your dataset is this small, but adding new columns and new rows to your core dataset is not a sustainable practice, especially when your dataset is significantly larger and/or constantly updated. A common gripe about working in Excel with large datasets is either mispointed cell ranges or broken links caused by physically moving the data across cells.

Excel, while excellent for allowing one to 'touch' every single data point, fails quickly when you have an increasing number of datasets that you need to perform calculations on.

So how can I work better?
-------------------------

'Tidy'-ing your data simply means to discern between variables and values. In our example here, there are 3 variables and their corresponding values in brackets.

1.  **Names** (Angela, Bryce, Charlie)
2.  **Examination terms** (1, 2, 3)
3.  **Examination results** (10, 20, 30, ... , 90)

What we've been accustomed to, as Excel users, is to co-mingle variables with values.

Let's put our variables and values where they're supposed to be!

``` r
myresults.tidy <- myresults %>%
  gather(key = Term, value = Result, -Names)

myresults.tidy
```

    ##     Names  Term Result
    ## 1  Angela Term1     20
    ## 2   Bryce Term1     80
    ## 3 Charlie Term1     10
    ## 4  Angela Term2     30
    ## 5   Bryce Term2     70
    ## 6 Charlie Term2     50
    ## 7  Angela Term3     90
    ## 8   Bryce Term3     80
    ## 9 Charlie Term3     30

Notice what's different about our dataset? We now have 3 columns, corresponding to the 3 **variables** we identified earlier.

The **structure** of the dataset has changed and it has become **longer**!

This looks like it has been transposed, but notice that unlike transpose, which only switches rows to columns and vice-versa, we've actually created more data!

Each name is now duplicated 3 times, because it has to appear once per term.

The corresponding examination results are now displayed in the column titled *Result*.

While this is admittedly less readable than our initial form, this data structure is meant to allow us to perform analysis quicker and make code more readable.

How to use tidy data?
---------------------

Let's perform the same average calculations as done before, but leveraging on the new data structure!

``` r
# Student averages
myresults.tidy %>% 
  group_by(Names) %>%
  summarise(StudentAVG = mean(Result))
```

    ## # A tibble: 3 x 2
    ##   Names   StudentAVG
    ##   <fct>        <dbl>
    ## 1 Angela        46.7
    ## 2 Bryce         76.7
    ## 3 Charlie       30.0

``` r
# Term averages
myresults.tidy %>% 
  group_by(Term) %>% 
  summarise(TermAVG = mean(Result))
```

    ## # A tibble: 3 x 2
    ##   Term  TermAVG
    ##   <chr>   <dbl>
    ## 1 Term1    36.7
    ## 2 Term2    50.0
    ## 3 Term3    66.7

The benefit of working on tidy data may not be apparent yet, but will be once you begin working on more dimensions to your data.

An example where this dataset could get more complicated...

``` r
myInfo <- data.frame(Names = myresults$Names, 
                     Race = c("Chinese","Indian","Chinese"), 
                     Town = c("East Coast","Bedok","Yishun"))

myresults.tidy %>% left_join(myInfo)
```

    ## Joining, by = "Names"

    ##     Names  Term Result    Race       Town
    ## 1  Angela Term1     20 Chinese East Coast
    ## 2   Bryce Term1     80  Indian      Bedok
    ## 3 Charlie Term1     10 Chinese     Yishun
    ## 4  Angela Term2     30 Chinese East Coast
    ## 5   Bryce Term2     70  Indian      Bedok
    ## 6 Charlie Term2     50 Chinese     Yishun
    ## 7  Angela Term3     90 Chinese East Coast
    ## 8   Bryce Term3     80  Indian      Bedok
    ## 9 Charlie Term3     30 Chinese     Yishun

One might ask additional questions such as

-   What is the average result per Town?
-   What is the average result per Race?
-   What is the average result per Race within each Town?

We'll explore the power of tidy data in subsequent articles.
