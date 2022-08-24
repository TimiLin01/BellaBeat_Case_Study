# BellaBeat Case Study
### Google Data Analytics Capstone Project
![bellabeat](https://user-images.githubusercontent.com/72343428/185792124-e7fe7db9-b2e6-47c8-bdcf-55da42c3b54f.png)

## Introduction
#### Company Background
BellaBeat was founded by Urška Sršen and Sandro Mur in 2013. It is a high-tech company that manufactures health-focused smart products, and collects data on activity, sleep, stress and reproductive health of its customers in order to provide information regarding their health and habits.

## Ask
#### Business Task
Identifying the trends in smart device usage and provide recommendations that will help skateholders to make data-driven decisions regarding the marketing strategy and discover new potentials for the company.
#### Key Stakeholders
 * Urška Sršen: BellaBeat Co-founder and Chief Creative Officer (CCO)
 * Sandro Mur: Mathematician, BellaBeat Co-founder and Chief Executive Officer (CEO)
 * BellaBeat Marketing Team

## Prepare
#### About the Data
The dataset[[1]](#1) I used is *FitBit Fitness Tracker Data*, which is avaiable on Kaggle provided by MÖBIUS. <br />
It is collected from eligible Fitbit users, who consented to the submission of personal tracker data via a survey distributed by Amazon Mechanical Turk between 03.12.2016 - 05.12.2016.  <br />
This dataset has overall 18 CSV files including information about minute-level output for physical activity, heart rate, weight and sleep monitoring between 04.12.2016 - 15.12.2016 (31 days period).
#### Data Storage and Organization
After downloading the dataset, I opened all the CSV file in spreadsheet (EXCEL) to see how are the data structured. <br />
The dataset includes wide and narrow data. Each user has a unique ID, so that we can distinguish between them.
I noticed that *'dailyCalories_merged.csv'*, *'dailyIntensities_merged.csv'* and *'dailySteps_merged.csv'* are all included in *'dailyActivity_merged.csv'*, so I decided not using these three files. <br />
I also discovered that some tables have too many rows, hence using hte BigQuery to analyse the data will be a better choice than spreadsheet. <br />
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
#### Credibility and Integrity
To determine the credibility and integrity of the data I will use the **'ROCCC'** system:
  * **Reliability**: The data is **NOT** reliable, since we are not sure what is the marginal error, and a small sample size (~30 participants) has been used. So our analysis might not be true for the whole population.
  * **Originality**: The data is **NOT** original, since it was collected by Amazon Mechanical Turk and has been made avaiable through MÖBIUS, but has been check against [[1]](#1).
  * **Comprehensiveness**: The data is **NOT** comprehensive, because we have no information regarding whether our sample are randomly collected our not. This might lead to biasness.
  * **Current**: The data is **NOT** current, since it was collected in 2016. Therefore our analysis cannot represent the current trend in smart device usage.
  * **Cited**: It is cited. <br />
As a result, our data do not satisfies the 'ROCCC' system. This means that using this dataset, we are **unable** to provide realiable and comprehensive recommendations for BellaBeats. Hence, our analysis can only act as directions which should be verified through a more reliable dataset.

## Processing Data
#### Uploading and Transforming Data
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
SELECT *
FROM `bellabeat-timilin.Fit_Data.daily_activity`
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
FROM `bellabeat-timilin.Fit_Data.sleep_day`
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

Hence, we need to remove three rows in total from this table. To do this, we can go back temporarily to EXCEL, and by filtering for these rows we can easily delete the duplicates. I chose to do this in EXCEL, because we only have 3 rows to delete, but if we have more, it is better to use R.


## Refrence
<a id="1">[1]</a> Furberg, R., Brinton, J., Keating, M., & Ortiz, A. (2016). Crowd-sourced Fitbit datasets 03.12.2016-05.12.2016 [Data set]. Zenodo. https://doi.org/10.5281/zenodo.53894
