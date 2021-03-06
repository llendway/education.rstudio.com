---
title: Presentation-Ready Summary Tables with gtsummary
author:
  - '[Daniel Sjoberg](http://www.danieldsjoberg.com/)'
  - '[Margie Hannum](https://margarethannum.github.io/about.html)'
  - '[Karissa Whiting](https://github.com/karissawhiting)'
date: '2020-07-20'
categories:
  - learn
  - package
tags:
  - rmarkdown
description: |
  The gtsummary package is for making beautiful summary tables with R, in R Markdown documents. 
slug: gtsummary
photo:
  url: https://unsplash.com/photos/hk2gSD_S5j0
  author: Daniel von Appen
---



We are thrilled to introduce you to the [gtsummary package](http://www.danieldsjoberg.com/gtsummary/)!<a href='https://github.com/ddsjoberg/gtsummary'><img src='logo.png' align="right" height="200" /></a>

The gtsummary package provides an elegant and flexible way to create publication-ready analytical and summary tables in R.

  The motivation behind the package stems from our work as statisticians, where every day we summarize datasets and regression models in R, share these results with collaborators, and eventually include them in published manuscripts. Many of our colleagues had our own scripts to create the tables we needed, and even then would often need to modify the formatting in a document editor later, which did not lead to **reproducible results**.

  At the time we created the package, we had several ideas in mind for our ideal table summary package. We also wanted our tables to be able to take advantage of all the features in RStudio's newly released [gt](https://gt.rstudio.com/) package, which offers a variety of table customization options like spanning column headers, table footnotes, stubhead label, row group labels and more. So, gtsummary was born! 
  
  Here's what you can do with gtsummary:

- [**Summarize data frames or tibbles**](http://www.danieldsjoberg.com/gtsummary/articles/tbl_summary.html) to present descriptive statistics, compare group demographics (e.g creating a Table 1 for medical journals), and more!
- [**Summarize regression models**](http://www.danieldsjoberg.com/gtsummary/articles/tbl_regression.html). Using `broom::tidy()` in the background, gtsummary plays nicely with many model types (lm, glm, coxph, glmer etc.). You may also use custom functions to summarize regression models that do not currently have [broom](https://broom.tidymodels.org/index.html) tidiers.
- [**Customize gtsummary tables**](http://www.danieldsjoberg.com/gtsummary/reference/index.html#section-general-formatting-styling-functions) using a growing list of formatting/styling functions: everything from which statistics and tests to use to how many decimal places to round to, bolding labels, indenting categories and more!
- [**Report statistics inline**](http://www.danieldsjoberg.com/gtsummary/articles/tbl_summary.html#inline_text) from summary tables and regression summary tables in **R markdown**. Make your reports completely reproducible! 
- [**Leverage compatibility with multiple R Markdown outputs**](http://www.danieldsjoberg.com/gtsummary/articles/rmarkdown.html) to create beautiful, reproducible reports in a variety of formats (HTML, PDF, Word, RTF)


Install gtsummary from CRAN with the following code: 


```r
install.packages("gtsummary")  
```

## Summarize descriptive statistics {#stats}

Throughout the post we will use an example dataset of 200 subjects treated with either Drug A or Drug B, with a mix of categorical, dichotomous, and continuous demographic and response data. The dataset has label attributes (using the [labelled](http://larmarange.github.io/labelled/) package) for column names. 


```r
sm_trial <- trial %>% select(trt, age, response, grade)

head(sm_trial)
#> # A tibble: 6 x 4
#>   trt      age response grade
#>   <chr>  <dbl>    <int> <fct>
#> 1 Drug A    23        0 II   
#> 2 Drug B     9        1 I    
#> 3 Drug A    31        0 II   
#> 4 Drug A    NA        1 III  
#> 5 Drug A    51        1 III  
#> 6 Drug B    39        0 I
```


In **one line of code** we can summarize the overall demographics of the dataset!

Notice some nice default behaviors:  
:purple_heart: Detects variable types of input data and calculates descriptive statistics  
:purple_heart: Variables coded as 0/1, TRUE/FALSE, and Yes/No are presented dichotomously  
:purple_heart: Recognizes `NA` values as "missing" and lists them as unknown  
:purple_heart: Label attributes automatically printed  
:purple_heart: Variable levels indented and footnotes added 


```r
tbl_summary_1 <- tbl_summary(sm_trial)
```

<p align="center"><img src="tbl_summary_1.png" width=30%></p>



**Start customizing by adding arguments and functions**

Next you can start to customize the table by using arguments of the `tbl_summary()` function, as well as pipe the table through additional gtsummary functions to add more information, like p-value to compare across groups and overall demographic column.  


```r
tbl_summary_2 <- sm_trial %>%
  tbl_summary(by = trt) %>%
  add_p() %>%
  add_overall() %>% 
  bold_labels()
```

<p align="center"><img src="tbl_summary_2.png" width=60%></p>

**Customize further using formula syntax and tidy selectors**

Most arguments to `tbl_summary()` and `tbl_regression()` require formula syntax:

> select variables ~ specify what you want to do

* To select, use quoted or unquoted variables, or minus sign to negate (e.g. `age` or `"age"` to select, `-age` to deselect)
* Or use any {tidyselect} functions, e.g. `contains("stage") ~ ...`, including type selectors 
* To specify what you want to do, some arguments use [{glue}](https://github.com/tidyverse/glue) syntax where whatever is in the curly brackets gets evaluated and passed directly into the string. e.g `statistic = ... ~ "{mean} ({sd})"`


```r
tbl_summary_3 <- sm_trial %>%
  tbl_summary(
    by = trt,
    statistic = list(
      all_continuous() ~ "{mean} ({sd})",
      all_categorical() ~ "{n} / {N} ({p}%)"), 
    label = age ~ "Patient Age") %>%
  add_p(test = all_continuous() ~ "t.test",
        pvalue_fun = function(x) style_pvalue(x, digits = 2))
```

<p align="center"><img src="tbl_summary_3.png" width=50%></p>



## Summarize regression models {#models}

First, create a logistic regression model to use in examples. 


```r
m1 <- glm(response ~ trt + grade + age, 
          data = trial,
          family = binomial) 
```


`tbl_regression()` accepts regression model object as input. Uses {broom} in the background, outputs table with nice defaults:

  :purple_heart: Reference groups added to the table  
  :purple_heart: Sensible default number rounding and formatting  
  :purple_heart: Label attributes printed  
  :purple_heart: Common model types detected and appropriate header added with footnote
  

```r
tbl_reg_1 <- tbl_regression(m1, exponentiate = TRUE)
```


<p align="center"><img src="tbl_regression_1.png" width=40%></p>

We have a growing list of [vetted models](http://www.danieldsjoberg.com/gtsummary/reference/vetted_models.html) that can be passed to `tbl_regression()`. You may also pass a [custom tidier](http://www.danieldsjoberg.com/gtsummary/reference/vetted_models.html#custom-tidiers) for model types that are not yet officially supported!

## Join two or more tables {#join}

Oftentimes we must present results for multiple outcomes of interest, and there are many other reasons you might want to join two summary tables together. We've got you covered!

In this example we can use `tbl_merge()` to merge two gtsummary objects side-by-side. There is also a `tbl_stack()` function to place tables on top of each other. 


```r
library(survival)

tbl_reg_3 <- 
  coxph(Surv(ttdeath, death) ~ trt + grade + age, 
        data = trial) %>%
  tbl_regression(exponentiate = TRUE)

tbl_reg_4 <-
  tbl_merge(
    tbls = list(tbl_reg_1, tbl_reg_3), 
    tab_spanner = c("**Tumor Response**", "**Time to Death**") 
  ) 
```


<p align="center"><img src="tbl_regression_4.png" width=60%></p>

## Report results inline {#inline}

Tables are important, but we often need to report results in-line in a report. Any statistic reported in a gtsummary table can be extracted and reported in-line in a R Markdown document with the `inline_text()` function.

`inline_text(tbl_reg_1, variable = trt, level = "Drug B")`

1.13 (95% CI 0.60, 2.13; p=0.7)

- The pattern of what is reported can be modified with the `pattern = ` argument.  

- Default is `pattern = "{estimate} ({conf.level*100}% CI {conf.low}, {conf.high}; {p.value})"`.

## gtsummary + R Markdown {#rmd}

The **gtsummary** package was written to be a companion to the **gt** package from RStudio.
But not all output types are supported by the **gt** package (yet!).
Therefore, we have made it possible to print **gtsummary** tables with various engines.

Review the **[gtsummary + R Markdown](http://www.danieldsjoberg.com/gtsummary/articles/rmarkdown.html)** vignette for details.

<a href="http://www.danieldsjoberg.com/gtsummary/articles/rmarkdown.html">
<img src="gt_output_formats.PNG" width="55%" style="display: block; margin: auto;" />
</a>

## Using external customization functions {#custom}

It's natural a gtsummary package user would want to customize the aesthetics of the table with some of the many functions available in the print engines listed above. 

Once you convert a gtsummary object to another kind of object (e.g. gt), every function compatible that object will be available to use! 

Example workflow and code using gt customization:

1. Create a gtsummary table.  
1. Convert the table to a gt object with the [`as_gt()`](http://www.danieldsjoberg.com/gtsummary/reference/as_gt.html) function. (Also available: [`as_flextable()`](http://www.danieldsjoberg.com/gtsummary/reference/as_flextable.html), [`as_kable_extra()`](http://www.danieldsjoberg.com/gtsummary/reference/as_kable_extra.html)...  
1. Continue formatting as a gt table with any [gt](https://gt.rstudio.com/) function. 
    

```r
tbl_summary_5 <- sm_trial %>%
  # create a gtsummary table
  tbl_summary(by = trt) %>%
  # convert from gtsummary object to gt object
  as_gt() %>%
  # modify with gt functions
  gt::tab_header("Table 1: Baseline Characteristics") %>% 
  gt::tab_spanner(
    label = "Randomization Group",  
    columns = starts_with("stat_")
  ) %>% 
  gt::tab_options(
    table.font.size = "small",
    data_row.padding = gt::px(1)) 
```


<p align="center"><img src="tbl_summary_5.png" width=50%></p>



## Additional features {#more}

There are a few other functions we'd like you to know about! 

- [`tbl_uvregression()`](http://www.danieldsjoberg.com/gtsummary/reference/tbl_uvregression.html) to make tables summarizing univariable regression models (input a dataset, specify model and parameters... make a UVA regression table easily!)
- [`tbl_survfit()`](http://www.danieldsjoberg.com/gtsummary/reference/tbl_survfit.html) function to summarize survival models
- [`tbl_cross()`](http://www.danieldsjoberg.com/gtsummary/reference/tbl_cross.html) to make beautiful cross tables
- Use [themes](http://www.danieldsjoberg.com/gtsummary/dev/articles/themes.html) to format your table for specific journal/aesthetic requirements! 

**Additional customization options**

See the full list of gtsummary functions [here](http://www.danieldsjoberg.com/gtsummary/reference/index.html). You can use them to do all sorts of things to your tables, like:

- Use **custom functions** for calculating p-values and reporting any statistic for continuous variables (including user-written functions)
- Bold and italicize labels and levels
- Present **missing data** in various ways
- **Sort variables** by significance (`sort_p()`); sort categorical variables by frequency  
- Calculate **cell percents and row percents** (default is column-wide)  
- Report p-values for select variables (`add_p(include = ...)`); report q-values (like false discovery rate)  
- Set global **rounding options** with themes (for estimates, confidence intervals, and p-values)

There is a growing [gallery of tables](http://www.danieldsjoberg.com/gtsummary/articles/gallery.html) which highlights some of the many customization options! 

## Wrap-up

The gtsummary package website contains [well-documented functions](http://www.danieldsjoberg.com/gtsummary/reference/index.html), detailed [tutorials](http://www.danieldsjoberg.com/gtsummary/articles/), and [examples](http://www.danieldsjoberg.com/gtsummary/articles/gallery.html)! 

If you have any questions on usage, please post to StackOverflow and use the [gtsummary tag](https://stackoverflow.com/questions/tagged/gtsummary?tab=Newest). We try to answer questions ASAP! You can also report bugs or make feature requests by submitting an issue on [GitHub](https://github.com/ddsjoberg/gtsummary/issues). 

May your code be short, your tables beautiful, and your reports fully reproducible! 


![](https://media.giphy.com/media/13AcmSNW5O7WV2/giphy.gif)


## Acknowledgements

A big thank you to all gtsummary contributors:
[&#x0040;ablack3](https://github.com/ablack3), [&#x0040;ahinton-mmc](https://github.com/ahinton-mmc), [&#x0040;barthelmes](https://github.com/barthelmes), [&#x0040;calebasaraba](https://github.com/calebasaraba), [&#x0040;CodieMonster](https://github.com/CodieMonster), [&#x0040;davidgohel](https://github.com/davidgohel), [&#x0040;davidkane9](https://github.com/davidkane9), [&#x0040;dax44](https://github.com/dax44), [&#x0040;ddsjoberg](https://github.com/ddsjoberg), [&#x0040;DeFilippis](https://github.com/DeFilippis), [&#x0040;emilyvertosick](https://github.com/emilyvertosick), [&#x0040;gorkang](https://github.com/gorkang), [&#x0040;GuiMarthe](https://github.com/GuiMarthe), [&#x0040;hughjonesd](https://github.com/hughjonesd), [&#x0040;jalavery](https://github.com/jalavery), [&#x0040;jeanmanguy](https://github.com/jeanmanguy), [&#x0040;jemus42](https://github.com/jemus42), [&#x0040;jennybc](https://github.com/jennybc), [&#x0040;JesseRop](https://github.com/JesseRop), [&#x0040;jflynn264](https://github.com/jflynn264), [&#x0040;joelgautschi](https://github.com/joelgautschi), [&#x0040;jwilliman](https://github.com/jwilliman), [&#x0040;karissawhiting](https://github.com/karissawhiting), [&#x0040;khizzr](https://github.com/khizzr), [&#x0040;larmarange](https://github.com/larmarange), [&#x0040;leejasme](https://github.com/leejasme), [&#x0040;ltin1214](https://github.com/ltin1214), [&#x0040;margarethannum](https://github.com/margarethannum), [&#x0040;matthieu-faron](https://github.com/matthieu-faron), [&#x0040;michaelcurry1123](https://github.com/michaelcurry1123), [&#x0040;moleps](https://github.com/moleps), [&#x0040;MyKo101](https://github.com/MyKo101), [&#x0040;oranwutang](https://github.com/oranwutang), [&#x0040;proshano](https://github.com/proshano), [&#x0040;ryzhu75](https://github.com/ryzhu75), [&#x0040;sammo3182](https://github.com/sammo3182), [&#x0040;sbalci](https://github.com/sbalci), [&#x0040;simonpcouch](https://github.com/simonpcouch), [&#x0040;slb2240](https://github.com/slb2240), [&#x0040;slobaugh](https://github.com/slobaugh), [&#x0040;tormodb](https://github.com/tormodb), [&#x0040;UAB-BST-680](https://github.com/UAB-BST-680), [&#x0040;zabore](https://github.com/zabore), and [&#x0040;zeyunlu](https://github.com/zeyunlu)
