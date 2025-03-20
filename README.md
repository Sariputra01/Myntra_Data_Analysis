# Myntra_Data_Analysis
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

xls = pd.ExcelFile('C:/Users/kk/Downloads/Myntra dataset.xlsx')

df_products = xls.parse("dim_products")
df_customers = xls.parse("dim_customers")
df_orders = xls.parse("fact_orders")

print(df_products.head())
print(df_customers.head())
print(df_orders.head())

df_products.info()

df_products.isnull().sum()

df_products.describe()

df_orders['Date'] = pd.to_datetime(df_orders['Date'])

df_orders['Selling Price'] = df_orders['Original Price'] * (1 - df_orders['Discount%'])

df_orders['Year'] = df_orders['Date'].dt.year
df_orders['Month'] = df_orders['Date'].dt.month
print("Cleaning and processing data...")

print("Analyzing sales trends...")

sales_trends = df_orders.groupby(['Year', 'Month'])['Selling Price'].sum().reset_index()
plt.figure(figsize=(12,4))  # Reduced size
sns.lineplot(data=sales_trends, x='Month', y='Selling Price', hue='Year', marker='o')
plt.title("Monthly Sales Trend")
plt.xlabel("Month")
plt.ylabel("Total Sales (₹)")
plt.legend(title="Year", loc="upper right", fontsize=6)
plt.show()

print("Analyzing customer purchasing behavior...")
customer_spending = df_orders.groupby('Customer ID')['Selling Price'].sum().reset_index()
plt.figure(figsize=(12, 4))
sns.histplot(customer_spending['Selling Price'], bins=30, kde=True)
plt.title("Customer Spending Distribution")
plt.xlabel("Total Spend (₹)")
plt.ylabel("Number of Customers")
plt.show()

print("Analyzing impact of discounts on sales...")

discount_analysis = df_orders.groupby('Discount%')['Selling Price'].sum().reset_index()
plt.figure(figsize=(12, 4))
sns.lineplot(data=discount_analysis, x='Discount%', y='Selling Price', marker='o')
plt.title("Impact of Discount on Sales")
plt.xlabel("Discount Percentage")
plt.ylabel("Total Sales (₹)")
plt.show()

# Identify top spending customers
top_customers = df_orders.groupby('Customer ID')['Selling Price'].sum().reset_index()
top_customers = top_customers.sort_values(by='Selling Price', ascending=False).head(10)

plt.figure(figsize=(12,4))  
sns.barplot(
    data=top_customers, 
    x='Customer ID', 
    y='Selling Price', 
    hue='Customer ID',
    palette="Blues_r",
    legend=False
)
plt.xticks(rotation=45)
plt.title("Top 10 Highest Spending Customers")
plt.show()

# Merge orders with product data to get category information
category_sales = df_orders.merge(df_products, on='Product ID')
category_sales = category_sales.groupby('Category')['Selling Price'].sum().reset_index()

plt.figure(figsize=(12, 4))  
sns.barplot(
    data=category_sales, 
    x='Category', 
    y='Selling Price', 
    hue='Category',  
    palette="coolwarm",  
    legend=False,  
    width=0.6
)
plt.xticks(rotation=45)
plt.title("Sales by Product Category")
plt.xlabel("Category")
plt.ylabel("Total Sales (₹)")
plt.show()
