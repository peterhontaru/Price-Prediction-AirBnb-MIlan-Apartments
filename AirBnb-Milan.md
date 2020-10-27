---
    output:
      html_document:
        
        toc: true
        toc_depth: 2
        number_sections: true
        
        code_folding: hide 
        
        fig_height: 5
        fig_width: 9
        fig_align: "center"
    
        highlight: pygments
        theme: cerulean
        
        keep_md: true
        
    title: "AirBnb - Milan Apartments"
    subtitle: "Price Prediction Model"
    author: "by Peter Hontaru"
---



```r
knitr::opts_chunk$set(
    echo = TRUE, # show all code
    tidy = FALSE, # cleaner code printing
    size = "small", # smaller code
    
    fig.path = "figures/",# where the figures will end up
    out.width = "100%",

    message = FALSE,
    warning = FALSE
    )
```


# Introduction


## Why this dataset


I've always been fascinated with the italian culture (and food!). So much so, that this year I've started learning Italian. While looking to work on a data science project, I found this dataset on kaggle and thought it would be interesting to dive into it. This might also count as very good research before I visit the city as well, at some point in the future, in order to ensure I don't pay any more than I have to, and thus, be able to spend more money on italian pasta and local experiences.


## Problem statement


**We would like to predict the price at which an apartment should be rented, given a set of variables.**


## Who would benefit from this analysis?


1. **Help landlords calculate the optimal price for their apartment**
2. **Help tourists understand if they're getting a good deal**
3. **Potentially useful EDA & Regression data for beginner/intermediate data scientists looking to explore what's possible** (although in no way do I consider myself an expert)
4. **Provide a model that others might want to use if they were interested in other cities, as AirBnb data is fairly standardised**


## Summary:

* 1
* 2
* 3


## Dataset information
Dataset available from, pulled by Antonio and representative of the AirBnb data in July 2019.


# Data import, tidying, cleaning





Clean up data and add some new features:



```r
setwd("~/DS/AHT/Data")

#import data
raw_data <- read_csv("~/DS/AirBnb-Milano-Price-Prediction/raw data/Airbnb_Milan.csv")

#input the coordinates for the centre of Milano
Milano_City_Centre <- c(45.465, 9.185)

#tidy up data
raw_data <- raw_data %>%
        select(-X1, -room_type)%>%
        mutate(Region = as.factor(neighbourhood_cleansed),
               #new features
               total_price = daily_price + cleaning_fee,
               cleaning_fee_perc = round(cleaning_fee/total_price,2)*100,
               security_deposit_perc = round(security_deposit/total_price,2)*100,
               availability_30_perc = round(availability_30/30,2)*100,
               availability_60_perc = round(availability_60/60,2)*100,
               availability_90_perc = round(availability_90/90,2)*100,
               availability_365_perc = round(availability_365/365,2)*100)
              #distance_from_city_centre = c(c(latitude, longitude) - c(45.27, 9.11)))

#clear variables from the environment
rm(Milano_City_Centre)
```


Check the *structure* of the dataset:



```r
#check structure
glimpse(raw_data)
```

