---
layout: page
title: R for reproducible scientific analysis
subtitle: Control flow
minutes: 35
---



> ## Learning objectives {.objectives}
>
> * Write conditional statements with `if` and `else`.
> * Write and understand `for` loops.
>

Often when we're coding we want to control the flow of our actions. This can be done
by setting actions to occur only if a condition or a set of conditions are met.
Alternatively, we can also set an action to occur a particular number of times.

There are several ways you can control flow in R.
For conditional statements, the most commonly used approaches are the constructs:


~~~{.r}
# if
if (condition is true) {
  perform action
}

# if ... else
if (condition is true) {
  perform action
} else {  # that is, if the condition is false,
  perform alternative action
}
~~~

Say, for example, that we want R to print a message if a variable `x` has a particular value:


~~~{.r}
# sample a random number from a Poisson distribution
# with a mean (lambda) of 8

x <- rpois(1, lambda=8)

if (x >= 10) {
  print("x is greater than or equal to 10")
}

x
~~~



~~~{.output}
[1] 8

~~~

Note you may not get the same output as your neighbour because
you may be sampling different random numbers from the same distribution.

Let's set a seed so that we all generate the same 'pseudo-random'
number, and then print more information:


~~~{.r}
x <- rpois(1, lambda=8)

if (x >= 10) {
  print("x is greater than or equal to 10")
} else if (x > 5) {
  print("x is greater than 5")
} else {
  print("x is less than 5")
}
~~~



~~~{.output}
[1] "x is greater than 5"

~~~

> #### Tip: pseudo-random numbers {.callout}
>
> In the above case, the function `rpois` generates a random number following a
> Poisson distribution with a mean (i.e. lambda) of 8. The function `set.seed`
> guarantees that all machines will generate the exact same 'pseudo-random'
> number ([more about pseudo-random numbers](http://en.wikibooks.org/wiki/R_Programming/Random_Number_Generation)).
> So if we `set.seed(10)`, we see that `x` takes the value 8. You should get the
> exact same number.
>

**Important:** when R evaluates the condition inside `if` statements, it is
looking for a logical element, i.e., `TRUE` or `FALSE`. This can cause some
headaches for beginners. For example:


~~~{.r}
x  <-  4 == 3
if (x) {
  "4 equals 3"
}
~~~

As we can see, the message was not printed because the vector x is `FALSE`


~~~{.r}
x <- 4 == 3
x
~~~



~~~{.output}
[1] FALSE

~~~

> #### Challenge 1 {.challenge}
>
> Use an `if` statement to print a suitable message
> reporting whether there are any records from 2002 in
> the `gapminder` dataset.
> Now do the same for 2012.
>

Did anyone get a warning message like this?


~~~{.output}
Warning in if (gapminder$year == 2012) {: the condition has length > 1 and
only the first element will be used

~~~

If your condition evaluates to a vector with more than one logical element,
the function `if` will still run, but will only evaluate the condition in the first
element. Here you need to make sure your condition is of length 1.

> #### Tip: `any` and `all` {.callout}
> The `any` function will return TRUE if at least one
> TRUE value is found within a vector, otherwise it will return `FALSE`.
> This can be used in a similar way to the `%in%` operator.
> The function `all`, as the name suggests, will only return `TRUE` if all values in
> the vector are `TRUE`.
>

> #### Challenge 2 {.challenge}
>
> Compare the objects output_vector and
> output_vector2. Are they the same? If not, why not?
> How would you change the last block of code to make output_vector2
> the same as output_vector?
>
