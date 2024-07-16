[[_TOC_]]

---
# Introduction
In 2024, the Mississippi Department of Education (MSDE) will begin using the Generate application to create the following EDFacts directory files: 
1. Directory Files (FS029)
1. Grades Offered (FS039)
1. Management Organization for Charter Schools Roster (FS196)
1. Crosswalk of Charter Schools to Management Organizations ETL Checklist (FS197)

This document contains technical details of the processes used to load directory data in Generate from the Mississippi Department of Education’s (MDE’s) source environment.

--- 
# ETL Process Overview

For more information on the technology, environments, and processes used to ETL data for this project, please visit the [Generate MSIS 2.0 - ETL Process Overview](/Generate-MSIS-2.0-%2D-ETL-Process-Overview) page.

Detailed field level documentation of data lineage is available in the following ETL Checklists: 

- [FS029 Directory ETL Checklist.xlsx](https://example.link.sharepoint)
- [FS039 Grades Offered ETL Checklist.xlsx](https://example.link.sharepoint).
- [FS196 Management Organization for Charter Schools Roster File ETL Checklist.xlsx](https://example.link.sharepoint).
- [FS197 Crosswalk of Charter Schools to Management Organizations ETL Checklist.xlsx](https://example.link.sharepoint).

For the original ETL Checklist Template please go here: 
https://ciidta.communities.ed.gov/#communities/pdc/documents/17074


## ODS_SIS to Curated Zone

There are two Azure pipelines that populate all of the required Curated Zone tables. The "Dimension Tables" pipeline will populate all dimension tables and the "Fact Tables" pipeline will populate all the fact tables for all Fact Types.

Stored Procedures called by these Azure Pipelines follow a naming convention:
 _"dim or fact schema name"_ + _"Curated Zone table name"_ + _"Select"_ ("Select" is sometimes omitted)

For example, if a stored procedure is called to help migrate data for rds.K12Schools it would be called "dim.K12SchoolsSelect" or "dim.K12Schools".

Stored Procedures called by Azure Pipelines are located within the ODS_SIS database in the "dim" or "fact" schemas.

**Image 1**: ODS_SIS, Stored Procedures Viewed from SQL Server Management Studio (SSMS)
![ssms_odstocurated_sprocs_1programmability.PNG](/.attachments/images/ssms_odstocurated_sprocs_1programmability.PNG =300x)

![ssms_odstocurated_sprocs_2dimk12schoolsselect.PNG](/.attachments/images/ssms_odstocurated_sprocs_2dimk12schoolsselect.PNG =300x)

The ETLs are used to populate various Curated Zone tables in MSIS 2.0 according to the data requirements for the EDFacts file specification(s) and other data warehousing needs. Directory tables needed for the EDFacts pipeline can be found in the ODS_SIS database in the "dbo" schema.

**Image 2**: ODS_SIS, Tables Viewed in SQL Server Management Studio (SSMS)
![ssms_odstocurated_tables_1externaltables.PNG](/.attachments/images/ssms_odstocurated_tables_1externaltables.PNG =300x)

![ssms_odstocurated_tables_2leasK12Org.PNG](/.attachments/images/ssms_odstocurated_tables_2leasK12Org.PNG =300x)

**Image 3**: Generate Data Flow Diagram (Source: ODS_SIS, Destination: Curated)
![msis2_odstocurated_diagram.PNG](/.attachments/images/msis2_odstocurated_diagram.PNG =700x)

## Curated Zone to Generate CEDS Data Warehouse

There is a parent pipeline used to call other pipelines in the correct order. This pipeline is called Parent_Job_CuratedToGenerate. There are two Azure pipelines for Directory (FS029, FS196, FS197) and two for Grades Offered (FS039) that populate the required Generate CEDS Data Warehouse tables from the Curated Zone. For each, one pipeline will populate the set of Dimension tables required and another will populate the Fact tables.

- Directory (FS029, FS196, FS197)
1. Directories_Load_Dim_CuratedToGenerate
1. Directories_Load_Fact_CuratedToGenerate
- Grades Offered (FS039) 
1. GradesOffered_Load_Dim_CuratedToGenerate (component to be built)
1. GradesOffered_Load_Fact_CuratedToGenerate (component to be built)

The Dimension pipeline will follow a standard flow: 
1. Clear data for all tables to be populated by the pipeline.
1. For each table, copy data from the Curated Zone database to the Generate database version of the table then create a “Minus 1 Record” in the table if one does not already exist. 

**Image 4**: Directories_Load_Dim_CuratedToGenerate
![azure_curatedtogenerate_pipeline_directories_dim.PNG](/.attachments/images/azure_curatedtogenerate_pipeline_directories_dim.PNG =1000x)

The ETL can be executed from within Azure Synapse User Interface (UI). The Azure Pipeline will be run from the Microsoft Azure Synapse Analytics environment.

Stored Procedures called by Azure Pipelines follow a naming convention:
_"usp_Gen_CuratedToGenerate" + "CEDS Data Warehouse table name"_

For example, if a stored procedure is called to help migrate data for rds.DimLEAs it would be called "dbo. usp_Gen_CuratedToGenerate_DimLeas".
 
**Image 5**: Curated, Stored Procedures Viewed from SQL Server Management Studio (SSMS)
![ssms_curatedtogenerate_sprocs.PNG](/.attachments/images/ssms_curatedtogenerate_sprocs.PNG =300x)
 
Directory tables needed for the EDFacts pipeline can be found in the Curated Zone database in the "rds" schema.

**Image 6**: Curated, Tables Viewed from SQL Server Management Studio (SSMS)
![ssms_curatedtogenerate_tables_1externaltables.PNG](/.attachments/images/ssms_curatedtogenerate_tables_1externaltables.PNG =300x)

![ssms_curatedtogenerate_tables_2dimleas.PNG](/.attachments/images/ssms_curatedtogenerate_tables_2dimleas.PNG =300x)

Each Azure Pipeline contains a "Copy Data" task (under the "Move and Transform" activity) with three configuration tabs for Source, Sink and Mapping that perform the ETL work for each table: 
1. **Source**: Defines the source datasets used and any Stored Procedures called by the Pipeline. (see image 6)
1. **Sink**: Defines any transformations performed after the "Source" processes. Typically, this data is removed from the Generate destination table to prepare it from the data load. (see image 7)
1. **Mapping**: Defines the field level mappings for the source and destination tables. (see image 8)

**Image 7**: Table Level Pipeline Example - “Source” Component for dimLeas 
![azure_curatedtogenerate_pipelinesource_dimleas.PNG](/.attachments/images/azure_curatedtogenerate_pipelinesource_dimleas.PNG =500x)

**Image 8**: Table Level Pipeline Example - “Sink” Component for dimLeas 
![azure_curatedtogenerate_pipelinesink_dimleas.PNG](/.attachments/images/azure_curatedtogenerate_pipelinesink_dimleas.PNG =500x)
 
**Image 9**: Table Level Pipeline Example - “Mapping” Component for dimLeas 
![azure_curatedtogenerate_pipelinemapping_dimleas.PNG](/.attachments/images/azure_curatedtogenerate_pipelinemapping_dimleas.PNG =500x)

**Image 10**: Generate Data Flow Diagram (Source: Curated, Destination: Generate)
![msis2_curatedtogenerate_diagram.PNG](/.attachments/images/msis2_curatedtogenerate_diagram.PNG =700x)

---
# Directory (FS029, FS039, FS196, FS197)

The Directory Azure Pipelines call several SQL stored procedures:
- ODS_SIS to Curated Zone
1. dim.CharterSchoolAuthorizersSelect (component to be built)
1. dim.CharterSchoolManagementOrganizationsSelect (component to be built)
1. dim.K12SchoolsSelect
1. dim.LeasSelect
1. dim.SeasSelect (component to be built)
1. fact.K12SchoolGradeLevelsSelect (component to be built)
1. fact.LEAGradeLevelsSelect (component to be built)
1. fact.OrganizationCountsSelect (component to be built)

- Curated Zone to Generate CEDS Data Warehouse
1. dbo.usp_Gen_CuratedToGenerate_BridgeK12SchoolGradeLevels (component to be built)
1. dbo.usp_Gen_CuratedToGenerate_BridgeLEAGradeLevels (component to be built)
1. dbo.usp_Gen_CuratedToGenerate_CharterSchoolAuthorizers (component to be built)
1. dbo.usp_Gen_CuratedToGenerate_CharterSchoolManagementOrganizations (component to be built)
1. dbo.usp_Gen_CuratedToGenerate_DimK12Schools
1. dbo.usp_Gen_CuratedToGenerate_DimLeas
1. dbo.usp_Gen_CuratedToGenerate_DimSeas
1. dbo.usp_Gen_CuratedToGenerate_FactOrganizationCounts (component to be built)


## Source System(s)
All data for the Directory reports is sourced from the following databases/tables/views:

**Database: ODS_SIS**

||FS029| FS039| FS196| FS197|
|--|--|--|--|--|
|dbo.LEAS_K12Org|x|x|||
|dbo.School_K12Org|x|x|||
|Unknown**|||x|x|

**Note: The ODS_SIS destination table(s) for charter management and school authorizer organizations have not yet been determined. This data will be manually input for around 10 charter organizations into a front-end application that is under development (as of May 2024) and will be stored in the Cosmos DB. The data will be migrated from Cosmos DB to ODS_SIS. Once source tables are developed, they will be added to the table above.  

**Database: Curated**
*State-Data Dimension & Fact Table(s)*
||FS029| FS039| FS196| FS197|
|--|--|--|--|--|
|rds.DimCharterSchoolAuthorizers|x||x||
|rds.DimCharterSchoolManagementOrganizations|||x|x|
|rds.DimK12Schools|x|x||x|
|rds.DimLeas|x|x||x|
|rds.DimSeas|x|x||x|
|rds.FactOrganizationCounts|x|x|x|x|
|rds.BridgeK12SchoolGradeLevels||x|||
|rds.BridgeLEAGradeLevels||x|||

Additionally, some dimensions used to store option set values will be moved from the Curated Zone to Generate to ensure that these values are in sync with id numbers stored in the fact tables. For more information on Generate dimension types and id management, please visit the [Generate Dimension Types](/Generate-MSIS-2.0-%2D-ETL-Process-Overview#generate-dimension-types) section of the Generate MSIS 2.0 - ETL Process Overview page. The following Option-Set Value Dimension(s) are used for this Fact Type.

*Option-Set Value Dimension(s)*
||FS029| FS039| FS196| FS197|
|--|--|--|--|--|
|rds.DimGradeLevels||x|||
|rds.DimK12SchoolStatuses|x||||
|n/a|||x|x|

