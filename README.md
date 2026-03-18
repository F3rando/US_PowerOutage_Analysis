## Illuminating the Grid: A Predictive Analysis of U.S. Power Outage Resilience

**By: Luis Corona**

## Introduction

Power outages are more than a temporary inconvenience, they are a critical indicator of a nation's infrastructure resilience. In an era where extreme weather events are becoming more frequent, understanding how and why our electrical grids fail is essential for effective urban planning and emergency response.

This project investigates a dataset of major power outages in the United States, seeking to answer one central question:

> **What factors—specifically related to geography, climate, and population density—are the most significant predictors of power outage duration?**

## The Dataset

The data utilized in this analysis contains 1,534 rows, each representing a major power outage in the U.S. occurring between 2000 and 2016. By analyzing these records, I aim to identify patterns in how different regions respond to and recover from grid failures.

## Relevant Columns

To answer my research question and build a predictive model, I focused on the following key features:

- `**OUTAGE.DURATION` (Minutes)**: My target variable, the total time from the start of the outage until power was fully restored.
- `**CLIMATE.REGION`**: The geographical area where the outage occurred (e.g., Northeast, Central).
- `**CAUSE.CATEGORY`**: The primary reason for the outage (e.g., Severe Weather, Equipment Failure).
- `**ANOMALY.LEVEL`**: A quantitative measure of climate anomalies (like El Niño/La Niña) present during the outage.
- `**POPDEN_URBAN**`: The population density of urban areas in the affected state, which serves as a proxy for infrastructure complexity and repair priority.

## Data Cleaning and Exploratory Data Analysis

To transform the raw power outage data into a format suitable for analysis and predictive modeling, I performed several key cleaning and feature engineering steps. These were chosen to better reflect the data-generating process—specifically how time and geography influence infrastructure failure.

- **Timestamp Integration**: Combined the separate date and time columns for the start and restoration of outages into unified `pd.Timestamp` objects. This enabled precise calculation of `OUTAGE.DURATION` and facilitated the extraction of seasonal information.
- **Numeric Conversion**: Converted columns such as `ANOMALY.LEVEL` and `POPDEN_URBAN` from strings to numeric types to support mathematical modeling and statistical analysis.
- **Target Variable Refinement**: Removed rows with missing `OUTAGE.DURATION` values, as these incomplete records could bias model estimates and degrade predictive performance.
- **Feature Engineering**: Derived a `SEASON` feature from the outage start month, recognizing that grid stress is often driven by seasonal weather patterns (e.g., summer heat vs. winter storms).

### Cleaned Data (First 5 Rows)

The table below shows a sample of the cleaned dataset used for modeling:

> The cleaned sample illustrates how temporal, climatic, and demographic variables come together in a structured format suitable for exploratory analysis and prediction.


| YEARMONTH | CLIMATE.REGION     | CAUSE.CATEGORY     | OUTAGE.DURATION | POPDEN_URBAN |
| --------- | ------------------ | ------------------ | --------------- | ------------ |
| 20117     | East North Central | severe weather     | 3060            | 2279         |
| 20145     | East North Central | intentional attack | 1               | 2279         |
| 201010    | East North Central | severe weather     | 3000            | 2279         |
| 20126     | East North Central | severe weather     | 2550            | 2279         |
| 20157     | East North Central | severe weather     | 1740            | 2279         |


### Univariate Analysis

The univariate analysis focuses on understanding the marginal distribution of outage durations and other key variables before introducing more complex relationships.

<iframe src="assets/my_plot_Distribution of Outage Durations.html" width="100%" height="450" frameborder="0"></iframe>

> This histogram displays the distribution of outage durations, revealing a heavily right skewed pattern: most outages are resolved relatively quickly, but a small number of extreme events last for several days and exert a disproportionate influence on the mean.

### Bivariate Analysis and Interesting Aggregates

For the bivariate analysis, I examined the relationship between geographical regions and the duration of outages to identify spatial patterns in grid resilience.