```
## Rows: 9,322
## Columns: 67
## $ id                               <dbl> 73892, 74169, 77958, 93025, 132705...
## $ host_id                          <dbl> 387110, 268127, 387110, 499743, 39...
## $ host_location                    <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1...
## $ host_response_time               <dbl> 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1...
## $ host_response_rate               <dbl> 57, 57, 57, 57, 57, 29, 57, 57, 57...
## $ host_is_superhost                <dbl> 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 0...
## $ host_total_listings_count        <dbl> 3, 1, 3, 1, 2, 8, 6, 2, 8, 22, 22,...
## $ host_has_profile_pic             <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1...
## $ host_identity_verified           <dbl> 1, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 0...
## $ neighbourhood_cleansed           <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1...
## $ zipcode                          <dbl> 20121, 20121, 20121, 20121, 20135,...
## $ latitude                         <dbl> 45.47015, 45.47169, 45.47117, 45.4...
## $ longitude                        <dbl> 9.19134, 9.18412, 9.19135, 9.19640...
## $ accommodates                     <dbl> 2, 4, 2, 3, 2, 2, 4, 4, 6, 5, 5, 4...
## $ bathrooms                        <dbl> 3, 3, 3, 3, 3, 3, 5, 3, 3, 3, 3, 3...
## $ bedrooms                         <dbl> 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1...
## $ beds                             <dbl> 1, 1, 1, 2, 1, 1, 2, 2, 4, 4, 4, 3...
## $ bed_type                         <dbl> 1, 1, 1, 1, 3, 1, 3, 1, 1, 1, 1, 1...
## $ daily_price                      <dbl> 94, 125, 100, 120, 70, 200, 700, 2...
## $ security_deposit                 <dbl> 1, 31, 1, 48, 13, 48, 1, 73, 13, 1...
## $ cleaning_fee                     <dbl> 45, 30, 45, 57, 45, 57, 1, 80, 1, ...
## $ guests_included                  <dbl> 1, 1, 1, 2, 1, 2, 1, 2, 1, 4, 4, 3...
## $ extra_people                     <dbl> 0, 0, 0, 40, 0, 20, 55, 25, 20, 20...
## $ minimum_nights                   <dbl> 2, 2, 2, 2, 30, 15, 1, 1, 1, 1, 1,...
## $ availability_30                  <dbl> 29, 0, 29, 10, 27, 0, 0, 30, 4, 15...
## $ availability_60                  <dbl> 59, 0, 59, 40, 57, 3, 0, 60, 10, 3...
## $ availability_90                  <dbl> 89, 9, 89, 70, 87, 33, 0, 90, 15, ...
## $ availability_365                 <dbl> 364, 284, 364, 160, 362, 308, 202,...
## $ number_of_reviews                <dbl> 84, 3, 70, 57, 44, 79, 72, 126, 37...
## $ review_scores_rating             <dbl> 94, 100, 97, 97, 90, 98, 96, 98, 9...
## $ review_scores_accuracy           <dbl> 9, 10, 9, 10, 9, 10, 10, 10, 10, 9...
## $ review_scores_cleanliness        <dbl> 9, 10, 10, 10, 9, 10, 10, 10, 10, ...
## $ review_scores_checkin            <dbl> 10, 10, 10, 10, 10, 10, 10, 10, 10...
## $ review_scores_communication      <dbl> 10, 10, 10, 10, 10, 10, 10, 10, 10...
## $ review_scores_location           <dbl> 10, 10, 10, 10, 9, 10, 10, 10, 10,...
## $ review_scores_value              <dbl> 9, 9, 9, 10, 9, 10, 9, 10, 9, 9, 9...
## $ instant_bookable                 <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1...
## $ cancellation_policy              <dbl> 1, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0...
## $ require_guest_profile_picture    <dbl> 1, 0, 1, 0, 1, 1, 0, 0, 0, 0, 0, 0...
## $ require_guest_phone_verification <dbl> 1, 0, 1, 0, 1, 1, 0, 0, 0, 0, 0, 0...
## $ TV                               <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1...
## $ WiFi                             <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1...
## $ Air_Condition                    <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1...
## $ Wheelchair_accessible            <dbl> 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0...
## $ Kitchen                          <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1...
## $ Breakfast                        <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
## $ Elevator                         <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1...
## $ Heating                          <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1...
## $ Washer                           <dbl> 0, 1, 0, 0, 1, 1, 1, 0, 1, 0, 1, 1...
## $ Iron                             <dbl> 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1...
## $ Host_greets_you                  <dbl> 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 1, 1...
## $ Paid_parking_on_premises         <dbl> 0, 0, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0...
## $ Luggage_dropoff_allowed          <dbl> 0, 0, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0...
## $ Long_term_stays_allowed          <dbl> 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0...
## $ Doorman                          <dbl> 0, 0, 0, 0, 0, 1, 1, 1, 0, 1, 1, 1...
## $ Pets_allowed                     <dbl> 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 1, 1...
## $ Smoking_allowed                  <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
## $ Suitable_for_events              <dbl> 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0...
## $ `24_hour_check_in`               <dbl> 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0...
## $ Region                           <fct> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1...
## $ total_price                      <dbl> 139, 155, 145, 177, 115, 257, 701,...
## $ cleaning_fee_perc                <dbl> 32, 19, 31, 32, 39, 22, 0, 24, 1, ...
## $ security_deposit_perc            <dbl> 1, 20, 1, 27, 11, 19, 0, 22, 13, 0...
## $ availability_30_perc             <dbl> 97, 0, 97, 33, 90, 0, 0, 100, 13, ...
## $ availability_60_perc             <dbl> 98, 0, 98, 67, 95, 5, 0, 100, 17, ...
## $ availability_90_perc             <dbl> 99, 10, 99, 78, 97, 37, 0, 100, 17...
## $ availability_365_perc            <dbl> 100, 78, 100, 44, 99, 84, 55, 100,...
```


Check for *Null*s:



```r
#check for nulls
raw_data %>% summarise_all(~ sum(is.null(.))) %>% sum()
```

```
## [1] 0
```


Check for *NA*s:



```r
#check NAs
raw_data %>% summarise_all(~ sum(is.na(.))) %>% sum()
```

```
## [1] 0
```


Now that we've cleaned up the table a bit and added some columns that will help with data exploration, we can start by exploring and understanding our data.


# Exploratory Data Analysis


## Correlation



