Allocation of staff hours to various energy projects
================

``` r
rm(list = ls())
library(ggplot2)
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.0     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.0
    ## ✔ lubridate 1.9.2     ✔ tibble    3.1.8
    ## ✔ purrr     1.0.1     ✔ tidyr     1.3.0
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the ]8;;http://conflicted.r-lib.org/conflicted package]8;; to force all conflicts to become errors

``` r
library(dplyr)
#library(hrbrthemes)
setwd("~/Desktop/DAIP/Case 6")

hours_needed <- read.csv(file = 'combined.csv', header=TRUE)
potential_sites <- read.csv(file = 'potential_sites.csv', header=TRUE)
staff <- read.csv(file= "staff.csv", header = TRUE)
```

``` r
#Total staff hours needed for each technology (irrespective of the size of the project)
hours_by_tech <- hours_needed %>%
  group_by(Technology, Role) %>%
  summarise(total_hours = sum(Sum.of.Hours))
```

    ## `summarise()` has grouped output by 'Technology'. You can override using the
    ## `.groups` argument.

``` r
ggplot(hours_by_tech, aes(y=total_hours, x=Role , fill=Technology)) +
  geom_bar(position="dodge", stat="identity")+
  coord_flip()+
  ggthemes::scale_fill_ptol()+
  #ggtitle("Number of hours required on 1 project from each employee")+
  labs(y="Total number of hours")
```

![](/Users/kevin/Desktop/DAIP/Case%206/graphsunnamed-chunk-3-1.png)<!-- -->

\#All 50 Potential sites

``` r
#Making some changes to the potential sites dataset, to accomodate inner join later 

potential_sites$Technology[potential_sites$Technology == "Solar"] <- "SOLAR"
potential_sites$Technology[potential_sites$MW >=50 & potential_sites$Technology=="Wind" ] <- "Wind >=50MW SPP"
potential_sites$Technology[potential_sites$MW <50 & potential_sites$Technology=="Wind" ] <- "Wind <50MW SPP"
potential_sites$Technology[potential_sites$Technology == "BESS"] <- "BESS SPP"

##Overseeing the number of technologies employed in the sites and total production

potential_sites %>%
  group_by(Technology) %>%
  summarise(n = n(), production=sum(MW))
```

    ## # A tibble: 4 × 3
    ##   Technology          n production
    ##   <chr>           <int>      <int>
    ## 1 BESS SPP            6        165
    ## 2 SOLAR              11        488
    ## 3 Wind <50MW SPP     21        659
    ## 4 Wind >=50MW SPP    12       1070

``` r
#Average production by technology
potential_sites %>%
  group_by(Technology) %>%
  summarise(n = n(), production=mean(MW))
```

    ## # A tibble: 4 × 3
    ##   Technology          n production
    ##   <chr>           <int>      <dbl>
    ## 1 BESS SPP            6       27.5
    ## 2 SOLAR              11       44.4
    ## 3 Wind <50MW SPP     21       31.4
    ## 4 Wind >=50MW SPP    12       89.2

``` r
ggplot(potential_sites, aes(x=MW))+
  geom_histogram(aes(fill=Technology), position = "identity", bins = 5)+
  facet_wrap(~Technology)+
  theme_minimal()+
  ggthemes::scale_fill_ptol()
```

![](/Users/kevin/Desktop/DAIP/Case%206/graphsunnamed-chunk-4-1.png)<!-- -->

``` r
#Staff & number of hours required for all of the 50 hours 

#Joining all 50 sites to the people required and their hours 
all_fifty <- potential_sites %>%
  inner_join(hours_by_tech, by="Technology", multiple = "all")

#Calculating number of total hours required for all 50 sites from each role 
role_wise_hours <- all_fifty %>%
  group_by(Role) %>%
  summarise(aggregate_hours=sum(total_hours))
```

``` r
#Staff list
staff %>%
  group_by(Role) %>%
  summarise(n=n())
