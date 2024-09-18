# README: Data Merging and Cleaning

## Overview

This script performs data merging and cleaning on datasets from various sources to create a unified dataset with product names and their respective prices. The data is sourced from a CSV file, an Excel file, and a JSON file. The final output is saved as a CSV file containing consolidated product information.

## Dependencies

The script requires the following Python libraries:
- `pandas`: For data manipulation and analysis.
- `json`: For working with JSON data (though it's used internally by pandas).

## Datasets

1. **Amazon Dataset (CSV)**
   - File path: `./Data sets/amazon.csv`
   - Columns: `product_id`, `product_name`, `category`, `discounted_price`, `actual_price`, `discount_percentage`, `rating`, `rating_count`, `about_product`, `user_id`, `user_name`, `review_id`, `review_title`, `review_content`, `img_link`, `product_link`

2. **Sales Dataset (Excel)**
   - File path: `./Data sets/Sales.xlsx`
   - Sheet: `Orders`
   - Columns: `Row ID`, `Order ID`, `Order Date`, `Ship Date`, `Delivery Duration`, `Ship Mode`, `Customer ID`, `Customer Name`, `Segment`, `Country`, `City`, `State`, `Postal Code`, `Region`, `Product ID`, `Category`, `Sub-Category`, `Product Name`, `Sales`, `Quantity`, `Discount`, `Discount Value`, `Profit`, `COGS`

3. **Steam Games Dataset (JSON)**
   - File path: `./Data sets/steamgames.json`
   - Columns: `name`, `discount`, `orig_price`, `disc_price`, `tags`, `reviews`, `link`

## Code Explanation

### 1. Data Loading

```python
import pandas as pd
import json
```

# Load datasets
```python
csv_dataset = pd.read_csv("./Data sets/amazon.csv")
excel_dataset = pd.read_excel('./Data sets/Sales.xlsx', sheet_name='Orders')
json_dataset = pd.read_json("./Data sets/steamgames.json")
```
# Print dataset columns
```python
print("CSV Dataset Columns:", csv_dataset.columns)
print("Excel Dataset Columns:", excel_dataset.columns)
print("JSON Dataset Columns:", json_dataset.columns)
```
### 2. Data Preparation

```python
# Rename columns for consistency
json_dataset.rename(columns={'name': 'product_name', 'orig_price': 'actual_price', 'disc_price': 'discounted_price'}, inplace=True)
excel_dataset.rename(columns={'Product Name': 'product_name', 'Sales': 'actual_price'}, inplace=True)
```

### 3. Selecting Relevant Columns
```python
csv_df = csv_dataset[['product_name', 'actual_price']]
excel_df = excel_dataset[['product_name', 'actual_price']]
json_df = json_dataset[['product_name', 'actual_price', 'discounted_price']]
```

### 4. Handling Missing Values
```python
# Fill missing actual_price in JSON with discounted_price if actual_price is NaN
json_df['actual_price'] = json_df['actual_price'].fillna(json_df['discounted_price'])

# Drop the now redundant 'discounted_price' column
json_df = json_df.drop(columns=['discounted_price'])
```

### 5. Merging Dataframes
```python
# Merge dataframes
merged_df = pd.merge(csv_df, excel_df, on='product_name', how='outer')
merged_df = pd.merge(merged_df, json_df, on='product_name', how='outer')

# Combine actual_price columns from different sources
merged_df['actual_price'] = merged_df[['actual_price_x', 'actual_price_y', 'actual_price']].bfill(axis=1).iloc[:, 0]
```


### 6. Finalizing the Dataframe
```python
# Keep only relevant columns
merged_df = merged_df[['product_name', 'actual_price']]

# Drop rows with missing actual_price
merged_df.dropna(subset=['actual_price'], inplace=True)
```
### 7. Saving the Combined Dataset
```python
# Save the combined dataset to a CSV file
merged_df.to_csv('combined_dataset.csv', index=False)

# Display the first few rows of the merged dataframe
print(merged_df.head())
```
