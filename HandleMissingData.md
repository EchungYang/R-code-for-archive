Handle missing data - Basics & Multiple Imputation with mice
================
Yiqiong Yang (Miriam)
2022-08-18

### Missing data

Missing data occur when no data value is stored for the variable in an
observation. The problem of missing data is relatively common in almost
all research and can have a significant effect on the conclusions that
can be drawn from the data (Kang, 2013).

In this markdown we will introduce some basic approaches to address
missing data.

-   Remove missing data completely
-   Replacement
-   Multiple Imputation

``` r
library(knitr) # Markdown presentation
library(tidyverse) # Data wragnling and visualisation
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──

    ## ✔ ggplot2 3.3.6     ✔ purrr   0.3.4
    ## ✔ tibble  3.1.7     ✔ dplyr   1.0.9
    ## ✔ tidyr   1.2.0     ✔ stringr 1.4.0
    ## ✔ readr   2.1.2     ✔ forcats 0.5.1

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(visdat) # Visualise missing data
library(mice) # Multiple imputation
```

    ## 
    ## Attaching package: 'mice'

    ## The following object is masked from 'package:stats':
    ## 
    ##     filter

    ## The following objects are masked from 'package:base':
    ## 
    ##     cbind, rbind

``` r
library(dplyr) # Data wrangling
```

Now we are creating a dataset with four columns Col1: the name of eight
students Col2 - Col4: numbers of supervision hours across three academic
years

``` r
suphours <- data.frame(Student = c("John", "Amos", "Can", "Eman", "Joy",
             "Flora", "Max", "Kay"),
           year2019 = c(10, NA, 13, 8, 12, NA, 9, 11),
           year2020 = sample(12:20, 8, replace = F),
           year2021 = c(12, NA, 12, NA, NA, 9, 25, 7))

## Transpose the wide format to long format for demo purpose
suphourslong <- suphours %>%
  gather("Years", "Hours", 2:4) %>%
  mutate(Years = gsub("\\D", "", Years))

head(suphours, 8)
```

    ##   Student year2019 year2020 year2021
    ## 1    John       10       18       12
    ## 2    Amos       NA       13       NA
    ## 3     Can       13       12       12
    ## 4    Eman        8       16       NA
    ## 5     Joy       12       14       NA
    ## 6   Flora       NA       19        9
    ## 7     Max        9       20       25
    ## 8     Kay       11       15        7

``` r
head(suphourslong, 8)
```

    ##   Student Years Hours
    ## 1    John  2019    10
    ## 2    Amos  2019    NA
    ## 3     Can  2019    13
    ## 4    Eman  2019     8
    ## 5     Joy  2019    12
    ## 6   Flora  2019    NA
    ## 7     Max  2019     9
    ## 8     Kay  2019    11

From the dataset we could see that there are several na in Hours. The
first approach is to remove all rows contain NA. When we have a large
sample size with less missing values, removal of missing data would not
necessarily bias the result of analysis.

### Visualising missing data through visdat & mice

Visualising missing date provide you a general idea of where they are
and how many they are. For example, whether missing data follow a
certain pattern? Whether there are a big chunk of missing data? If they
do, then they are probably not missing at random, i.e., missing data at
entry stage. Therefore, different treatment is needed.

``` r
# Visualise missing data 
visdat::vis_dat(suphourslong)
```

    ## Warning: `gather_()` was deprecated in tidyr 1.2.0.
    ## Please use `gather()` instead.
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was generated.

![](HandleMissingData_files/figure-gfm/Visualise%20missing%20data-1.png)<!-- -->

``` r
mice::md.pattern(suphourslong)
```

![](HandleMissingData_files/figure-gfm/Visualise%20missing%20data-2.png)<!-- -->

    ##    Student Years Hours  
    ## 19       1     1     1 0
    ## 5        1     1     0 1
    ##          0     0     5 5

### Remove rows containing NA

There are different ways to remove rows containing NA. We can do that
through dplyr::filter or drop_na. Visualised graphs tell that the NA
have been completely removed.

``` r
# dplyr::filter
suphourslong %>%
  filter(!is.na(Hours)) %>%  # is.na() return rows containing na. '!' means not equal
  visdat::vis_dat()
```

![](HandleMissingData_files/figure-gfm/Remove%20rows%20containing%20NA-1.png)<!-- -->