```r
#select all to start
#raw_data_corr <- select_if(raw_data, is.numeric)  

#only a few specific ones
raw_data_corr <- raw_data%>%
        select(accommodates, bathrooms, bedrooms, beds, bed_type, daily_price, total_price, security_deposit, 
               cleaning_fee, review_scores_rating, guests_included, 
               extra_people, minimum_nights, host_total_listings_count, host_is_superhost)

# Compute a correlation matrix
corr <- round(cor(raw_data_corr),2)

# Compute a matrix of correlation p-values
p.mat <- cor_pmat(raw_data_corr)

# Visualize the correlation matrix
ggcorrplot(corr, method = "square", 
           ggtheme = ggplot2::theme_minimal, 
           title = "We can observe some clear patterns",
           
           outline.col = "black",
           colors = c("blue","white", "red"),
           
           lab = TRUE,
           lab_size = 2.5,
           digits = 2,
           
           type = "lower",
           legend = "",
           tl.cex = 8,
           #show insignificant ones as blank
           p.mat = p.mat,
           hc.order = TRUE,
           insig = "blank")
```

<img src="figures/unnamed-chunk-6-1.png" width="100%" />


Unsurprisingly, the total price correlated with the number of bedrooms, bathrooms and the number of people it accommodates. Note that there was a higher correlation with the number of bathrooms than the bedrooms. This could be due to the fact that some apartments have an artificially higher number of bedrooms, but not bathrooms.

Also, to be noted that there is a positive correlation between the superhost status and the rating. This might mean that superhost are more likely to be experienced landlords and thus be able to improve the customer experience.


## What does the distribution of the apartments look like?



```r
ggplot(raw_data, aes(longitude, latitude, col=Region)) +  
        geom_point()+
        labs(x=NULL, 
             y=NULL, 
             title="Region 1 represents the city centre, surrounded by the other 8 regions all around")+
        theme(axis.ticks.y = element_blank(),axis.text.y = element_blank(),
              axis.ticks.x = element_blank(),axis.text.x = element_blank(),
              #plot.title = element_text(lineheight=.8, face="bold", vjust=1),
              legend.position = "top")
```

<img src="figures/unnamed-chunk-7-1.png" width="100%" />


## What does the price distribution look like across all regions?



```r
ggplot(raw_data, aes(total_price))+
  geom_histogram(binwidth = 50,col = "black", fill = "red")+
  scale_y_log10()+
  scale_x_continuous(labels = dollar_format(), n.breaks = 15)+
  labs(x="Average Daily Price",
       y= "Number of Apartments",
       title = "While most of the data is below $394 (98%), there are some outliers all the way to >$3,000 (log scale)")
```

<img src="figures/unnamed-chunk-8-1.png" width="100%" />


## How does the Price differ between Regions?



```r
data_by_region <- raw_data%>%
        group_by(Region)%>%
        summarize(count = n(),
                  avg_price = mean(total_price),
                  med_price = median(total_price))

ggplot(data_by_region, aes(Region, count, fill = Region))+
        geom_col(col = "black")+
        geom_line(aes(Region, avg_price*10), group = 1, lty = 2)+
        geom_label(aes(Region, avg_price*10, label = paste("$", round(avg_price), sep = "")), fill = "black", col = "white")+
        labs(y = "Total apartments / Average Daily Price",
             x = "Region",
             title = "Region 1 has a higher average than all others due its central location and it is also the most common")+
        theme(legend.position = "none")    
```

<img src="figures/unnamed-chunk-9-1.png" width="100%" />


## What is the Price Variability of each Region?



```r
ggplot(raw_data, aes(reorder(Region, desc(total_price)), total_price, fill = Region))+
        geom_boxplot(varwidth = TRUE)+
        scale_y_continuous(labels = dollar_format())+
        labs(y = NULL,
             x = "Region",
             title = "We can observe some very high outliers for almost all of the 9 regions")+
        theme(legend.position = "none")
```

<img src="figures/unnamed-chunk-10-1.png" width="100%" />


Let's break down our price range into percentiles in order to better understand how much it varies:



```r
percentiles <- quantile(raw_data$total_price, c(0.50, 0.75, 0.80, 0.90, 0.95, 0.96, 0.97, 0.98, 0.99, 0.999, 1))

percentiles %>%
  kbl(caption = c("Custom percentile table"), 
      col.names = "Average Daily Price",
      align = c("c", "c")) %>%
  kable_paper("hover", full_width = F)
```

