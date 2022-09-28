---
title: "CASE STUDY TITLE"
author: "YOUR NAME"
date: "July 18, 2022"
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
# Use this R-Chunk to import all your datasets!
food_data <- read_csv("DS 350 Visualizations/OnionsFoodLion.csv")
View(food_data)

names(food_data) <- str_replace_all(names(food_data),c(" " = ".", "," = ""))
```

## Background

_Place Task Background Here_

## Data Wrangling


```r
# Use this R-Chunk to clean & wrangle your data!
food_data_date <- food_data %>% 
  mutate(Date = as.POSIXct(
    strptime(Week, format="%m/%d/%Y")),
    Year = year(Date),
    Month = months(Date))


View(food_data_date)

edit_food <- food_data_date %>% 
  #select(Gamoney,Week,GAmoneyPriorYr,Year,Month,Date) %>% 
  filter(!is.na(Gamoney))

oniontype <- food_data_date %>% 
  group_by(Sub.Category, Pack.Size) %>% 
  summarise(total_sales_GA = sum(na.omit(Gamoney)),
            total_sales_RM = sum(na.omit(Rmmoney)))

oniontype2 <- pivot_longer(oniontype, cols = c(total_sales_RM,total_sales_GA))

View(oniontype)
View(oniontype2)


ggplot(oniontype2)+
  geom_col(aes(x = Pack.Size,y = value ,fill = name), position = "dodge")+
  facet_wrap(vars(Sub.Category), scales = "free")
```

![](job-application_files/figure-html/tidy_data-1.png)<!-- -->

```r
View(edit_food)
edit_food2 <- food_data_date %>% 
  filter(!is.na(Gamoney)) %>% 
  filter(!is.na(GAmoneyPriorYr)) %>% 
  filter(!is.na(GA.EQ)) %>%       
  filter(!is.na(GA.EQ.Prior.Yr))%>% 
  filter(GA.EQ > 0) %>% 
  filter(GA.EQ.Prior.Yr >0) %>% 
  mutate(foodlionperpound = Gamoney/GA.EQ,
         foodlionprperpound = GAmoneyPriorYr/GA.EQ.Prior.Yr) %>%
  group_by(Year)%>% 
  summarise(foodavg = mean(foodlionperpound),
            foodavgpr = mean(foodlionprperpound))

View(edit_food2)

edit_food4 <- food_data_date %>% 
  filter(!is.na(Rmmoney)) %>% 
  filter(!is.na(RMmoneyPriorYr)) %>%
  filter(!is.na(RM.EQ)) %>%       
  filter(!is.na(RM.EQ.Prior.Yr))%>%
  filter(RM.EQ >0) %>%       
  filter(RM.EQ.Prior.Yr > 0)%>%
  group_by(Year) %>% 
  summarise(rmpound = Rmmoney/RM.EQ,
            rmprpound = RMmoneyPriorYr/RM.EQ.Prior.Yr)

View(edit_food4)
edit_food5 <- pivot_longer(edit_food4, cols = c(rmpound,rmprpound)) 
  
edit_food3 <- pivot_longer(edit_food2, cols = c(foodavg,foodavgpr))

View(edit_food3)

ggplot(edit_food3)+
  geom_col(aes(x = Year, y = value, fill = name ),position = "dodge")
```

![](job-application_files/figure-html/tidy_data-2.png)<!-- -->

```r
ggplot(edit_food5)+
  geom_col(aes(x = Year, y = value, fill = name ),position = "dodge")
```

![](job-application_files/figure-html/tidy_data-3.png)<!-- -->

```r
ggplot(edit_food)+
  geom_point(aes(x = Week, y = Gamoney ), posistion = "dodge")+
  facet_wrap(vars(Year),scales = "free_x")
```

![](job-application_files/figure-html/tidy_data-4.png)<!-- -->

## Data Visualization


```r
oniontype <- food_data_date %>%
  filter(Year != 2017,Year!= 2021) %>% 
  group_by(Sub.Category,Year) %>% 
  summarise(total_sales_GA = mean(na.omit(Gamoney)),
            total_sale_GApr = mean(na.omit(GAmoneyPriorYr)),
            total_sales_RM = mean(na.omit(Rmmoney)),
            total_sale_RMpr = mean(na.omit(RMmoneyPriorYr)))

oniontype3 <- food_data_date %>%
  filter(Year != 2017,Year!= 2021) %>% 
  group_by(Sub.Category,Year) %>% 
  summarise(total_sales_GA = sum(na.omit(Gamoney)),
            total_sale_GApr = sum(na.omit(GAmoneyPriorYr)),
            total_sales_RM = sum(na.omit(Rmmoney)),
            total_sale_RMpr = sum(na.omit(RMmoneyPriorYr)))


