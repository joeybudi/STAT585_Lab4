---
title: "STAT585_LAB4"
author: "Team 8"
output:
  html_document:
    keep_md: True
---

## Link
[https://github.com/MirandaTilton/STAT585_Lab4](https://github.com/MirandaTilton/STAT585_Lab4)

## Team Members
- Joshua Budi
- Jing Li
- Yang Qiao
- Miranda Tilton



## Reading the data

```r
library(dplyr)
raw_df <- "https://data.iowa.gov/resource/m3tr-qhgy.json?county=Story" %>%
  jsonlite::read_json() %>%
  purrr::map_df(~ as_tibble(as.list(unlist(.x, use.names = T))))
```

## Cleaning the data

```r
library(stringr)
library(lubridate)

# Download and unzip .csv file
filename <- list(zip = "data/story-sales.zip", csv = "data/Iowa_Liquor_Sales-Story.csv")
if (!file.exists(filename$csv)) {
  if (!file.exists(filename$zip))
    download.file("https://stat585-at-isu.github.io/materials-2019/data/story-sales.zip", filename$zip)
  
  temp <- unzip(filename$zip, junkpaths = T, exdir = "data")
  unlink(str_subset(temp, "._Iowa_Liquor_Sales-Story.csv"))
}

# Read and clean data
raw_df <- readr::read_csv(filename$csv)
df <- raw_df %>%
  rename_all(~ str_remove_all(., " ")) %>%
  mutate_at(vars(City, County, CategoryName), str_to_upper) %>% 
  mutate(Date = mdy(Date), ZipCode = as.character(ZipCode),
         StoreLocation = str_extract(StoreLocation, "(?<=\\()\\S+, ?\\S+(?=\\)$)"),
         Category = case_when(
           str_detect(CategoryName, "BOURBON") ~ "BOURBON",
           str_detect(CategoryName, "WHISKIES|WHISKEY") ~ "WHISKIES",
           str_detect(CategoryName, "SCHNAPP") ~ "SCHNAPPS",
           str_detect(CategoryName, "CORDIAL") ~ "CORDIALS",
           str_detect(CategoryName, "LIQUEUR") ~ "LIQUEURS",
           str_detect(CategoryName, "VODKA") ~ "VODKA",
           str_detect(CategoryName, "GIN") ~ "GINS",
           str_detect(CategoryName, "BRANDIES|BRANDY") ~ "BRANDIES",
           str_detect(CategoryName, "RUM") ~ "RUM",
           str_detect(CategoryName, "AMARETTO") ~ "AMARETTO",
           str_detect(CategoryName, "TEQUILA") ~ "TEQUILA",
           T ~ "OTHER"
         )) %>%
  tidyr::separate(StoreLocation, c("Latitude", "Longitude"), sep = ", ?", convert = T) %>%
  select(-CategoryName, -CountyNumber, -`VolumeSold(Gallons)`,
         -ends_with("ItemNumber"), -starts_with("Store"), -starts_with("Vendor")) %>%
  select(Item = ItemDescription, Category, everything()) %>%
  select(-City, -County, -ZipCode, -Address, everything()) %>%
  filter_at(vars(Date, Longitude, Latitude), ~ !is.na(.)) %>%
  arrange(Date)
df
```

```
## # A tibble: 457,020 x 16
##    Item  Category Date       Latitude Longitude  Pack `BottleVolume(m…
##    <chr> <chr>    <date>        <dbl>     <dbl> <dbl>            <dbl>
##  1 Cana… WHISKIES 2012-01-03     42.0     -93.5     6             1750
##  2 Crow… WHISKIES 2012-01-03     42.0     -93.5    12              750
##  3 Para… GINS     2012-01-03     42.0     -93.7    12             1000
##  4 Hawk… VODKA    2012-01-03     42.0     -93.6    24              375
##  5 Sved… VODKA    2012-01-03     42.0     -93.5    12              750
##  6 Crow… WHISKIES 2012-01-03     42.0     -93.6    12              750
##  7 Smir… VODKA    2012-01-03     42.0     -93.5    12              750
##  8 Bart… VODKA    2012-01-03     42.0     -93.5    12             1000
##  9 Hawk… VODKA    2012-01-03     42.0     -93.5     6             1750
## 10 E & … BRANDIES 2012-01-03     42.0     -93.6    12              750
## # … with 457,010 more rows, and 9 more variables: StateBottleCost <dbl>,
## #   StateBottleRetail <dbl>, BottlesSold <dbl>, `Sale(Dollars)` <dbl>,
## #   `VolumeSold(Liters)` <dbl>, Address <chr>, City <chr>, ZipCode <chr>,
## #   County <chr>
```

## Shiny app
