# Customer Behaviors
 [![](https://img.shields.io/badge/-Survey-blue)](#) [![](https://img.shields.io/badge/-Python-blue)](#)
## Introduction
A customer survay "Google form" is export to an excel file (xlsx), then analyzed and shown in this topic.

**Raw Data File :** [Excel Raw Data](./%5BRaw%20Data%5D%20Customer%20Behaviors%20(Responses).xlsx)  
**Jupyter Notebook File :** [Analysis of Customer Behaviors](./HW%201.ipynb)  
 
## Contents
### Raw Data
![rawdata](./raw_data.png)

### Demographic
The respondents mainly age 28 years old, with approximately half male and female. 
![demographic](./demographic.png)

### Scoring
How much people interest and consumption things in each categories.
#### Interests
**No surprising** : 
Men are intereting on Games, Beers than female.
Female are interesing on Make-Up than men.
![interest frequency](./interest_freq.png)
#### Consumptions
![consumption frequency](./consumption_freq.png)

### Paired Distribution
Comparing distributions of the responses on both interest and consumption in the same topic.  
![boxplot](./boxplot.png)

### Correlation of Interests
![interest correlation](./interest_corr_overall.png)

### Correlation of Consumptions
![consumption correlation](./consumption_corr_overall.png)

### Correlation between Interests and Consumptions
**Surprising result** : People who consumptions thai food is low correlate with interest thai food.
![Interests_Consumption_Correlation](./InxCo_overall.png)


### K-Means [Unsupervised Learning]
No appropiate cluster could be made from this data. the reasons are including less number of responded(63) comparing to the number of features(60).

![K_Mean](./K-Mean.png)
