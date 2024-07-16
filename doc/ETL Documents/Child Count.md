
[[_TOC_]]

---
# Introduction
In 2024, the Mississippi Department of Education (MSDE) will begin using the Generate application to create the following EDFacts Child Count files: 
1. Children with Disabilities (IDEA) School-Age (FS002)
1. Children with Disabilities (IDEA) Early Childhood (FS089)

This document contains technical details of the processes used to load Child Count data in Generate from the Mississippi Department of Education’s (MDE’s) source environment.

---
# ETL Process Overview

For more information on the technology, environments, and processes used to ETL data for this project, please visit the [Generate MSIS 2.0 - ETL Process Overview](/Generate-MSIS-2.0-%2D-ETL-Process-Overview) page.
Detailed field level documentation of data lineage is available in the following ETL Checklists: 
- [FS002 Children with Disabilities (School Age) ETL Checklist.xlsx](https://example.link.sharepoint)
- [FS089 Children with Disabilities (Early Childhood) ETL Checklist.xlsx](https://example.link.sharepoint).

For the original ETL Checklist Template please go here: 
https://ciidta.communities.ed.gov/#communities/pdc/documents/17074


## ODS_SIS to Curated Zone

There are two Azure pipelines that populate all of the required Curated Zone tables. The "Dimension Tables" pipeline will populate all dimension tables and the "Fact Tables" pipeline will populate all the fact tables for all Fact Types.

Stored Procedures called by these Azure Pipelines follow a naming convention:
 _"dim or fact schema name"_ + _"Curated Zone table name"_ + _"Select"_ ("Select" is sometimes omitted)

For example, if a stored procedure is called to help migrate data for rds.DimPeople it would be called "dim.PeopleSelect" or "dim.People".

Stored Procedures called by Azure Pipelines are located within the ODS_SIS database in the "dim" or "fact" schemas.

**Image 1**: ODS_SIS, Stored Procedures Viewed from SQL Server Management Studio (SSMS)
![ssms_odstocurated_sprocs_1programmability.PNG](/.attachments/images/ssms_odstocurated_sprocs_1programmability.PNG =300x)

![ssms_odstocurated_sprocs_2dimpeopleselect.PNG](/.attachments/images/ssms_odstocurated_sprocs_2dimpeopleselect.PNG =300x)

The ETLs are used to populate various Curated Zone tables in MSIS 2.0 according to the data requirements for the EDFacts file specification(s) and other data warehousing needs. Tables needed for the EDFacts pipeline can be found in the ODS_SIS database in the "dbo" schema.

**Image 2**: ODS_SIS, Tables Viewed in SQL Server Management Studio (SSMS)
![ssms_odstocurated_tables_1externaltables.PNG](/.attachments/images/ssms_odstocurated_tables_1externaltables.PNG =300x)

![ssms_odstocurated_tables_2demographicsk12student.PNG](/.attachments/images/ssms_odstocurated_tables_2demographicsk12student.PNG =300x)

**Image 3**: Generate Data Flow Diagram (Source: ODS_SIS, Destination: Curated)
![msis2_odstocurated_diagram.PNG](/.attachments/images/msis2_odstocurated_diagram.PNG =700x)

## Curated Zone to Generate CEDS Data Warehouse

There is a parent pipeline used to call other pipelines in the correct order. This pipeline is called Parent_Job_CuratedToGenerate. There are two Azure pipelines for Child Count (FS002, FS089) that populate the required Generate CEDS Data Warehouse tables from the Curated Zone. For each, one pipeline will populate the set of Dimension tables required and another will populate the Fact tables.

- Child Count (FS002, FS089) 
1. ChildCount_Load_Dim_CuratedToGenerate (component to be built)
1. ChildCount_Load_Fact_CuratedToGenerate (component to be built)

The Dimension pipeline will follow a standard flow: 
1. Clear data for all tables to be populated by the pipeline.
1. For each table, copy data from the Curated Zone database to the Generate database version of the table then create a “Minus 1 Record” in the table if one does not already exist. 

**Image 4**: ChildCount_Load_Dim_CuratedToGenerate (component to be built)
![azure_curatedtogenerate_pipeline_childcount_dim.PNG](/.attachments/images/azure_curatedtogenerate_pipeline_childcount_dim.PNG =1000x)

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

This Child Count ETL documentation does not include information about Directory ETLs. We assume that a parent Azure Pipeline will load the required directory data prior to the Child Count data loads. Details about Directory ETL are available in the [Generate MSIS 2.0 - Directory ETL](/Generate-MSIS-2.0-%2D-Directory-ETL) page.

---
# Child Count (FS002, FS089)

The Child Count Azure Pipelines call several SQL stored procedures (these pipelines are still being built as of 6/20/2024 and updates will be made to the following list when possible):
- ODS_SIS to Curated Zone 
1. dim.PeopleSelect
2. fact.K12StudentCountsSelect (component to be built)

- Curated Zone to Generate CEDS Data Warehouse
1. usp_Gen_CuratedToGenerate_DimPeople
1. usp_Gen_CuratedToGenerate_FactK12StudentCounts

## Source System(s)
All data for the Child Count reports is sourced from the following databases/tables/views:

**Database: ODS_SIS**
*State-Data Dimension & Fact Table(s)*
||FS002|FS089|
|--|--|--|
|dbo.Calendar_K12Org|x|x|
|dbo.Demographics_K12Student|x|x|
|dbo.Enrollment_K12Student|x|x|
|dbo.Identities_K12Student|x|x|

**Note: The ODS_SIS destination data elements for special education student data have not yet been determined (as of June 2024). The Child Count files will not populate without these elements (specifically Idea Indicator, Idea Disability Type, and Idea Educational Environment) so testing is not possible until the pipeline for these elements is built. 

**Database: Curated**
||FS002|FS089|
|--|--|--|
|rds.DimPeople|x|x|
|rds.FactK12StudentCounts|x|x|

Additionally, some dimensions used to store option set values will be moved from the Curated Zone to Generate to ensure that these values are in sync with id numbers stored in the fact tables. 

For more information on Generate dimension types and id management, please visit the [Generate Dimension Types](/Generate-MSIS-2.0-%2D-ETL-Process-Overview#generate-dimension-types) section of the Generate MSIS 2.0 - ETL Process Overview page. The following Option-Set Value Dimension(s) are used for this Fact Type.

*Option-Set Value Dimension(s)*
||FS002|FS089|
|--|--|--|
|rds.DimAges|x|x|
|rds.DimEnglishLearnerStatuses|x|x|
|rds.DimGradeLevels|x|x|
|rds.DimIdeaDisabilityTypes|x|x|
|rds.DimIdeaStatuses|x|x|
|rds.DimK12Demographics|x|x|
|rds.DimRaces|x|x|


