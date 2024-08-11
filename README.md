**CLOUD MIGRATION PROJECT**

**Planning**
Cloud Solutions Factory ICT has been contracted to provide a data analysis solution to XYZ, a small environmental non-profit organization based in Wichita, Kansas. XYZ is focused on studying the impact of electric vehicles on air pollution and climate change, and has decided to migrate their IT infrastructure to the cloud in order to reduce costs and increase efficiency. As part of this migration, Cloud Solutions Factory ICT will build a data lake and data warehouse to store and analyze the data provided by XYZ.
The ultimate goal is to build a data lake, data warehouse and provide dashboards to the customer using various AWS services. The project involves analyzing and gaining insights from data that comes in various formats through different methods, including manual data load and API call. In this document, we will provide a detailed overview of all AWS services used for cloud migration and highlight the benefits of this transition.

**Implementation**
**Designing the Architecture**

The first milestone was to design the architecture using AWS services and present it to the class. The architecture was developed using the draw.io tool which included our AWS services that we chose. You can see the diagram below:

<img width="502" alt="image" src="https://github.com/user-attachments/assets/d75ee3ed-4da1-43c0-9a75-98c261a3778a">


**AWS Services**

Below is the detailed explanation of how each AWS service was applied to the cloud solution with a brief explanation why a particular practice was applied. These services are the foundation of the proposed architecture and will be used to build the data lake, data warehouse, and dashboards for the customer.

<img width="36" alt="image" src="https://github.com/user-attachments/assets/e4b03d49-f6f9-499b-b4d2-2d1e5644e044"> Simple Storage Service (S3)

This service was used to temporarily store data provided by the customer.
Created a bucket with a name: csf-ict-251572410613 to store the data related to this customer
Created 3 folder inside the bucket:
input - to upload the raw data file, csv manually, json by an API
output - to store clean data, in parquet format
athena-results - to store athena queried results


<img width="41" alt="image" src="https://github.com/user-attachments/assets/06d5acd8-96de-4ed6-8cd0-55274a1afed7">  Lambda

Automated 4 functions:
TriggerGlueCrawlerCSV
Trigger - A new object created in S3 input folder
This function triggers the Glue Crawler to perform data crawling over any new object added in the input bucket.
The code can be seen below:

