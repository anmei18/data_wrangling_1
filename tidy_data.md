Tidy data
================

## `pivot_longer`

Use the PULSE data

``` r
pulse_df = 
  #read from sas data
  haven::read_sas("./data/public_pulse_data.sas7bdat") %>% 
  janitor::clean_names()
```

From Wide format to long format… Instead of having 4 columns for
bdi\_score, we want to have two columns, with one has the visit baseline
and the other one with the bdi\_score observed

``` r
pulse_tidy = 
  pulse_df %>% 
  pivot_longer(
    #specify the range using `:`
    bdi_score_bl:bdi_score_12m,
    # Column names need to be put into a new variable named "visit"
    names_to = "visit",
    #get rid of the prefix "bdi_score_"
    names_prefix =  "bdi_score_",
    #All values goes into new variable named "bdi_score_"
    values_to = "bdi"
  ) %>% 
  #relocate so that id and visit goes to the first two columns
  relocate(id,visit) %>% 
  mutate(
    visit = replace(visit, visit == "bl", "00m"),
    visit = factor(visit)
   #visit = recode(visit,"bl" = "00m")
  )
```

## `pivot_wider`

make up a results data table

``` r
#computer-friendly table
analysis_df = 
  tibble(
    group = c("treatment","treatment","control","control"),
    time = c("a","b","a","b"),
    group_mean = c(4,8,3.5,6)
  )

#human-friendly table : make it easier for human to read
analysis_df %>% 
  pivot_wider(
    names_from = "time",
    values_from = "group_mean"
  ) %>% 
  knitr::kable()
```

| group     |   a |   b |
|:----------|----:|----:|
| treatment | 4.0 |   8 |
| control   | 3.5 |   6 |

\#\#bind\_rows

We have data in multiple tables and want to stack those rows up..

import the LotR movie words stuff

First stiep: import each table

``` r
fellowship_df = 
  #only read in some columns
  read_excel("data/LotR_Words.xlsx",range = "B3:D6") %>% 
  #add in an extra column with the movie name
  mutate(movie = "fellowship_rings")

two_towers_df = 
  read_excel("data/LotR_Words.xlsx",range = "F3:H6") %>% 
  mutate(movie = "two_towers")

return_df = 
  read_excel("data/LotR_Words.xlsx",range = "J3:L6") %>% 
  mutate(movie = "return_king")


#bind all the rows together

Lotr = 
  bind_rows(fellowship_df, two_towers_df,return_df) %>% 
  janitor::clean_names() %>% 
  #from wider to longer
  pivot_longer(
    female:male,
    names_to = "sex",
    values_to = "words"
  ) %>% 
  mutate(race = str_to_lower(race)) %>% 
  relocate(movie)
```

Never use `rbind()` , always use `bind_rows()`

Don’t us `spread()`, use `pivot_wider()`

## joins

Look at FAS data.

Merge Litters dataset into the pups dataset Litter\_number is the key
because its the variable that exists in both dataset

We want to have one variable in each column In `litters_df`, group
column consist of three kinds of dosage: con, mod and low, and two
different day of treatment: 7 and 8.

``` r
litters_df = 
  read_csv("data/FAS_litters.csv") %>% 
  janitor::clean_names() %>% 
  # split group column  into twe columns: (specify where to do the splitting : split after 3 characters)
  separate(group, into = c("dose","day_of_tx"), 3) %>% 
  # move litter_number to the beginning
  relocate(litter_number) %>% 
  mutate(dose = str_to_lower(dose))
```

    ## Rows: 49 Columns: 8

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
pups_df = 
  read_csv("data/FAS_pups.csv") %>% 
  janitor::clean_names() %>% 
  # use ` ` to turn a number to a variable
  # from the 'sex' column, take '1' from datset and set it as male and '2' and set it as female
  mutate(sex = recode(sex, `1` = "male", `2` = "female"))
```

    ## Rows: 313 Columns: 6

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Let’s join these up!

``` r
fas_df =
  # `left_join` join by "litter_number"
  left_join(pups_df, litters_df, by = "litter_number") %>% 
  arrange(litter_number) %>% 
  relocate(litter_number, dose, day_of_tx)
```
