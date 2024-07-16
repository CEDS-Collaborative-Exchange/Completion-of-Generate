[[_TOC_]]

---
## Extract Transform and Load (ETL) Tools
### Microsoft Azure Pipelines 

Two of the key databases in MDE’s source environment used for EDFacts reporting are hosted in Microsoft Azure (see Image 1). ETL from these databases uses Microsoft Azure Pipelines. MDE developers create and access these Pipelines using Microsoft Azure Synapse Analytics.

**Image 1**: Microsoft Azure Synapse Analytics Environment
![azure_curatedtogenerate_pipeline.PNG](/.attachments/images/azure_curatedtogenerate_pipeline.PNG =700x)

####Azure Pipeline Execution
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

### SQL Server Stored Procedures

Microsoft Azure Pipelines will sometimes call a SQL Server Stored Procedure to help ETL the data within the source system or into Generate. Generate also uses Stored Procedures for ETL. Stored Procedures can be viewed from SQL Server Management (SSMS).
 

---
## Data Flow

Using an ETL process that calls one or more stored procedures, data is moved from MDE’s Azure Data Lake Storage (ADLS) repository into the ODS_SIS SQL database. The ODS_SIS is the staging environment that will be used for the MSIS 2.0 Generate project. The Generate staging environment will not be used. Using a combination of Azure Pipelines and Stored Procedures, data from the ODS_SIS are moved into MDE’s Curated Zone (which is a [CEDS Data Warehouse](https://ceds.ed.gov/dataModel.aspx)).

Since the Curated Zone and the Generate Reporting Data Store (RDS) both use the CEDS Data Warehouse architecture we expect that they will be in sync with each other, specifically the RDS Dimension (Dim) and Fact tables/columns should match. Using a combination of Azure Synapse pipelines and Stored Procedures, the data in the Curated Zone are then moved into the Generate database.

See Image 2 for a diagram that outlines this process. Please note, this document does not describe the entire data flow outlined in the diagram below. The goal of this document is to describe the ETL components used starting from the ODS_SIS staging environment (diagram: item 3) into the CEDS Data Warehouse Data Layer of the Generate database (diagram: item 6) so that an ETL developer can successfully run the ETL needed for the specific EDFacts file specifications they're working with.

**Image 2**: MDE Data Flow Diagram
![msis2_etlsystem_diagram.PNG](/.attachments/images/msis2_etlsystem_diagram.PNG =700x)

---
## Environments

There are three Microsoft Azure Synapse workspaces used for this project that follow the dev-test-prod model:
- azure-msis-dev
- azure-msis-test
- azure-msis-prod

Additionally, there are failover workspaces that use the "centralus" label (i.e. syn-msis2-centralus-a-dev). These workspaces will not be documented in the Generate ETL. 

[azure-msis-dev](https://portal.azure.com/projectname/insertyourenvironmentspecificworkspacedetails/azure-msis-dev/overview)
[azure-msis-test](https://portal.azure.com/projectname/insertyourenvironmentspecificworkspacedetails/azure-msis-test/overview)
[azure-msis-prod](https://portal.azure.com/projectname/insertyourenvironmentspecificworkspacedetails/azure-msis-prod/overview)

---
## Generate Fact Types

Generate ETL Stored Procedures are organized by **Fact Types**. 

Many EDFacts file specifications have shared timelines, reporting requirements, and/or a high degree of overlap in source system field mappings. When this happens, the data are organized into the same Fact Type to make data migration, testing, and file submission more efficient.

The Fact Type determines where the data is stored in the Curated Zone and the Generate Reporting Data Store (RDS). For example, the Directory data are stored in RDS.FactOrganizationCounts. Generate has a series of tables used by the application where more information about Fact Types can be found. See the queries below. 

``` get information about Generate Fact Type
-- RDS.DimFactTypes describes Fact Types used in the Generate database
SELECT * FROM RDS.DimFactTypes

-- App.GenerateReport_FactType captures the Fact Type associated with a report
-- This table is available in Generate 11.3 or later
SELECT * FROM App.GenerateReport_FactType

-- App.GenerateReports describes reports produced by Generate
SELECT * FROM App.GenerateReports

-- Get Fact Type to Report relationship with descriptions
SELECT		 agrft.FactTypeId 
			,agrft.GenerateReportId
			,rdft.FactTypeCode
			,rdft.FactTypeDescription
			,agr.ReportCode
FROM		App.GenerateReport_FactType AS agrft
LEFT JOIN	RDS.DimFactTypes			AS rdft
		ON rdft.DimFactTypeId = agrft.FactTypeId
LEFT JOIN	App.GenerateReports			AS agr
		ON agr.GenerateReportId = agrft.GenerateReportId	
ORDER BY	agrft.FactTypeId, agrft.GenerateReportId
```
ETL wiki documentation for this project will be organized by Fact Type. 

|MDE ETL Wiki|Fact Type Code |Fact Type Description |File Specification(s)|
|--|--|--|--|
|[Assessment ETL](/Generate-MSIS-2.0-%2D-Assessment-ETL)|assessment|Assessment Data|FS113, FS125, FS126, FS137, FS138, FS139, FS175, FS178, FS179, FS185, FS188, FS189|
||childcount|Child Count|FS002, FS089|
||chronic|Chronic Absenteeism|FS195|
||cte|Career and Technical Education|FS132|
|[Directory ETL](/Generate-MSIS-2.0-%2D-Directory-ETL)|directory|Directory |FS029, FS039, FS129, FS130, FS131, FS163, FS170, FS193, FS196, FS197, FS205, FS206|
||discipline|Discipline|FS005, FS006, FS007, FS086, FS088, FS143, FS144|
|[Dropouts ETL](/Generate-MSIS-2.0-%2D-Dropouts-ETL)|dropouts|Dropouts|FS032|
||exiting|Exit from Special Education|FS009|
|[Graduates & Completers ETL](/Generate-MSIS%202.0%20%252D%20Graduates%20%26%20Completers%20ETL)|graduatescompleters|Graduates/Completer|FS040|
|[Graduation Rate ETL](/Generate-MSIS%202.0%20%252D%20Graduation%20Rate%20ETL)|graduationrate|Graduation Rate|FS150, FS151|
||homeless|Homeless Students|FS118, FS194|
||hsgradpsenroll|HS Grad PS Enrollment|FS160|
|[Membership ETL](/Generate-MSIS%202.0%20%252D%20Membership-ETL)|membership|Membership|FS033, FS052|
||neglectedordelinquent|N or D Students|FS119, FS127|
||organizationstatus|Organization Status related reports|FS200, FS201, FS202|
||personnel|Personnel|FS059, FS067, FS070, FS099, FS112, FS203|
||titleI|Title I Students|FS037, FS134|
||titleIIIELOct|Title III EL Students - October|FS141|
||titleIIIELSY|Title III EL Students - School Year|FS045, FS116|
| |Unassigned| |FS050, FS210, FS211|



### EDFacts File Specifications

Word document versions of EDFacts File Specifications are available here:
https://www2.ed.gov/about/inits/ed/edfacts/file-specifications.html

The EDFacts Data Submission Organizer is a searchable table that contains information about submission timelines and high level information about file specifications.  https://edfacts.communities.ed.gov/#program/data-submission-organizer

---
## Generate Dimension Types

### Option-Set Value Dimensions vs. State-Data Dimensions
The Common Education Data Standards (CEDS) Data Warehouse uses a [star schema](https://en.wikipedia.org/wiki/Star_schema)design. Star schema data models consist of fact and dimension tables.

Many of the dimension tables used in the CEDS Data Warehouse are pre-defined categorical data fields in CEDS. This allows education agencies to use a standard definition for common fields used across agencies. We refer to dimension tables like this as “Option-Set Value Dimensions” because they contain the set of “options” defined in CEDS for that field. The values in these fields typically don’t change much even across school years. 

For example, the query results for the rds.DimRaces in a Curated Zone CEDS Data Warehouse will look something like the following table.
|DimRaceId|RaceCode|RaceDescription|RaceEdFactsCode|
|--|--|--|--|
|-1|MISSING|MISSING|MISSING|
|1|AmericanIndianorAlaskaNative|American Indian or Alaska Native|AM7|
|2|Asian|Asian|AS7|
|3|BlackorAfricanAmerican|Black or African American|BL7|
|4|DemographicRaceTwoOrMoreRaces|Demographic Race Two or More Races|MU7|
|5|NativeHawaiianOrOtherPacificIslander|Native Hawaiian or Other Pacific Islander|PI7|
|6|RaceAndEthnicityUnknown|Race and Ethnicity Unknown|MISSING|
|7|White|White|WH7|
|8|HispanicorLatinoEthnicity|Hispanic|HI7|
|156|RaceAndEthnicityUnknown|Race and Ethnicity Unknown|MISSING|

The definitions and descriptions for this field come from CEDS and can be viewed in the Domain Entity Schema (DES) page on the CEDS website: https://ceds.ed.gov/element/001943.

State-Data Dimensions like DimK12Schools differ from Option-Set Value Dimensions, because State-Data Dimensions will contain state specific data and are likely to change often.

Example Tables:
- Option-Set Value Dimensions: rds.DimCharterSchoolAuthorizers, rds.DimGradeLevels, rds.DimDisabilityType, rds.DimK12SchoolStatuses, rds.DimRaces
- State-Data Dimensions: rds.DimSEAs, rds.DimLEAs, rds.DimK12Schools, rds.DimPeople

### Why are Option-Set Value Dimensions in pipelines to Generate?

If Option-Set Value Dimensions are pre-defined in Generate, why are they in Curated Zone to Generate pipelines?

This is done as a precaution to ensure that the ID numbers for the option-set values within the dimension, which are used as key in the fact tables, are always in sync with the fact tables in the ETL. 

For example, if the Generate database updated the id numbers for dimension because of change in CEDS but the Curated Zone had not yet received the update then the Generate version of this table could not be used in a join to retrieve the correct text values for the option set. 




