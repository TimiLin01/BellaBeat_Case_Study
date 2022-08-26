# BellaBeat Case Study
### Google Data Analytics Capstone Project
![bellabeat](https://user-images.githubusercontent.com/72343428/185792124-e7fe7db9-b2e6-47c8-bdcf-55da42c3b54f.png)

## Introduction
### Company Background
BellaBeat was founded by Urška Sršen and Sandro Mur in 2013. It is a high-tech company that manufactures health-focused smart products and collects data on the activity, sleeps, stress and reproductive health of its customers in order to provide information regarding their health and habits.

## Ask
### Business Task
Identifying the trends in smart device usage and providing recommendations that can help stakeholders to make data-driven decisions regarding the marketing strategy and discover new potentials for the company.
### Key Stakeholders
 * Urška Sršen: BellaBeat Co-founder and Chief Creative Officer (CCO)
 * Sandro Mur: Mathematician, BellaBeat Co-founder and Chief Executive Officer (CEO)
 * BellaBeat Marketing Team

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
  * **Originality**: The data is **not** original since it was collected by Amazon Mechanical Turk and made available through MÖBIUS, but has been checked against [[1]](#1).
  * **Comprehensiveness**: The data is **not** comprehensive because we have no information regarding whether our samples are randomly collected or not. This might lead to bias.
  * **Current**: The data is **not** current since it was collected in 2016. Therefore our analysis cannot represent the current trend in smart device usage.
  * **Cited**: It is cited. <br />
As a result, our data do not satisfy the 'ROCCC' system. This means that using this dataset, we are **unable** to provide reliable and comprehensive recommendations for BellaBeats. Hence, our analysis can only act as directions which should be verified through a more reliable dataset.

## Processing Data
### Uploading and Transforming Data
When I tried to upload the abouve CSV files into BigQuery, only the schema for *'dailyIntensities_merged.csv'* can be dedected automatically. For other files, the below error occur:
```
Failed to create table: Error while reading data, error message: Could not parse '4/12/2016 12:00:00 AM' as TIMESTAMP for field ActivityHour (position 1) starting at location 26 with message 'Invalid time zone: AM'
```
Therefore, I had to define the schema for them manually, and let `ActivityHour` temporary be `STRING` instead of `TIMESTAMP`. <br />
Now, having the files uploaded. I checked the datatypes for each table. Since, we temporarily set the `ActivityHour/Time/SleepDay` as `STING` for some tables, we can now set them as `DATETIME` using the following code:
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
And similarly we can do this for `hourly_cal`, `hourly_intensities`, `hourly_step`, `sleep_day` and `weight_log` tables.
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
We only found NULL Values in `weight_log` table, under the `Fat` column. These NULL Values can be ignored if we do not want to use information regarding users' fat. <br />
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
And we can actually find duplicates in `sleep_day` table:
|   **Id**   |       **Time**      | **TotalSleepRecords** | **TotalMinutesAsleep** | **TotalTimeInBed** | **No_Of_Dup** |
|:----------:|:-------------------:| ---------------------:| ----------------------:| ------------------:| -------------:|
| 4388161847 | 2016-05-05T00:00:00 |                     1 |                    471 |                495 |             2 |
| 8378563200 | 2016-04-25T00:00:00 |                     1 |                    388 |                402 |             2 |
| 4702921684 | 2016-05-07T00:00:00 |                     1 |                    520 |                543 |             2 |

Hence, we need to remove three rows in total from this table. To do this, we can go back temporarily to EXCEL, and by filtering for these rows we can easily delete the duplicates. I chose to do this in EXCEL, because we only have 3 rows to delete, but if we have more, it is better to use R. <br />
This gives a new table:
|   **Table Name**  |       **CSV File Name**       |       **Discription**        |
|:-----------------:|:-----------------------------:|:-----------------------------|
|  sleep_day_new    |  sleepDay_merged_cleaned.csv  | Cleaned version of sleep_day |

## Analyzing Data
### Checking the # of users
Before we start analyzing our data, it is always good to make sure how many unique IDs are in different tables. Because not all the participants answers all the questions.
```sql
-- eg. For daily_activity table
SELECT
    COUNT(DISTINCT Id) AS No_of_users
FROM `bellabeat-timilin.Fit_Data.daily_activity`
```
|  **Table Name** | daily_activity | heartrate | hourly_cal | hourly_intensities | hourly_step | sleep_day_new | weight_log |
|:---------------:|:--------------:|:---------:|:----------:|:------------------:|:-----------:|:-------------:|:----------:|
| **No_of_users** |       33       |     14    |     33     |         33         |      33     |       24      |      8     |

### Average Steps VS Average Active Minutes
The Centers for Disease Control and Prevention (CDC) suggest that an adult should aim for 10000 steps per day [[2]](#2). But recently, there are also many articles says that 7000 steps is already enough. Therefore, we can first investigate on average how many user walks more than 7000 steps a day on average.
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
This give us back 20 users out of 33, that is *60.6%* of our sample population. <br />
However, does having more steps means that you were more active and did more sport than others who completed fewer steps? Not necessarily, since step numbers can be faked. Therefore, we should look at the active minutes. <br />
An adult should aim for at least 30 "active minutes" per day [[3]](#3). But how can we link this with our dataset? Actually, Fitbit measures active minutes, but in our data, it is splitted into "VeryActiveMinutes", "FairlyActiveMinutes", "LightlyActiveMinutes" and "SedentaryMinutes". And the "active minutes" we want to measure is indeed "VeryActiveMinutes" + "FairlyActiveMinutes" = "Active Minutes" [[4]](#4). <br />
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
And this time we get back 21 users out of 33, which is *63.6%*. Moreover, we can also find such user, whose average step and average active minutes differ a lot, and might lead us to different conclusion. <br />
For example:
|     **Id**    | **Average Daily Steps** | **Average Daily Active Minutes** |
|:-------------:|:-----------------------:|:--------------------------------:|
|  6117666160   |         7046.714        |              3.607               |

### During the week
After checking the daily averages, we can also investigate about on which days during the week, users are more active or complete their daily step goal. <br />
*See code in the Appendix*
| **Weekday** | **Average Total Step** | **Average Active Minutes** | **No_of_Users** |
|:-----------:|:----------------------:|:--------------------------:|:---------------:|
|    Sunday   |        6933.231        |           34.512           |       121       |
|    Monday   |        7780.867        |           37.108           |       120       |
|   Tuesday   |        8125.007        |           37.289           |       152       |
|  Wednesday  |        7559.373        |            33.88           |       150       |
|   Thursday  |        7405.837        |           31.367           |       147       |
|    Friday   |         7448.23        |           32.167           |       126       |
|   Saturday  |        8152.976        |           37.121           |       124       |

We can see that except of Sunday, on other days, users on average completes their daily step goal. But if we lookat the average active minutes, then the minumum 30 active minutes is achieved on the whole week. One possibility for this situation might be, people tend to stay at home on Sunday, but they still do some exercise to kepp fit. <br />
Overall, users on average walk the most on Tuesday and Saturday, and their average active minutes are the highest on these two days as well.

### Active Hours

## Refrence
<a id="1">[1]</a> Furberg, R., Brinton, J., Keating, M., & Ortiz, A. (2016). Crowd-sourced Fitbit datasets 03.12.2016-05.12.2016 [Data set]. Zenodo. https://doi.org/10.5281/zenodo.53894 <br />
<a id="2">[2]</a> Centers for Disease Control and Prevention (CDC). Stepping Up to Physcial Activity. Lifestyle Coach Facilitation Guide: Post-Core https://www.cdc.gov/diabetes/prevention/pdf/postcurriculum_session8.pdf <br />
<a id="3">[3]</a> Bumgardner, W. "Why Your Fitbit Active Minutes Mean More Than Your Steps [2020-06-23]." https://www.verywellfit.com/why-active-minutes-mean-more-than-steps-4155747 <br />
<a id="4">[4]</a> Semanik, P., Lee, J., Pellegrini, C. A., Song, J., Dunlop, D. D., & Chang, R. W. (2020). Comparison of physical activity measures derived from the Fitbit Flex and the ActiGraph GT3X+ in an employee population with chronic knee symptoms. ACR Open Rheumatology, 2(1), 48-52. https://onlinelibrary.wiley.com/doi/full/10.1002/acr2.11099

## Appendix
For whole sql code, check the txt files.
