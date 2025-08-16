# Bellabeat-Case-Study-Google
Bellabeat Case Study for Google Data Analysis Certificate

---
title: "Case Study: Bellabeat Fitness Data Analysis"
author: "Jeremie Diaz"
date: "2025-08-14"
output: html_document
---

## Company Background

Bellabeat is a company that develops fitness products for women. Their products include smart water bottles, fashionable fitness watches and jewelry, and yoga mats. Users can access their health data collected through these devices in the Bellabeat app.

Bellabeat’s co-founders would like to analyze data from non-Bellabeat fitness devices to see how consumers are using these products. The company hopes to use these insights to help guide new marketing strategies for the company.

## ASK

### Business Task

Analyze the tracker data to gain insights into how users use the app and discover trends and insights for the marketing team of Bellabeat.

### Expected Outputs

1.  Summary of the Task
2.  Description of all data sourced used
3.  Documentation of any cleaning or manipulation of data
4.  Summary of Analysis
5.  Supporting visualizations and key findings
6.  High-level content recommendations based on the analysis

### Stakeholders

1.  Urska Srsen, co-founder and CCO
2.  Mur, company mathematician
3.  Bellabeat Marketing Team

## Prepare

The data being used in this case study can be found here: [FitBit Fitness Tracker Data](https://www.kaggle.com/datasets/arashnic/fitbit) CC0: Public Domain, dataset made available through [Mobius](https://www.kaggle.com/arashnic)

The data analysis is done using the R Studio. The data set contains personal fitness tracker from 30 fitbit users. Thirty eligible Fitbit users consented to the submission of personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring. It includes information about daily activity, steps, and heart rate that can be used to explore users’ habits.

The data set contains 18 CSV files organized in long format using the ROCCC approach:

-   **Reliability - LOW:** The data comes from 30 fitbit users who consented to the submission of personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring.
-   **Original - LOW:** Third party data collect using Amazon Mechanical Turk.
-   **Comprehensive - MED:** The dataset contains multiple fields on daily activity intensity, calories used, daily steps taken, daily sleep time and weight record.
-   **Current - LOW:** This data is from March 2016 through May 2016. The data is not current, meaning that user habits may have changed over the years.
-   **Cited - LOW:** Data was collected from a third party, therefore unknown.

### Understanding the Metrics of Fitbite User Data

The Bellabeat app provide users health data related to their activity, sleep, stress, menstrual cycle, and mindfulness activities. This data can help users better understand their current habits and make healthy decisions.

Fitbit data contains user's information about their heart rate, sleep monitoring, daily activity by tracking totals for steps, intensity, distance, calories, and logged activities.

**Calories**

Fitbit estimates calorie burn by combining BMR (calories burned at rest) with activity data. BMR is calculated based on age, weight, height, and sex.

**Intensities**

Fitbit tracks activity intensity by monitoring heart rate and movement patterns. Intensities are often categorized into different levels, such as light, moderate, and vigorous. Heart Rate Variability (HRV) is used to assess sleep stages and can also be an indicator of activity intensity. Intensity values are categorized into:

```         
0 = Sedentary
1 = Light
2 = Moderate
3 = Very Active
```

**Sleep** 

Fitbit tracks sleep stages (light, deep, REM) by analyzing rate and movement. HRV is used to determine the transitions between these stages. The Fitbit app provides sleep score based on sleep duration, quality, and restoration. A higher sleep score indicates better sleep quality and duration.

**METs or Metabolic Equivalents**

Fitbit tracks METs showing intensity of physical activity. A MET value of 1 represents resting metabolism, while activities like brisk walking are around 3 - METs, and vigorous activities like running can be 10 METs or higher.

**Steps**

Fitbit tracks count steps using an accelerometer. The sensors detect movement in three axes (up/down, left/right, forward/backward) to estimate the number of steps taken.

**Heart Rate**

Fitbit tracks heart rate using optical sensors that detect blood volume changes in capillaries. This data is used to calculate heart rate in beats per minutes and also to determine heart rate zones (Fat Burn, Cardio, Peak) during exercise.

# Process

## Loading Datasets

To start the processing, import the datasets on activity, calories, intentsities, sleep, METs, heart rate, and weight. The analyst selected the datasets from March 12 to April 11, 2016.

```
activity <- read_csv("dailyActivity_merged.csv")
calories <- read_csv("hourlyCalories_merged.csv")
intensities <- read_csv("hourlyIntensities_merged.csv")
sleep <- read_csv("minuteSleep_merged.csv")
weight <- read_csv("weightLogInfo_merged.csv")
heart_rate <- read_csv("heartrate_seconds_merged.csv")
steps <- read_csv("hourlySteps_merged.csv")
```

## Loading Packages

```
library(tidyverse)
library(lubridate) 
library(dplyr)
library(ggplot2)
library(tidyr)
library(janitor)
```

## Checking datasets for NULL values, duplicates, etc. 
For example, the analyst checked the dimensions(columm and row), null values, and duplicates of the dataset activity.

```
dim(activity)
sum(is.na(activity))
sum(duplicated(activity))
```
Doing this with the rest of the datasets, not all have the same rows and columns which will affect the analysis later. Calories, Intensities, METs, and Steps have the same 1021 rows while Activity have 457 rows, Weight have 33 rows, Heart Rate have 143 rows, and Sleep have 467 rows. Only the dataset Weight has 31 null values. No dataset have duplicated values. 

## Converting Data Types

Upon inspecting the datasets, the time and dates are not properly attributed in the date time format. Also, the Id which will be the primary key to join the datasets is considered as numeric value. It have to be converted to character type so R would treat it such when making computations or analysis. 

```
activity <- activity %>% 
  mutate_at(vars(Id), as.character)
```

## Combining Dataframes

The analyst combined the activity, sleep, and weight datasets. Add the days of the week to the combined dataframe for analysis later. 

```
combined_data <-sleep %>%
  right_join(activity, by=c("Id","Day")) %>%
  left_join(weight, by=c("Id", "Day")) %>%
  mutate(Weekday = weekdays(as.Date(Day, "m/%d/%Y")))
```
The analyst also combined four datasets: Daily Calories, Daily Intensities, Daily METs, and Daily Steps. They have the same number of rows. 

# Analyze

The analyst focused on the user activity to uncover patterns affecting wellness outcomes. By visualizing key metrics such as steps, calories, active minutes, the analyst created meaningful insights that can answer the business task. 

## Trends in smart device usage

### Relationship Between Steps and Calories Burned

Key questions would be:

*Do users burn more calories as they walk?*
*What day do users mostly walk?*

```
ggplot(data=combined_data_final1, aes(x=TotalSteps, y=TotalCalories))+
  geom_point()+
  geom_smooth()+
  labs(
    title="Calories Burned and Steps",
    x="Total Steps",
    y="Total Calories Burned"
  )+
  theme_minimal()
```

Insights would be:

There is a positive relationship of how many steps taken to how many calories burned. As the users walk more, they burn calories even more. The trendline going upwards to the right suggest steps taken as a reliable predictor to burned calories.

```
ggplot(data=combined_data_final, aes(x=Weekday, y=TotalSteps))+
  geom_bar(stat="identity", fill="#fa2")+
  labs(
    title="Daily Steps",
    x="Days of the Week",
    y="Total Steps"
  )
```

Insights would be:

Most users are active walking during the weekends notably Saturday reaching with a total of 125 million steps. The least active walking is on Tuesday and Thursday reaching for a total of 75 million steps independently. This means that users attend to their wellness on weekends after a week of working.

### Other Relationships

There are also positive relationships of walking to intensity of metabolic equivalent, heart rate, and physical movement. 


### Percentage of Users' Active Minutes 

The dataset also presented how active the users can be, ranging from "Very Active" to "Sedentary." The analyst then computed them to check how many users are active in varying degrees. 

```
value <- c(very_active_mins, fairly_active_mins, lightly_active_mins, sedentary_mins)
labels <- c("very active", "fairly active", "lightly active", "sedentary")
piepercent <- (round(100*value/sum(value), 1))

pie(value, labels = piepercent, 
    main= "Active Profile of Users", col=rainbow(length(value)))
    legend("topright", c("Very Active", "Fairly Active", "Lightly Active", "Sedentary"),
       cex=0.5, fill = rainbow(length(value)))
```

Insights would be:

Most users spend 83.3% of their daily activity doing nothing while only 1.1% very active. 

### Days of the Week Users are Very Active

The analyst checked which day of the week the users are very active. 

```
ggplot(data=summary_activity_rate, aes(x=Weekday, y=AvgVeryActive))+
  geom_bar(stat="identity", fill="#fa2")+
  labs(
    title="Average Very Active",
    x="Day of the Week",
    y="Very Active (Mins)"
  )  
```

Insights would be:

Thursday and Saturday are days when the users are very active. This means that the users are active going into the weekends starting in Thursday but suddenly goes down on Sunday. 

# Act

**1. There is a positive relationship of how many steps taken to how many calories burned. As the users walk more, they burn calories even more. The trendline going upwards to the right suggest steps taken as a reliable predictor to burned calories.**

**2. Most users are active walking during the weekends notably Saturday reaching with a total of 125 million steps. The least active walking is on Tuesday and Thursday reaching for a total of 75 million steps independently. This means that users attend to their wellness on weekends after a week of working.**

**3. Most users spend 83.3% of their daily activity doing nothing while only 1.1% very active.**

**4. Thursday and Saturday are days when the users are very active. This means that the users are active going into the weekends starting in Thursday but suddenly goes down on Sunday.** 
