---
layout: page
title: Intermediate R for reproducible scientific analysis
subtitle: reshape2
minutes: 20
---


> ## Learning objectives {.objectives}
>
> * To know how split a column by a group to create multiple columns
> * To know how to combine multiple columns into one with different groupings.
>

So far we've been working with the gapminder dataset:


```r
library("data.table")
gap <- fread("data/gapminder-FiveYearData.csv")
gap
```

```
##           country year      pop continent lifeExp gdpPercap
##    1: Afghanistan 1952  8425333      Asia  28.801  779.4453
##    2: Afghanistan 1957  9240934      Asia  30.332  820.8530
##    3: Afghanistan 1962 10267083      Asia  31.997  853.1007
##    4: Afghanistan 1967 11537966      Asia  34.020  836.1971
##    5: Afghanistan 1972 13079460      Asia  36.088  739.9811
##   ---                                                      
## 1700:    Zimbabwe 1987  9216418    Africa  62.351  706.1573
## 1701:    Zimbabwe 1992 10704340    Africa  60.377  693.4208
## 1702:    Zimbabwe 1997 11404948    Africa  46.809  792.4500
## 1703:    Zimbabwe 2002 11926563    Africa  39.989  672.0386
## 1704:    Zimbabwe 2007 12311143    Africa  43.487  469.7093
```

The format that this dataset has been provided in is called a "long" format:
each variable has its own column, and there are several columns with identifying
information: where those observations came from/groupings in the data.

This is a format that is convenient for rapid analysis using R: it is easy to
manipulate, filter and query using `data.table`, and is the format `ggplot2`
expects for plotting.

However, data often does not come in this format. We might sometimes find we are
given data in what is known as a "wide" format: in which variables may be split
over multiple columns, one for each group. Let's read in an alternative version
of the gapminder dataset:


```r
# This is an example where `fread` doesn't work: it loses the column names!
gapWide <- as.data.table(read.csv("data/gapminder-wide-format.csv", header=TRUE))
gapWide
```

