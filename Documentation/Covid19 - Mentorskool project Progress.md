# Covid19 - Mentorskool project Progress

**Resources**


1. Data Catalog - https://docs.google.com/spreadsheets/d/12h8WQmD6u6IPCW63Y23zZR6NdCVO9396ANv8cVa6bCk/

1. Orginal Data Dictionary - https://drive.google.com/file/d/1f1B8pALq46zinX31Gne8K_ESqCDoQ1BR/

2. Modified data dictionary (for data that has an update cycle update, we can modify the above 					accordingly.) - https://docs.google.com/spreadsheets/d/1Ov7SQrRUI7CebOp2R2Rho6mOBri93fktmejR0klXsjI/

3. Duplicate Modified Data dictionary - https://docs.google.com/spreadsheets/d/1Ov7SQrRUI7CebOp2R2Rho6mOBri93fktmejR0klXsjI/

**Plan**

- [x] Have a data pipeline that provides access to a SQL server to the Analysts
- [ ] Collaborate on Analysis - Present findings.

# Data Pipeline – Steps for creation.

## 1. Raw Data Ingestion to GCS Bucket Cloud Function  - Deployed

 Gets triggered by a HTTP Request using cron job/cloud scheduler 
 >GCP_Cloud_function.py \
 deployed as – fetch_raw_covid_api_data

	
	Note - If you add new data sources that are to be fetched on a schedule. Update this. 

HTTP Trigger (No authentication Dont share)-\
 https://us-central1-covid19-india-analysis-284814.cloudfunctions.net/fetch_raw_covid_api_data 


	
	GCP Logs - "Function execution started"
			"Downloaded Daily Stats CSV at '/tmp/COVID_India_National.csv'"
		   "Downloaded Daily States Stats CSV at '/tmp/COVID_India_National.csv'"   
		   "Downloaded Daily Testing tats CSV at '/tmp/COVID_India_National.csv'"  
		   "Uploaded ['COVID_India_National.csv', 'COVID_India_State.csv', 'COVID_India_Test_data.csv'] to "mskl-data-lake" bucket."   
		   "Function execution took 4718 ms, finished with status code: 200"    

## 2. Create PostgreSQL CloudSQL instance and create covid-19-india server 
	```
    # Details
    instance id - covid19-india
	cloud_sql_connection_name = ***REMOVED***:***REMOVED***:covid19-india

	# Credentials
	db_user = 'postgres'
	db_password = '***REMOVED***'
	cloud_sql_ip = '***REMOVED***'
	db_name = 'covid19-india'
	
    2nd User - client 
	password - ***REMOVED***

	# Specs of deployed Instance (all lowest possible) : 
	machine selected - 1 shared vCPU
	0.6GB RAM
	10GB HDD
	NO Storage Updates
	NO automatic backups
	Single Zone
    ```

### Two Options to create Database : 
	a. pg_dump of local server.
		1. Upload dump of local SQL server to Bucket - Cleaned folder.
		2. Import that dump to create Schema on Instance overview page. 
	
    b. Connect to SQL server instance and database using CloudSQL proxy and SQL Alchemy and use SQL alchemy to update schema. (see below)

### When Adding new data sources to SQL Server/Modifying Schema, modify using starter notebook:
>Data_Ingestion_Pipeline_Starter.ipynb

## 3. Connecting to CloudSQL Server:
Two ways to connect - 
1. Cloud SQL Proxy 
2. Authorised public IP

Details in starter notebook:
>Query_Cloud_SQL_Starter_Notebook.ipynb

## 4. Cloud Function To Add Cleaned Data to SQL server 
Loads the raw files from the buckets, cleans it and Uploads to CloudSQL Server. \
Triggered by a HTTP Request using Cron Job/Cloud scheduler 
 >Cloud_function_Ingestion_SQL.py \
 deployed as – ingestion-clean-cloud-sql

Trigger (No authentication Dont share) -\
 https://***REMOVED***-covid19-india-analysis-284814.cloudfunctions.net/ingestion-clean-cloud-sql 

	GCP Logs- Function execution started
		199 Records in overall_stats
		Added 0 Records to table
		6045 Records in states_info
		Added 39 Records to table
		144 Records in testing_stats
		Added 1 Records to table
		Function execution took 2736 ms, finished with status code: 200


**Steps for Permissions to connect with cloudSQl and bucket** : 
Assigned Service account ***REMOVED***@appspot.gserviceaccount.com\
with **Cloud SQL Admin, Storage Admin Roles**

## 5. Created Cron Jobs that call [1] and [4] on a schedule using Google cloud sceduler.
>schedule-fetch_raw_covid_api_data\
schedule-clean-cloud-sql-ingestion

These run at 02:00 and 02:30 respectively.\
The cloud function have unauthorized access enabled for the cron jobs.
	
## 6. Modifications required when adding new data sources :

    a. (Modify Cloud function in #1)
    - Download the raw CSV into GCS bucket using cloud function if data Updates from External Source.
    – If snapshot data, simply store in GCS bucket.

	b.(Modify Cloud Function in #4)
    – Read filefrom GCS bucket
    – clean it and make sure it is acc. to schema of table in DB:
	– add_data_table(engine, 'table_name', Pandas_dataframe_ cleaned for ingestion)

