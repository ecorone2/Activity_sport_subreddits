Extracting subreddit activity
================

Packages needed
---------------

``` r
library(tidyverse)
library(scales)
library(gridExtra)
```

Data extraction script
----------------------

This script needs to be run once manually. Then, it can be run on a schedule, using programs such as Task Scheduler in Windows. With each iteration, new data are added to the same files over time.

``` r
# During the first manual run, do not load this file, as it would not have been yet created
load("subreddit_activity.RData")

# Data frame with links
links <- data_frame(URLs = c("https://www.reddit.com/r/soccer/about.json", 
                             "https://www.reddit.com/r/nba/about.json",
                             "https://www.reddit.com/r/nfl/about.json",
                             "https://www.reddit.com/r/hockey/about.json",
                             "https://www.reddit.com/r/baseball/about.json"))

# Initializing lists 
double_list <- list(0)
single_list <- list(0)

# Function to extract online data from Reddit
for (i in seq_along(links$URLs)) { 
  double_list[[i]] <- jsonlite::fromJSON(url(links$URLs[i]))
  single_list[[i]] <- double_list[[i]][["data"]]
}

# Saving each iteration of data
tracking <- data.frame(subreddit = map_chr(single_list, "display_name_prefixed"),
                       accounts_active = map_int(single_list, "accounts_active"),
                       subscribers = map_int(single_list, "subscribers"),
                       datetime = Sys.time())

# Only run this once. Then comment it out.
start <- tracking

# Saving new extraction with previous runs
perm_file <- rbind(start, tracking)

# To prepare for a new run
start <- perm_file

# Saving the image
save.image("subreddit_activity.RData")

# Save externally just in case main file gets corrupted
write.csv(perm_file, file = paste(Sys.Date(), "reddit_extract.txt", sep = "_"), row.names = F)
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

Plot
----

![Graph](https://github.com/ecorone2/Activity_sport_subreddits/blob/master/figures/activity.png)

Processing data and creating a table for subscriber gains
---------------------------------------------------------

``` r
gains <- subreddit_data %>% 
  group_by(subreddit) %>% 
  summarize(minimum = min(subscribers),
            maximum = max(subscribers)) %>% 
  mutate(subs_gains = format(maximum - minimum, big.mark = ",")) %>% 
  select(subreddit, subs_gains) %>% 
  rename("Subreddit" = subreddit,
         "Subscriber Gains" = subs_gains)

# Exporting table
gains_table <- tableGrob(gains, rows = NULL)
grid.arrange(gains_table)
```

Table
-----

Subscribers gained by each subreddit from August 9 to August 27 2018.

![Table](https://github.com/ecorone2/Activity_sport_subreddits/blob/master/figures/subscriber_gains.png)
