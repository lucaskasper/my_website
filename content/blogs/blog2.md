---
categories:
- ""
- ""
date: "2017-10-31T22:26:09-05:00"
description: Lorem Etiam Nullam
draft: false
image: 400px-U.S._Treasury_Yield_Curves_-_v1.png
keywords: ""
slug: magna
title: Yield Curve Inversion
---
Every so often, we hear warnings from commentators on the "inverted yield curve" and its predictive power with respect to recessions. An explaination on what a [inverted yield curve is can be found here](https://www.reuters.com/article/us-usa-economy-yieldcurve-explainer/explainer-what-is-an-inverted-yield-curve-idUSKBN1O50GA). If you'd rather listen to something, here is a great podcast from [NPR on yield curve indicators](https://www.podbean.com/media/share/dir-4zgj9-6aefd11)

In addition, many articles and commentators think that, e.g., [*Yield curve inversion is viewed as a harbinger of recession*](https://www.bloomberg.com/news/articles/2019-08-14/u-k-yield-curve-inverts-for-first-time-since-financial-crisis). One can always doubt whether inversions are truly a harbinger of recessions, and [use the attached parable on yield curve inversions](https://twitter.com/5_min_macro/status/1161627360946511873).


```{r yield_curve_parable.jpg, echo=FALSE, out.width="100%"}
knitr::include_graphics(here::here("images", "yield_curve_parable.jpg"), error = FALSE)

```



In our case we will look at US data and use the [FRED database](https://fred.stlouisfed.org/) to download historical yield curve rates, and plot the yield curves since 1999 to see when the yield curves flatten. If you want to know more, a very nice article that explains the [yield curve is and its inversion can be found here](https://fredblog.stlouisfed.org/2018/10/the-data-behind-the-fear-of-yield-curve-inversions/). At the end of this chllenge you should produce this chart

```{r yield_curve_challenge, echo=FALSE, out.width="100%"}
knitr::include_graphics(here::here("images", "yield_curve_challenge.png"), error = FALSE)
```


First, we will use the `tidyquant` package to download monthly rates for different durations. 

```{r get_rates, warning=FALSE}
# Get a list of FRED codes for US rates and US yield curve; choose monthly frequency
# to see, eg., the 3-month T-bill https://fred.stlouisfed.org/series/TB3MS
tickers <- c('TB3MS', # 3-month Treasury bill (or T-bill)
             'TB6MS', # 6-month
             'GS1',   # 1-year
             'GS2',   # 2-year, etc....
             'GS3',
             'GS5',
             'GS7',
             'GS10',
             'GS20',
             'GS30')  #.... all the way to the 30-year rate

# Turn  FRED codes to human readable variables
myvars <- c('3-Month Treasury Bill',
            '6-Month Treasury Bill',
            '1-Year Treasury Rate',
            '2-Year Treasury Rate',
            '3-Year Treasury Rate',
            '5-Year Treasury Rate',
            '7-Year Treasury Rate',
            '10-Year Treasury Rate',
            '20-Year Treasury Rate',
            '30-Year Treasury Rate')

maturity <- c('3m', '6m', '1y', '2y','3y','5y','7y','10y','20y','30y')

# by default R will sort these maturities alphabetically; but since we want
# to keep them in that exact order, we recast maturity as a factor 
# or categorical variable, with the levels defined as we want
maturity <- factor(maturity, levels = maturity)

# Create a lookup dataset
mylookup<-data.frame(symbol=tickers,var=myvars, maturity=maturity)
# Take a look:
mylookup %>% 
  knitr::kable()

df <- tickers %>% tidyquant::tq_get(get="economic.data", 
                   from="1960-01-01")   # start from January 1960

glimpse(df)
```

Our dataframe `df` has three columns (variables):

- `symbol`: the FRED database ticker symbol
- `date`: already a date object
- `price`: the actual yield on that date

The first thing would be to join this dataframe `df` with the dataframe `mylookup` so we have a more readable version of maturities, durations, etc.

```{r join_data, warning=FALSE}

yield_curve <-left_join(df,mylookup,by="symbol") 
```

## Plotting the yield curve

This may seem long but it should be easy to produce the following three plots

### Yields on US rates by duration since 1960

```{r yield_curve_1, echo=FALSE, out.width="100%"}
knitr::include_graphics(here::here("images", "yield_curve1.png"), error = FALSE)
```

### Monthly yields on US rates by duration since 1999 on a year-by-year basis


```{r yield_curve_2, echo=FALSE, out.width="100%"}
knitr::include_graphics(here::here("images", "yield_curve2.png"), error = FALSE)
```



### 3-month and 10-year yields since 1999

```{r yield_curve_3, echo=FALSE, out.width="100%"}
knitr::include_graphics(here::here("images", "yield_curve3.png"), error = FALSE)
```


According to [Wikipedia's list of recession in the United States](https://en.wikipedia.org/wiki/List_of_recessions_in_the_United_States), since 1999 there have been two recession in the US: between Mar 2001–Nov 2001 and between Dec 2007–June 2009. Does the yield curve seem to flatten before these recessions? Can a yield curve flattening really mean a recession is coming in the US? Since 1999, when did short-term (3 months) yield more than longer term (10 years) debt?


Besides calculating the spread (10year - 3months), there are a few things we need to do to produce our final plot

1. Setup data for US recessions 
1. Superimpose recessions as the grey areas in our plot
1. Plot the spread between 30 years and 3 months as a blue/red ribbon, based on whether the spread is positive (blue) or negative(red)


- For the first, the code below creates a dataframe with all US recessions since 1946

```{r setup_US-recessions, warning=FALSE}

# get US recession dates after 1946 from Wikipedia 
# https://en.wikipedia.org/wiki/List_of_recessions_in_the_United_States

recessions <- tibble(
  from = c("1948-11-01", "1953-07-01", "1957-08-01", "1960-04-01", "1969-12-01", "1973-11-01", "1980-01-01","1981-07-01", "1990-07-01", "2001-03-01", "2007-12-01"),  
  to = c("1949-10-01", "1954-05-01", "1958-04-01", "1961-02-01", "1970-11-01", "1975-03-01", "1980-07-01", "1982-11-01", "1991-03-01", "2001-11-01", "2009-06-01") 
  )  %>% 
  mutate(From = ymd(from), 
         To=ymd(to),
         duration_days = To-From)

recessions
```

- To add the grey shaded areas corresponding to recessions, we use `geom_rect()`
- to colour the ribbons blue/red we must see whether the spread is positive or negative and then use `geom_ribbon()`. You should be familiar with this from last week's homework on the excess weekly/monthly rentals of Santander Bikes in London.

```{r}

plot<-df %>% 
  filter(symbol=="GS10"|symbol=="TB3MS") %>% 
  pivot_wider(names_from = symbol,values_from=price) %>% 
  mutate(diff=GS10-TB3MS) %>% 
  mutate(max=ifelse(diff>0,diff,0),min=ifelse(diff<0,diff,0)) 

plot %>% 
  ggplot(aes(x=date,y=diff))+
  geom_line()+
  geom_rect(data=recessions[4:11,], aes(NULL,NULL,xmin=From,xmax=To,ymin=-Inf,ymax=Inf),fill='grey',colour='grey',alpha=0.7)+
  labs(title = "Yield Curve Inversions:10-year minus 3-month U.S.Treasury rates",
       y="Difference (10 year-3 month)yield in %")+
  scale_x_date(date_breaks = "2 years",date_labels = "%Y")+
  geom_hline(aes(yintercept=0))+
  geom_ribbon(aes(x=date,ymin=min,ymax=0),fill="#EAB5B7",alpha=0.7)+
  geom_ribbon(aes(x=date,ymin=0,ymax=max),fill="#a6bddb",alpha=0.7)+
  geom_rug(colour=ifelse(df$diff>=0,"#EAB5B7","#a6bddb"),alpha=0.7)
  
  
```
geom_rug(aes(colour=ifelse(diff>=0,">=0","<0")),sides="b",alpha=0.7)