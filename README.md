# Tutorial: Data virtualization with Azure SQL Managed Instance – preview (Under Construction)

This template allows you to create a a Virtual Machine with SSMS installed which acts as a jump box for accessing Azure SQL Managed instance inside a new virtual network. Storage account is deployed which is required to store data sets in CSV, Parquet or Json format.


# Solution overview and deployed resources. 

In this tutorial, you will deploy resources required for the lab.  

## Target audience

- Data analyst
- Application Developer
- IT Professional
- Cloud Solution Architect

# Architecture

The [Template.json](https://github.com/Ganapathivarma07/LRS-Migration-AzureSQLMI/blob/master/template.json) Azure Resource Manager template will help you automatically deploy the diagram below, which includes:


![alt image]()

- Azure VM with SSMS pre-installed
- Azure SQL managed instance inside a virtual network
- A storage account


[Template.json](https://github.com/Ganapathivarma07/LRS-Migration-AzureSQLMI/blob/master/template.json) can be modified to match your current infrastructure needs.

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

- Allow 30 minutes for the deployment to complete

## Azure services and related products

- Azure Blob storage
- Azure Virutal machine with SSMS installed
- Azure SQL Managed Instance


## Pre-requisites:

 

### SQL Server 


### Azure 



### Azure RBAC permissions


## When to use data virtualization
Typical use cases for data virtualization include:

 

- Providing always up-to-date relational abstraction on top of your raw or disparate data without relocating or transforming it. This way any application capable of running T-SQL queries can consume the data: from BI solutions like Power BI, to line of business applications, to client tools like SQL Server Management Studio or Azure Data Studio. This is an easy and elegant way to expand the list of data sources for your operational reporting solutions.
- Reducing managed instance storage consumption and total cost of ownership (TCO) by archiving cold data to Azure Data Lake Storage, keeping it still within the reach of interactive queries and joins.
- Exploratory data analysis of data sets stored in the most common file formats. This approach is typically used by data scientists and data analysts to collect insights about the data set, from basic ones like number and structure of records, to extracting important variables, detecting outliers and anomalies, and testing underlying assumptions.



## Deployment steps

Getting started
If you are familiar with PolyBase feature of SQL Server, you may have already recognized the scenarios and underlying concepts. Data virtualization capabilities of Azure SQL Managed Instance use the same syntax as PolyBase and enrich it further with new options. PolyBase queries running on your SQL Server instance and targeting files stored in Azure Data Lake Storage or Blob Storage will continue working on your managed instance with minimal intervention to specify location prefix corresponding to the type of external source and endpoint, like abs:// instead of the generic https:// location prefix.

 

To enable data virtualization capabilities on your managed instance, run the following commands: 

exec sp_configure 'polybase_enabled', 1;
go
reconfigure;
go
 

For simplicity, we are going to use publicly available Bing COVID-19 dataset that allows anonymous access.

First, create external data source encapsulating data related to the file location:

 

CREATE EXTERNAL DATA SOURCE DemoPublicExternalDataSource
WITH (
	LOCATION = 'abs://public@pandemicdatalake.blob.core.windows.net/curated/covid-19/bing_covid-19_data/latest' 
)
 

You’re now ready to run the first query:

 

--Number of confirmed Covid-19 cases per country/territory during 2020:
SELECT countries_and_territories, sum(cases) FROM
	OPENROWSET(
        BULK 'abs://public@pandemicdatalake.blob.core.windows.net/curated/covid-19/ecdc_cases/latest/ecdc_cases.parquet',
        FORMAT='PARQUET'
    ) AS [CovidCaseExplorer]
WHERE year = '2020'
group by countries_and_territories
order by sum(cases) desc
 


## Related references


## License & Contribute

You are responsible for the performance, the necessary testing, and if needed any regulatory clearance for any of the models produced by this toolbox.
Please refer [LICENSE](LICENSE) &  [Contribute](https://github.com/Ganapathivarma07/LRS-Migration-AzureSQLMI/blob/master/Contribute.md) for more details


