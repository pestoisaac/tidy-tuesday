student-teacher-ratios
================
Basil
5/7/2019

``` r
## load packages
pacman::p_load(tidyverse,
               readr,
               janitor,
               cowplot ## to use ggdraw(), draw_image(), and draw_plot() to overlay the plot
                       ## on an image
               )

## load the student teacher ratio data

sr <- readr::read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-05-07/student_teacher_ratio.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   edulit_ind = col_character(),
    ##   indicator = col_character(),
    ##   country_code = col_character(),
    ##   country = col_character(),
    ##   year = col_double(),
    ##   student_ratio = col_double(),
    ##   flag_codes = col_character(),
    ##   flags = col_character()
    ## )

``` r
## obtained country and continents data from https://datahub.io/JohnSnowLabs/country-and-continent-codes-list
## Thanks to John Snow Labs for making this available
## load the country and continents data
countries <- read.csv("../data/country-continents.csv") %>% janitor::clean_names()
```

``` r
## create subset of data containing only country level data
## and only for those countries that have data for the years 2013 and 2016
srCntry <- 
  sr %>% 
    filter(sr$country_code %in% countries$three_letter_country_code) %>% 
    group_by(country_code) %>% 
    mutate(minYear = min(year),
           maxYear = max(year)) %>% 
    ungroup() %>% 
    filter(minYear <= 2013, maxYear >= 2016) %>% 
    select(-c("minYear", "maxYear")) %>% 
    left_join(y = countries %>%       ## add colums containing continent code
                    rename(country_code = three_letter_country_code) %>% 
                    select(c("country_code", "continent_code")),
              by = "country_code")
```

``` r
plotTT <- 
  srCntry %>% 
  spread(key = year, value = student_ratio) %>% 
  mutate(delta = `2016` - `2013`) %>% 
  filter(indicator %in% c("Pre-Primary Education", "Primary Education",
                          "Secondary Education", "Tertiary Education")) %>% 
  ggplot(aes(x = `2013`, y = -delta, col = continent_code)) +
  
    geom_point(aes(color = ifelse(continent_code %in% c("AF", "AS"),
                                  as.character(continent_code), 
                                  "Others")),
               alpha = 2/3,
               size = 1) +
  
    facet_wrap(~indicator) +
  
    scale_color_manual("Continent",
                       labels = c("Africa", "Asia", "Others"),
                       values = c("darkblue", "maroon3", "grey"), aesthetics = "color") +
  
    labs(x = "Student to Teacher Ratio in 2013",
         y = "Improvement between 2013 and 2016",
         title = "Convergence in Student Teacher Ratios between 2013 and 2016") +
  
    theme(title = element_text(color = "grey85", size = 14),
          panel.border = element_blank(),
          panel.grid.major = element_line(color = "grey95", size = 1/5),
          panel.grid.minor = element_line(color = "grey95", size = 1/20),
          axis.ticks = element_blank(),
          axis.line = element_blank(),
          axis.text = element_text(color = "grey85"),
          
          strip.background = element_rect(fill = NA, linetype = 0),
          strip.text = element_text(color = "grey75", size = 12),
          axis.title = element_text(color = "grey85", size = 12),
          
          legend.position = "bottom",
          legend.justification = "center",
          legend.title = element_text(color = "grey85", size = 12),
          legend.text = element_text(color = "grey75", size = 12),
          
            
          )


ggdraw() +
  draw_image("../images/blackboard-green-clean.jpg", scale = 1.5) + 
  draw_plot(plotTT)  
```

![](19-05-07-student-teacher-ratios_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
ggsave("../plots/19-05-07-student-teacher-ratio.png", width = 29, height = 21, units = "cm", dpi = "retina")
```