<table class=" lightable-paper lightable-hover" style='font-family: "Arial Narrow", arial, helvetica, sans-serif; width: auto !important; margin-left: auto; margin-right: auto;'>
<caption>Custom percentile table</caption>
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:center;"> Average Daily Price </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 50% </td>
   <td style="text-align:center;"> 124.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 75% </td>
   <td style="text-align:center;"> 160.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 80% </td>
   <td style="text-align:center;"> 174.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 90% </td>
   <td style="text-align:center;"> 221.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 95% </td>
   <td style="text-align:center;"> 284.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 96% </td>
   <td style="text-align:center;"> 304.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 97% </td>
   <td style="text-align:center;"> 335.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 98% </td>
   <td style="text-align:center;"> 394.58 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 99% </td>
   <td style="text-align:center;"> 567.74 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 99.9% </td>
   <td style="text-align:center;"> 3045.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 100% </td>
   <td style="text-align:center;"> 3100.00 </td>
  </tr>
</tbody>
</table>


## What does the Price Variability of each Region look like if we exclude the outliers?



```r
ggplot(raw_data, aes(reorder(Region, desc(total_price)), total_price, fill = Region))+
        geom_boxplot(outlier.shape = NA, varwidth = TRUE)+
        coord_cartesian(ylim=c(0,350))+
        scale_y_continuous(labels = dollar_format())+
        labs(y = NULL,
             x = "Region",
             title = "Region 1 also has a higher price range, not just a higher average and median \nRegion 2 and 9 seem to have a slightly lower median and IQR")+
        theme(legend.position = "none")
```

<img src="figures/unnamed-chunk-12-1.png" width="100%" />


## Could the Zipcode be a better predictor of Price than the Region?


Given the structure of Milan, it could be posssible that even within a region, prices differ significantly (ie. the inner side vs outer side). Thus, zipcodes might help us understand the data better.



```r
data_zipcode_quantile <- raw_data%>%
        group_by(zipcode)%>%
        summarize(zipcode_quantile_price = mean(total_price),
                  zipcode_quantile_count = n())%>%
        mutate(zipcode_quantile_group = ntile(zipcode_quantile_price, 5))

raw_data <- raw_data%>%
        left_join(data_zipcode_quantile, by = "zipcode")

data_zipcode_quantile_group <- data_zipcode_quantile %>%
        group_by(zipcode_quantile_group) %>% 
        summarize(count = sum(zipcode_quantile_count),
                  avg_price = sum(zipcode_quantile_price*zipcode_quantile_count)/count,
                  proportion = round(count/9322,2)*100)

ggplot(data = data_zipcode_quantile_group, aes(zipcode_quantile_group, count, fill = zipcode_quantile_group))+
        geom_col(col = "black")+
        geom_line(aes(zipcode_quantile_group, avg_price*15), col = "black", lty = 2)+
        geom_label(aes(zipcode_quantile_group, avg_price*15, label = paste("$", round(avg_price), sep = "")), fill = "black", col = "white")+
        scale_fill_gradient(low = "yellow", high = "red")+
        labs(y = "Total apartments",
             x = "Zipcode quantile",
             title = "The zipcode quantile with the highest prices has a higher proportion out of all apartments (45%)")+
        theme(legend.position = "none")
```

<img src="figures/unnamed-chunk-13-1.png" width="100%" />


Let's visualise this data on the map!



```r
ggplot(raw_data, aes(longitude, latitude)) +  
        geom_point(aes(colour = zipcode_quantile_group), size = 2)+
        labs(x="", 
             y="",
             col = "Zipcode Quantile Group",
             title="Rather than Region, price difference might be better predicted by proximity to the City Centre\nThis implies that the inner areas of each Region has a higher price than the outer areas")+
        scale_colour_gradient(low = "yellow", high = "red")+
        theme(legend.position = "top",
              axis.ticks.y = element_blank(),axis.text.y = element_blank(),
              axis.ticks.x = element_blank(),axis.text.x = element_blank())
```

<img src="figures/unnamed-chunk-14-1.png" width="100%" />

```r
              #plot.title = element_text(lineheight=.8, face="bold", vjust=1))
```


Irrespective of region, the highest prices seem to be those within the zipcodes closer to the city centre (Region 1 is an exception because it is placed in the City Centre). An even better alternative would've been proximity to the city centre based on latitude and longitude. 

In simple terms, this means that if your apartment is located within these zipcodes, you could charge more. However, price alone is not that interesting, if no one books the apartment. This brings us to the next question:


## What does the availability look like over a year, irrespective of area? 



```r
 ggplot(raw_data, aes(x = availability_365, fill = ..count..)) +  
        geom_histogram(aes(y = (..count..)/sum(..count..)), binwidth = 10, col = "black") + 
        scale_fill_gradient(low="yellow", high="red")+
        scale_y_continuous(labels = percent)+
   labs(x = "Days available in the next year (buckets of 10 days)",
        y = "% of all apartments available",
        title = "There is a trend for houses to be either available for too long or too little")
```

<img src="figures/unnamed-chunk-15-1.png" width="100%" />


