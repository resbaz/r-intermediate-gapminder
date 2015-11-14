---
layout: page
title: IntermediateR for reproducible scientific analysis
subtitle: Apply functions
minutes: 20
---


> ## Learning objectives {.objectives}
>
> * To learn how to use *apply* to automate tasks efficiently
> * To know the difference between `apply`, `lapply`, `sapply`, `tapply` and 
>   `mapply`.
>

### Vectorized task automation

Previously we introduced you to `for` loops: a basic programming construct that 
is common across many programming languages. R has more optimised way of 
automating tasks that are not only faster than for loops, but also take away the
pain of having to pre-define your results object.

The most common function you will encounter is `lapply`, and the closely related
`sapply`. 

Lets take a look at the following `for` loop: 

~~~{.r}
for (cc in gap[,unique(continent)]) {
  popsum <- gap[year == 2007 & continent == cc, sum(pop)]
  print(paste(cc, ":", popsum))
}
~~~

It calculates the total population on each continent, then prints out the result.
If instead we want to save these results, we can either make a vector in advance
and save the results, or use one of the `apply` to take care of this detail for
us:


~~~{.r}
results <- lapply(gap[,unique(continent)], function(cc) {
  popsum <- gap[year == 2007 & continent == cc, sum(pop)]
  popsum
})
names(results) <- gap[,unique(continent)]
results
~~~



~~~{.output}
$Asia
[1] 3811953827

$Europe
[1] 586098529

$Africa
[1] 929539692

$Americas
[1] 898871184

$Oceania
[1] 24549947

~~~

`lapply` takes a vector (or list) as its first argument (in this case a vector 
of the continent names), then a function as its second argument. This function
is then executed on every element in the first argument. This is very similar to
a for loop: first, `cc` stores the first continent name, "Asia", then runs the 
code in the function body, then `cc` stores the second continent name, and runs
the function body, and so on. The code in the function body can be thought of in 
exactly the same way as the body of the `for` loop. The result of the last line
is then returned to `lapply`, which combines the results into a list.

`sapply` is identical to `lapply`, except that it tries to simplify the results
object. If we run the same code with `sapply` instead of `lapply` the results 
will be returned as a vector:


~~~{.r}
results <- sapply(gap[,unique(continent)], function(cc) {
  popsum <- gap[year == 2007 & continent == cc, sum(pop)]
  popsum
})
names(results) <- gap[,unique(continent)]
results
~~~



~~~{.output}
      Asia     Europe     Africa   Americas    Oceania 
3811953827  586098529  929539692  898871184   24549947 

~~~

### apply

The `apply` function is useful for matrix data: it allows you loop over either
the rows or columns of a matrix.


~~~{.r}
# create some dummy data
r <- matrix(rnorm(10*4), nrow=10)
colnames(r) <- letters[1:4]
rownames(r) <- LETTERS[1:10]
~~~


~~~{.r}
# Get the maximum value in each row:
apply(r, 1, max)
~~~



~~~{.output}
         A          B          C          D          E          F 
 0.1100324  1.6890364  0.4121760  0.4107368  0.9641043  1.4261307 
         G          H          I          J 
 1.1943626  0.1369749 -0.3391804  1.5082227 

~~~



~~~{.r}
# and for each column:
apply(r, 2, max)
~~~



~~~{.output}
        a         b         c         d 
1.6890364 1.1943626 1.5082227 0.9899344 

~~~

> ### means and sums {.callout}
>
> R has inbuilt functions for summing or calculating the mean of rows and 
> columns: `colSums`, `colMeans`, `rowSums`, `rowMeans`. These are faster than
> writing your own `apply`!
>

> ### the return of apply {.callout}
>
> When the function given to `apply` returns a vector instead of a single value
> the results will always be combined into columns, even if running the 
> function across the rows!
>

### mapply

The `mapply` function can be used to run a function with different combinations
of arguments. Let's take a look at an example:


~~~{.r}
a <- 1:4
b <- 4:1
mapply(rep, a, b)
~~~



~~~{.output}
[[1]]
[1] 1 1 1 1

[[2]]
[1] 2 2 2

[[3]]
[1] 3 3

[[4]]
[1] 4

~~~

This is the same as running the following code:


~~~{.r}
rep(a[1], b[1])
~~~



~~~{.output}
[1] 1 1 1 1

~~~



~~~{.r}
rep(a[2], b[2])
~~~



~~~{.output}
[1] 2 2 2

~~~



~~~{.r}
rep(a[3], b[3])
~~~



~~~{.output}
[1] 3 3

~~~



~~~{.r}
rep(a[4], b[4])
~~~



~~~{.output}
[1] 4

~~~

or the following `lapply` statement:


~~~{.r}
lapply(1:4, function(ii) {
  rep(a[ii], b[ii])
})
~~~



~~~{.output}
[[1]]
[1] 1 1 1 1

[[2]]
[1] 2 2 2

[[3]]
[1] 3 3

[[4]]
[1] 4

~~~

### tapply

The `tapply` function allows you to run a function on different groups within
a vector. Going back to our first example of the lesson, we can use `tapply` to
calculate the total population for each continent in 2007:


~~~{.r}
gap2007 <- gap[year == 2007] # first filter by the year
tapply(
  gap2007[,pop], # The column of population counts from the data frame
  gap2007[,continent], # The column of continent labels for each entry
  sum # The function to apply to the population vector within each continent
)
~~~



~~~{.output}
    Africa   Americas       Asia     Europe    Oceania 
 929539692  898871184 3811953827  586098529   24549947 

~~~






