<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EMR Challenges with Cloud-Based Solutions</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f1f1f1;
            color: #333;
        }

        h1 {
            color: blue;
            text-align: center;
        }

        h3 {
            color: blue;
            text-align: center;
        }

        p {
            text-align: justify;
        }

        h4 {
            color: green;
        }

        details {
            margin-bottom: 20px;
        }

        summary {
            cursor: pointer;
            font-weight: bold;
        }

        details>summary::-webkit-details-marker {
            display: none;
        }

        details>summary::before {
            content: "+";
            display: inline-block;
            margin-right: 5px;
        }

        details[open]>summary::before {
            content: "-";
        }
    </style>
</head>

<body>
    <h1>Overcoming EMR Challenges with Cloud-Based Solutions</h1>
    <h3>Harnessing Cloud Technology for an Efficient Data Warehouse Solution</h3>
    <p>I worked for a health company that encountered a major issue with their EMR system because it did not align with
        their business process. In turn, this caused the system to be buggy, as too many custom builds were implemented.
        The company decided to move away from their current system and instead implemented eClinicalWorks. The EMR company
        owned the database, so my company had to arrange an amendment to the contract that enables them to extend their
        usage agreement. The EMR company also agreed to FTP the live data files before work begins at 2:00 am and after
        work ends at 7:00 pm.<br><br>

        My job was to design and implement a data warehouse from these files. The requirements included creating various
        production reports and KPI’s that matched with the EMR system. The business owners would compare eClinicalWorks
        integrated reports with my reports and if aligned, they would be flagged to be used for production. In the
        company’s view, this was critical for data migration because it guaranteed that all operational reports would be
        correct and more importantly, would prove that eClinicalWorks was configured based on the company’s business
        requirements.<br><br>

        My intention with this project is to replicate some of the more important aspects of the above scenario. Please
        note that the healthcare dataset is fake and is being used only for demonstration purposes.</p>

    <h3>PHASE ONE: Data Integration and Data Consolidation with Standardization</h3>

    <details open>
        <summary>
            <h4>Extraction Approach: Apache Nifi</h4>
        </summary>
        <p>
            The Ingestion (Apache Nifi) is designed to automate data across systems. In real time, it will load (PutFile)
            the files into a local database (SQL Server) before pushing the files to the cloud storage (S3) environment.
        </p>
        <h5>Table of Content</h5>
        <ul>
            <li>NIFI: Goto <a href="http:/localhost:8443/nifi/">http:/localhost:8443/nifi/</a></li>
            <li>Setup Nifi Environment
                <ul>
                    <li>Installing Nifi Toolkit & Nifi</li>
                </ul>
            </li>
            <li>Automate Log parsing
                <ul>
                    <li>INFO</li>
                    <li>DEBUG</li>
                    <li>WARN</li>
                    <li>ERROR</li>
                </ul>
            </li>
            <li>Push files to Postges Database
                <ul>
                    <li>parameter-context
                        <ul>
                            <li>JSON FILE</li>
                        </ul>
                    </li>
                    <li>postgres
                        <ul>
                            <li>Create Tables</li>
                        </ul>
                    </li>
                    <li>Upload Files</li>
                </ul>
            </li>
            <li>Push files to Storage
                <ul>
                    <li>Parameter-Context
                        <ul>
                            <li>JSON FILE</li>
                        </ul>
                    </li>
                    <li>AWS: S3 Storage
                        <ul>
                            <li>Identity and Access Management (IAM)</li>
                            <li>Access Keys</li>
                            <li>Bucket</li>
                            <li>Folder</li>
                            <li>Upload Files</li>
                        </ul>
                    </li>
                </ul>
            </li>
        </ul>
    </details>

    <details open>
        <summary>
            <h4>2) Goto <a href="http:/localhost:8443/nifi/">http:/localhost:8443/nifi/</a>: Files to Postgres
                Database</h4>
        </summary>
        <p>
            1) Nifi Configuration<br>
            2) Ingest Files to Postgres Database<br>
            3) Move Files to S3 bucket
        </p>
    </details>

    <details open>
        <summary>
            <h4>3) Move Files to S3 bucket</h4>
        </summary>
        <p>
            1) Nifi Configuration<br>
            2) Ingest Files to Postgres Database<br>
            3) Move Files to S3 bucket
        </p>
    </details>

    <details open>
        <summary>
            <h3>PHASE TWO: Reporting and Analytics</h3>
        </summary>
        <details open>
            <summary>
                <h4>Transformation, Documentation: DBT and SQL</h4>
            </summary>
            <p>
                Another requirement was implementing a Data Warehouse that enabled the stakeholders to view and compare
                the reports and KPIs. Since Data Warehouse usage is mainly for analytical purposes rather than
                transactional, I decided to design a Star Schema because the structure is less complex and provides
                better query performance. Documenting wasn’t required, however, adding the Data Build Tool (DBT) to
                this process allowed us to document each dimension, columns and visualize the Star Schema. DBT also
                allowed us to neatly organize all data transformations into discrete models.
            </p>
            <h5>DBT: Documentation and Transformation</h5>
            <ul>
                <li>Tables
                    <ul>
                        <li>Dimensions</li>
                        <li>Facts</li>
                        <li>SCD
                            <ul>
                                <li>Type-1</li>
                                <li>Type-2</li>
                            </ul>
                        </li>
                    </ul>
                </li>
                <li>View
                    <ul>
                       
