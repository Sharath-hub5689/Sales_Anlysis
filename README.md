
get_ipython().system('pip install pandas numpy openpyxl sqlalchemy pymysql')

import pandas as pd
import numpy as np

df = pd.read_excel(r"C:\Users\sharath r\Videos\Ecommerce_Unclean_Project.xlsx")

df.head(10)

df.info()

df.describe()

df.isnull().sum()

df.duplicated().sum()

df.drop_duplicates(inplace=True)

df.duplicated().sum()

## Replace comma invalid values with NAN 

df.replace(['N/A','NULL',''],np.nan,inplace=True)

## removing space
df = df.apply(lambda x: x.str.strip() if x.dtype =='object' else x)


## case sensitive

df['Customer_Name'] = df['Customer_Name'].str.title()

df['City'] = df['City'].str.title()

df['State'] = df['State'].str.title()


df = df[df['Email'].str.contains('@',na=False)]



## convert date columns

df['Order_Date']=pd.to_datetime(df['Order_Date'],errors='coerce')
df['Delivery_Date']=pd.to_datetime(df['Delivery_Date'],errors='coerce')

print(df['Order_Date'].head())
print(df['Delivery_Date'].head())


cols = ['Qty', 'unit_price', 'Discount']

for c in cols:
    df[c] = pd.to_numeric(df[c], errors='coerce')


print(df.columns.tolist())


df.columns = df.columns.str.strip().str.replace(' ', '_')

print(df.columns.tolist())



cols = ['Qty', 'Unit_Price', 'Discount']

for c in cols:
    df[c] = pd.to_numeric(df[c], errors='coerce')

##remove negative or zero qty

df = df[df['Qty']>0]

## fill missing values

df['Discount'] = df['Discount'].fillna(0)

df['Phone'] = df['Phone'].fillna('Unknown')

df['Delivery_Date'] = df['Delivery_Date'].fillna(df['Order_Date'])

## total sales

df['Sales']=df['Qty']*df['Unit_Price']

df['Net Amount']=df['Sales']-(df['Sales']*df['Discount']/100)

df['Profit']=df['Net Amount']*0.20

df['Month']=df['Order_Date'].dt.month_name()

df["Year"]=df['Order_Date'].dt.year

df['Weekday']=df['Order_Date'].dt.day_name()

df.info()

df.isnull().sum()

df.to_csv('Clean_Ecommerce.csv',index=False)

from sqlalchemy import create_engine
engine = create_engine("mysql+pymysql://root:577423@127.0.0.1:3306/ecom")


df.to_sql(
    name='orders',
    con=engine,
    if_exists='replace',
    index=False
)


df.shape


pd.read_sql(
    "SELECT SUM(Net_amount) AS Total_Sales FROM orders",
    engine
)


from sqlalchemy import text

with engine.connect() as conn:
    result = conn.execute(text("SHOW TABLES"))
    for row in result:
        print(row)



df.to_sql(
    name='orders',
    con=engine,
    if_exists='replace',
    index=False
)



DESCRIBE orders;



SQL QUERIES


USE ecommerce;

-- View Data
SELECT * FROM orders;
SELECT * FROM orders LIMIT 10;

-- Check Table Structure
DESC orders;

-- Create Net Amount Column
ALTER TABLE orders
ADD COLUMN Net_Amount DECIMAL(10,2);

-- Calculate Net Amount
UPDATE orders
SET Net_Amount =
    Qty * Unit_Price * (1 - IFNULL(Discount,0)/100);

-- Create Profit Column
ALTER TABLE orders
ADD COLUMN Profit DECIMAL(10,2);

-- Calculate Profit (20% Margin)
UPDATE orders
SET Profit = Net_Amount * 0.20;

-- Total Sales
SELECT
    ROUND(SUM(Net_Amount),2) AS Total_Sales
FROM orders;

-- Total Profit
SELECT
    ROUND(SUM(Profit),2) AS Total_Profit
FROM orders;

-- Average Order Value
SELECT
    ROUND(AVG(Net_Amount),2) AS Average_Order_Value
FROM orders;

-- Total Orders
SELECT
    COUNT(*) AS Total_Orders
FROM orders;

-- Top 10 Customers
SELECT
    Customer_Name,
    ROUND(SUM(Net_Amount),2) AS Total_Spent
FROM orders
GROUP BY Customer_Name
ORDER BY Total_Spent DESC
LIMIT 10;

-- Top 10 Products by Revenue
SELECT
    Product,
    ROUND(SUM(Net_Amount),2) AS Revenue
FROM orders
GROUP BY Product
ORDER BY Revenue DESC
LIMIT 10;

-- Sales by Category
SELECT
    Category,
    ROUND(SUM(Net_Amount),2) AS Total

