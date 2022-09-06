# BellaBeat Case Study
### Google Data Analytics Capstone Project
![bellabeat](https://user-images.githubusercontent.com/72343428/185792124-e7fe7db9-b2e6-47c8-bdcf-55da42c3b54f.png)

## Contents  
 * [Introduction](#intro)
 * [Ask](#ask)
 * [Prepare](#pre)
 * [Processing Data](#process)
 * [Analysing Data](#analyse)
 * [Sharing Our Findings](#share)
 * [Act](#act)
 * [Reference](#ref)
 * [Appendix](#appendix)

<a name="intro"/>

## Introduction
### Company Background
BellaBeat was founded by Urška Sršen and Sandro Mur in 2013. It is a high-tech company that manufactures health-focused smart products and collects data on the activity, sleeps, stress and reproductive health of its customers in order to provide information regarding their health and habits.

<a name="ask"/>

## Ask
### Business Task
Identifying the trends in smart device usage and providing recommendations that can help stakeholders to make data-driven decisions regarding the marketing strategy and discover new potentials for the company.
### Key Stakeholders
 * Urška Sršen: BellaBeat Co-founder and Chief Creative Officer (CCO)
 * Sandro Mur: Mathematician, BellaBeat Co-founder and Chief Executive Officer (CEO)
 * BellaBeat Marketing Team

<a name="pre"/>

## Prepare
### About the Data
The dataset[[1]](#1) I used is FitBit Fitness Tracker Data, which is available on Kaggle and provided by MÖBIUS. <br />
It is collected from eligible Fitbit users who consented to submit personal tracker data via a survey distributed by Amazon Mechanical Turk between 03.12.2016 - 05.12.2016. <br />
This dataset has overall 18 CSV files, including information about the minute-level output for physical activity, heart rate, weight and sleep monitoring between 04.12.2016 - 15.12.2016 (31 days period).
### Data Storage and Organization
After downloading the dataset, I opened all the CSV files in the spreadsheet (EXCEL) to see how are the data structured. <br />
The dataset includes wide and narrow data. Each user has a unique ID so that we can distinguish between them. I noticed that 'dailyCalories_merged.csv', 'dailyIntensities_merged.csv' and 'dailySteps_merged.csv' are all included in 'dailyActivity_merged.csv', so I decided not to use these three files. <br />
I also discovered that some tables have too many rows. Hence using BigQuery to analyse the data will be a better choice than the spreadsheet. <br />
So I stored the below files in BigQuery under the project **BellaBeat Case Study** (ID: `bellabeat-timilin`), dataset **Fit-Data**:
|   **Table Name**   |       **CSV File Name**      |                                                  **Discription**                                                  |
|:------------------:|:----------------------------:|:-----------------------------------------------------------------------------------------------------------------|
|   daily_activity   |   dailyActivity_merged.csv   | Daily activity over 31 days period of 33 users. Tracking the daily steps, distance, intensities, and calories.    |
|      heartrate     | heartrate_seconds_merged.csv | 14 users' heartrate during the day.                                                                               |
|     hourly_cal     |   hourlyCalories_merged.csv  | Hourly calories burnt over 31 days period of 33 users.                                                            |
| hourly_intensities | hourlyIntensities_merged.csv | Hourly total intensity over 31 days of 33 users.                                                                  |
|     hourly_step    |    hourlySteps_merged.csv    | Hourly steps over 31 days of 33 users.                                                                            |
|      sleep_day     |      sleepDay_merged.csv     | 24 users' sleeping info over 31 days, including total sleep records, total minutes asleep and total time in bed.  |
|     weight_log     |   weightLogInfo_merged.csv   | 8 users' weight in Kg and Pounds, and their BMI, Fat, Manual Entry or not recorded in a day, over 31 days.        |
### Credibility and Integrity
To determine the credibility and integrity of the data, I will use the **'ROCCC'** system:
  * **Reliability**: The data is **not** reliable since we are not sure what the marginal error is, and a small sample size (~30 participants) has been used. So our analysis might not be true for the whole population.
  * **Originality**: The data is **not** original since it was collected by Amazon Mechanical Turk and made available through MÖBIUS but has been checked against [[1]](#1).
  * **Comprehensiveness**: The data is **not** comprehensive because we have no information regarding whether our samples are randomly collected or not. This might lead to bias.
  * **Current**: The data is **not** current since it was collected in 2016. Therefore our analysis cannot represent the current trend in smart device usage.
  * **Cited**: It is cited. <br />
As a result, our data do not satisfy the 'ROCCC' system. This means that using this dataset, we are **unable** to provide reliable and comprehensive recommendations for BellaBeats. Hence, our analysis can only act as directions which should be verified through a more reliable dataset.

<a name="process"/>

## Processing Data
### Uploading and Transforming Data
When I tried to upload the above CSV files into BigQuery, only the schema for *'dailyIntensities_merged.csv'* could be detected automatically. For other files, the below error occurs:
```
Failed to create table: Error while reading data, error message: Could not parse '4/12/2016 12:00:00 AM' as TIMESTAMP for field ActivityHour (position 1) starting at location 26 with message 'Invalid time zone: AM'
```
Therefore, I had to manually define the schema for them and let `ActivityHour` temporarily be `STRING` instead of `TIMESTAMP`. <br />
Now, having the files uploaded. I checked the datatypes for each table. Since we temporarily set the `ActivityHour/Time/SleepDay` as `STING` for some tables, we can now set them as `DATETIME` using the following code:
```sql
-- eg. For heartrate table
CREATE OR REPLACE TABLE `bellabeat-timilin.Fit_Data.heartrate`
AS
SELECT 
      Id,
      PARSE_DATETIME("%m/%d/%Y %l:%M:%S %p",Time) AS Time,
      Value
FROM 
     `bellabeat-timilin.Fit_Data.heartrate` 
```
And similarly we can do this for `hourly_cal`, `hourly_intensities`, `hourly_step`, `sleep_day` and `weight_log` tables. <br />
*(See code in the Appendix)*

### Data Cleaning
First, we check whether there is any NULL Value in our dataset:
```sql
-- eg. For daily_activity table
SELECT
      *
FROM
      `bellabeat-timilin.Fit_Data.daily_activity`
WHERE
      Id IS NULL OR
      ActivityDate IS NULL OR
      TotalSteps IS NULL OR
      TotalDistance IS NULL OR
      TrackerDistance	IS NULL OR
      LoggedActivitiesDistance IS NULL OR
      VeryActiveDistance	IS NULL OR
      ModeratelyActiveDistance	IS NULL OR
      LightActiveDistance	IS NULL OR
      SedentaryActiveDistance	IS NULL OR
      VeryActiveMinutes	IS NULL OR
      FairlyActiveMinutes	IS NULL OR
      LightlyActiveMinutes	IS NULL OR	
      SedentaryMinutes	IS NULL OR
      Calories	IS NULL
```
We only found NULL Values in the `weight_log` table, under the `Fat` column. These NULL Values can be ignored if we do not want to use information regarding users' fat. <br />
Next, we search for any duplicates:
```sql
-- eg. For sleep_day table
SELECT 
      Id,
      Time,
      TotalSleepRecords,
      TotalMinutesAsleep,
      TotalTimeInBed,
      COUNT(*) AS No_Of_Dup
FROM
     `bellabeat-timilin.Fit_Data.sleep_day`
GROUP BY
     Id,
     Time,
     TotalSleepRecords,
     TotalMinutesAsleep,
     TotalTimeInBed
HAVING No_Of_Dup > 1
```
And we can actually find duplicates in the `sleep_day` table:
|   **Id**   |       **Time**      | **TotalSleepRecords** | **TotalMinutesAsleep** | **TotalTimeInBed** | **No_Of_Dup** |
|:----------:|:-------------------:| ---------------------:| ----------------------:| ------------------:| -------------:|
| 4388161847 | 2016-05-05T00:00:00 |                     1 |                    471 |                495 |             2 |
| 8378563200 | 2016-04-25T00:00:00 |                     1 |                    388 |                402 |             2 |
| 4702921684 | 2016-05-07T00:00:00 |                     1 |                    520 |                543 |             2 |

Hence, we need to remove three rows in total from this table. To do this, we can go back temporarily to EXCEL, and by filtering for these rows, we can easily delete the duplicates. I chose to do this in EXCEL because we only have three rows to delete, but if we have more, it is better to use R. <br />
This gives a new table:
|   **Table Name**  |       **CSV File Name**       |       **Discription**        |
|:-----------------:|:-----------------------------:|:-----------------------------|
|  sleep_day_new    |  sleepDay_merged_cleaned.csv  | Cleaned version of sleep_day |

*(See codes in the Appendix)*

<a name="analyse"/>

## Analysing Data
### Checking the # of users
Before we start analysing our data, it is always good to ensure how many unique IDs are in different tables because not all participants answer all the questions.
```sql
-- eg. For daily_activity table
SELECT
    COUNT(DISTINCT Id) AS No_of_users
FROM `bellabeat-timilin.Fit_Data.daily_activity`
```
|  **Table Name** | daily_activity | heartrate | hourly_cal | hourly_intensities | hourly_step | sleep_day_new | weight_log |
|:---------------:|:--------------:|:---------:|:----------:|:------------------:|:-----------:|:-------------:|:----------:|
| **No_of_users** |       33       |     14    |     33     |         33         |      33     |       24      |      8     |

*(See code in the Appendix)*
### Average Steps VS Average Active Minutes
The Centers for Disease Control and Prevention (CDC) suggest that adults should aim for 10000 steps per day [[2]](#2). But recently, there have been researches [[3]](#3) which also said that 7000 steps are already enough. Therefore, we can first investigate how many users walk more than 7000 steps a day on average.
```sql
SELECT 
      DISTINCT Id,
      ROUND(AVG(TotalSteps),3) AS daily_avg_step
FROM
      `bellabeat-timilin.Fit_Data.daily_activity`
GROUP BY
      Id
HAVING daily_avg_step >= 7000
```
This gives us back 20 users out of 33, which is *60.6%* of our sample population. <br />
However, does having more steps means that you were more active and did more sport than others who completed fewer steps? Not necessarily, since step numbers can be faked. Therefore, we should look at the active minutes. <br />
An adult should aim for at least 30 "active minutes" per day [[4]](#4). But how can we link this with our dataset? Actually, Fitbit measures active minutes, but in our data, it is split into "VeryActiveMinutes", "FairlyActiveMinutes", "LightlyActiveMinutes" and "SedentaryMinutes". And the "active minutes" we want to measure is indeed "VeryActiveMinutes" + "FairlyActiveMinutes" = "Active Minutes" [[5]](#5). <br />
Therefore, to gain the average active minutes:
```sql
SELECT 
      DISTINCT Id,
      ROUND(AVG(TotalSteps),3) AS daily_avg_step,
      ROUND(AVG(VeryActiveMinutes+FairlyActiveMinutes),3) AS daily_avg_minute
FROM 
      `bellabeat-timilin.Fit_Data.daily_activity`
GROUP BY
      Id
HAVING 
      daily_avg_step >= 7000 OR
      daily_avg_minute >= 30
```
And this time we get back 21 users out of 33, which is *63.6%*. Moreover, we can also find users whose average step and average active minutes differ a lot, which might lead us to different conclusions. <br />
For example:
|     **Id**    | **Average Daily Steps** | **Average Daily Active Minutes** |
|:-------------:|:-----------------------:|:--------------------------------:|
|  6117666160   |         7046.714        |              3.607               |
#### During the week
After checking the daily averages, we can also investigate which days during the week users are more active or complete their daily step goal. <br />
| **Weekday** | **Average Total Step** | **Average Active Minutes** | **No_of_Users** |
|:-----------:|:----------------------:|:--------------------------:|:---------------:|
|    Sunday   |        6933.231        |           34.512           |       121       |
|    Monday   |        7780.867        |           37.108           |       120       |
|   Tuesday   |        8125.007        |           37.289           |       152       |
|  Wednesday  |        7559.373        |            33.88           |       150       |
|   Thursday  |        7405.837        |           31.367           |       147       |
|    Friday   |         7448.23        |           32.167           |       126       |
|   Saturday  |        8152.976        |           37.121           |       124       |

*(See code in the Appendix)* <br />
We can see that except for Sunday, users complete their daily step goals on average. But if we look at the average active minutes, the minimum 30 active minutes is achieved throughout the week. One possibility for this situation might be that people tend to stay at home on Sunday, but they still do some exercise to keep fit. <br />
Overall, users on average walk the most on Tuesday and Saturday, and their average active minutes are the highest on these two days as well.
#### During the day
We can go even deeper by checking on which hours during the day people are the most active.
```sql
SELECT
      DISTINCT Hours,
      ROUND(AVG(StepTotal),3) AS hourly_avg_step
FROM
(SELECT
       Hours,
       StepTotal
 FROM(
      SELECT
            EXTRACT(HOUR FROM Time) AS Hours,
            StepTotal
      FROM
          `bellabeat-timilin.Fit_Data.hourly_step`
     )
)
GROUP BY
        Hours
ORDER BY
        Hours
```
We found that the `hourly_avg_step` increases a lot around 8 AM and reaches its peak of 599.17 at 6 PM.

### Average Calorie burnt
The U.S. Department of Health and Human Services [[6]](#6) suggest that the average adult woman burns roughly 1,600 - 2,400 calories per day, and the average adult man uses 2,000 - 3,000 calories per day. However, to determine the exact numbers, we will need people's information regarding height, weight and gender. Unfortunately, we only have the weight and BMI information. Therefore, we will only examine the average calories burnt for each user during the 31 days and when users burn the most calories.
#### Per user
```sql
SELECT
      DISTINCT Id,
      ROUND(AVG(Calories),3) AS avg_cal,
      ROUND(AVG(VeryActiveMinutes + FairlyActiveMinutes),3) AS avg_active_minute
FROM
      `bellabeat-timilin.Fit_Data.daily_activity`
GROUP BY
      Id
```
We discovered that each user's average calorie expenses range from 1483.355 to 3436.581 Cal. And what we can definitely say is that there are four users who on average did not burn enough calories because their average calorie expenses are less than 1600 Cal. <br />
I also retrieved the average active minutes for each user, but at the moment we cannot see any obvious relationship between active minutes and calorie expense. We can keep these data and visualise them later so that we might see some correlation.
#### During the day
|        **AM**        |  **12** |  **1**  |  **2**  |  **3**  |  **4**  |  **5**  |   **6**   |  **7**  |  **8**  |  **9**  |  **10** |  **11** |
|:--------------------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:---------:|:-------:|:-------:|:-------:|:-------:|:-------:|
| **Average Calories** |  71.805 |  70.165 |  69.186 |  67.538 |  68.262 |  81.708 |   86.997  |  94.478 | 103.337 | 106.143 | 110.461 | 109.807 |
|        **PM**        |  **12** |  **1**  |  **2**  |  **3**  |  **4**  |  **5**  |   **6**   |  **7**  |  **8**  |  **9**  |  **10** |  **11** |
| **Average Calories** | 117.197 | 115.309 | 115.733 | 106.637 | 113.327 | 122.753 | *123.492* | 121.485 | 102.358 |  96.056 |  88.265 |  77.594 |

*(See code in the Appendix)* <br />
We found out that on average the highest calories of 123.492 Cal were burnt around 6 PM and the lowest of 67.538 Cal around 3 AM.

### Heart Rate
The normal resting heart rate for adults is between 60 - 100 beats per minute (bpm) [[7]](#7). But when people are doing physical activities their heart rate usually exceeds 100 bpm. The maximum heart rate a person can reach is *220 - his/her age* [[8]](#8), however, we do not have any age-related data, so let's see their average, minimum and maximum heart rates first.
```sql
SELECT
      Id,
      ROUND(AVG(Value),3) AS avg_heartrate,
      MIN(Value) AS Min_heartrate,
      MAX(Value) AS Max_heartrate
FROM
      `bellabeat-timilin.Fit_Data.heartrate`
GROUP BY
      Id
```
All 14 users have an average heart rate between 60 and 100 bpm. We can also discover that most of the users have a minimum heart rate of around or even under 40 bpm, this is not a big concern for healthy young adults and trained athletes, since they commonly have 40 - 60 bpm during sleep and rest. But for a general adult heart rate under 60 bpm is qualified as bradycardia. <br />
There are many records of a maximum heart rate exceeding 100 bpm, which can be qualified as tachycardia but can also be acceptable if the user was doing physical exercise. However, some records can be dangerous even if the user was doing exercise. For instance, *user 2022484408* reached 203 bpm, which can be the estimated maximum age-related heart rate for a 17-year-old (220 - 203 = 17) user, but is still dangerous even if he/she was doing vigorous-intensity physical activity. <br />
By assuming users sleep between 10 PM and 6 AM, we can look for their average heart rate during their sleep. Luckily, most of them have an average between 60 and 100 bpm, only two users had around 54 and 58 bpm. <br />
*(See code in the Appendix)*

### Sleep
According to the Centers for Disease Control and Prevention (CDC), the recommended hours of sleep per day changes as the user age. In general, an adult needs 7 - 9 hours of sleep a day. So let's find out the average sleeping hours for each user.
```sql
SELECT
      Id,
      ROUND((total_sleep/no_of_sleep)/60,3) AS avg_daily_sleephour
FROM
(SELECT
      DISTINCT Id,
      SUM(TotalMinutesAsleep) AS total_sleep,
      COUNT(TotalMinutesAsleep) AS no_of_sleep
FROM
      `bellabeat-timilin.Fit_Data.sleep_day_new`
GROUP BY
      Id
)
```
There are only 10 out of 24 users (*41.6%*) sleep 7 - 9 hours on average, 1 out of 24 sleeps over 9 hours (10.867h) and all of the rest of the users do not have enough sleep. We are not sure whether those users really do not have enough sleep or they only recorded their nap and not their evening sleep.

### Weight, BMI and Height
As we said before, we only have the weight and BMI information of only 8 users. We can use the below equation to calculate the height of the users: <br />
<p align="center"><img width="298" alt="bellabeat-bmi" src="https://user-images.githubusercontent.com/72343428/187093390-17f4d125-e093-4edd-9ba5-1bfe51ad7c62.png"></p>

So for each user, we get:

|   **Id**   | **Average Height(cm)** | **Average Weight(kg)** | **Average BMI** |
|:----------:|-----------------------:|-----------------------:|:---------------:|
| 4558609924 |          160.0         |                  69.64 |          27.214 |
| 4319703577 |          162.5         |                  72.35 |          27.415 |
| 1503960366 |          152.4         |                   52.6 |           22.65 |
| 8877689391 |          182.8         |                 85.146 |          25.487 |
| 2873212765 |          162.6         |                   57.0 |           21.57 |
| 5577150313 |          180.0         |                   90.7 |            28.0 |
| 1927972279 |          167.6         |                  133.5 |           47.54 |
| 6962181067 |          160.1         |                 61.553 |          24.028 |

*(See code in the Appendix)*

<a name="type"/>

### Smart Device Usage
Finally, we will examine user engagement in order to know the smart device usage. <br />
We will categorise their engagement as below:
 * Active User: Have 25 to 31 daily activity records.
 * Moderately Active User: Have 16 to 24 daily activity records.
 * Lightly Active User: Have 7 to 15 daily activity records.
 * Not Active User: Have 1 to 6 daily activity records.

|          **Id**         |   *4057192912*  |      *2347167796*      |      *8253242879*      |      *3372868164*      | *6775888955* | *7007744171* | *6117666160* | *8792009665* | *6290855005* | *1644430081* | *3977333714* | *5577150313* | *1624580081* | *2022484408* | *4319703577* | *4388161847* | *4702921684* | *6962181067* | *7086361926* | *8583815059* | *1844505072* | *1927972279* | *2026352035* | *2320127002* | *2873212765* | *4020332650* | *4445114986* | *4558609924* | *5553957443* | *8053475328* | *8378563200* | *8877689391* | *1503960366* |
|:-----------------------:|:---------------:|:----------------------:|:----------------------:|:----------------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|
| **# of Active Records** |               4 |                     18 |                     19 |                     20 |           26 |           26 |           28 |           29 |           29 |           30 |           30 |           30 |           31 |           31 |           31 |           31 |           31 |           31 |           31 |           31 |           31 |           31 |           31 |           31 |           31 |           31 |           31 |           31 |           31 |           31 |           31 |           31 |           31 |
|  **Total Usage Hours**  |          88.167 |                292.117 |                455.717 |                  472.9 |      591.667 |      599.467 |       507.85 |       559.35 |      689.733 |      685.633 |      481.233 |      509.767 |      736.617 |        736.6 |      506.583 |      573.267 |      534.783 |       490.55 |      548.817 |       742.65 |        683.8 |      701.683 |      488.983 |      734.817 |      736.467 |       684.45 |      541.133 |      724.717 |      470.667 |      720.083 |      486.267 |      735.517 |       581.75 |
|      **User Type**      | Not Active User | Moderately Active User | Moderately Active User | Moderately Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |  Active User |

*(See code in the Appendix)* <br />
Overall we have 1 Not Active User, 3 Moderately Active User and 29 Active User.

<a name="share"/>

## Sharing Our Findings
*(All the graphs can be found on my [Tableau](https://public.tableau.com/app/profile/jia.xin.lin) under the BellaBeat Case Study viz)*
### Steps
**1.** The graph below shows the average steps during the day. We can see that around 3 AM, there are almost 0 steps, which is reasonable since people are sleeping. From 5 AM, the step number increases dramatically, people wake up and go to work. The average step number floats between 400 and 600 from 8 AM to 7 PM.
<p align="center"><img src="https://user-images.githubusercontent.com/72343428/188267130-d25a1aa2-9fee-4bc4-b1de-78386a359c3b.png" data-canonical-src="https://user-images.githubusercontent.com/72343428/188267130-d25a1aa2-9fee-4bc4-b1de-78386a359c3b.png" width="900" /></p>


**2.** We now can see the average step walked by the users during the 31 days period. And we can notice that *60.6%* of the users complete the daily 7000-step goal.
![Steps_Goal](https://user-images.githubusercontent.com/72343428/188491397-b23e5fcf-33c5-4f29-b87c-56a72bfe722d.jpg)

<a name="step_week"/>

<img align="right" src="https://user-images.githubusercontent.com/72343428/188494830-707cecb9-f345-4a43-a687-eef0f5480682.jpg" data-canonical-src="https://user-images.githubusercontent.com/72343428/188494830-707cecb9-f345-4a43-a687-eef0f5480682.jpg" height="400" /> <br />
**3.** Taking the average by weekdays, we can discover that except for Sunday, the daily 7000-step goal is completed on the other days. <br clear="right"/>

### Active Minutes
**1.** Step number is not the only criterion to determine whether a user has done enough sport or not a day. We can also check their active minutes. An adult should have 30 active minutes daily, and we can see that only 51.5% of the users completed this goal.
![ActMin_Goal](https://user-images.githubusercontent.com/72343428/188504400-5fc30696-9bf5-44c8-9e25-a9c396cc8bf8.jpg)

**2.** Based on the above standard, the daily goal is completed on all the weekdays. And we notice that people are especially active on Monday, Tuesday and Saturday. This is also suggested from the [Average Steps During the Week](#step_week) graph. 
<p align="center"><img src="https://user-images.githubusercontent.com/72343428/188505574-6b1f102a-eca8-4859-8b12-56669ab386b1.jpg" data-canonical-src="https://user-images.githubusercontent.com/72343428/188505574-6b1f102a-eca8-4859-8b12-56669ab386b1.jpg" height="350" /></p>

**3.** If we combine our 7000-step and 30-minute goals, we can see *63.6%* of users completed at least one or both goals.
<p align="center"><img src="https://user-images.githubusercontent.com/72343428/188505806-1fe989c1-e4e1-4696-bfd8-9a2eef57e7c0.jpg" data-canonical-src="https://user-images.githubusercontent.com/72343428/188505806-1fe989c1-e4e1-4696-bfd8-9a2eef57e7c0.jpg" height="300" /></p>

### Calories Burnt
**1.** Most of the calories were burnt between 9 AM and 7 PM.
![Calories_day](https://user-images.githubusercontent.com/72343428/188677989-287efd44-ffbb-4274-a331-804b24daba93.png)

**2.** We want to find the relationship between calories burnt and step number, so we tried to find the best fit trendline. We know that a trendline is most reliable when its R-squared value is at or near 1. Therefore we found out that the polynomial trendline fits our data the most. The graphs below both have a cubic trendline. However, if we try to increase the degree of our model, then we will get a closer R-squared value to 1. Since we only have around 30 data points, I chose not to visualise a higher degree model to avoid bias. <br />
If we group our data by user Id, then the R-squared of the model is only around 0.213, but if we group by date, then R-squared is 0.967. So might use this cubic model to estimate the daily average calories burnt given the average step, or vice versa. 
![Calories_Step](https://user-images.githubusercontent.com/72343428/188678069-35efcfab-bf69-48b1-9913-22550aa89fe2.png)

**3.** Similar to above.
![Calories_ActMin](https://user-images.githubusercontent.com/72343428/188678103-16fda551-6312-41fa-b749-4d51c57afc76.png)

### Smart Device Usage
I have divided the users into 4 groups as defined [above](#type), and overall we have *87.88%* active users, they almost use a smart device everyday during the period.
<p align="center"><img src="https://user-images.githubusercontent.com/72343428/188747799-42b9f716-450c-47c9-a18f-25f47ffefe22.jpg" data-canonical-src="https://user-images.githubusercontent.com/72343428/188747799-42b9f716-450c-47c9-a18f-25f47ffefe22.jpg" height="300" /></p>

<a name="act"/>

## Act


<a name="ref"/>

## Refrence
<a id="1">[1]</a> Furberg, R., Brinton, J., Keating, M., & Ortiz, A. (2016). Crowd-sourced Fitbit datasets 03.12.2016-05.12.2016 [Data set]. Zenodo. https://doi.org/10.5281/zenodo.53894 <br />
<a id="2">[2]</a> Centers for Disease Control and Prevention (CDC). Stepping Up to Physcial Activity. Lifestyle Coach Facilitation Guide: Post-Core https://www.cdc.gov/diabetes/prevention/pdf/postcurriculum_session8.pdf <br />
<a id="3">[3]</a> Paluch, Amanda E., et al. "Steps per day and all-cause mortality in middle-aged adults in the coronary artery risk development in young adults study." JAMA network open 4.9 (2021): e2124516-e2124516. https://jamanetwork.com/journals/jamanetworkopen/fullarticle/2783711 <br />
<a id="4">[4]</a> Bumgardner, W. "Why Your Fitbit Active Minutes Mean More Than Your Steps [2020-06-23]." https://www.verywellfit.com/why-active-minutes-mean-more-than-steps-4155747 <br />
<a id="5">[5]</a> Semanik, P., Lee, J., Pellegrini, C. A., Song, J., Dunlop, D. D., & Chang, R. W. (2020). Comparison of physical activity measures derived from the Fitbit Flex and the ActiGraph GT3X+ in an employee population with chronic knee symptoms. ACR Open Rheumatology, 2(1), 48-52. https://onlinelibrary.wiley.com/doi/full/10.1002/acr2.11099 <br />
<a id="6">[6]</a> U.S. Department of Health and Human Services. 2015–2020 Dietary Guidelines for Americans. https://health.gov/our-work/nutrition-physical-activity/dietary-guidelines/previous-dietary-guidelines/2015 <br />
<a id="7">[7]</a> American Heart Association website. All About Heart Rate (Pulse). https://www.heart.org/en/health-topics/high-blood-pressure/the-facts-about-high-blood-pressure/all-about-heart-rate-pulse <br />
<a id="8">[8]</a> Centers for Disease Control and Prevention (CDC). Target Heart Rate and Estimated Maximum Heart Rate. https://www.cdc.gov/physicalactivity/basics/measuring/heartrate.htm <br />
<a id="9">[9]</a> Centers for Disease Control and Prevention (CDC). How Much Sleep Do I Need? https://www.cdc.gov/sleep/about_sleep/how_much_sleep.html

<a name="appendix"/>

## Appendix
For all the SQL codes, check the txt files.
