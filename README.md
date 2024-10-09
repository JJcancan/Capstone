# Capstone

## Business Understanding
Question: Develop a regression model to accurately predict time of arrival 200 Nautical Miles away from the target airport location LAX. More accurate estimated time of arrivals can help with airport traffic and reduce delays.

## Data Understanding

Two months of aircraft position and speed data surrounding LAX was compiled. The time to land was calculated from actual landing information of each aircraft approaching LAX. Data at points of entry at or around the 200NM zone around LAX were used for this projects dataset.

Originally there were 19 features with two categorical and 17 numeric.
['hrstart', 'distapt', 'flightLevel', 'emitterCat', 'airGroundVector_groundSpeed', 'h3_id', 'temp', 'dwpt', 'rhum', 'wdir', 'wspd', 'pres', 'coco']

- logTime: Full timestamp of each row of data
- startlat: Latitude at entry of 200NM zone
- startlon: Longitude at entry of 200NM zone
- hrstart: Hour of the timestamp generally traffic is time dependent and the time of day is most important
- distapt: Distance to airport
- flightLevel: The altitude in 100Ft 
- emitterCat: emitter category Id that describes the aircraft's size, weight, or performance characteristics
- airGroundVector_groundSpeed: Ground Speed in nautical miles per second
- h3_id: The H3 tile id 
- temp: Temperature
- dwpt: Dew Point
- rhum: Relative Humidity
- wdir: Wind Direction
- wspd: Wind Speed
- pres: Atmospheric Pressure 
- coco: Extent of the sky covered by clouds
- prcp: precipitation
- AC Type: Aircraft Model Type
- AC Role: Aircraft Role (Commercial, Cargo, Etc)
- ttland: target variable is the actual time to land (not a feature)
- calculated_time: calculated time to land this is a common back of the envelope calculation that uses ground speed and distance to airport (not a feature)

Basic look at the data prcp is all zeroes and was dropped. Variables emitter category, aircraft type, and aircraft role that describe the aircraft characteristics were highly correlated. 

Aircraft type had a high amount of variation and other aircraft characteristics could be boiled down to the feature called emitter category (EmitterCat). The EmitterCat provides an emitter category Id that describes the aircraft's size, weight, or performance characteristics.

Datetime in the data that corresponds to each lat and long was rounded to the nearest hour. This was done in order to correlate Meteostat weather data separated by hour, which gets its data from the LAX weather station.

A calculated time was included as the estimation of time of arrival typically is done using current ground speed and distance to landing.

H3 library was used to convert lat and long coordinates to categories similar to using a northwest, northeast, type directional feature. This estimates the direction of approach. A resolution is set to indicate the size/amount of hexagonal cells. 

