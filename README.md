## Illuminating the Grid: A Predictive Analysis of U.S. Power Outage Resilience

**By: Luis Corona**

## Introduction

Power outages are more than a temporary inconvenience, they are a critical indicator of a nation's infrastructure resilience. In an era where extreme weather events are becoming more frequent, understanding how and why our electrical grids fail is essential for effective urban planning and emergency response.

This project investigates a dataset of major power outages in the United States, seeking to answer one central question:

> **What factors—specifically related to geography, climate, and population density—are the most significant predictors of power outage duration?**

### The Dataset

The data utilized in this analysis contains 1,534 rows, each representing a major power outage in the U.S. occurring between 2000 and 2016. By analyzing these records, I aim to identify patterns in how different regions respond to and recover from grid failures.

### Relevant Columns

To answer my research question and build a predictive model, I focused on the following key features:

* **OUTAGE.DURATION (Minutes)**: My target variable, the total time from the start of the outage until power was fully restored.
* **CLIMATE.REGION**: The geographical area where the outage occurred (e.g., Northeast, Central).
* **CAUSE.CATEGORY**: The primary reason for the outage (e.g., Severe Weather, Equipment Failure).
* **ANOMALY.LEVEL**: A quantitative measure of climate anomalies (like El Niño/La Niña) present during the outage.
* **POPDEN_URBAN**: The population density of urban areas in the affected state, which serves as a proxy for infrastructure complexity and repair priority.

## Data Cleaning and Exploratory Data Analysis

To transform the raw power outage data into a format suitable for analysis and predictive modeling, I performed several key cleaning and feature engineering steps. These were chosen to better reflect the data-generating process—specifically how time and geography influence infrastructure failure.

- **Timestamp Integration**: Combined the separate date and time columns for the start and restoration of outages into unified `pd.Timestamp` objects. This enabled precise calculation of `OUTAGE.DURATION` and facilitated the extraction of seasonal information.
- **Numeric Conversion**: Converted columns such as `ANOMALY.LEVEL` and `POPDEN_URBAN` from strings to numeric types to support mathematical modeling and statistical analysis.
- **Target Variable Refinement**: Removed rows with missing `OUTAGE.DURATION` values, as these incomplete records could bias model estimates and degrade predictive performance.
- **Feature Engineering**: Derived a `SEASON` feature from the outage start month, recognizing that grid stress is often driven by seasonal weather patterns (e.g., summer heat vs. winter storms).

### Cleaned Data (First 5 Rows)

The table below shows a sample of the cleaned dataset used for modeling:

> The cleaned sample illustrates how temporal, climatic, and demographic variables come together in a structured format suitable for exploratory analysis and prediction.


| YEAR | MONTH | CLIMATE.REGION     | CAUSE.CATEGORY     | OUTAGE.DURATION | POPDEN_URBAN | ANOMALY.LEVEL |
| ---- | ----- | ------------------ | ------------------ | -------------- | ------------ | ------------ |
| 2011 | 7     | East North Central | severe weather     | 3060           | 2279         | -0.3         |
| 2014 | 5     | East North Central | intentional attack | 1              | 2279         | -0.1         |
| 2010 | 10    | East North Central | severe weather     | 3000           | 2279         | -1.5         |
| 2012 | 6     | East North Central | severe weather     | 2550           | 2279         | -0.1         |
| 2015 | 7     | East North Central | severe weather     | 1740           | 2279         | 1.2          |


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

## Hypothesis Testing

To further explore the factors influencing power outage recovery, I conducted a formal hypothesis test comparing the restoration times of two major regions: the Northeast and the West.

### Hypothesis Setup

- **Null Hypothesis (H0)**: The mean power outage duration in the Northeast region is equal to the mean power outage duration in the West region. Any observed difference is due to random chance.
- **Alternative Hypothesis (H1)**: The mean power outage duration in the Northeast region differs from the mean power outage duration in the West region.
- **Test Statistic**: Absolute difference in means.
- **Significance Level (\(\alpha\))**: 0.05.

### Justification

I chose the Absolute Difference in Means as the test statistic because my research question aims to identify whether geography is associated with outage duration, regardless of which region is specifically "worse." This two-sided approach provides a robust assessment of regional disparity. A permutation test is appropriate here because it does not assume outage durations are normally distributed, which is important given the strong right-skew observed in the EDA.

### Results and Conclusion

After conducting a permutation test with 1,000 shuffles, I obtained the following results:

- **Observed Absolute Difference**: 1,363.33 minutes.
- **P-value**: < 0.001.

<iframe src="assets/hypothesis_test_plot.html" width="100%" height="450" frameborder="0"></iframe>

Conclusion: Since the resulting P-value is far below our threshold of 0.05, we reject the null hypothesis. This suggests the observed difference in outage durations between the Northeast and West is unlikely to be explained by random chance alone. While this does not establish a causal link, it provides evidence that regional characteristics, such as infrastructure age, severe weather patterns, or population density, may contribute to differences in grid resilience.

## Framing a Prediction Problem

### Problem Identification

The goal of this analysis is to predict the duration of a major power outage (OUTAGE.DURATION).

Prediction Type: Regression.

Response Variable: OUTAGE.DURATION (minutes).

Evaluation Metric: Root Mean Squared Error (RMSE).

### Justification of Metric

