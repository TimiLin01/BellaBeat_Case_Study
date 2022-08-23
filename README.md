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
  * **Originality**: The data is **NOT** original, since it was collected by Amazon Mechanical Turk and has been made avaiable through MÖBIUS, but has been check against [1].
  * **Comprehensiveness**: The data is **NOT** comprehensive, because we have no information regarding whether our sample are randomly collected our not. This might lead to biasness.
  * **Current**: The data is **NOT** current, since it was collected in 2016. Therefore our analysis cannot represent the current trend in smart device usage.
  * **Cited**: It is cited. <br />
As a result, our data do not satisfies the 'ROCCC' system. This means that using this dataset, we are **unable** to provide realiable and comprehensive recommendations for BellaBeats. Hence, our analysis can only act as directions which should be verified through a more reliable dataset.

## Processing Data
#### Uploading Data

## Refrence
<a id="1">[1]</a> Furberg, R., Brinton, J., Keating, M., & Ortiz, A. (2016). Crowd-sourced Fitbit datasets 03.12.2016-05.12.2016 [Data set]. Zenodo. https://doi.org/10.5281/zenodo.53894
