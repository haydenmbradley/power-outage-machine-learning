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
| CAUSE.CATEGORY   |  Categories of all the events causing the major power outages   |
| CUSTOMERS.AFFECTED  |   Number of customers affected by the power outage event   |

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

Before we are able to analyze our data effectively, we must first look at what to do with missing values. We see that 1493 of the 1534 columns contain atleast one null value, meaning if we were to remove all of these columns we would have very little data left to work with. In order to tackle this, let's first focus on just the variables that will be useful for our analysis.
| Variable | Number of Null Values |
|----------|----------|
|OUTAGE.START    |        9|
|CLIMATE.REGION   |       6|
|CLIMATE.CATEGORY   |     9|
|CAUSE.CATEGORY     |     0|
|CUSTOMERS.AFFECTED  |  443|
|ANOMALY.LEVEL      |     9|
|NERC.REGION        |     0|
|U.S._STATE         |     0|
|OUTAGE.DURATION    |    58|

Since variables like CLIMATE.REGION, CLIMATE.CATEGORY, and ANOMALY.LEVEL are all specific data points that are not easy to predict, and there are not that many null values for these categories, we will simply remove all rows where these columns are null. Additionally, since we will eventually try to predict OUTAGE.DURATION, it does not make sense to impute values here which would introduce bias into our model. Therefore, we will also remove all rows where OUTAGE.DURATION is null. This leaves us with 1471 rows of data which should still be plenty to perform a thorough analysis.

### Imputation

Since the 'CUSTOMERS.AFFECTED' column is an important column with a lot of missing values, we cannot simply remove all its null values. Instead, we will perform an imputation which attempts to accurately replace these values. Since these values appear to be missing at random, we can use the strategy of mean imputation. To be even more specific, we will group the mean values based on state, because outages within the same state tended to be closer in the 'CUSTOMERS.AFFECTED' than between different states. Here is the distribution of 'CUSTOMERS.AFFECTED' before the imputations:

<iframe src="assets/figure1.html"
        width="800"
        height="600"
        frameborder="0">
 </iframe>

Here is the distribution after:

 <iframe src="assets/figure2.html"
        width="800"
        height="600"
        frameborder="0">
 </iframe>

Since there were 443 missing values, the imputation did slightly alter the distribution. Nevertheless, the general shape remains the same and now we have no more remaining null values. In our remaining 1466 rows:

| Variable | Number of Null Values |
|----------|----------|
|OUTAGE.START    |        0|
|CLIMATE.REGION   |       0|
|CLIMATE.CATEGORY   |     0|
|CAUSE.CATEGORY     |     0|
|CUSTOMERS.AFFECTED  |    0|
|ANOMALY.LEVEL      |     0|
|NERC.REGION        |     0|
|U.S._STATE         |     0|
|OUTAGE.DURATION    |     0|

### Univariate Analysis

 
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


### Bivariate Analysis

 
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

### Interesting Aggregates



## Framing a Prediction Problem

For the final prediction, I will be trying to answer the question of 'Can we predict the duration of power outages?' This prediction will use various of the collected values in order to predict the output OUTAGE.DURATION. Since OUTAGE.DURATION is in the units of hours, which is a continuous variable, this is a regression problem. In order to make this realistic, I will only use variables that contain information that would be readily available shortly after a power outage has occured. Variables related to the location (U.S._STATE, NERC.REGION), the climate (CLIMATE.REGION, CLIMATE.CATEGORY, ANOMALY.LEVEL), the cause (CAUSE.CATEGORY), and the number of outages (CUSTOMERS.AFFECTED), should all either be identifiable immediately or after a short inspection following the initial outage. For these reasons, I think all these variables are fair game. Finally, since this is a regression task and a situation where outliers (long outages) are important to be predicted properly, I will be using Mean Squared Error to evaluate my model. The MSE will penalize these large errors ensuring proper prediction of longer outages. I will also use R^2 as a comparison baseline to see how each iteration of the model compares with each other.

## Baseline Model

For our baseline model, we will use the variables from earlier that seemed to have the most predictive impact and that could be immediately determined once the location of the power outage is given. This will give an initial model which we can evaluate then iterate upon. For this initial model we will one hot encode all of the categorical variables and standard scale the quantitative (which will not improve the effectiveness of the model but will help us analyze the coefficients). In addition, to keep our model quick and simple, we will use a Linear Regression.

 <iframe src="assets/figure7.html"
        width="800"
        height="600"
        frameborder="0">
 </iframe>

| Performance Metric | Performance |
|----------|----------|
| Mean Squared Error | 10193.898269331174 |
| R² Score | 0.08269072178571046 |

Unfortunately our initial model only achieved a R^2 of 0.08, which means it barely beats out a model that just predicts the mean. Additionally, we see from the graph above that our predictions (orange) do not seem to estimate the large spikes in the actual durations (blue). Nevertheless, the prediction do accurately predict some small aspect of the variance, giving us hope to improve our final model.

## Final Model

Our final model will take in all the variables from our previous model and incorporate additional ones that might edge our final predictions closer to the real values. These new variables will consist of CLIMATE.CATEGORY, NERC.REDION, and CUSTOMERS.AFFECTED. We will continue to use mean squared error to evaluate our final model for the same reasons as before (and similarly incorporating R^2 as a benchmark).

In addition, I will engineer a few new variables from our given list. First, I added a polynomial degree three transformation to both of the quantitative variables. These new features help to determine if the impact to outage duration changes in a non-linear way as CUSTOMERS.AFFECTED and ANOMALY.LEVEL increase. This was prompted by an earlier graph which shows instances where OUTAGE.DURATION would increase then decrease then increases again as CUSTOMERS.AFFECTED increased. Secondly, I created a flag based on CAUSE.CATEGORY which shows a 1 if the outage was weather related and a 0 otherwise. This allowed the model to distinguish weather related events which may have a more specific affect on power outages.

Finally, I decided to switch to a random forest regressor to better capture the non-linear relationship between these variables. Since a lot of the relationships are clearly non-linear (based on the graphs above), this model should do much better than the linear regression. In addition, I will use grid search CV on the random forest max depth and number of estimators. The max depth with help find a good balance between bias and variance, and the n_estimators will make sure the model takes into account enough trees so that it generalizes well to new data.

 <iframe src="assets/figure8.html"
        width="800"
        height="600"
        frameborder="0">
 </iframe>

 | Performance Metric | Performance |
|----------|----------|
| Mean Squared Error | 9066.514600721574 |
| R² Score | 0.18413959561194515 |

Our R^2 has bumped up to 0.18 which is a vast improvement over our initial model. In addition, we can clearly see based on the graph above that the predictions (orange) do a better job of capturing the spikes in the actual outage durations (blue) which was one of the most important goals for our model. By capturing more non-linear relationships and incorporating more helpful inputs, our model has clearly improved over the baseline.
 