```
##      continent        country pop.1952 pop.1957 pop.1962 pop.1967 pop.1972
##   1:    Africa        Algeria  9279525 10270856 11000948 12760499 14760787
##   2:    Africa         Angola  4232095  4561361  4826015  5247469  5894858
##   3:    Africa          Benin  1738315  1925173  2151895  2427334  2761407
##   4:    Africa       Botswana   442308   474639   512764   553541   619351
##   5:    Africa   Burkina Faso  4469979  4713416  4919632  5127935  5433886
##  ---                                                                      
## 138:    Europe    Switzerland  4815000  5126000  5666000  6063000  6401400
## 139:    Europe         Turkey 22235677 25670939 29788695 33411317 37492953
## 140:    Europe United Kingdom 50430000 51430000 53292000 54959000 56079000
## 141:   Oceania      Australia  8691212  9712569 10794968 11872264 13177000
## 142:   Oceania    New Zealand  1994794  2229407  2488550  2728150  2929100
##      pop.1977 pop.1982 pop.1987 pop.1992 pop.1997 pop.2002 pop.2007
##   1: 17152804 20033753 23254956 26298373 29072015 31287142 33333216
##   2:  6162675  7016384  7874230  8735988  9875024 10866106 12420476
##   3:  3168267  3641603  4243788  4981671  6066080  7026113  8078314
##   4:   781472   970347  1151184  1342614  1536536  1630347  1639131
##   5:  5889574  6634596  7586551  8878303 10352843 12251209 14326203
##  ---                                                               
## 138:  6316424  6468126  6649942  6995447  7193761  7361757  7554661
## 139: 42404033 47328791 52881328 58179144 63047647 67308928 71158647
## 140: 56179000 56339704 56981620 57866349 58808266 59912431 60776238
## 141: 14074100 15184200 16257249 17481977 18565243 19546792 20434176
## 142:  3164900  3210650  3317166  3437674  3676187  3908037  4115771
##      lifeExp.1952 lifeExp.1957 lifeExp.1962 lifeExp.1967 lifeExp.1972
##   1:       43.077       45.685       48.303       51.407       54.518
##   2:       30.015       31.999       34.000       35.985       37.928
##   3:       38.223       40.358       42.618       44.885       47.014
##   4:       47.622       49.618       51.520       53.298       56.024
##   5:       31.975       34.906       37.814       40.697       43.591
##  ---                                                                 
## 138:       69.620       70.560       71.320       72.770       73.780
## 139:       43.585       48.079       52.098       54.336       57.005
## 140:       69.180       70.420       70.760       71.360       72.010
## 141:       69.120       70.330       70.930       71.100       71.930
## 142:       69.390       70.260       71.240       71.520       71.890
##      lifeExp.1977 lifeExp.1982 lifeExp.1987 lifeExp.1992 lifeExp.1997
##   1:       58.014       61.368       65.799       67.744       69.152
##   2:       39.483       39.942       39.906       40.647       40.963
##   3:       49.190       50.904       52.337       53.919       54.777
##   4:       59.319       61.484       63.622       62.745       52.556
##   5:       46.137       48.122       49.557       50.260       50.324
##  ---                                                                 
## 138:       75.390       76.210       77.410       78.030       79.370
## 139:       59.507       61.036       63.108       66.146       68.835
## 140:       72.760       74.040       75.007       76.420       77.218
## 141:       73.490       74.740       76.320       77.560       78.830
## 142:       72.220       73.840       74.320       76.330       77.550
##      lifeExp.2002 lifeExp.2007 gdpPercap.1952 gdpPercap.1957
##   1:       70.994       72.301      2449.0082      3013.9760
##   2:       41.003       42.731      3520.6103      3827.9405
##   3:       54.406       56.728      1062.7522       959.6011
##   4:       46.634       50.728       851.2411       918.2325
##   5:       50.650       52.295       543.2552       617.1835
##  ---                                                        
## 138:       80.620       81.701     14734.2327     17909.4897
## 139:       70.845       71.777      1969.1010      2218.7543
## 140:       78.471       79.425      9979.5085     11283.1779
## 141:       80.370       81.235     10039.5956     10949.6496
## 142:       79.110       80.204     10556.5757     12247.3953
##      gdpPercap.1962 gdpPercap.1967 gdpPercap.1972 gdpPercap.1977
##   1:      2550.8169      3246.9918       4182.664       4910.417
##   2:      4269.2767      5522.7764       5473.288       3008.647
##   3:       949.4991      1035.8314       1085.797       1029.161
##   4:       983.6540      1214.7093       2263.611       3214.858
##   5:       722.5120       794.8266        854.736        743.387
##  ---                                                            
## 138:     20431.0927     22966.1443      27195.113      26982.291
## 139:      2322.8699      2826.3564       3450.696       4269.122
## 140:     12477.1771     14142.8509      15895.116      17428.748
## 141:     12217.2269     14526.1246      16788.629      18334.198
## 142:     13175.6780     14463.9189      16046.037      16233.718
##      gdpPercap.1982 gdpPercap.1987 gdpPercap.1992 gdpPercap.1997
##   1:      5745.1602      5681.3585      5023.2166       4797.295
##   2:      2756.9537      2430.2083      2627.8457       2277.141
##   3:      1277.8976      1225.8560      1191.2077       1232.975
##   4:      4551.1421      6205.8839      7954.1116       8647.142
##   5:       807.1986       912.0631       931.7528        946.295
##  ---                                                            
## 138:     28397.7151     30281.7046     31871.5303      32135.323
## 139:      4241.3563      5089.0437      5678.3483       6601.430
## 140:     18232.4245     21664.7877     22705.0925      26074.531
## 141:     19477.0093     21888.8890     23424.7668      26997.937
## 142:     17632.4104     19007.1913     18363.3249      21050.414
##      gdpPercap.2002 gdpPercap.2007
##   1:       5288.040       6223.367
##   2:       2773.287       4797.231
##   3:       1372.878       1441.285
##   4:      11003.605      12569.852
##   5:       1037.645       1217.033
##  ---                              
## 138:      34480.958      37506.419
## 139:       6508.086       8458.276
## 140:      29478.999      33203.261
## 141:      30687.755      34435.367
## 142:      23189.801      25185.009
```