avg <- food_data_date %>% 
  group_by(Year)


oniontype2 <- pivot_longer(oniontype, cols = c(total_sales_RM,total_sale_GApr,total_sales_GA,total_sale_RMpr))

oniontype4 <- pivot_longer(oniontype3, cols = c(total_sales_RM,total_sale_GApr,total_sales_GA,total_sale_RMpr))

View(oniontype)
View(oniontype4)

onion_per_year <- food_data_date %>% 
  group_by(Year) %>%
  filter(Year != 2017,Year != 2021) %>% 
  summarise(FoodLion_per_pound = mean(na.omit(Gamoney/GA.EQ)),
            Prior_FoodLion_per_Pound = mean(na.omit(GAmoneyPriorYr/GA.EQ.Prior.Yr)),
            Market_per_pound = mean(na.omit(Rmmoney/RM.EQ)),
            Prior_Market_per_pound = mean(na.omit(RMmoneyPriorYr/RM.EQ.Prior.Yr)))

onion_per_year2 <- pivot_longer(onion_per_year, cols = c(FoodLion_per_pound,Prior_FoodLion_per_Pound,Market_per_pound,Prior_Market_per_pound))
  
View(onion_per_year2)

png('onionyear.png',height = 500,width = 1000)
ggplot(onion_per_year2)+
  geom_col(aes(x = Year, y = value,fill = name),position= "dodge")+
  theme_bw()+
  theme(axis.text.x = element_text(angle = 60, hjust = 1))+
  scale_y_continuous(labels = scales::comma)+
  scale_fill_discrete(labels=c('Prior Yr. Food Lion', 
                               'Prior Yr. Rest of Market', 
                               'Current Food Lion',
                               'Current Rest of Market'))+
  labs(y = 'Price sold per Pound', 
       x = "Year", 
       title = "Food Lion VS Rest of Market Price per lb of Onion Sold year over year")
  dev.off()
```

```
## png 
##   2
```

```r
png("onion_avg.png",height = 500,width = 1000)
ggplot(oniontype2)+
  geom_col(aes(x = Sub.Category,y = value ,fill = name), position = "dodge")+
  facet_wrap(~Year, scale = "free_y")+
  theme_dark()+
  theme(axis.text.x = element_text(angle = 60, hjust = 1))+
  scale_y_continuous(labels = scales::comma)+
  scale_fill_discrete(labels=c('Prior Yr. Food Lion', 'Prior Yr. Rest of Market', 'Current Food Lion','Current Rest of Market'))+
  labs(y = 'Average Onion Sale Revennue Per Week', 
       x = "Type of Onion Sold", 
       title = "Comparison of Food Lion Average Revenue and Rest of Market Average Revenue")
dev.off()
```

```
## png 
##   2
```

```r
ggplot(oniontype4)+
  geom_col(aes(x = Sub.Category,y = value ,fill = name), position = "dodge")+
  facet_wrap(~Year, scale = "free_y")+
  theme_dark()+
  theme(axis.text.x = element_text(angle = 60, hjust = 1))+
  scale_y_continuous(labels = scales::comma)+
  scale_fill_discrete(labels=c('Prior Yr. Food Lion', 'Prior Yr. Rest of Market', 'Current Food Lion','Current Rest of Market'))
```

![](job-application_files/figure-html/plot_data-1.png)<!-- -->

```r
  labs(y = "Total revenue from onion sales per year", 
       x = "Type of Onion Sold", 
       title = "Comparison between Food Lion total revenue and Rest of market total   revenue")
```

```
## $y
## [1] "Total revenue from onion sales per year"
## 
## $x
## [1] "Type of Onion Sold"
## 
## $title
## [1] "Comparison between Food Lion total revenue and Rest of market total   revenue"
## 
## attr(,"class")
## [1] "labels"
```

```r
  png("onion")
  
  

  pander(onion_per_year)
```


-------------------------------------------------------------------------
 Year   FoodLion_per_pound   Prior_FoodLion_per_Pound   Market_per_pound 
------ -------------------- -------------------------- ------------------
 2018         1.818                   1.857                  3.612       

 2019          1.88                   1.818                  3.482       

 2020         1.895                    1.88                   3.36       
-------------------------------------------------------------------------

Table: Table continues below

 
------------------------
 Prior_Market_per_pound 
------------------------
         3.606          

         3.612          

         3.482          
------------------------

## Conclusions
