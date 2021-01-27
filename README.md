New Zealand Bird of the Year
================
Jamie Hargreaves

``` r
library(tidyverse)
library(magrittr)
library(hrbrthemes)

# get the data
tuesdata <- tidytuesdayR::tt_load(2019, week = 47)
nz_bird <- tuesdata$nz_bird

# count number of votes by breed each day
num_votes <- nz_bird %>%
  filter(!is.na(bird_breed)) %>%
  group_by(date, bird_breed) %>%
  tally() %>%
  ungroup()

# get the cumulative number of votes by breed each day
cum_votes <- num_votes %>%
  group_by(bird_breed) %>%
  mutate(cs = cumsum(n))

# get top 10 breeds by total vote count
top_10 <- num_votes %>%
  group_by(bird_breed) %>%
  summarise(n = sum(n)) %>%
  top_n(10, n) %>%
  arrange(desc(n))

# add a flag for top 10 breeds
cum_votes %<>%
  mutate(top_10 = ifelse(bird_breed %in% top_10$bird_breed, TRUE, FALSE))

# filter the top 10 breeds
main_breeds <- cum_votes %>%
  filter(top_10 == TRUE)

# filter all the other breeds
other_breeds <- cum_votes %>%
  filter(top_10 == FALSE)

# arrange the facors of main_breeds
main_breeds$bird_breed <- factor(main_breeds$bird_breed, levels = top_10$bird_breed)
  
# plot cumulative number of votes for each breed
main_breeds %>%
  
  # plot the top 10 breeds and colour by breed
  ggplot(aes(x = date, y = cs, group = bird_breed, colour = bird_breed)) + 
  geom_line() + 
  
  # plot the remaining breeds in gray
  geom_line(data = other_breeds, aes(x = date, y = cs), colour = "#e9e9e9") +
  
  # add commas to y-axus labels
  scale_y_comma() + 
  
  # rename the legend
  scale_colour_discrete(name = "Breed of Bird") +
  
  # add axis labels and titles
  labs(
    x = "Date",
    y = "Cumulative Sum of Votes",
    title = "New Zealand Bird of the Year",
    subtitle = "Cumulative Sum of Votes by Breed"
  ) + 
  
  # set the theme
  theme_ipsum()
```

<img src="README_files/figure-gfm/unnamed-chunk-1-1.png" width="672" style="display: block; margin: auto;" />
