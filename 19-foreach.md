---
layout: page
title: IntermediateR for reproducible scientific analysis
subtitle: parallel progamming
minutes: 20
---



> ## Learning objectives {.objectives}
>
> * To understand the concept of "embarassingly parallel" problems
> * To be able to set up a parallel backend for R
> * To be able to effectively parallelise for loops using the foreach package
> * To be able to assess the overall runtime of a piece of code in R.
>

Some problems encountered in scientific research fall into the category known as
"embarrasingly parallel": analyses which require a very large number of 
calculations, but whose calculations do not depend on each other. A classic 
example of this comes from genetics: Genome-wide association studies involve the
testing of hundreds of thousands of genetic variants across the genome for 
an association with a disease or trait. Each genetic variant can be tested 
independently, that is, the calculation of their association does not require
the results of any of the other tests.

In this lesson we're going to learn how to parallelise this kind of task over
multiple cores on your computer, and the pitfalls to avoid.

### Registering a parallel backend

First, we need to tell R how many cores to use for any parallel calculation. For
this lesson we will tell R to use one less core than the total number of cores
on our machine: this leaves one processor free for other tasks while calculations
happen in the background:


```r
library(doParallel)
```

```
## Loading required package: foreach
## Loading required package: iterators
## Loading required package: parallel
```

```r
cl <- makeCluster(2)
registerDoParallel(cl)
```

> #### How many cores? {.callout}
>
> In practice you should never register more cores with R than 
> you have on your computer, otherwise the parallel calculations might cause your
> computer to "thrash": grind to a halt and run extremely slowly. Note even the
> parallel calculations will run extremely slowly, so this offers no advantage.
>
> The `detectCores` function will tell you how many cores you can safely 
> parallelise over:
>
> 
> ```r
> library(parallel)
> detectCores()
> ```
> 
> ```
> ## [1] 8
> ```
>

### Parallel for loops

There are several components to a parallel for loop. Let's learn through an 
example:


```r
# Load in the parallel for loop library
library(foreach)
# Get a list of countries
countries <- unique(gap$country)
# Calculate the change in life expectancy over the last 55 years for each country
diffLifeExp <- foreach(cc = countries, .combine=rbind, .packages="data.table") %dopar% {
  diffLE <- gap[country == cc, max(lifeExp) - min(lifeExp)]
  data.table(
    country=cc,
    diffLifeExp=diffLE
  )
}
diffLifeExp
```

```
##                 country diffLifeExp
##   1:        Afghanistan      15.027
##   2:            Albania      21.193
##   3:            Algeria      29.224
##   4:             Angola      12.716
##   5:          Argentina      12.835
##  ---                               
## 138:            Vietnam      33.837
## 139: West Bank and Gaza      30.262
## 140:         Yemen Rep.      30.150
## 141:             Zambia      12.628
## 142:           Zimbabwe      22.362
```

Here, we use the `foreach` function instead of a `for` loop. The `%dopar%` 
operator immediately after the `foreach()` function tells `foreach` to run its
calculations (i.e. the body of the foreach loop) independently across multiple
cores. 

If we are parallelising over 4 cores, the calculations for the first 
four countries will run in parallel: the change in life expectancy will be 
calculated for Afghanistan, Albania, Algeria, and Angola at the same time, these
results will be returned, then the next four countries will be calculated, and
so on.

Just like the `apply` family of functions, the results of the last line in the 
body of the foreach loop will be returned and combined by `foreach`. The default
is to combine the results into a `list`, just like `lapply`. However, we can 
specify a different function to combine our results with using the `.combine`
argument. Here we have told `foreach` to use the `rbind` function, so that each 
result becomes a row in a new `data.table`.

Finally, we use the `.packages` argument to tell `foreach` which packages it 
needs for the calculations in the body of the `foreach` loop. The parallel 
calculations happen in independent R sessions, so each of these R sessions needs
to be aware of the packages it needs to run the calculations.

### Efficient parallelisation and communication overhead

Parallelisation doesn't come for free: there is a communication overhead in 
sending objects to and receiving objects from each parallel core.

In the following examples, we will calculate the mean global life expectancy in
each year, and use `system.time` to examine the total runtime of the code.


```r
years <- unique(gap$year)
runtime1 <- system.time({
  meanLifeExp <- foreach(
    yy = years, 
    .packages="data.table"
   ) %dopar% {
    mean(gap[year == yy, lifeExp])
  }
})
runtime1
```

```
##    user  system elapsed 
##   0.010   0.001   0.020
```

The `system.time` function takes an some R code (wrapped in curly braces, `{`, 
`}`), runs it, then returns the total time taken by the code to run. In this 
case we care about the "elapsed" field: this gives the "real" time taken by the
computer to run the code. The "user" and "system" fields provide information 
about CPU time taken by various levels of the operating system, see 
`help("proc.time")`.

This was actually a very inefficient way to parallelise our code. Since there
are 12 different `years`, the code is sending out the calculations
for 1952 and 1957 to the two cores, collecting the results, then
sending out 1962 and 1967 and so on. 

It is much more effective to break the problem into "chunks": one for each 
parallel core. The `itertools` package provides useful functions that integrate
with `foreach` for solving this. Here we'll use `isplitVector` to split `years`
into two chunks, then on each parallel core, use a non-parallelised (serial)
`foreach` loop to run the calculations for each year. Note we need to tell each
parallel core about the `foreach` package. We'll also use the `getDoParWorkers`
function to automatically detect how many cores we have registered. This means 
you can return with a different number of cores later without having to update
values throughout your code:


```r
library(itertools)
runtime2 <- system.time({
  meanLifeExp <- foreach(
      chunk = isplitVector(years, chunks=getDoParWorkers()), 
      .packages=c("foreach", "data.table")
  ) %dopar% {
    foreach(yy = chunk, .combine=rbind) %do% {
      mean(gap[year == yy, lifeExp])
    }
  }
})
runtime2
```

```
##    user  system elapsed 
##   0.008   0.000   0.059
```

Suprisingly this code was much slower! 
So what happened? In this case the overhead of setting up the parallel code 
efficiently outweighed the benefits. Since we're working with a toy dataset, the
actual calculations are very, very, fast. In comparison to our first example,
we've now added the `foreach` package, which gets loaded in each new R session,
as well as the chunking of tasks through `isplitVector`. Together, these take
longer than the actual calculations!

For example we run the code not in parallel using `sapply` we see the fastest 
overall time:


```r
runtime3 <- system.time({
  meanLifeExp <- sapply(years, function(yy) {
    mean(gap[year == yy, lifeExp])
  })
})
runtime3
```

```
##    user  system elapsed 
##   0.013   0.000   0.013
```

Finally, an important consideration when parallelising code is deciding how many 
cores is the most effective for the task at hand. With too many cores the chunks
can become small enough that the communication and setup overhead outweighs the 
benefit.

