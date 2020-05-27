---
layout: post
title: Gasoline Demand and US Mobility
tags: [economics, R, gasoline, energy]
---

![Gasoline Supplied]({{ site.baseurl }}/img/2020-05-27-2.png)

[Apple](https://www.apple.com/covid19/mobility) and [Google](https://www.google.com/covid19/mobility/) have released data drawn from cell phone usage as proxies for compliance with stay-at-home orders. The Apple data is a measure of the change in requests for directions to Apple Maps, by driving, transit, and on foot. The Google data reports the relative change in visits to retail, grocery/pharmacy, parks, etc. Helpfully, they have been combined, and their data format has been standardized by GitHub user [ActiveConclusion](https://github.com/ActiveConclusion). 

These datasets came to mind recently when I was looking at the recovery in gasoline consumption in the US as stay-at-home restrictions have been relaxed (see plot above). I was curious what the relationship between the Mobility data and Gasoline consumption would actually be. Obviously, we know that there is a connection, but how complete is the mobility data? In order to find out, I ran a simple regression model of Total US Driving Index from Apple on [Motor Gasoline supplied](https://www.eia.gov/opendata/qb.php?category=401676&sdid=PET.WGFUPUS2.W) from the EIA.  (full code and results below)

Plotting the two series against one another indicated a strong relationship: 
![Mobility vs. Gasoline]({{ site.baseurl }}/img/2020-05-27-1.png)

Fitting a model to the two series (I'm just exploring here, so don't anyone get wound up about the lack of sophistication of this model) indicates a very strong relationship--changes in the mobility index explain 84.2% of the changes in gasoline supplied on a weekly basis, and each one point change in the Mobility Index results in a 0.8% change in gasoline usage; and these results are very highly statistically significant. What is the practical implication? As Apple is releasing its data each day, whereas the EIA data is released only weekly, with a five day lag (each Wednesday's release is for the week ended the previous Friday). Using this quite simple model, it would permit an analyst to create potentially more accurate forecasts of gasoline consumption, ahead of the EIA release each week.

Code:
`
# I'm going to do some plotting and analysis of 
# Google & Apple Mobility data vs. EIA gasoline usage

library(tidyverse)
library(eia)
library(lubridate)

# Apple Mobility Data, Downloaded from here:
# https://github.com/ActiveConclusion/COVID19_mobility
us_mobility_data <- read_csv("https://github.com/ActiveConclusion/COVID19_mobility/raw/master/summary_reports/summary_report_US.csv")

# Naive Lubridate Week Counting is Weeks Since Jan 1, 2020 (which was a Wednesday) 
# So Jan 01-07 Wk 1
#    Jan 08-14 Wk 2
us_mobility <- us_mobility_data %>%
  filter(sub_region_1=="Total") %>%
  mutate( date = as.Date(date), week = 1+trunc(time_length(interval(ymd("2019-12-28"),date),"week"))) %>%
  group_by(week) %>%
  summarise( driving=mean(driving, na.rm = T)) 

# EIA Weekly Petroleum Demand is calculated on Fridays
# So Jan 3 is first 2020 report
gas_supplied <- eia_series(id="PET.WGFUPUS2.W",key=Sys.getenv("EIA_KEY"),n=100)$data[[1]] %>%
  filter(year==2020) %>%
  rename(gas = value) %>%
  mutate(gas = gas/1000)

mobility_gas <- gas_supplied %>%
  left_join(us_mobility,by="week") %>%
  mutate(lngas = log(gas))

ggplot(mobility_gas) + 
  geom_line(aes(x=date,y=gas), size=2) + 
  geom_line(aes(x=date,y=driving),color="red",size=2) + 
  labs(y="Mobility Index, Million Bbbls/Day",
       x="2020") +
  geom_text(aes(label="Gasoline Usage (Million bbls/day)", x=ymd("2020-01-01"), y=-20), color="black", size=7, hjust=0) +
  geom_text(aes(label="Apple Mobility Index", x=ymd("2020-01-01"), y=-30), color="red", size=7, hjust=0) +
  theme_light()

ggplot(gas_supplied, aes(x=date,y=gas)) + geom_line() + theme_minimal() + labs(y="Million Barrels/Day Supplied")

gas_drive <- lm(lngas ~ driving, data=mobility_gas)
moderndive::get_regression_table(gas_drive)
moderndive::get_regression_summaries(gas_drive)
`

Regression Results for the model `ln(gasoline) = b_0 + b_1 * driving + e`
`
> moderndive::get_regression_table(gas_drive)
# A tibble: 2 x 7
  term      estimate std_error statistic p_value lower_ci upper_ci
  <chr>        <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
1 intercept    2.10      0.024     88.9        0    2.06      2.15
2 driving      0.008     0.001      9.24       0    0.006     0.01
> moderndive::get_regression_summaries(gas_drive)
# A tibble: 1 x 8
  r_squared adj_r_squared     mse   rmse sigma statistic p_value    df
      <dbl>         <dbl>   <dbl>  <dbl> <dbl>     <dbl>   <dbl> <dbl>
1     0.842         0.832 0.00767 0.0876 0.093      85.4       0     2
`