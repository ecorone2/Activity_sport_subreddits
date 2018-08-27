Extracting data from Reddit
================

Package needed
--------------

``` r
library(tidyverse)
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
