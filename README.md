# BellaBeat Case Study
### Google Data Analytics Capstone Project
![bellabeat](https://user-images.githubusercontent.com/72343428/185792124-e7fe7db9-b2e6-47c8-bdcf-55da42c3b54f.png)

## Introduction

#### Company Background
BellaBeat was founded by Urška Sršen and Sando Mur in 2013. It is a high-tech company that manufactures health-focused smart products, and collects data on activity, sleep, stress and reproductive health of its customers in order to provide information regarding their health and habits.
#### Business Task
Identifying the trends in smart device usage and provide recommendations that will help skateholders to make data-driven decisions regarding the marketing strategy and discover new potentials for the company.
#### About the Data
The dataset[1] I used is *FitBit Fitness Tracker Data*, which is avaiable on Kaggle provided by MÖBIUS. <br />
It is collected from eligible Fitbit users, who consented to the submission of personal tracker data via a survey distributed by Amazon Mechanical Turk between 03.12.2016 - 05.12.2016.  <br />
This dataset has overall 18 CSV files including information about minute-level output for physical activity, heart rate, weight and sleep monitoring between 04.12.2016 - 15.12.2016 (31 days period).
#### Credibility and Integrity
To determine the credibility and integrity of the data I will use the **'ROCCC'** system:
  * **Reliability**: The data is **NOT** reliable, since we are not sure what is the marginal error, and a small sample size (~30 participants) has been used. So our analysis might not be true for the whole population.
  * **Originality**: The data is **NOT** original, since it was collected by Amazon Mechanical Turk and has been made avaiable through MÖBIUS, but has been check against [1].
  * **Comprehensiveness**: The data is **NOT** comprehensive, because we have no information regarding whether our sample are randomly collected our not. This might lead to biasness.
  * **Current**: The data is **NOT** current, since it was collected in 2016. Therefore our analysis cannot represent the current trend in smart device usage.
  * **Cited**: It is cited.
As a result, our data do not satisfies the 'ROCCC' system. This means that using this dataset, we are **unable** to provide realiable and comprehensive recommendations for BellaBeats. Hence, our analysis can only act as directions which should be verified。

## Processing Data


### Refrence
[1] Furberg, R., Brinton, J., Keating, M., & Ortiz, A. (2016). Crowd-sourced Fitbit datasets 03.12.2016-05.12.2016 [Data set]. Zenodo. https://doi.org/10.5281/zenodo.53894