I chose RMSE as the primary evaluation metric because it penalizes larger errors more heavily than metrics like Mean Absolute Error (MAE). In the context of emergency response and urban planning, a massive under prediction of outage duration (e.g., predicting 2 hours when it actually lasts 2 days) is significantly more costly for emergency services and resource allocation than a series of small, consistent errors. RMSE ensures the model prioritizes minimizing these high impact "misses."

### Time of Prediction

To ensure the model is practical and realistic, I am strictly using features that would be available at the "time of prediction", the moment the outage is first reported.

Included Features: I will utilize CLIMATE.REGION, ANOMALY.LEVEL, and urbanization data like POPDEN_URBAN. These factors are known geographical or climatic constants at the onset of an event.

Excluded Features (Data Leakage): I have excluded features such as CUSTOMERS.AFFECTED and DEMAND.LOSS.MW. These values are typically only finalized after the power has been fully restored, and using them to predict duration would result in "data leakage," leading to an unrealistically high (and practically useless) model performance.

## Baseline Model

### Model Description

For my baseline model, I implemented a Linear Regression model using a scikit-learn Pipeline. This model serves as a starting point to determine how much of the variance in outage duration can be explained by basic geographic and climatic factors.

### Features and Encodings

I incorporated two initial features into this model:

CLIMATE.REGION (Nominal): This feature was One-Hot Encoded to transform the categorical regional labels into a numerical format that the linear regression model can interpret.

ANOMALY.LEVEL (Quantitative): This feature was passed through the pipeline without further transformation, as it is already a continuous numeric value representing oceanic climate anomalies.

### Performance and Evaluation

To evaluate the model's ability to generalize to unseen data, I performed an 80/20 train-test split. The resulting performance on the test set was:

Baseline RMSE: ~7,739.85 minutes.

Is this a "good" model?
Currently, I would not consider this a "good" model. An RMSE of approximately 7,740 minutes represents a prediction error of roughly 5.3 days.

Given that many power outages are resolved within hours, this level of error is too high for practical emergency management. The poor performance is likely due to the simplicity of the features and the linear model's sensitivity to extreme outliers (the right-skewed "extreme events" identified in Section 2). This baseline establishes a clear "floor" for performance that I will attempt to improve upon in the Final Model.

## Final Model

### Model Description

For my final model, I transitioned from a simple Linear Regression to a Random Forest Regressor. I chose this algorithm because power outage durations are non-linear and driven by complex "if-then" interactions (e.g., IF it is winter AND the cause is severe weather, THEN restoration is significantly delayed). Random Forests excel at capturing these high dimensional relationships through an variety of decision trees.

### Engineered Features

I added three key features to better reflect the data generating process:

SEASON (Categorical): Captures seasonal recovery hurdles, such as snow and ice in winter versus high grid load in summer.

CAUSE.CATEGORY (Nominal): Provides the "why" behind the event. A hurricane requires structural rebuilding, while an "Intentional Attack" often involves a localized mechanical fix.

POPDEN_URBAN (Quantitative): Acts as a proxy for infrastructure density and repair priority.

### Hyperparameter Tuning

I used GridSearchCV with 3-fold cross-validation to optimize the following:

n_estimators (Number of Trees): I selected 100 trees to ensure a stable, averaged prediction that minimizes noise.

max_depth (Complexity): I tested depths of [None, 10, 20] to prevent the model from "overfitting", ensuring it learns generalizable patterns rather than memorizing individual historic outages.

### Performance and Visual Analysis

Baseline RMSE: ~7,739.85 minutes

Final Model RMSE: ~5,646.65 minutes (27% Improvement)

<iframe src="assets/final_model_residuals.html" width="100%" height="450" frameborder="0"></iframe>

Residual Interpretation: My residual analysis shows that the model is highly effective at predicting "standard" outages, as seen by the high density of errors tightly clustered around the zero-line. While the model naturally struggles with extreme "Black Swan" events (the outliers above 20,000 minutes), the symmetric distribution in the marginal histogram confirms that the Random Forest has successfully captured the central tendencies of U.S. grid resilience without significant systematic bias.

## Fairness Analysis

### Group Definitions

To evaluate the fairness of my final model, I divided my test set into two groups based on the median population density:

Group X (Urban): States with a population density above the median.

Group Y (Rural): States with a population density at or below the median.

### Hypothesis Setup

**Null Hypothesis (H0)**: The model is fair. Its RMSE for urban and rural states is roughly the same, and any observed differences are due to random chance.

**Alternative Hypothesis (H1)**: The model is unfair. Its RMSE for rural states is significantly different from its RMSE for urban states.

Evaluation Metric: Root Mean Squared Error (RMSE).

Test Statistic: Absolute difference in RMSE.

Significance Level (\(\alpha\)): 0.05.

### Results and Conclusion

After performing a permutation test with 1,000 shuffles, I obtained the following results:

RMSE for High Density (Urban): 4,764.96

RMSE for Low Density (Rural): 8,907.19

Observed Difference: 4,142.24

P-value: 0.9270

<iframe src="assets/fairness_analysis_plot.html" width="100%" height="450" frameborder="0"></iframe>

Conclusion: I fail to reject the null hypothesis. With a P-value of 0.9270, which is well above our significance threshold of 0.05, there is no statistically significant evidence to suggest that the model's performance varies unfairly between urban and rural areas. While the raw RMSE was higher in rural states, the permutation test indicates that this difference is likely driven by the presence of a few extreme, high-duration outliers in the rural subset rather than a systemic bias in the model's predictive ability.
