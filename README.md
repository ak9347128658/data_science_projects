# Data Science Project: E-Commerce Sales Analysis

## Objective
This project will help you apply your Python and SQL skills to analyze an e-commerce dataset, derive insights, and create visualizations. You’ll use Python for data cleaning, analysis, and visualization, and SQL for querying data from a database. The code provided is robust, with error handling and clear comments to guide you.

## Dataset
- **Source**: Use the "Online Retail Dataset" from Kaggle (https://www.kaggle.com/datasets/carrie1/ecommercedata). It contains fields like `InvoiceNo`, `StockCode`, `Description`, `Quantity`, `InvoiceDate`, `UnitPrice`, `CustomerID`, and `Country`.
- **Format**: CSV file, which you’ll load into Python (Pandas) and SQLite for SQL queries.
- **Note**: Ensure you download the dataset and place it in your project folder as `online_retail.csv`.

## Tools and Setup
- **Python**: Use Jupyter Notebook with libraries: Pandas, Matplotlib, Seaborn, NumPy, sqlite3.
- **SQL**: Use SQLite (built into Python) for database queries.
- **Installation**:
  ```bash
  pip install pandas matplotlib seaborn numpy
  ```
- **Environment**: Jupyter Notebook or Google Colab.

## Project Steps and Code

### 1. Setup and Data Loading
You’ll load the dataset into a Pandas DataFrame and an SQLite database.

```python
import pandas as pd
import sqlite3
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
from datetime import datetime
import warnings
warnings.filterwarnings('ignore')  # Suppress warnings for cleaner output

# Load dataset
try:
    df = pd.read_csv('online_retail.csv', encoding='ISO-8859-1')
    print("Dataset loaded successfully.")
except FileNotFoundError:
    print("Error: 'online_retail.csv' not found. Please download the dataset.")
    exit()
except Exception as e:
    print(f"Error loading dataset: {e}")
    exit()

# Create SQLite database
try:
    conn = sqlite3.connect('ecommerce.db')
    df.to_sql('sales', conn, if_exists='replace', index=False)
    print("Database created and data loaded into SQLite.")
except Exception as e:
    print(f"Error creating database: {e}")
    exit()
```

### 2. Data Cleaning (Python)
Clean the dataset to handle missing values, invalid entries, and create a new column for total price.

```python
# Handle missing values
df.dropna(subset=['CustomerID', 'Description'], inplace=True)  # Drop rows with missing CustomerID or Description
df = df[df['Quantity'] > 0]  # Remove negative quantities
df = df[df['UnitPrice'] > 0]  # Remove non-positive prices

# Convert InvoiceDate to datetime
try:
    df['InvoiceDate'] = pd.to_datetime(df['InvoiceDate'])
except Exception as e:
    print(f"Error converting InvoiceDate: {e}")
    exit()

# Create TotalPrice column
df['TotalPrice'] = df['Quantity'] * df['UnitPrice']

# Remove duplicates
df.drop_duplicates(inplace=True)

# Verify cleaning
print(f"Cleaned dataset shape: {df.shape}")
print(df.head())
```

### 3. SQL Queries
Write SQL queries to extract insights from the SQLite database.

```sql
-- Query 1: Top 5 products by total sales
SELECT Description, SUM(Quantity * UnitPrice) as TotalSales
FROM sales
WHERE Quantity > 0 AND UnitPrice > 0
GROUP BY Description
ORDER BY TotalSales DESC
LIMIT 5;

-- Query 2: Sales by country
SELECT Country, COUNT(DISTINCT InvoiceNo) as TransactionCount, SUM(Quantity * UnitPrice) as TotalSales
FROM sales
WHERE Quantity > 0 AND UnitPrice > 0
GROUP BY Country
ORDER BY TotalSales DESC
LIMIT 5;

-- Query 3: Monthly sales trend
SELECT strftime('%Y-%m', InvoiceDate) as Month, SUM(Quantity * UnitPrice) as TotalSales
FROM sales
WHERE Quantity > 0 AND UnitPrice > 0
GROUP BY Month
ORDER BY Month;
```

Run these queries in Python:

```python
# Execute SQL queries
def run_query(query, connection):
    try:
        result = pd.read_sql_query(query, connection)
        return result
    except Exception as e:
        print(f"Error executing query: {e}")
        return None

# Query 1: Top products
top_products_query = """
SELECT Description, SUM(Quantity * UnitPrice) as TotalSales
FROM sales
WHERE Quantity > 0 AND UnitPrice > 0
GROUP BY Description
ORDER BY TotalSales DESC
LIMIT 5;
"""
top_products = run_query(top_products_query, conn)
print("Top 5 Products by Sales:")
print(top_products)

# Query 2: Sales by country
country_query = """
SELECT Country, COUNT(DISTINCT InvoiceNo) as TransactionCount, SUM(Quantity * UnitPrice) as TotalSales
FROM sales
WHERE Quantity > 0 AND UnitPrice > 0
GROUP BY Country
ORDER BY TotalSales DESC
LIMIT 5;
"""
country_sales = run_query(country_query, conn)
print("\nSales by Country:")
print(country_sales)

# Query 3: Monthly sales trend
monthly_query = """
SELECT strftime('%Y-%m', InvoiceDate) as Month, SUM(Quantity * UnitPrice) as TotalSales
FROM sales
WHERE Quantity > 0 AND UnitPrice > 0
GROUP BY Month
ORDER BY Month;
"""
monthly_sales = run_query(monthly_query, conn)
print("\nMonthly Sales Trend:")
print(monthly_sales)
```

### 4. Data Analysis (Python)
Analyze the data to derive insights.

```python
# Summary statistics
total_sales = df['TotalPrice'].sum()
avg_order_value = df.groupby('InvoiceNo')['TotalPrice'].sum().mean()
top_country = df.groupby('Country')['TotalPrice'].sum().idxmax()

print(f"Total Sales: ${total_sales:,.2f}")
print(f"Average Order Value: ${avg_order_value:,.2f}")
print(f"Top Country by Sales: {top_country}")
```

### 5. Data Visualization
Create visualizations to present your findings.

```python
# Set plot style
sns.set_style('whitegrid')

# Plot 1: Top 5 products by sales
plt.figure(figsize=(10, 6))
sns.barplot(x='TotalSales', y='Description', data=top_products)
plt.title('Top 5 Products by Total Sales')
plt.xlabel('Total Sales ($)')
plt.ylabel('Product')
plt.tight_layout()
plt.show()

# Plot 2: Sales by country
plt.figure(figsize=(10, 6))
sns.barplot(x='TotalSales', y='Country', data=country_sales)
plt.title('Top 5 Countries by Total Sales')
plt.xlabel('Total Sales ($)')
plt.ylabel('Country')
plt.tight_layout()
plt.show()

# Plot 3: Monthly sales trend
plt.figure(figsize=(12, 6))
sns.lineplot(x='Month', y='TotalSales', data=monthly_sales, marker='o')
plt.title('Monthly Sales Trend')
plt.xlabel('Month')
plt.ylabel('Total Sales ($)')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

### 6. Reporting
Summarize your findings in a Jupyter Notebook with markdown cells. Example:

```markdown
## E-Commerce Sales Analysis Report

### Key Findings
- **Total Sales**: ${total_sales:,.2f}
- **Average Order Value**: ${avg_order_value:,.2f}
- **Top Country**: {top_country}
- **Top Products**: The top 5 products contribute significantly to revenue, with {top_products.iloc[0]['Description']} leading.
- **Sales Trend**: Sales peak in [identify peak month from plot], indicating a seasonal trend.

### Recommendation
Focus marketing efforts on promoting top-selling products in {top_country} during peak sales months to maximize revenue.

### Visualizations
[Include the plots generated above]
```

### 7. Save SQL Queries
Save the SQL queries to a file for reference.

```sql
-- File: ecommerce_queries.sql
-- Query 1: Top 5 products
SELECT Description, SUM(Quantity * UnitPrice) as TotalSales
FROM sales
WHERE Quantity > 0 AND UnitPrice > 0
GROUP BY Description
ORDER BY TotalSales DESC
LIMIT 5;

-- Query 2: Sales by country
SELECT Country, COUNT(DISTINCT InvoiceNo) as TransactionCount, SUM(Quantity * UnitPrice) as TotalSales
FROM sales
WHERE Quantity > 0 AND UnitPrice > 0
GROUP BY Country
ORDER BY TotalSales DESC
LIMIT 5;

-- Query 3: Monthly sales trend
SELECT strftime('%Y-%m', InvoiceDate) as Month, SUM(Quantity * UnitPrice) as TotalSales
FROM sales
WHERE Quantity > 0 AND UnitPrice > 0
GROUP BY Month
ORDER BY Month;
```

```python
# Save SQL queries to a file
with open('ecommerce_queries.sql', 'w') as f:
    f.write("""
-- Query 1: Top 5 products
SELECT Description, SUM(Quantity * UnitPrice) as TotalSales
FROM sales
WHERE Quantity > 0 AND UnitPrice > 0
GROUP BY Description
ORDER BY TotalSales DESC
LIMIT 5;

-- Query 2: Sales by country
SELECT Country, COUNT(DISTINCT InvoiceNo) as TransactionCount, SUM(Quantity * UnitPrice) as TotalSales
FROM sales
WHERE Quantity > 0 AND UnitPrice > 0
GROUP BY Country
ORDER BY TotalSales DESC
LIMIT 5;

-- Query 3: Monthly sales trend
SELECT strftime('%Y-%m', InvoiceDate) as Month, SUM(Quantity * UnitPrice) as TotalSales
FROM sales
WHERE Quantity > 0 AND UnitPrice > 0
GROUP BY Month
ORDER BY Month;
""")
print("SQL queries saved to 'ecommerce_queries.sql'.")
```

## Deliverables
- **Jupyter Notebook**: Includes all Python code, visualizations, and markdown cells with explanations and findings.
- **SQL File**: `ecommerce_queries.sql` with your queries.
- **Report**: A markdown section in the notebook summarizing insights and recommendations.

## Tips
- **Error Handling**: The code includes try-except blocks to handle common issues like missing files or invalid data.
- **Modularity**: The `run_query` function makes it easy to execute and reuse SQL queries.
- **Validation**: Cross-check SQL results with Pandas to ensure accuracy.
- **Portfolio**: Upload your notebook and SQL file to GitHub for a professional portfolio piece.

## Learning Outcomes
- Master data cleaning and analysis with Pandas.
- Practice SQL for querying relational data.
- Create clear visualizations with Matplotlib/Seaborn.
- Develop skills in reporting and deriving actionable insights.

## Next Steps
- Share your notebook with me for feedback.
- Extend the project by exploring customer segmentation or predictive modeling (e.g., using scikit-learn).
- Explore other datasets on Kaggle for more practice.

Happy coding, and I look forward to your results!
