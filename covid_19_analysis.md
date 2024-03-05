covid_19_analysis
================
Benji Edwards
2024-03-03

Is state an indicator for COVID deaths?

# Import

Load R packages and data from Johns Hopkins COVID-19 repository. This
data includes case reports from January 21, 2020 to March 10, 2023.

``` r
# Load in R packages and data
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.0     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.3     ✔ tidyr     1.3.1
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
us_covid_cases <- read_csv("https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_US.csv")
```

    ## Rows: 3342 Columns: 1154
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr    (6): iso2, iso3, Admin2, Province_State, Country_Region, Combined_Key
    ## dbl (1148): UID, code3, FIPS, Lat, Long_, 1/22/20, 1/23/20, 1/24/20, 1/25/20...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
us_covid_deaths <- read_csv("https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_deaths_US.csv")
```

    ## Rows: 3342 Columns: 1155
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr    (6): iso2, iso3, Admin2, Province_State, Country_Region, Combined_Key
    ## dbl (1149): UID, code3, FIPS, Lat, Long_, Population, 1/22/20, 1/23/20, 1/24...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

\#Inspect Let’s take a look at the data that we’ve imported.

``` r
# US COVID cases
nrow(us_covid_cases)
```

    ## [1] 3342

``` r
head(us_covid_cases)
```

    ## # A tibble: 6 × 1,154
    ##        UID iso2  iso3  code3  FIPS Admin2  Province_State Country_Region   Lat
    ##      <dbl> <chr> <chr> <dbl> <dbl> <chr>   <chr>          <chr>          <dbl>
    ## 1 84001001 US    USA     840  1001 Autauga Alabama        US              32.5
    ## 2 84001003 US    USA     840  1003 Baldwin Alabama        US              30.7
    ## 3 84001005 US    USA     840  1005 Barbour Alabama        US              31.9
    ## 4 84001007 US    USA     840  1007 Bibb    Alabama        US              33.0
    ## 5 84001009 US    USA     840  1009 Blount  Alabama        US              34.0
    ## 6 84001011 US    USA     840  1011 Bullock Alabama        US              32.1
    ## # ℹ 1,145 more variables: Long_ <dbl>, Combined_Key <chr>, `1/22/20` <dbl>,
    ## #   `1/23/20` <dbl>, `1/24/20` <dbl>, `1/25/20` <dbl>, `1/26/20` <dbl>,
    ## #   `1/27/20` <dbl>, `1/28/20` <dbl>, `1/29/20` <dbl>, `1/30/20` <dbl>,
    ## #   `1/31/20` <dbl>, `2/1/20` <dbl>, `2/2/20` <dbl>, `2/3/20` <dbl>,
    ## #   `2/4/20` <dbl>, `2/5/20` <dbl>, `2/6/20` <dbl>, `2/7/20` <dbl>,
    ## #   `2/8/20` <dbl>, `2/9/20` <dbl>, `2/10/20` <dbl>, `2/11/20` <dbl>,
    ## #   `2/12/20` <dbl>, `2/13/20` <dbl>, `2/14/20` <dbl>, `2/15/20` <dbl>, …

``` r
# US COVID deaths
nrow(us_covid_deaths)
```

    ## [1] 3342

``` r
head(us_covid_deaths)
```

    ## # A tibble: 6 × 1,155
    ##        UID iso2  iso3  code3  FIPS Admin2  Province_State Country_Region   Lat
    ##      <dbl> <chr> <chr> <dbl> <dbl> <chr>   <chr>          <chr>          <dbl>
    ## 1 84001001 US    USA     840  1001 Autauga Alabama        US              32.5
    ## 2 84001003 US    USA     840  1003 Baldwin Alabama        US              30.7
    ## 3 84001005 US    USA     840  1005 Barbour Alabama        US              31.9
    ## 4 84001007 US    USA     840  1007 Bibb    Alabama        US              33.0
    ## 5 84001009 US    USA     840  1009 Blount  Alabama        US              34.0
    ## 6 84001011 US    USA     840  1011 Bullock Alabama        US              32.1
    ## # ℹ 1,146 more variables: Long_ <dbl>, Combined_Key <chr>, Population <dbl>,
    ## #   `1/22/20` <dbl>, `1/23/20` <dbl>, `1/24/20` <dbl>, `1/25/20` <dbl>,
    ## #   `1/26/20` <dbl>, `1/27/20` <dbl>, `1/28/20` <dbl>, `1/29/20` <dbl>,
    ## #   `1/30/20` <dbl>, `1/31/20` <dbl>, `2/1/20` <dbl>, `2/2/20` <dbl>,
    ## #   `2/3/20` <dbl>, `2/4/20` <dbl>, `2/5/20` <dbl>, `2/6/20` <dbl>,
    ## #   `2/7/20` <dbl>, `2/8/20` <dbl>, `2/9/20` <dbl>, `2/10/20` <dbl>,
    ## #   `2/11/20` <dbl>, `2/12/20` <dbl>, `2/13/20` <dbl>, `2/14/20` <dbl>, …

# Tidy

Now let’s tidy the data so it can be analyzed. I want to summarize all
of the cases and deaths by county.

``` r
us_cases_clean <- us_covid_cases %>%
  select(-c('Lat', 'Long_', 'UID', 'iso2', 'iso3', 'code3', 'FIPS', 'Country_Region', 'Combined_Key'))
us_cases_clean <-  us_cases_clean %>%
  pivot_longer(cols = -c('Admin2', 'Province_State'),
               names_to = "date",
               values_to = "cases")
us_cases_clean
```

    ## # A tibble: 3,819,906 × 4
    ##    Admin2  Province_State date    cases
    ##    <chr>   <chr>          <chr>   <dbl>
    ##  1 Autauga Alabama        1/22/20     0
    ##  2 Autauga Alabama        1/23/20     0
    ##  3 Autauga Alabama        1/24/20     0
    ##  4 Autauga Alabama        1/25/20     0
    ##  5 Autauga Alabama        1/26/20     0
    ##  6 Autauga Alabama        1/27/20     0
    ##  7 Autauga Alabama        1/28/20     0
    ##  8 Autauga Alabama        1/29/20     0
    ##  9 Autauga Alabama        1/30/20     0
    ## 10 Autauga Alabama        1/31/20     0
    ## # ℹ 3,819,896 more rows

``` r
# Let's repeat this for {us_covid_deaths}
```
