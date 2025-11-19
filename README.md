# explaining what I did in the lab (Part 1)

- In this lab, we used Azure Blob Storage to store data. And Azure Synapse Analytics (serverless SQL pool) to Query
the data. to do so, we:

- created a **storage account** to act as a data lake (climatedatalake60305805).
- added two **containers**:
    - raw: for original CSV files.
    - processed: for data in Parquet format.
- uploaded NOAA GHCN-D CSV files (2023–2025) into the raw container.
- verified the files exist in my storage account (Storage Accounts -> containers -> raw).
- used **Azure Synapse** serverless SQL to query the files directly, as follows:
    - we created an Azure **Synapse workspace** (synapse-60305805).
    - and linked it to our storage account (climatedatalake60305805).
    - in **Synapse Studio**, under develop, we used the SQL script to open the query editor.
    - Then we created a new **logical database** named **ghcn_raw**, to organize and query he raw CSV files in a structured way.
    - we created a **database-scoped credentials using SAS token**, cause Synapse needs credentials to read files from Azure Data Lake Storage.
    - **External data source**: Synapse does not automatically know where our Azure Blob Storage is. So, we created an external data source to point to our storage account and container (CREATE EXTERNAL DATA SOURCE MyBlobStorage). 
    - **External file format**: our files could be CSV, JSON, Parquet, etc. Synapse must know how the data is organized inside each file, the externsl file formate is like instructions telling Synapse how to read the files inside the container (CREATE EXTERNAL FILE FORMAT CsvFormat).
    - **External table**: We want to query the files directly in Synapse using SQL, without copying the data into Synapse. To do so, we create a virtual table in Synapse that points to our files. (CREATE EXTERNAL TABLE raw)
    - with the external table (ghcn_raw.raw) defined, **we can now run SQL queries on it**. <mark>(First Part Of The HW)</mark>

# explaining what I did in the lab (Part 2)

- as CSV format is not efficient for large-scale analytics. We will **convert our raw data into Parquet** (a
columnar, compressed format) which can greatly reduce query I/O. To do so:
    - again, in Synapse Studio, in a new SQL script, we created a new **database** named **ghcn_processed**, where we will register the Parquet table. 
    - In Synapse, credentials are scoped to a database. The credential we created earlier in ghcn_raw does not apply to ghcn_processed. So, again we created a **database-scoped credentials using SAS token** for the new database **ghcn_processed**.
    - then, we created an **external data source** that tells Synapse where to write the Parquet files.
    - also, an **external file format** that defines Parquet with Snappy compression (which is a data compression algorithm developed by Google) for efficient storage and query performance.
    - finally, We used **CETAS** to convert the data to Parquet.
    - we verified everything is working correctly by querying the Parquet table in Synapse.
    - we monitored the **performance** between querying CSV (raw) and Parquet (processed), Parquet query Scans fewer bytes and Runs faster than the CSV query.
    - Now that our data is ingested and optimized, <mark>we will analyze a single station’s time series (USW00094728)</mark>.
    - Directly querying climate_parquet for time series pulls millions of rows, and it is heavy for Jupyter, So:
        - we will **filter and extract ** the data for the station we need to analyze as smaller tables.
        - we used **CETAS** (CREATE EXTERNAL TABLE AS SELECT) to save the two filtered subsets. (prcp_series , tmax_series)
    - We used Python in a **Jupyter notebook** in **Azure Machine Learning**.
    - we connected the notebook to Synapse serverless SQL.
    - we loaded both datasets into Pandas DataFrames.
    - Created **visualizations** to:
        - Display temperature and precipitation over time.
        - Compare yearly differences (2024-2023 and 2025-2024).
    - finally, we repeated the same process for another station of our choice (**USW00093820**) <mark>(Second part of the HW)</mark>


# Where the ETL Steps Happened?

- ETL stands for Extract, Transform, Load. These are the three main steps used when preparing data for analysis. 
    
1. <mark>Extract (Getting the data)</mark>

- This is about bringing in the raw data from its source.
- In our lab:
    - We uploaded raw weather CSV files (2023, 2024, 2025 data) into our Azure Blob Storage.
    - These files came directly from NOAA (the weather data provider).

2. <mark>Transform (Cleaning and preparing the data)</mark>

- This step is about changing the data to make it useful.
- In our lab:
    - We used SQL queries in Azure Synapse to filter, clean, and organize the data.
    - Examples of transformations we did:
        - Filtering data by year.
        - Calculating monthly averages for temperature (TMAX).
        - Calculating total monthly precipitation (PRCP).
        - Selecting only the columns we needed.
        - Storing the cleaned data in Parquet format, which is faster and smaller than raw CSV.

3. <mark>Load (Making data ready for use)</mark>

- This step is about putting the cleaned data into a system where we can analyze it.
- In our project:
    - We created external tables in Synapse that point to the cleaned Parquet files.
    - This let us query the data without copying it again, saving time and storage.
    - Finally, we connected our Python Jupyter Notebook to Synapse and loaded the data into Pandas for visualizations.
