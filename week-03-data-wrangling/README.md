
<!-- Please edit README.Rmd - not README.md -->

# Week 02: Data Wrangling

This week we’ll be building on what was covered last week, using `tidyr`
and `dplyr` to perform some more serious data wrangling, this time using
some real-world educational data taken from
[Ofsted](https://www.gov.uk/government/statistics/further-education-and-skills-inspections-and-outcomes-as-at-31-august-2021)
and the [Apprenticeships Statistical First Release
(SFR)](https://explore-education-statistics.service.gov.uk/find-statistics/apprenticeships-and-traineeships/2020-21#dataDownloads-1).
All data used in these examples is the most recent available data as of
December 2021.

# Resources

- [R for Data Science](https://r4ds.had.co.nz/transform.html):
  - See chapter 5 for adding columns with `mutate()`, removing rows with
    `filter()` and aggregating with `group_by()` and `summarise()`
  - See chapter 12 for ‘pivoting’, dealing with missing values and
    columns which have multiple observations per cell
  - See chapter 13 for joins (this chapter is especially relevant!)

# Exercises

## 1. Setup/Data import

<div>

> **Difficulty**
>
> Not too bad (hopefully)

</div>

The data for these exercises has been saved in this repo as three
tables: [`course_details.csv`](course_details.csv),
[`ofsted.csv`](ofsted.csv), and [`starts.csv`](starts.csv). Load these
into R using the following steps:

1.  Manually download the files from the repo

2.  Save these as files in your local R project

3.  Read the files into your R session using `readr::read_csv()`. It’s
    probably a good idea to save these as objects named
    `course_details`, `starts`, `ofsted` and `ofsted_scores`

    Your project folder should end up looking something like this:

    ![example project
    structure](project-structure-example.jpg "example project structure")

    ..and your script should look something like this:

    ``` r
    # == 01. Load packages =========================================================
    library(tidyverse)

    # == 02. Load data =============================================================
    course_details <- read_csv("course_details.csv")
    starts         <- read_csv("starts.csv")
    ofsted         <- read_csv("ofsted.csv")
    ofsted_scores  <- read_csv("ofsted_scores.csv")
    ```

## 2. Basic wrangling

<div>

> **Difficulty**
>
> Not too bad (hopefully)

</div>

Answer the following questions using the **`ofsted`** dataset.

1.  Using `count()`, verify that each provider has only one recorded
    inspection in the data (otherwise we could get skewed results later
    on).

2.  The UK Provider Reference Number (UKPRN) is a unique identifier for
    education providers. Use this to see whether providers are uniquely
    named in the `ofsted` dataset.

3.  What proportion of providers are ‘Independent specialist colleges’?

4.  Use `left_join()` to combine `ofsted` and `ofsted_scores`. What
    percentage of providers have ineffective safeguarding according to
    the data? Hint: look at `ofsted$inspection_number` and
    `ofsted_scores$inspection_id`

5.  `{lubridate}` is a package for working with dates. Using functions
    from this package, convert the `inspection_date` column to `date`
    format, and then create a new column giving the day of the week on
    which the inspection occurred. Using this column, find out if there
    is any pattern in how Ofsted chooses the day of the week to run
    inspections. Does this change based on inspection type?

## 3. More complex wrangling

<div>

> **Difficulty**
>
> Tricky

</div>

1.  Using the `ofsted` and `ofsted_scores` datasets, work out which
    provider group has the highest proportion of ‘outstanding’ scores
    for apprenticeships. Do this in a generalised way, e.g. so that your
    output is a data.frame/tibble which also includes proportions of
    ‘inadequate’ scores by provider group.

2.  Create a lookup table using your answer to question 3.1. Your output
    should look something like this (your figures should **not** match
    these):

    | provider_group                                                | outstanding | good | requires_improvement | inadequate |
    |:--------------------------------------------------------------|------------:|-----:|---------------------:|-----------:|
    | Independent learning providers (including employer providers) |        0.54 | 0.26 |                 0.95 |       0.40 |
    | Adult community education providers                           |        0.52 | 0.76 |                 0.83 |         NA |
    | Colleges                                                      |        0.25 | 0.80 |                 0.88 |       0.39 |
    | Higher education institutions                                 |        0.01 | 0.53 |                 0.27 |         NA |
    | Independent specialist colleges                               |          NA | 0.98 |                 0.65 |       0.99 |

    Hint: `janitor::clean_names()` may come in handy here.

3.  Using `starts`, `ofsted` and `ofsted_scores`, work out the total
    number of apprenticeship starts associated with each ofsted score
    for overall effectiveness.

4.  Similarly to 3.3, work out the total number of apprenticeships
    starts associated with each ofsted score for apprenticeships. Using
    this information with your results from 3.3, see if you can draw any
    conclusions about how providers perform in apprenticeships compared
    with how they perform overall.

## 4. A recursive problem

<div>

> **Difficulty**
>
> Challenging

</div>

**Note**: These questions are intended to give you a real run for your
money. If you get frustrated and aren’t having fun, feel free to abandon
this question at any point - this is more tricky than most stuff you’ll
come across in real life! If you do want to give them a try it’s at
least recommended that you collaborate with someone else. Remember to
‘make your code beautiful’ before you submit :)

1.  A provider may be at risk of closing if their overall effectiveness
    is rated as ‘inadequate’. Create a dataset which gives the
    percentage of starts which are at risk for each combination of
    `delivery_region` and `std_fwk_flag`. Call the additional column
    `risk_factor`.

2.  Now, say that a provider is at risk of closing if they are rated
    inadequate in any category, or if they have ineffective
    safeguarding. Modify your solution to 4.1 to include a column
    `risk_category` which gives the inspection category/categories which
    the starts are at risk for.

3.  Modify your answer to 4.2 to include several more ‘breakdown
    columns’. These should be the following:

    ``` r
    breakdowns <- c(
      "risk_category", "delivery_region", "std_fwk_flag", "provider_type", 
      "provider_group", "route", "apps_level"
    )
    ```

4.  Your dataset from 4.3 will include *implicit missing values*. Use
    `tidyr::complete()` to make these explicit. (Hint: read [R for Data
    Science chapter
    12.5](https://r4ds.had.co.nz/tidy-data.html?q=complete#missing-values-3)
    if you’re not sure what this is all about)

5.  Where there are missing values, we can attempt to fill these in
    using a more a version of the dataset from 4.3 which is generalised
    to include all `breakdowns` except `"apps_level"`. Populate the
    missing values of `risk_factor` in your answer to 4.4. with the more
    general values. If there are still missing values, repeat the
    process, this time using breakdowns except `"apps_level"` **and**
    `"route"`, and so on until there are no missing values remaining.
    Your output should include a column to indicate how ‘general’ the
    values of `risk_factor` are.

    (Application: this could be used as a method of modelling risk for
    future starts which don’t have historic data.)

6.  Go back and ‘refactor’ your solution so that there is as little
    repeating code as possible. Try writing functions to help with this
    :)
