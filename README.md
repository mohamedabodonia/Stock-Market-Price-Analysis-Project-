# Stock-Market-Price-Analysis-Project

# OverView


This project leverages **Apache Spark** for big data processing to analyze stock market price trends efficiently.  
Using **PySpark** on **Databricks**, we perform data ingestion, transformation, and visualization of large-scale stock market data.  
The project explores key data engineering techniques, including ETL (Extract, Transform, Load), data cleaning, feature engineering, and time-series analysis.

By utilizing Spark’s distributed computing power, this analysis helps uncover market trends, volatility patterns, and predictive insights.  
The project demonstrates how to handle real-world financial data and optimize big data workflows in a scalable cloud environment.

# project Workflow  

**1. Data Ingestion**  
Import stock market data from CSV file.  
Load data into Databricks using PySpark DataFrames.  

**2. Data Preprocessing & Cleaning**  
Handle missing values, duplicates, and data inconsistencies.  
Writing simple PySpark UDF using lambda function to Convert date columns.  
Filter and standardize stock symbols, ensuring data consistency.  

**3. Exploratory Data Analysis (EDA)**   
Compute summary statistics (mean, describe, max,min, etc.).    
Visualize closing prices, trading volumes, and stock trends in Databricks notebooks.  

**4. Data Storage**  
Store processed data in Parquet/Delta format for efficient querying.  
Optimize performance using Spark partitions.  


# Project Step

**Creating spark session**

```python
# Initialize Spark session

import pyspark
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("StockAnalysis").getOrCreate()
```

**Reading data file**

```python
# create a variable to point the file source
stock=spark.read.csv("/FileStore/tables/ALL_STOCKS.csv",header=True,inferSchema=True)
```
**printing the scema of data**

```python
# show the scema of data 
stock.printSchema()
```
**show sample of data and make filteration on data**

```python
stock.select("Ticker").show(5)
stock.filter(stock.Ticker == "MSFT").show(5)
```

**filter data using isin function**

```python
stock.filter((stock.Ticker.isin(["AMZN","MSFT"]))&(stock.Date=="2023-05-31")).show()
```

**making transformation on close/open/high/low column to delete "$" symbol from this column**

```python
def numparser(value):
  if isinstance(value,str):
    return float(value.strip("$"))
  elif isinstance(value,int) or isinstance(value,float):
    return value
  else:
      return None

from pyspark.sql.functions import udf
from pyspark.sql.types import FloatType
parserfloat = udf(numparser,FloatType())
stock= stock.withColumn("Open",parserfloat("Open")).withColumn("Close",parserfloat("Close/Last")).withColumn("High",parserfloat("High")).withColumn("Low",parserfloat("Low"))

```

**select specefic column from data and save the column on "cleaned_stock"**

```python
cleaned_stock=stock.select("Ticker","Date","Volume","Open","High","Low","Close")
 ```
