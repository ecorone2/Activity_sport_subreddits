Plotting data over time
================

Package needed
--------------

``` r
library(tidyverse)
library(scales)
```

Loading dataset
---------------

``` r
load("subreddit_activity.RData")
```

Creating graph
--------------

``` r
# For breaks
daily_breaks <- seq(as.POSIXct("2018-08-09 12:00:00", origin = "1970-01-01"),
                    as.POSIXct("2018-08-27 12:00:00", origin = "1970-01-01"), "1 day")

# To save externally
png("activity.png", units = "in", height = 10, width = 16, res = 300)

ggplot(subreddit_data, 
       aes(x = datetime, y = accounts_active, 
           group = interaction(subreddit, date), color = subreddit)) +
  geom_point(size = 1) +
  geom_line(size = 1) +
  scale_x_datetime(breaks = daily_breaks, labels = date_format("%b-%d \n%I %p", tz = "America/Toronto")) +
    scale_y_continuous(limits = c(0,70000), breaks = seq(0, 70000, 10000), 
                     labels = function(x) format(x, big.mark = ",", scientific = FALSE)) +
  labs(x = "Date", 
       y = "Accounts \n Active", 
       title = "Tracking Activity of Sport Subreddits",
       caption = "Source: data extracted from Reddit every 15 minutes whenever my laptop was connected to the Internet",
       color = "Subreddit") +
  guides(linetype = FALSE) +
  scale_color_manual(values = c(
    "r/hockey" = "green3",
    "r/nba" = "blue",
    "r/nfl"="yellow4",
    "r/soccer"="red",
    "r/baseball" = "brown")) +  
  theme(axis.text=element_text(size = 11, face = "bold"),
        axis.title=element_text(size = 13, face = "bold"),
        axis.title.y = element_text(angle = 0, vjust = 0.5),
        legend.title = element_text(size=13, face="bold"),
        legend.text = element_text(face="bold"),
        strip.text.x = element_text(size = 10, colour = "black", face="bold"),
        plot.caption = element_text(size = 11))

dev.off()
```

    ## png 
    ##   2

Plot
----

![Graph](https://github.com/ecorone2/Activity_sport_subreddits/blob/master/activity.png)
