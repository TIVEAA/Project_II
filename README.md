# Time series prediction

Description: 
- Revenue forecast using: simple average, arima, arimax and sarimax models
- PCA in order to reduce the number of store IDS
- Hierarchical clustering using the selected PC

### Part I: Revenue forecast for the 5 upcoming weeks 

**Datasets**

All the datasets used can be found below:

-
-
-

The CRISP DM methodology has been used throughout the report:

- Business/Data Understanding
- Data Preparation
- Modeling
- Evaluation
- Forecast

Each step is briefly described as follows:

**1) Business/Data understanding**

As mentioned above, there are 3 datasets with a csv extension:

- Sales (14 columns in total):
  1. Unnamed column (PK);
  2. Store ID: ID of each store (FK);
  3. Product ID: ID of each product (FK);
  4. Date: date of each sale (yyyy-mm-dd format)
  5. Sales: number of sales
  6. Revenue: revenue of each sale
  7. Stock
  8. Price
  9. Promo columns: promo_type_1, promo_bin_1, promo_type_2, promo_bin_2, promo_discount_2, promo_discount_type_2

- Cities (6 columns in total):
  1. Store ID (PK);
  2. Store type ID
  3. Store size
  4. City old ID
  5. Country ID
  6. City code 

- Product (10 columns in total):
  1. Product ID (PK);
  2. Product length;
  3. Product depth;
  4. Prodcut width;
  5. Cluster ID
  6. Hierarchy: hierarchy1_id, hierarchy2_id, hierarchy3_id, hierarchy4_id and hierarchy5_id

After analysing all datasets, it can be deducted:
- There are **63** different stores located in Turkey;
  ![Stores with their sales](Documentação/Number_stores.jpg)
  
- The sales dataset includes **daily** sales/revenue per store from January 2017 to September 2019;
  ![](Documentação/Daily_revenue.png)
  
- Total products and total revenue vary between stores:
  
  ![](Documentação/Total_products.png)
  ![](Documentação/Total_revenue.png)

**2) Data Preparation**

- In order to reduce the number of rows (8886058), sales dataset has been grouped to weekly revenue (8610 rows).  

  ![](Documentação/Weekly_revenue.png)

- Seasonal Decomposition

  A time series can exhibit a variety of patterns. It is helpful to split the time series into several components, each representing a pattern category. 
  Below is shown the seasonal composition for Store ID S0003:

  ![](Documentação/Seasonal_decomposition.png)
  
  There is a steady upward trend over the years and a clear seasonal pattern.

- Train vs. Test

  Before we start modelling, we need to split our dataset into training and test:

  - **Training**: 140 weeks
  - **Test**: 4 weeks
  
  ![](Documentação/Train_test_v2.PNG)

  **3) Modeling**

  Four different models have been used to forecast revenue for the next 5 weeks:

    1. Simple average
    
       All future values are equal to the average of the historical data. 

       Simple average vs test set:
       ![](Documentação/Average.PNG)

    2. Arima (auto-arima)

       An ARIMA model is characterized by 3 terms: p, d, q:

       - p is the order of the AR term
       - q is the order of the MA term
       - d is the number of differencing required to make the time series stationary
      
       Auto arima uses a stepwise approach to search multiple combinations of p,d,q parameters and chooses the best model that has the least AIC.

       ![](Documentação/Arima.PNG)

       Parameters considered:
         - y: the time-series to which to fit the ARIMA estimator (Revenue column)
         - seasonal: False (not considering seasonality)
         - suppress_warnings: True
         - random_state: Equals to 20 (Ensures replicable testing and results)
         - stepwise: default = True (Whether to use the stepwise algorithm outlined in Hyndman and Khandakar (2008) to identify the optimal model parameters)
         - method: default = 'lbfgs' (The method determines which solver from scipy.optimize is used - ‘lbfgs’ for limited-memory BFGS with optional box constraints)
         - start_p: defalut = 2 (order (or number of time lags) of the auto-regressive (“AR”) model)
         - start_q: default = 2 (order of the moving-average (“MA”) model)
         - d: default=None (If None, the value will automatically be selected based on the results of the test (i.e., either the Kwiatkowski–Phillips–Schmidt–Shin, Augmented Dickey-Fuller or the 
           Phillips–Perron test will be conducted to find the most probable value)
         - max_p: default = 5 (The maximum value of p, inclusive)
         - max_d: default = 2 (The maximum value of d, or the maximum number of non-seasonal differences)
         - max_q: default = 5 (The maximum value of q, inclusive)
         - test: default = 'kpss' (Type of unit root test to use in order to detect stationarity if stationary is False and d is None: Default is ‘kpss’ (Kwiatkowski–Phillips–Schmidt–Shin)

       Note: All the best parameters of auto-arima are stored in a dictionary:
          ![](Documentação/Model_arima.PNG)

       Arima vs test set:
       ![](Documentação/Arima_S0038.PNG)

    3. Arimax
 
       ARIMAX is an extension of the traditional ARIMA model that allows for the inclusion of additional variables, known as exogenous variables, which may have an effect on the time series being forecaste.

       ![](Documentação/Arimax_exogenous_variables.PNG)
       
       The exogenous variables considered in this study are:

         - Promo_discount_2: boolean variable, where value 1 means there was at least one promo in that week;
         - Is_holiday: boolean variable, where value 1 means there was at least one national holiday in that week;
         - Month: number of the month

       Comparison between ARIMA and ARIMAX:

       ![](Documentação/Arima_vs_Arimax_2.PNG)

       Arimax vs test set:

       ![](Documentação/Arimax_S0003.PNG)

     4. Sarimax
 
        Introducing seasonality into arimax.

        ![](Documentação/Sarimax_variables.PNG)
        
        Parameters considered:
          - m: The period for seasonal differencing, m refers to the number of periods in each season (m = 52 weeks)
          - D: The order of the seasonal differencing (D = 1)

        Sarimax vs test set:

        ![](Documentação/Sarimax_S0143.PNG)

  **4) Evaluation**

    Let's evaluate the results for 4 stores: S0003, S0023, S0038 and S0143.
  
    RMSE comparison can be seen below:
  
    ![](Documentação/RMSE_comparison.PNG) 

    After analyzing all RMSE values, the number of stores vs best model can be plotted:

    ![](Documentação/Stores_models.PNG) 

    Finally, the forecast revenue for the next 5 weeks is shown below:

    ![](Documentação/Forecast_S0003.PNG)
    ![](Documentação/Forecast_S0023.PNG)
    ![](Documentação/Forecast_S0038.PNG)
    ![](Documentação/Forecast_S0143.PNG)
    
### Part II: PCA and Hierarchical clustering

  The idea of this study is to perform a dimensionality reduction by using PCA. Why? 
  - Reduces the number of stores in future analyses;
    - Identify stores that capture key patterns of variation
  - Reduces processing time
  - Reduces number of processing machines
  - Facilitates data visualisation and identification of trends or patterns 

  Procedure to perform the anlysis:
    1. Prepare dataset
      - Use dataset with daily sales (Could have used weekly sales or total sales instead) 
          ![](Documentação/Daily_revenue_PCA.PNG) 
      - Transpose dataset in order to have stores ID as features 
          ![](Documentação/Transposed_dataset_PCA.PNG) 
      - Data standardization
          ![](Documentação/Standardization_PCA.PNG) 

    
  
  ![](Documentação/PC.PNG)
  

       
       
  

  
  
  
  
