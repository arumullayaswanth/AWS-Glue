
# 🛠️ Step-by-Step: Incremental Data Load from S3 to Redshift


## Create an IAM Role
Go to AWS Console → IAM → Create Role:

- trusted entity: `AWS service`
- Use case: **Glue**
- select `Glue` Allows Glue to call AWS services on your behalf.
- Attach these policies:
  - `AmazonS3FullAccess` (for demo)
  - `AWSGlueServiceRole`
  - `AmazonRedshiftFullAccess`
- Name it something like: `Aws-GlueRole`
- Create role


## create an S3 Bucket

my-awesome-glue-bucket/
└── data-store/
    └── custormers_data/
        └── csv_report/
            └── custormers_data.csv


1. Go to AWS Console → S3 → Create bucket:
  - Bucket name: `my-awesome-glue-bucket`
  - Region: Choose one and use it everywhere (e.g. `us-east-1`)
  - Create bucket
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
  
## create Amazon Redshift Cluster
1. Go to AWS Console → Redshift → serverless dashboard:
      - Create workgroup
      - workgroup name: `readshidt-cluster`
      - next
        -  Choose namespace
        - select Create a new namespace
        - Namespace : `redshiftdb`
        - select Customize admin user credentials
        - select Manually add the admin password
        - Admin user password : `Yaswanth123`
        - next
        - create
    
## create vpc Endpoint to S3
1. Go to AWS Console → VPC → Endpoint → Create endpoint
      - name: `s3-endpoint`
      - type: `ASW service`
      - Service name: `com.amazonaws.us-east-1.s3`
      - VPC: `your_vpc_id`
      - Subnet: `your_subnet_id`
      - Create endpoint

## Create External Table to Read S3 Data
1. Go to AWS Console → Redshift → serverless dashboard:
      - click workgroup
      - click `readshidt-cluster`
      - click ` ouery data`
      - click `Create external table`

      ```bash
            CREATE TABLE Persons (
        industry_name_ANZSIC int,
        rme_size_grp varchar (225),
        variables varchar (255)

      );
      ```
      - click `RUN`
2. check table is created or not
       ###  📂 Redshift Serverless Cluster Structure
      - Serverless: `readshidt-cluster`
           - native databases
              - `dev`
                - `public`
                  - **Tables** (1)
                     - `persons` ✅
                         - Right-click `persons` → **Select table** 

                          ```bash
                          SELECT
                                *
                            FROM
                                "dev"."public"."persons";
                          ```      

                          - Click **Run** in the right-side editor

                              - **Columns** (4)
                                - `industry_name_ANZSIC`
                                - `rme_size_grp`
                                - `variables`
                                - *(any others if present)*


      - name: `spectrum_table`
      - database: `spectrumdb`
      - schema: `spectrum_schema`
      - table: `spectrum_table`
      - format: `csv`
      - location: `s3://my-awesome-glue-bucket/data-store/custormers_data/csv_report/`
      - next
      - click `Create external table`

---

## create AWS glue crawles
1. Go to AWS Console → Glue → Databases: 
          - add Database:
          - database name: `custormers_data`
          - location: `s3://my-awesome-glue-bucket/data-store/custormers_data/csv_report/`
          - description: `My first Glue DB`
          - create database

2. go to AWS console → glue → create crawler
      - name : `test`
      - description: `test`
          - next
          - Data source: ` Add a data source`
              - Data source type: `S3`
              - S3 path: `s3://my-awesome-glue-bucket/data-store/custormers_data/csv_report/`
                - subsequent crawler runs : ` crawl all sub-folder`
                  - Add an s3 data source
                      - next
                          - IAM role: `Aws-GlueRole`
                            - next
                                - database: `custormers_data`
                                - table name prefix: `spectrum_table`
                                - next
                                - click `Create Crawler`


3. AWS Glue connection
      - go to AWS console → Glue → Data catalog: → Connections:
      - create connection
      - choose data source
      - choose `Amazon Redshift`
      - next
          - connection name: `redshift_connection`
          - next
            - Database instance: `redshift-cluster`
            - Database name: `dev`
            - username: `admin`
            - password: `Yaswanth123`
            - next
            - set properties:
                - Name : Redshift connection
                - Next    
            - click `Create connection`
4. Testing in `Redshift connection`
   - select `Redshift connection` go to Action
   - click `Test connection`
   - IAM role : select `Aws-GlueRole`
   - confirm

5. create ETL job
      - go to AWS console → Glue → Jobs: 
      - create job 
      - click visual ETL
      - source `Amazon s3`
             - in left side click on Data source properties - S3
             - name : `Amazon s3`
             - s3 source type ` Data catalog table`
             - Database : `custormers_data`
             - table name : `test`
      
      - targets ` Amazon Redshift` 
              - in left side click on Data source properties - Amazon Redshift
             - name : `Amazon Redshift`
             - node parents : `Amazon s3`
             - redshift access type : ` Direct data connection`
             - Redshift connection : `redshift_connection`
             - schema : `public`
             - table : `---`
             - handing of data and target table : `marge data into traget table`
             - choose keys and simple action
             - matching keys : `customer_id`
             - save
6. Run job
   - select job name
   - click `Run job`

7. job run moonitoring

8. go run in redshift
  - you can see data in redshift





## ✅ You're Done!
You now have an automated, incremental ETL pipeline using:
- **S3** as source
- **Redshift + Spectrum** as destination
- **SQL logic** for incremental loads