![image](https://github.com/user-attachments/assets/35bd9f57-9cbf-4fa1-944d-77b9da89ef0c)

TriggerGlueJob
Trigger - EventBridge: CsvCrawlerFinished
This function triggers GlueJob to transform extracted data into parquet format The code can be seen below:

![image](https://github.com/user-attachments/assets/38ca0de2-d0c6-4d21-8c82-7d08e9ab99b5)


TriggerGlueCrawlerParquet
Trigger - EventBridge: CsvParquetJobFinished
This function triggers the Glue Crawler to crawl the parquet data after it has gone through the ETL stage.
The code can be seen below:

![image](https://github.com/user-attachments/assets/98fdf94b-523a-47ae-91ba-5378b3032a30)


**LoadDataRedshift ( in theory )**
**Trigger - a new object created in s3 output folder**
This function loads parquet files from s3 output folder to Redshift Cluster The code can be seen below:
 
![image](https://github.com/user-attachments/assets/441aca79-75d9-46cd-8b00-d48320bb1c03)


**Glue Crawler**

Created two crawler to extract the data from the data file, and create data catalog

**InputCrawler**
**Trigger - Lambda: TriggerGlueCrawlerInput**
The first crawler extracts all the raw data in the input folder. That data will be used to create a data catalog and database for the first overview.

**outputCrawlerParquet**
**Trigger - Lambda: TriggerGlueCrawlerParquet**
The second crawler extracts all the data from the output folder after it has been extracted, transformed, and loaded. This data will be used to create a final database with clean data, that can be later accessed through athena for any testing or troubleshooting purposes.


**Glue Data Catalog**
 
Was used in the solution to create a schema for the tables. Input database stores table schema for the raw data before it has been transformed. Output database stores table schema for the cleaned data after it has been Transformed and loaded back in S3, right before getting uploaded to Redshift. When Created Glue Job, we referred to Data Catalog for the schema.


**Glue Job**

**Created a GlueJob: csv-parquet-csf-ict and json-parquet-csf-ict**
Glue job was the main part of the data pipeline. It transformed the data from the csv and json format into Parquet format. Parquet is a binary file format that stores data in a compressed and highly efficient way.


**CloudWatch**

**Created an alarm**: Budget Alarm to keep track of our spendings in the AWS environment. Set up to automatically notify the administrator through SNS, if the overall spending for the project is more than five dollars.

**Created three EventBridges:**

**CsvCrawlerFinished **- When the inputCrawler is successfully finished, it invokes TriggerGlueJob Lambda function

**CsvParquetJobFinished** - When the csv-parquet-csf-ict is successfully finished, it invokes TriggerGlueCrawlerParquet Lambda function

**ParquetCrawlerFinished** - When the outputCrawlerParquet is successfully finished, it invokes an SNS topic CrawlerFinished, and sends out an email to the administrator that the Crawler has successfully finished.

**CloudTrail Integration:**
 
Two CloudWatch Log Groups were created for auditing and security purposes. See more in the CloudTrail section of this documentation


**Athena**
Athena was used for testing purposes in the solution. Once the data has been put in the output folder, Athena has been set to store the queries in the following path: s3://csf-ict-251572410613/athena-results/
We connected Athena to the output database, as that is the schema of the clean data that will be used for further goals.


**QuickSight**
Connected QuickSight To the S3 Output folder used to analyze and explore data from tables to provide interactive visualizations, dashboards empowering users to gain insights and make data-driven decisions. Quicksight helps us to access and visualize relevant data enabling us to make informed decisions and drive business growth at CSF-ICT.


**Amazon Redshift**


Explored Redshift and Redshift Serverless. Decided as the team, that Redshift Cluster will fit more for our goals. Redshift Serverless imports data from S3 for querying, while Redshift Cluster stores the data.
We explored the NodeType options, and after doing some estimates in AWS Pricing Calculator decided that the best option would be ds2.large. It is the most affordable option, and we don’t plan on storing large data sets, so this particular node was good enough.
After we created the cluster **“csf-ict-datawarehouse”**, we enables a Lambda function LoadDataRedshift to load data into the cluster using COPY command, which gets invoked every time, when a new file has been added to the output folder. Redshift Serverless can be also used to query data from the redshift cluster.


**Simple Notification Service (SNS)**


We implemented SNS with **CloudWatch**. Created two topics, and added a subscriber to each of them as users who need to be notified. The first topic informs the admin when our billing is above 5$. The other one informs the admin, when the 2nd crawler, that extracted parquet data is successfully finished. SNS offers multiple ways of informing subscribers, such as SMS or email.


**Identity and Access Management (IAM)**

IAM has been the service we probably used the most. The first step was to create an Admin user and give it admin access. We also Enabled MFA to add an extra layer of security. The rest of the cloud solution was deployed through the admin account. We created multiple user groups: Administrator, Business Analyst, Cloud Architect, Customer, Develop and assigned multiple permissions to each of those groups. We decided to go with Identity based policies, since we have less team members who work on this project compared to the services implemented. We created roles for Glue Crawler, Glue Job, Lambda, CloutTrail, Redshift and CloudTrail.



**CloudTrail**


We integrated CloudTrail with CloudWatch. Created two types of Logs: **Management events** - to keep track of who signs in, what services they access inside the AWS ecosystem, and **Data events** - to keep track of the Lambda functions execution.
CloudWatch was integrated due to simplicity of viewing, filtering, predefining searches, ability and ability to set an alarm when a particular event happens.
 
**AWS Well-Architected Framework**

In this section, our team evaluated our solution and compared it with AWS
Well-Architected Framework, created by Amazon Web Services to make sure it follows best practices and use cases. Below we will discuss Each pillar of the Framework and talk about how our solution aligns with them.

**Operational excellence**

Implementation of the data lake, data warehouse is part of improving its business management. Making it cloud native as well, as it offers some more benefits for this specific use case. Most of the services in AWS are connected to CloudWatch. which lets management track how services operate, its cost, and possible failures. We also implemented CloudTrail, Data Events, to see how well the services function, and help eliminate errors. Implementation of data lake automates the process of clearing, and transforming data for future review.

**Security**

Implemented the project through an Admin account. Root user account was only used to monitor the cost of a daily basis.
Implement strict access controls to limit who can invoke lambda functions, initiate Glue Crawlers and execute Glue jobs. We also used AWS Identity and Access Management (IAM) roles and policies to grant least privilege access, ensuring that only authorized users or services can perform these actions. We have enabled logging and monitoring to capture and analyze activity logs and events related to Lambda functions, Glue crawlers, and Glue jobs. We also implemented AWS CloudTrail for auditing and tracking API calls and AWS CloudWatch for monitoring and alerting on specific events or anomalies. Leveraged features such as MFA for our project too.


**Reliability**

We have used data redundancy and replication techniques to enhance data durability and availability by leveraging services like Amazon S3 for data storage, which automatically replicates data across multiple AZs. We have designed our data lake architecture to handle varying workloads by using automated scaling mechanisms by utilizing services like AWS Glue and AWS Lambda, which can automatically scale resources based on demand, allowing your data lake to adapt to workload fluctuations.
 
AWS Cloudwatch and AWS Lambda, can be used for automated responses to events, such as failures or resource scaling, ensuring timely recovery and reducing manual intervention.

**Performance Efficiency**

Implemented data compression techniques to reduce storage requirements and improve data transfer speeds by using compression formats like Parquet or ORC that are optimized for query performance. We have optimized data queries by leveraging services like Amazon Athena to improve query performance. We have AWS Lambda or AWS Glue for on demand and scalable data processing to optimize resource utilization and reduce costs. We leveraged services like Amazon CloudWatch to implement monitoring and analysis to gain insights into the performance of your data lake architecture and applications.


**Cost Optimization**

We have utilized serverless computing services like AWS Lambda, AWS Glue for data processing tasks. These services automatically scale based on demand, allowing you to pay only for the actual compute resources used, reducing costs during idle periods. This ensures that you only pay for the resources needed to handle the current workload. We have implemented data compression techniques to reduce storage costs to compress data using formats like Parquet which offer excellent compression ratios while still allowing for efficient query processing. We were regularly monitoring and analyzing data lake’s cst and usage patterns using the AWS Cost Explorer and also regularly checking the free-tier resources to check if we were falling out of the tier services .
 

**Challenges**

●	Redshift

●	Uploading API data with Lambda function to S3

●	Learner Lab Limitations

●	Implementation of the project in the actual AWS account

●	Ability to experiment due to possible charge

●	Schedule conflict

●	Athena



