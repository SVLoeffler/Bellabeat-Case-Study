# Bellabeat-Case-Study
Capstone Project for my Google Data Analytics Certificate

## Objective
Bellabeat is a tech manufacturer of women’s wellness products. They have developed  wearable smart devices which record and measure various biometrics of the user. In order to gain further insight into the usage of these types of devices, publicly available data collected from Fitbit users will be analyzed to find key insights that could drive marketing strategy for Bellabeat. The objective is to identify opportunity for growth among Bellabeats users by utilizing trends observed from Fitbit usage data. 

## Key Stakeholders
- Urška Sršen: Bellabeat cofounder and chief creative officer
- Sando Mur: Bellabeat cofounder and member of the executive team
- Bellabeat Marketing Analytics team

## Questions
What are some observable trends in smart device usage? 
How can these trends be applied to Bellabeat customers?
How can Bellabeat marketing strategy be oriented to these trends? 

## Data
FitBit Fitness Tracker Data (CC0: Public Domain, dataset made available through Mobius)

The data contains various personal data of 30 Fitbit users who consented to the submission of fitness tracker data. Data includes physical activity, heart rate, and sleep monitoring. All data collected was anonymous. 

Limitations to the data include the following:
- Small sample size. Data from only 30 Fitbit users was collected.
- No demographic information accompanies the data. We do not know the users age, gender, geographic location, economic status, country of citizenship, race, or ethnicity. The possibility exists, therefore, that this data is biased towards specific demographics. 
- Data was only collected for a few months. It would not be unreasonable to think that the time of year may factor into some of the data collected. Similarly to the small sample size, a longer window of collection may have provided more reliable data on usage. 
- Lastly, the data is also somewhat outdated. It was collected in the Spring of 2016. Smart device usage and public opinion may have changed in the intervening years. 

## Data Preparation
I focused mainly on data collected on the users’ daily sleep and activity level. 

Data collected for Daily Activity included: ID, ActivityDate, TotalSteps, TotalDistance, TrackerDistance, VeryActiveDistance, ModeratelyActiveDistance, LightlyActiveDistance, VeryActiveMinutes, FairlyActiveMinutes, LightlyActiveMinutes, SedentaryMinutes, and Calories. Data was organized by user ID and the Date recorded. 

Data regarding sleep included ID, SleepDay, TotalSleepRecorded,TotalMinutesAsleep,TotalTimeInBed.

Data was first uploaded to Google Sheets for preliminary cleaning. Any duplicate data and white space was trimmed. Additionally, as we were looking at daily data, time stamps were removed as irrelevant. I also inserted a new column to each data set identifying the day of the week the data was collected. Lastly, using the concatenate function, I created a SpecialID column using the data recorded and ID of the user. 

Next, the data was uploaded to Microsoft Azure Data Studio for further cleaning and exploration via SQL. 

The Sleep Data revealed only 24 users recorded data out of the 30 who participated in the survey:

` Select Count (Distinct Id)
From Bellabeat_CaseStudy.dbo.Sleep
Group By (Id) `

Additionally, 15 rows showed less than 2 hours of sleep recorded for that day. This to me indicates the device was recording naps but not night time sleep. This data was not removed from the data set but does suggest that at least a portion of users do not record their sleep data with a smart device. 

`Select TotalMinutesAsleep
From Bellabeat_CaseStudy.dbo.Sleep
Where TotalMinutesAsleep < 120`


Average amount of time asleep were determined for each user:

`Select Id, Avg (TotalMinutesAsleep) As AvgUserAsleep
From Bellabeat_CaseStudy.dbo.Sleep
Group By Id`

No data was found containing 0 hours of sleep recorded: 

`Delete From Bellabeat_CaseStudy.dbo.Sleep
Where TotalMinutesAsleep = 0`

From the Activity data set; rows indicating 0 total distance recorded for the day were dropped. This to me indicated days where the device was not actually worn: 
 
`Delete From Bellabeat_CaseStudy.dbo.Activity
Where TotalDistance = 0`

Another column was added to the dataset for the total number of minutes the device was worn for a particular day: 
`Alter TABLE Bellabeat_CaseStudy.dbo.Activity
Add TotalMinutes Int`
 
`Update Bellabeat_CaseStudy.dbo.Activity
Set TotalMinutes = ( VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes + SedentaryMinutes)`

 Unused columns were dropped from the dataset:

`Alter Table Bellabeat_CaseStudy.dbo.Activity
Drop Column TrackerDistance, VeryActiveDistance, ModeratelyActiveDistance, LightActiveDistance`
 