``` r
suphourslong %>%
  filter(complete.cases(Hours)) %>% # complete.cases is self-explanatory
  mice::md.pattern()
```

    ##  /\     /\
    ## {  `---'  }
    ## {  O   O  }
    ## ==>  V <==  No need for mice. This data set is completely observed.
    ##  \  \|/  /
    ##   `-----'

![](HandleMissingData_files/figure-gfm/Remove%20rows%20containing%20NA-2.png)<!-- -->

    ##    Student Years Hours  
    ## 19       1     1     1 0
    ##          0     0     0 0

``` r
# drop_na
suphourslong %>%
  drop_na(Hours) %>%
  visdat::vis_dat()
```

![](HandleMissingData_files/figure-gfm/Remove%20rows%20containing%20NA-3.png)<!-- -->

### Replace NA with other values

Removing cases or participants would cause bias results, so we want to
keep them into the analysis. We could replace them with other values or
characters. The code below demonstrate how to replace missing values
with 0.

``` r
# For example, replace NA with 0 with long format data
suphourslong %>%
  mutate(Hours = replace_na(Hours, 0)) %>%
  head()
```

    ##   Student Years Hours
    ## 1    John  2019    10
    ## 2    Amos  2019     0
    ## 3     Can  2019    13
    ## 4    Eman  2019     8
    ## 5     Joy  2019    12
    ## 6   Flora  2019     0

``` r
# For example, replace NA with 0 with wide format data applying to all the element and return a data frame, this is when 'apply' is applicable.
suphours_na_0 <- 
  as.data.frame(apply(suphours[, 2:4], 2, replace_na, 0)) %>%
# Combine it with the student name from the original dataset  
  cbind(Student = suphours[,1])

# Check the new dataset, na has been replaced by 0.
head(suphours_na_0, 8)
```

    ##   year2019 year2020 year2021 Student
    ## 1       10       18       12    John
    ## 2        0       13        0    Amos
    ## 3       13       12       12     Can
    ## 4        8       16        0    Eman
    ## 5       12       14        0     Joy
    ## 6        0       19        9   Flora
    ## 7        9       20       25     Max
    ## 8       11       15        7     Kay

### Replace NA with mean

We can replace NA with mean

``` r
suphourslong %>%
  mutate(Hours = replace_na(Hours, round(mean(Hours, na.rm = T), digits = 0))) %>%
  head(8)
```

    ##   Student Years Hours
    ## 1    John  2019    10
    ## 2    Amos  2019    13
    ## 3     Can  2019    13
    ## 4    Eman  2019     8
    ## 5     Joy  2019    12
    ## 6   Flora  2019    13
    ## 7     Max  2019     9
    ## 8     Kay  2019    11

``` r
# Through error
# suphours_na_mean <- 
#   as.data.frame(sapply(suphours[,2:4], as.numeric)) %>%
#   mutate_at(vars(1:3), ~replace_na(., mean(., na.rm = TRUE)))
# 
```

### Repalce NA with characters

When we want to replace na with character, we need to be careful with
the type. If we replace na with character in numeric column/rows, we
need to make sure the type is consistent throughout. So we need to
convert that column into a matching type, i.e., character in the code
below. Inconsistent type would through an error.

``` r
suphourslong %>%
  mutate(Hours = replace_na(as.character(Hours), "none"),
         Student = replace_na(as.character(Student), "none")) %>%
  head(8)
```

    ##   Student Years Hours
    ## 1    John  2019    10
    ## 2    Amos  2019  none
    ## 3     Can  2019    13
    ## 4    Eman  2019     8
    ## 5     Joy  2019    12
    ## 6   Flora  2019  none
    ## 7     Max  2019     9
    ## 8     Kay  2019    11

## Multiple Imputation & mice

The above approaches to replace or remove na are basic and limited to
certain sample size or data. When we do not want to remove any missing
cases, or replace them with other values, we could apply multiple
imputation.

The mice package implements a method to deal with missing data. The
package creates multiple imputations (replacement values) for
multivariate missing data. The method is based on Fully Conditional
Specification, where each incomplete variable is imputed by a separate
model. The MICE algorithm can impute mixes of continuous, binary,
unordered categorical and ordered categorical data. In addition, MICE
can impute continuous two-level data, and maintain consistency between
imputations by means of passive imputation. Many diagnostic plots are
implemented to inspect the quality of the imputations.

### mice syntax explained

m – is the number of imputations, the default is 5. defaultMethod –
indicates which method you want to use for imputation, the default is
PMM. Accepted values are logreg, polyreg, and polr. maxit– A scalar
gives the number of iterations. The default is 5. seed – offsetting the
random number generator.

