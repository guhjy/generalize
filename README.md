generalize: an R package for estimating population effects from randomized trial data
================

<!-- README.md is generated from README.Rmd. Please edit that file -->
<!-- badges: start -->
[![Travis build status](https://travis-ci.com/benjamin-ackerman/generalize.svg?branch=master)](https://travis-ci.com/benjamin-ackerman/generalize) [![AppVeyor build status](https://ci.appveyor.com/api/projects/status/github/benjamin-ackerman/generalize?branch=master&svg=true)](https://ci.appveyor.com/project/benjamin-ackerman/generalize) <!-- badges: end -->

**VERSION 0.1.1 UPDATE**: The `covariate_table` function now returns rows for *all* levels of factor variables, not excluding a reference level anymore.

**VERSION 0.1.2 UPDATE**: If population data come from a complex survey, the survey weights are now incorporated in the TATE estimation when using generalizability weighting methods. The survey weights are also used when assessing the generalizability and when creating covariate tables.

Randomized controlled trials (RCTs) are considered the gold standard for estimating the causal effect of a drug or intervention in a study sample. However, while RCTs have strong internal validity, they often have weaker external validity, making it difficult to generalize trial results from a “non-representative” study sample to a broader population. This makes it challenging for policymakers to accurately draw population-level conclusions from trial evidence.

Given increasing concern about potential lack of generalizability of RCT findings, statistical methods have recently been proposed to estimate population average treatment effects by supplementing trial data with target population-level data. The `generalize` R package is designed for researchers to implement these methods, and to better assess and improve upon the generalizability of RCT findings to a well-defined target population.

More details on the statistical methods can be found in this paper:

> Ackerman, B., Schmid, I., Rudolph, K. E., Seamans, M. J., Susukida, R., Mojtabai, R., Stuart, E.A. (2018). ["Implementing statistical methods for generalizing randomized trial findings to a target population"](https://www.sciencedirect.com/science/article/abs/pii/S0306460318312309?via%3Dihub). Addictive Behaviors, In Press.

For more information on how survey weights are incorporated when using generalizability weighting methods to estimate the TATE, see this paper:

> Ackerman, Lesko, C.R., Siddique, J., Susukida, R., Stuart, E.A. (2020). ["Generalizing randomized trial findings to a target population using complex survey population data"](https://arxiv.org/abs/2003.07500). Under Review.

Installation
------------

To install this R package, use the `install_github()` function from the `devtools` package:

``` r
devtools::install_github('benjamin-ackerman/generalize')
```

Core functions
--------------

The `generalize` package contains two core functions: `assess` and `generalize`.

### `assess`

`Assess` evaluates similarities and differences between the trial sample and the target population based on a specified list of common covariates. This is done in a few ways:

1.  providing a **summary table of covariate means** in the trial and the population, along with absolute standardized mean differences (ASMD) between the two sources of data.
2.  estimating the **probability of trial participation** based on a vector of covariate names and specified method, and summarizes their distribution across the trial and target populations. For this, logistic regression is the default method, but estimation using Random Forests or Lasso are currently supported by the package as well.
3.  calculating the Tipton **generalizability index** using the estimated probabilities of trial participation.
4.  **"trimming" the target population** data set so that the covariate bounds do not exceed those of the trial covariates. This checks for any violations of the coverage assumption (that the distribution of the covariates in the population are within the bounds of the covariate distributions in the trial), and reports how many individuals in the population would be excluded.

``` r
assess(trial = "trial", selection_covariates = covariates, 
       data = df, selection_method = "lr",
       survey_weights = "svy_wt_var", trim_pop = TRUE)
```

The summary of an object created with `assess` returns the selection model, the distribution of the trial participation probabilities by data source, and the method of trial participation probability estimation. It also contains the calculated Tipton generalizability index, the number of individuals excluded due to coverage violations, and a table of the covariate distributions.

### `generalize`

After assessing the generalizability, the `generalize` function can be used to implement methods to estimate the target population average treatment effect (TATE). Weighting by the odds using logistic regression is the default method, though weights based on other models (Lasso or Random Forests) or using BART or TMLE are available for use as well.

``` r
generalize(outcome = "outcome_name", treatment = "treatment", 
           trial = "trial_indicator", selection_covariates = covariates, 
           data = df, method = "weighting", selection_method = "lr", 
           survey_weights = "svy_wt_var", trim_pop = FALSE)
```

The summary of an object created using `generalize` returns a table with the SATE and TATE estimates, along with their standard errors and 95% confidence intervals. When weighting is used, a covariate distribution table is printed as well, where the covariate means in the trial are weighted by the generated trial participation weights.
