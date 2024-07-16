[[_TOC_]]

---
# Introduction
In 2024, the Mississippi Department of Education (MSDE) will begin using the Generate application to create the EDFacts Membership (FS052) file.

This document contains technical details of the processes used to load Membership data in Generate from the Mississippi Department of Education’s (MDE’s) source environment.

---
# ETL Process Overview

For more information on the technology, environments, and processes used to ETL data for this project, please visit the [Generate MSIS 2.0 - ETL Process Overview](/Generate-MSIS-2.0-%2D-ETL-Process-Overview) page.


Detailed field level documentation of data lineage is available in the [FS052 Membership ETL Checklist.xlsx](https://example.link.sharepoint).

For the original ETL Checklist Template please go here: 
https://ciidta.communities.ed.gov/#communities/pdc/documents/17074


## ODS_SIS to Curated Zone

There are two Azure pipelines for Membership (FS052) that populate the required Curated Zone tables. One pipeline will populate the set of Dimension tables required and another will populate the Fact tables. WWT is currently determining the naming conventions for these Azure Pipelines.

Stored Procedures called by Azure Pipelines follow a naming convention:
_"usp_Gen_ODSToCurated" + "Curated Zone table name"_

For example, if a stored procedure is called to help migrate data for rds.DimPeople it would be called "dbo.usp_Gen_ODSToCurated_DimPeople".

Stored Procedures called by Azure Pipelines are located within the ODS_SIS database in the "dbo" schema.

**Image 1**: ODS_SIS, Stored Procedures Viewed from SQL Server Management Studio (SSMS)
![ssms_odstocurated_sprocs.PNG](/.attachments/images/ssms_odstocurated_sprocs.PNG =300x)

The ETLs are used to populate various Curated Zone tables in MSIS 2.0 according to the data requirements for the EDFacts file specification(s) and other data warehousing needs. Tables needed for the EDFacts pipeline can be found in the ODS_SIS database in the "dbo" schema.

**Image 2**: ODS_SIS, Tables Viewed in SQL Server Management Studio (SSMS)
![ssms_odstocurated_tables_1externaltables.PNG](/.attachments/images/ssms_odstocurated_tables_1externaltables.PNG =300x)

![ssms_odstocurated_tables_2demographicsk12student.PNG](/.attachments/images/ssms_odstocurated_tables_2demographicsk12student.PNG =300x)

**Image 3**: Generate Data Flow Diagram (Source: ODS_SIS, Destination: Curated)
![msis2_odstocurated_diagram.PNG](/.attachments/images/msis2_odstocurated_diagram.PNG =700x)

## Curated Zone to Generate CEDS Data Warehouse

There is a parent pipeline used to call other pipelines in the correct order. This pipeline is called Parent_Job_CuratedToGenerate. There are two Azure pipelines for Membership (FS052) that populate the required Generate CEDS Data Warehouse tables from the Curated Zone. One pipeline will populate the set of Dimension tables required and another will populate the Fact tables.

- Membership (FS052) 
1. Membership_Load_Dim_CuratedToGenerate
1. Membership_Load_Fact_CuratedToGenerate

The Dimension pipeline will follow a standard flow: 
1. Clear data for all tables to be populated by the pipeline.
1. For each table, copy data from the Curated Zone database to the Generate database version of the table then create a “Minus 1 Record” in the table if one does not already exist. 

**Image 4**: Membership_Load_Dim_CuratedToGenerate
![azure_CuratedToGenerate_Pipeline_Membership_Dim.png](/.attachments/images/azure_CuratedToGenerate_Pipeline_Membership_Dim.png  =1000x)

The ETL can be executed from within Azure Synapse User Interface (UI). The Azure Pipeline will be run from the Microsoft Azure Synapse Analytics environment.

Stored Procedures called by Azure Pipelines follow a naming convention:
_"usp_Gen_CuratedToGenerate" + "CEDS Data Warehouse table name"_

For example, if a stored procedure is called to help migrate data for rds.FactK12StudentCounts it would be called "dbo.usp_Gen_CuratedToGenerate_FactK12StudentCounts".
 
**Image 5**: Curated, Stored Procedures Viewed from SQL Server Management Studio (SSMS)
![ssms_curatedtogenerate_sprocs.PNG](/.attachments/images/ssms_curatedtogenerate_sprocs.PNG =300x)
 
Tables needed for the EDFacts pipeline can be found in the Curated Zone database in the "rds" schema.

**Image 6**: Curated, Tables Viewed from SQL Server Management Studio (SSMS)
![ssms_curatedtogenerate_tables_1externaltables.PNG](/.attachments/images/ssms_curatedtogenerate_tables_1externaltables.PNG =300x)

![ssms_curatedtogenerate_tables_2factk12studentcounts.PNG](/.attachments/images/ssms_curatedtogenerate_tables_2factk12studentcounts.PNG =300x)

Each Azure Pipeline contains a "Copy Data" task (under the "Move and Transform" activity) with three configuration tabs for Source, Sink and Mapping that perform the ETL work for each table: 
1. **Source**: Defines the source datasets used and any Stored Procedures called by the Pipeline. (see image 6)
1. **Sink**: Defines any transformations performed after the "Source" processes. Typically, this data is removed from the Generate destination table to prepare it from the data load. (see image 7)
1. **Mapping**: Defines the field level mappings for the source and destination tables. (see image 8)

**Image 7**: Table Level Pipeline Example - “Source” Component for factK12StudentCounts
![azure_curatedtogenerate_pipelinesource_factk12studentcounts.PNG](/.attachments/images/azure_curatedtogenerate_pipelinesource_factk12studentcounts.PNG =500x)

**Image 8**: Table Level Pipeline Example - “Sink” Component for factK12StudentCounts 
![azure_curatedtogenerate_pipelinesink_factk12studentcounts.PNG](/.attachments/images/azure_curatedtogenerate_pipelinesink_factk12studentcounts.PNG =500x)
 
**Image 9**: Table Level Pipeline Example - “Mapping” Component for factK12StudentCounts
![azure_curatedtogenerate_pipelinemapping_factk12studentcounts.PNG](/.attachments/images/azure_curatedtogenerate_pipelinemapping_factk12studentcounts.PNG =500x)

**Image 10**: Generate Data Flow Diagram (Source: Curated, Destination: Generate)
![msis2_curatedtogenerate_diagram.PNG](/.attachments/images/msis2_curatedtogenerate_diagram.PNG =700x)


##Directory Dependencies  

Nearly every EDFacts file has some Directory data dependencies since EDFacts files are organized by education organization levels: 
1. State Education Agencies (SEA)
1. Local Education Agencies (LEA)
1. School (SCH)

This Membership ETL documentation does not include information about Directory ETLs. We assume that a parent Azure Pipeline will load the required directory data prior to the Membership data loads. Details about Directory ETL are available in the [Generate MSIS 2.0 - Directory ETL](/Generate-MSIS-2.0-%2D-Directory-ETL) page.

---
# Membership (FS052)

The Membership Azure Pipelines call several SQL stored procedures:
- ODS_SIS to Curated Zone (these pipelines are still being built as of 3/14/2024 and updates will be made to the following list when possible)
1. usp_Gen_ODSToCurated_FactStudentCounts_Stub
1. usp_Gen_ODSToCurated_StagingPersonRace
1. usp_Gen_ODSToCurated_StagingPersonRace1
- Curated Zone to Generate CEDS Data Warehouse
1. usp_Gen_CuratedToGenerate_DimPeople (disabled - should already be migrated with Directory ETLs)
1. usp_Gen_CuratedToGenerate_FactK12StudentCounts

## Source System(s)
All data for the Membership reports is sourced from the following databases/tables/views:

**Database: ODS_SIS**
- dbo.Demographics_K12Student
- dbo.Enrollment_K12Student
- dbo.Identities_K12Student 

**Database: Curated**
- rds.DimPeople
- rds.FactK12StudentCounts 

Additionally, some dimensions used to store option set values will be moved from the Curated Zone to Generate to ensure that these values are in sync with id numbers stored in the fact tables. Tables include rds.DimGradeLevels, rds.DimK12SchoolStatuses, and rds.DimRaces. 

For more information on Generate dimension types and id management, please visit the [Generate Dimension Types](/Generate-MSIS-2.0-%2D-ETL-Process-Overview#generate-dimension-types) section of the Generate MSIS 2.0 - ETL Process Overview page.

