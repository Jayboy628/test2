# Overcoming EMR Challenges with Cloud-Based Solutions
## Harnessing Cloud Technology for an Efficient Data Warehouse Solution

I worked for a health company that encountered a major issue with their EMR system because it did not align with their business process. In turn, this caused the system to be buggy, as too many custom builds were implemented. The company decided to move away from their current system and instead implemented eClinicalWorks. The EMR company owned the database, so my company had to arrange an amendment to the contract that enables them to extend their usage agreement. The EMR company also agreed to FTP the live data files before work begins at 2:00 am and after work ends at 7:00 pm.

My job was to design and implement a data warehouse from these files. The requirements included creating various production reports and KPIs that matched with the EMR system. The business owners would compare eClinicalWorks integrated reports with my reports and if aligned, they would be flagged to be used for production. In the company’s view, this was critical for data migration because it guaranteed that all operational reports would be correct and more importantly, would prove that eClinicalWorks was configured based on the company’s business requirements.

My intention with this project is to replicate some of the more important aspects of the above scenario. Please note that the healthcare dataset is fake and is being used only for demonstration purposes.

## PHASE ONE: Data Integration and Data Consolidation with Standardization

<details>
  <summary><strong>Extraction Approach: Apache Nifi</strong></summary>
  
  The Ingestion (Apache Nifi) is designed to automate data across systems. In real time, it will load (PutFile) the files into a local database (SQL Server) before pushing the files to the cloud storage (S3) environment.
  
  #### Table of Contents
  - NIFI: Goto [http:/localhost:8443/nifi/](http:/localhost:8443/nifi/)
    - Setup Nifi Environment
      - Installing Nifi Toolkit & Nifi
    - Automate Log parsing
      - INFO
      - DEBUG
      - WARN
      - ERROR
    - Push files to Postges Database
      - parameter-context
        - JSON FILE
      - postgres
        - Create Tables
      - Upload Files
    - Push files to Storage
      - Parameter-Context
        - JSON FILE
      - AWS: S3 Storage
        - Identity and Access Management (IAM)
        - Access Keys
        - Bucket
        - Folder
        - Upload Files
</details>

<details>
  <summary><strong>2) Goto [http:/localhost:8443/nifi/](http:/localhost:8443/nifi/): Files to Postgres Database</strong></summary>
  
  - Nifi Configuration
  - Ingest Files to Postgres Database
  - Move Files to S3 bucket
</details>

<details>
  <summary><strong>3) Move Files to S3 bucket</strong></summary>
  
  - Nifi Configuration
  - Ingest Files to Postgres Database
  - Move Files to S3 bucket
</details>

## PHASE TWO: Reporting and Analytics

<details>
  <summary><strong>Transformation, Documentation: DBT and SQL</strong></summary>
  
  Another requirement was implementing a Data Warehouse that enabled the stakeholders to view and compare the reports and KPIs. Since Data Warehouse usage is mainly for analytical purposes rather than transactional, I decided to design a Star Schema because the structure is less complex and provides better query performance. Documenting wasn’t required, however, adding the Data Build Tool (DBT) to this process allowed us to document each dimension, columns and visualize the Star Schema. DBT also allowed us to neatly organize all data transformations into discrete models.
  
  - DBT: Documentation and Transformation
    - Tables
      - Dimensions
      - Facts
      - SCD
        - Type-1
        - Type-2
    - View
      - DBT (explained in next section)
    - Stored procedure
    - Snow Pipe
    - Stream
    - Task
</details>

<details>
  <summary><strong>Analyze Approach: Language of choice Python and Tableau</strong></summary>
  
  My intention with this project is to replicate some of the more important aspects of the above scenario. <strong>Please note that the healthcare dataset is fake and is being used only for demonstration purposes.</strong>
  
  - Jupyter Lab
    - Data Exploring
    - Data Cleansing
    - Recycle Revenue Reports
  - Tableau Healthcare Reports
    - Revenue Reports
    - PMI Reports
    - CMS Reports
</details>

## PHASE THREE: Models
- Models
