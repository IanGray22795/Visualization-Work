---
title: "Case Study 4: Take me out to the ball game"
author: "Ian Gray"
date: "April 07, 2022"
output:
  html_document:  
    keep_md: true
    toc: true
    toc_float: true
    code_folding: hide
    fig_height: 6
    fig_width: 12
    fig_align: 'center'
---






```r
playerid <- People
playerschools <- Schools
playerincome <- Salaries %>% 
  mutate(
         adjust_salary = adjust_for_inflation(
                                              price = salary, 
                                              from_date = yearID, 
                                              country = "US", 
                                              to_date = 2020
                                              )
        )
```

```
## Generating URL to request all 299 results
## Retrieving inflation data for US 
## Generating URL to request all 61 results
```

```r
playcollege <- CollegePlaying %>% 
  select(!yearID)
```

## Background

Over the campfire, you and a friend get into a debate about which college in Utah has had the best MLB success. As an avid BYU fan, you want to prove your point, and you go to data to settle the debate. You need a clear visualization that depicts the performance of BYU players compared to other Utah college players that have played in the major leagues. The library(Lahman) package has a comprehensive set of baseball data. It is great for testing out our relational data skills.

## Data Wrangling


```r
#joins
playercollege <- left_join(playerschools, playcollege, by = "schoolID")
player_college_salary <- left_join(playerincome, playercollege, by = "playerID")
player_data_full <- left_join(player_college_salary, playerid, by = "playerID")

#filtering data
player_dataUT <- player_data_full %>% 
  filter(!is.na(schoolID), state == "UT", schoolID != "byu")

player_dataBYU <- player_data_full %>% 
  filter(!is.na(schoolID), schoolID == "byu")
```

## Data Visualization


```r
ggplot()+
  geom_smooth(data = player_dataUT, 
              mapping = aes(x = yearID, 
                            y = adjust_salary), 
              color = "Red1",
              se = FALSE)+
  geom_smooth(data = player_dataBYU, 
              mapping = aes(x = yearID, 
                            y = adjust_salary),
              color = "Blue2",
              se = FALSE)+
  geom_point(data = player_dataUT, 
             mapping = aes(x = yearID, 
                           y = adjust_salary), 
             color = "Red1")+
  geom_point(data = player_dataBYU, 
             mapping = aes(x = yearID, 
                           y = adjust_salary),
             color = "Blue2")+
  scale_y_continuous(labels = scales::comma)+
  labs(title = "BYU Alumni Baseball Players Earn more than Other Utah College Alumni",
       subtitle = "BYU is in blue and other Utah colleges are in red",
       x = "Year Salary was Earned", 
       y = "Salary Earnings of Players * ", 
       color = "Year of Birth",
       caption = "* Salaries are adjusted for inflation as of 2020")+
  theme_dark()+
  theme(plot.caption = element_text(hjust = 0))
```

![](Case_study_4_files/figure-html/plot_data-1.png)<!-- -->

## Conclusions

Overall I was surprised at how many more data points there are for BYU alumni versus Utah College alumni. Because of the larger number of data points and the clear difference between Utah alumni earnings and BYU alumni earnings, I can easily say that yes BYU alumni Baseball players do tend to make more than Utah alumni Baseball players. There could be any number of factors that play into why this is. for instance the Utah Player data starts after the year 2000 while the BYU data reaches all the way back to 1985. This could indicate many things specifically in the early to mid 2000's there were many players earning very little during those years for both BYU and non BYU alumni in Utah. At the same time there does seem to be outliers on both sides so those should not have a major affect on skewing the results.
