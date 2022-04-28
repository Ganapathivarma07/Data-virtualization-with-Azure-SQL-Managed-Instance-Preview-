# Tutorial: Data virtualization with Azure SQL Managed Instance – preview (Under Construction)

This template allows you to create a a Virtual Machine with SSMS installed which acts as a jump box for accessing Azure SQL Managed instance inside a new virtual network. Storage account is deployed which is required to store data sets in CSV, Parquet or Json format.


# Solution overview and deployed resources. 

Data virtualization with Azure SQL Managed Instance allows you to execute Transact-SQL (T-SQL) queries against data from files stored in Azure Data Lake Storage Gen2 or Azure Blob Storage, and combine it with locally stored relational data using joins. This way you can transparently access external data while keeping it in its original format and location - also known as data virtualization.

## When to use data virtualization
Typical use cases for data virtualization include:

- Providing always up-to-date relational abstraction on top of your raw or disparate data without relocating or transforming it. This way any application capable of running T-SQL queries can consume the data: from BI solutions like Power BI, to line of business applications, to client tools like SQL Server Management Studio or Azure Data Studio. This is an easy and elegant way to expand the list of data sources for your operational reporting solutions.
- Reducing managed instance storage consumption and total cost of ownership (TCO) by archiving cold data to Azure Data Lake Storage, keeping it still within the reach of interactive queries and joins.
- Exploratory data analysis of data sets stored in the most common file formats. This approach is typically used by data scientists and data analysts to collect insights about the data set, from basic ones like number and structure of records, to extracting important variables, detecting outliers and anomalies, and testing underlying assumptions.

# Architecture

