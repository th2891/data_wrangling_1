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

## binding rows

Using the LotR data.

First step: import each table (3).

``` r
fellowship_ring = 
  readxl::read_excel("./data/LotR_Words.xlsx", range = "F3:H6") %>% 
  mutate(movie = "two_towrs")

two_towers= 
  readxl::read_excel("./data/LotR_Words.xlsx", range = "B3:D6") %>% 
  mutate(movie = "fellowship_ring")

return_king = 
  readxl::read_excel("./data/LotR_Words.xlsx", range = "J3:L6") %>% 
  mutate(movie = "return_king")
```

Bind all the rows together

``` r
lotr_tidy = 
  bind_rows (fellowship_ring, two_towers, return_king) %>% 
  janitor::clean_names() %>% 
  relocate(movie) %>% 
  pivot_longer(
      female:male,
      names_to = "gender", 
      values_to = "words"
  )
```

## Joining datasets

Import and clean the FAS datasets.

``` r
pups_df = 
    read_csv("./data/FAS_pups.csv") %>% 
      janitor::clean_names() %>% 
      mutate(sex = recode(sex,`1` = "male", `2` = "female"))
```

    ## Rows: 313 Columns: 6

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
litters_df = 
  read_csv("./data/FAS_litters.csv") %>% 
  janitor::clean_names() %>% 
  relocate(litter_number) %>% 
  separate(group, into = c("dose", "day_of_tx"),sep = 3)
```

    ## Rows: 49 Columns: 8

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Next up, time to join them!

``` r
fas_df = 
  left_join(pups_df, litters_df, by = "litter_number") %>% 
  arrange(litter_number) %>% 
  relocate(litter_number, dose, day_of_tx)
```
