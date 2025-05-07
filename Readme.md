# Solar Dayahead Forecast Data
**21th** place solution for NTNU course TDT4173 ML fall 2023 Kaggle competition on solar energy production forecasting (https://www.kaggle.com/competitions/solar-energy-production-forecasting).

10 out of 10 virtual teams defeated

## Definition
This dataset provides data for evaluating solar production dayahead forecasting methods.
The dataset contains three locations (A, B, C), corresponding to office buildings with solar panels installed.
There is one folder for each location.

There are 4 files in each folder:

1. train_targets.parquet - target values for the train period (solar energy production)
2. X_train_observed.parquet - actual weather features for the first part of the training period
2. X_train_estimated.parquet - predicted weather features for the remaining part of the training period
2. X_test_estimated.parquet - predicted weather features for the test period

For Kaggle submissions we have two more files: 
1. test.csv — test file with zero predictions (for all three locations)
2. sample_submission_kaggle.csv — sample submission in the Kaggle format (for all three locations)

Kaggle expects you to submit your solutions in the "sample_sumbission_kaggle.csv" format. Namely, you need to have two columns: "id" and "prediction".
The correspondence between id and time/location is in the test.csv file. An example solution is provided in "read_files.ipynb"

All files that are in the parquet format that can be read with pandas:
```shell
pd.read_parquet()
```

Baseline and targets production values have hourly resolution.
Weather has 15 min resolution.
Weather parameter descriptions can be found [here](https://www.meteomatics.com/en/api/available-parameters/alphabetic-list/).

There is a distinction between train and test features.
For training, we have both observed weather data and its forecasts, while for testing we only have forecasts.
While file `X_train_observed.parquet` contains one time-related column `date_forecast` to indicate when the values for the current row apply,
both `X_train_estimated.parquet` and  `X_test_estimated.parquet` additionally contain `date_calc` to indicate when the forecast was produced.
This type of test data makes evaluation closer to how the forecasting methods that are used in production.
Evaluation measure is [MAE](https://en.wikipedia.org/wiki/Mean_absolute_error).

## Our Resolution
The final solution was achieved after several months of **data analysis, preprocessing, model experimentation, optimization, and ensemble learning**, all implemented in **Python**.

---

### 🔧 Data Preprocessing

A substantial part of the project involved cleaning and transforming the data. Key preprocessing steps included:

- **Reading and exploring data** using `pandas`
- **Date handling**: converting timestamps into datetime objects and extracting features like year, month, and day
- **Merging multiple datasets** into a single table with **one-hot encoding** to indicate the source dataset
- **Temporal aggregation**: the original data recorded every 15 minutes was **aggregated to hourly records** using the **median** (which empirically performed better than the mean)
- **Handling missing values** by replacing `NaN`s with 0
- **Dropping non-informative features** to improve model clarity and performance

---

### 🧠 Base Models Used

We trained and stacked several powerful regression models, including:

- **LightGBM (lgbm)**: a fast, efficient gradient boosting framework developed by Microsoft
- **LGBMRegressor (lgbm_reg)**: a LightGBM variant optimized for continuous regression tasks
- **CatBoost**: a boosting algorithm by Yandex, designed to handle categorical variables natively
- **XGBoost**: a highly popular and competitive gradient boosting method
- **Random Forest**: a classic ensemble method based on bagging with decision trees

---

### 🔗 Final Model: **Stacking Ensemble with Linear Regression**

We implemented a **stacking ensemble approach**, where the outputs of all base models were used as input features for a **Linear Regression** meta-model. This model learns to **optimally combine** the individual predictions to minimize the final error:

y^=w1⋅y^1+w2⋅y^2+w3⋅y^3+⋯+b

This linear combination helps capture the strengths of each base model, improving overall performance and generalization.


