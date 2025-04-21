# Can we predict the duration of major power outages?
by Hayden Bradley (haydenmb@umich.edu)

## Introduction

In this project, I will look at data on major power outage events in the United States. This dataset provides all the important details about when and where these power outages occurred, what circumstances caused the outages, how large an impact these outages had, and details about the environment in which these outages took place. Using the necessary aspects of this data, important questions about when, where, why, and how long these outages occur can be explored. In this specific project, I would like to explore the question: "Can we predict the duration of power outages given all the information leading up to and including the initial outage?" Answering this question can help to set better expectations for those affected by these outages, as well as to better channel the efforts of emergency response teams when needed.

### Exploring The Data

This dataset contains 1534 rows, each representing an individual outage event, and 57 columns detailing various characteristics of each outage. Below are some of the relevant columns for our analysis:

| Variable | Description |
|----------|----------|
| `YEAR`    | Indicates the year when the outage event occurred |
| `MONTH`    | Indicates the month when the outage event occurred  |
| `U.S._STATE`    |  Represents all the states in the continental U.S.  |
| `NERC.REGION`    |  The North American Electric Reliability Corporation (NERC) regions involved in the outage event   |
| `CLIMATE.REGION`   | U.S. Climate regions as specified by National Centers for Environmental Information   |
| `CLIMATE.CATEGORY`   |  This represents the climate episodes corresponding to the years   |
| `ANOMALY.LEVEL`   |  This represents the oceanic El Niño/La Niña (ONI) index referring to the cold and warm episodes by season   |
| `OUTAGE.START.DATE`   |  	This variable indicates the day of the year when the outage event started  |
| `OUTAGE.START.TIME`   |  This variable indicates the time of the day when the outage event started  |
| `OUTAGE.RESTORATION.DATE`  |  This variable indicates the day of the year when power was restored to all the customers   |
| `OUTAGE.RESTORATION.TIME`  |   This variable indicates the time of the day when power was restored to all the customers |
| `OUTAGE.DURATION`  |   Duration of outage events (in minutes)  |
| `CAUSE.CATEGORY`   |  Categories of all the events causing the major power outages   |
| `CUSTOMERS.AFFECTED`  |   Number of customers affected by the power outage event   |

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

In order to access this data, one can download the data from Purdue's engineering website [here](https://engineering.purdue.edu/LASCI/research-data/outages) as an xlsx document. I then loaded it into jupyter notebook using the `pd.read_excel` command. While loading it in, I skipped the first 5 rows and deleted the first column and row of the data, which leaves only the 57 columns and 1534 rows of data. In addition to this, I combined the `OUTAGE.START.DATE` and `OUTAGE.START.TIME` columns into a `pd.Timestamp` column named `OUTAGE.START`. I similarly combined `OUTAGE.RESTORATION.DATE` and `OUTAGE.RESTORATION.TIME` into one column named `OUTAGE.RESTORATION`. On top of this, I divided every `OUTAGE.DURATION` value by 60, to convert it from minutes to hours. Finally, I removed the columns of data that are unnecessary to my current investigation, leaving just the columns: `OUTAGE.START`, `CLIMATE.REGION`, `CLIMATE.CATEGORY`, `CAUSE.CATEGORY`, `CUSTOMERS.AFFECTED`, `ANOMALY.LEVEL`, `NERC.REGION`, `U.S._STATE`, and `OUTAGE.DURATION`. With all of this done, here is what the head of my DataFrame looks like:

| OUTAGE.START        | CLIMATE.REGION     | CLIMATE.CATEGORY   | CAUSE.CATEGORY     | CUSTOMERS.AFFECTED | ANOMALY.LEVEL | NERC.REGION | U.S._STATE | OUTAGE.DURATION |
|---------------------|--------------------|---------------------|---------------------|---------------------:|----------------:|:-------------|:------------|------------------:|
| 2011-07-01 17:00:00 | East North Central | normal              | severe weather      |               70000 |           -0.3 | MRO         | Minnesota   |             3060 |
| 2014-05-11 18:38:00 | East North Central | normal              | intentional attack  |                   NaN |           -0.1 | MRO         | Minnesota   |                1 |
| 2010-10-26 20:00:00 | East North Central | cold                | severe weather      |               70000 |           -1.5 | MRO         | Minnesota   |             3000 |
| 2012-06-19 04:30:00 | East North Central | normal              | severe weather      |               68200 |           -0.1 | MRO         | Minnesota   |             2550 |
| 2015-07-18 02:00:00 | East North Central | warm                | severe weather      |              250000 |            1.2 | MRO         | Minnesota   |             1740 |