There could be a multitude of reasons for this to happen whether due to popularity or places being closed. For the ones available for almost a full year, could be lack of popularity or a new addition.


## Are there any differences between the Regions/Zipcode Quantiles? {.tabset .tabset-fade .tabset-pills}


### by Region {-}



```r
#is there anything we can do with this?
#2) max latitude is a great idea to use for text on map
ggplot(raw_data, aes(longitude, latitude)) +  
  geom_jitter(aes(colour = availability_365_perc), alpha = 0.4, width = 0.01, height = 0.01)+
  labs(x="", 
       y="",
       col = "Availability (%) in the next 365 days",
       title="There doesn't seem to be a trend in availability between the different areas",
       subtitle = "Data points for each region are much narrower on the map but they were spread out so that all points can be observed")+
  scale_colour_gradient(low = "yellow", high = "red")+
  facet_wrap(.~Region)+
  theme(axis.ticks.y = element_blank(),axis.text.y = element_blank(),
        axis.ticks.x = element_blank(),axis.text.x = element_blank(),
        plot.title = element_text(lineheight=.8, face="bold", vjust=1),
        legend.position = "top")
```

<img src="figures/unnamed-chunk-16-1.png" width="100%" />


### by Zipcode Quantile Group {-}



```r
ggplot(raw_data, aes(longitude, latitude)) +  
  geom_jitter(aes(colour = availability_365_perc), alpha = 0.4, height = 0.01, width = 0.01)+
  labs(x="", 
       y="",
       col = "Availability (%) in the next 365 days",
       title="There doesn't seem to be a trend in availability between the different areas",
       subtitle = "Data points for each region are much narrower on the map but they were spread out so that all points can be observed")+
  scale_colour_gradient(low = "yellow", high = "red")+
  facet_wrap(.~zipcode_quantile_group)+
  theme(axis.ticks.y = element_blank(),axis.text.y = element_blank(),
        axis.ticks.x = element_blank(),axis.text.x = element_blank(),
        plot.title = element_text(lineheight=.8, face="bold", vjust=1),
        legend.position = "top")
```

<img src="figures/unnamed-chunk-17-1.png" width="100%" />



There doesn't seem to be a clear pattern, as there is a fairly equal mix of houses with high **and** low availability between all regions. Let's have a quick look at the aggregated data.


## What does the aggregated availability look like over 365 days? {.tabset .tabset-fade .tabset-pills}


### by Region {-}



```r
data_availability_by_region <- raw_data%>%
  group_by(Region)%>%
  summarize(count = n(),
            availability = (sum(availability_365)/count)/365*100,
            diff_from_mean = availability - mean(raw_data$availability_365_perc))

ggplot(data = data_availability_by_region, aes(Region, diff_from_mean, fill = diff_from_mean >= 0))+
        geom_col(col = "black")+
        scale_fill_manual(values = c("dark red", "#009E73"))+
        labs(y = "% difference from Mean Availability",
             x = "Region",
             title = "City Centre apartments have a slightly higher availability than others, perhaps due to the higher price")+
        theme(legend.position = "none")
```

<img src="figures/unnamed-chunk-18-1.png" width="100%" />


### by Zipcode Quantile Group {-}



```r
data_availability_by_zipcode_quantile_group <- raw_data %>%
  group_by(zipcode_quantile_group) %>%
  summarize(count = n(),
            availability = (sum(availability_365)/count)/365*100,
            diff_from_mean = availability - mean(raw_data$availability_365_perc))

ggplot(data = data_availability_by_zipcode_quantile_group, aes(zipcode_quantile_group, diff_from_mean, fill = diff_from_mean >= 0))+
        geom_col(col = "black")+
        scale_fill_manual(values = c("dark red", "#009E73"))+
        labs(y = "% difference from Mean Availability",
             x = "Region",
             title = "City Centre apartments have a slightly higher availability than others, perhaps due to the higher price")+
        theme(legend.position = "none")
```

<img src="figures/unnamed-chunk-19-1.png" width="100%" />

```r
#maybe add n = sample size
```


From the ***by Zipcode Quantile Group*** graph, we can see that people tend to book closer to the city centre (although there is a much higher sample size). At the same time, we can see from the ***by Region*** graph that they don't necessarily want to be in the Region 1, where the prices are also considerably higher.


## What type of Extra Services are there available during an AirBnb stay in Milan?



```r
data_Extras <- raw_data %>%
  select(TV:"24_hour_check_in")%>%
  gather("Extras", "Status")%>%
  group_by(Extras)%>%
  summarize(Count = n(),
            Availability = round(sum(Status)/Count,2)*100)%>%
  mutate(Extras = fct_reorder(Extras, Availability))

ggplot(data_Extras, aes(Extras, Availability))+
  geom_col(aes(fill=Availability), col = "black")+
  scale_fill_gradient(low="red", high="yellow")+
  theme(legend.position = "none")+
  labs(x=NULL,
       y= "Availability (%)",
       title = "Since most places have the services we'd normally want and need (ie. Heating/Kitchen/WiFi), \nwe expect them to not be very important in our price prediction model")+
  coord_flip()
```