<iframe src="assets/my_plot_Distribution of Outage Durations by Climate Region.html" width="100%" height="450" frameborder="0"></iframe>

> By comparing climate region against outage duration, we can see how geographical context influences restoration speed. Regions such as the Northeast exhibit a higher frequency of long duration outages, likely linked to the intensity and type of local weather events.

To further explore how climate, cause, and population density interact, I constructed aggregate tables summarizing average outage durations.


| CLIMATE.REGION     | equipment failure | fuel supply emergency | intentional attack | islanding | public appeal | severe weather | system operability disruption |
| ------------------ | ----------------- | --------------------- | ------------------ | --------- | ------------- | -------------- | ----------------------------- |
| Central            | 322               | 10035.2               | 346.06             | 125.33    | 1410          | 3250.01        | 2695.2                        |
| East North Central | 26435.3           | 33971.2               | 2376.05            | 1         | 733           | 4434.82        | 2610                          |
| Northeast          | 215.8             | 14629.6               | 195.98             | 881       | 2655          | 4429.9         | 773.5                         |
| Northwest          | 702               | 1                     | 373.81             | 73.33     | 898           | 4838           | 141                           |
| South              | 295.78            | 17482.5               | 325.61             | 493.5     | 1163.98       | 4391.35        | 866.07                        |
| Southeast          | 554.5             | nan                   | 504.67             | nan       | 2865.4        | 2662.56        | 169.31                        |
| Southwest          | 113.8             | 76                    | 265.67             | 2         | 2275          | 11572.9        | 329.22                        |
| West               | 524.81            | 6154.6                | 857.68             | 214.86    | 2028.11       | 2928.37        | 363.67                        |
| West North Central | 61                | nan                   | 23.5               | 68.2      | 439.5         | 2442.5         | nan                           |


> This pivot-style summary highlights how outage duration varies across regions and causes. Severe Weather consistently produces the longest recovery times across nearly all regions, whereas Intentional Attacks are typically resolved much faster, underscoring the role of environmental stress in driving prolonged grid failures.

## Assessment of Missingness

### MNAR Analysis

In analyzing the data-generating process, I believe the column `**CAUSE.CATEGORY.DETAIL`** is likely **MNAR (Missing Not At Random)**. This column provides granular technical details about the specific cause of a power outage (e.g., "Insulator Failure" vs. "Tree Falling"). 

The missingness is likely dependent on the specific details themselves, if an outage was caused by a complex or rare event that was difficult for the reporting utility to categorize quickly, it is more likely to be left blank. To move this toward **MAR**, I would need additional data from utility maintenance logs or technician "trouble tickets" to see if reporting thoroughness correlates with infrastructure age or the seniority of the technician on duty.

### Missingness Dependency

I conducted permutation tests to determine if the missingness of the `**CUSTOMERS.AFFECTED`** column (the number of people impacted) was dependent on other observed variables.

- **Dependency on `CLIMATE.REGION` (MAR):** Using **Total Variation Distance (TVD)** as the test statistic, I performed a permutation test with 1,000 shuffles. I obtained an **Observed TVD of 0.2760** and a **P-value of 0.000**. Because the P-value is below our significance level of 0.05, we reject the null hypothesis. This indicates that the missingness of `CUSTOMERS.AFFECTED` is **MAR** and depends significantly on the geographical region.
- **Dependency on `YEAR` (MAR):** Interestingly, a second permutation test against the `**YEAR`** column also resulted in an **Observed TVD of 0.3059** and a **P-value of 0.000**. This suggests that missingness is not consistent over time, reporting practices for customer impact likely evolved or shifted significantly during the 16 year period covered by the dataset.

<iframe src="assets/missingness_plot.html" width="100%" height="450" frameborder="0"></iframe>

> **Interpretation:** The plot above displays the empirical distribution of the TVD statistic under the null hypothesis for the Climate Region test. My observed TVD (represented by the red dashed line) sits far in the tail of the distribution, resulting in a P-value of 0.000. This provides strong evidence that regional reporting practices—rather than random chance—drive the missingness in customer impact data.

