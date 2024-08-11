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

<img width="176" alt="image" src="https://github.com/user-attachments/assets/5b7b4dea-a9be-4e61-a2fd-09b3928a05fa">


TriggerGlueJob
Trigger - EventBridge: CsvCrawlerFinished
This function triggers GlueJob to transform extracted data into parquet format 
The code can be seen below:
![image](https://github.com/user-attachments/assets/ac6e1594-79fc-4a6d-977b-ebda83c2d82f)


