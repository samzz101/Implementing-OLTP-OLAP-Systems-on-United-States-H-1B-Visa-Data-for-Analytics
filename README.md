# Implementing OLTP, OLAP Systems on United States H-1B Visa Data for Operational Usage and Analytics

## Introduction
Online Transactional Processing (OLTP) systems, which function as operational data stores (ODS), facilitate real-time data processing. On the other hand, Online Analytical Processing (OLAP) systems are used for analytical purposes and provide insights based on aggregated data. Implementing these systems with H-1B visa data enables efficient access and analysis of data from ODS and performs in-depth analysis on a data warehouse (DWH).
The H-1B visa is a nonimmigrant work visa that permits U.S. employers to hire foreign workers for specialty occupations requiring at least a bachelor's degree or equivalent. Foreign professionals with specialized skills and education often apply for H-1B visas. By leveraging the H-1B visa data, it is possible to perform analysis and create dashboards to understand various educational and geographical factors of the applicants.
An Entity Relationship (ER) model was developed by gathering the necessary data points from U.S. Citizenship and Immigration Services (USCIS). This ER diagram was used to construct the OLTP database within a cloud MySQL instance. Data from the MySQL database was then extracted and transformed using Python and loaded into a data warehouse. Analytics were performed on the data warehouse, with the results represented through a dashboard. This approach provided valuable insights into the characteristics and trends within the H-1B visa application data, facilitating a better understanding of the educational and geographical backgrounds of applicants.

## Goals and Motivation
1. The goal and motivation of this analysis are to provide comprehensive insights into various aspects of H-1B visa data. The specific objectives are:
2. Top 10 Applicant Countries: Identify and rank the top 10 countries from which H-1B visa applicants originate.
3. Popular Education Background: Analyze the educational backgrounds of all applicants to determine the most common qualifications and fields of study among H-1B visa holders.
4. Major Employers: Determine the employers with the highest number of H-1B visa applications and approvals.
5. State-Wise Application Distribution: Examine the distribution of H-1B visa applications across different states in the United States.
6. Average Wage by Job Title: Calculate the average wage for H-1B visa applicants across different job titles to understand compensation trends.

## Overview and Architecture
The goal of this project is to implement Online Transactional Processing (OLTP) and Online Analytical Processing (OLAP) systems on the cloud, with the entire process automated in a single data pipeline or workflow. After evaluating various cloud service providers, Amazon Web Services (AWS) was selected for its competitive pricing and flexible services. The data for this project was extracted from the U.S. Citizenship and Immigration Services (USCIS) in CSV format. This raw data, which consists of 154 columns, is unnormalized and requires preprocessing to prepare it for analysis.

<img width="820" alt="Screenshot 2024-08-07 at 9 08 11 PM" src="https://github.com/user-attachments/assets/7dca0a89-b4d9-4ac7-bc8e-98aba71a311f">

Post-normalization of the data, the CSV files will be migrated to Amazon Simple Storage Service (S3), located in the US East (N. Virginia) region (us-east-1). S3 provides scalable object storage services through a web service interface, ensuring reliable and cost-effective data storage. The cloud SQL MySQL instance, functioning as the OLTP system, will be located in the us-east-1b availability zone. Identity and Access Management (IAM) roles, users, and groups will be created to manage access to the Relational Database Service (RDS). The instance will be configured with a Virtual Private Cloud (VPC) to monitor and filter incoming and outgoing traffic, allowing access only to authorized users. The MySQL instance will also have a backup policy in place to safeguard against unexpected data loss.
ETL (Extract, Transform, Load) scripts will be developed using Python and orchestrated through AWS Glue Jobs. AWS Glue is a serverless data integration service based on the Apache Spark Structured Streaming engine. This setup will handle the extraction of data from the MySQL instance, its transformation, and loading into the data warehouse (DWH). Once the data is in the data warehouse, analysis will be performed, and SQL views will be created for reporting purposes. These views will facilitate the generation of visual reports based on the analyzed data, providing actionable insights and facilitating data-driven decision-making.

## Project Flow
An on-demand AWS Glue workflow has been designed to automate the creation and maintenance of OLTP and OLAP systems. The workflow begins with jobs that set up the tables in the MySQL database, ensuring that the schema adheres to the defined constraints. Once the tables are created, data from CSV files stored in Amazon S3 is loaded into these tables, populating the MySQL instance with the necessary raw data. The next stage involves the ETL process, where data is extracted from the newly populated MySQL tables and loaded into a Pandas DataFrame for preprocessing. This data is then transformed and ingested into Snowflake, which serves as the data warehouse for OLAP operations. Finally, the Tableau data source is refreshed to reflect the updated data, showcasing the latest insights on dashboards. This workflow efficiently manages the end-to-end data processing, from initial ingestion to real-time visualization, ensuring accurate and timely reporting.

