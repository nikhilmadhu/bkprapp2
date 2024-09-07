# Used Car Price Analysis
Detailed Analysis avaialble in this [Jupyter Notebook](https://github.com/nikhilmadhu/bkprapp2/blob/main/src/UsedCars.ipynb)

## Business Goal
- Identify the most important drivers of used car prices and what consumers value in a used car
- Create a model to predict used car prices based on the identified driver parameters

## Conclusions
- The top 5 features that drives the price of a used car are -
  - Year of manufacturing
  - Odometer reading
  - Number of Cylinders
  - Manufacturer ( Cars from manufacturers like Ferrari, Aston Martin and Tesla sell at a premium, where as Fiat fared very poorly)
  - Fuel (Diesel being more expensive than gas)
    ![coeff](https://github.com/user-attachments/assets/95c42413-5817-4130-a952-b38c02f712a4)
- In our model, the following features were the most influential : 
   - Type of 'drive' [fwd < rwd < 4wd]
   - Number of 'cylinders'
   - 'year' of manufacturing,
   - The amount of miles on the 'odometer'
   - 'size' of car [full-size > mid-size > compact > sub-compact]
     ![pif](https://github.com/user-attachments/assets/c86e4770-5cd5-4fb4-a60a-df7c5256155d)
    
 ## Actionable Insights
   - Older year gas cars from non-premium brands should be priced cheaper. Teslas command a premium
   - Reduce the price of cars if the title is not clean
   - If a car has a bigger engine (more cylinders), then, price it higher
   - Sedans and Hatchbacks are cheaper than SUVs and Convertibles. Let your prices show the same
   - 4WD should be priced higher than 2WD/RWD - Decide on Loss leaders using this if you have a tie

 ## Predictions
- Few Example Predictions and its comparison with the actual values :
  > | Actual | Predicted | Error     |
  > |-------|------------|-----------|
  > | 8507   | 7800       | 527.00    | 
  > | 243994 | 4000       | 3534.58    |
  > | 140101 | 4299       | 5855.51    |
  > | 175662 | 19990      | 24800.56   |
  > | 148590 | 6900       | 14035.26   |
  > | 247926 | 8780       | 6079.56    |
  > | 233137 | 46990      | 30076.55   |
  > | 303618 | 36984      | 26655.55   |
  > | 224315 | 8800       | 17803.79   |
  > | 149966 | 27399      | 23586.95   |
  > | 240981 | 5995       | 1626.11    |
  > | 51114  | 45988      | 33488.87   |
  > | 185792 | 9995       | 15226.92   |
  > | 24095  | 399        | 21666.94   |
  > | 117304 | 5000       | 8075.40    |
  > | 187289 | 7499       | 6526.05    |
  > | 134045 | 1250       | -2249.62   |
  > | 234205 | 343        | 27431.91   |
  > | 232099 | 6990       | 8884.81    |
  > | 297127 | 39999      | 36596.19   | 

- A sample spread of errors are captured below
    > ![error](https://github.com/user-attachments/assets/e0c141dd-e33b-4e1c-bf7d-f4df7c8929e9)
- Steps and information on how this model can be applied to predict the prices of new vehicles arriving at the dealership can be found in the accompanying Jupyter notebook.

  
# Detailed Analysis
## Data
- Contains 426K used car sales transactions, with the target variable being "price". The quality of data within the features were as follows
  ![data_before](https://github.com/user-attachments/assets/9b673c31-c8a2-4646-aecb-565d0d166003)

- ### Other issues noted and Data clensing steps taken  :
   - There were transactions with 0 price [these records were dropped]
   - There were trasactions with prices in the Billions [these records were dropped]
   - Large amount of missing values in 'drive', 'size', 'type', 'paint_color' and 'condition' fields [these records were filled using a three tiered approach, first the most common value for the manufacturer/model/year combination, then, the most common at manufacturer/model and finally from the model alone]
   - Multiple transactions with the same data for the same VIN [these duplicate records were dropped]
   - Rows with missing values in year, model, fuel, transmission and odometer fields [these records were dropped]
   - As the VIN number itself did not provide any material value, it ws converted to a boolean field that showed the presence / absence of VIN during the transaction
   - For 'title_status' field the empty fields were replaced with 'missing'
   - As state and region provides similar information and as oridnal mapping of region had far more values than state, region field was dropped
   - Model is a crucial field, but as there were 21K unique categorical values in that field, it would not be feasible to run regression on it using the limited compute I have. Hence, it was dropped with the hope that it is highly correlated with data in the other datapoints like size, type and drive [In future, will analyse and prove this using Thiel's U and Cramer's V

- ### Data Quality after clensing :  ![1](https://github.com/user-attachments/assets/ce03811a-8b98-4f2e-afd9-fc355b9751f1)

## Modeling

- Multiple regression models were evaluated to arrive at the final Lasso model that used minMax scaler and a poyonmial transformation of the 3rd degree for numeric features
- Some important numbers
   - ### Baseline using LinearRegression
   - Using polynomial of degree 1 transformation of year and odomter values as well as one hot encoding of 'manufacturer', 'fuel', 'transmission', 'paint_color', 'region', 'type', 'status_title' and 'VIN' fields and Ordinal Encodng of other categorical features resulted in
     - Baseline Train: RMSE = 9217.1418, R2 = 0.3347
     - Baseline Test: RMSE = 9235.2744, R2 = 0.3226
  
   - ### Baseline on various models with default settings
     - Linear Regression: RMSE = 9235.2744, R2 = 0.3226 (using standard scaler)
     - Ridge Regression: RMSE = 19262.9098, R2 = -2.1207 (alpha = 1 and using standard scaler)
     - Lasso Regression: RMSE = 10152.9000, R2 = 0.0055  (max_iter = 15000 and MinMaxScaler to avoid convergence error)
 
   - ### GridSearch to find the best hyper parameter value across various models
     - Best Ridge Regression: {'model__alpha': 0.01, 'transform__poly__degree': 3}, RMSE = 19127.3551
     - <mark>Best Lasso Regression: {'model__alpha': 0.1, 'transform__poly__degree': 3}, RMSE = 9099.6546<mark>

     - Results using the Test dataset (a 70/30 split was used)
        - Test : Ridge Regression: RMSE = 19117.3712, R2 = -0.7302
        - <mark>Test : Lasso Regression: RMSE = 9085.7835, R2 = 0.6092<mark>
  
 - ## Final Model
   - Polynomial degree 3 for year, cylinders and odometer columns
   - One hot encoding for 'manufacturer', 'fuel', 'transmission', 'paint_color', 'state', 'type', 'VIN' and 'title_status' fields
   - Ordinal encoding for 'drive', 'size' and'condition' fields
   - Lasso Regression with alpha = 0.1
   - MinMaxScaling for all numeric fields
   