```

    ## # A tibble: 9 × 2
    ##   Role                           n
    ##   <chr>                      <int>
    ## 1 Assistant Project Manager      9
    ## 2 Civil Engineer                 3
    ## 3 Electrical Engineer            2
    ## 4 Energy Yield Engineer          2
    ## 5 New Business Manager           1
    ## 6 Project Manager                4
    ## 7 Senior C&I Engineer            3
    ## 8 Senior Electrical Engineer     2
    ## 9 Senior Project Manager         2

``` r
#Calculate number of hours available per job role for 9 years 
available <- staff %>%
  group_by(Role) %>%
  summarise(Number_of_staff = n(), aggregate_available=sum(X9_years_availability))

#Creating new dataframe with availability of hours and requirement 

allocation <- available %>%
  inner_join(role_wise_hours, by="Role")

#Making dataset into longer format 
allocation_longer= pivot_longer(data = allocation, cols = c(aggregate_hours,aggregate_available), names_to = "requirement", values_to = "hours")

#To get rid of the scientific notations 
options(scipen=5)

ggplot(allocation_longer, aes(fill=requirement, y=hours, x=requirement)) + 
  geom_bar(position="dodge", stat="identity") +
  facet_wrap(~Role) +
  xlab("")+
  theme_minimal()+
  theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust=1), axis.ticks.y = element_blank(),
        axis.text.y = element_blank())+
  coord_flip()+
  ggthemes::scale_fill_ptol(name="Staff hours allocated", 
                    labels = c("Hours available", "Hours needed"))
```

![](/Users/kevin/Desktop/DAIP/Case%206/graphsunnamed-chunk-6-1.png)<!-- -->

``` r
  #ggtitle("Aggregate staff hours needed and available for next 9 years")
```

``` r
allocation$max_hours_per_person = 15318
allocation$hours_left=allocation$aggregate_hours-allocation$aggregate_available
allocation$people_needed=allocation$hours_left/allocation$max_hours_per_person
write_csv(allocation, "~/Desktop/DAIP/Case 6/people_needed_for_50.csv")
```

``` r
allocation_new <- data.frame(allocation$Role, allocation$Number_of_staff, allocation$people_needed)

#Making dataset into longer format 
allocation_longer2= pivot_longer(data = allocation_new, cols = c(allocation.Number_of_staff, allocation.people_needed), names_to = "People", values_to = "value")

allocation_longer2 %>% 
  filter(value>0) %>%
  ggplot(aes(y=value, x=allocation.Role , fill=People)) +
  geom_bar(position="dodge", stat="identity")+
  coord_flip()+
  ggthemes::scale_fill_ptol(name="Number of staff", 
                    labels = c("Present staff", "New hires required"))+
  labs(x="Role")
```

![](/Users/kevin/Desktop/DAIP/Case%206/graphsunnamed-chunk-8-1.png)<!-- -->

``` r
  #ggtitle("Present number of staff vs more required")
  #scale_fill_manual(labels=c("Present staff numbers", "Requirement of staff"))
```

OPTIMIZATION PROBLEM : Finding which energy technologies to invest in
given staff hours available

``` r
##Optimization problem

#Hours of the team required for each of the technology 
hours_needed_tech <- hours_by_tech %>%
  group_by(Technology) %>%
  summarise(hours = sum(total_hours))


opt <- all_fifty %>%
  select(Site, Technology, MW, Role, total_hours)
sum(available$aggregate_available)
```

    ## [1] 411288.3

``` r
#Optimize
library(lpSolve)

f.obj <- c(1, 1, 1, 1)

const.mat <- matrix(c(13786, 0, 0, 0, 
                      13786, 17083, 0, 0, 
                      13786, 17083, 24315, 0, 
                      13786, 17083, 24315, 28382, 
                      1, 0, 0, 0, 
                      0, 1, 0, 0, 
                      0, 0, 1, 0, 
                      0, 0, 0, 1) , ncol=4 , byrow=TRUE) 
f.dir <- c(rep("<=", 8))
f.rhs <- c(411288, 411288, 411288, 411288, 6, 11, 21, 12)

lp("max", f.obj, const.mat, f.dir, f.rhs, int.vec = 1:8)
```

    ## Success: the objective function is 22

``` r
lp("max", f.obj, const.mat, f.dir, f.rhs, int.vec = 1:8)$solution
```

    ## [1]  5 11  6  0

``` r
#Optimize
library(lpSolve)

f.obj <- c(1, 1, 1, 1)

