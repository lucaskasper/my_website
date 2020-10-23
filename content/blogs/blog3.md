---
categories:
- ""
- ""
date: "2017-10-31T22:26:13-05:00"
description: Data on bike rentals around London.
draft: false
image: image-placeholder-title.jpg
keywords: ""
slug: tempus
title: Bike Data in R
---
```{r, get_tfl_data, cache=TRUE}
url <- "https://data.london.gov.uk/download/number-bicycle-hires/ac29363e-e0cb-47cc-a97a-e216d900a6b0/tfl-daily-cycle-hires.xlsx"

# Download TFL data to temporary file
httr::GET(url, write_disk(bike.temp <- tempfile(fileext = ".xlsx")))

# Use read_excel to read it as dataframe
bike0 <- read_excel(bike.temp,
                   sheet = "Data",
                   range = cell_cols("A:B"))

# change dates to get year, month, and week
bike <- bike0 %>% 
  clean_names() %>% 
  rename (bikes_hired = number_of_bicycle_hires) %>% 
  mutate (year = year(day),
          month = lubridate::month(day, label = TRUE),
          week = isoweek(day))
```



We can easily create a facet grid that plots bikes hired by month and year.

```{r tfl_month_year_grid, echo=FALSE, out.width="100%"}
knitr::include_graphics(here::here("images", "tfl_distributions_monthly.png"), error = FALSE)
```

Look at May and Jun and compare 2020 with the previous years. What's happening?

However, the challenge I want you to work on is to reproduce the following two graphs.

```{r tfl_absolute_monthly_change, echo=FALSE, out.width="100%"}
knitr::include_graphics(here::here("images", "tfl_monthly.png"), error = FALSE)
```

The second one looks at percentage changes from the expected level of weekly rentals. The two grey shaded rectangles correspond to the second (weeks 14-26) and fourth (weeks 40-52) quarters.

```{r tfl_percent_change, echo=FALSE, out.width="100%"}
knitr::include_graphics(here::here("images", "tfl_weekly.png"), error = FALSE)
```

For both of these graphs, you have to calculate the expected number of rentals per week or month between 2015-2019 and then, see how each week/month of 2020 compares to the expected rentals. Think of the calculation `excess_rentals = actual_rentals - expected_rentals`. 

Should you use the mean or the median to calculate your expected rentals? Why?

In creating your plots, you may find these links useful:

- https://ggplot2.tidyverse.org/reference/geom_ribbon.html
- https://ggplot2.tidyverse.org/reference/geom_tile.html 
- https://ggplot2.tidyverse.org/reference/geom_rug.html