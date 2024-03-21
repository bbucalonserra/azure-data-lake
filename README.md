# Data Engineering Project
A data engineering project involves designing, constructing, and maintaining the infrastructure necessary for the collection, storage, processing, and analysis of data. These projects often require expertise in programming languages like Python and usage of technologies like cloud-based solutions such as Azure, AWS or Google Cloud Platform. Data engineering is crucial for enabling organizations to derive insights, make data-driven decisions and build data-driven applications.

# Index
1. [Objective](#objective)
2. [Problem Definition](#problem-definition)
3. [The Project](#the-project)
   - [1. Data Search](#1-data-search)
   - [2. Data Collection](#2-data-collection)
     - [2.1 Definition of Cloud Computing System](#21-definition-of-cloud-computing-system)
     - [2.2 Storage Resources](#22-storage-resources)
   - [3. Modeling and Loading](#3-modeling-and-loading)
     - [3.1 Data Lake and Databricks Connection](#31-data-lake-and-databricks-connection)
     - [3.2 Schema Creation](#32-schema-creation)
     - [3.3 Creation of Bronze Layer Tables](#33-creation-of-bronze-layer-tables)
     - [3.4 ETL - Extract, Transform and Load (Bronze - Silver)](#34-etl---extract-transform-and-load-bronze---silver)
     - [3.5 Creation of Silver Layer Tables](#35-creation-of-silver-layer-tables)
     - [3.6 ETL - Extract, Transform and Load (Silver - Gold)](#36-etl---extract-transform-and-load-silver---gold)
     - [3.7 Creation of Gold Layer Tables](#37-creation-of-gold-layer-tables)
     - [3.8 Data Catalog](#38-data-catalog)
   - [4. Analysis](#4-analysis)
     - [4.1 Data Quality](#41-data-quality)
     - [4.2 Problem Resolution](#42-problem-resolution)
   - [5. Self Assessment](#5-self-assessment)


## Problem Definition
Education is a crucial aspect of human and social development, playing a fundamental role in creating more just and egalitarian societies. Brazil, with its diversified cultural composition and numerous ethnic groups, including indigenous communities that contribute significantly to the nation's identity, requires special consideration for indigenous education. It is important to prioritize the preservation of cultural traditions and the empowerment of native communities.


## Objective
The objective of this project is to examine and analyze different facets of education in indigenous schools in Brazil. The focus will be on existing educational data. Through this analysis, it is intended to address important issues that will provide insights into the current situation of indigenous education in the country and highlight areas that require attention for improvements. Throughout this investigation, we will seek to answer the following crucial questions:

- Where are the schools located in indigenous lands?
- What is the dropout rate in indigenous schools and how does it compare to non-indigenous schools?
- What is the average number of technological equipment per state in schools with indigenous education?
- What percentage of schools in indigenous areas have internet access, segmented by state?
- In which language are subjects taught in indigenous schools, and how does this relate to the preservation of tribal mother tongues?
- Does the presence of computers in indigenous schools have any influence on the dropout rate?

These questions are fundamental to understanding the reality of indigenous education in Brazil and to directing efforts and policies aimed at improving this important area of education.

## The Project
### 1. Data Search
The data search was carried out through the brazilian government website (https://www.gov.br/inep/pt-br/acesso-a-informacao), where tables related to the school census of Brazil in 2022 were sought. Two tables were found to be part of the analysis:
- [Microdata from the School Census of Education in Brazil 2022](https://docs.google.com/spreadsheets/d/1er1GJqNIxP_sw-0k_DF3SqBpDyo0XOLU/edit?usp=sharing&ouid=105765042535713670711&rtpof=true&sd=true)
- [School Dropout Rate 2022](https://docs.google.com/spreadsheets/d/1g02tYUN2sTgARp9_JTlF7RUMj3RhHHaZ/edit?usp=sharing&ouid=105765042535713670711&rtpof=true&sd=true)

*Note: the website is in portugese (PT-BR).*

The first table refers to various information about schools in Brazil (school code, internet availability, classrooms, etc.), with 385 columns. The second one relates to the school performance rate (grades, dropout rate, failure rate, etc.), with 12 columns. **The datasets are not stored on GitHub but on Google Drive due to the size of the files.**


### 2. Data Collection
Data collection was carried out by downloading two tables directly from the official government website. This step was crucial to ensure that the data used is accurate and reliable since government sources are recognized as a primary source of educational information. The decision to obtain the data directly from the government website was made to ensure that the collected information aligns with the official sources available.

#### 2.1 Definition of Cloud Computing System
The choice of `Azure` <img align="center" src="https://github.com/bbucalonserra/data_engineering/blob/main/pictures/azure_logo.png" alt="drawing" width="40"/> as the cloud computing system for this project was based on its global reputation as one of the leading cloud platforms in the world. Additionally, Azure is known for its excellent integration with widely used tools such as Power BI, facilitating data analysis. The Azure platform also stands out for its robust security and scalability, ensuring data protection and flexibility.

#### 2.2 Storage Resources
Initially, a free Azure account was created for academic purposes. Here, the following resources were created in order:
- `A Resource Group` <img align="center" src="https://github.com/bbucalonserra/data_engineering/blob/main/pictures/resource_group_icon.png" alt="drawing" width="40"/> a logical container that helps manage and organize related resources in Azure
- `Storage Account` <img align="center" src="https://github.com/bbucalonserra/data_engineering/blob/main/pictures/storage_account_icon.png" alt="drawing" width="40"/> an Azure resource that provides cloud storage for files, using Azure Data Lake Storage Gen2 <img align="center" src="https://github.com/bbucalonserra/data_engineering/blob/main/pictures/adls_icon.png" alt="drawing" width="40"/>
- Bronze, Silver, and Gold Containers

**In this project, it is going to be used the Medallion architecture for a data lake**. The Medallion architecture offers a structured and efficient approach to managing and processing vast amounts of data. By employing this architecture, the objective is to streamline data ingestion, storage and retrieval processes while ensuring scalability and reliability. With the Medallion architecture at the core of the data lake, it is anticipated an enhanced data accessibility, improved analytics capabilities and greater agility in extracting insights to drive informed decision-making. With this setup, the collected data was stored in the Bronze container. Subsequently, after performing ETL, the data will be stored in the Silver layer and, finally, in the Gold layer. The definition of each used layer was:
- Bronze: raw data in the format of collection, such as JSON, CSV, XLS, Parquet
- Silver: cleaned and transformed data, removing unwanted columns, special characters, and spaces
- Gold: organized data, with joins between tables if necessary, and business rules applied according to defined metrics / questions that need to be answered

<div align="center">
  <img src="https://github.com/bbucalonserra/data_engineering/blob/main/pictures/containers_lake.PNG"/>
</div>



### 3. Modeling and Loading
Data modeling is a fundamental process in the field of computer science and information management. The main purpose of data modeling is to ensure that data is organized, stored, and managed efficiently and accurately to meet the specific needs of an organization or project. **Here, Modeling and Loading are in the same topic because a Data Lake system will be directly used, storing the data by layers.** 

#### 3.1 Data Lake and Databricks Connection
Now, it is necessary to perform checks on the transformations made to the raw data. For this, the Azure Databricks feature will be used.
To create a connection between the Data Lake and Databricks, the following resources need to be created:
- `Databricks` <img align="center" src="https://github.com/bbucalonserra/data_engineering/blob/main/pictures/dbx_icon.png" alt="drawing" width="40"/> a cloud-based data analytics platform that combines big data and advanced analytics resources
- `Registration App` <img align="center" src="https://github.com/bbucalonserra/data_engineering/blob/main/pictures/app_reg_icon.jpeg" alt="drawing" width="30"/> the entity representing an application or service that wants to access resources in the Azure platform. It is an identifier for an application or service that wants to make changes in the Microsoft cloud
- `Key Vault` <img align="center" src="https://github.com/bbucalonserra/data_engineering/blob/main/pictures/key%20vault%20icon.png" alt="drawing" width="40"/> a key and secret management service

With the resources created, just go to Databricks, create a notebook, and use the following Spark code:

```py
service_credential = dbutils.secrets.get(scope="<scope>",key="<service-credential-key>")

spark.conf.set("fs.azure.account.auth.type.<storage-account>.dfs.core.windows.net", "OAuth")
spark.conf.set("fs.azure.account.oauth.provider.type.<storage-account>.dfs.core.windows.net", "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider")
spark.conf.set("fs.azure.account.oauth2.client.id.<storage-account>.dfs.core.windows.net", "<application-id>")
spark.conf.set("fs.azure.account.oauth2.client.secret.<storage-account>.dfs.core.windows.net", service_credential)
spark.conf.set("fs.azure.account.oauth2.client.endpoint.<storage-account>.dfs.core.windows.net", "https://login.microsoftonline.com/<directory-id>/oauth2/token")
```

Where:
- scope = secret scope, created in Databricks itself
- service-credential-key = credential key of Key Vault
- storage-account = Storage Account
- application-id = application ID of App Registration
- directory-id = directory ID of App Registration

Once this is done, there is a connection between Databricks and the Data Lake. Now it is possible to create tables and populate them with data from the Lake.

#### 3.2 Schema Creation
Within Databricks, for organizational bias, it will be necessary to create schemas to store the analysis tables. One schema will be created for each layer of the Data Lake. To do this, simply open a notebook and use the following SQL commands:

```py
CREATE SCHEMA bronze;

CREATE SCHEMA silver;

CREATE SCHEMA gold;
```

#### 3.3 Creation of Bronze Layer Tables
In Databricks itself, a notebook will be opened to check the data quality present in the Bronze layer. For this, the use of SPARK to read the data in CSV stored as BLOBS will be used in conjunction with the creation of views:

**Table microdata_basic_education_2022**

Table View
```py
spark.read.options(delimiter = ';', header = True).csv('abfss://bronze@basiceducation.dfs.core.windows.net/microdata_basic_education_2022/microdados_ed_basica_2022.csv').display()
```
Table Visualization
```py
spark.read.options(delimiter = ';', header = True).csv('abfss://bronze@basiceducation.dfs.core.windows.net/microdata_basic_education_2022/microdata_basic_education_2022.csv').createOrReplaceTempView('microdata_basic_education_2022')
```
**Table school_retention_rate_2022**

Table View
```py
spark.read.options(delimiter = ';', header = True).csv('abfss://bronze@microdata_basic_education_2022.dfs.core.windows.net/microdados_ed_basica_2022/school_retention_rate_2022.csv').display()
```
Table Visualiation
```py
spark.read.options(delimiter = ';', header = True).csv('abfss://bronze@microdata_basic_education_2022.dfs.core.windows.net/microdata_basic_education_2022/school_retention_rate_2022.csv').createOrReplaceTempView('microdata_basic_education_2022')
```

With this, some inconsistencies in the data were observed, such as special characters and unwanted columns.
The data was stored in the BRONZE schema. For this activity, SQL commands were used:

**Table microdata_basic_education_2022**
```py
CREATE TABLE bronze.microdata_basic_education_2022 USING CSV LOCATION 'abfss://bronze@basiceducation.dfs.core.windows.net/microdata_basic_education_2022/microdata_basic_education_2022.csv'
OPTIONS (
  header = "true",
  delimiter = ";"
)
```
**Table school_retention_rate_2022**
```py
CREATE TABLE bronze.rend_escolar_2022
USING CSV LOCATION 'abfss://bronze@basiceducation.dfs.core.windows.net/microdata_basic_education_2022/school_retention_rate_2022.csv'
OPTIONS (
  header = "true",
  delimiter = ";"
)
```

Note: data types have not been defined yet because they are raw data. They will be defined in the Silver layer.


#### 3.4 ETL - Extract, Transform and Load (Bronze - Silver)
After inserting the raw data into the Bronze layer, selecting the columns, noticing some data inconsistencies, and creating the tables, the next step is to perform the transformations. For this task, the `Data Factory` resource <img align="center" src="https://github.com/bbucalonserra/data_engineering/blob/main/pictures/azure-data-factory2539.jpg" alt="drawing" width="40"/>, was used, as it is a visual and easy-to-use tool, and the required transformations are not advanced. The language used by this resource is called "Data Flow Expression Language." This language allows you to define data transformations using a syntax similar to SQL and includes functions and operators to perform transformation, filtering, projection, and much more. Below are the transformations used in Data Factory:

![ETL - Bronze para Silver](https://github.com/bbucalonserra/data_engineering/blob/main/pictures/ETL_bronze_to_silver.PNG)


Description of the transformations:
- Data collection from the Data Lake
- `SELECT` to select the columns used in the analysis
- `DERIVED COLUMN` to remove special and strange characters from the columns
- `SINK`  to send the transformed data back to the Data Lake, but now stored in the Silver layer/container


#### 3.5 Creation of Silver Layer Tables
The next step is to analyze the resulting data from the ETL process from the Bronze to Silver layer. To do this, it will be necessary to create the new tables after the ETL in Databricks already with the  **data typology defined and the variables of null or not null as well**:

**Table microdata_basic_education_2022**
```py
CREATE TABLE silver.education_basic_2022
  (YEAR_CENSUS INT NOT NULL,
  REGION_NAME STRING NOT NULL,
  STATE_NAME STRING NOT NULL,
  MESOREGION_NAME STRING NOT NULL,
  ENTITY_NAME STRING NOT NULL,
  ENTITY_CODE INTEGER NOT NULL,
  DEPENDENCY_TYPE INT,
  SCHOOL_CATEGORY_PRIVATE INT,
  LOCATION_TYPE INT,
  QTY_BASIC_STUDENTS INT,
  QTY_BASIC_STUDENTS_FEMALE INT,
  QTY_BASIC_STUDENTS_MALE INT,
  QTY_DVD_EQUIPMENT INT,
  QTY_TV_EQUIPMENT INT,
  QTY_DIGITAL_WHITEBOARD_EQUIPMENT INT,
  QTY_MULTIMEDIA_EQUIPMENT INT,
  QTY_VCR_EQUIPMENT INT,
  QTY_SATELLITE_DISH_EQUIPMENT INT,
  QTY_COPIER_EQUIPMENT INT,
  QTY_OVERHEAD_PROJECTOR_EQUIPMENT INT,
  QTY_PRINTER_EQUIPMENT INT,
  QTY_MULTIFUNCTION_PRINTER_EQUIPMENT INT,
  QTY_FAX_EQUIPMENT INT,
  QTY_PHOTO_EQUIPMENT INT,
  QTY_COMPUTER INT,
  QTY_ADMINISTRATIVE_COMPUTER INT,
  QTY_EXISTING_ROOMS INT,
  HAS_INTERNET INT,
  HAS_INDIGENOUS_EDUCATION INT,
  INDIGENOUS_LANGUAGE_TYPE INT,
  INDIGENOUS_LANGUAGE_CODE_1 INT,
  INDIGENOUS_LANGUAGE_CODE_2 INT,
  INDIGENOUS_LANGUAGE_CODE_3 INT,
  HAS_INDIGENOUS_EDUCATIONAL_MATERIAL INT)
USING CSV LOCATION 'abfss://silver@basiceducation.dfs.core.windows.net/microdata_basic_education_2022/basic_education_2022_silver'
OPTIONS (
  header = "true",
  delimiter = ","
)
```

**Table school_retention_rate_2022**
```py
CREATE TABLE silver.TX_REND_ESCOLAS_2022 
(
  YEAR INT NOT NULL, 
  REGION STRING NOT NULL,
  STATE STRING NOT NULL,
  MUNICIPALITY_CODE INT NOT NULL,
  MUNICIPALITY_NAME STRING,
  SCHOOL_CODE INT NOT NULL,
  SCHOOL_NAME STRING,
  LOCATION STRING,
  ADMINISTRATIVE_DEPENDENCY STRING,
  BASIC_EDUCATION_APPROVAL_RATE FLOAT,
  BASIC_EDUCATION_REPROVATION_RATE FLOAT,
  BASIC_EDUCATION_ABANDONMENT_RATE FLOAT
)
USING CSV LOCATION 'abfss://silver@basiceducation.dfs.core.windows.net/microdata_basic_education_2022/school_retention_rate_2022_silver'
OPTIONS (
  HEADER = "true",
  DELIMITER = ","
)
```

#### 3.6 ETL - Extract, Transform and Load (Silver - Gold)
Now, the second and final ETL will be performed, which will be related to the Silver layer to Gold. Here, the join of the two tables was made through the School Code column (1:1), the total sum of equipment per school was calculated (since for the analysis, it is only interesting to know the total quantity and not separated by type of equipment), and some more unused columns were removed:

![ETL - Silver para Gold](https://github.com/bbucalonserra/data_engineering/blob/main/pictures/ETL_silver_to_gold.PNG)

Description of transformations:
- Data collection from the Data Lake
- `JOIN`  to merge both tables
- `SELECT`  to remove some columns
- `DERIVED COLUMN`  to remove any remaining special characters
- `SINK`  to send the transformed data back to the Data Lake, but now stored in the Gold layer/container

#### 3.7 Creation of Gold Layer Tables
Finally, it is now possible to perform the final analysis in a much more practical, fast, and consistent way, since we only have usable columns according to the business rules of the analyses.

``` py
CREATE TABLE gold.EDUCATION_RETENTION_SCHOOLS_JOINED
(
  YEAR_CENSUS INT NOT NULL,
  REGION STRING NOT NULL,
  STATE_NAME STRING NOT NULL,
  STATE STRING NOT NULL,
  MUNICIPALITY_NAME STRING,
  MUNICIPALITY_CODE INTEGER,
  DEPENDENCY INTEGER,
  ADMINISTRATIVE_DEPENDENCY STRING,
  LOCATION STRING,
  LOCATION_TYPE INTEGER,
  SCHOOL_CODE INTEGER NOT NULL,
  SCHOOL_NAME STRING,
  PRIVATE_SCHOOL_CATEGORY INTEGER,
  DISTINCT_LOCATION INTEGER,
  BASIC_EDUCATION_ENROLLMENTS INTEGER NOT NULL,
  BASIC_EDUCATION_ENROLLMENTS_FEMALE INTEGER NOT NULL,
  BASIC_EDUCATION_ENROLLMENTS_MALE INTEGER NOT NULL,
  TOTAL_EQUIPMENTS INTEGER,
  COMPUTERS INTEGER,
  ADMINISTRATIVE_COMPUTERS INTEGER,
  EXISTING_ROOMS INTEGER,
  INTERNET INTEGER,
  INDIGENOUS_EDUCATION INTEGER,
  INDIGENOUS_LANGUAGE INTEGER,
  INDIGENOUS_LANGUAGE_1 INTEGER,
  INDIGENOUS_LANGUAGE_2 INTEGER,
  INDIGENOUS_LANGUAGE_3 INTEGER,
  INDIGENOUS_MATERIAL INTEGER,
  BASIC_EDUCATION_APPROVAL_RATE FLOAT, 
  BASIC_EDUCATION_REPROVATION_RATE FLOAT,
  BASIC_EDUCATION_ABANDONMENT_RATE FLOAT,
  PRIMARY KEY ("SCHOOL_CODE")
)

USING CSV LOCATION 'abfss://gold@basiceducation.dfs.core.windows.net/microdata_basic_education_2022/education_retention_schools_joined'
OPTIONS (
  HEADER = "true",
  DELIMITER = ","
)
```

#### 3.8 Data Catalog
A data catalog is a tool that organizes and describes information about available datasets, providing details such as origin, structure, meaning, and relationship between them. It is essential for the management and efficient use of data in an organization. Below is the catalog for the final table in the Gold layer:


| ID | VARIABLE | DESCRIPTION | TYPE | MINIMUM | MAXIMUM |
|----|----------|-------------|------|---------|---------|
| 1 | YEAR_CENSUS | Year of the data | INT | 2022 | 2022 |
| 2 | REGION | Region | STRING | Midwest | South |
| 3 | STATE_NAME | State name | STRING | Acre | Tocantins |
| 4 | STATE | Federative Unit | STRING | AC | TO |
| 5 | MUNICIPALITY_NAME | Municipality name | STRING | Abadia de Goiás | Zumbi dos Palmares |
| 6 | MUNICIPALITY_CODE | Municipality code | INTEGER | 1100015 | 5300108 |
| 7 | DEPENDENCY | "1 - Federal 2 - State 3 - Municipal 4 - Private" | INTEGER | 1 | 4 |
| 9 | LOCATION | Rural or Urban | STRING | Rural | Urban |
| 10 | LOCATION_TYPE | Rural or Urban | INTEGER | 0 | 1 |
| 11 | SCHOOL_CODE | School code | INTEGER | 11000058 | 53086007 |
| 12 | SCHOOL_NAME | School name | STRING | 0101001 ESCOLA MUNICIPAL VICENTE LICINIO CARDOSO | ZUMBI DOS PALMARES EEF |
| 13 | PRIVATE_SCHOOL_CATEGORY | "1 - Private 2 - Community 3 - Confessional 4 - Philanthropic - Not applicable for public schools" | INTEGER | 1 | 4 |
| 14 | DISTINCT_LOCATION | "0 - The school is not in a distinct location area 1 - Settlement area 2 - Indigenous land 3 - Area where a remnant quilombola community is located" | INTEGER | 0 | 3 |
| 15 | BASIC_EDUCATION_ENROLLMENTS | Number of enrollments in basic education | INTEGER | 1 | 999 |
| 16 | BASIC_EDUCATION_ENROLLMENTS_FEMALE | Number of female enrollments in basic education | INTEGER | 0 | 999 |
| 17 | BASIC_EDUCATION_ENROLLMENTS_MALE | Number of male enrollments in basic education | INTEGER | 0 | 999 |
| 18 | TOTAL_EQUIPMENTS | Total technological equipment | INTEGER | 0 | 99 |
| 19 | COMPUTERS | Total computers | INTEGER | null | null |
| 20 | ADMINISTRATIVE_COMPUTERS | Total administrative computers | INTEGER | null | null |
| 21 | EXISTING_ROOMS | Number of existing rooms | INTEGER | 0 | 1 |
| 22 | INTERNET | Has internet or not (1 or 0) | INTEGER | 0 | 1 |
| 23 | INDIGENOUS_EDUCATION | "0 - No 1 - Yes" | INTEGER | 1 | 3 |
| 24 | INDIGENOUS_LANGUAGE | "1 - Only in Indigenous Language 2 - Only in Portuguese Language 3 - In Indigenous and Portuguese Language - Not applicable for schools without Indigenous School Education" | INTEGER | 1 | 3 |
| 25 | INDIGENOUS_LANGUAGE_1 | Indigenous Education - Language in which teaching is conducted - Indigenous Language - Language Code 1 | INTEGER | 1 | 999 |
| 26 | INDIGENOUS_LANGUAGE_2 | Indigenous Education - Language in which teaching is conducted - Indigenous Language - Language Code 2 | INTEGER | 100 | 999 |
| 27 | INDIGENOUS_LANGUAGE_3 | Indigenous Education - Language in which teaching is conducted - Indigenous Language - Language Code 3 | INTEGER | 126 | 999 |
| 28 | INDIGENOUS_MATERIAL | Socio-cultural and/or pedagogical instruments and materials in use in the school for the development of teaching and learning activities - Indigenous | INTEGER | 0 | 1 |
| 29 | BASIC_EDUCATION_APPROVAL_RATE | Basic education approval rate | FLOAT | null | null |
| 30 | BASIC_EDUCATION_REPROVATION_RATE | Basic education reprovation rate | FLOAT | null | null |
| 31 | BASIC_EDUCATION_ABANDONMENT_RATE | Basic education abandonment rate | FLOAT | 0.0 | 9.0 |



### 4. Analysis
Data analysis is an essential practice in an increasingly digital and information-driven world. It plays a fundamental role in various areas, from the business world to academic research. The primary goal of major technology companies is to become increasingly data-driven, meaning they are guided by data. In this final stage, the analysis will focus on education in indigenous lands in Brazil.


#### 4.1 Data Quality
Before delving into the analysis itself, it is crucial to perform an assessment of the data quality contained in the gold layer (final layer) to comprehensively understand how these data may influence the final analyses to be conducted. In this context, our attention will be dedicated to identifying possible inconsistencies or flaws in the data, aiming to ensure that subsequent analyses are based on reliable information.

There are still some issues with the data quality for certain columns.
The column **MUNICIPALITY_NAME** is still obtaining the value "�" for letters with accents or for the letter "ç" ("still" because this issue was addressed in the ETL from Bronze to Silver layer). Since these are only naming problems, it will not affect the answers provided below. However, in the case of creating a data visualization dashboard, for example, a map chart with "�" characters, Power BI will not be able to identify the municipality's location.
The column **BASIC_EDUCATION_APPROVAL_RATE** has null values throughout the ETL process for some reason. This prevents analyses related to student approval in indigenous schools, a comparison between approvals with students in indigenous areas and regular schools from being performed.
The column **EXISTING_ROOMS** also has null values, possibly due to some stage of the ETL process. This prevents analyses on the number of students per classroom in schools in indigenous areas or checks if the infrastructure of schools in indigenous areas meets the population's needs.
The columns **COMPUTERS** and **ADMINISTRATIVE_COMPUTERS** are also null, possibly due to some stage of the ETL process. This prevents answering questions regarding computers in indigenous schools ("Does the presence of computers in indigenous schools have any influence on the dropout rate?") and may bias the results regarding technological equipment.
For the remaining data, no issues were found. However, it would be interesting to remove some columns to improve data processing in queries since not all columns were used.


#### 4.2 Problem Resolution
In this section, an analysis and answers to the questions raised regarding indigenous education in Brazil will be presented. Through graphical representations and analysis, insights will be provided regarding education in indigenous lands.
Throughout this section, there will be charts and analyses addressing key questions, including the location of schools in indigenous lands, dropout rates, availability of technological equipment, internet access, and language of instruction. For all analyses below, the `SQL` (Structured Query Language) was used.


**1. Where are located the indiginous schools in Brazil?**

<details>
  <summary>Show Answer</summary>
  
<img src="https://github.com/bbucalonserra/data_engineering/blob/main/graphics/loc_escolas_indigenas.PNG" align="left"
     alt="loc_escola_indigena">

Query:
``` py
SELECT
STATE_NAME,
COUNT(SCHOOL_CODE) AS COUNT_SCHOOLS
FROM gold.EDUCATION_RETENTION_SCHOOLS_JOINED
WHERE DISTINCT_LOCATION = 2
GROUP BY STATE_NAME
ORDER BY COUNT_SCHOOLS DESC
```

Response: Schools in indigenous lands are located in various states of Brazil. Based on the count of schools per state, we can identify the states with the highest number of schools in indigenous lands:
- Amazonas: 2,190 schools
- Roraima: 674 schools
- Maranhão: 642 schools
- Pará: 618 schools
- Acre: 456 schools

Therefore, schools in indigenous lands are mainly concentrated in the states of the Northern region, with Amazonas and Roraima leading in terms of the number of schools. This distribution reflects the presence of indigenous communities in these regions and the need for education in the areas of their lands.

</details>
</details>


**2. What is the dropout rate in indigenous schools? Is this value higher or lower than regular schools?**

<details>
  <summary>Show Answer</summary>
  
<img src="https://github.com/bbucalonserra/data_engineering/blob/main/graphics/Taxa_de_Abandono.PNG" align="left"
     alt="taxa_de_abandono">

Query:
``` py
WITH INDIGENOUS_ED AS (
  SELECT
    AVG(BASIC_EDUCATION_ABANDONMENT_RATE) AS AVERAGE_INDIGENOUS_EDUCATION
  FROM gold.EDUCATION_RETENTION_SCHOOLS_JOINED
  WHERE
    DISTINCT_LOCATION = 2
    AND BASIC_EDUCATION_ABANDONMENT_RATE <> 0
    AND BASIC_EDUCATION_ABANDONMENT_RATE IS NOT NULL 
),

GENERAL_ED AS (
  SELECT
    AVG(BASIC_EDUCATION_ABANDONMENT_RATE) AS AVERAGE_BASIC_EDUCATION_GENERAL
  FROM gold.EDUCATION_RETENTION_SCHOOLS_JOINED
  WHERE
    BASIC_EDUCATION_ABANDONMENT_RATE <> 0
    AND BASIC_EDUCATION_ABANDONMENT_RATE IS NOT NULL  
)

SELECT
  ROUND(INDIGENOUS_ED.AVERAGE_INDIGENOUS_EDUCATION, 2) AS AVERAGE_INDIGENOUS_ED,
  ROUND(GENERAL_ED.AVERAGE_BASIC_EDUCATION_GENERAL, 2) AS AVERAGE_GENERAL_ED,
  ROUND((INDIGENOUS_ED.AVERAGE_INDIGENOUS_EDUCATION - GENERAL_ED.AVERAGE_BASIC_EDUCATION_GENERAL), 2) AS PERCENTUAL_DIFFERENCE,
  ROUND((INDIGENOUS_ED.AVERAGE_INDIGENOUS_EDUCATION - GENERAL_ED.AVERAGE_BASIC_EDUCATION_GENERAL) / GENERAL_ED.AVERAGE_BASIC_EDUCATION_GENERAL * 100, 2) AS DIFFERENCE_IN_PERCENTAGE
FROM INDIGENOUS_ED, GENERAL_ED
```


Response: The dropout rate in indigenous schools is 18.59%, while in regular schools it is 7.32%. Therefore, we can conclude that the dropout rate in indigenous schools is considerably higher than in regular schools, with a difference of 11.27% higher than regular schools. This suggests that indigenous schools may face additional or different challenges that contribute to a higher dropout rate compared to non-indigenous schools. It is important to investigate and address these challenges to improve access and the quality of education for indigenous communities.

</details>
</details>



**3. What is the average number of technological equipment per state in schools with indigenous education?**

<details>
  <summary>Show Answer</summary>

  <img src="https://github.com/bbucalonserra/data_engineering/blob/main/graphics/media_equip_escolas_por_estado.PNG" align="left"
     alt="media_equipamentos_estado">

Query:
``` py
SELECT
  STATE_NAME,
  ROUND(AVG(TOTAL_EQUIPMENTS),2) AS AVERAGE_EQUIPMENTS
FROM gold.EDUCATION_RETENTION_SCHOOLS_JOINED
WHERE
  BASIC_EDUCATION_ABANDONMENT_RATE IS NOT NULL
  AND BASIC_EDUCATION_ABANDONMENT_RATE <> 0
  AND DISTINCT_LOCATION = 2
GROUP BY ALL
ORDER BY AVERAGE_EQUIPMENTS DESC
```

Answer: The above graph shows the average number of technological equipment available in schools with indigenous education in each state. Santa Catarina has the highest average, with 9 equipment, while Mato Grosso, Tocantins, Mato Grosso do Sul, Acre, Amapá, and Maranhão have very low averages, close to zero. These numbers indicate the disparity in the availability of technological equipment in indigenous schools in different states of Brazil.

</details>
</details>

**4. What is the percentage of schools in indigenous locations that have internet access by state?**

<details>
  <summary>Show Answer</summary>


  <img src="https://github.com/bbucalonserra/data_engineering/blob/main/graphics/porcentagem_escolas_indigenas_com_internet.PNG" align="left"
     alt="internet_por_estado">

Query:
``` py
SELECT
  STATE_NAME,
  ROUND((SUM(CASE WHEN INTERNET = 1 THEN 1 ELSE 0 END) / COUNT(*)) * 100, 2) AS PERCENTAGE_WITH_INTERNET
FROM gold.EDUCATION_RETENTION_SCHOOLS_JOINED
WHERE 
  DISTINCT_LOCATION = 2
GROUP BY STATE_NAME
ORDER BY PERCENTAGE_WITH_INTERNET DESC
```

Answer: The above numbers represent the percentage of indigenous schools in each state that have internet access. While some states, such as Paraná and Goiás, have 100% of their indigenous schools with internet access, others, such as Piauí and Acre, have a very low or even zero percentage of schools with internet access. This reflects the variation in information technology infrastructure in different regions of the country and highlights the need to improve internet access in indigenous schools across Brazil.

</details>
</details>

**5. In which language are subjects taught in indigenous schools? Are we maintaining the roots of the tribes regarding the mother tongue?**

<details>
  <summary>Show Answer</summary>


  <img src="https://github.com/bbucalonserra/data_engineering/blob/main/graphics/linguas_indigenas.PNG" align="left"
     alt="lingua_indigena">

Query:
``` py
SELECT
  INDIGENOUS_LANGUAGE,
  ROUND(COUNT(SCHOOL_CODE) * 100.0 / SUM(COUNT(SCHOOL_CODE)) OVER (), 2) AS PERCENTAGE_OF_SCHOOLS
FROM gold.EDUCATION_RETENTION_SCHOOLS_JOINED
WHERE
  DISTINCT_LOCATION = 2
GROUP BY INDIGENOUS_LANGUAGE
ORDER BY INDIGENOUS_LANGUAGE
```

Answer: In indigenous schools, subjects are taught in different languages, and some schools adopt a bilingual approach. Here is the distribution based on the data:
  - Indigenous language only: 3.30% of indigenous schools exclusively adopt the indigenous language as the medium of instruction
  - Portuguese: 22.70% of indigenous schools teach subjects only in Portuguese
  - Indigenous language and Portuguese: The majority of indigenous schools, 71.97%, adopt a bilingual approach, teaching subjects in both the indigenous language and Portuguese
  - Not applicable without indigenous education: 2.02% of the data is not applicable, indicating that these schools do not offer indigenous education or did not provide information about the language of instruction

Therefore, most indigenous schools in Brazil adopt a bilingual approach, teaching subjects in both the indigenous language and Portuguese, which reflects the importance of preserving the roots of the tribes regarding the mother tongue while providing access to education in Portuguese.

</details>
</details>


### 5. Self Assessment
The project was conducted with an extremely detailed approach, resulting in documentation that I consider excellent. Every line of code and step of the cloud computing system was explained, including not only what was done. This transparency and clarity significantly contributed to my understanding of the process.

Also, the development of this work was integrated with studies, covering detailed explanations of the types of data engineering performed to use Azure, creation of tables, ETL processes and analysis. This correlation between the practical project and the theoretical foundation was very important, providing a deeper and contextualized understanding of the entire analytical process.

The cloud computing system presented the greatest challenge. Yet, I gleaned invaluable insights and a good data lake environment, enhancing my technical prowess and problem-solving skills. Despite its difficulty, navigating the cloud proved huge step in my studies, equipping me with the knowledge and confidence to tackle future endeavors.

**Additionally, I'm proud to share that my project received a perfect score of `10` of `10` (MAXIMUM SCORE) from the esteemed professors at the university.**
