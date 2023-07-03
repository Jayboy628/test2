<!-- ABOUT THE PROJECT -->

<!-- Still working on the project -->

## <center>Overcoming EMR Challenges with Cloud-Based Solutions:</center>
<br>
<img src="images/main2.png" alt="header" style="width: 900px; height: 400px;"><br>

#### <font color="blue"><em><center>Harnessing Cloud Technology for an Efficient Data Warehouse Solution</em></center></font>
I worked for a health company that encountered a major issue with their EMR system because it did not align with their business process. This led to numerous bugs due to excessive custom builds. As a solution, the company decided to transition to eClinicalWorks, a cloud-based EMR system. However, since the EMR company owned the database, we had to amend the contract to extend our usage agreement. Additionally, the EMR company agreed to FTP the live data files to us before work begins at 2:00 am and after work ends at 7:00 pm.

My role was to design and implement a data warehouse using these files. The requirements included creating various production reports and KPIs that matched the EMR system. The business owners would compare eClinicalWorks integrated reports with my reports, and if they aligned, they would be flagged for production use. This was critical for data migration as it ensured that all operational reports were accurate and validated that eClinicalWorks was configured based on the company's business requirements.
---------------------------------------------------------------------------------------------------------------------
### Agenda

- Cloud-Based Solutions: Healthcare Data Warehouse
  - Ingestion Approach
    - [Installing Nifi Toolkit & Nifi](https://nifi.apache.org/docs/nifi-docs/html/getting-started.html): Setup Nifi Environment
    - Automate Log parsing:
    - Nifi Ingest Data to PostgreSQL Database
    - Nifi Automate PostgreSQL Database to Store JSON File in AWS (S3)
  - FTP Environment:The EMR company push the health data to an Environmet 
  - Staging Database (PostgreSQL)
  - Cloud Storage (S3)
  - Data Warehouse 
  - DBT 
### <font color="green">PHASE ONE: Data Ingestion, Data Storage, Data Warehouse Layers</font>
---------------------------------------------------------------------------------------------------------------------

<details>
  <summary><strong><em>Ingestion Approach: Apache NiFi</em></strong></summary>

The ingestion process involves automating data movement across systems using Apache NiFi. In real-time, the data is loaded into a local database (PostgreSQL) before being pushed to the cloud storage environment (AWS S3).

#### Table of Contents
- [NIFI](http:/localhost:8443/nifi/): Setup Nifi Environment
  - [Installing Nifi Toolkit & Nifi](http:/localhost:8443/nifi/)
  - Automate Log parsing:
    - INFO
    - DEBUG
    - WARN
    - ERROR
  - FTP Environment: The EMR company pushes the health data to an environment 
      - JSON FILE: File (FTP Location) configuration
      - Upload Files
  - Staging Database (PostgreSQL): Ingest files into the database, temporary storage location for data cleansing, validation, and transformation processes
    - parameter-context
      - JSON FILE: Database configuration
    - postgresql
      - Create Tables
      - Load Data 
    - Cloud Storage (S3): Store processed and transformed files
        - Parameter-Context
          - JSON FILE: File configuration
        - AWS (S3)
        - Identity and Access Management (IAM)
        - Access Keys
        - Bucket
        - Folder
        - Load JSON Files

<details>
<summary>
    
##### 1) [NIFI](http:/localhost:8443/nifi/): Setup Nifi Environment
</summary>

- Setup Nifi Environment (I am using a MAC)
  - Open Terminal
  - Move to the following folder: `cd /opt`
- Installing Nifi Toolkit: You can download the Apache Nifi [here](https://nifi.apache.org/download.html) or follow these steps:
  - Create the following variables:
    - `export version='1.22.0'`
    - `export nifi_registry_port='18443'` (I am keeping the illustration simple. However, install registry, prod, dev stg is recommended)
    - `export nifi_prd_port='8443'`
  - Download Nifi Toolkit: I am using a MAC, and my environment location is `cd/opt`
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
    
    ### NIFI CLI STEPS:
    
    <strong>The config files have the following properties</strong>
    -----------------------------------------------------------------------------
    
    - Configure this nifi-PRD
      - Type the following: `cd /opt/nifi-toolkit/nifi-envs`
      - Add the following to `baseUrl`: `baseUrl=http://localhost:8443` 
    - Type the following and enter Nifi Toolkit env: `/opt/nifi-toolkit/bin/cli.sh`
    - Show Session Keys: `session keys`
    - Add session: `session set nifi.props /opt/nifi-toolkit/nifi-envs/nifi-DEV`

    <strong>View the Nifi Environment</strong>
    ---------------------------------------------------------------
     
    - Start Nifi: `/opt/nifi-prd/bin/nifi.sh start` 
    - Start Nifi-toolkit: `/opt/nifi-toolkit/bin/cli.sh`                 
    - View current Session: `session show`
    - Find the root PG Id: `nifi get-root-id`
    - List all Process Groups: `nifi pg-list` (it's empty, but will be used in `Files to Postgres Database` section)
    - Find the current user: `nifi current-user`
    - List all available templates: `nifi list-templates` (it's empty, haven't added any template as yet)

     <strong>Below is a basic view of the Nifi Environment</strong>
    ---------------------------------------------------------------
     
    <img src="images/fileconfig.png" alt="header" style="width: 1000px; height: 700px;"><br> 

</details>


<details>
<summary>
  
##### 2) [NIFI](http:/localhost:8443/nifi/): Automate Log Parsing
</summary>

<strong> Setup Log Parsing inside NIFI</strong>
---------------------------------------------------------------

- Log file location: `/opt/nifi-prd/logs` we can view the log files `nifi-app.log`
- Start Nifi: `/opt/nifi-prd/bin/nifi.sh start` 
- Start Nifi-toolkit: `/opt/nifi-toolkit/bin/cli.sh`
- Go to your Nifi web location: `http:/localhost:8443/nifi/`
    - Drag the Process Group icon onto the plane and name it `Healthcare Data Process`, then double click to open another plane
    - Drag another `Process Group` and name it `LOGS`

<strong> Create the Log Flow in Nifi</strong>
---------------------------------------------------------------

- Drag the `Processor` onto the plane and type `TailFile`, and set the Relationship to success
- Open the TailFile Configure page and click on `SETTINGS` and then click on `Bulletin Level`
    - This will mirror the flow based on the `Bulletin Level`. Then click on `PROPERTIES`
    - In the `Property` column, choose `Tailing mode` with the value `Single file`. In the `File(s) to Tail` column, add the log path
    - **Log file Path**: `/opt/nifi-prd/logs/nifi-app.log`<br><br>

    - TailFile Configure Processor: `Bulletin Level`
    ------------------------------------------
    <img src="images/Bulletin.png" alt="header" style="width: 700px; height: 400px;"> <br>

    - TailFile Configure Processor: `PROPERTIES`
    ------------------------------------------
    <img src="images/TailFile.png" alt="header" style="width: 700px; height: 500px;"> <br>

    - Connect `TailFile` RELATIONSHIPS to Success `SplitText`
    - Configure Processor for `SplitText`: Line Split Count `1` to split the `Bulletin Level type`
        - **Header Line Count**: `0`
        - **Removing Trailing Newlines**: `True`
    - Connect `SplitText` RELATIONSHIPS to Success `RouteOnContent` and Terminate: `failure` and `original`
    - Configure Processor for `RouteOnContent`
        - **Match Requirement**: `content must contain match`
        - **Character Set**: `UTF`
        - **Content Buffer Size**: `1 MB`
        - Click on `+` and manually add the following:
            - DEBUG: connect to LongAttribute
            - ERROR: connect to `ExtractGrok`
            - INFO: connect to LongAttribute
            - WARN: connect to LongAttribute
            - See Below <br>
                <img src="images/AddBulltin.png" alt="header" style="width: 600px; height: 400px;"> <br>
    - Connect `RouteOnContent` RELATIONSHIPS to Success `ExtractGrok` and Terminate: `unmatched`
    - Configure Processor for `ExtractGrok`
        - **Grok Expression**: `%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} \[%{DATA:thread}\] %{DATA:class} %{GREEDYDATA:message}`
        - **Character Set**: `flowfile-attribute`

    - If you have a `Slack` account, connect `RouteOnContent` RELATIONSHIPS to Success `PutSlack`
    - Configure Processor for `RouteOnContent`
        - **Webhook URL**: `Sensitive value set`
        - **Webhook Text**: ` An Error occurred at ${grok.timestamp} with Service ${grok.thread}. Error msg ${grok.message}`
        - Channel: <Your Slack Channel>

    NIFI: LOG DATA FLOW
    ------------------------------------------
    <img src="images/logfile.png" alt="header" style="width: 700px; height: 500px;"> <br>   
            
 
</details>

  <details>
<summary>
  
 ##### 3) [NIFI](http:/localhost:8443/nifi/): Push Files to PostgreSQL Database
</summary>
    
- Incorporating a staging database may seem unnecessary since the files are already standardized. However, there are several benefits to consider. Firstly, it provides cost-effectiveness. Utilizing the cloud for repeated SELECT operations can be expensive. Secondly, the staging database allows for the identification of any unforeseen data issues and enables additional data cleansing and standardization processes. The ultimate goal is to minimize the number of updates and inserts into Snowflake, ensuring optimal efficiency.
- **FTP LOCATION**: I used a Python script to create a `timestamp` and `increment count` for each file.
  - **Python Script**: [Script](code): I also implemented `Slack` to notify me when the file reaches `2 AM before work and 7 PM`
  - To integrate the Incoming `Webhooks` feature into the code, you'll need to make the following modifications:
    1. Install the slack_sdk library if you haven't already: `pip install slack_sdk`
    2. Import the necessary modules: `from slack_sdk import WebClient`, `from slack_sdk.errors import SlackApiError`
    3. Set up the Slack webhook URL: `slack_webhook_url = 'YOUR_SLACK_WEBHOOK_URL'`: Click here to view the script [Script](code)

- Automate configuration file within parameter-context 
    - **Create two folders**: Process-Nifi and parameter_context
    - /opt/nifi-toolkit/nifi-envs/`Process-Nifi/parameter_context` and add the files [`postgres-config.json`](parameter-context) to the folder
    - **Start Nifi-toolkit**: `/opt/nifi-toolkit/bin/cli.sh`
    - **Create the parameter Context for the database**:
    `nifi import-param-context -i /opt/nifi-toolkit/nifi-envs/Excel-NiFi/parameter_context/postgres-config.json' -u http://localhost:8443`
    - **Create the parameter Context for the file Tracker**:
    `nifi import-param-context -i /opt/nifi-toolkit/nifi-envs/Excel-NiFi/parameter_context/excell-healthcare-tracker-config.json' -u http://localhost:8443`
    - **Go to your Nifi web location**: `http:/localhost:8443/nifi/`
    - **Open Nifi**: In the top-right corner, click the icon and click on `Parameter Contexts` to confirm that the above files are loaded
    - **Global Gear**: Click on it and search in the `Process Group Parameter Context` for your loaded files and click apply
        - Drag the Process Group icon onto the plane and name it `Healthcare Data Process`, then double click to open another plane
        - Drag another `Process Group` and name it `File Extraction to Databases`
            - Click the process group `File Extraction to Database` and then Drag the Processor and type `List File`
                - In the ListFile processor, the file configuration should be loaded automatically
                - **Input Directory**: `#{source_directory}`
                - **File Filter**: `#{file_list}`
                - **Entity Tracking Node Identifier**: `${hostname()}`

            - Drag the Processor and type `FetchFile`
                - **File to Fetch**: `${absolute.path}/${filename}`
                - **Move Conflict Strategy**: `Rename`
            
            - Drag the Processor and type `ConvertRecord`: Read CSV files and convert them to `JSON`
                - **Record Reader**: `CSVReader`: we need to configure a `Controller Service Details`, click on `properties`
                    - **Schema Access Strategy**: `Inherit Record Schema`
                    - **Reader Schema**: click on the `+` icon and manually input the schema
                - **Record Writer**: `JsonRecordSetWriter`: we need to configure a `Controller Service Details`, click on `properties`
                    - **Schema Write Strategy**: `Use Schema Name Property`
                    - **Schema Name**: `schema.name`
                    - **Pretty Print JSON**: `true`
                    - **Schema Access Strategy**: `Inherit Record Schema`
            
            - Drag the Processor and type `AttributesToJson`
                - **Include Core Attributes**: `false`
                - **Include Dynamic Attributes**: `true`
                - **Include All Attributes**: `true`
                - **Overwrite Existing Attributes**: `false`
                - **Remove Attributes**: `output.data` 
            - Drag the Processor and type `EvaluateJsonPath`
                - **Attributes to JSON**: `true`
                - **Destination**: `flowfile-content`
                - **Null Value**: `null`
                - **Replacement Value**: `null`
                - **Include Root Group**: `false`
            
            - Drag the Processor and type `PutDatabaseRecord`
                - **PutDatabaseRecord**: we need to configure a `Controller Service Details`, click on `properties`
                    - **Database Connection Pooling Service**: `Database Connection Pooling Service`: Click on `controller service details` and configure it
                        - **Database Connection URL**: `jdbc:postgresql://localhost:5432/postgres`
                        - **Database Driver Class Name**: `org.postgresql.Driver`
                        - **Database Driver Location(s)**: `/opt/nifi-prd/drivers/postgresql-42.3.1.jar`
                        - **Database User**: `admin`
                        - **Database Password**: `admin`
                - **Table Name**: `t_file_info`
                - **Max Rows Per Flow File**: `1`
                - **Statement Type**: `Use Statement Attribute`
                - **Statement Attribute Name**: `dml.type`

                - **PutDatabaseRecord**: `Properties`
                ----------------------------------------
                <img src="images/PutDBRecord.png" alt="header" style="width: 700px; height: 400px;"> <br>  

            - Drag the Processor and type `ReplaceText`
                - **Replacement Strategy**: `Literal Replace`
                - **Search Value**: `insert`
                - **Replacement Value**: `select * from`
                - **Evaluate Repl. Value**: `false`
                - **Replacement Strategy**: `Literal Replace`
                - **Search Value**: `into`
                - **Replacement Value**: ``
                - **Evaluate Repl. Value**: `false`
            - Drag the Processor and type `ExecuteSQL`
                - **Database Connection Pooling Service**: `Database Connection Pooling Service`: Click on `controller service details` and configure it
                - **Table Name**: `t_file_info`
                - **Max Rows Per Flow File**: `1`
                - **Statement Type**: `Use Statement Attribute`
                - **Statement Attribute Name**: `dml.type`
                - **SQL select**: `SELECT * FROM t_file_info`
                
            - Drag the Processor and type `ListDatabaseTables`
                - **DB connection pool**: `Database Connection Pooling Service`: Click on `controller service details` and configure it

        FLOW FILES: Process Group Flow
        ----------------------------------------
        <img src="images/database.png" alt="header" style="width: 1000px; height: 700px;"> <br>

</details>

<details>
<summary>
  
##### 4) [NIFI](http:/localhost:8443/nifi/): Store Processed Files in AWS S3
</summary>

- Drag the Processor and type `PutS3Object`
    - **Access Key**: `AWS Access Key ID`
    - **Secret Key**: `AWS Secret Access Key`
    - **Bucket**: `your-bucket-name`
    - **Folder**: `${file_ftp_source_path}`

AWS S3 CONFIGURATION
----------------------------------------
- **Create a User**:
    - Login to the `AWS Management Console`
    - In the search bar, type `IAM` and click on `IAM (Identity and Access Management)`
    - Click on `Users` from the left-hand menu and then click on `Add User`
    - Enter a name for the user and select `Programmatic access` for the `Access type`
    - Click on `Next: Permissions` and then select `Attach existing policies directly`
    - Search for and select the `AmazonS3FullAccess` policy
    - Click on `Next: Tags` (optional) and then click on `Next: Review`
    - Review the user details and click on `Create user`
    - Take note of the `Access key ID` and `Secret access key` as you will need them in the Nifi configuration

- **Create an S3 Bucket**:
    - Go to the `AWS Management Console`
    - In the search bar, type `S3` and click on `S3`
    - Click on `Create bucket`
    - Enter a unique name for the bucket and choose the region
    - Click on `Next` and leave the rest of the settings as default
    - Click on `Next` and review the bucket settings
    - Click on `Create bucket`

NIFI: AWS S3 CONFIGURATION
----------------------------------------
- Click on the `gear` icon in the top-right corner and select `Controller Settings`
- Click on the `AWS S3` tab
- Enter the `Access Key ID` and `Secret Access Key` of the IAM user created earlier
- Click on `Test AWS S3 Credentials` to verify the connection
- Click on `Apply` and then `OK`

NIFI: S3 PROCESSOR CONFIGURATION
----------------------------------------
- **PutS3Object** Processor: Click on the processor and go to the `Properties` tab
- Enter the `Bucket` name created earlier
- Set the `Folder` property to `${file_ftp_source_path}` (assuming `file_ftp_source_path` is a flowfile attribute that contains the desired folder path)
- Configure any other properties as needed (e.g., access key, secret key, region, etc.)
- Connect the processor to the previous processor in your flow to continue the flow of data

NIFI: S3 PROCESSOR CONFIGURATION
----------------------------------------
- **PutS3Object** Processor: Click on the processor and go to the `Properties` tab
- Enter the `Bucket` name created earlier
- Set the `Folder` property to `${file_ftp_source_path}` (assuming `file_ftp_source_path` is a flowfile attribute that contains the desired folder path)
- Configure any other properties as needed (e.g., access key, secret key, region, etc.)
- Connect the processor to the previous processor in your flow to continue the flow of data

FLOW FILES: Process Group Flow
----------------------------------------
<img src="images/aws-s3.png" alt="header" style="width: 1000px; height: 700px;"> <br>
            
</details>
<br>

### <font color="green">PHASE TWO: Data Validation, Data Cleansing, and Transformation</font>
---------------------------------------------------------------------------------------------------------------------

<details>
<summary>
  
##### 5) [NIFI](http:/localhost:8443/nifi/): Data Validation and Cleansing
</summary>

- Drag the Processor and type `PutDatabaseRecord`
    - **PutDatabaseRecord**: we need to configure a `Controller Service Details`, click on `properties`
        - **Database Connection Pooling Service**: `Database Connection Pooling Service`: Click on `controller service details` and configure it
            - **Database Connection URL**: `jdbc:postgresql://localhost:5432/postgres`
            - **Database Driver Class Name**: `org.postgresql.Driver`
            - **Database Driver Location(s)**: `/opt/nifi-prd/drivers/postgresql-42.3.1.jar`
            - **Database User**: `admin`
            - **Database Password**: `admin`
    - **Table Name**: `t_file_info`
    - **Max Rows Per Flow File**: `1`
    - **Statement Type**: `Use Statement Attribute`
    - **Statement Attribute Name**: `dml.type`

    - **PutDatabaseRecord**: `Properties`
    ----------------------------------------
    <img src="images/PutDBRecord.png" alt="header" style="width: 700px; height: 400px;"> <br>  

- Drag the Processor and type `ReplaceText`
    - **Replacement Strategy**: `Literal Replace`
    - **Search Value**: `insert`
    - **Replacement Value**: `select * from`
    - **Evaluate Repl. Value**: `false`
    - **Replacement Strategy**: `Literal Replace`
    - **Search Value**: `into`
    - **Replacement Value**: ``
    - **Evaluate Repl. Value**: `false`
    
- Drag the Processor and type `ExecuteSQL`
    - **Database Connection Pooling Service**: `Database Connection Pooling Service`: Click on `controller service details` and configure it
    - **Table Name**: `t_file_info`
    - **Max Rows Per Flow File**: `1`
    - **Statement Type**: `Use Statement Attribute`
    - **Statement Attribute Name**: `dml.type`
    - **SQL select**: `SELECT * FROM t_file_info`
    
- Drag the Processor and type `ListDatabaseTables`
    - **DB connection pool**: `Database Connection Pooling Service`: Click on `controller service details` and configure it

FLOW FILES: Process Group Flow
----------------------------------------
<img src="images/database.png" alt="header" style="width: 1000px; height: 700px;"> <br>

</details>

<details>
<summary>
  
##### 6) [NIFI](http:/localhost:8443/nifi/): Data Transformation
</summary>

- Drag the Processor and type `PutSQL`
    - **Database Connection Pooling Service**: `Database Connection Pooling Service`: Click on `controller service details` and configure it
    - **Table Name**: `t_file_info`
    - **Max Rows Per Flow File**: `1`
    - **Statement Type**: `Use Statement Attribute`
    - **Statement Attribute Name**: `dml.type`
    - **SQL select**: `SELECT * FROM t_file_info`
    - **SQL insert**: `SELECT * FROM t_file_info`
    
- Drag the Processor and type `ReplaceText`
    - **Replacement Strategy**: `Literal Replace`
    - **Search Value**: `insert`
    - **Replacement Value**: `select * from`
    - **Evaluate Repl. Value**: `false`
    - **Replacement Strategy**: `Literal Replace`
    - **Search Value**: `into`
    - **Replacement Value**: ``
    - **Evaluate Repl. Value**: `false`
    
- Drag the Processor and type `ExecuteSQL`
    - **Database Connection Pooling Service**: `Database Connection Pooling Service`: Click on `controller service details` and configure it
    - **Table Name**: `t_file_info`
    - **Max Rows Per Flow File**: `1`
    - **Statement Type**: `Use Statement Attribute`
    - **Statement Attribute Name**: `dml.type`
    - **SQL select**: `SELECT * FROM t_file_info`

- Drag the Processor and type `ListDatabaseTables`
    - **DB connection pool**: `Database Connection Pooling Service`: Click on `controller service details` and configure it

FLOW FILES: Process Group Flow
----------------------------------------
<img src="images/transformation.png" alt="header" style="width: 1000px; height: 700px;"> <br>

</details>

<br>

### <font color="green">PHASE THREE: Reporting and KPIs</font>
---------------------------------------------------------------------------------------------------------------------

<details>
<summary>
  
##### 7) [NIFI](http:/localhost:8443/nifi/): Generate Reports and KPIs
</summary>

- Drag the Processor and type `PutSQL`
    - **Database Connection Pooling Service**: `Database Connection Pooling Service`: Click on `controller service details` and configure it
    - **Table Name**: `t_file_info`
    - **Max Rows Per Flow File**: `1`
    - **Statement Type**: `Use Statement Attribute`
    - **Statement Attribute Name**: `dml.type`
    - **SQL select**: `SELECT * FROM t_file_info`
    - **SQL insert**: `SELECT * FROM t_file_info`

- Drag the Processor and type `ReplaceText`
    - **Replacement Strategy**: `Literal Replace`
    - **Search Value**: `insert`
    - **Replacement Value**: `select * from`
    - **Evaluate Repl. Value**: `false`
    - **Replacement Strategy**: `Literal Replace`
    - **Search Value**: `into`
    - **Replacement Value**: ``
    - **Evaluate Repl. Value**: `false`
    
- Drag the Processor and type `ExecuteSQL`
    - **Database Connection Pooling Service**: `Database Connection Pooling Service`: Click on `controller service details` and configure it
    - **Table Name**: `t_file_info`
    - **Max Rows Per Flow File**: `1`
    - **Statement Type**: `Use Statement Attribute`
    - **Statement Attribute Name**: `dml.type`
    - **SQL select**: `SELECT * FROM t_file_info`

- Drag the Processor and type `ListDatabaseTables`
    - **DB connection pool**: `Database Connection Pooling Service`: Click on `controller service details` and configure it

FLOW FILES: Process Group Flow
----------------------------------------
<img src="images/reports.png" alt="header" style="width: 1000px; height: 700px;"> <br>

</details>

<br>

### <font color="green">PHASE FOUR: Data Migration and Validation</font>
---------------------------------------------------------------------------------------------------------------------

<details>
<summary>
  
##### 8) [NIFI](http:/localhost:8443/nifi/): Data Migration and Validation
</summary>

- Drag the Processor and type `PutSQL`
    - **Database Connection Pooling Service**: `Database Connection Pooling Service`: Click on `controller service details` and configure it
    - **Table Name**: `t_file_info`
    - **Max Rows Per Flow File**: `1`
    - **Statement Type**: `Use Statement Attribute`
    - **Statement Attribute Name**: `dml.type`
    - **SQL select**: `SELECT * FROM t_file_info`
    - **SQL insert**: `SELECT * FROM t_file_info`

- Drag the Processor and type `ReplaceText`
    - **Replacement Strategy**: `Literal Replace`
    - **Search Value**: `insert`
    - **Replacement Value**: `select * from`
    - **Evaluate Repl. Value**: `false`
    - **Replacement Strategy**: `Literal Replace`
    - **Search Value**: `into`
    - **Replacement Value**: ``
    - **Evaluate Repl. Value**: `false`
    
- Drag the Processor and type `ExecuteSQL`
    - **Database Connection Pooling Service**: `Database Connection Pooling Service`: Click on `controller service details` and configure it
    - **Table Name**: `t_file_info`
    - **Max Rows Per Flow File**: `1`
    - **Statement Type**: `Use Statement Attribute`
    - **Statement Attribute Name**: `dml.type`
    - **SQL select**: `SELECT * FROM t_file_info`

- Drag the Processor and type `ListDatabaseTables`
    - **DB connection pool**: `Database Connection Pooling Service`: Click on `controller service details` and configure it

FLOW FILES: Process Group Flow
----------------------------------------
<img src="images/migration.png" alt="header" style="width: 1000px; height: 700px;"> <br>

</details>

<br>

### <font color="green">PHASE FIVE: Production Use</font>
---------------------------------------------------------------------------------------------------------------------

<details>
<summary>
  
##### 9) [NIFI](http:/localhost:8443/nifi/): Production Use
</summary>

- Drag the Processor and type `PutSQL`
    - **Database Connection Pooling Service**: `Database Connection Pooling Service`: Click on `controller service details` and configure it
    - **Table Name**: `t_file_info`
    - **Max Rows Per Flow File**: `1`
    - **Statement Type**: `Use Statement Attribute`
    - **Statement Attribute Name**: `dml.type`
    - **SQL select**: `SELECT * FROM t_file_info`
    - **SQL insert**: `SELECT * FROM t_file_info`

- Drag the Processor and type `ReplaceText`
    - **Replacement Strategy**: `Literal Replace`
    - **Search Value**: `insert`
    - **Replacement Value**: `select * from`
    - **Evaluate Repl. Value**: `false`
    - **Replacement Strategy**: `Literal Replace`
    - **Search Value**: `into`
    - **Replacement Value**: ``
    - **Evaluate Repl. Value**: `false`
    
- Drag the Processor and type `ExecuteSQL`
    - **Database Connection Pooling Service**: `Database Connection Pooling Service`: Click on `controller service details` and configure it
    - **Table Name**: `t_file_info`
    - **Max Rows Per Flow File**: `1`
    - **Statement Type**: `Use Statement Attribute`
    - **Statement Attribute Name**: `dml.type`
    - **SQL select**: `SELECT * FROM t_file_info`

- Drag the Processor and type `ListDatabaseTables`
    - **DB connection pool**: `Database Connection Pooling Service`: Click on `controller service details` and configure it

FLOW FILES: Process Group Flow
----------------------------------------
<img src="images/production.png" alt="header" style="width: 1000px; height: 700px;"> <br>

</details>

