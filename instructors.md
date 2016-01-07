---
layout: page
title: Lesson Title
subtitle: Instructors Guide
---

## Preface

The goal of this lesson is to teach researchers already experienced with R 
some useful programming concepts that will make writing code more efficient
(e.g. functions, loops, etc.), as well as more advanced data analysis packages
(that I find useful in my day to day work) that they are unlikely to have 
encountered before. 

The programming concepts taught here are identical to those in the novice lessons:
task automation using for loops, control flow, and functions. We found that
at the University of Melbourne, and in the life sciences in particular, there
were many researchers who use R but had never been taught programming. This
meant that although the novice material was way too slow, they were still missing
the fundamentals of scientific computing.

This lesson is a first step towards addressing this niche. It has only been taught
once, and the lessons written down after they were taught.

## Pacing

We moved through a lot of this material very quickly. The group was extremely
switched on, all came from the same lab, and experienced programmers were paired
with less experienced students. The focus was to expose the group to concepts
they wouldnt otherwise be familiar with so they had a starting point to learn 
from later.

We broke for challenges only for the key programming concepts (i.e. loops, 
functions, control flow). Attendees were generally happy to follow along 
without challenges for the other topics, so we were able to cover a lot of
material. exposing attendees to concepts and packages they might find useful
later.

## Lesson data

This lesson is based around the [gapminder](http://www.gapminder.org/) dataset. 
It contains interesting data about countries around the world: their gdp, population,
and life expectancy, over a 50 year interval. It is provided as a single csv file,
which can be read in as a single `data.frame`.

## Lesson topics and packages

The majority of this lesson is based around efficient manipulation of data, once
you have loaded into R. The first lesson is on the `data.table` package, and the
syntax used to access rows, columns, and subset `data.table` objects is used
throughout the lesson. 

Next, we introduce the idea of "dirty" data: that often you will be given data
that isn't in the ideal format for analysis. We teach the `reshape2` package
for converting `data.frames` and `data.tables` from wide to long format and 
back again.

We then move on to basic programming tools for task automation: covering for
loops, then the `apply` family of functions, because most people we talk to
have heard of them, seen them, but are thoroughly confused by them. Then we
move on to control flow (`if`, `else`). Note that although the control flow
lessons contain sections on `for` and `while` loops, we covered these before
`apply`. In general I would suggest leaving control flow until after the 
functions lesson, because they are much more naturally useful inside functions.

After slogging through several programming heavy topics, we break up the lesson
with a short demo of R markdown and let attendees play for 20 minutes. Next we
show `plyr` as a way of solving "split-apply-combine" problems before moving on
to a whirlwind tour of parallel computing and the `foreach` package. 
`foreach` is useful even when not dispatched over multiple cores because it 
allows you to flexibly combine the output of the loop into one data structure.
Finally we end the lessons with functions.

Plotting was omitted from these lessons as the group had one of their own 
come in for an additional half day of lessons, and we were told they would
mostly be covering plotting there. They ended up covering `magrittr` and 
package development instead. 