The figure below shows the lat and longitude before h3 tiling
![starting_lat_lon](https://github.com/user-attachments/assets/3ae30f1d-f989-4e83-a5ff-e77896808ab2)

The figure below shows lat and long converted/grouped to h3 tiles
![LAXH3tiles](https://github.com/user-attachments/assets/2c595807-d5a0-4a22-aa91-c1a729283219)


## Data Preparation
Because emitter category is highly correlated with other aircraft characteristic features, the emitter category is used in place of those other features.

The datetime, starting latitude, starting longitudes were dropped as these features were only used to fetch weather data and convert positions to h3 tiles.

Percipitation (prcp) was dropped as all values were 0. Missing values were then dropped from the dataset. 

The calculated time and the target variable time to land (ttland) were removed from the dataset.

A test train split was done splitting the data 20/80.

Our dataset for modelling has 13 features with 20784 rows of data.
- hrstart
- distapt
- flightLevel
- emitterCat
- airGroundVector_groundSpeed
- h3_id
- temp
- dwpt
- rhum
- wdir
- wspd
- pres
- coco

## Modeling
A base linear regression model was used to get a baseline performance for MSE.

- Baseline Mean Squared Error: 4.740309637874746
- The hard calculated time to land using ground speed and distance to airport had a large MSE: 229.29945981265553 

From this point, four other models were used with default hyperparameters to evaluate the models. Ridge, Support Vector, Gradient Boosting, and XGBoost regression models were used.

Ridge Regression is great for linear relationships and regularization.

SVR is useful for non-linear relationships, especially in high-dimensional spaces.

Gradient Boosting is effective for capturing complex patterns and interactions.

XGBoost is an advanced, highly efficient version of gradient boosting, ideal for large datasets and high performance.

The first run will be done without hyperparameter tuning to see which models would show promise. This was done to eliminate unnecessary runtime for models that we know did not perform well. The second run will do hyperparameter tuning using gridsearchcv  with a focus on negative mean squared error (because we wanted lower MSE results).

## Evaluation

The results are as follows:

| Model                           | Training MSE | Test MSE | R2       | Runtime (s) |
|---------------------------------|---------------|----------|----------|--------------|
| Linear Regression               | 5.247364      | 4.740310 | 0.792723 | 0.187235     |
| Ridge Regression                | 5.247551      | 4.739043 | 0.792779 | 0.049623     |
| Support Vector Regression        | 4.811447      | 4.397559 | 0.807711 | 101.546587   |
| Gradient Boosting               | 4.249344      | 4.088642 | 0.821218 | 6.199033     |
| XGBoost                        | 4.335094      | 4.108316 | 0.820358 | 0.288604     |

SVR runtime was fairly large and was not the best performing model. Gradient and XGBoost models were shown to be promising starts and therefore chosen to undergo hyperparameter tuning via gridsearchcv using the parameter grid below. Ridge regression was also evaluated as run-time for this model was fairly fast.

Ridge, Gradient Boosting, and XGBoost regression models were evaluated 

models = {
    'Ridge Regression': (Ridge(), {
        'alpha': [0.1, 1.0, 10.0],
        'max_iter': [100, 200, 300]
    }),
    'Gradient Boosting': (GradientBoostingRegressor(), {
        'n_estimators': [50, 100, 150],
        'learning_rate': [0.01, 0.1, 0.2],
        'max_depth': [3, 5]
    }),
    'XGBoost': (XGBRegressor(), {
        'n_estimators': [50, 100, 200],
        'learning_rate': [0.01, 0.1, 0.3],
        'max_depth': [3, 5],
        'subsample': [0.4, 0.8, 1.0]
    }),
}

The results are shown below. XGBoost with the parameters shown were the best performing according to the MSE and R2 scores. Ridge regression performed similar to default and gradient boosting regression performed fairly well but not as well as XGBoost and had a large run time.

| Model              | Best Parameters                                                              | Training MSE | Test MSE | R2       | Runtime (s) |
|--------------------|-----------------------------------------------------------------------------|--------------|----------|----------|--------------|
| Ridge Regression    | `{'alpha': 1.0, 'max_iter': 100}`                                          | 5.247551     | 4.739043 | 0.792779 | 0.623461     |
| Gradient Boosting   | `{'learning_rate': 0.1, 'max_depth': 3, 'n_estimators': 150}`             | 4.019953     | 4.011836 | 0.824577 | 555.955555   |
| XGBoost            | `{'learning_rate': 0.1, 'max_depth': 5, 'n_estimators': 200, 'subsample': 0.8}` | 2.758963     | 3.852231 | 0.831556 | 104.953677   |

Below shows each models best tuned test MSE and the baseline (no tuning) MSE against the linear regression baseline MSE.
![mse_comparison_grouped_final](https://github.com/user-attachments/assets/354c3849-87f1-42ed-a825-e62d51730c4a)

The final model result is shown below against the other baseline MSE values. 
| Hard Calc MSE | Linear Regression Baseline MSE | Final Tuned XGBoost MSE |
|---------------|-------------------------------|--------------------------|
| 229.29946     | 4.74031                       | 3.852231                 |

Other considerations for this project;
- Further zone of interest
- More accurate weather data
- More data in general
- Getting flight plan information (if flight approaching was delayed at takeoff of other location).



