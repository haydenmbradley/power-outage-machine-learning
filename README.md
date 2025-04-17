# Can we predict the duration of major power outages?

## Introduction

In this project, I will be looking at data on major power outage event in the United States. This dataset provides all the important details about when and where these power outages occurered, what circumstances caused the outages, how large an impact these outages had, and details about the environment where these outages took place. Using the necessary aspects of this data, important questions about when, where, why, and how long these outages occur can be explored. In this specific project, I would like to explore the question: Can we predict the duration of power outages given all the information leading up to and including the initial outage? Answering this question can help better set the expectations of those affected by these outages, as well as better channel the efforts of emergency response teams when needed.

### Exploring The Data

This dataset contains 1534 rows, each representing an individual outage event, and 57 columns detailing various characteristics of each outage. Below are some of the relevant columns for our analysis:

| Variable | Description |
|----------|----------|
| YEAR    | Indicates the year when the outage event occurred |
| MONTH    | Indicates the month when the outage event occurred  |
| U.S._STATE    |  Represents all the states in the continental U.S.  |
| NERC.REGION    |  The North American Electric Reliability Corporation (NERC) regions involved in the outage event   |
| CLIMATE.REGION   | U.S. Climate regions as specified by National Centers for Environmental Information   |
| CLIMATE.CATEGORY   |  This represents the climate episodes corresponding to the years   |
| ANOMALY.LEVEL   |  This represents the oceanic El Niño/La Niña (ONI) index referring to the cold and warm episodes by season   |
| OUTAGE.START.DATE   |  	This variable indicates the day of the year when the outage event started  |
| OUTAGE.START.TIME   |  This variable indicates the time of the day when the outage event started  |
| OUTAGE.RESTORATION.DATE  |  This variable indicates the day of the year when power was restored to all the customers   |
| OUTAGE.RESTORATION.TIME  |   This variable indicates the time of the day when power was restored to all the customers |
| OUTAGE.DURATION  |   Duration of outage events (in minutes)  |
| CAUSE.CATEGORY   |          |
| CUSTOMERS.AFFECTED  |   Number of customers affected by the power outage event   |



<iframe src="assets/figure1.html"
        width="800"
        height="600"
        frameborder="0">
 </iframe>

 <iframe src="assets/figure2.html"
        width="800"
        height="600"
        frameborder="0">
 </iframe>
 
 <iframe src="assets/figure3.html"
        width="800"
        height="600"
        frameborder="0">
 </iframe>

 <iframe src="assets/figure4.html"
        width="800"
        height="600"
        frameborder="0">
 </iframe>
 
 <iframe src="assets/figure5.html"
        width="800"
        height="600"
        frameborder="0">
 </iframe>

 <iframe src="assets/figure6.html"
        width="800"
        height="600"
        frameborder="0">
 </iframe>

 <iframe src="assets/figure7.html"
        width="800"
        height="600"
        frameborder="0">
 </iframe>

 <iframe src="assets/figure8.html"
        width="800"
        height="600"
        frameborder="0">
 </iframe>
 
