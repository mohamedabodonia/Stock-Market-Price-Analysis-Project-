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


**Get the max open value for each ticker**

```python
import pyspark.sql.functions as func
# maximum value in each stock
cleaned_stock.groupBy("Ticker").agg(func.max("Open").alias("MAXSTOCK")).show()
```

**Get the total volume value for each stock**

```python
cleaned_stock.groupBy("Ticker").agg(func.sum("Volume").alias("Totalvolume")).show()
```

**Add (year, month, day) column to the Cleaned data**

```python
cleaned_stock=cleaned_stock.withColumn("year",func.year("Date")).withColumn("month",func.month("Date")).withColumn("day",func.dayofmonth("Date"))
```

**calcualte the maximum and minimum value for each Ticker per YEAR**

```python
cleaned_stock.groupBy("Ticker","year").agg(func.max("Open").alias("yearlyhigh"),func.min("open").alias("lowyear")).show()
```
**calcualte the maximum and minimum value for each Ticker per month**

```python
monthlyanalysis=cleaned_stock.groupBy("Ticker","year","month").agg(func.max("Open").alias("highmonth"),func.min("open").alias("lowmonth"))
monthlyanalysis.show(5)
```

**join Cleaned-data with the monthlyanalysis data**

```python
historicalAnalysis=cleaned_stock.join(monthlyanalysis,(cleaned_stock.year==monthlyanalysis.year) &(cleaned_stock.month==monthlyanalysis.month)& (cleaned_stock.Ticker==monthlyanalysis.Ticker),how="inner").drop(monthlyanalysis.year,monthlyanalysis.month,monthlyanalysis.Ticker )
```

**Select Wpecific column and save it as a View "stockData"**
```python
finalstocks=historicalAnalysis.select(["Ticker","high","low","year","month","open","close","highmonth","lowmonth","Date"]).createOrReplaceTempView("stockData")
```

**Using spark.sql function to deal with view**

```python
spark.sql("select * from stockData where Ticker='AAPL'").show(15)
```

**Using window Function to compare each open value with the last day value**

```python
from pyspark.sql.window import Window
lag1day=Window.partitionBy("Ticker").orderBy("Date")
snapshot.withColumn("previousopen",func.lag("Open",1).over(lag1day)).show(50)
```

**Saving Data as CSV in DBFS**

```python
cleaned_stock.write.mode("overwrite").option("header", "true").csv("/mnt/data/stock_prices")
```
