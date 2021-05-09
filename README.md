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

### 2 Exploratory Data Analysis on Customer Segments
After the data cleaning process, we defined mertics of interest that will drive metrics of interest that will drive our analysis :
- Customer Segments who spend the most on chips, described based on their lifestage and the premium tier they belong to. 
Sales are coming mainly from Budget - older families ($ 168,363), Mainstream - young singles/couples ($ 157,622) and Mainstream- retirees ($ 155,677) which contributes to 25% of the total sales of $ 1,933 K.
Chart

- Young Singles/Couples (Mainstream) has the highest population, followed by Retirees (Mainstream). Which explains their high total sales.
There are more Mainstream - young singles/couples and Mainstream - retirees who buy chips. This contributes to there being more sales to these customer segments but this is not a major driver for the Budget - Older families segment.
Higher sales may also be driven by more units of chips being bought per customer. Let’s have a look at this next
Chart

- Affluence appears consistent across each individual life stage profile; Older and Young Family shoppers purchase the highest avg units per transaction
- Price per unit purchase
Mainstream midage and young singles and couples are more willing to pay more per packet of chips compared to their budget and premium counterparts. This may be due to premium shoppers being more likely to buy healthy snacks and when they buy chips, this is mainly for entertainment purposes rather than their own consumption. This is also supported by there being fewer premium midage and young singles and couples buying chips compared to their mainstream counterparts.

Chart

- Chips brand Kettle is dominating every segment as the most purchased brand.Most frequent chip size purchased is 175gr followed by the 150gr chip size for all segments.


t-test

### 3 Deep Drive into Customer Segments for Insights
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

### DAGs Airflow Graph View Representation
#### File Upload DAG
![capstone_dag](img/file_upload_dag.png "File Upload DAG")
#### Capstone DAG
![capstone_dag](img/capstone_dag.png "Capstone DAG")


## Workflow/Process
1. A sample size of the json data files that were processed by spark are already part of this project's files. You can find them in `data/processed-data`. If you would wish to generate them from scratch, you can follow the same step by step process I went through [here](#Steps-to-generate-the-s3-bound-json-data-files)
2. On the terminal, ensure your virtualenv is set then add the AIRFLOW_HOME environment variable.
    ```
    $ export AIRFLOW_HOME=~/path/to/project
    ```
3. Run `airflow initdb` to setup the airflow db locally the run both the `airflow scheduler` and `airflow webserver` commands on seperate terminal windows. If you get a psycopg2 related error, you might need to check if the postgresql service is running on your computer.
4. Once you run the above commands successfully you can open the airflow UI on localhost using port 8080 http://0.0.0.0:8080
5. Navigate to your AWS s3 console and create a bucket named `udend-data`. If you wish to provide another name, ensure you set it in the `capstone_dag` and `file_upload_dag` operator configs for s3_bucket.
6. Create a Redshift cluster with a redshift database. Once it's finished creating, take note of the endpoint and database credentials.
7. Add your AWS and Redshift credentials in the airflow UI. You can accomplish this in the following steps:
    - Click on the __Admin tab__ and select __Connections__.
    - Under __Connections__, select __Create__.
    - In the Create tab enter the following creds:
        - Conn Id: `aws_credentials`
        - Conn Type: `Amazon Web Services`
        - Login: Your `<AWS Access Key ID>`
        - Password: `<Your AWS Secret Access Key>`
    - Once done, click on __Save and Add Another__
    - On the new create page add the following:
        - Conn Id: `redshift`
        - Conn Type: `Postgres`
        - Host: `<Endpointof your redshift cluster>`
        - Schema: `<Redshift database name>`
        - Login: `<Database username>`
        - Password: `<Database password>`
        - Port: `<Database port which is usually 5439>`
    - Click save
8. Trigger the `file_upload_dag` first. This will upload the files to your s3 bucket. You can view the status of the dag tasks in the `Graph View` of the dag.
9. Once the files are uploaded, you can trigger the `capstone_dag`that will create the necessary tables on redshift and load data to them, as well as perform data quality checks.


## Steps to generate the s3 bound json data files
1. Pull the immigration files in SAS format from the udacity workspace or from the source then in the `data` directory, create a new directory named `immigration-data` where you will store them.
2. Retrieve the `GlobalLandTemperaturesByCity.csv`file from the udacity workspace and create a `temperature-data` subdirectory within the `data` directory and save the file there. The other data files, namely `us-cities-demographics.csv` and `airport-codes.csv` are already provided in this project in the `data` folder.
3. Navigate to the `additional_helpers` directory and run the following command:
    ```
    $ python data_to_json.py
    ```
 This will create a new set of json data files in `data/processed-data`. __NOTE__ This may take a while due to the nature of the size of some of the datasets.


## Possible Scenarios that may arise and how they can be handled.
- If the data gets increased by 100x:
    - The increase of reads and writes to the database can be handled by increasing the number of compute nodes being used in the redshift cluster using elastic resize that can handle for more storage. 
    - Use of distkeys in case of a need to join the tables.
    - Compress the s3 data.
- If the pipelines were needed to be run on a daily basis by 7am:
    - dags can be scheduled to run daily by setting the start_date config as a datetime value containing both the date and time when it should start running, then setting schedule_interval to @daily which will ensure the dag runs everyday at the time provided in start_date.
- If the database needed to be accessed by 100+ people:
    - Utilizing elastic resize for better performance.
    - Utilizing Concurrency Scaling on Redshift by setting it to auto and allocating it's usage to specific user groups and workloads. This will boost query processing for an increasing amount of users.

## Built With
- Python 2.7, Tableau

## Authors
- Abhishek Chowdhury - [Github Profile](https://github.com/AbhishekGit-hash)

