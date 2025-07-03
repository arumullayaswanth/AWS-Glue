# 🧒 AWS Glue Project: 

Hi there! Let's build a project in AWS Glue step-by-step. You don't need to be a cloud genius. Just follow along like LEGO blocks. 🧱

---

## ✅ What Are We Doing?

We will:
1. Store some sample data in S3 🍯
2. Tell AWS Glue to look at it with a crawler 🔍
3. Store the table info in a database 📚
4. (Optional) Use Athena to see the data 💻

---

## 🧰 What You Need First

- An AWS Account
- IAM role for AWS Glue with access to S3
- Some CSV data (we'll help you make one)
- Region: Choose one and use it everywhere (e.g. `us-east-1`)

---


## 1️⃣ Create an IAM Role for Glue (Skip if you already have one)

Go to AWS Console → IAM → Create Role:

- trusted entity: `AWS service`
- Use case: **Glue**
- select `Glue` Allows Glue to call AWS services on your behalf.
- Attach these policies:
  - `AmazonS3FullAccess` (for demo)
  - `AWSGlueServiceRole`
- Name it something like: `Aws-GlueRole`
- Create role


## create an S3 Bucket

### 🗂️ S3 Folder Structure

```
my-awesome-glue-bucket/
└── data-store/
    └── custormers_data/
        └── csv_report/
            └── custormers_data.csv
```



1. Go to AWS Console → S3 → Create bucket:
      
    -  Bucket name: `my-awesome-glue-bucket`
    -  Region: Choose one and use it everywhere (e.g. `us-east-1`)
    -  Create bucket

2. open `my-awesome-glue-bucket` bucket
    - create a folder 
    - folder name: `data-store`
    - create folder
3. open `data-store` folder
    - create a folder 
    - folder name: `custormers_data`
    - create folder
4. open `custormers_data` folder
    - create a folder 
    - folder name: `csv_report`
    - create folder
5. open `csv_report` folder
        - upload a file 
**Note:** here successful uploaded the custormers_data.csv file
  


## 3️⃣ Create a Glue Database

1. Go to AWS Console → Glue → Databases: 
   - add Database:
   - database name: `custormers_data`
   - location: `s3://my-awesome-glue-bucket/data-store/custormers_data/`
   - description: `My first Glue DB`
   - create database
2. createing table 
     - go to AWS console → Glue → Crawlers: 
          - add table using crawler:
          - crawler name: `custormers_data`
          - next
              - Data source configuration: `not yet`
              - Data source : `Add a data source`
                  - data source type: `S3`
                  - S3 path: `s3://my-awesome-glue-bucket/data-store/custormers_data/csv_report/`
                  - subsequent crawler runs : ` crawl all sub-folder`
                  - Add an s2 data source
              - next:
                  - IAM role: `Aws-GlueRole`
                  - next:
                      - output configuration:
                      - target database : `custormers_data`
                      - next

                      - finish
                      - create table
3. run the table
     - go to AWS console → Glue → Crawlers: 
     - select `custormers_data`
     - **run**

4. check the table
     - go to AWS console → Glue → Databases: 
     - select `custormers_data`
     - **table**: `custormers_data`
          



## 🎉 You're a Data Engineer Now!

