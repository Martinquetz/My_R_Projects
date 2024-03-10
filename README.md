# Fitness Product Analysis

## Table of Contents
- [Project Overview](#project-overview)
- [Data Sources and Description](#data-sources-and-description)
- [Tools](#tools)
- [Data Cleaning and Preparation](#data-cleaning-and-preparation)
- [Analyze](#analyze)
- [Results and Findings](#results-and-findings)
- [recommendations](#recommendations)
- [Limitations](#limitations)
- [References](#references)

### Project Overview
This is a report on a case study to analyze how a wellness tech company can improve its existing Smart products to improve ratings and sales. The objective is to analyze smart device data to gain insight into how consumers use their smart devices and to use discovered insights to guide the marketing strategy for the company (Bellabeat). Also, to present high-level recommendations for marketing strategy to the executive team.

### Data Sources and Description
The data was from a Kaggle open-source repository with 18 CSV files organized in a long format. The data is open-source with no copy-write, licensing, or privacy restrictions. According to the data's author, the datasets were gathered from respondents of a distributed survey via Amazon Mechanical Turk between 03.12.2016 and 05.12.2016. Thirty eligible Fitbit users consented to submit personal tracker data (although thirty-three unique user IDs were noticed), including minute-level output for physical activity, heart rate, and sleep monitoring.

### Tools
- Excel - Data preview
- RStudio - Data Cleaning, Transformation, Analysis, and Visualization. [Download RStudio here](https://rstudio-education.github.io/hopr/starting.html)

### Data Cleaning and Preparation
In preparation for the task, the following steps were taken:
1. Preview data in Excel
2. Load the relevant libraries into the RStudio session.
   ```r
     # To start the cleaning, relevant libraries must first be loaded into the session.
    library(tidyverse)
    library(lubridate)
    library("skimr")
    library("dplyr")
    library("here")
    library("janitor")
   ```
3. Load the relevant csv files into R
4. Data Cleaning & Transformation
5. Handle missing values

### Analyze
The resultant merged clean dataset was analyzed for:
- Summary statistics
  ```{r Changing the datatype and name of the date attribute, echo=FALSE}
    # Convert activityDates to date type and rename
    activity <- activity %>% mutate(date = mdy(ActivityDate))
    sleep <- sleep %>% mutate(date = mdy_hms(SleepDay))
    hr <- hr %>% mutate(date = mdy_hms(Time))
    weight <- weight %>% mutate(date = mdy_hms(Date))

    summary(activity)
  ```
- Identifying and transforming desired columns relevant to the task
  ```{r Created a new dataframe to contain only intrested attributes from activity_sleep dataframe, echo=FALSE}
    # Selecting interested columns, renaming them and saving them in a new dataframe.
    activityz <- activity_sleep %>% select(Id, date, TotalSteps, TotalDistance, Calories, SedentaryMinutes, TotalMinutesAsleep) %>% 
    rename(Steps=TotalSteps, distance=TotalDistance, Sedentary=SedentaryMinutes, Sleep=TotalMinutesAsleep) %>% 
    mutate(Sedentary_hour=Sedentary/60, Sleep_hour = Sleep/60) %>% mutate(Weekday = weekdays(date)) %>% clean_names()

    # Taking a peak into what our new dataframe looks like
    head(activityz)
    summary(activityz)
    str(activityz)
    #skim_without_charts(activityz)

  ```
- Merge multiple relevant files
  ```{r Merged activity and sleep files, echo=FALSE}
    # It's clear that the activity table already contained attributes of the intensity, steps, and calorie dataframes.
    # We'll merge the activity dataframe with the sleep dataframe only, excluding the hr & weight dataframe.
    activity_sleep <- merge(activity, sleep, by=c("Id", "date"))
    head(activity_sleep)
  ```
- Aggregated the dataset
  ```{r Aggregating the Activities data, echo=FALSE}
    # Summarize daily averages
    activityz_avg <- activityz %>% group_by(id, date, weekday) %>% 
    summarize(avg_steps=mean(steps), avg_dist=mean(distance), avg_cal=mean(calories),avg_sedentary=mean(sedentary_hour), avg_sleep=mean(sleep_hour)) %>% 
    arrange(-avg_steps, -avg_dist, -avg_cal)
- Visualize

### Results and Findings
- Surprisingly, the data did not include the demographic and geographic details of the respondents, making it difficult to determine if it represented our desired market segment (women).
- Also, it was observed that not all the respondents recorded all items of interest because “the variation between output represents the use of different types of Fitbit trackers and individual tracking behaviors/preferences,” as noted by the data’s author.
- Most respondents either can’t or choose not to record heart rate and weight data, as only 42% had data on heart rate and less than 24% documented weight data.
- The most popular tracked activities
  ```{r Chart of Tracked Activities by Number of Users, echo=TRUE}
    # Showing how respondents use non-Bella devices
    ggplot(data = tracked_activity) +
    geom_col(mapping = aes(act_type, id_count, fill=act_type)) +
    theme(axis.text.x = element_text(angle = 0)) +
    labs(title="Tracked Activities by Non-Bella devices by Number of Users")
  ```

  ![popularly_tracked_activities](https://github.com/Martinquetz/My_R_Projects/assets/92187086/7279d17e-e53f-4b75-9a80-cc404a303db1)

- A strong positive relationship exists between the number of steps taken, distance traveled, and the calories expended daily.
- There is a negative relationship between the number of steps taken, distance traveled, and the hours of sleep daily. This was surprising as the reverse was expected to be the case.
  ```{r Chart of Distance vs. Hours of Sleep, echo=FALSE}
    ggplot(data = activityz) + geom_point(mapping = aes(y = sleep_hour, x=distance)) +
      geom_smooth(mapping = aes(y = sleep_hour, x=distance)) +
      labs(title="Distance vs Sleep")
  ```

  ![dist_v_calories](https://github.com/Martinquetz/My_R_Projects/assets/92187086/fa649011-5320-4b85-b890-c2f531b9f6b6)

![steps_v_calories](https://github.com/Martinquetz/My_R_Projects/assets/92187086/8053c95e-8787-4a6f-ac14-9d561f94847d)


- participants, on average, slept in the longest on Wednesdays. The average most extended sleep hours were on weekdays and Wednesdays.
  ```{r Charts of Duration of Sleep in minutes by Weekday, echo=FALSE}
    ggplot(data = activityz_avg) +
      geom_col(mapping = aes(x= factor(weekday,levels = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday")), y= avg_sleep, fill=weekday)) +
      theme(axis.text.x = element_text(angle = 0)) +
      labs(title="Average Minutes of Sleep per Weekday")
  ```
- People tend to expend a significant amount of calories while working at their desks in the office. It is assumed most respondents work in the office, as explained by the sedentary vs. calories chart, which showed the highest average sedentary minutes during the week are Tuesday through Friday.
  ```{r Chart of Sedentary time in minutes by Calories, and Average Sedentary by Weekday, echo=FALSE}
    ggplot(data = activityz) + geom_point(mapping = aes(x=sedentary, y = calories)) +
      geom_smooth(mapping = aes(x=sedentary, y = calories)) +
      labs(title="Sedentary vs Calories")
  ```
![sedentary_v_calories](https://github.com/Martinquetz/My_R_Projects/assets/92187086/1d161219-c332-45a1-ac3e-0e63ed3f98b9)

![sed_by_day](https://github.com/Martinquetz/My_R_Projects/assets/92187086/ead447fb-d66b-4f09-a17d-34c0685c9fc5)


### Recommendations
The following recommendations are made on the assumption that:
- the used sample size is acceptable.
- the data was properly and randomly collected
- the data is representative of the organization's intended market

My recommendations are as follows:
1. The organization's app currently provides only a fraction of metrics users are interested in, such as step counts and burned calories. A redesign e of the app to provide a medium for data such as weight, heart rate, and sleep tracking is required to help the organization stay ahead of the competition.
2. Access to a robust data point from the upgraded app will empower the organization to use the information to make better tailor-made customer recommendations or even help them set goals through data analytics, thereby keeping their customers connected to the brand.
3. Since the organization intends to grow its global market position, it should invest in a more extensive survey (sample size of about 2400) to have a representative sample. This will help them understand their market better and improve their confidence in concluding any analysis.

### Limitations
The sample size is too small to represent the organization's market size goal. Missing data from some of the CSV files caused some files to be disregarded and null values dropped.

### References
Mobius: [background and datasets from kaggle.com can be assessed here](https://www.kaggle.com/datasets/arashnic/fitbit)
