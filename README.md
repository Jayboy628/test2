# <font color="blue"><center>Overcoming EMR Challenges with Cloud-Based Solutions:</center></font>
### <font color="blue"><center>Harnessing Cloud Technology for an Efficient Data Warehouse Solution</center></font>

I worked for a health company that encountered a major issue with their EMR system because it did not align with their business process. In turn, this caused the system to be buggy, as too many custom builds were implemented. The company decided to move away from their current system and instead implemented eClinicalWorks. The EMR company owned the database, so my company had to arrange an amendment to the contract that enables them to extend their usage agreement. The EMR company also agreed to FTP the live data files before work begins at 2:00 am and after work ends at 7:00 pm.

My job was to design and implement a data warehouse from these files. The requirements included creating various production reports and KPI’s that matched with the EMR system. The business owners would compare eClinicalWorks integrated reports with my reports and if aligned, they would be flagged to be used for production. In the company’s view, this was critical for data migration because it guaranteed that all operational reports would be correct and more importantly, would prove that eClinicalWorks was configured based on the company’s business requirements.

My intention with this project is to replicate some of the more important aspects of the above scenario. Please note that the healthcare dataset is fake and is being used only for demonstration purposes.

## <font color="green"><left>PHASE ONE: Data Integration and Data Consolidation with Standardization</left></font>

<details>
  <summary><strong>Extraction Approach: Apache Nifi</strong></summary>

The Ingestion (Apache Nifi) is designed to automate data across systems. In real-time, it will load (PutFile) the files into a local database (SQL Server) before pushing the files to the cloud storage (S3) environment.

#### Table of Content
- NIFI: Goto [http:/localhost:8443/nifi/](http:/localhost:8443/nifi/)
  - Setup Nifi Environment
    - Installing Nifi Toolkit & Nifi
  - Automate Log parsing:
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

<details>
<summary>
    
#### 1) Goto [http:/localhost:8443/nifi/](http:/localhost:8443/nifi/): Setup Nifi Environment
</summary>

- Setup Nifi Environment: I am using a MAC
  - Open Terminal
  - Move to the following folder: `cd /opt`
