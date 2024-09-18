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
```output
CSV Dataset Columns: Index(['product_id', 'product_name', 'category', 'discounted_price',
       'actual_price', 'discount_percentage', 'rating', 'rating_count',
       'about_product', 'user_id', 'user_name', 'review_id', 'review_title',
       'review_content', 'img_link', 'product_link'],
      dtype='object')
Excel Dataset Columns: Index(['Row ID', 'Order ID', 'Order Date', 'Ship Date', 'Delivery Duration',
       'Ship Mode', 'Customer ID', 'Customer Name', 'Segment', 'Country',
       'City', 'State', 'Postal Code', 'Region', 'Product ID', 'Category',
       'Sub-Category', 'Product Name', 'Sales', 'Quantity', 'Discount',
       'Discount Value', 'Profit', 'COGS'],
      dtype='object')
JSON Dataset Columns: Index(['name', 'discount', 'orig_price', 'disc_price', 'tags', 'reviews',
       'link'],
      dtype='object')
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
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>product_name</th>
      <th>actual_price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>!!1000 Watt/2000-Watt Room Heater!! Fan Heater...</td>
      <td>₹1,599</td>
    </tr>
    <tr>
      <th>1</th>
      <td>!!HANEUL!!1000 Watt/2000-Watt Room Heater!! Fa...</td>
      <td>₹1,599</td>
    </tr>
    <tr>
      <th>2</th>
      <td>"While you Were Out" Message Book, One Form pe...</td>
      <td>7.42</td>
    </tr>
    <tr>
      <th>3</th>
      <td>"While you Were Out" Message Book, One Form pe...</td>
      <td>8.904</td>
    </tr>
    <tr>
      <th>4</th>
      <td>"While you Were Out" Message Book, One Form pe...</td>
      <td>8.904</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>14182</th>
      <td>骸心 Mainbody</td>
      <td>99.0</td>
    </tr>
    <tr>
      <th>14183</th>
      <td>눈 떠보니 임진왜란이었다 - Back To the Joseon</td>
      <td>239.0</td>
    </tr>
    <tr>
      <th>14184</th>
      <td>미연시 시뮬레이터 : 미소녀 게임의 주인공을 조종하는 게임</td>
      <td>299.0</td>
    </tr>
    <tr>
      <th>14185</th>
      <td>피랍 일지 - 그 남자로부터의 탈출</td>
      <td>329.0</td>
    </tr>
    <tr>
      <th>14186</th>
      <td>🧠 OUT OF THE BOX</td>
      <td>1499.0</td>
    </tr>
  </tbody>
</table>
<p>14187 rows × 2 columns</p>
</div>