Start imputation using mice:

``` r
# Here we are asking R to iterate the data for 3 times,
imputed_data <- mice(suphours,m=3,maxit=50,meth='pmm',seed=500)
```

    ## 
    ##  iter imp variable
    ##   1   1  year2019  year2021
    ##   1   2  year2019  year2021
    ##   1   3  year2019  year2021
    ##   2   1  year2019  year2021
    ##   2   2  year2019  year2021
    ##   2   3  year2019  year2021
    ##   3   1  year2019  year2021
    ##   3   2  year2019  year2021
    ##   3   3  year2019  year2021
    ##   4   1  year2019  year2021
    ##   4   2  year2019  year2021
    ##   4   3  year2019  year2021
    ##   5   1  year2019  year2021
    ##   5   2  year2019  year2021
    ##   5   3  year2019  year2021
    ##   6   1  year2019  year2021
    ##   6   2  year2019  year2021
    ##   6   3  year2019  year2021
    ##   7   1  year2019  year2021
    ##   7   2  year2019  year2021
    ##   7   3  year2019  year2021
    ##   8   1  year2019  year2021
    ##   8   2  year2019  year2021
    ##   8   3  year2019  year2021
    ##   9   1  year2019  year2021
    ##   9   2  year2019  year2021
    ##   9   3  year2019  year2021
    ##   10   1  year2019  year2021
    ##   10   2  year2019  year2021
    ##   10   3  year2019  year2021
    ##   11   1  year2019  year2021
    ##   11   2  year2019  year2021
    ##   11   3  year2019  year2021
    ##   12   1  year2019  year2021
    ##   12   2  year2019  year2021
    ##   12   3  year2019  year2021
    ##   13   1  year2019  year2021
    ##   13   2  year2019  year2021
    ##   13   3  year2019  year2021
    ##   14   1  year2019  year2021
    ##   14   2  year2019  year2021
    ##   14   3  year2019  year2021
    ##   15   1  year2019  year2021
    ##   15   2  year2019  year2021
    ##   15   3  year2019  year2021
    ##   16   1  year2019  year2021
    ##   16   2  year2019  year2021
    ##   16   3  year2019  year2021
    ##   17   1  year2019  year2021
    ##   17   2  year2019  year2021
    ##   17   3  year2019  year2021
    ##   18   1  year2019  year2021
    ##   18   2  year2019  year2021
    ##   18   3  year2019  year2021
    ##   19   1  year2019  year2021
    ##   19   2  year2019  year2021
    ##   19   3  year2019  year2021
    ##   20   1  year2019  year2021
    ##   20   2  year2019  year2021
    ##   20   3  year2019  year2021
    ##   21   1  year2019  year2021
    ##   21   2  year2019  year2021
    ##   21   3  year2019  year2021
    ##   22   1  year2019  year2021
    ##   22   2  year2019  year2021
    ##   22   3  year2019  year2021
    ##   23   1  year2019  year2021
    ##   23   2  year2019  year2021
    ##   23   3  year2019  year2021
    ##   24   1  year2019  year2021
    ##   24   2  year2019  year2021
    ##   24   3  year2019  year2021
    ##   25   1  year2019  year2021
    ##   25   2  year2019  year2021
    ##   25   3  year2019  year2021
    ##   26   1  year2019  year2021
    ##   26   2  year2019  year2021
    ##   26   3  year2019  year2021
    ##   27   1  year2019  year2021
    ##   27   2  year2019  year2021
    ##   27   3  year2019  year2021
    ##   28   1  year2019  year2021
    ##   28   2  year2019  year2021
    ##   28   3  year2019  year2021
    ##   29   1  year2019  year2021
    ##   29   2  year2019  year2021
    ##   29   3  year2019  year2021
    ##   30   1  year2019  year2021
    ##   30   2  year2019  year2021
    ##   30   3  year2019  year2021
    ##   31   1  year2019  year2021
    ##   31   2  year2019  year2021
    ##   31   3  year2019  year2021
    ##   32   1  year2019  year2021
    ##   32   2  year2019  year2021
    ##   32   3  year2019  year2021
    ##   33   1  year2019  year2021
    ##   33   2  year2019  year2021
    ##   33   3  year2019  year2021
    ##   34   1  year2019  year2021
    ##   34   2  year2019  year2021
    ##   34   3  year2019  year2021
    ##   35   1  year2019  year2021
    ##   35   2  year2019  year2021
    ##   35   3  year2019  year2021
    ##   36   1  year2019  year2021
    ##   36   2  year2019  year2021
    ##   36   3  year2019  year2021
    ##   37   1  year2019  year2021
    ##   37   2  year2019  year2021
    ##   37   3  year2019  year2021
    ##   38   1  year2019  year2021
    ##   38   2  year2019  year2021
    ##   38   3  year2019  year2021
    ##   39   1  year2019  year2021
    ##   39   2  year2019  year2021
    ##   39   3  year2019  year2021
    ##   40   1  year2019  year2021
    ##   40   2  year2019  year2021
    ##   40   3  year2019  year2021
    ##   41   1  year2019  year2021
    ##   41   2  year2019  year2021
    ##   41   3  year2019  year2021
    ##   42   1  year2019  year2021
    ##   42   2  year2019  year2021
    ##   42   3  year2019  year2021
    ##   43   1  year2019  year2021
    ##   43   2  year2019  year2021
    ##   43   3  year2019  year2021
    ##   44   1  year2019  year2021
    ##   44   2  year2019  year2021
    ##   44   3  year2019  year2021
    ##   45   1  year2019  year2021
    ##   45   2  year2019  year2021
    ##   45   3  year2019  year2021
    ##   46   1  year2019  year2021
    ##   46   2  year2019  year2021
    ##   46   3  year2019  year2021
    ##   47   1  year2019  year2021
    ##   47   2  year2019  year2021
    ##   47   3  year2019  year2021
    ##   48   1  year2019  year2021
    ##   48   2  year2019  year2021
    ##   48   3  year2019  year2021
    ##   49   1  year2019  year2021
    ##   49   2  year2019  year2021
    ##   49   3  year2019  year2021
    ##   50   1  year2019  year2021
    ##   50   2  year2019  year2021
    ##   50   3  year2019  year2021

    ## Warning: Number of logged events: 1

