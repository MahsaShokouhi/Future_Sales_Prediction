# Future_Sales_Prediction


## Business Problem

Future sales forecasting is an important part of the financial planning of a company. An accurate sales forecast is essential for business and revenue growth by allowing companies to gauge the demand for their products, plan their supply and manage the inventory accordingly. 


## Objective

The goal of this project is to use historical daily sales for all products of a software company with multiple stores/branches and predict the total sales for every product at each store over the following month.


## Data

The training data is a time-series dataset consisting of 2935849 records of daily sales for 21807 products across 60 stores over a period of 34 months (from January 2013 to October 2015) as summarized in the following table:

stats | month_block |	shop_id |	item_id |	item_price |	item_cnt_day |	item_category_id
----- | ----------- |  ------- |  ------ |  --------- |  ------------ |  ----------------
min |	0.00 |	0.00 |	0.00 |	-1.00 |	-22.00 |	0.00
0.25 |	7.00 |	22.00 |	4476.00 |	249.00 |	1.00 |	28.00
0.50 |	14.00 |	31.00 |	9343.00 |	399.00 |	1.00 |	40.00
0.75 |	23.00 |	47.00 |	15684.00 |	999.00 |	1.00 |	55.00
max |	33.00 |	59.00 |	22169.00 |	307980.00 |	2169.0 |0	83.00

- month_block: index of the month, ranging from 0 to 33.
- shop_id, and item_id: unique identifier for each store and product, respectively.
- item_price: product price (daily)
- item_cnt_day: number of daily sales for each product at each store
- item_category_id: unique identifier for each item category (such as DVD-Movie, Book, Games, software, etc.)


## Data Wrangling

As can be seen from the previous table, there are records with negative price and negative daily sale count (item_cnt_day). A closer look shows 7356 records with negative item_cnt_day, corresponding to returns, and only one record with negative price. 
The record with negative item_price was removed, and the data frame was reformatted to get separate columns for sales (sell_cnt) and returns (return_cnt). Here’s a snapshot of the first and last 3 rows of the data:

date    |	month_block |	shop_id |	item_id |	item_price |	item_category_id |	return_cnt |	sell_cnt
------- |  ---------- | ------- | ------- | ---------- |  ---------------- |  ---------- |  --------
2013-01-02 |	0 |	59 |	22154 |	999.0 |	37 |	0.0 |	1.0
2013-01-03 |	0 |	25 |	2552 |	899.0 |	58 |	0.0 |	1.0
2013-01-05 |	0 |	25 |	2552 |	899.0 |	58 |	1.0 |	0.0
2015-10-14 |	33 |	25 |	7459 |	349.0 |	55 |	0.0 |	1.0
2015-10-22 |	33 |	25 |	7440 |	299.0 |	57 |	0.0 |	1.0
2015-10-03 |	33 |	25 |	7460 |	299.0 |	55 |	0.0 |	1.0


## Exploratory Data Analysis

Since the goal is to predict monthly sales, the daily records were aggregated to obtain total monthly sales of each product at each store: total_monthly_sale (target variable).

To assess autocorrelation of the target variable, 12 lags (months) were used and the correlation between total sales of each product sold at each store in a given month and the following 12 months were estimated.  The following figure shows that this correlation decreases over longer periods, with strongest correlation seen with 1-month lag.

![Figure1](/images/fig1.png)


## Feature Engineering

The following parameteres were extracted and added to features:
1. month, season, and year for each sales record.
2. monthly store sales (monthly_sale_shop: monthly sales of all products at each store), and monthly products sales (monthly_sale_item : total sales of a given product across all stores)
3. 1-6 months lags of total_monthly_sale, monthly_sale_shop, monthly_sale_item
4. Mean-encode 'item_id' and 'shop_id': mean target value for each product, and each shop, respectively. To reduce overfitting, regularizared mean encoding was achieved with 5-fold cross-validation. 
Here are the correlations between mean-encoded features and the target variable: 
    * Correlation between mean_encoded shop_id and target = 0.01
    * Correlation between mean_encoded item_id and target = 0.48

The following figure shows correlations between all features and the target variable: total montly sale for each product at each store (monthly_sale_item).

![Figure2](/images/fig2.png)


## Modelling

- Train-test split: Keep the first 33 month for training and the last month for test
- To evaluate and compare models, root mean squared error (rmse) was chosen as the average rmse was computed using 3-fold cross-validation. 
- Baseline model: Predict sales based on previous month (monthly sale for each item, at each shop = total sale for the previous month). The root mean squared error of prediction on the test set with baseline model is: 14.50

### Linear Model
For linear model, feaures were scaled using standard scaling and principal component analysis was used to obtain uncorrelated predictors. Here are the rmse values during cross-validation and test:
The average rmse with 3-fold cross-validation: 5.86
The rmse of prediction on the test set: 11.96

### Light Gradient Boosting Algorithm
The model parameters were tuned duing cross-validation, which shows improved rmse for both cross-validation and test set:
The average rmse with 3-fold cross-validation: 5.46
The rmse of prediction on the test set: 11.17


## Feature Importance

![Figure3](/images/fig3.png)


## Conclusions and Considerations

- The model shows that the 4 most important predictors of future sales of a product at a store, in the order of importance are:
   1. Store's monthly sales 
   2. Product's monthly sales
   3. Previous month's sales of the product at the store
   4. The store
- A few more approaches were tested as summarized below, but did not improve the model performance:
   - Extending item categoies by text-encoding the item category names
   - Ensembling and combining linear and gradient boosting models, using either weighted averaging or stacking the two models.  