Similarly to the sleep data, average number of minutes worn per user and per day of the week were found: 
 
`Select Avg(TotalSteps) as AvgSteps, DayofWeek
From Bellabeat_CaseStudy.dbo.Activity
Group by DayofWeek`

&

`Select Id, Avg(TotalMinutes) as AvgTimeWorn
From Bellabeat_CaseStudy.dbo.Activity
Group By Id`

&

`Select DayofWeek, Avg(TotalMinutes) as AvgTimeWorn
From Bellabeat_CaseStudy.dbo.Activity
Group By DayofWeek`

Finally, data was joined between the two data sets utilizing the SpecialID column as the relationship: 

`Select *
From Bellabeat_CaseStudy.dbo.Activity
Join Bellabeat_CaseStudy.dbo.Sleep ON Bellabeat_CaseStudy.dbo.Sleep.Special_ID = Bellabeat_CaseStudy.dbo.Activity.SpecialID`

## Key Takeaways

Data visualizations were created utilizing Tableau Public software. 

The first trend identified was the amount of time users wore their devices. They fell into roughly two camps; those that wore their device all day and night, and those that only wore their device for part of the day. The green users below wore their devices for between 950-1100 minutes or about 15 ½ to 18 hours. It appears most users at least wear their device for at least most of the day, rather than for specific, short activities. This leaves the question, why is there a subset of users who do not wear their device all the time? Why are they taking it off? 

![WornUser](https://user-images.githubusercontent.com/99687584/153956924-49260abf-4efb-41c0-978f-680e9020b951.png)

Additionally, a significant number of users do not wear their device to sleep and do not record their sleep data. The CDC recommends at least 7 hours (420 minutes) of sleep per night for adults. While most users recorded an average close to this amount, there were several with noticably less sleep recorded and 3 users whose average fell well below the suggested amount of sleep. While initially I thought these users were simply not wearing their devices for a siginificant amount of time and perhaps only recording naps, when cross referencing these user IDs with the previous wearing history chart, I found that these 3 users actually wore their devices for the majority of the day. 

![SleepUser](https://user-images.githubusercontent.com/99687584/153958287-a8c7d550-806e-4eed-804b-14cd7f848277.png)

While reviewing sleep trends over the course of the week I found users generally slept less during the work week and had a narrower range of sleep time recorded. While there were certainly some instances of little sleep being recorded on the weekends I found a wide range in the distribution of sleep on the weekend. Users appeared to also be getting more sleep on the weekends possibly catching up from the work week. 
![SleepRange](https://user-images.githubusercontent.com/99687584/153961582-edf9ce61-85ae-410e-807c-e6395ea9747e.png)

The CDC recommends at least 8,000 steps a day to decrease the risk of all-cause mortality. Based on the average number of steps taken per day, users appear to either meet that recommendation or at least come close. Sundays and Fridays appear to show slighly few steps recorded than other days of the week. 

![StepDayofWeek](https://user-images.githubusercontent.com/99687584/153963127-6d22244a-b0e6-40ef-a653-d1d008d408e8.png)

Finally, a negative correlation was found between the number of sedentary minutes and amount of time asleep recorded (r-squared= 0.511)

![SleepVsSed](https://user-images.githubusercontent.com/99687584/153963834-68d67674-674f-45b3-b68a-1d5c2de18ee8.png)

## Recommendations

Keeping in mind the original objective to determine opportunities for growth my recommendations are as follows:

-Based on observed trends regarding steps and activity on Sundays and Friday, I recommend enacting "community challenge days" on these days to drive engagement across the bellabeat user base. While personalizing push notifications to inidivual users during sedentary times may also be helpful, I believe implimenting community wide step challenge days can bolster a sense of engagement with the broader Bellabeat brand. 
-I suggest starting a root cause analysis regarding why a subset of users do no record their sleep date or only record data for part of the day. This could invovle further data collection or opinion surveys. If the cause is due to discomfort with the device while sleeping, then this is an issue that should be addressed with a redesign. If the cause is due to perceived low value of gathering sleep data then an opportunity exists to build user opinions of the value of their device. This can be done by highlighting benefits of recording and monitoring this data a long with recommendations from the CDC or other health organizations. 
-In addition to my recommendation regarding futher analysis into why some users do not record their sleep, I would suggest promoting Bellabeat as an overall wellness device. The correlation between inactivity and sleep could be used as a way to promote Bellabeat as a comprehensive wellness brand. 