<img src="figures/unnamed-chunk-20-1.png" width="100%" />


## Are there any differences in Survey Ratings? {.tabset .tabset-fade .tabset-pills}


### Full scale (0-100) {-}



```r
ggplot(raw_data, aes(review_scores_rating, Region, fill = factor(stat(quantile)))) +
  stat_density_ridges(
    geom = "density_ridges_gradient", calc_ecdf = TRUE,
    quantiles = 4, quantile_lines = TRUE) +
  labs(x = "Average Rating",
       title = "No major differences between the different regions")+
  scale_fill_viridis_d(name = "Quartiles")
```

<img src="figures/unnamed-chunk-21-1.png" width="100%" />


### Zoomed in axis (50-100) {-}



```r
ggplot(raw_data, aes(review_scores_rating, Region, fill = factor(stat(quantile)))) +
  stat_density_ridges(
    geom = "density_ridges_gradient", calc_ecdf = TRUE,
    quantiles = 4, quantile_lines = TRUE) +
  scale_fill_viridis_d(name = "Quartiles")+
  labs(title = "No major differences between the different regions",
       x= "Average Rating")+
  coord_cartesian(xlim = c(70,100))
```

<img src="figures/unnamed-chunk-22-1.png" width="100%" />


# Regression Model


## Prepare our model


