---
title: "Tidy data"
subtitle: "Lesson 1: Summarizing data"
date: "September 2020"
output: 
  html_document:
    theme: flatly
    highlight: haddock 
    # code_folding: show
    toc: yes
    toc_depth: 4
    toc_float: yes
    keep_md: true
---

## Goals of this lesson: 

- Pipe commands with **%>%**
- Group data with **group_by()**
- Summarize over rows of data with **summarize()**


## 0. Set up 
load the tidyverse package, which you've already installed.

```r
library(tidyverse)
```

```
## ── Attaching packages ────────────────────────────────────────── tidyverse 1.3.0 ──
```

```
## ✓ ggplot2 3.3.1     ✓ purrr   0.3.3
## ✓ tibble  3.0.1     ✓ dplyr   1.0.0
## ✓ tidyr   1.1.0     ✓ stringr 1.4.0
## ✓ readr   1.3.1     ✓ forcats 0.5.0
```

```
## ── Conflicts ───────────────────────────────────────────── tidyverse_conflicts() ──
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

open the demo data.

```r
demo_data <- read_csv("../data/demo_data.csv")
```

## 1. Using the pipe %>%

The pipe allows you to chain together multiple functions, instead of writing repetitive code. We'll  start with a simple example.

### How many participants are in each group?

First, we tell R which dataset we are working with (demo_data)
Next, we can use **group_by()** to group the data by our categorical variable. 
Then, we can use **count()** to get the number of rows per group. 

We can use the pipe %>% to write out the functions step by step. 

```r
demo_data %>% 
  group_by(intervention) %>% 
  count() 
```

```
## # A tibble: 2 x 2
## # Groups:   intervention [2]
##   intervention     n
##   <chr>        <int>
## 1 Group 1         50
## 2 Group 2         50
```

### How many males and females are in each group?

We can add more than one variable in the **group_by()** function.

```r
demo_data %>%
  group_by(intervention, sex)  %>%
  count()
```

```
## # A tibble: 5 x 3
## # Groups:   intervention, sex [5]
##   intervention sex        n
##   <chr>        <chr>  <int>
## 1 Group 1      Female    25
## 2 Group 1      Male      25
## 3 Group 2      Female    24
## 4 Group 2      Male      24
## 5 Group 2      <NA>       2
```

Looks like we have some NAs in the sex column. 

We can remove these rows of data with **is.na()** and  **filter()**.

- **Filter()** removes any rows that don't meet the conditional statement. 

- **!(is.na())** returns a vector, where True = value and False = NA.

- Once we have filtered out the rows with NA, we can pipe the results into **group_by()** and **count().


```r
demo_data %>%
  # removes entire row for subjects with NA for sex.
  filter(!is.na(sex)) %>%
  group_by(intervention, sex) %>% 
  count()
```

```
## # A tibble: 4 x 3
## # Groups:   intervention, sex [4]
##   intervention sex        n
##   <chr>        <chr>  <int>
## 1 Group 1      Female    25
## 2 Group 1      Male      25
## 3 Group 2      Female    24
## 4 Group 2      Male      24
```

## 2. Summarize over rows of data 

### what is the mean age in our dataset?

First, we'll start with the data (demo_data)
Then, we can use **summarize()** and **mean()** to get the mean age.


```r
demo_data %>%
  summarize(mean(age))
```

```
## # A tibble: 1 x 1
##   `mean(age)`
##         <dbl>
## 1          NA
```

But something went wrong! why is the output NA?

If we don't want to filter(exclude) the participants with no age, we can just tell R to ignore NAs when computing the mean with the argument: **(na.rm = T)**

```r
demo_data %>%
  summarize(mean(age, na.rm = T))
```

```
## # A tibble: 1 x 1
##   `mean(age, na.rm = T)`
##                    <dbl>
## 1                   50.3
```

### What is the mean age per group? 

We can use the pipe to chain together several functions:

First, use group_by() to group the data by your categorical variable.

Then, we pipe the results of theh grouping into summarize() and mean() 

and this returns a very handy table!


```r
demo_data %>%
  group_by(intervention) %>%
  summarize(mean(age, na.rm = T))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```
## # A tibble: 2 x 2
##   intervention `mean(age, na.rm = T)`
##   <chr>                         <dbl>
## 1 Group 1                        54.2
## 2 Group 2                        46.4
```

### we can make this output prettier with a few customizations 
round(value, 2) will round the value to 2 decimal points 

```r
demo_data %>%
  group_by(intervention) %>%
  summarize(n = n(), 
            # we can assign new names for the table columns 
            'Age (mean)' = round(mean(age, na.rm = T),2),
            'Age (sd)' = round(sd(age, na.rm = T), 2),
             'Min Age' = min(age, na.rm = T), 
             'Max Age'= max(age, na.rm = T))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```
## # A tibble: 2 x 6
##   intervention     n `Age (mean)` `Age (sd)` `Min Age` `Max Age`
##   <chr>        <int>        <dbl>      <dbl>     <dbl>     <dbl>
## 1 Group 1         50         54.2       29.0         1       100
## 2 Group 2         50         46.4       29.0         3        99
```


## 4. Make a final table

```r
# assign the table to a new object 
demo_table <- demo_data %>%
  filter(!is.na(sex)) %>%
  group_by(intervention, sex) %>%
  summarize(n = n(), 
            'Age (mean)' = round(mean(age, na.rm = T),2),
            'Age (sd)' = round(sd(age, na.rm = T), 2),
             'Min Age' = min(age, na.rm = T), 
             'Max Age'= max(age, na.rm = T))
```

```
## `summarise()` regrouping output by 'intervention' (override with `.groups` argument)
```


```r
# print the new table object 
demo_table
```

```
## # A tibble: 4 x 7
## # Groups:   intervention [2]
##   intervention sex        n `Age (mean)` `Age (sd)` `Min Age` `Max Age`
##   <chr>        <chr>  <int>        <dbl>      <dbl>     <dbl>     <dbl>
## 1 Group 1      Female    25         56.1       27.6         2       100
## 2 Group 1      Male      25         52.4       30.7         1        98
## 3 Group 2      Female    24         45.9       28.4         5        99
## 4 Group 2      Male      24         49.2       30.0         3        97
```


## 5. Save the table object to csv.
Just like the copy of your data, this table only lives in your R environment! 

If you want to reference it for a future paper, or presentation, it's best to save it. 
For now, we will save the table in csv format.

While you might be used to excel format, you'll see that csv is a more universal file type that multiple programming languages can handle easily without distorting things. 

Before we write out this file, we need to decide where to save it to. 
Let's make a new "tables" folder in the 02_tidying_data directory.
**Files --> New Folder --> 'tables' --> OK**


```r
#write.csv(demo_table, file = "tables/demo_table.csv", row.names = F)
```

**Note:** The default settings of write.csv() will add row numbers as a column in your output. 99% of the time, we do not want this, so we add the argument 'row.names = F'




