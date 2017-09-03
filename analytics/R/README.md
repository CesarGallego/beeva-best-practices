# R for Data Science
![alt text](static/r-logo.png "R")

## Index
  * [Introduction](#introduction)
  * [General Information](#general-information)
  * [Load data](#load-data) 
  * [Manipulate data](#manipulate-data)
    * [tidyr](#tidyr)
    * [dplyr](#dplyr)
    * [data.table](#data.table)
    * [foreach](#foreach)
  * [Visualize](#visualize)
    * [ggplot2](#ggplot2)
    * [shiny](#shiny)
  * [Model](#model)
    * [caret](#caret)
    * [mclust](#mclust)
  * [Deploy services](#deploy-services)
  * [Big data](#big-data)



## INTRODUCTION

This document is a brief summary of the knowledge acquired using R for data science. It may help to avoid some common mistakes and dead ends and cast some light on the tools, idioms and quirks of the language.

There are thousands of packages available for R, but in this guide we will mention some of the most used for different purposes.

## GENERAL INFORMATION

### What is R

*R* is a scripting language designed to make data analysis and statistics easier. It is free, open source and available on every major platform making it easily replicable.

### What is R good for

*R* has a great community support and a massive set of packages for statistical modeling, machine learning, visualization and data manipulation. The data mungling capabilities are second to no one, and whatever model you are trying to build, it is very likely that a package for it.

### What is not R good for

*R* loads everything in memory. That is a powerful feature for iterative processes but also imposes a strong limit on your data size. Furthermore, some R functions do not modify objects themselves but copies instead requiring even more memory.

It is also an interpreted language, which makes it [quite slow](#http://adv-r.had.co.nz/Performance.html). You should use as many packages and functions as you can instead of building a function from the scratch, since most of them are interfaces to a much more optimized versions programmed in C, Fortran or C++. Some functions written in pure R can take 45 minutes and only 0.15s when using the proper package.

## LOAD DATA

### SQL

[RSQLite](https://cran.r-project.org/web/packages/RSQLite/index.html), [RODBC](https://cran.r-project.org/web/packages/RODBC/index.html),[RMySQL](https://cran.r-project.org/web/packages/RMySQL/index.html),[RPostgresSQL](https://cran.r-project.org/web/packages/RPostgreSQL/index.html),[RJDBC](https://cran.r-project.org/web/packages/RJDBC/index.html) are great packages to connect R to your database. Choose the package that fits your database. All of them use a package called *rJava* which is a simple R-to-Java interface. You should keep this in mind when retrieving data from a database: if your java heap size is too low, the function will fail. You can increase the java heap size for the session with:

```
    options( java.parameters = "-Xmx4g")
```

### Excel

[XLConnect](https://cran.r-project.org/web/packages/XLConnect/index.html), [xlsx](https://cran.r-project.org/web/packages/xlsx/index.html) - These packages help you read and write Excel files from R.

### Fast reading

[data.table](https://cran.r-project.org/web/packages/data.table/index.html) ([*fread*](http://www.inside-r.org/packages/cran/data.table/docs/fread) function)and [readr](https://cran.r-project.org/web/packages/readr/) are two packages that provide function for fast tabular data reading (up to 10x-15x faster).


## MANIPULATE DATA

### tidyr:

2 data formats can be used :  *long format* and *wide format*.

Long format is something you usually obtain from queries to a database, with one or more rows for each observation depending on the different features, for example:

| subject | sex | condition | measurement |
| :-----: | :--: | :-----: | :------: |
| 1 | M  | control |        7.9 |
| 1 | M  |   cond1 |       12.3 |
| 1 | M  |   cond2 |       10.7 |
| 2 | F  | control |        6.3 |
| 2 | F  |   cond1 |       10.6 |
| 2 | F  |   cond2 |       11.1 |
| 3 | F  | control |        9.5 |
| 3 | F  |   cond1 |       13.1 |
| 3 | F  |   cond2 |       13.8 |
| 4 | M  | control |       11.5 |
| 4 | M  |   cond1 |       13.4 |
| 4 | M  |  cond2  |       12.9 |


Wide format, on the other hand has the following properties:

* Each row is an observation
* Each column is a variable

For example:


| subject | sex | control | cond1 | cond2 |
| :----: | :---: | :---: | :---: | :---: |
|       1 |  M  |   7.9 | 12.3 | 10.7 |
|       2 |  F  |   6.3 | 10.6 | 11.1 |
|       3 |  F  |   9.5 | 13.1 | 13.8 |
|       4 |  M  |  11.5 | 13.4 | 12.9 |


This one is also known as *tidy data*, and while is the preferred format, you will have to choose the most adequate between them depending on which function or algorithm you use. The package *tidyr* has two functions for that purpose:

* From wide to long:

```
	- gather(*input_data*, *variable_name*, *value*, *columns_to_convert*)
```

* From long to wide:

```
	- spread(*input_data*, *variable_column_name*, *value_column_name*)
```

[Source](http://www.cookbook-r.com/Manipulating_data/Converting_data_between_wide_and_long_format/)

### dplyr

*dplyr* is one of the best packages for data manipulation (also known as *data munging*). It provides an array of tools to model data with great efficiency. It also allows function chaining with pipes (*%>%*) and even supports connections to databases. You should use this functions instead of the base ones for efficiency reasons.


* filter() (and slice())
* arrange()
* select() (and rename())
* distinct()
* mutate() (and transmute())
* summarise()
* sample_n() and sample_frac()
* group_by()

A very usual example of this functions chained together is the equivalent of

```
	select id,avg(*)
	from data
	group by id
```

in *SQL*.

For example, given the following dataset: 

```
#>     year month   day dep_time dep_delay arr_time arr_delay carrier tailnum
#>    (int) (int) (int)    (int)     (dbl)    (int)     (dbl)   (chr)   (chr)
#> 1   2013     1     1      517         2      830        11      UA  N14228
#> 2   2013     1     1      533         4      850        20      UA  N24211
#> 3   2013     1     1      542         2      923        33      AA  N619AA
#> 4   2013     1     1      544        -1     1004       -18      B6  N804JB
#>...
```


You could obtain the average arrival delay and departure delay for each day:

```
flights %>%
  group_by(year, month, day) %>%
  select(arr_delay, dep_delay) %>%
  summarise(
    arr = mean(arr_delay, na.rm = TRUE),
    dep = mean(dep_delay, na.rm = TRUE)
  ) %>%
  filter(arr > 30 | dep > 30)

#> Source: local data frame [49 x 5]
#> Groups: year, month [11]
#> 
#>     year month   day      arr      dep
#>    (int) (int) (int)    (dbl)    (dbl)
#> 1   2013     1    16 34.24736 24.61287
#> 2   2013     1    31 32.60285 28.65836
#> 3   2013     2    11 36.29009 39.07360
#> 4   2013     2    27 31.25249 37.76327
#> ..   ...   ...   ...      ...      ...
```

dplyr is a really big package and it's extremely recommended to check it's documentation. The [introduction](ttps://cran.rstudio.com/web/packages/dplyr/vignettes/introduction.html) is a great starting point, and you can move onto the [windows functions](https://cran.rstudio.com/web/packages/dplyr/vignettes/window-functions.html)

### data.table

*data.table* is an extremely efficient version of data.frame, especially useful when working with very large datasets (from 1Gb to more than 100Gb in RAM). It is even more efficient than dplyr, specially when working with >100k groups.

The basic command of data.table is **DT[i, j, by]**. The command has 3 elements: *i*,*j* and *by*. A rough SQL equivalent of this would be:

	i =  WHERE
	j = SELECT
	by = GROUP BY

Using the example dataset from the *dplyr* section, the equivalent of 

	select UniqueCarrier,mean(AirTime)
	from data
	where Month = 10
	group by UniqueCarrier

Would be

	DT[Month==10,mean(na.omit(AirTime)),by=UniqueCarrier]

You can also create an index on a data.table by defining a key

	setkey(DT,*column*)

This will reorder the *DT* and will improve access time significantly. When subsetting in the form of *DT[x == .]* and *DT[x %in% .]*, *DT* is optimised to build and index automatically on the first run, so successive runs can make use of binary search fpr fast subsets instead of vector scans.

The [documentation](https://cran.r-project.org/web/packages/data.table/vignettes/datatable-intro.pdf) of *data.table* is extensive but very recommended.

#### dplyr VS data.table

On [this stackoverflow debate](http://stackoverflow.com/questions/21435339/data-table-vs-dplyr-can-one-do-something-well-the-other-cant-or-does-poorly/27840349#27840349) there is some debate about it, including [this](https://github.com/Rdatatable/data.table/wiki/Benchmarks-%3A-Grouping) benchmark where *data.table* is clearly the winner. On the other hand, *dplyr* has some capabilities that *data.table* lacks, such as SQL connections of a better position to improve parallel performance (but this is still on the air) and also has a more verbose but approachable syntax.

### foreach

There are some cases in which a *for-each* loop is inevitable. In other circumstances, parallel work would improve performance greatly. *foreach* is a package created by the open-souce team at Revolution Analytics to speed up the foreach based loops running the iterations in parallel. It needs another package that acts as backend for multicore usage, such as doParallel (linux) or doMC(windows). A simple example execution would be:

```
	library(foreach)
	library(doParallel)
	cl<-makeCluster(4)
	registerDoParallel(cl)
	foreach(i=1:4) %doPar% sqrt(i)
```


The foreach function allows a *combine* parameter which will process the task results as they are generated. Specifying 'c' is useful for concatenating results into a vector. The values 'rbind' and 'cbind' can combine vectors into a matrix, and values such as '+' or '*' can be used to process numerical data. By default the results are returned into a list. An important note about this is the parameter *multicombine* that allows to specify how many parameters *combine* will take. In some cases (like using *cbind*) performance will improve greatly if it's allowed to combine more than 2 elements each time.

There's something to keep in mind. Because the multicore mode starts its workers using fork without doing a subsequent exec, it has some limitations. Some operations cannot be performed properly by forked processes. For example, a connection object most likely won't work, which can corrupt the object thus forcing the *R* session to crash.

Finally, *foreach* allows to operate not only in a multicore scenario, but can be applied to a cluster using the *doSNOW* library.


## VISUALIZE

### ggplot2

*ggplot2* is a plotting system for *R*. It takes care of many of the fiddly details that make plotting a hassle (like drawing legends). It also provides a powerful graphic model that makes it easy to produce complex multi-layered graphics. The multi-layered part is important, since it allows to add new information to the graphic incrementally, instead of rendering it completely each time.

There are two major functions for *ggplot2*:

* *qplot()* for quick plots.
* *ggplot()* for granular control.

A couple examples using the *mtcars* default dataset:

```
	qplot(wt, mpg, data=mtcars, geom=c("point", "smooth"),
	      method="lm", formula=y~x, color=cyl,
	      main="Regression of MPG on Weight",
	      xlab="Weight", ylab="Miles per Gallon")
```

(converted strings into factors)

```
	qplot(mpg, data=mtcars, geom="density", fill=gear, alpha=I(.5),
	      main="Distribution of Gas Milage", xlab="Miles Per Gallon",
	      ylab="Density")
```


#### Faceting

Frequently, the same plot has to be performed for different data subsets. The *facets=* argument does exactly that:

```
	qplot(hp, mpg, data=mtcars, shape=am, color=am,
	      facets=gear~cyl, size=I(3),
	      xlab="Horsepower", ylab="Miles per Gallon") 
```


Since *ggplot2* works with layers, we can add incremental changes to a plot. We could start with:

```
	g<-ggplot(data=mtcars, aes(x=cyl, y=gear, fill=am)) +
	  geom_bar(stat="identity", position=position_dodge(), colour="black")
```


Then add axis labels, titles ... 

```
	g<-g+xlab("cylinders") + ylab("gears") +ggtitle("Gears and cylinders in mtcars dataset")
```


We could add lines or other elements, since they are treated as independent layers.

Finally, we can save them with *ggsave("filename.jpg")*. It will autodetect the saved filename extension.

The [documentation](http://docs.ggplot2.org/current/) of *ggplot* is really big bug useful.

### Shiny

[Shiny](https://shiny.rstudio.com) is a web application framework for *R*. It is designed for fully interactive visualizations. 

Shiny is intended to easily deploy web apps based on *R* analysis. No web development skills are required.

## MODEL

### caret

The *caret* package contains functions to streamline the model training process for complex regression and classification problems. While it uses other external *R* packages, they are loaded lazily (only when necessary).

#### train
* evaluate, using resampling, the effect of model tuning parameters on performance
* choose the "optimal" model across these parameters
* estimate model performance from a training set

An example from the documentation:

```
plsFit <- train(Class ~ .,
+                 data = training,
+                 method = "pls",
+                 ## Center and scale the predictors for the training
+                 ## set and all future samples.
+                 preProc = c("center", "scale"))
```

### mclust

The *mclust* package is a contributed *R* package for model-based clustering, classification, and density estimation. It assumes a variety of data models and apply maximum likelihood estimation and Bayes criteria to identify the most likely model and number of clusters.

## DEPLOY SERVICES

There are several options to deploy Machine Learning models on *R* as REST services. At the moment, the best option we found for this task is to use [Azure ML](https://docs.microsoft.com/en-us/azure/machine-learning/machine-learning-r-csharp-web-service-examples). 

However, for high demand scenarios, *Azure ML* services based on custom *R* models could provide not enough QoS, in terms of latency and concurrency compared, for instance, to services based on native *Azure ML* models. The latter with better performance but less flexibility than custom models.

More info on this topic:
- http://blog.algorithmia.com/deploying-r-models-production-web-services/
- http://stackoverflow.com/questions/22309236/options-for-deploying-r-models-in-production
- https://github.com/jpmml/r2pmml

## BIG DATA

*R* language is not oriented to (true) Big Data in general terms. Anyways there are specialized packages and alternatives as:
* [pbdR](http://pbdr.org): a set of *R* packages for large scale, distributed computing and profiling.
* [SparkR](http://spark.apache.org/docs/latest/sparkr.html): a light-weight frontend to use Apache Spark from *R*. Good article to read: [10 things I wish someone had told me before I started using Apache SparkR](https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/8599738367597028/1792412399382575/3601578643761083/latest.html).

Moreover, as [previously said](#general-information), efficiency is not precisely the strongest feature in *R*. However, the work from [Revolution Analytics](https://en.wikipedia.org/wiki/Revolution_Analytics) to get a multithreaded version of R has been integrated into [Microsoft R Open](https://mran.microsoft.com/documents/rro/multithread).

___

[BEEVA](https://www.beeva.com) | Technology and innovative solutions for companies