const.mat <- matrix(c(13786, 0, 0, 0, 
                      13786, 17083, 0, 0, 
                      13786, 17083, 24315, 0, 
                      13786, 17083, 24315, 28382, 
                      1, 0, 0, 0, 
                      0, 1, 0, 0, 
                      0, 0, 1, 0, 
                      0, 0, 0, 1, 
                      1, 0, 0, 0, 
                      0, 1, 0, 0, 
                      0, 0, 1, 0, 
                      0, 0, 0, 1), ncol=4 , byrow=TRUE)

f.dir <- c("<=","<=","<=","<=","<=","<=","<=","<=",">=", ">=", ">=", ">=")

f.rhs <- c(411288, 411288, 411288, 411288, 6, 11, 21, 12, 1, 1, 1, 1)

lp("max", f.obj, const.mat, f.dir, f.rhs, int.vec = 1:12)
```

    ## Success: the objective function is 22

``` r
lp("max", f.obj, const.mat, f.dir, f.rhs, int.vec = 1:12)$solution
```

    ## [1]  5 11  5  1

``` r
potential_sites %>%
  group_by(Technology, MW) %>%
  arrange((desc(MW))) %>%
  top_n(n = 11)
```

    ## Selecting by MW

    ## # A tibble: 50 × 6
    ## # Groups:   Technology, MW [34]
    ##       X. Site                       Latitude      Longtitude   Technology     MW
    ##    <int> <chr>                      <chr>         <chr>        <chr>       <int>
    ##  1     1 " Loch Doon"               " 55.2841° N" " 4.3762° W" Wind >=50M…   200
    ##  2    35 " Loch Ness"               " 57.3229° N" " 4.4244° W" Wind >=50M…   150
    ##  3    17 " Glenkiln Sculpture Walk" " 55.1417° N" " 3.8498° W" Wind >=50M…    95
    ##  4    26 " Brimham Rocks"           " 54.0868° N" " 1.6569° W" Wind >=50M…    95
    ##  5    32 " Glenfinnan Viaduct"      " 56.8727° N" " 5.4317° W" Wind >=50M…    90
    ##  6     4 " Straiton Pond"           " 55.2392° N" " 4.5411° W" SOLAR          85
    ##  7    47 " Polesden Lacey"          " 51.2641° N" " 0.3995° W" Wind >=50M…    85
    ##  8     6 " Ailsa Craig"             " 55.2489° N" " 5.1397° W" SOLAR          80
    ##  9    39 " Glen Coe"                " 56.6827° N" " 5.0179° W" Wind >=50M…    75
    ## 10    22 " Aysgarth Falls"          " 54.3054° N" " 1.9481° W" Wind >=50M…    65
    ## # … with 40 more rows

``` r
potential_sites %>%
  group_by(Technology) %>%
  slice_max(order_by = MW, n = 11)
```

    ## # A tibble: 41 × 6
    ## # Groups:   Technology [4]
    ##       X. Site                          Latitude      Longtitude   Techno…¹    MW
    ##    <int> <chr>                         <chr>         <chr>        <chr>    <int>
    ##  1    50 " Cranborne Chase AONB"       " 50.9754° N" " 2.0032° W" BESS SPP    40
    ##  2    25 " Gordale Scar"               " 54.0609° N" " 2.1635° W" BESS SPP    35
    ##  3    14 " Doonfoot Beach"             " 55.3299° N" " 4.6519° W" BESS SPP    25
    ##  4    24 " Studley Royal Water Garden" " 54.1368° N" " 1.5596° W" BESS SPP    25
    ##  5     5 " Glenapp Castle"             " 55.0576° N" " 4.9456° W" BESS SPP    20
    ##  6    20 " Blairquhan Castle"          " 55.2655° N" " 4.6575° W" BESS SPP    20
    ##  7     4 " Straiton Pond"              " 55.2392° N" " 4.5411° W" SOLAR       85
    ##  8     6 " Ailsa Craig"                " 55.2489° N" " 5.1397° W" SOLAR       80
    ##  9    28 " Hardcastle Crags"           " 53.7666° N" " 2.0116° W" SOLAR       65
    ## 10    10 " Ardwell Gardens"            " 54.8343° N" " 4.9567° W" SOLAR       53
    ## # … with 31 more rows, and abbreviated variable name ¹​Technology
