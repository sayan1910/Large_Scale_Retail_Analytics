# 📊 Large-Scale Retail Analytics & Power BI Insights

## 🔍 Project Overview

This project demonstrates an end-to-end large-scale retail analytics pipeline, built using the Online Retail II dataset, to simulate real-world retail business scenarios.

The objective is to transform raw transactional data into actionable business insights using:

 - Advanced NumPy-based data processing
 - Performance-oriented data cleaning
 - Analytical KPIs for store and category performance
 - A/B testing for promotion impact
 - Power BI dashboards for stakeholder decision-making

The project mirrors challenges faced by large grocery and e-commerce retailers, including missing values, noisy data, high transaction volume, and pricing complexity.

## Dataset

Online Retail II Dataset (Kaggle)
A public dataset with ~1M retail transactions containing:

 - Invoice details
 - Product descriptions and pricing
 - Quantities sold and returns
 - Customer identifiers
 - Transaction timestamps
 - Geographic information



## 🛠️ Tools & Technologies

 - Python: NumPy, Pandas
 - Advanced NumPy: Boolean masking, vectorization, reshaping
 - Power BI: DAX, interactive dashboards
 - Git & GitHub: Version control and documentation


## ⚙️ Step-by-Step Project Implementation


### Step 1: Data Extraction
Raw data was loaded from the Online Retail II Excel file.
Only business-critical columns were selected:
Invoice number and date
Product details
Quantity and unit price
Customer ID
Country (store proxy)

```
df_dict = pd.read_excel(
    '../Data/Input/online_retail_II.xlsx',
    sheet_name=['Year 2009-2010', 'Year 2010-2011']
)

df = pd.concat(df_dict.values(), ignore_index=True)
df
```

#### Outcome: Reduced data volume and faster downstream processing.


### Step 2: Data Filtering

Applied business-driven filtering rules:
Removed records with missing transaction dates
Removed zero or invalid prices
Retained negative quantities (returns) for return analysis

#### Outcome: Reliable transactional dataset for analytics.



### Step 3: Handling Missing Values & NaNs

Missing CustomerID values were treated as Non-Loyal Customers
Critical numeric fields were validated and cleaned
No blind deletion of rows — logic followed business meaning


```
df['UnitPrice'] = df.groupby('Category')['Price'] \
                    .transform(lambda x: x.fillna(x.median()))

df = df[df['UnitPrice'] > 0]
```


#### Outcome: Realistic customer segmentation without KPI distortion.

### Step 4: Feature Engineering: Product Category Derivation

The original retail dataset did not contain a structured product category field, which is essential for category-level KPI analysis, inventory planning, and business reporting. To address this, a rule-based text classification approach was applied using product descriptions.

```
df['Category'] = np.where(
    df['Description'].str.contains('SET|PACK|BAG', na=False),
    'Household',
    'Grocery'
)
```

### Step 5: Advanced NumPy Operations

To ensure scalability and performance, the project heavily uses:

 - Boolean Masking
This approach is vectorized, efficient, and commonly used for pricing logic, promotions, and feature engineering.

```
Premium_Masking  = df['Price'] > 10
df['Extra_Discount'] = np.where(Premium_Masking, 5, 0) 
```

 - Vectorization
Applied pricing, discount, tax, and revenue calculations without loops.

```
discount_map = {
        'Household' : 5,
        'Grocery' : 10
}
df['Base_Discount'] = df['Category'].map(discount_map)

Premium_Masking  = df['Price'] > 10
df['Extra_Discount'] = np.where(Premium_Masking, 5, 0) 

df['Total_Discount'] = df['Base_Discount'] + df['Extra_Discount']

df['Price_After_Discount'] = df['Price'] * (1 - df['Total_Discount'] / 100)
df['Final_Price'] = df['Price_After_Discount'] * (1 + Gst)
df['Final_Price']
```

 - Broadcasting
Applied discount and tax rates across large arrays efficiently.

```
Gst = 18
```

 - Reshaping & Multidimensional Arrays
I reshaped transactional data into a multidimensional demand matrix using groupby and unstack, where time is one axis and country is another. This enables efficient vectorized analysis, time-series modeling, and BI reporting.

```
df['date'] = df['InvoiceDate'].dt.date

daily_demand = (
        df.groupby(['date', 'Country'])['Quantity']
        .sum()
        .unstack(fill_value = 0)
)
df['date'].tail(20)
```
#### Outcome: High-performance preprocessing suitable for large retail datasets.