With this initial data cleaning done, I shifted to the task of dealing with null values. Using a quick query for all remaining null cells, I determined that 483 rows contained at least one null value. This means if we were to remove all of these rows, we would erase nearly 30% of our data. To tackle this more effectively, let's focus on each of these variables one by one.

| Variable             | Number of Null Values |
|----------------------|------------------------|
| `OUTAGE.START`         | 9                      |
| `CLIMATE.REGION`       | 6                      |
| `CLIMATE.CATEGORY`     | 9                      |
| `CAUSE.CATEGORY`       | 0                      |
| `CUSTOMERS.AFFECTED`   | 443                    |
| `ANOMALY.LEVEL`        | 9                      |
| `NERC.REGION`          | 0                      |
| `U.S._STATE`           | 0                      |
| `OUTAGE.DURATION`      | 58                     |

Since variables like `CLIMATE.REGION`, `CLIMATE.CATEGORY`, and `ANOMALY.LEVEL` are all specific data points that are not easy to reliably predict, and there are not that many null values for these categories, we will simply remove all rows where these columns are null. This deletion removed a total of 14 rows. Additionally, since we will eventually try to predict the OUTAGE.DURATION variable, it does not make sense to impute values here which would introduce bias into our prediction model. Therefore, we will also remove all rows where OUTAGE.DURATION is null. This deletion removed another 49 rows. In the process of these deletions, we inadvertently removed all rows where `OUTAGE.START` was null, which means we do not need to do any additional deletions for that column. For the moment, I will skip over `CUSTOMERS.AFFECTED` which will be covered next in the imputation section. This cleaning process left us with 1471 rows of data which is a large percentage of our original data and still plenty of rows to perform a thorough analysis.

### Imputation

The `CUSTOMERS.AFFECTED` column presents more of a challenge because it has 443 missing values. Removing all of these rows would significantly reduce the dataset size, which could hinder the effectiveness of our predictive model. Instead, I will perform an imputation that attempts to accurately replace these values. 

For the `CUSTOMERS.AFFECTED` variable, I decided to use a median imputation strategy grouped by `U.S._STATE`. Since these values appear to be missing at random, it is reasonable to impute them using typical values (median or mean). I choose the median specifically, because the data is extremely right skewed, and the median helps preserve the overall distribution more consistently. Additionally, underlying factors like population, geography, infrastructure, etc. would likely influence `CUSTOMERS.AFFECTED`. For this reason, I grouped each median by `U.S._STATE`, which helps capture these underlying patterns. Here is what the distributions of `CUSTOMERS.AFFECTED` looked like before and after imputation:

<iframe src="assets/figure1.html"
        width="800"
        height="425"
        frameborder="0">
 </iframe>



 <iframe src="assets/figure2.html"
        width="800"
        height="425"
        frameborder="0">
 </iframe>

Despite the imputation of 443 missing values, the distribution appears to remain pretty consistent. With the previous deletions and imputation, we have successfully removed all rows with any null values:

| Variable             | Number of Null Values |
|----------------------|------------------------|
| `OUTAGE.START`         | 0                      |
| `CLIMATE.REGION`       | 0                      |
| `CLIMATE.CATEGORY`     | 0                      |
| `CAUSE.CATEGORY`       | 0                      |
| `CUSTOMERS.AFFECTED`   | 0                      |
| `ANOMALY.LEVEL`        | 0                      |
| `NERC.REGION`          | 0                      |
| `U.S._STATE`           | 0                      |
| `OUTAGE.DURATION`      | 0                      |

### Univariate Analysis

In order to get a better understanding of the data before building a predictive model, I looked at various relationships between variables in the dataset. First, I conducted a couple of univariate analyses, starting with the distribution of the `OUTAGE.DURATION`:
 
 <iframe src="assets/figure3.html"
        width="800"
        height="425"
        frameborder="0">
 </iframe>

This chart shows that a vast majority of the outages are very short, which would not be as negatively impactful to those in the outage region. Nevertheless, we see a relatively steady distribution of outages beyond even 24 or 48 hours, with a total of 82 outages that lasted over a week. These extra-long outages would likely be the ones that a proper prediction of duration would yield the most benefit. Hopefully, our predictive model will be able to find differences between the many short outages and the longer, more series outages.

