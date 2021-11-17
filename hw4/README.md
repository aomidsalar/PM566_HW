---
title: "Homework 4"
author: "Audrey Omidsalar"
date: "11/19/2021"
output:
  html_document:
    toc: yes
    toc_float: yes
    keep_md: yes
  github_document:
  always_allow_html: true
---



# HPC

## Problem 1: Make sure your code is nice

Rewrite the following R functions to make them faster. It is OK (and recommended) to take a look at Stackoverflow and Google


```r
# Total row sums
fun1 <- function(mat) {
  n <- nrow(mat)
  ans <- double(n) 
  for (i in 1:n) {
    ans[i] <- sum(mat[i, ])
  }
  ans
}

fun1alt <- function(mat) {
  rowSums(mat)
}

# Cumulative sum by row
fun2 <- function(mat) {
  n <- nrow(mat)
  k <- ncol(mat)
  ans <- mat
  for (i in 1:n) {
    for (j in 2:k) {
      ans[i,j] <- mat[i, j] + ans[i, j - 1]
    }
  }
  ans
}

fun2alt <- function(mat) {
  t(apply(mat,1,cumsum))
}


# Use the data with this code
set.seed(2315)
dat <- matrix(rnorm(200 * 100), nrow = 200)

# Test for the first
microbenchmark::microbenchmark(
  fun1(dat),
  fun1alt(dat), unit = "relative", check = "equivalent"
)
```

```
## Unit: relative
##          expr      min       lq     mean   median       uq      max neval
##     fun1(dat) 9.058614 9.268546 8.091609 9.256586 8.622572 2.851653   100
##  fun1alt(dat) 1.000000 1.000000 1.000000 1.000000 1.000000 1.000000   100
```

```r
# Test for the second
microbenchmark::microbenchmark(
  fun2(dat),
  fun2alt(dat), unit = "relative", check = "equivalent"
)
```

```
## Unit: relative
##          expr      min       lq     mean   median       uq      max neval
##     fun2(dat) 4.124539 2.522451 1.798733 2.229226 1.710504 1.154492   100
##  fun2alt(dat) 1.000000 1.000000 1.000000 1.000000 1.000000 1.000000   100
```

## Problem 2: Make things run faster with parallel computing

The following function allows simulating PI


```r
sim_pi <- function(n = 1000, i = NULL) {
  p <- matrix(runif(n*2), ncol = 2)
  mean(rowSums(p^2) < 1) * 4
}

# Here is an example of the run
set.seed(156)
sim_pi(1000) # 3.132
```

```
## [1] 3.132
```

In order to get accurate estimates, we can run this function multiple times, with the following code:


```r
# This runs the simulation a 4,000 times, each with 10,000 points
set.seed(1231)
system.time({
  ans <- unlist(lapply(1:4000, sim_pi, n = 10000))
  print(mean(ans))
})
```

```
## [1] 3.14124
```

```
##    user  system elapsed 
##   4.111   1.259   5.668
```

Rewrite the previous code using `parLapply()` to make it run faster. Make sure you set the seed using `clusterSetRNGStream()`:


```r
library('parallel')
system.time({
  cl <- makePSOCKcluster(4)
  clusterSetRNGStream(cl, 1231)
  ans <- unlist(parLapply(cl=cl, 1:4000,sim_pi, n=10000))
  print(mean(ans))
   stopCluster(cl)
})
```

```
## [1] 3.141578
```

```
##    user  system elapsed 
##   0.015   0.010   4.704
```

Not surprisingly, this method is faster (less elapsed time) than the previous method.

# SQL

Setup a temporary database by running the following chunk


```r
# install.packages(c("RSQLite", "DBI"))

library(RSQLite)
library(DBI)

# Initialize a temporary in memory database
con <- dbConnect(SQLite(), ":memory:")

# Download tables
film <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film.csv")
film_category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film_category.csv")
category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/category.csv")

# Copy data.frames to database
dbWriteTable(con, "film", film)
dbWriteTable(con, "film_category", film_category)
dbWriteTable(con, "category", category)
```

## Question 1

How many many movies is there avaliable in each `rating` category?

*Films with a ___PG-13___ rating have the most movies available.*


```sql
SELECT rating, COUNT(title) as N_movies
FROM film
GROUP BY rating
```


<div class="knitsql-table">


Table: 5 records

|rating | N_movies|
|:------|--------:|
|G      |      180|
|NC-17  |      210|
|PG     |      194|
|PG-13  |      223|
|R      |      195|

</div>

## Question 2

What is the average replacement cost and rental rate for each `rating` category?

*Films with a ___PG___ rating have the highest average rental rate and lowest average replacement cost.*


```sql
SELECT rating, AVG(replacement_cost) as avg_replacement_cost,
AVG(rental_rate) as avg_rental_rate
FROM film
GROUP BY rating
```


<div class="knitsql-table">


Table: 5 records

|rating | avg_replacement_cost| avg_rental_rate|
|:------|--------------------:|---------------:|
|G      |             20.12333|        2.912222|
|NC-17  |             20.13762|        2.970952|
|PG     |             18.95907|        3.051856|
|PG-13  |             20.40256|        3.034843|
|R      |             20.23103|        2.938718|

</div>

## Question 3

Use table `film_category` together with `film` to find the how many films there are with each category ID

*There are 16 categories of films. The most popular is ___category ID 15___ with 74 films, closely followed by ___category ID 9___ with 73 films.*


```sql
SELECT category_id, COUNT(title) as N_films
FROM film AS a INNER JOIN film_category AS b
ON a.film_id = b.film_id
GROUP BY category_id
ORDER BY N_films DESC
```


<div class="knitsql-table">


Table: Displaying records 1 - 10

| category_id| N_films|
|-----------:|-------:|
|          15|      74|
|           9|      73|
|           8|      69|
|           6|      68|
|           2|      66|
|           1|      64|
|          13|      63|
|           7|      62|
|          14|      61|
|          10|      61|

</div>

## Question 4

Incorporate table `category` into the answer to the previous question to find the name of the most popular category.

*The most popular category is ___Sports___ with 74 films, followed by ___Foreign___ with 73 films.*


```sql
SELECT category.category_id, category.name, COUNT(film.title) as N_films
FROM ((film_category INNER JOIN film ON film_category.film_id = film.film_id)
INNER JOIN category ON film_category.category_id = category.category_id)
GROUP BY category.category_id
ORDER BY N_films DESC
```


<div class="knitsql-table">


Table: Displaying records 1 - 10

| category_id|name        | N_films|
|-----------:|:-----------|-------:|
|          15|Sports      |      74|
|           9|Foreign     |      73|
|           8|Family      |      69|
|           6|Documentary |      68|
|           2|Animation   |      66|
|           1|Action      |      64|
|          13|New         |      63|
|           7|Drama       |      62|
|          14|Sci-Fi      |      61|
|          10|Games       |      61|

</div>

```r
# clean up
dbDisconnect(con)
```