- Installing Nifi Toolkit: You can download the Apache Nifi [here](https://nifi.apache.org/download.html) or follow these steps:
  - Create the following variables:
    - `export version='1.22.0'`
    - `export nifi_registry_port='18443'` (I am keeping the illustration simple. However, install registry, prod, dev stg is recommended)
    - `export nifi_prd_port='8443'`
  - Download Nifi Toolkit: I am using a MAC and my environment location is `cd/opt`
    - `wget https://dlcdn.apache.org/nifi/${version}/nifi-toolkit-${version}-bin.zip cd /opt`
    - `unzip nifi-toolkit-${version}-bin.zip -d /opt/nifi-toolkit && cd /opt/nifi-toolkit/nifi-toolkit-${version} && mv * .. && cd .. && rm -rf nifi-toolkit-${version}`
  - Configuration Files
  
    Using the variables created above to configure Loop
    ----------------------------------------------------
    
    ```shell
    prop_replace () {
      target_file=${3:-${nifi_props_file}}
      echo 'replacing target file ' ${target_file}
      sed -i -e "s|^$1=.*$|$1=$2|" ${target_file}
    }

    mkdir -p /opt/nifi-toolkit/nifi-envs
    cp /opt/nifi-toolkit/conf/cli.properties.example /opt/nifi-toolkit/nifi-envs/nifi-PRD
    prop_replace baseUrl http://localhost:${nifi_prd_port} /opt/nifi-toolkit/nifi-envs/nifi-PRD
    cp /opt/nifi-toolkit/conf/cli.properties.example /opt/nifi-toolkit/nifi-envs/registry-PRD
    prop_replace baseUrl http://localhost:${nifi_registry_port} /opt/nifi-toolkit/nifi-envs/registry-PRD
    ```
    
    ##### NIFI CLI test
    This utility is used to automate NiFi or NiFi Registry tasks.
    
    - Configure this nifi-PRD
      - Type the following: `cd /opt/nifi-toolkit/nifi-envs`
      - Add the following to `baseUrl`: `baseUrl=http://localhost:8443` (add pic)
    - Type the following and enter Nifi Toolkit env: `/opt/nifi-toolkit/bin/cli.sh`
    - Show Session Keys: `session keys`
      - The config files have the following properties
    - Add session: `session set nifi.props /opt/nifi-toolkit/nifi-envs/nifi-DEV`
    - View current Session: `session show`
    - Find the root PG Id: `nifi get-root-id`
    - List all Process Groups: `nifi pg-list`
    - Find the current user: `nifi current-user`
    - List all available templates: `nifi list-templates`

</details>


<details>
<summary>
  
#### 2) Goto [http:/localhost:8443/nifi/](http:/localhost:8443/nifi/): Automate Log parsing
</summary>

- Setup Nifi Environment: I am using a MAC
  - Open Terminal
  - Move to the following folder: `cd /opt`
- Installing Nifi Toolkit: You can download the Apache Nifi [here](https://nifi.apache.org/download.html) or follow these steps:
  - Type the following variables and click enter:
    - `export version='1.22.0'`
    - `export nifi_registry_port='18443'`
    - `export nifi_prd_port='8443'`
  - Download Nifi Toolkit: I am using a MAC and my environment location is `cd/opt`
    - `wget https://dlcdn.apache.org/nifi/${version}/nifi-toolkit-${version}-bin.zip cd /opt`
    - `unzip nifi-toolkit-${version}-bin.zip -d /opt/nifi-toolkit && cd /opt/nifi-toolkit/nifi-toolkit-${version} && mv * .. && cd .. && rm -rf nifi-toolkit-${version}`
  - Configuration Files
  
    Using the variables created above to configure Loop
    ----------------------------------------------------
    
    ```shell
    prop_replace () {
      target_file=${3:-${nifi_props_file}}
      echo 'replacing target file ' ${target_file}
      sed -i -e "s|^$1=.*$|$1=$2|" ${target_file}
    }

    mkdir -p /opt/nifi-toolkit/nifi-envs
    cp /opt/nifi-toolkit/conf/cli.properties.example /opt/nifi-toolkit/nifi-envs/nifi-PRD
    prop_replace baseUrl http://localhost:${nifi_prd_port} /opt/nifi-toolkit/nifi-envs/nifi-PRD
    cp /opt/nifi-toolkit/conf/cli.properties.example /opt/nifi-toolkit/nifi-envs/registry-PRD
    prop_replace baseUrl http://localhost:${nifi_registry_port} /opt/nifi-toolkit/nifi-envs/registry-PRD
    ```
    
    - The config files have the following properties
      - session keys
      - The config files have the following properties
      - session set nifi.props /opt/nifi-toolkit/nifi-envs/nifi-DEV 
      - export nifi_prd_port='8443'
          
   
</details>

  <details>
<summary>
  
 #### 2) Goto [http:/localhost:8443/nifi/](http:/localhost:8443/nifi/): Files to Postgres Database
</summary>
    
- Nifi Configuration
- Ingest Files to Postgres Database
- Move Files to S3 bucket
</details>
  <details>
<summary>
  
 #### 3) Move Files to S3 bucket
</summary>
    
- Nifi Configuration
- Ingest Files to Postgres Database
- Move Files to S3 bucket
</details>
</details>


<details>
<summary>
    
### Load Approach: Snowflake and SQL
</summary>

<p>
The next step is to populate the cloud database. Snowpipe will pull the normalized JSON files from AWS into tables. As previously stated, the agreement with the EMR company was to FTP the files twice a day. I would be required to configure the load by creating a Task (Acron) and a Stream (CDC). This would enable triggers for a scheduled load and would continuously update the appropriate tables.
</p>

- Snowflake: Database
  - Data Warehouse and SQS Setup
    - Database and Schema
      - Table
        - Type-1
        - Type-2
      - View
        - DBT (explained in the next section)
      - Stored procedure
      - Snowpipe
      - Stream
      - Task

</details>

## <font color="green"><left>PHASE TWO: Reporting and Analytics</left></font>
<details>
<summary>
    
### Transformation, Documentation: DBT and SQL
</summary>

<p>
Another requirement was implementing a Data Warehouse that enabled the stakeholders to view and compare the reports and KPIs. Since Data Warehouse usage is mainly for analytical purposes rather than transactional, I decided to design a Star Schema because the structure is less complex and provides better query performance. Documenting wasn’t required, however, adding the Data Build Tool (DBT) to this process allowed us to document each dimension, columns, and visualize the Star Schema. DBT also allowed us to neatly organize all data transformations into discrete models.
</p>

- DBT: Documentation and Transformation
  - Tables
    - Dimensions
    - Facts
    - SCD
      - Type-1
      - Type-2
    - build operational reports (push to BI Tool)
      
</details>

<details>
<summary>
    
### Analyze Approach: Language of choice Python and Tableau
</summary>

<p>
My intention with this project is to replicate some of the more important aspects of the above scenario. <font color="red">Please note that the healthcare dataset is fake and is being used only for demonstration purposes.</font>
</p>

- Jupyter Lab
  - Data Exploring
  - Data Cleansing
  - Recycle Revenue Reports
- Tableau Healthcare Reports
  - Revenue Reports
  - PMI Reports
  - CMS Reports

</details>

## <font color="green"><left>PHASE THREE</left></font>
* Models