The [Template.json](https://github.com/Ganapathivarma07/LRS-Migration-AzureSQLMI/blob/master/template.json) Azure Resource Manager template will help you automatically deploy the diagram below, which includes:

- Azure VM with SSMS pre-installed
- Azure SQL managed instance inside a virtual network
- A storage account

Using this template, you can quickly deploy resources and then follow below step by step instructions to query parquet/CSV files in Azure Data lake using T-SQL Syntax.

[Template.json](https://github.com/Ganapathivarma07/LRS-Migration-AzureSQLMI/blob/master/template.json) can be modified to match your current infrastructure needs.


![alt image](https://github.com/Ganapathivarma07/Data-virtualization-with-Azure-SQL-Managed-Instance-Preview-/blob/master/Images/data%20virtualization%20image%20github.png)

## Target audience

- Data analyst
- Application Developer
- IT Professional
- Cloud Solution Architect


## One Click Deploying Template
<!-- Powershell command for Translating Git URL for template.json
    $url = "https://raw.githubusercontent.com/Ganapathivarma07/LRS-Migration-AzureSQLMI/master/template.json"
    [uri]::EscapeDataString($url)
    >> uri = https%3A%2F%2Fgithub.com%2FGanapathivarma07%2FLRS-Migration-AzureSQLMI%2Fblob%2F
master%2Ftemplate.json

Base URL: https://portal.azure.com/#create/Microsoft.Template/uri
Final URL: <Base URL>/<uri>
-->
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FGanapathivarma07%2FLRS-Migration-AzureSQLMI%2Fmaster%2Ftemplate.json)

## Deploying an ARM Template using the Azure portal

- Visit https://portal.azure.com

- Allow all the resources in the deployment to complete
- Create Azure blob storage or Data lake gen2 containter
- Copy the postalcodes.parquet & populatio.csv dataset to Azure Blob storage
- To access a private location, use a Shared Access Signature (SAS) with proper access permissions and validity period to authenticate to the storage account.


## Azure services and related products

- Azure Blob storage
- Azure Virutal machine with SSMS installed
- Azure SQL Managed Instance


### Azure SQLManaged Instance

- To use the BULK option requires ADMINISTER BULK OPERATIONS or ADMINISTER DATABASE BULK OPERATIONS permission

### Azure Blob storage

- Ensure that SAS token has the appropriate time validity taking time zones into consideration for the entire duration of your migration.
- Ensure Read and List only permissions are selected

### Azure RBAC permissions

- Subscription Owner role, or Azure operator needs to have the Managed Instance Contributor RBAC Role, or
- Custom role with the following permission: Microsoft.Sql/managedInstances/databases/*


## Deployment steps

If you are familiar with PolyBase feature of SQL Server, you may have already recognized the scenarios and underlying concepts. Data virtualization capabilities of Azure SQL Managed Instance use the same syntax as PolyBase and enrich it further with new options. PolyBase queries running on your SQL Server instance and targeting files stored in Azure Data Lake Storage or Blob Storage will continue working on your managed instance with minimal intervention to specify location prefix corresponding to the type of external source and endpoint, like abs:// instead of the generic https:// location prefix.

```
--Blob Storage endpoint
abs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>.parquet

--Data Lake endpoint
adls://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>.parquet
```

## To enable data virtualization capabilities on your managed instance, run the following commands: 

```sql
exec sp_configure 'polybase_enabled', 1;
go
reconfigure;
go
```

## To access a dataset in public location:

For simplicity, we are going to use publicly available Bing COVID-19 dataset that allows anonymous access.

First, create external data source encapsulating data related to the file location:

```sql
CREATE EXTERNAL DATA SOURCE DemoPublicExternalDataSource
WITH (
	LOCATION = 'abs://public@pandemicdatalake.blob.core.windows.net/curated/covid-19/bing_covid-19_data/latest' 
)
``` 

You’re now ready to run the first query:

--Number of confirmed Covid-19 cases per country/territory during 2020:
	
```sql
SELECT countries_and_territories, sum(cases) FROM
	OPENROWSET(
        BULK 'abs://public@pandemicdatalake.blob.core.windows.net/curated/covid-19/ecdc_cases/latest/ecdc_cases.parquet',
        FORMAT='PARQUET'
    ) AS [CovidCaseExplorer]
WHERE year = '2020'
group by countries_and_territories
order by sum(cases) desc
``` 

## To access a dataset in private location, include the file path and credential when querying the external data source:


## Step 0 (optional): Create master key if it doesn't exist in the database:

```sql
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'sasxansv34dtws4g5643634$'
GO
```

## Step 1: Create database-scoped credential (requires database master key to exist):
	
```sql
Use RecoveryDB
GO
CREATE DATABASE SCOPED CREDENTIAL [DemoCredential]
WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
SECRET = 'sv=2020-10-02&st=2022-04-04T20%3A01%3A25Z&se=2022-04-08T20%3A01%3A00Z&sr=c&sp=rl&sig=pOjEPffk3dCUTKlZuGEKEAMLEED%2FGtXhU%3D';
GO
```
	
## Step 2: Create external data source pointing to the file path, and referencing database-scoped credential:

```sql
CREATE EXTERNAL DATA SOURCE DemoPrivateExternalDataSource
WITH (
	LOCATION = 'abs://Demodata@demosqlmi.blob.core.windows.net/',
    CREDENTIAL = [DemoCredential] 
)
```

## Step 3: Query Parquet format data sources using OPENROWSET

```sql
SELECT TOP 1000 *
FROM OPENROWSET(
 BULK 'postalcodes.parquet',
 DATA_SOURCE = 'DemoPrivateExternalDataSource',
 FORMAT = 'parquet'
) AS filerows
```

## Step 4: Query CSV format data sources using OPENROWSET
	
```sql
SELECT TOP 100 *
FROM OPENROWSET(
BULK 'population.csv',
DATA_SOURCE = 'DemoPrivateExternalDataSource',
FORMAT = 'CSV')
WITH (
[country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
[country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
[year] smallint,
[population] bigint
) AS filerows
```

## Creainge view on top of OPENROWSET
	
```sql
CREATE VIEW Demoview AS
SELECT TOP 100 *
FROM OPENROWSET(
BULK 'population.csv',
DATA_SOURCE = 'DemoPrivateExternalDataSource',
FORMAT = 'CSV')
WITH (
[country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
[country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
[year] smallint,
[population] bigint
) AS filerows
```

## Query CSV format data source from view

```sql
select * from Demoview
```

## Step 5: Create external file format

```sql
CREATE EXTERNAL FILE FORMAT DemoDataFileFormat
WITH (
FORMAT_TYPE=DELIMITEDTEXT,
FORMAT_OPTIONS(
FIELD_TERMINATOR = ',',
STRING_DELIMITER = '"')
)
GO
```

## Step 6: Create external table to get local query experience:

```sql	
CREATE EXTERNAL TABLE tbl_population(
[country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
[country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
[year] smallint,
[population] bigint
)
WITH (
LOCATION = 'population.csv',
DATA_SOURCE = DemoPrivateExternalDataSource,
FILE_FORMAT = DemoDataFileFormat,
);
GO
```

## Step 7: Once the external table is created, you can query it just like any other table:

/****** Script for SelectTopNRows command from SSMS  ******/
	
```sql	
SELECT TOP (1000) [country_code]
      ,[country_name]
      ,[year]
      ,[population]
  FROM [RecoveryDB].[dbo].[tbl_population]
```

## Related references

1. https://docs.microsoft.com/en-us/azure/azure-sql/managed-instance/data-virtualization-overview?view=azuresql
2. https://azure.microsoft.com/en-in/products/azure-sql/managed-instance/
3. https://docs.microsoft.com/en-us/sql/t-sql/functions/openrowset-transact-sql?view=sql-server-ver15

## License & Contribute

You are responsible for the performance, the necessary testing, and if needed any regulatory clearance for any of the models produced by this toolbox.
Please refer [LICENSE](LICENSE) &  [Contribute](https://github.com/Ganapathivarma07/LRS-Migration-AzureSQLMI/blob/master/Contribute.md) for more details


