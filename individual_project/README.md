# Flight Delay Analysis

This directory contains an R-based Jupyter notebook that explores the 2024 flight delay dataset and tests whether a small set of schedule-related variables can predict whether a flight arrives late.

## Project goal

The notebook asks a practical question: given information known before departure, can we identify flights that are more likely to arrive late?

The analysis focuses on a sample of 10,000 flights drawn from a larger 2024 dataset and uses logistic regression to model arrival delay as a binary outcome.

## Data source

The dataset comes from Kaggle:

- https://www.kaggle.com/datasets/hrishitpatil/flight-data-2024/data

The original source file is described as a large flight dataset containing operational and scheduling fields for flights, including departure time, arrival time, delay values, cancellation status, airport metadata, and delay components.

## Files used in the notebook

The analysis reads the following files from the same working directory:

- `flight_data_2024_sample.csv` — sampled flight records used for modeling
- `flight_data_2024_data_dictionary.csv` — column descriptions and metadata

## Notebook workflow

### 1. Loading and inspection

The notebook reads the sample CSV into a dataframe and confirms its size:

- 10,000 rows loaded initially

It also inspects column metadata, including data types and missing-value percentages. This step is important because flight datasets commonly contain incomplete records in fields such as `dep_time`, `arr_time`, and delay-related metrics.

### 2. Data cleaning

The notebook removes rows with any missing values using `na.omit()`. This reduces the data to 9,836 usable rows.

Important note:

- 164 rows were discarded due to missing values
- The remaining dataset is used for all model building and interpretation

### 3. Response variable creation

A binary response variable is created:

- `arr_delay_bin = 1` if `arr_delay > 0`
- `arr_delay_bin = 0` otherwise

This converts the continuous arrival delay variable into a classification problem, where the question becomes: is the flight late or not?

### 4. Candidate predictors

The notebook begins with a set of variables that are known before the flight occurs, making them useful for prediction:

- `month`
- `day_of_week`
- `crs_arr_time`
- `crs_dep_time`
- `crs_elapsed_time`
- `distance`

These are schedule- and route-related variables rather than post-departure operational outcomes.

### 5. Correlation analysis

A correlation matrix is computed for the numeric predictors.

The key finding is that `crs_elapsed_time` and `distance` are strongly correlated (about 0.98), which indicates redundancy. Because `crs_elapsed_time` is a schedule-derived metric and `distance` is highly overlapping with it, the notebook removes `distance` to reduce multicollinearity before fitting the model.

This is a sound modeling choice: when two predictors carry almost the same information, they can inflate standard errors and make interpretation less stable.

## Modeling strategy

The notebook focuses on arrival delay, rather than both departure and arrival delay, to keep the analysis clean and interpretable.

### Baseline logistic regression

The first model is:

`arr_delay_bin ~ crs_arr_time + crs_dep_time + crs_elapsed_time + month + day_of_week`

This model is fit with a binomial logistic regression.

The model summary shows that a subset of predictors significantly explains arrival delay behavior:

- `crs_dep_time`
- `month`
- `day_of_week`

The notebook interprets the lower residual deviance compared with the null model as evidence that the predictors add explanatory power.

### Extended categorical model

To better understand patterns by month and day, the notebook converts:

- `month` to a factor
- `day_of_week` to a factor

This allows the model to estimate different effects for each calendar month and weekday rather than assuming a linear effect across categories.

The extended model uses:

`arr_delay_bin ~ crs_arr_time + crs_dep_time + crs_elapsed_time + month_factor + day_factor`

This model produces a lower AIC than the more compact version, suggesting improved fit.

## Key findings from the fitted models

### Timing matters most

The strongest predictor in the model is the scheduled departure time, `crs_dep_time`. This suggests that flights scheduled at certain periods of the day are more likely to arrive late, likely because of traffic, congestion, and operational bottlenecks.

### Month matters

The notebook compares average delay rates across months and finds:

- Highest delay probability: July
- Lowest delay probability: October

This indicates seasonal variation in arrival delays, which may reflect demand patterns, weather, or seasonal scheduling effects.

### Day of week matters

The day-level analysis finds:

- Highest delay probability: Sunday
- Lowest delay probability: Tuesday

This suggests that weekend schedules may be disproportionately affected by congestion or demand differences.

### Best overall interpretation

The notebook concludes that a flight scheduled for a later departure time, during certain months, or on certain weekdays is more likely to arrive late. The strongest practical takeaway is that flight delay risk is not random; it is associated with scheduling and temporal patterns.

## Model comparison and validation

The notebook fits several logistic-regression variants:

1. A baseline model using numeric month/day predictors
2. A reduced model containing only the most important variables
3. A model with an interaction term between month and day of week

The comparison shows:

- The first extended factor-based model performs best
- The interaction model does not improve results meaningfully
- The ANOVA-like comparison confirms that the factor-based model provides the strongest predictive structure

This supports the interpretation that the main signal is driven by time-based schedule factors rather than a special interaction between month and day.

## Conclusion

This notebook demonstrates a straightforward but effective approach to flight delay prediction:

- clean the data
- define a binary delay outcome
- remove redundant variables
- fit logistic regression models
- interpret the seasonality and scheduling signals

The final takeaway is that arrival delays can be partially explained by pre-departure schedule information, particularly:

- departure time
- month
- day of week

In other words, some of the variation in flight delays is systematic and predictable rather than purely random.

## Future work

Possible follow-up analyses include:

- modeling departure delay instead of arrival delay
- including airport-level effects (origin/destination)
- testing airline-specific differences
- comparing logistic regression with tree-based models such as random forests or gradient boosting
- evaluating model performance with train/test splits or cross-validation

## Summary

This project is a compact applied statistics exercise in predictive analysis using real-world aviation data. It showcases how logistic regression can be used to analyze operational patterns and link them to quantifiable scheduling factors that impact flight performance.

The notebook is a good example of how data cleaning, correlation checks, variable selection, and model interpretation work together in a realistic forecasting workflow.
