# GENERAL 

On June 12th 2024, the Mississippi Department of Education (MDE) provided a demonstration to their Azure DevOps Wiki documentation to the Generate ScDU Workgroup.
The markdown files provided in this repository are examples of the backend files shown in the demonstration. 
These files are specific to Mississippi's environments and ETLs needs and are not intended as a universal template for Generate ETL documentation.
If being used as a starter template for your documentation, please customize as needed for your own practices and use cases. 

# PAGE SPECIFIC NOTES 

## ETL Process Overview Page
1. "Generate Fact Types"
- The "Generate Fact Types" section of the ETL Process Overview page contains a table with EDFacts File Specifications by Fact Type. 
- These are specific to Mississippi's scope of work. There may be more EDFacts File Specifications within a Fact Type in the Generate ETL. 
- The Generate database contains current metadata with these mappings that can be used to view. The queries in the "Generate Fact Types" section of this page can be used to explore the broader Generate relationships.

## Fact Type Pages	
1. Mississippi Not Using Generate Staging (Curated Zone to Curated Zone to Generate CEDS Data Warehouse):
- Mississippi is bypassing the Generate Staging data layer by ETLing their data into a Generate CEDS Data Warehouse from a CEDS Data Warehouse called the "Curated Zone" in their source system. 
- Typically, states ETL their source data into Generate's Staging environment and then from Generate Staging to the Generate CEDS Data Warehouse.
- Mississippi's wiki documentation does not have any references for how to use the Generate Staging data layer. Visit the Generate website for further information. https://center-for-the-integration-of-id.gitbook.io/generate-documentation/developer-guides/developer-guides-getting-started/implementation 

2. Business Rules Section Recommended on Fact Type Pages: 
- Fact Type ETL pages do not currently contain a "Business Rules" section.
- Mississippi plans to add this section when working through data migrations and testing. It will include findings from testing, data limitations, and state specific business rules. 
