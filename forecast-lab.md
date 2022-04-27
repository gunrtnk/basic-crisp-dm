# Lab 4 - Student Notebook

## Overview

In this lab, you will prepare a dataset for creating a forecast by using Amazon Forecast.

This lab includes two Jupyter notebooks:

1. This notebook contains the steps that you will follow to prepare the dataset and evaluate the forecast.
2. The `forecast-autorun.ipynb` notebook contains the steps to create the forecast by using Amazon Forecast. This notebook is run in the background when the lab starts, and it can take between 1–2 hours to complete. You will refer to this notebook during the lab steps, but you won't need to run any cells.


## About the dataset

This [Online Retail II](https://archive.ics.uci.edu/ml/datasets/Online+Retail+II) dataset contains all transactions that occurred between January 12, 2009 and September 12, 2011 for a non-store, online retail organization that's registered and based in the United Kingdom. The company mainly sells unique all-occasion giftware. Many customers of the company are wholesalers.


## Attribute information

- **InvoiceNo** – Invoice number. Nominal. A 6-digit integral number that's uniquely assigned to each transaction. If this code starts with the letter *c*, it indicates a cancelation.
- **StockCode** – Product (item) code. Nominal. A 5-digit integral number that's uniquely assigned to each distinct product.
- **Description** – Product (item) name. Nominal.
- **Quantity** – The quantities of each product (item) per transaction. Numeric.
- **InvoiceDate** – Invoice date and time. Numeric. The day and time when a transaction was generated.
- **UnitPrice** – Unit price. Numeric. Product price per unit in pounds sterling (£).
- **CustomerID** – Customer number. Nominal. A 5-digit integral number that's uniquely assigned to each customer.
- **Country** – Country name. Nominal. The name of the country where a customer resides.


## Dataset attributions

This dataset was obtained from:
Dua, D. and Graff, C. (2019). UCI Machine Learning Repository (http://archive.ics.uci.edu/ml). Irvine, CA: University of California, School of Information and Computer Science.

## Lab instructions

To complete this lab, read and run the cells below.

## Task 1: Importing Python packages

Start by importing the Python packages that you need.

In the following code:

- *boto3* represents the AWS SDK for Python (Boto3), which is the Python library for AWS
- *pandas* provides DataFrames for manipulating time series data
- *matplotlib* provides plotting functions
- *sagemaker* represents the API that's needed to work with Amazon SageMaker
- *time*, *sys*, *os*, *io*, and *json* provide helper functions 



```python
import warnings
warnings.filterwarnings('ignore')
bucket_name='c47433a664956l1987599t1w5868757851-forecastbucket-a1zl74oqu7di'

import boto3
import pandas as pd
import matplotlib
import matplotlib.pyplot as plt
import sagemaker
import time, sys, os, io, json
import xlrd

```

## Task 2: Exploring the data


The data is in the *Microsoft Excel* format. pandas can read Excel files.

**Note:** This data might take 1–2 minutes to load


```python
retail = pd.read_excel('online_retail_II.xlsx',engine='openpyxl')
```

According to the description for the dataset, some values are missing. To keep things simple, you will remove anything wtih a missing value.


```python
retail = retail.dropna()
```

Start by examining the data.

How many rows and columns are in the dataset?


```python
retail.shape
```




    (417534, 8)



What are the data types?


```python
retail.dtypes
```




    Invoice                object
    StockCode              object
    Description            object
    Quantity                int64
    InvoiceDate    datetime64[ns]
    Price                 float64
    Customer ID           float64
    Country                object
    dtype: object



What does the data look like?


```python
retail.head(20)
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
      <th>Invoice</th>
      <th>StockCode</th>
      <th>Description</th>
      <th>Quantity</th>
      <th>InvoiceDate</th>
      <th>Price</th>
      <th>Customer ID</th>
      <th>Country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>489434</td>
      <td>85048</td>
      <td>15CM CHRISTMAS GLASS BALL 20 LIGHTS</td>
      <td>12</td>
      <td>2009-12-01 07:45:00</td>
      <td>6.95</td>
      <td>13085.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>1</th>
      <td>489434</td>
      <td>79323P</td>
      <td>PINK CHERRY LIGHTS</td>
      <td>12</td>
      <td>2009-12-01 07:45:00</td>
      <td>6.75</td>
      <td>13085.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2</th>
      <td>489434</td>
      <td>79323W</td>
      <td>WHITE CHERRY LIGHTS</td>
      <td>12</td>
      <td>2009-12-01 07:45:00</td>
      <td>6.75</td>
      <td>13085.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>3</th>
      <td>489434</td>
      <td>22041</td>
      <td>RECORD FRAME 7" SINGLE SIZE</td>
      <td>48</td>
      <td>2009-12-01 07:45:00</td>
      <td>2.10</td>
      <td>13085.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>4</th>
      <td>489434</td>
      <td>21232</td>
      <td>STRAWBERRY CERAMIC TRINKET BOX</td>
      <td>24</td>
      <td>2009-12-01 07:45:00</td>
      <td>1.25</td>
      <td>13085.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>5</th>
      <td>489434</td>
      <td>22064</td>
      <td>PINK DOUGHNUT TRINKET POT</td>
      <td>24</td>
      <td>2009-12-01 07:45:00</td>
      <td>1.65</td>
      <td>13085.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>6</th>
      <td>489434</td>
      <td>21871</td>
      <td>SAVE THE PLANET MUG</td>
      <td>24</td>
      <td>2009-12-01 07:45:00</td>
      <td>1.25</td>
      <td>13085.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>7</th>
      <td>489434</td>
      <td>21523</td>
      <td>FANCY FONT HOME SWEET HOME DOORMAT</td>
      <td>10</td>
      <td>2009-12-01 07:45:00</td>
      <td>5.95</td>
      <td>13085.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>8</th>
      <td>489435</td>
      <td>22350</td>
      <td>CAT BOWL</td>
      <td>12</td>
      <td>2009-12-01 07:46:00</td>
      <td>2.55</td>
      <td>13085.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>9</th>
      <td>489435</td>
      <td>22349</td>
      <td>DOG BOWL , CHASING BALL DESIGN</td>
      <td>12</td>
      <td>2009-12-01 07:46:00</td>
      <td>3.75</td>
      <td>13085.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>10</th>
      <td>489435</td>
      <td>22195</td>
      <td>HEART MEASURING SPOONS LARGE</td>
      <td>24</td>
      <td>2009-12-01 07:46:00</td>
      <td>1.65</td>
      <td>13085.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>11</th>
      <td>489435</td>
      <td>22353</td>
      <td>LUNCHBOX WITH CUTLERY FAIRY CAKES</td>
      <td>12</td>
      <td>2009-12-01 07:46:00</td>
      <td>2.55</td>
      <td>13085.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>12</th>
      <td>489436</td>
      <td>48173C</td>
      <td>DOOR MAT BLACK FLOCK</td>
      <td>10</td>
      <td>2009-12-01 09:06:00</td>
      <td>5.95</td>
      <td>13078.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>13</th>
      <td>489436</td>
      <td>21755</td>
      <td>LOVE BUILDING BLOCK WORD</td>
      <td>18</td>
      <td>2009-12-01 09:06:00</td>
      <td>5.45</td>
      <td>13078.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>14</th>
      <td>489436</td>
      <td>21754</td>
      <td>HOME BUILDING BLOCK WORD</td>
      <td>3</td>
      <td>2009-12-01 09:06:00</td>
      <td>5.95</td>
      <td>13078.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>15</th>
      <td>489436</td>
      <td>84879</td>
      <td>ASSORTED COLOUR BIRD ORNAMENT</td>
      <td>16</td>
      <td>2009-12-01 09:06:00</td>
      <td>1.69</td>
      <td>13078.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>16</th>
      <td>489436</td>
      <td>22119</td>
      <td>PEACE WOODEN BLOCK LETTERS</td>
      <td>3</td>
      <td>2009-12-01 09:06:00</td>
      <td>6.95</td>
      <td>13078.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>17</th>
      <td>489436</td>
      <td>22142</td>
      <td>CHRISTMAS CRAFT WHITE FAIRY</td>
      <td>12</td>
      <td>2009-12-01 09:06:00</td>
      <td>1.45</td>
      <td>13078.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>18</th>
      <td>489436</td>
      <td>22296</td>
      <td>HEART IVORY TRELLIS LARGE</td>
      <td>12</td>
      <td>2009-12-01 09:06:00</td>
      <td>1.65</td>
      <td>13078.0</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>19</th>
      <td>489436</td>
      <td>22295</td>
      <td>HEART FILIGREE DOVE LARGE</td>
      <td>12</td>
      <td>2009-12-01 09:06:00</td>
      <td>1.65</td>
      <td>13078.0</td>
      <td>United Kingdom</td>
    </tr>
  </tbody>
</table>
</div>



Amazon Forecast has schemas for domains such as retail. Review the schema information at [RETAIL Domain](https://docs.aws.amazon.com/forecast/latest/dg/retail-domain.html) in the AWS Documentation.

The target time series is the historical time series data for each item or product that's sold by the retail organization. The following fields are required:

- **item_id** (string) – A unique identifier for the item or product that you want to predict the demand for.
- **timestamp** (timestamp)
- **demand** (float) – The number of sales for that item at the timestamp. It's also the target field that Amazon Forecast generates a forecast for.



If you examine the previous data, there are certain columns that you don't need for your investigation. You can drop these columns. The columns you can drop are **Invoice**, **Description**, and **Customer ID**. 

**Note:** It's possible that items in the same order (as shown by the **Invoice** column) could have a correlation that impacts the model. For this lab, you will ignore this possibility.

Drop the columns that you don't need.


```python
retail = retail[['StockCode','Quantity','Price','Country','InvoiceDate']]
```

The **InvoiceDate** column is your datetime data. You can inform pandas of this by using the `to_datetime` function. You can explore the data by time by setting the index of the DataFrame to the **InvoiceDate** column.


```python
retail['InvoiceDate'] = pd.to_datetime(retail.InvoiceDate)
retail = retail.set_index('InvoiceDate')
```

You will now examine the updated DataFrame.

The number of rows and columns are:


```python
retail.shape
```




    (417534, 4)



The new data looks like this example:


```python
retail.head()
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
      <th>StockCode</th>
      <th>Quantity</th>
      <th>Price</th>
      <th>Country</th>
    </tr>
    <tr>
      <th>InvoiceDate</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>85048</td>
      <td>12</td>
      <td>6.95</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>79323P</td>
      <td>12</td>
      <td>6.75</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>79323W</td>
      <td>12</td>
      <td>6.75</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>22041</td>
      <td>48</td>
      <td>2.10</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>21232</td>
      <td>24</td>
      <td>1.25</td>
      <td>United Kingdom</td>
    </tr>
  </tbody>
</table>
</div>



Note that **InvoiceDate** is the index, and it's shown in the first column.

Because you set the index to your datetime data, you can use it to select data.

To select all the rows from a specific date, use the date in the index.


```python
retail['2010-01-04']
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
      <th>StockCode</th>
      <th>Quantity</th>
      <th>Price</th>
      <th>Country</th>
    </tr>
    <tr>
      <th>InvoiceDate</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2010-01-04 09:24:00</th>
      <td>TEST001</td>
      <td>5</td>
      <td>4.50</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2010-01-04 09:43:00</th>
      <td>21539</td>
      <td>-1</td>
      <td>4.25</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2010-01-04 09:53:00</th>
      <td>TEST001</td>
      <td>5</td>
      <td>4.50</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2010-01-04 10:28:00</th>
      <td>21844</td>
      <td>36</td>
      <td>2.55</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2010-01-04 10:28:00</th>
      <td>21533</td>
      <td>12</td>
      <td>4.25</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2010-01-04 17:39:00</th>
      <td>90214G</td>
      <td>1</td>
      <td>1.25</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2010-01-04 17:39:00</th>
      <td>90214N</td>
      <td>1</td>
      <td>1.25</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2010-01-04 17:39:00</th>
      <td>90214N</td>
      <td>1</td>
      <td>1.25</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2010-01-04 17:39:00</th>
      <td>90214C</td>
      <td>1</td>
      <td>1.25</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2010-01-04 17:39:00</th>
      <td>21690</td>
      <td>2</td>
      <td>3.75</td>
      <td>United Kingdom</td>
    </tr>
  </tbody>
</table>
<p>633 rows × 4 columns</p>
</div>



You can use parts of a date, and date ranges. To view the **Jan** and **Feb** rows:


```python
retail['2010-01':'2010-02']
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
      <th>StockCode</th>
      <th>Quantity</th>
      <th>Price</th>
      <th>Country</th>
    </tr>
    <tr>
      <th>InvoiceDate</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2010-01-04 09:24:00</th>
      <td>TEST001</td>
      <td>5</td>
      <td>4.50</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2010-01-04 09:43:00</th>
      <td>21539</td>
      <td>-1</td>
      <td>4.25</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2010-01-04 09:53:00</th>
      <td>TEST001</td>
      <td>5</td>
      <td>4.50</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2010-01-04 10:28:00</th>
      <td>21844</td>
      <td>36</td>
      <td>2.55</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2010-01-04 10:28:00</th>
      <td>21533</td>
      <td>12</td>
      <td>4.25</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2010-02-28 16:14:00</th>
      <td>84279B</td>
      <td>1</td>
      <td>3.75</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2010-02-28 16:14:00</th>
      <td>84882</td>
      <td>1</td>
      <td>3.75</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2010-02-28 16:14:00</th>
      <td>84882</td>
      <td>1</td>
      <td>3.75</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2010-02-28 16:14:00</th>
      <td>44242B</td>
      <td>5</td>
      <td>1.25</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <th>2010-02-28 16:16:00</th>
      <td>10133</td>
      <td>40</td>
      <td>0.85</td>
      <td>United Kingdom</td>
    </tr>
  </tbody>
</table>
<p>46345 rows × 4 columns</p>
</div>



The date range starts at:


```python
retail.index.min()
```




    Timestamp('2009-12-01 07:45:00')



The date range ends at:


```python
retail.index.max()
```




    Timestamp('2010-12-09 20:01:00')



With pandas, you can extract date information easily. You might extract date information to explore the data further and look for time-related trends.

Extract the year, month, and day of the week.


```python
retail['Year'] = retail.index.year
retail['Month'] = retail.index.month
retail['weekday_name'] = retail.index.day_name()
```


```python
retail.head()
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
      <th>StockCode</th>
      <th>Quantity</th>
      <th>Price</th>
      <th>Country</th>
      <th>Year</th>
      <th>Month</th>
      <th>weekday_name</th>
    </tr>
    <tr>
      <th>InvoiceDate</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>85048</td>
      <td>12</td>
      <td>6.95</td>
      <td>United Kingdom</td>
      <td>2009</td>
      <td>12</td>
      <td>Tuesday</td>
    </tr>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>79323P</td>
      <td>12</td>
      <td>6.75</td>
      <td>United Kingdom</td>
      <td>2009</td>
      <td>12</td>
      <td>Tuesday</td>
    </tr>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>79323W</td>
      <td>12</td>
      <td>6.75</td>
      <td>United Kingdom</td>
      <td>2009</td>
      <td>12</td>
      <td>Tuesday</td>
    </tr>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>22041</td>
      <td>48</td>
      <td>2.10</td>
      <td>United Kingdom</td>
      <td>2009</td>
      <td>12</td>
      <td>Tuesday</td>
    </tr>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>21232</td>
      <td>24</td>
      <td>1.25</td>
      <td>United Kingdom</td>
      <td>2009</td>
      <td>12</td>
      <td>Tuesday</td>
    </tr>
  </tbody>
</table>
</div>



The dataset that you now have includes purchases made between December 2009 and December 2010. It's reasonable to assume there would be some seasonality in this data. You will now investigate whether there is seasonality.


```python
retail.Month.value_counts(sort=False).plot(kind='bar')
```




    <AxesSubplot:>




![png](output_38_1.png)


From the chart, you could deduce some seasonality:

1. November and December seem to be higher than the rest of the year.

2. Q4 seems to be higher than other quarters.

3. For Q1, Q2, and Q3: The last month of the quarter (months 3, 6, and 9) seem to have spikes.

Do you notice any other seasonal patterns?

Now, investigate whether there is any seasonality during the week.


```python
day_order = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"]
retail.weekday_name.value_counts(sort=False).loc[day_order].plot(kind='bar')
```




    <AxesSubplot:>




![png](output_41_1.png)


Saturday shows very few orders. Why might this be the case?

## Task 3: Cleaning and reducing the size of the data

In this task, you will reduce the size of the data. You will also remove any anomalies, such as negative prices, outliers, and country data.

### Reducing the countries
Examine the **Country** data.


```python
retail.Country.unique()
```




    array(['United Kingdom', 'France', 'USA', 'Belgium', 'Australia', 'EIRE',
           'Germany', 'Portugal', 'Japan', 'Denmark', 'Netherlands', 'Poland',
           'Spain', 'Channel Islands', 'Italy', 'Cyprus', 'Greece', 'Norway',
           'Austria', 'Sweden', 'United Arab Emirates', 'Finland',
           'Switzerland', 'Unspecified', 'Nigeria', 'Malta', 'RSA',
           'Singapore', 'Bahrain', 'Thailand', 'Israel', 'Lithuania',
           'West Indies', 'Korea', 'Brazil', 'Canada', 'Iceland'],
          dtype=object)




```python
retail.Country.value_counts()
```




    United Kingdom          379423
    EIRE                      8710
    Germany                   8129
    France                    5710
    Netherlands               2769
    Spain                     1278
    Switzerland               1187
    Belgium                   1054
    Portugal                  1024
    Channel Islands            906
    Sweden                     883
    Italy                      731
    Australia                  654
    Cyprus                     554
    Austria                    537
    Greece                     517
    Denmark                    428
    Norway                     369
    Finland                    354
    United Arab Emirates       318
    Unspecified                280
    USA                        244
    Japan                      224
    Poland                     194
    Malta                      172
    Lithuania                  154
    Singapore                  117
    Canada                      77
    Thailand                    76
    Israel                      74
    Iceland                     71
    RSA                         65
    Korea                       63
    Brazil                      62
    West Indies                 54
    Bahrain                     42
    Nigeria                     30
    Name: Country, dtype: int64



Most of the data seems to be for the United Kingdom. To make your job easier, filter the data by *United Kingdom*.


```python
country_filter = ['United Kingdom']
retail = retail[retail.Country.isin(country_filter)]
```

Because the **Country** column only contains the same value, you can drop it.


```python
retail = retail[['StockCode','Quantity','Price']]
```


```python
retail.head()
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
      <th>StockCode</th>
      <th>Quantity</th>
      <th>Price</th>
    </tr>
    <tr>
      <th>InvoiceDate</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>85048</td>
      <td>12</td>
      <td>6.95</td>
    </tr>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>79323P</td>
      <td>12</td>
      <td>6.75</td>
    </tr>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>79323W</td>
      <td>12</td>
      <td>6.75</td>
    </tr>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>22041</td>
      <td>48</td>
      <td>2.10</td>
    </tr>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>21232</td>
      <td>24</td>
      <td>1.25</td>
    </tr>
  </tbody>
</table>
</div>



### Examining StockCode and removing anomalies

Examine the distribution of the **StockCode** column:


```python
retail.StockCode.describe()
```




    count     379423
    unique      4015
    top       85123A
    freq        3140
    Name: StockCode, dtype: object



There are 4,015 unique values for **StockCode**. A quick plot of the counts might give you some insight into how the values are distributed.


```python
retail.StockCode.value_counts().plot()
```




    <AxesSubplot:>




![png](output_56_1.png)


It seems that there are a few high-selling products, with a long tail behind them. You could investigate this situation further. However, for now, examine **Quantity**.


```python
retail.Quantity.describe()
```




    count    379423.000000
    mean         11.451517
    std          68.943709
    min       -9360.000000
    25%           2.000000
    50%           4.000000
    75%          12.000000
    max       10000.000000
    Name: Quantity, dtype: float64




```python
retail.Quantity.plot()
```




    <AxesSubplot:xlabel='InvoiceDate'>




![png](output_59_1.png)


From the initial plot, notice a couple of interesting aspects.

1. There appear to be negative quantities.

2. There are very large spikes throughout the year.


Negative and zero quantities could impact the forecast if you don't know why these values exist. To make things easier for now, you will remove negative and zero quantities


```python
retail = retail[retail.Quantity>0]
```

Now, examine **Price**.


```python
retail.Price.describe()
```




    count    370951.000000
    mean          3.145220
    std          30.551482
    min           0.000000
    25%           1.250000
    50%           1.950000
    75%           3.750000
    max       10953.500000
    Name: Price, dtype: float64




```python
retail.Price.plot()
```




    <AxesSubplot:xlabel='InvoiceDate'>




![png](output_65_1.png)


The plot shows some clear price spikes. You will now try to find out why these spikes exist.


```python
retail[retail.Price>500].head()
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
      <th>StockCode</th>
      <th>Quantity</th>
      <th>Price</th>
    </tr>
    <tr>
      <th>InvoiceDate</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2009-12-10 11:50:00</th>
      <td>M</td>
      <td>1</td>
      <td>1213.02</td>
    </tr>
    <tr>
      <th>2010-01-29 11:04:00</th>
      <td>M</td>
      <td>1</td>
      <td>8985.60</td>
    </tr>
    <tr>
      <th>2010-03-23 15:22:00</th>
      <td>M</td>
      <td>1</td>
      <td>10953.50</td>
    </tr>
    <tr>
      <th>2010-06-08 16:39:00</th>
      <td>M</td>
      <td>1</td>
      <td>849.45</td>
    </tr>
    <tr>
      <th>2010-06-11 15:54:00</th>
      <td>M</td>
      <td>1</td>
      <td>1000.63</td>
    </tr>
  </tbody>
</table>
</div>



The **StockCode** value of *M* looks unusual. If you had access to a domain expert, you could learn about the importance of *M*. Because you can't ask a domain expert for this lab, you will drop everything that has a **StockCode** value of *M*.


```python
retail = retail[retail.StockCode!='M']
```


```python
retail.Price.describe()
```




    count    370576.000000
    mean          3.009463
    std           4.576951
    min           0.000000
    25%           1.250000
    50%           1.950000
    75%           3.750000
    max         387.540000
    Name: Price, dtype: float64



This result is better, but the **max** value is still high. You will now investigate this situation further.


```python
retail[retail.Price>300].head(20)
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
      <th>StockCode</th>
      <th>Quantity</th>
      <th>Price</th>
    </tr>
    <tr>
      <th>InvoiceDate</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2010-01-26 16:29:00</th>
      <td>ADJUST</td>
      <td>1</td>
      <td>342.80</td>
    </tr>
    <tr>
      <th>2010-01-26 17:28:00</th>
      <td>ADJUST</td>
      <td>1</td>
      <td>387.54</td>
    </tr>
    <tr>
      <th>2010-06-25 14:15:00</th>
      <td>ADJUST2</td>
      <td>1</td>
      <td>300.13</td>
    </tr>
    <tr>
      <th>2010-06-25 14:15:00</th>
      <td>ADJUST2</td>
      <td>1</td>
      <td>358.47</td>
    </tr>
    <tr>
      <th>2010-08-04 11:38:00</th>
      <td>POST</td>
      <td>1</td>
      <td>334.88</td>
    </tr>
  </tbody>
</table>
</div>



It seems that some adjustments occurred. You will also drop any data that shows these adjustments.


```python
stockcodes = ['ADJUST', 'ADJUST2', 'POST']
retail = retail[~retail.StockCode.isin(stockcodes)]
```


```python
retail.Price.describe()
```




    count    370554.000000
    mean          3.002500
    std           4.363688
    min           0.000000
    25%           1.250000
    50%           1.950000
    75%           3.750000
    max         295.000000
    Name: Price, dtype: float64



You will now examine zero-priced items.


```python
retail[retail.Price==0].count
```




    <bound method DataFrame.count of                     StockCode  Quantity  Price
    InvoiceDate                                   
    2009-12-02 13:34:00     22076        12    0.0
    2009-12-03 11:19:00     48185         2    0.0
    2009-12-08 15:25:00     22065         1    0.0
    2009-12-08 15:25:00     22142        12    0.0
    2009-12-15 13:49:00     85042         8    0.0
    2009-12-18 14:22:00     21143        12    0.0
    2010-01-06 14:54:00     79320        24    0.0
    2010-01-15 12:43:00     21533        12    0.0
    2010-02-12 14:58:00   TEST001         5    0.0
    2010-02-12 15:47:00   TEST001         5    0.0
    2010-03-04 11:44:00     21662         1    0.0
    2010-04-01 17:13:00     22459         8    0.0
    2010-04-01 17:13:00     22458         8    0.0
    2010-06-11 11:12:00     21765         1    0.0
    2010-06-17 10:12:00     20914         2    0.0
    2010-06-24 12:34:00     22423         5    0.0
    2010-07-19 13:13:00     22690         6    0.0
    2010-09-27 16:59:00    46000M       648    0.0
    2010-09-30 12:19:00     22218         2    0.0
    2010-10-18 15:13:00     22121         1    0.0
    2010-11-07 14:26:00     21843         2    0.0>



There aren't many values in these results, so you can drop zero-priced items.


```python
retail = retail[retail.Price>0]
```

### Splitting the data

The timeseries data that you need to create a forecast requires a *timestamp*, an *itemId*, and a *demand*. These features will map to the **InvoiceDate**, **StockCode**, and **Quantity** columns.

The related timeseries data needs a *timestamp*, an *itemId*, and a *price*. These features will map to the **InvoiceDate**, **StockCode**, and **Price** columns.

Create the two DataFrames:


```python
df_time_series = retail[['StockCode','Quantity']]
df_related_time_series = retail[['StockCode','Price']]
```

### Downsampling

You will now examine a single item.


```python
df_time_series[df_time_series.StockCode==21232]['2009-12-01']
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
      <th>StockCode</th>
      <th>Quantity</th>
    </tr>
    <tr>
      <th>InvoiceDate</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>21232</td>
      <td>24</td>
    </tr>
    <tr>
      <th>2009-12-01 10:49:00</th>
      <td>21232</td>
      <td>48</td>
    </tr>
    <tr>
      <th>2009-12-01 12:13:00</th>
      <td>21232</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2009-12-01 12:14:00</th>
      <td>21232</td>
      <td>20</td>
    </tr>
    <tr>
      <th>2009-12-01 13:31:00</th>
      <td>21232</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2009-12-01 13:37:00</th>
      <td>21232</td>
      <td>12</td>
    </tr>
    <tr>
      <th>2009-12-01 13:43:00</th>
      <td>21232</td>
      <td>24</td>
    </tr>
    <tr>
      <th>2009-12-01 14:19:00</th>
      <td>21232</td>
      <td>12</td>
    </tr>
    <tr>
      <th>2009-12-01 15:26:00</th>
      <td>21232</td>
      <td>12</td>
    </tr>
    <tr>
      <th>2009-12-01 16:18:00</th>
      <td>21232</td>
      <td>12</td>
    </tr>
  </tbody>
</table>
</div>



You can see multiple orders for each day. You want to create a forecast that predicts demand at a daily level.

You must *downsample* the data from the individual orders into a daily total.

The orders for each day can be summed, because the total demand for the day is the value that you will forecast.

pandas provides the `resample` function for this purpose. `sum` will sum the **Quantity** column. You will also reset the index based on the **InvoiceDate** value. However, this time, it will be a date without the time portion.

**Note:** It might take up to 1 minute for this process to complete.


```python
df_time_series = df_time_series.groupby('StockCode').resample('D').sum().reset_index()
```


```python
df_time_series['InvoiceDate'] = pd.to_datetime(df_time_series.InvoiceDate)
df_time_series = df_time_series.set_index('InvoiceDate')
df_time_series.head()
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
      <th>StockCode</th>
      <th>Quantity</th>
    </tr>
    <tr>
      <th>InvoiceDate</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2009-12-01</th>
      <td>10002</td>
      <td>12</td>
    </tr>
    <tr>
      <th>2009-12-02</th>
      <td>10002</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2009-12-03</th>
      <td>10002</td>
      <td>7</td>
    </tr>
    <tr>
      <th>2009-12-04</th>
      <td>10002</td>
      <td>25</td>
    </tr>
    <tr>
      <th>2009-12-05</th>
      <td>10002</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_time_series = df_time_series.groupby('StockCode').resample('D').sum().reset_index().set_index(['InvoiceDate'])
```

Examine the new DataFrame.


```python
df_time_series[df_time_series.StockCode==21232]

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
      <th>StockCode</th>
      <th>Quantity</th>
    </tr>
    <tr>
      <th>InvoiceDate</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2009-12-01</th>
      <td>21232</td>
      <td>171</td>
    </tr>
    <tr>
      <th>2009-12-02</th>
      <td>21232</td>
      <td>164</td>
    </tr>
    <tr>
      <th>2009-12-03</th>
      <td>21232</td>
      <td>192</td>
    </tr>
    <tr>
      <th>2009-12-04</th>
      <td>21232</td>
      <td>264</td>
    </tr>
    <tr>
      <th>2009-12-05</th>
      <td>21232</td>
      <td>36</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2010-12-04</th>
      <td>21232</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2010-12-05</th>
      <td>21232</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2010-12-06</th>
      <td>21232</td>
      <td>12</td>
    </tr>
    <tr>
      <th>2010-12-07</th>
      <td>21232</td>
      <td>28</td>
    </tr>
    <tr>
      <th>2010-12-08</th>
      <td>21232</td>
      <td>61</td>
    </tr>
  </tbody>
</table>
<p>373 rows × 2 columns</p>
</div>



The order now has a single entry for each day.

Repeat this process with the related time series data.


```python
df_related_time_series.head()
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
      <th>StockCode</th>
      <th>Price</th>
    </tr>
    <tr>
      <th>InvoiceDate</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>85048</td>
      <td>6.95</td>
    </tr>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>79323P</td>
      <td>6.75</td>
    </tr>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>79323W</td>
      <td>6.75</td>
    </tr>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>22041</td>
      <td>2.10</td>
    </tr>
    <tr>
      <th>2009-12-01 07:45:00</th>
      <td>21232</td>
      <td>1.25</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_related_time_series2 = df_related_time_series.groupby('StockCode').resample('D').mean().reset_index().set_index(['InvoiceDate','StockCode'])
```


```python
df_related_time_series2.head(20)
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
      <th></th>
      <th>Price</th>
    </tr>
    <tr>
      <th>InvoiceDate</th>
      <th>StockCode</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2009-12-01</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-02</th>
      <th>10002</th>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2009-12-03</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-04</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-05</th>
      <th>10002</th>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2009-12-06</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-07</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-08</th>
      <th>10002</th>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2009-12-09</th>
      <th>10002</th>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2009-12-10</th>
      <th>10002</th>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2009-12-11</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-12</th>
      <th>10002</th>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2009-12-13</th>
      <th>10002</th>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2009-12-14</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-15</th>
      <th>10002</th>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2009-12-16</th>
      <th>10002</th>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2009-12-17</th>
      <th>10002</th>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2009-12-18</th>
      <th>10002</th>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2009-12-19</th>
      <th>10002</th>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2009-12-20</th>
      <th>10002</th>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



**Question:** Why are some of the previous values showing as *NaN*?

**Answer:** That product had no orders for those days, and thus it has no price. Should you fill these NaN values with a numerical value?


```python
retail[retail.StockCode == 10002]['2009-12']
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
      <th>StockCode</th>
      <th>Quantity</th>
      <th>Price</th>
    </tr>
    <tr>
      <th>InvoiceDate</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2009-12-01 09:08:00</th>
      <td>10002</td>
      <td>12</td>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-03 13:49:00</th>
      <td>10002</td>
      <td>1</td>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-03 13:49:00</th>
      <td>10002</td>
      <td>1</td>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-03 19:13:00</th>
      <td>10002</td>
      <td>1</td>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-03 20:03:00</th>
      <td>10002</td>
      <td>4</td>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-04 08:46:00</th>
      <td>10002</td>
      <td>12</td>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-04 12:20:00</th>
      <td>10002</td>
      <td>12</td>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-04 17:31:00</th>
      <td>10002</td>
      <td>1</td>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-06 15:24:00</th>
      <td>10002</td>
      <td>1</td>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-07 16:40:00</th>
      <td>10002</td>
      <td>2</td>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-11 12:21:00</th>
      <td>10002</td>
      <td>9</td>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-14 12:02:00</th>
      <td>10002</td>
      <td>12</td>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-14 14:12:00</th>
      <td>10002</td>
      <td>24</td>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-21 13:29:00</th>
      <td>10002</td>
      <td>12</td>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-23 12:07:00</th>
      <td>10002</td>
      <td>1</td>
      <td>0.85</td>
    </tr>
  </tbody>
</table>
</div>



You can use `pad` to forward-fill the price. The previous value will be used to fill the gap for each missing value. 


```python
df_related_time_series3 = df_related_time_series2.groupby('StockCode').pad()
```


```python
df_related_time_series3.head(20)
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
      <th></th>
      <th>Price</th>
    </tr>
    <tr>
      <th>InvoiceDate</th>
      <th>StockCode</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2009-12-01</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-02</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-03</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-04</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-05</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-06</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-07</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-08</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-09</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-10</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-11</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-12</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-13</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-14</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-15</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-16</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-17</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-18</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-19</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>2009-12-20</th>
      <th>10002</th>
      <td>0.85</td>
    </tr>
  </tbody>
</table>
</div>



## Task 4: Reviewing the creation of the forecast

The following cells are Markdown. They demonstrate the API calls that are needed to create a forecast based on the data that you have been working with. Creating a forecast with Amazon Forecast involves three stages:

1. Creating the datasets and importing the data. This process typically takes 5–10 minutes.
2. Creating the predictor. This process trains a model by using the data that you provided. It takes 30–60 minutes to complete.
3. Creating the forecast. This process generates a forecast for a particular item by using the predictor. It also takes 30–60 minutes to complete.

To save time, when this lab was started, the `forecast-autorun.ipynb` was also ran in the background. The notebook will be updated with the results after running completes. It takes about 65 minutes to run, but it might take a little longer. By the time you review this cell, the forecast creation should in process. While it's finishing, you will review the code.

**Note:** Feel free to review the actual `forecast-autorun.ipynb` notebook if you want some more detail. However, make sure that you don't run any cells!

### Creating the datasets and importing the data

The first step is to create a Forecast Dataset Group:

```python
session = boto3.Session()
forecast = session.client(service_name='forecast') 
create_dataset_group_response = forecast.create_dataset_group(DatasetGroupName=dataset_group_name, Domain="RETAIL")
dataset_group_arn = create_dataset_group_response['DatasetGroupArn']
```
    
The `create_dataset` function requires a few parameters:

- **DOMAIN** – This parameter specifies the domain, such as *retail*, that the forecast should use.
- **DatasetType** – For the time series data, this parameter will be set to *TARGET_TIME_SERIES*.
- **DatasetName** – This parameter specifies the name of the dataset.
- **DataFrequency** – This parameter specifices the frequency. For the daily dataset, it will be *D*.
- **Schema** – This parameter specifies the schema of the dataset.

The dataset schema for the time series data is:

```python
schema ={
   "Attributes":[
      {
         "AttributeName":"timestamp",
         "AttributeType":"timestamp"
      },
      {
         "AttributeName":"item_id",
         "AttributeType":"string"
      },
      {
         "AttributeName":"demand",
         "AttributeType":"float"
      }
   ]
}
```


The code to create the dataset is:

```python
time_series_response=forecast.create_dataset(
                    Domain="RETAIL",
                    DatasetType='TARGET_TIME_SERIES',
                    DatasetName='retail_time_series_data',
                    DataFrequency='D', 
                    Schema = schema
)
dataset_arn = time_series_response['DatasetArn']
```
    
Now that the dataset is defined, a job is needed to import the data:

```python
ds_import_job_response=forecast.create_dataset_import_job(DatasetImportJobName='retail_import_job',
                                                      DatasetArn=dataset_arn,
                                                      DataSource= data_source,
                                                      TimestampFormat=timestamp_format
                                                     )
```

Note that the *data_source* is a path to the data that's stored in Amazon Simple Storage Service (Amazon S3).

The final step is to add the dataset to the dataset group:

```python
forecast.update_dataset_group(DatasetGroupArn=dataset_group_arn, DatasetArns=[dataset_arn])
```
    

The process of adding the related data or metadata is done in the same way: by  changing the names, schema, and dataset type. Although you have prepared this data, you won't use it in the predictor because the model wasn't impacted by the additional data.

### Creating the predictor

The next step is to create the predictor. The `create_predictor` command needs a few parameters:

- **PredictorName** – This parameter specifies the name that you want to give the predictor.

    ```python
    predictor_name= prefix+'_deeparp_algo'
    ```


- **AlgorithmArn** – This parameter is the path to the algorithm that you want to use. In this example, you will use DeepAR+.

    ```python
    algorithm_arn = 'arn:aws:forecast:::algorithm/Deep_AR_Plus
    ```


- **EvaluationParameters** – This parameter enables you to specify the number and size of the back test windows. Recall from the module that this parameter controls the size and number of testing windows that are created from the data.

    ```python
    evaluation_parameters= {"NumberOfBacktestWindows": 1, "BackTestWindowOffset": 30}
    ```


- **ForecastHorizon** – How many units to forecast (in this case, the units are days).

    ```python
    forecast_horizon = 30
    ```


- **InputDataConfig** – This parameter specifies the data, along with optional vacation days.

    ```python
    input_data_config = {"DatasetGroupArn": dataset_group_arn, "SupplementaryFeatures": [ {"Name": "holiday","Value": "UK"} ]}
    ```


- **FeaturizationConfig** – This parameter sets the frequency, but it can also be used to specify filling methods for data.

    ```python
    featurization_config= {"ForecastFrequency": dataset_frequency }
    ```

The code to create the predictor is:

```python
create_predictor_response=forecast.create_predictor(PredictorName = predictor_name,
      AlgorithmArn = algorithm_arn,
      ForecastHorizon = forecast_horizon,
      PerformAutoML = False,
      PerformHPO = False,
      EvaluationParameters= evaluation_parameters, 
      InputDataConfig = input_data_config,
      FeaturizationConfig = featurization_config
     )
```
                                                 
After the predictor is created, you can create a forecast.

### Creating the forecast

To create the forecast, use the `create_forecast` method:

```python
predictor_arn = create_predictor_response['PredictorArn']

create_forecast_response=forecast.create_forecast(ForecastName=forecast_Name,
                                                  PredictorArn=predictor_arn)

```

After the forecast is generated, the results can be queried by using the `query_forecast` method:

```python
forecast_response = forecast_query.query_forecast(
    ForecastArn=forecast_arn,
    Filters={"item_id":"22423"}
)
```


## Task 5: Waiting for the forecast creation to complete

The forecast should now be created. You can investigate to see whether the forecast creation is complete.

First, create a helper method to show the status.


```python
import sys

class StatusIndicator:
    
    def __init__(self):
        self.previous_status = None
        self.need_newline = False
        
    def update( self, status ):
        if self.previous_status != status:
            if self.need_newline:
                sys.stdout.write("\n")
            sys.stdout.write( status + " ")
            self.need_newline = True
            self.previous_status = status
        else:
            # sys.stdout.write(".")
            print('.',end='')
            self.need_newline = True
        sys.stdout.flush()

    def end(self):
        if self.need_newline:
            sys.stdout.write("\n")
```

Next, create instances of the forecast and the forecast query objects.


```python
bucket='mlf-lab4-forecastbucket-12sb9sjex9iv'

session = boto3.Session() 
forecast = session.client(service_name='forecast') 
forecast_query = session.client(service_name='forecastquery')
```

You will read the variables from the store, and check whether the forecast was defined. After the forecast is defined, you will wait until its status becomes active.


```python
print('Waiting for the predictor arn to be available')
while True:
    %store -r
    is_local = "forecast_arn" in locals()
    if is_local: break
    print('.',end='')
    time.sleep(10)

print('Waiting for the predictor to be available')
status_indicator_predictor = StatusIndicator()
while True:
    status = forecast.describe_predictor(PredictorArn=predictor_arn)['Status']
    status_indicator_predictor.update(status)
    if status in ('ACTIVE', 'CREATE_FAILED'): break
    time.sleep(10)

status_indicator_predictor.end()
    
print('Waiting for forecast to be available')
status_indicator = StatusIndicator()
while True:
    status = forecast.describe_forecast(ForecastArn=forecast_arn)['Status']
    status_indicator.update(status)
    if status in ('ACTIVE', 'CREATE_FAILED'): break
    time.sleep(10)

status_indicator.end()
```

## Task 6: Using the forecast

At this point, there should be a forecast that's ready to be queried.

Check that you get data for the following test stock code: *21232*

print()
forecast_response = forecast_query.query_forecast(
    ForecastArn=forecast_arn,
    Filters={"item_id":"21232"}
)
print(forecast_response)

### Plotting the actual results

Earlier, you split the data and held back the *November* and *December* values. You will plot these values against the predicted values for the same time period.

You will start by reading the test values back into a DataFrame.



```python
actual_df = pd.read_csv(test, names=['InvoiceDate','StockCode','Quantity'])
actual_df['InvoiceDate'] = pd.to_datetime(actual_df.InvoiceDate)
actual_df = actual_df.set_index('InvoiceDate')
actual_df.head()
```

Check that you only have data for the *21232* stock code.


```python
stockcode_filter = ['21232']
actual_df = actual_df[actual_df['StockCode'].isin(stockcode_filter)]
```


```python
actual_df.head()
```

You can do a quick plot of the data. Remember that this data is test data, so the actual values are plotted. In the next step, you will plot the predicted values.


```python
actual_df.Quantity.plot()
```

### Plotting the prediction

Next, you must convert the JSON response from the predictor to a DataFrame that you can plot.

Start by getting the P10 predictions.



```python
# Generate DF 
prediction_df_p10 = pd.DataFrame.from_dict(forecast_response['Forecast']['Predictions']['p10'])
prediction_df_p10.head()
```

Next, plot the P10 predictions.


```python
# Plot
prediction_df_p10.plot()

```

The previous code only retrieved the P10 values and put them in a DataFrame. Now, complete the same process for the P50 and P90 values.



```python
prediction_df_p50 = pd.DataFrame.from_dict(forecast_response['Forecast']['Predictions']['p50'])
prediction_df_p90 = pd.DataFrame.from_dict(forecast_response['Forecast']['Predictions']['p90'])
```


### Comparing the prediction to actual results

After you obtain the DataFrames, the next task is to plot them together to determine the best fit.



```python
# Start by creating a DataFrame to house the content. Here, Source will be which DataFrame it came from.
results_df = pd.DataFrame(columns=['timestamp','value','Source'])

results_df.head()
```



Import the observed values into the DataFrame:



```python
import dateutil.parser
for index, row in actual_df.iterrows():
    #clean_timestamp = dateutil.parser.parse(index)
    results_df = results_df.append({'timestamp' : index , 'value' : row['Quantity'], 'Source': 'Actual'} , ignore_index=True)
```


```python
# To show the new DataFrame
results_df.head()
```


```python
# Now add the P10, P50, and P90 Values
for index, row in prediction_df_p10.iterrows():
    clean_timestamp = dateutil.parser.parse(row['Timestamp'])
    results_df = results_df.append({'timestamp' : clean_timestamp , 'value' : row['Value'], 'Source': 'p10'} , ignore_index=True)
for index, row in prediction_df_p50.iterrows():
    clean_timestamp = dateutil.parser.parse(row['Timestamp'])
    results_df = results_df.append({'timestamp' : clean_timestamp , 'value' : row['Value'], 'Source': 'p50'} , ignore_index=True)
for index, row in prediction_df_p90.iterrows():
    clean_timestamp = dateutil.parser.parse(row['Timestamp'])
    results_df = results_df.append({'timestamp' : clean_timestamp , 'value' : row['Value'], 'Source': 'p90'} , ignore_index=True)
```

By creating a pivot on the data, you can compare the actual P10, P50, and P90 values.


```python
pivot_df = results_df.pivot(columns='Source', values='value', index="timestamp")
pivot_df
```

Charts can be easier to analyze than the raw values.


```python
pivot_df.plot(figsize=(20,10))
```

### Examining the results

Hopefully, in the previous chart, you will see at least some correlation between the predicted values and the actual values. The correlation might not be good, and there could be several reasons for this outcome:

- The sales are mostly wholesale, but they do include some smaller orders.
- You held back data, which meant that an entire season wasn't included in the training data.
- You might have been missing useful category or sales promotion data.

Like all machine learning models, the results are as good as the data you use to train the model. As noted previously, the model could be improved with more data.

## Task 7: Cleaning up

The following cells will clean up the resources that were created during the lab.


```python
%store -r
```


```python
print(forecast_arn)
```


```python
forecast.delete_forecast(ForecastArn=forecast_arn)
time.sleep(60)
```


```python
forecast.delete_predictor(PredictorArn=predictor_arn)
time.sleep(60)
```


```python
forecast.delete_dataset_import_job(DatasetImportJobArn=ds_related_import_job_arn)
```


```python
forecast.delete_dataset_import_job(DatasetImportJobArn=ds_import_job_arn)
```


```python
time.sleep(60)
```


```python
forecast.delete_dataset(DatasetArn=related_dataset_arn)
```


```python
forecast.delete_dataset(DatasetArn=dataset_arn)
```


```python
time.sleep(60)
```


```python
forecast.delete_dataset_group(DatasetGroupArn=dataset_group_arn)
```