Next, I looked at the distribution of the `CAUSE.CATEGORY` variable:

 <iframe src="assets/figure4.html"
        width="800"
        height="425"
        frameborder="0">
 </iframe>

This visual shows that severe weather is the largest cause, followed relatively closely by intentional attacks. Seeing the difference in number of outages under each category will help verify the solidity of our prediction. This will help add nuance to our predictions, since causes with more data points (more common) may allow us to trust our model's output more than causes with fewer data points (less common).

### Bivariate Analysis

Next, I will continue my exploration with a bivariate analysis of a few variables. First, I looked at the distribution of `OUTAGE.DURATION` across `CAUSE.CATEGORY`:
 
 <iframe src="assets/figure5.html"
        width="800"
        height="525"
        frameborder="0">
 </iframe>

This figure highlights the point that different outage causes have dramatically different distributions of their resulting power outage durations. This means that incorporating `CAUSE.CATEGORY` into our model could carry meaningful predictive weight.

Continuing, I looked at the relationship between `OUTAGE.DURATION` and `CUSTOMERS.AFFECTED`:

 <iframe src="assets/figure6.html"
        width="800"
        height="425"
        frameborder="0">
 </iframe>

Although this graph seems to be somewhat chaotic at first glance, we do see that for each cause there appears to be general underlying relationships between duration and customers affected. In instances like Severe Weather and Equipment Failure, this relationship seems more linear, while in instances like Intentional Attack and Fuel Supply Emergency, this relationship may be non-linear. Noting the linear and non-linear relationships between these variables will be useful when making our final predictive model.

### Interesting Aggregates

Finally, for the last stage of my exploratory analysis, I looked at aggregate statistics of several groupings that should help reveal what variables will carry predictive weight for `OUTAGE.DURATION`.

#### Outage Duration by Region

| Region              | Mean Duration | Std Duration |
|---------------------|---------------|--------------|
| Central             | 45.02         | 71.81        |
| East North Central  | 89.20         | 199.74       |
| Northeast           | 49.86         | 87.14        |
| Northwest           | 21.41         | 46.14        |
| South               | 47.44         | 85.48        |
| Southeast           | 36.96         | 60.47        |
| Southwest           | 26.10         | 123.08       |
| West                | 27.14         | 78.48        |
| West North Central  | 16.28         | 47.79        |

This table shows that `OUTAGE.DURATION` varies considerably across `CLIMATE.REGION`, pointing to the fact that it might be an important feature for prediction. This is likely because the region captures underlying factors such as weather patterns, infrastructure, and population dynamics, that impact outages.

#### Outage Duration by Climate

| Climate              | Mean Duration | Std Duration |
|---------------------|---------------|--------------|
| Cold             | 44.33         | 112.40        |
| Normal  | 42.46         | 89.66       |
| Warm           | 47.47         | 100.45       |

Despite the climate type seeming like it would have predictive power, the table shows far less variability in `OUTAGE.DURATION` based on `CLIMATE.CATEGORY`. This means we will likely prioritize other variables before resorting to `CLIMATE.CATEGORY` in our model.

#### Outage Duration by NERC Region

| NERC Region   | Mean Duration | Std Duration |
|---------------|---------------|---------------|
| ECAR          | 93.39         | 49.89         |
| FRCC          | 71.19         | 95.83         |
| FRCC, SERC    | 6.20          | NaN           |
| MRO           | 51.14         | 82.14         |
| NPCC          | 54.37         | 112.61        |
| RFC           | 58.10         | 128.61        |
| SERC          | 28.97         | 54.90         |
| SPP           | 44.90         | 71.07         |
| TRE           | 49.34         | 90.35         |
| WECC          | 24.80         | 82.11         |

Similarly to the first table, we see notable differences in `OUTAGE.DURATION` based on `NERC.REGION`. Since `NERC.REGION` also corresponds to large sections of the U.S., we might get some degree of redundancy including both `NERC.REGION` and `CLIMATE.REGION` in our final model. This introduces multicollinearity which would make certain linear models unstable, but that could be avoided using tree-based models. This is important to note and will guide us to use only one of the two if incorporating a linear model.

#### Outage Duration by State

