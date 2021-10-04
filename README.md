# ADF-ETL-Pipeline

# AIM:
Design of an ETL pipeline in Azure Data Factory, to copy data from Azure SQL Server DB tables, transform it and finally load into ADLS Blob container.

# Source:
- Server: Azure SQL Server (East US)
- Database: AdventureWorksLT sample Database
- Tables: SalesLT.Product & SalesLT.SalesOrderDetail

# Target:
- Azure Data Lake Storage Blob Container (East US, LRS)
- Container: raw (Sample created)

# Staging Location:
- Azure Data Lake Storage Blob Container (East US, LRS)
- Container: staging (Sample created)

# ETL Tool:
- Azure Data Factory v2 (East US)

# Flow withing Azure Portal:
- Creation of a Resource Group(RG)
- Create Source Database (SQL Server -> SQL Database(AdventureWorksLT))
- Create Target and Staging ADLS Blob containers (raw, staging)
- Create a ADF

# Flow within Azure Data Factory:
- Create Linked Services for all three Source, Staging and Target
- Create the required Datasets from the above Linked Services
- We will create 3 separate pipelines:
    - First - Simple ETL Copy data from source to target
        - For this we will use **Move & Transform - Copy Data** from **Activities**
        - Source will be SQL Server dataset and Target will be the ADLS Blob dataset
        - Finally Debug the pipeline
    - Second - ETL including DataFlow
        - For this we will use **Move & Transform - Data Flow** from **Activities**
        - Source will be SQL Server dataset
        - We will create [3 separate branches for Product Table] and [1 branch for SalesOrderDetail Table] for different transformations:
           - **Product Table Branch 1 (Filter and Sort) - Using ProductID Table:**
             - Filter: Expression Builder -  ProductID > 600 && ProductID < 750 && LIKE(Color, 'Black')
             - Sort: Sort on ProductID column in ascending order
             - Select: Select the necessary columns (ProductID, Name, Color)
             - Sink: ADLS Blob container respective dataset
           - **Product Table Branch 2 (Group By and Aggregate) - Using ProductID Table:**
             - Aggregate: GroupBy - ProductID Column and Aggregate - Weight Column. Expression Builder - sum(Weight)
             - Sort: Sort on ProductID column in ascending order
             - Select: Select the ProductID and Weight Columns
             - Sink: ADLS Blob container respective dataset
           - **Product Table Branch 3 (Select Columns for Join with SalesOrderDetails table on another branch):**
             - Select: Select columns ProductID, Name, Color, StandardCost, ListPrice, Size, Weight from Product Table
           - **SalesOrderDetail Branch 1 (Select Columns for Join with Product table on this branch):**
             - Select: Select columns SalesOrderID, OrderQty, ProductID, UnitPrice
             - Join: Select Inner Join option and mention the tables from left and right, then provide the join condition: ProductID {Left Table} == ProductID {Right Table}
             - Sort:  Sort on SalesOrderID and ProductID {Right Table} column in ascending order
        - Finally Debug the pipeline
    - Third - ETL with Staging Location:
        - For this we will use **Move & Transform - Copy Data** from **Activities**
        - Source will be SQL Server dataset and Target will be the ADLS Blob dataset
        - Withing the settings of Copy Data, select enable staging mention the linked service and the dataset
        - Finally Debug the pipeline
- As a verification step compared the results of the .csv files generated in ADLS Blob Storage account to the result of the queries run on SSMS, by connecting the SSMS with the Azure SQL Server.

Note:
All done on single partition as small dataset