``` r
summary(imputed_data)
```

    ## Class: mids
    ## Number of multiple imputations:  3 
    ## Imputation methods:
    ##  Student year2019 year2020 year2021 
    ##       ""    "pmm"       ""    "pmm" 
    ## PredictorMatrix:
    ##          Student year2019 year2020 year2021
    ## Student        0        1        1        1
    ## year2019       0        0        1        1
    ## year2020       0        1        0        1
    ## year2021       0        1        1        0
    ## Number of logged events:  1 
    ##   it im dep     meth     out
    ## 1  0  0     constant Student

The output of calling mice is a group of lists. If we want to check the
imputed values, we need to specify it in ‘imp’, and further specify the
column of the imputation.

``` r
imputed_data$imp$year2019 # For example, check the imputed values for year2019
```

    ##   1  2  3
    ## 2 8 11 12
    ## 6 9 12 12

When imputation has done, we could replace the NA in the original
dataset by the imputed values we just created.

``` r
# Assign the imputed dataset to 'finished_imputed'
finished_imputed <- complete(imputed_data, 2)
head(suphours, 8)
```

    ##   Student year2019 year2020 year2021
    ## 1    John       10       18       12
    ## 2    Amos       NA       13       NA
    ## 3     Can       13       12       12
    ## 4    Eman        8       16       NA
    ## 5     Joy       12       14       NA
    ## 6   Flora       NA       19        9
    ## 7     Max        9       20       25
    ## 8     Kay       11       15        7

``` r
head(finished_imputed, 8)
```

    ##   Student year2019 year2020 year2021
    ## 1    John       10       18       12
    ## 2    Amos       11       13       12
    ## 3     Can       13       12       12
    ## 4    Eman        8       16        7
    ## 5     Joy       12       14       12
    ## 6   Flora       12       19        9
    ## 7     Max        9       20       25
    ## 8     Kay       11       15        7

``` r
# Note the replacement of categorical column is still incomplete
```

References:

Kang H. The prevention and handling of the missing data. Korean J
Anesthesiol. 2013 May;64(5):402-6. doi: 10.4097/kjae.2013.64.5.402. Epub
2013 May 24. PMID: 23741561; PMCID: PMC3668100.

<https://datasciencebeginners.com/2018/11/11/a-brief-introduction-to-mice-r-package/>

<https://www.rdocumentation.org/packages/mice/versions/3.14.0/topics/mice>

<https://www.codingprof.com/how-to-replace-nas-with-the-mean-in-r-examples/>

<https://datascienceplus.com/imputing-missing-data-with-r-mice-package/>

<https://www.youtube.com/watch?v=sUAMiAIUhcI>
