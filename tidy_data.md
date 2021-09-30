tidy\_data
================
Paulina Han
2021/9/30

## pivot longer

don’t use gather

``` r
pulse_df = 
  haven::read_sas("./data_import_examples/data_import_examples/public_pulse_data.sas7bdat") %>%
  janitor::clean_names()
```

    ## Warning in FUN(X[[i]], ...): strings not representable in native encoding will
    ## be translated to UTF-8

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00C4>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00D6>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00E4>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00F6>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00DF>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00C6>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00E6>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00D8>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00F8>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00C5>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00E5>' to native encoding

``` r
pulse_tidy_data = 
  pivot_longer(
    pulse_df, 
    bdi_score_bl:bdi_score_12m,
    names_to = "visit", 
    names_prefix = "bdi_score_",
    values_to = "bdi") %>%
  mutate(
    visit = replace(visit,visit == "bl","00m"),
    visit = factor(visit) # good to make it factor
  )
```

## pivot\_wider

don’t use spread!

``` r
analysis_result = tibble(
  group = c("treatment", "treatment", "placebo", "placebo"),
  time = c("pre", "post", "pre", "post"),
  mean = c(4, 8, 3.5, 4)
)

analysis_result %>%
  pivot_wider(
  names_from = "time", 
  values_from = "mean") %>%
  knitr::kable() #make a readable table
```

| group     | pre | post |
|:----------|----:|-----:|
| treatment | 4.0 |    8 |
| placebo   | 3.5 |    4 |

## binding rows

import the lord of rings data

``` r
#import all the data then do the data tidying 
#always use bind_rows

fellowship_df = 
  read_excel("./data_import_examples/data_import_examples/LotR_Words.xlsx",range = "B3:D6") %>%
  mutate(movie = "fellowship_rings") 

return_df = 
  read_excel("./data_import_examples/data_import_examples/LotR_Words.xlsx",range = "J3:L6") %>%
  mutate(movie = "retuen_king") 

two_towers_df = 
  read_excel("./data_import_examples/data_import_examples/LotR_Words.xlsx",range = "F3:H6") %>%
  mutate(movie = "two_towers") 

lotr_tidy = 
  bind_rows(fellowship_df, two_towers_df, return_df) %>%
  janitor::clean_names() %>%
  pivot_longer(
    female:male,
    names_to = "gender", 
    values_to = "words") %>%
  mutate(race = str_to_lower(race)) %>%
  relocate(movie)
```

## joins

Look st FAS data

``` r
litters_df =
  read_csv("./data_import_examples/data_import_examples/FAS_litters.csv") %>%
  janitor::clean_names()%>%
  separate(group, into = c("dose","day_of_tx"),3)%>%
  relocate(litter_number) %>%
  mutate(dose = str_to_lower(dose))
```

    ## Rows: 49 Columns: 8

    ## -- Column specification --------------------------------------------------------
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...

    ## 
    ## i Use `spec()` to retrieve the full column specification for this data.
    ## i Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
pups_df = 
  read_csv("./data_import_examples/data_import_examples/FAS_pups.csv") %>%
  janitor::clean_names()%>%
  mutate(sex = recode(sex, `1`="male", `2`="female")) #make `1` into a character
```

    ## Rows: 313 Columns: 6

    ## -- Column specification --------------------------------------------------------
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk

    ## 
    ## i Use `spec()` to retrieve the full column specification for this data.
    ## i Specify the column types or set `show_col_types = FALSE` to quiet this message.

lets join these up

``` r
fas_df = 
  left_join(pups_df, litters_df,  by ="litter_number") %>% 
  relocate(litter_number,dose,day_of_tx)
```