This contains exactly the same data, but the three variables, "gdpPercap", "pop",
and "lifeExp" have one column for each year of collection. This format is useful
if you wish to calculate group-wise properties, e.g. the correlation structure
between groups. It is also a more natural way for data collectors to structure
data in another application, such as Excel.

Being able to transform between "wide" and "long" formats is an incredible useful
skill that will save you a lot of time in the future. We can do this using the
`reshape2` package.

### Combining columns

To collapse multiple columns into one we use the `melt` function:


```r
library(reshape2)
```

```
## 
## Attaching package: 'reshape2'
## 
## The following objects are masked from 'package:data.table':
## 
##     dcast, melt
```

```r
gapLong <- melt(
  data=gapWide,
  id.vars=c("continent", "country") # All other columns will be collapsed into one
)
```

```
## Warning in melt.data.table(data = gapWide, id.vars = c("continent",
## "country")): 'measure.vars' [pop.1952, pop.1957, pop.1962, pop.1967, pop.
## 1972, pop.1977, pop.1982, pop.1987, pop.1992, pop.1997, pop.2002, pop.
## 2007, lifeExp.1952, lifeExp.1957, lifeExp.1962, lifeExp.1967, lifeExp.
## 1972, lifeExp.1977, lifeExp.1982, lifeExp.1987, lifeExp.1992, lifeExp.1997,
## lifeExp.2002, lifeExp.2007, gdpPercap.1952, gdpPercap.1957, gdpPercap.1962,
## gdpPercap.1967, gdpPercap.1972, gdpPercap.1977, gdpPercap.1982, gdpPercap.
## 1987, gdpPercap.1992, gdpPercap.1997, gdpPercap.2002, gdpPercap.2007] are
## not all of the same type. By order of hierarchy, the molten data value
## column will be of type 'double'. All measure variables not of type 'double'
## will be coerced to. Check DETAILS in ?melt.data.table for more on coercion.
```

We get a warning because the 'pop' columns are of type 'integer' (i.e. whole 
numbers) while the 'gdpPercap' and 'lifeExp' columns are type 'double' (i.e. 
decimal values).

This melt has collapsed the table too far. We need to separate out the different
variable types into their own columns. First, lets split the variable column into
variable type and year:


```r
gapLong[, c("var", "year") := colsplit(variable, "\\.", c("var", "year"))]
```

Let's break this down.

First, `colsplit(variable, "\\.", c("var", "year"))` creates two columns called 
"var" and "year" from the "variable" column, splitting each value on the ".".
We need to specific the pattern as `\\.` because `.` by itself is a wild card
character: it would match every character in the string.

Next, we create two new columns in the `gapLong` data table using the `:=` 
operator. To create multiple columns, we specify their names in a vector to the
left of the `:=` operator.

Now that we're happy this has worked, we can delete the old `variable` column:


```r
gapLong[,variable := NULL]
```

Finally, to split out the `value` column into the groups stored in `var`, we use
the `dcast` function. Since we'd like to keep the result as a data table, we'll
explicitly call the method from the `data.table` package:


```r
gapLong <- dcast.data.table(
  data=gapLong,  
  # unique identifier columns go to the left of the '~', separated by '+' signs.
  # The grouping column goes to the right of the '~'.
  formula=continent+country+year~var,
  # which column stores the values to be spread over the new columns
  value.var="value" 
)
```

And now we're back where we've started with the long data!

We've shown you the simplest cases for reshaping data. More complex cases may
require you to `melt` or `dcast` in several steps, and `rbind` the results.

Other packages you may find useful for this task are 
[tidyr](http://cran.r-project.org/web/packages/tidyr/index.html) and 
[splitstackshape](http://cran.r-project.org/web/packages/splitstackshape/index.html)
