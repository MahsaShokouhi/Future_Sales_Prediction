# Future_Sales_Prediction
[Link to notebook](https://github.com/MahsaShokouhi/Future_Sales_Prediction/blob/master/Future_Sales_Prediction.ipynb)

<br>

## Business Problem and Project Goal

Future sales forecasting is an important part of the financial planning of a company. An accurate sales forecast is essential for business and revenue growth by allowing companies to gauge the demand for their products, plan their supply and manage the inventory accordingly. 

The goal of this project is to use historical daily sales for all products of a software company with multiple stores/branches and predict the total sales for every product at each store in the following month.

The main complexity of this project was the multi-faceted nature of the dataset: a wide range of products (Books, Movies, Game Consoles, Software, etc), with different popularity and seasonality patterns, sold at different stores. Typically some stores are more crowded than others, depending on location, staff, customer service, etc.   

<br>

## Data

The training data was a time-series dataset consisting of 2935849 records of daily sales for 21807 products across 60 stores over a period of 34 months (from January 2013 to October 2015):
sales_train.csv : daily sales records
items.csv : items specifications including "item name", "item_id", and "item_category_id"
item_categories.csv: items categories information including "item category name" and "item category id"

Here's a snapshot of the dataset after renaming columns, removing product name, and merging "sales" records with items specifications:

stats | month_index |	shop_id |	item_id |	item_price |	item_cnt_day |	item_category_id
----- | ----------- |  ------- |  ------ |  --------- |  ------------ |  ----------------
min |	0.00 |	0.00 |	0.00 |	-1.00 |	-22.00 |	0.00
0.25 |	7.00 |	22.00 |	4476.00 |	249.00 |	1.00 |	28.00
0.50 |	14.00 |	31.00 |	9343.00 |	399.00 |	1.00 |	40.00
0.75 |	23.00 |	47.00 |	15684.00 |	999.00 |	1.00 |	55.00
max |	33.00 |	59.00 |	22169.00 |	307980.00 |	2169.0 |0	83.00

- month_index: index of the month, ranging from 0 to 33.
- shop_id, and item_id: unique identifier for each store and product, respectively.
- item_price: daily price of each product
- item_cnt_day: number of daily sales for each product at each store
- item_category_id: unique identifier for each item category (such as DVD-Movie, Book, Games, software, etc.)

<br>

## Data Wrangling

As can be seen from the previous table, there are records with negative price and negative daily sale count (item_cnt_day). A closer look shows 7356 records with negative item_cnt_day, corresponding to returns, and only one record with negative price. 
The record with negative item_price was removed (due to ambiguity), and the data frame was reformatted to get separate columns for sales (sell_cnt) and returns (return_cnt). Here’s a snapshot of the first and last 3 rows of the data:

date    |	month_index |	shop_id |	item_id |	item_price |	item_category_id |	return_cnt |	sell_cnt
------- |  ---------- | ------- | ------- | ---------- |  ---------------- |  ---------- |  --------
2013-01-02 |	0 |	59 |	22154 |	999.0 |	37 |	0.0 |	1.0
2013-01-03 |	0 |	25 |	2552 |	899.0 |	58 |	0.0 |	1.0
2013-01-05 |	0 |	25 |	2552 |	899.0 |	58 |	1.0 |	0.0
2015-10-14 |	33 |	25 |	7459 |	349.0 |	55 |	0.0 |	1.0
2015-10-22 |	33 |	25 |	7440 |	299.0 |	57 |	0.0 |	1.0
2015-10-03 |	33 |	25 |	7460 |	299.0 |	55 |	0.0 |	1.0

<br>

## Exploratory Data Analysis

Since the goal is to predict monthly sales, the daily records were aggregated to obtain monthly sales of each product at each store: sales_item_shop (target variable).

The following figure shows plots of 3 different products at 3 different stores, which shows a different sales pattern for each case in terms of sales frequency and seasonality.

![Figure1](/images/fig1.png)

The autocorrelation of the monthy sales was estimated uing 12 lags (months). The following figure shows plots of correlation of each month's sales with previous months' sales(up to 12 previous month). As can be seen, montly sales has strongest correlaiton with previous month's sales and the correaltion decreases over longer period of time.

![Figure2](/images/fig2.png)

<br>

## Feature Engineering

The following parameteres were extracted and added to features:
1. month, season, and year for each sales record.
2. monthly store sales (sales_shop: monthly sales of all products at each store), and monthly products sales (sales_item : total sales of a given product across all stores)
3. 1-12 months lags of sales_item_shop (target variable), sales_shop, sales_item
4. Mean-encode 'item_id' and 'shop_id': mean target value for each product, and each shop, respectively. To reduce overfitting, regularizared mean encoding was achieved with 5-fold cross-validation. 
Here are the correlations between mean-encoded features and the target variable: 
    * Correlation between mean_encoded shop_id and target = 0.01
    * Correlation between mean_encoded item_id and target = 0.48

This led to a total number of 48 features. The following figure shows correlations between all features and the target variable: total montly sale for each product at each store (sales_item_shop). Most notably, the target variable shows highest correlation with the item-specific features such as item's monthly sales (across all stores), mean-encoded item, in the same month as well as previous months.

![Figure3](/images/fig3.png)

<br>

## Modelling

- Becasue of the complex and sparse data, the commonly used used methods for timeseries data (SARIMA model, for example) is not sutiable. Instead, the lagged data (from up to 12 previous months) were included in the regression models (with both linear and non-linear regression). In addition, date-related features were added to the model to capture potential, item-specific, sale seasonality as mentioned in the previous section. To evaluate and compare models, root mean squared error (rmse) was chosen as the average rmse was computed using 3-fold cross-validation.
- Train-test split: Keep the first 33 month for training and the last month for test 
- Baseline model (for comparison): Predict sales based on previous month: monthly sale (for each item, at each shop) = sale for the previous month. The root mean squared error of prediction on the test set with baseline model is: 14.50

### Linear Model
For linear model, feaures were scaled using standard scaling and principal component analysis was used to obtain uncorrelated predictors. Here are the rmse values during cross-validation and test:

The average rmse with 3-fold cross-validation: 5.80

The rmse of prediction on the test set: 12.02

### Light Gradient Boosting Algorithm
The model parameters were tuned duing cross-validation, which shows improved rmse for both cross-validation and test set:

The average rmse with 3-fold cross-validation: 5.29

The rmse of prediction on the test set: 10.21

<br>

## Feature Importance

![Figure4](/images/fig4.png)

<br>

## Conclusions and Considerations

- The model shows that the 4 most important predictors of future sales of a product at a store, in the order of importance are:
   1. Store's monthly sales 
   2. Product's monthly sales
   3. Previous month's sales of the product at the store
   4. The store
- A few more approaches were also examined as summarized below, but did not improve the model performance:
   - Extending item categoies by text-encoding the item category names
   - Ensembling and combining linear and gradient boosting models, using either weighted averaging or stacking the two models.  