<img width="1194" alt="Screenshot 2024-08-07 at 9 09 14 PM" src="https://github.com/user-attachments/assets/550989ed-08b0-4915-a03a-a2950e745f01">

## Database Implementation
Data is collected from various sources such as Kaggle, the US Department of Labor website, and other sources which contain a total of 64 columns. Multiple data cleaning procedures are performed on the data, including elimination of special characters, transformation of incomplete data into meaningful values, transformation of incorrectly formatted data, and formatting the inconsistent data to make data more meaningful. After the data cleaning process, data is normalized into 8 tables and data is modeled into a relational schema.

<img width="991" alt="Screenshot 2024-08-07 at 9 11 44 PM" src="https://github.com/user-attachments/assets/a4c6646c-d090-4705-bb93-53075d684365">

## About the Data
The US H1-B Visa data is collected from various sources to make the data more meaningful and comprehensive. It is open-source data from the US Department of Labor that maintains the work-related Visa. It provides information on applications they receive every year for an H1-B visa and related information about employers and applicants. Postal code data has been taken from the USPS official website to populate the postal code, city, and state data of the applicant, employer, and job-related information in the above H1-B visa application data. The above data provides information on applications that are approved, denied, or withdrawn for H1-B visas for the year 2022. This data includes the information of the applicant, employer, job of the applicant, prevailing wage, and status of the application. This dataset contains more than 70,000 applications and their related information. Some of the important attributes in the data set are:
* Application case Number
* Case Received date
* Case Status
* Case Decision Date
* Applicant Citizenship
* Employer Name
* Applicant Education
* Job information
* Job work place
* Job Title
* Prevailing wage
* Prevailing wage job title
Some of the important relationships between various attributes are as follows:
* Applicant details are provided in the application.
* Employers that are filing on behalf of the applicant.
* Applications filed by the third party agents for the applicants.
* Nationalities of the applicants.
* Job roles of the applicants that are applying.

The data set is normalized to satisfy 1NF, 2NF, and 3NF by splitting the data into 8 tables by following all the required constraints. The Application table contains the majority of the information about the H1-B visa data and the information of Agent, Employer, Applicant, Job Information, and Prevailing Wage information is separated and referenced to the Application table. A geography table has been created to match the correct postal codes with the data which is referenced to Employer, Applicant, and Job information tables. Education information contains the data related to the education of the applicant and the Education qualification required for the job.
## ETL Process
The ETL (Extract, Transform, Load) process is implemented using Python and orchestrated through AWS Glue jobs. During the extraction phase, data is retrieved from the cloud MySQL instance using secure access keys, which are stored in a .py file within an S3 bucket to prevent exposure of sensitive information. The connection to MySQL is established through the access details read from the file, and data is extracted into a DataFrame using the read_sql function before closing the connection. In the transformation phase, the data in the DataFrame undergoes several modifications, including renaming columns, handling null values, removing unnecessary columns, and correcting revenue field values. String formatting is applied to text columns, and a "Refreshed Date" column is added to each table to record the upload timestamp. Finally, in the loading phase, the transformed data is transferred to the Snowflake database using Python-based DB connectivity. Similar to the MySQL access details, Snowflake credentials are securely managed by storing them in a .py file. Once the connection to Snowflake is established, the cleaned DataFrame is loaded into the database tables, completing the ETL process.
### Logging and Exception Handling
The Python script developed for the ETL process utilizes modularization to ensure a structured and maintainable codebase. Each step of the process—connection to S3, MySQL, and Snowflake—is implemented with robust exception handling. This approach ensures that the script is resilient to errors and failures. If any issues arise during the connections or data processing, the exception handling mechanisms capture and display error messages, allowing for prompt identification and troubleshooting of problems without causing the entire code to break. This systematic approach to logging and exception handling enhances the reliability and stability of the ETL workflow.
## Datawarehouse Implementation
Upon successful completion of the ETL process, the data is loaded into the Snowflake cloud data platform, which acts as the data warehouse for the H-1B visa dataset. Snowflake is a powerful data warehousing system renowned for its large-scale data management capabilities. It offers cross-cloud deployment, high scalability, and robust security features. The data is ingested into Snowflake according to the defined data model. For this project, a STAR schema model is implemented in the data warehouse: the Application table serves as the fact table, while Geography, Prevailing Wage, Education Info, and Agent tables function as dimension tables. This schema design supports efficient querying and analysis of the H-1B visa data by organizing it into a structured format that enhances performance and scalability.

<img width="452" alt="Screenshot 2024-08-07 at 9 10 19 PM" src="https://github.com/user-attachments/assets/b8dbee60-df90-4fdf-8460-5e5d6e429557">

## Data Analysis
After inserting data into the snowflake, using SQL we queried the data for the required analysis.
### Top 10 Countries of the Applicants


### Popular Education Background for the Applicants


### Top 10 Employers that are Filing the H1-B Applications