| U.S. State             | Mean Duration | Std Duration |
|------------------------|---------------|---------------|
| Alabama                | 19.21         | 20.96         |
| Arizona                | 75.88         | 225.40        |
| Arkansas               | 25.24         | 42.85         |
| California             | 27.77         | 79.76         |
| Colorado               | 15.02         | 21.38         |
| Connecticut            | 21.31         | 32.39         |
| Delaware               | 2.42          | 11.26         |
| District of Columbia   | 71.73         | 85.92         |
| Florida                | 68.24         | 94.64         |
| Georgia                | 22.42         | 16.34         |
| Idaho                  | 6.91          | 8.94          |
| Illinois               | 26.71         | 29.41         |
| Indiana                | 58.69         | 96.24         |
| Iowa                   | 79.90         | 120.27        |
| Kansas                 | 72.94         | 108.09        |
| Kentucky               | 84.90         | 120.65        |
| Louisiana              | 68.08         | 110.34        |
| Maine                  | 18.28         | 19.89         |
| Maryland               | 38.55         | 48.21         |
| Massachusetts          | 15.74         | 23.60         |
| Michigan               | 88.38         | 151.50        |
| Minnesota              | 45.47         | 44.07         |
| Mississippi            | 1.40          | 2.41          |
| Missouri               | 56.23         | 77.35         |
| Nebraska               | 40.93         | 79.39         |
| Nevada                 | 9.22          | 13.42         |
| New Hampshire          | 4.66          | 11.18         |
| New Jersey             | 74.18         | 74.41         |
| New Mexico             | 2.34          | 2.34          |
| New York               | 100.58        | 151.32        |
| North Carolina         | 24.29         | 32.92         |
| North Dakota           | 12.00         | NaN           |
| Ohio                   | 47.80         | 58.76         |
| Oklahoma               | 50.32         | 63.90         |
| Oregon                 | 12.78         | 21.00         |
| Pennsylvania           | 63.53         | 59.35         |
| South Carolina         | 52.25         | 47.16         |
| Tennessee              | 17.37         | 24.55         |
| Texas                  | 45.08         | 85.90         |
| Utah                   | 4.17          | 10.80         |
| Vermont                | 0.59          | 0.88          |
| Virginia               | 17.52         | 22.06         |
| Washington             | 25.14         | 52.41         |
| West Virginia          | 116.32        | 135.58        |
| Wisconsin              | 131.74        | 413.40        |
| Wyoming                | 0.56          | 0.72          |

Finally, we zoom into each region and see a large degree of variation across each `U.S._STATE`. Since these are far more specific than any larger regions, we can safely add this variable into our model, which will likely have a meaningful predictive power.


## Framing a Prediction Problem

For the final prediction, I will be trying to answer the question "Can we predict the duration of power outages?" This prediction will make use of the power outage dataset to predict the target variable `OUTAGE.DURATION`. Being able to accurately estimate the duration of outages could be incredibly valuable for guiding public expectations and both personal and government resource allocation during severe power outages. Since `OUTAGE.DURATION` is in the units of hours, which is a continuous variable, this is a regression problem. In order to make this realistic, I will only use variables that contain information that would be readily available shortly after a power outage has occurred. Variables related to the location (`U.S._STATE`, `NERC.REGION`), the climate (`CLIMATE.REGION`, `CLIMATE.CATEGORY`, `ANOMALY.LEVEL`), the cause (`CAUSE.CATEGORY`), and the number of impacted customers (`CUSTOMERS.AFFECTED`), should all either be identifiable either immediately after or following a short inspection after the initial outage. For these reasons, I think all these variables are fair game. Finally, since this is a regression task and a situation where outliers (long outages) are very important to be predicted properly, I will be using Mean Squared Error to evaluate my model. The MSE will penalize larger errors more heavily, ensuring proper prediction of longer outages. I will also use R² as a normalized performance baseline to easily compare different iterations of the model.

## Baseline Model

For the baseline model, I will use the variables from earlier that seemed to have the most obvious predictive impact and that could be immediately determined once the location of the power outage is determined. These specific variables are:

-`CAUSE.CATEGORY` (nominal)
-`CLIMATE.REGION` (nominal)
-`U.S._STATE` (nominal)
-`ANOMALY.LEVEL` (quantitative)

This will give an initial model which I can evaluate and iterate upon. For this initial model, I encoded all of the nominal variables with `OneHotEncoder` and standardized the quantitative variable with `StandardScaler` (which will not improve the effectiveness of the model but will allow us to analyze the coefficients if needed). In addition, to keep the model quick and simple, I used a `LinearRegression` model.

 <iframe src="assets/figure7.html"
        width="800"
        height="425"
        frameborder="0">
 </iframe>

