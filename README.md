# Retail Strategy and Analytics Project

## Goal of the project
The purpose of this project is to gain insights from the yearly transactions and customer data and understand the customers who purchase chips and their purchase behaviour from a  <b>chain of retail stores</b>. The insights from this analysis will help the supermarket's strategic plan for category review for upcoming years. The analysis will help in determining which customers segments should be targeted by the defining metrics which drives the business - total sales, store performance over the year and other drivers of sales. A strategic recommendation (supported by data) is presented in form of a Tableau Story which can be instrumental in upcoming category review.


## Analysis Approach
### 1. Data Quality Assessment and Data Cleaning
The first step towards generating useful insights from the data was the data prepartion, quality assessment and data cleaning step. After the cleaning process exploratory data analysis on client's transaction dataset and identify customer purchasing behaviours to generate insights and provide commercial recommendations.

In the data cleaning step the data quality of the datasets was firstly assesed. After a data quality assessment the following data quality issues was observed and the necessary process to mitigate the issue was followed :
- The Date column in the  dataset was in integer format ie. number of days from Dec 30, 1985. Hence this column was converted to datetime format taking Dec 30, 1985 as a referennce date.
- Removed outlier records based on the PROD_QTY (quantity purchased of a particular product) column.
- The analysis concentrated on Chips products. Hence the "PROD_NAME" (Product Name) column was split and frequency of each word was counted then all rows containing "salsa" in "PROD_NAME" was removed. 
- A new feature namely Brand was created by extracting the brand name from the product name "PROD_NAME"
- Checked whether there are duplicate records present in the dataset. 

### 2. Exploratory Data Analysis on Customer Segments
After the data cleaning process, we defined mertics of interest that will drive metrics of interest that will drive our analysis :
- <b>Customer Segments who spend the most on chips, described based on their lifestage and the premium tier they belong</b><br> 
Sales are coming mainly from Budget - older families ($ 168,363), Mainstream - young singles/couples ($ 157,622) and Mainstream- retirees ($ 155,677) which contributes to 25% of the total sales of $ 1,933 K.
<img src="data%20visualization/Sales%20across%20Customer%20Segments.png" height="700" align="middle">

- <b>Number of Customers in each Customer Segment</b><br>
Young Singles/Couples (Mainstream) has the highest population, followed by Retirees (Mainstream) which explains their high total sales. There are more Mainstream - young singles/couples and Mainstream - retirees who buy chips. This contributes to there being more sales to these customer segments but this is not a major driver for the Budget - Older families segment. However amount of chips bought per customer can be a determining factor for higher sales.
<img src="data%20visualization/Number%20of%20Shoppers.png" height="700" align="middle">

- <b>Amount of chips bought per Customer Segment</b><br>
Affluence appears consistent across each individual life stage profile; Older and Young Family shoppers purchase the highest average units of chips pack per transaction.
<img src="data%20visualization/Average%20Units%20of%20chips%20purchased%20per%20Transaction.png" height="700" align="middle">

- <b>Average price of pack bought per unit purchase of chips across Customer Segments</b><br>
Mainstream midage and young singles and couples are more willing to pay more per packet of chips compared to their budget and premium counterparts. This may be due to premium shoppers being more likely to buy healthy snacks and when they buy chips, this is mainly for entertainment purposes rather than their own consumption. This is also supported by there being fewer premium midage and young singles and couples buying chips compared to their mainstream counterparts.
<img src="data%20visualization/Average%20Price%20of%20Chips%20per%20unit%20of%20purchase.png" height="700" align="middle">
The difference in average price of chips pack isn't quite large across the customer segments. Hence performed <b>Two Sample t-test</b> to find out whether this difference in average price is statistically different. <br>
The t-test results in a p-value came to be < 2.2e-16. Hence it means that the average price for mainstream young and midage singles and couples are significantly higher than that of budget or premium young and midage singles and couples.<br>
<br>
- <b>Brand and Pack Size preferred across Customer Segments</b><br>
Chips brand Kettle is dominating every segment as the most purchased brand.Most frequent chip size purchased is 175gr followed by the 150gr chip size for all segments.
<img src="data%20visualization/Brand%20and%20Pack%20size%20Popularity%20across%20segments.png" height="700" align="middle">