We need to remove the variables that were related to the price (ie. cleaning_fee_perc and security_perc) as well as the ones that are simply variations of a specific variable (ie. we shouldn't keep both absolute availability and relative availability due to multicolinearity).



```r
# Set seed for reproducibility
set.seed(123)

#might want to take these out if we use the perc values
raw_data_model <- raw_data %>%
  select (-availability_30, -availability_60, -availability_90, -availability_365, 
          -id, -host_id, 
          -cleaning_fee_perc, -security_deposit_perc,
          -cleaning_fee, -security_deposit, -daily_price,
          -zipcode_quantile_count, -zipcode_quantile_price)%>%
  filter (total_price <= quantile(raw_data$total_price, probs = c(0.98)))
```


We've seen that while most of the data is under \$400 (>98%), there are some extreme outliers up to \$3,000 that might affect our model. To improve the accuracy of our model, we can remove the highest 2% of the prices as we do not want to tailor this model to the luxury market for a couple reasons: 

* if you're a tourist, you're unlikely to be checking online prices if you can even consider paying ~$3,000 a night
* similarly, as a landlord, you probably don't need to compare your price to others'
* there is very limited data
* AirBnb's target market isn't focused on luxurious escapes


## Split Data


We need to split the dataset so that we have a train dataset of 75% of the raw data to train our prediction model on and 25% of the raw data to test it.



```r
split <- createDataPartition(raw_data_model$total_price,
                             times = 1,
                             p =0.75, 
                             list = FALSE)

train_data <- raw_data_model[split,]
test_data <- raw_data_model[-split,]


y <- train_data$total_price
```


## Pre-process


I use some basic standardisation offered by the caret package such as **centering** (subtract mean from values) and **scaling** (divide values by standard deviation) the data.



```r
#take out price temporarily as we do not want this to be processed
train_data <- train_data %>%
  select (-total_price)

#center and scale our data
preProcess_range_model <- preProcess(train_data, method=c("center", "scale"))

train_data <- predict(preProcess_range_model, newdata = train_data)
test_data <- predict(preProcess_range_model, newdata = test_data)
 
#Append the Y variable back on with original values
train_data$total_price <- y
```


## Resampling


We will perform a 10-fold Cross Validation 5 times. 

This means we will be dividing the training dataset randomly into 10 parts and then using each of 10 parts as testing dataset for the model trained on remaining 9. Essentially, we are "pretending" that some of our data is new and use the rest of the data to model on. We then take the average of the 10 error terms thus obtained and repeat this process 5 times. Doing it more than once will give a more realistic sense of how the model will perform on new data.



```r
train.control <- trainControl(method = "repeatedcv", 
                              number = 2, #10
                              repeats = 2, #5
                              search = "random")
```


## Model 1 - Stepwise Regression



```r
#run them all in paralel
cl <- makeCluster(3, type = "SOCK")

#register cluster train in paralel
registerDoSNOW(cl)

#train model
step_model <- train(total_price ~ .,
                  data = train_data,
                  method = "lmStepAIC",
                  trControl = train.control)

#shut the instances of R down
stopCluster(cl)
```



```r
#get predictions
predictions <- predict(step_model, test_data)

#create dataset
test_data2 <- test_data %>%
  mutate(predicted_total_price = predictions,
         actual_total_price = total_price,
         Residuals = predicted_total_price - actual_total_price)%>%
  select(predicted_total_price, actual_total_price, Residuals)

#visualise
ggplot(test_data2, aes(actual_total_price, predicted_total_price))+
  geom_point(alpha = 0.5, color = "blue")+
  geom_smooth(method = "loess", col = "red", se = FALSE)+
  scale_x_continuous(labels = dollar_format())+
  scale_y_continuous(labels = dollar_format())+
  labs(x = "Actual Price",
       y = "Predicted Price")
```

<img src="figures/unnamed-chunk-28-1.png" width="100%" />


## Model 2 - Gradient Boosting Machine



```r
#run them all in paralel
cl <- makeCluster(3, type = "SOCK")

#register cluster train in paralel
registerDoSNOW(cl)

#train model
gbm_model <- train(total_price ~ .,
                  data = train_data,
                  method = "gbm",
                  trControl = train.control)

#shut the instances of R down
stopCluster(cl)

#show model
gbm_model$results
```



```r
#get predictions
predictions <- predict(gbm_model, test_data)

#create dataset
test_data2 <- test_data %>%
  mutate(predicted_total_price = predictions,
         actual_total_price = total_price,
         Residuals = predicted_total_price - actual_total_price)%>%
  select(predicted_total_price, actual_total_price, Residuals)

#visualise
ggplot(test_data2, aes(actual_total_price, predicted_total_price))+
  geom_point(alpha = 0.5, color = "blue")+
  geom_smooth(method = "loess", col = "red", se = FALSE)+
  scale_x_continuous(labels = dollar_format())+
  scale_y_continuous(labels = dollar_format())+
  labs(x = "Actual Price",
       y = "Predicted Price")
```

<img src="figures/unnamed-chunk-30-1.png" width="100%" />


## Model 3 - Random Forrest Regression



```r
#run them all in paralel
cl <- makeCluster(3, type = "SOCK")

#register cluster train in paralel
registerDoSNOW(cl)

#train model
rf_model <- train(total_price ~ .,
                  data = train_data,
                  method = "rf",
                  trControl = train.control)

#shut the instances of R down
stopCluster(cl)
```



```r
#get predictions
predictions <- predict(rf_model, test_data)

#create dataset
test_data2 <- test_data %>%
  mutate(predicted_total_price = predictions,
         actual_total_price = total_price,
         Residuals = predicted_total_price - actual_total_price)%>%
  select(predicted_total_price, actual_total_price, Residuals)

#visualise
ggplot(test_data2, aes(actual_total_price, predicted_total_price))+
  geom_point(alpha = 0.5, color = "blue")+
  geom_smooth(method = "loess", col = "red", se = FALSE)+
  scale_x_continuous(labels = dollar_format())+
  scale_y_continuous(labels = dollar_format())+
  labs(x = "Actual Price",
       y = "Predicted Price")
```

<img src="figures/unnamed-chunk-32-1.png" width="100%" />


## Compare the three Models



```r
model_list <- list(lm = step_model, gbm = gbm_model, rf = rf_model)

model_comparison <- resamples(model_list)

summary(model_comparison)
```

```
## 
## Call:
## summary.resamples(object = model_comparison)
## 
## Models: lm, gbm, rf 
## Number of resamples: 4 
## 
## MAE 
##         Min.  1st Qu.   Median     Mean  3rd Qu.     Max. NA's
## lm  31.11526 31.13297 31.38807 31.40433 31.65942 31.72591    0
## gbm 30.14589 30.27200 30.41091 30.37597 30.51488 30.53615    0
## rf  29.59393 29.91580 30.20295 30.18282 30.46997 30.73144    0
## 
## RMSE 
##         Min.  1st Qu.   Median     Mean  3rd Qu.     Max. NA's
## lm  44.01209 44.15731 44.22836 44.33571 44.40676 44.87405    0
## gbm 42.37121 42.59590 42.79524 42.80099 43.00034 43.24230    0
## rf  41.05268 42.34422 42.90531 42.79670 43.35778 44.32349    0
## 
## Rsquared 
##          Min.   1st Qu.    Median      Mean   3rd Qu.      Max. NA's
## lm  0.3968808 0.4113166 0.4253097 0.4234260 0.4374191 0.4462037    0
## gbm 0.4502930 0.4603678 0.4668450 0.4664366 0.4729138 0.4817634    0
## rf  0.4389916 0.4439752 0.4650523 0.4648461 0.4859233 0.4902883    0
```


We can see that the Random Forrest is the most optimal model across the range of our data due to:

* the best/lowest MAE and RSE
* the best/highest Rsquared

We can also compare them quantitatively and see if the results of the Random Forrest model is significantly better than the other two.


Random Forrest versus Stepwise:



```r
compare_models(rf_model, step_model)
```

```
## 
## 	One Sample t-test
## 
## data:  x
## t = -2.298, df = 3, p-value = 0.1052
## alternative hypothesis: true mean is not equal to 0
## 95 percent confidence interval:
##  -3.6704050  0.5923731
## sample estimates:
## mean of x 
## -1.539016
```


Random Forrest versus Gradient Boosting Machine:



```r
compare_models(rf_model, gbm_model)
```

```
## 
## 	One Sample t-test
## 
## data:  x
## t = -0.0050679, df = 3, p-value = 0.9963
## alternative hypothesis: true mean is not equal to 0
## 95 percent confidence interval:
##  -2.703135  2.694540
## sample estimates:
##    mean of x 
## -0.004297765
```


These results also confirm that the Random Forrest model is significantly better than the other two.


## How many predictors did the most optimal model have?



```r
plot(rf_model, 
     main = "The most optimal model was that with 18 predictors")
```

<img src="figures/unnamed-chunk-36-1.png" width="100%" />


## What were the most important variables?



```r
imp <- varImp(rf_model)

plot(imp, top = 20,
     main = "Top 20 variables ranked by importance")
```

<img src="figures/unnamed-chunk-37-1.png" width="100%" />


We can see that the most important factors were:

* bathrooms
* bedrooms
* accommodates
* zipcode_quantile_group (rather than region)

This shouldn't come as a surprise, as we expect location and the number of people it accommodates to be at the top. It might be surprising that the number of bathrooms was more important than bedrooms. This could be due to the number of bathroom representing a more accurate way to track the size of the property, as it is common make 4 bedrooms out of a 2 bedroom apartment, but not many artificially increase the number of the bathrooms. In other words, an apartment with 6 bedrooms is not necessarily very large, as it could be due to the fact that each room was split into two. However, chances are that if an apartment has 6 bathrooms, it is quite large!


## Visualising the distribution of the Residuals



```r
ggplot(test_data2, aes(Residuals))+
  geom_histogram(binwidth = 10,col = "black", fill = "red")+
  scale_x_continuous(labels = dollar_format(), n.breaks = 10)+
  labs(x="Prediction error (Residual)",
       y= "Occurences",
       title = "It is more common to overpredict the Daily Price rather than underpredict \nMost data has an error of +/- 30$ but there are some outliers of over $100 and under $200",
       subtitle = "Data is displayed in buckets of $10")
```

<img src="figures/unnamed-chunk-38-1.png" width="100%" />


## Visualising the Random Forrest Model



```r
#get predictions
predictions <- predict(rf_model, test_data)

#create dataset
test_data2 <- test_data %>%
  mutate(predicted_total_price = predictions,
         actual_total_price = total_price,
         Residuals = predicted_total_price - actual_total_price)%>%
  select(predicted_total_price, actual_total_price, Residuals)

#visualise
ggplot(test_data2, aes(actual_total_price, Residuals))+
  geom_hline(yintercept = 0, size = 3, color = "grey52")+
  geom_point(alpha = 0.5, color = "blue")+
  geom_smooth(method = "loess", col = "red", se = FALSE)+
  scale_x_continuous(labels = dollar_format())+
  labs(title = "While the average prediction error is $30, there is a tendency to underpredict houses of over $250",
       subtitle = "While the model isn't precide enough for an exact prediction, it is enough to provide an approximate range",
       x = "Actual Price")
```

<img src="figures/unnamed-chunk-39-1.png" width="100%" />


## Potential reasons for the low Rsquared:


While this model has a similar or higher Rsquared values to that of other AirBnb datasets (Milano / New York), the value isn't perfect, which is shown by an average error of around $30. A few reasons for a "medium" Rsquared could be:

* the size of the apartment (squared feet/meters) is not included in the dataset and thus, in the model; this might be a better predictor than the number of bedrooms
* human behaviour/decision making is known to be hard to predict accurately
* pictures of the property not included - when is the last time we booked an apartment without seeing a picture? 
* on top of pictures, we might also be influenced by the message within the property listing written by the landlord which this dataset does not account for


## How to improve the project?

* build a dashboard that would inform potential landlords of what price range to use
* pull data from a different period of the year, as we know prices are seasonal
* look at other types of accommodation, as this dataset only includes apartments
* work upon the data limitations mentioned above which affected the Rsquared


# Other things to look into

* frequency plot of errors
* residuals vs actuals

* compare against that of others with same dataset (python or R)
* benchmark against others airbnb datasets (New York)

* AirBNB picture
* picture of milano somewhere

* summary