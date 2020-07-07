<!-- README.md is generated from README.Rmd. Please edit that file -->

<!-- badges: start -->

[![R build
status](https://github.com/mpadge/autotest/workflows/R-CMD-check/badge.svg)](https://github.com/mpadge/autotest/actions?query=workflow%3AR-CMD-check)
[![codecov](https://codecov.io/gh/mpadge/autotest/branch/master/graph/badge.svg)](https://codecov.io/gh/mpadge/autotest)
[![Project Status:
Concept](https://www.repostatus.org/badges/latest/concept.svg)](https://www.repostatus.org/#concept)
<!-- badges: end -->

# autotest

Automatic testing of R packages via a simple YAML schema specifying one
or more example workflows for each function of a package. The simplest
workflows involve nominating data sets which may be submitted to a
function, while more complicated workflows may involve multiple data
sets, intermediate pre-processing stages, and other transformations
prior to submission to a nominated function. The following illustrates a
typical `yaml` schema, where items in angle-brackets (`<`, `>`) must be
completed.

    package: <package_name>
    functions:
        - <name of function>:
            - preprocess:
                - "<R code required for pre-processing exlosed in quotation marks>"
                - "<second line of pre-processing code>"
                - "<more code>"
            - parameters:
                - <param_name>: <value>
                - <another_param>: <value>
        - <name of same or different function>::
            - parameters:
                - <param_name>: <value>

This `yaml` code should be in the `test` directory of a package – not in
the `test/testthat` directory. The above template can be generated via
the function `at_yaml_template()`.

## Installation

Not yet on CRAN, so must be installed from remote repository host
systems using any one of the following options:

``` r
# install.packages("remotes")
remotes::install_git("https://git.sr.ht/~mpadge/autotest")
remotes::install_bitbucket("mpadge/autotest")
remotes::install_gitlab("mpadge/autotest")
remotes::install_github("mpadge/autotest")
```

The package can then be loaded the usual way:

``` r
library (autotest)
```

    #> Loading autotest

## What gets tested?

The package is primarily intended to test packages and functions
intended to process *data*, with data suitable for input into functions
within a package specified in the `autotest.yaml` template. Other types
of packages, such as those which provide access to data, will not be
effectively tested by `autotest`. Standard tests include translation,
substitution, and manipulation of classes and attributes of most
standard one- and two-dimensional forms for representing data in R.

## Example

Current functionality applied to test two functions from the [`SmartEDA`
package](https://github.com/daya6489/SmartEDA) as specified in the
following `yaml`. This also illustrates the use of data transformation
steps, in this case implementing a couple of simple transformations on
columns the external data set,
[`ISLR::Carseats`](https://cran.r-project.org/package=ISLR), including
introducing some missing data through replacing values with `NA`.

``` r
yaml <- c ("package: SmartEDA",
"functions:",
"   - ExpData:",
"       - preprocess:",
"           - 'x <- ISLR::Carseats'",
"           - 'x$Sales <- sqrt (x$Sales)'",
"           - 'x$CompPrice <- as.integer (x$CompPrice)'",
"           - 'x$Sales [ceiling (runif (10) * nrow (x))] <- NA'",
"       - parameters:",
"           - data: x",
"   - ExpData:",
"       - parameters:",
"           - data: ISLR::College",
"   - ExpStat:",
"       - parameters:",
"           - X: ISLR::Carseats$Sales",
"           - Y: ISLR::Carseats$Urban",
"           - valueOfGood: 'Yes'")
```

Both functions `ExpData` and `ExpStat` accept rectangular input, and so
can be tested via the `autotest_rectangular()` function:

``` r
autotest_rectangular (yaml = yaml)
```

    #> ★ Loading the following libraries:
    #> ● ISLR
    #> ● SmartEDA
    #> ★ Testing functions:
    #> ● ExpData
    #> ● ExpData
    #> ● ExpStat

All tests work for those functions, so no additional diagnostic output
is generated.