### 3. Experimentation and Uplift testing
- Recomendations

## Database Model
The project comprises of a redshift postgres database in the cluster with staging tables that contain all the data retrieved from the s3 bucket and copied over to the tables. It also contains a fact table `fact_city_data_table` and three dimensional tables namely `dim_airport_table`, `dim_demographic_table` and `dim_visitor_table`. The data model representation for the fact and dimension tables is as below:

### City Fact Table

| Table Column | Data Type | Description |
| -------- | ------------- | --------- |
| city_id (PRIMARY_KEY) | varchar(32)  | auto generated primary key|
|  city_name | varchar(32) | name of city |
| country | varchar(32) | name of country |
| latitude | varchar(10) | latitude value |
| longitude | varchar(10) | longitude value |
| average_temperature | numeric |  average temperature of the city |
| date | date | date of temperature recording |

### Airport Dimension Table

| Table Column | Data Type | Description |
| ------------ | ---------- | --------- |
| airport_id (PRIMARY_KEY) | varchar(50)  | auto generated primary key|
|  airport_code | varchar(50) | airport short code |
| name | varchar(500) | name of airport |
| continent | varchar(50) | continent code |
| country_code | varchar(32) | country code |
| state | varchar(32) |  state code |

### Visitor Dimension Table

| Table Column             | Data Type     | Description |
| -------------------------| ------------- | ----------- |
| visitor_id (PRIMARY_KEY) | varchar(32)   | auto generated primary key|
|  year                    | int4          | year of visit |
| month                    | int4          | month of visit|
| city                     | varchar(32)   | city of visit |
| gender                   | varchar(32)   | gender of visitor |
| arrival_date             | date          |  arrival date of the visitor |
| departure_date           | date          |  departure time of the visitor |
| airline                  | varchar(32)   |  airline code |

### Demographic Dimension Table

| Table Column | Data Type | Description |
| ---------------------------- | ------------- | ------------------------- |
| demographic_id (PRIMARY_KEY) | varchar(100)  | auto generated primary key|
|  city                        | varchar(50)   | city name                 |
| state                        | varchar(50)   | state code                |
| male_population              | int4          | male population numbers by city |
| female_population | int4 | female population numbers by city |
| total_population | int4 |  total population numbers by city |


## Tools and Technologies used
The tools used in this project include:
- __Apache Spark__ - This was needed to process data from the big data SAS and csv files to dataframes and convert them to the more readable json data files. In the process, it maps the columns from the datasets to the relevant columns required for the staging tables and also maps some of the values such as the city codes from the provided `label descriptions` data dictionary to city names in the data generated by spark. To view how this is done, check out this [python helper file](additional_helpers/data_to_json.py) and the data dictionary for [city codes](data/label_descriptions/city_codes.json).
- __Apache Airflow__ - This was required to automate workflows, namely uploading processed json files from the local filesystem to s3, creating the staging, fact and dimension tables, copying the s3 files to the redshift staging table then performing ETL to load the final data to the fact and dimension tables. This is all done by pre-configured dags, the [file upload dag](dags/file_upload_dag.py) and the [capstone dag](dags/capstone_dag.py) that both perform a series of tasks.
- __Amazon Redshift__ - The database is located in a redshift cluster that store the data from s3 and the eventual data that gets added to the fact and dimension tables.
- __Amazon S3__ - Stores the json files generated by spark that are uploaded from the local filesystem.

## Datasets Used
The datasets used and sources include:
- __I94 Immigration Data__: This data is retrieved from the US National Tourism and Trade Office and the source can be found [here](https://travel.trade.gov/research/reports/i94/historical/2016.html).
- __World Temperature Data__: This dataset came from Kaggle and you can find out about it [here](https://www.kaggle.com/berkeleyearth/climate-change-earth-surface-temperature-data).
- __U.S. City Demographic Data__: This data comes from OpenSoft [here](https://public.opendatasoft.com/explore/dataset/us-cities-demographics/export/).
- __Airport Code Table__: This is a simple table of airport codes and corresponding cities that is retrieved from [here](https://datahub.io/core/airport-codes#data)

## Built With
- Python 2.7, Tableau

## Authors
- Abhishek Chowdhury - [Github Profile](https://github.com/AbhishekGit-hash)

