# AWS + Snowflake + Power BI Project

## Project Overview
This project demonstrates the integration of AWS S3, Snowflake, and Power BI for data storage, processing, and visualization. The workflow involves storing data in AWS S3, loading it into Snowflake for transformation, and finally visualizing it using Power BI.

## Steps Involved

### **1. AWS S3 Setup**
- Created an **Amazon S3 bucket** (`powerbi.project0`) to store CSV files.
- Uploaded the dataset (CSV file) to the S3 bucket.
- Configured IAM roles and permissions to allow Snowflake access to the S3 bucket.
- AWS S3 Bucket URL s3://csv.test.filee/YOUTUBE+CHANNELS+DATASET.csv

### **2. Snowflake Configuration**

#### **Creating a Storage Integration Object**
A **storage integration** was created to enable Snowflake to read from S3.

```sql
CREATE OR REPLACE STORAGE INTEGRATION PBI_Integration
 TYPE = EXTERNAL_STAGE
 STORAGE_PROVIDER = 'S3'
 ENABLED = TRUE
 STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::911167898089:role/powerbi.role'
 STORAGE_ALLOWED_LOCATIONS = ('s3://powerbi.project0/')
 COMMENT = 'Integration for Power BI project';
```

#### **Creating Database and Schema**
```sql
CREATE DATABASE PowerBI;
CREATE SCHEMA PBI_Data;
```

#### **Creating Table for Data Storage**
```sql
CREATE TABLE PBI_Dataset (
  Year INT,
  Location STRING,
  Area INT,
  Rainfall FLOAT,
  Temperature FLOAT,
  Soil_type STRING,
  Irrigation STRING,
  Yeilds INT,
  Humidity FLOAT,
  Crops STRING,
  Price INT,
  Season STRING
);
```

#### **Creating a Stage for S3 Data Import**
```sql
CREATE STAGE PowerBI.PBI_Data.pbi_stage
  URL = 's3://powerbi.project0'
  STORAGE_INTEGRATION = PBI_Integration;
```

#### **Loading Data from S3 into Snowflake**
```sql
COPY INTO PBI_Dataset
FROM @pbi_stage
FILE_FORMAT = (TYPE=CSV FIELD_DELIMITER=',' SKIP_HEADER=1)
ON_ERROR = 'CONTINUE';
```

#### **Performing Data Transformations**
##### **Creating a Replica Table for Transformations**
```sql
CREATE TABLE Agriculture AS
SELECT * FROM PBI_Dataset;
```
##### **Updating Values for Rainfall and Area**
```sql
UPDATE Agriculture
SET Rainfall = 1.1 * Rainfall;

UPDATE Agriculture
SET Area = 0.9 * Area;
```
##### **Adding Year Classification**
```sql
ALTER TABLE Agriculture ADD Year_Group STRING;

UPDATE Agriculture
SET Year_Group = 'Y1' WHERE Year BETWEEN 2004 AND 2009;
UPDATE Agriculture
SET Year_Group = 'Y2' WHERE Year BETWEEN 2010 AND 2015;
UPDATE Agriculture
SET Year_Group = 'Y3' WHERE Year BETWEEN 2016 AND 2019;
```
##### **Classifying Rainfall into Groups**
```sql
ALTER TABLE Agriculture ADD Rainfall_Groups STRING;

UPDATE Agriculture
SET Rainfall_Groups = 'Low' WHERE Rainfall BETWEEN 255 AND 1200;
UPDATE Agriculture
SET Rainfall_Groups = 'Medium' WHERE Rainfall BETWEEN 1201 AND 2800;
UPDATE Agriculture
SET Rainfall_Groups = 'High' WHERE Rainfall > 2800;
```

### **3. Data Visualization in Power BI**
After data preparation in Snowflake, the dataset was imported into Power BI for visualization. Four analysis pages were created:

- **Rainfall Analysis**: Trends and distributions of rainfall levels across years and locations.
- **Temperature Analysis**: Insights into temperature variations over different seasons and locations.
- **Humidity Analysis**: Visualization of humidity trends and their impact on crops.
- **Yield Analysis**: Evaluation of crop yield trends across different seasons and weather conditions.

## **Conclusion**
This project effectively demonstrates how AWS S3, Snowflake, and Power BI can be integrated for efficient data storage, transformation, and visualization. By leveraging Snowflake's SQL capabilities, data was transformed and categorized before being visualized in Power BI for meaningful insights.