| Performance Metric | Performance |
|----------|----------|
| Mean Squared Error | 10193.898269331174 |
| R² Score | 0.08269072178571046 |

Unfortunately, our initial model only achieved an R² of 0.08, which means it barely beats out a model that just predicts the mean of `OUTAGE.DURATION`. We see from the graph above that my predictions (orange) do not seem to estimate the large spikes in the actual durations (blue) very well. These factors indicate that the model likely does not yet generalize well to unseen data. Nevertheless, the prediction does accurately predict some small aspects of the variance, giving us hope to improve our final model.

## Final Model

Our final model will take in all the variables from our previous model and incorporate additional ones that might edge our final predictions closer to the real values. These new variables will consist of: 

- `CUSTOMERS.AFFECTED` (quantitative): The *Bivariate Analysis* section showed that this variable might contain both linear and non-linear relationships with `OUTAGE.DURATION` which will likely aid our final model's predictive power
- `CLIMATE.CATEGORY` (nominal): As explained in the *Interesting Aggregates* section, this variable might provide some predictive power beyond `CLIMATE.REGION`
- `NERC.REGION` (nominal): As touched on in *Interesting Aggregates*, this variable will provide slight nuance to the `CLIMATE.REGION` variable; multi-collinearity will not be a problem if using a tree-based final model

We will continue to use mean squared error to evaluate our final model for the same reasons as before (and similarly incorporating R² as a benchmark).

In addition, I will engineer a few new variables from our given list to help better capture non-linear relationships between variables: 

1. `Polynomial(5)`: I added a polynomial degree five transformation to both of the quantitative variables. As mentioned earlier, `CUSTOMERS.AFFECTED` clearly has a non-linear relationship with `OUTAGE.DURATION` which could be better represented with this polynomial form. Additionally, `OUTAGE.DURATION` seems to fluctuate between increasing and decreasing as `ANOMALY.LEVEL` increases. I choose polynomial degree 5 in order to capture multiple changes in directions as described.
2. `CAUSE.CATEGORY` Flag: I created a flag based on `CAUSE.CATEGORY` which shows a 1 if the outage cause was *severe weather* or *fuel supply emergency* and a 0 otherwise. This was motivated by the graph in *Bivariate Analysis* that shows these categories having substantially longer power outages than other causes. By flagging these categories, it allows the model to specifically give an additional weight to these causes which likely have distinct impacts on outage duration.

Finally, I decided to switch to a `RandomForestRegressor` to better capture the non-linear relationship between these variables and to improve model stability despite the collinearity mentioned earlier. In addition, I will use `GridSearchCV`, which will use cross-validation to tune the model's hyperparameters under *neg_mean_squared_error*. I will tune the `RandomForestRegressor`'s:

- `max_depth`: maximum depth of each tree; helps determine balance between bias and variance; values = `[10, 15, 20, 25]`
- `n_estimators`: number of decision trees in forests; helps ensure stability and generalization of model; values = `[50, 100, 150, 200]`
- `min_samples_split`: minimum samples required to split a node; reduces overfitting on noisy data; values = `[2, 3, 4, 5]`
- `min_samples_leaf`: minimum samples at a leaf node; improves generalization of model; values = `[1, 2, 3, 4]`

After performing `GridSearchCV` on these parameters, the model selected the following hyperparameters:

- `max_depth` = 15
- `n_estimators` = 50
- `min_samples_split` = 3
- `min_samples_leaf` = 1

With these parameters, here are the results of our improved final model:

 <iframe src="assets/figure8.html"
        width="800"
        height="425"
        frameborder="0">
 </iframe>

 | Performance Metric | Initial Performance | Final Performance |
|----------|----------|----------|
| Mean Squared Error | 10193.898269331174 | 8859.741271912088 |
| R² Score | 0.08269072178571046 | 0.20274632367542234 |

The R² has bumped up significantly to 0.20 which is a vast improvement over our initial model. Additionally, the MSE reduced by more than 1300, signaling a much better fit on testing data. Based on the graph above, we can see that the predictions (orange) do a better job of capturing the larger spikes in the actual outage durations (blue) which was one of the most important goals for our model. By capturing more non-linear relationships and incorporating more helpful inputs, our model has clearly improved over the baseline, giving us a solid foundation for predicting outage duration on unseen data. A model like this will hopefully make it easier for major power outages to be handled by those affected and supported by local and national organizations in an impactful way.
 
