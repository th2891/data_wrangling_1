Tidy Data
================

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──

    ## ✓ ggplot2 3.3.5     ✓ purrr   0.3.4
    ## ✓ tibble  3.1.4     ✓ dplyr   1.0.7
    ## ✓ tidyr   1.1.3     ✓ stringr 1.4.0
    ## ✓ readr   2.0.1     ✓ forcats 0.5.1

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

## Pivoting from wide format to long format

### pivot\_longer

Load the pulse data

``` r
pulse_data = 
  haven:: read_sas("./data/public_pulse_data.sas7bdat") %>% 
  janitor::clean_names()
```

Wide format to long format …

for each ID we have 4 observations on bdi score and visit on which those
things happened - instead of 4 columns for bdi score - want for each ID,
4 rows with bdi scores for each month  
- names\_prefix = “bdi\_score\_” removes that prefix from the variable
name to just get bl, 01 m, 06, 12m

``` r
pulse.data_tidy = 
  pulse_data %>% 
  pivot_longer(
    bdi_score_bl:bdi_score_12m, 
    names_to = "visit", 
    names_prefix = "bdi_score_",
    values_to = "bdi"
  )
```

Rewrite, combine, and extend (to add a mutate step)

``` r
pulse_data = 
  haven:: read_sas("./data/public_pulse_data.sas7bdat") %>% 
  janitor::clean_names() %>% 
  pivot_longer(
    bdi_score_bl:bdi_score_12m, 
    names_to = "visit", 
    names_prefix = "bdi_score_",
    values_to = "bdi"
  ) %>% 
  relocate(id, visit) %>% 
  mutate(visit = recode(visit, "bl" = "00m"))
```

## ‘pivot\_wider’

Make up some data to show pivot wider

``` r
analysis_result = 
  tibble(
      group = c("treatment", "treatment", "placebo", "placebo"), 
      time = c("pre", "post", "pre", "post"),
      mean = c (4, 8, 3.5, 4)
  )

analysis_result %>% 
    pivot_wider(
        names_from = "time", 
        values_from = "mean"
    )
```

    ## # A tibble: 2 × 3
    ##   group       pre  post
    ##   <chr>     <dbl> <dbl>
    ## 1 treatment   4       8
    ## 2 placebo     3.5     4