### Step 6: A/B Testing 
#### Calculated:

 - Revenue lift or loss

```
Control_Revenue = np.sum(df['Price'] * df['Quantity'])
Treated_Revenue = np.sum(df['Final_Price'] * df['Quantity'])

Lift = np.round((Treated_Revenue - Control_Revenue )/ Control_Revenue * 100 , 2)
Lift
```

### Step 7: Data Extraction for Power BI Consumption

After completing data cleaning, feature engineering, and KPI calculations in the Jupyter Notebook, the processed datasets were exported as Power BI–ready CSV files. This ensured a clear separation between data preparation (Python) and data visualization (Power BI).

```
df.to_csv('../Data/Output/Retail_Clean_Fact_Sales.csv', index=False)
Store_Kpi.to_csv('../Data/Output/Retail_Store_Kpi.csv', index=False)
daily_demand.to_csv('../Data/Output/Retail_Daily_Demand.csv')
```
Extracted Files

 1.Retail_Clean_Fact_Sales.csv → Transaction-level fact table
 2.Retail_Store_Kpi.csv → Store-level aggregated KPIs
 3.Retail_Daily_Demand.csv → Date × Country demand matrix

### Step 8: Power BI Data Model Creation

#### The extracted datasets were loaded into Power BI Desktop and modeled using a star schema approach.

Data Model Structure

Fact Table
 - Retail_Clean_Fact_Sales

Aggregated Tables
 - Retail_Store_Kpi
 - Retail_Daily_Demand


### Step 9: Power BI Visualisation (Dashboard Output)

The final interactive dashboard was developed in Power BI Desktop using the modeled datasets and DAX measures.
To make the visual insights easily accessible on GitHub (without requiring Power BI), the dashboard was exported as a PDF and linked below.

📄 Dashboard Preview (PDF)

📊 **Power BI Dashboard**  
👉 [View Dashboard (PDF)](https://github.com/sayan1910/Large_Scale_Retail_Analytics/blob/main/Power%20BI/Retail_Dashboard1.pdf)


### 📌 Key Business Insights & Findings

Based on the Power BI dashboard analysis:

- Total Revenue reached $358.26M, driven primarily by the Grocery category (~77%), indicating grocery items as the core revenue driver.

- Loyal customers contribute ~87% of total revenue ($313M), highlighting strong customer retention and the importance of loyalty programs.

- Promotion Impact is negative (-16.23%), suggesting that discounts may be eroding revenue rather than increasing volume.

- UK dominates sales and demand, contributing the highest revenue and daily demand compared to other countries.

- Household category shows higher stock days (~21 days) versus Grocery (~7 days), indicating slower inventory movement and potential overstock risk.

- Demand trends display high volatility with seasonal spikes, reinforcing the need for demand forecasting and rolling averages.



### 📊 KPIs Tracked

The dashboard tracks the following business-critical KPIs:

Total Revenue & Units Sold

 - Average Selling Price (ASP)
 - Promotion Impact %
 - Daily Demand & 7-Day Rolling Demand
 - Stock Days & Recommended Stock
 - Return %
 - Revenue by Country, Category & Loyalty Status

### Step 10: 🔄 Version Control & GitHub Workflow

This project follows best practices for managing large datasets and Power BI files using Git and Git LFS.

#### 🧩 Repository Setup

 - Created a GitHub repository to manage source code, datasets, and dashboards.
 - Organized files into structured folders for data, notebooks, and Power BI assets.
 - Initialized Git locally and linked it to the remote GitHub repository.

 <!--### 📂 Handling Large Files with Git LFS

Since datasets and Data/output/ Retail_Clean_Fact_Sales.csv files exceed GitHub’s 100 MB limit, Git Large File Storage (LFS) was used.
```
git lfs install
git lfs track "Retail_Clean_Fact_Sales.csv
git add .gitattributes
git commit -m "Configured Git LFS for large files"
```

### 📤 Pushing Data & Code to GitHub

Added Python scripts, notebook, and cleaned datasets.

Committed changes with meaningful messages for version tracking.

```
git add .
git commit -m "Added data cleaning, analysis scripts, and datasets"
git push origin main
```
### 📊 Managing Power BI Files
-->
Included both formats for usability:

 1. .pbix → Interactive Power BI dashboard

 2. .pdf → Quick preview for recruiters
```
git add PowerBI/Retail_Dashboard.pbix
git commit -m "Updated Power BI dashboard"
git push origin main
```







