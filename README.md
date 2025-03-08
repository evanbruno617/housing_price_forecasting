# House Sale Price Forecasting

## Purpose
The purpose of this project is to create a predictive model to estimate house sale prices. This model is useful for buyers, sellers, and investors as it helps determine the fair value of a property. It assists stakeholders in strategizing their market approach, whether for making offers, setting sale prices, or assessing investment opportunities.

---

## Data Cleaning / Extraction

### Source
The primary dataset containing house prices was extracted from the DOLT database, which included sales data for individual houses in LA County. The dataset contained numerous null and irrelevant columns, necessitating extensive data cleaning. Due to time constraints, only a few key variables were selected, including:
- **Sale price**  
- **Building year**  
- **Property type**  
- **Geographic features**  
- **Economic data** (unemployment rates and income)  

Unemployment data was aggregated monthly by city, while income data was provided yearly by zip code.

### Cleaning
Various data [cleaning processes](https://github.com/evanbruno617/housing_price_forecasting/blob/main/cleaning_combination.ipynb) were used, but the most effective one involved a reverse lookup. The dataset contained inconsistent city name variations (e.g., *"MONTEREY PARK"*, *"MONTEREY PRK"*, and *"MONTEREY PK"*), making standardization difficult.  

To address this, a workaround was implemented using a key feature from the income dataset. Each row included:
- **City name**
- **Zip code**
- **Latitude & longitude**  

By merging the main dataset with the income dataset using zip codes and applying an API lookup on latitude and longitude, a standardized city name column was generated. This enabled successful dataset merging with unemployment data.  
*(Insert API image here)*  

---

## Data Preprocessing

Since house sales are highly correlated with previous values, lags were incorporated to improve model performance. However, because multiple sales can occur on the same day, traditional lagging methods were ineffective. Instead, data was grouped by **zip code** and aggregated on a monthly basis.

For each row, the model included three lagged values:  
- **Mean of last month**  
- **Mean of the month before that**  
- **Mean of three months ago**  

### Spatial Correlation Handling
House sales exhibit **spatial correlation**, meaning nearby zip codes influence each other. To account for this:
- The **three nearest zip codes** for each target zip code were identified  
- **Distances between zip codes** were calculated  
- Lagged values for the nearest zip codes were included in the model  

To control for fixed effects, **dummy variables** were created for each zip code. However, due to time constraints, they were only created for the **primary zip code** to avoid excessive variable expansion.

### Neural Networks Preprocessing
For neural networks, data normalization was necessary. However, normalizing the entire dataset at once could introduce **data leakage**. To prevent this:
- A **rolling normalization** approach was used  
- **Exponentially weighted components** with a span of 20 were applied  

This ensured that normalization was performed only on past data, preserving the integrity of time-series forecasting.  
*(Include code snippets here)*  

---

## Machine Learning

### Random Forest
The initial predictive model was a **Random Forest Regression** with 500 estimators. A **time-based split** was used:
- **90% training data**
- **10% validation data**  

The training dataset consisted of **600,000 rows**, while the test dataset had **60,000 rows**.  
*(Insert prediction graph here)*  

**Model Performance:**
- **Mean Absolute Error (MAE):** $310,000  
- **Issue:** The model performed well for most houses but struggled with higher-priced homes  
- **Potential Cause:** Exclusion of **square footage**, which is critical for distinguishing high-value properties  

Due to API limitations, square footage data could not be included.

---

## Model Optimization

### Feature Selection
A **backward selection** process was used to minimize MAE. Instead of using the default feature importance from the random forest, **permutation importance** was applied.

Data splitting was done as follows:
- **70% training set**
- **20% validation set**
- **10% holdout set**  

Instead of selecting the feature set with the absolute lowest MAE, the **first performance minima** was chosen to improve generalizability.

### Neural Networks
A basic **sequential neural network** was tested with **early stopping** and minimal layers/neurons. However, due to limited complexity, it failed to outperform the random forest model.

### Gradient Boosting
To optimize performance, **grid search** was conducted on:
- **All features**
- **Selected features**  

**Results:**
| Model | MAE | RMSE |
|--------|------|------|
| All features | $303,000 | $670,000 |
| Selected features | $304,000 | $663,000 |

Despite having a slightly higher MAE, the **selected features model** was preferred due to its lower RMSE, which reduced large prediction errors.  
*(Insert comparison graph here)*  

### Model Comparison
- **For mid-priced homes:** Gradient Boosting was preferred due to better average performance (lower MAE).  
- **For high-value homes:** Random Forest was preferable due to its lower RMSE, reducing the impact of extreme errors.  

---

## Model Selection & Future Research

### Final Model
The **Gradient Boosting model** was chosen as the final model:  
- **MAE:** $304,000  
- **RMSE:** $663,000  

While the **Random Forest model** had a **lower RMSE**, it had a slightly **higher MAE** ($310,000). The choice of model depends on the specific use case:
- **Gradient Boosting for mid-range properties**
- **Random Forest for high-end properties**  

### Future Research
Several improvements could enhance model performance:
- **Adding more market factors** (e.g., interest rates, crime rates)  
- **Increasing Neural Network complexity** to explore deeper learning architectures  
- **Incorporating square footage**, a key determinant of home value  
- **Using latitude/longitude instead of zip codes** for better spatial relationships  

Currently, zip codes define spatial relationships, but using latitude/longitude would allow for a **dynamic radius**, capturing more precise influences from nearby sales.  

These enhancements would lead to more reliable price predictions across different market conditions.
