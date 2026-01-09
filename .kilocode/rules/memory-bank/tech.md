# Technology Stack

## Core Platform

### Matillion ETL
- **Version**: 1.68.19
- **Environment**: Snowflake
- **Purpose**: Primary ETL orchestration and transformation engine
- **Job Types**:
  - Orchestration jobs (.ORCHESTRATION) - JSON format
  - Transformation jobs (.TRANSFORMATION) - JSON format

### Snowflake Data Warehouse
- **Primary Database**: `PROD_DATALAKE`
- **Secondary Database**: `PROD_NPG_VIP`
- **Purpose**: Target data warehouse for all ETL processes
- **Features Used**:
  - Snowflake Managed staging
  - Virtual warehouses for compute
  - Time travel capabilities
  - Zero-copy cloning

## Source Systems

### ERP Systems
- **Lawson (SQL Server)**
  - Connection: `${SQLConnLaw}`
  - Authentication: `DATALAKE_PROD` credentials
  - Primary tables: EMPLOYEE, ACTRANS, ARCUSTOMER, ACACTIVITY
  - Incremental loading via OBJ_ID

### CRM Systems
- **Microsoft Dynamics CRM**
  - Database: `PROD_DATALAKE.CRM_MSCRM`
  - Tables: ACCOUNTBASE, Assignment, JobOrder
  - Integration: Direct database connection

- **Salesforce**
  - Integration pattern: API-based or database replication
  - Directory: `ROOT/PROD/SalesForce/`

### HR Systems
- **UKG (Ultimate Kronos Group)**
  - Integration: API-based
  - Data types: Employment details, location details, organization levels
  - Jobs: Platform configuration, requisitions, headcount

### Marketing & Job Boards
- **Google Analytics**
  - Both GA and GA4 supported
  - Event-based data collection
  - Timestamp conversions for event tracking

- **Indeed**
  - File-based integration
  - Sponsored jobs performance data
  - File naming: `filename_YYYYMMDD.csv`

- **Career Builder**
  - File-based integration
  - Job performance metrics

- **Zip Recruiter**
  - File-based integration
  - Job board analytics

### Financial Systems
- **Essbase**
  - Bi-directional integration
  - Export jobs: LPM, WBS, Headcount
  - Format: Custom export formats for Essbase consumption

- **Concur**
  - Expense management data
  - Directory: `ROOT/PROD/Concur Data/`

### Third-Party Services
- **Equifax**
  - Background check data
  - S3 to Snowflake integration

- **Demandbase**
  - Marketing intelligence
  - Account-based marketing data

- **Kite**
  - Industry and client mapping
  - Reference data management

## Development Tools & Patterns

### Scripting Languages
- **Python (Jython)**
  - Used in transformation components
  - Database cursor operations: `cursor = context.cursor()`
  - Variable updates: `context.updateVariable()`
  - Grid variables: `context.getGridVariable()`

### SQL
- **Snowflake SQL**
  - Complex aggregations and transformations
  - MERGE statements for SCD Type 2
  - Window functions for analytics
  - CTEs (Common Table Expressions) for complex queries

### Environment Variables
```
${DB_Datalake}          # Database name
${DB_DatalakeOrig}      # Original database reference
${SNFLKDB}              # Snowflake database
${SNFLKSCH}             # Snowflake schema
${Sch_[SOURCE]}         # Source-specific schemas
${run_history_id}       # Job execution ID
${environment_username} # Current user
${Max_ObjId}            # Incremental load marker
${BeginDt}              # Date range start
${EndDt}                # Date range end
```

## Data Integration Patterns

### Staging Strategies
- **Snowflake Managed Staging**: Default for most loads
- **S3 Staging**: For external file-based sources
- **Azure Blob Storage**: Alternative staging option

### Authentication Methods
- **Credentials-based**: Username/password for database connections
- **Storage Integration**: For cloud storage access
- **Service Accounts**: `DATALAKE_PROD` for Lawson

### Compression
- **Gzip**: Standard compression for data transfer
- Configured in load options: `"6":{"slot":6,"values":{"1":{"slot":1,"type":"STRING","value":"Gzip"}}}`

## Performance Optimization

### Warehouse Management
- **Dynamic Sizing**: `WAREHOUSE_RESIZE.ORCHESTRATION`
- **Concurrency**: Parallel job execution
- **Resource Allocation**: Environment-specific warehouse assignments

### Query Optimization
- **Incremental Loading**: OBJ_ID and date-based filtering
- **Partitioning**: Week-based partitioning for actuals
- **Indexing**: Primary keys on dimension tables

### Data Transfer
- **Bulk Loading**: COPY INTO commands
- **Batch Processing**: Grouped transformations
- **Compression**: Gzip for network transfer

## Monitoring & Logging

### Audit Framework
- **Audit Tables**: Track all job executions
- **Metrics Captured**:
  - Row counts
  - Start/end timestamps
  - Success/failure status
  - Component-level tracking

### Error Handling
- **Component-level**: Success/Failure connectors
- **Job-level**: Audit success/failure paths
- **Aggregation**: And/Or components for multiple error paths

## Data Quality

### Validation Patterns
- **Type Checking**: Explicit CAST operations
- **Null Handling**: COALESCE for default values
- **Data Cleansing**: LTRIM, SUBSTRING operations

### SCD Type 2 Implementation
- **Tracking Fields**:
  - `CREATED_DT`: Record creation
  - `MODIFIED_DT`: Last modification
  - `ACTIVE_STATUS`: Current vs historical
  - `VALID_FRM_DT`: Validity start
  - `VALID_TO_DT`: Validity end (9999-12-31 for current)

## Integration Technologies

### File Formats
- **CSV**: Primary format for file-based sources
- **JSON**: Matillion job definitions
- **XML**: Some API responses

### Data Transfer Protocols
- **JDBC**: Database connections
- **REST APIs**: Cloud service integrations
- **File Transfer**: S3, Azure Blob

## Business Intelligence

### Power BI Integration
- **Security Model**: User/branch/region access control
- **Active Directory**: User authentication
- **Dimension Security**: Row-level security implementation

### Reporting Layer
- **Fact Tables**: Pre-aggregated for performance
- **Dimension Tables**: Conformed dimensions
- **Bridge Tables**: Many-to-many relationships

## Development Constraints

### Technical Limitations
- **Manual Type Control**: `Fix Data Type Mismatches = No`
- **Session Settings**: Week of year policy = 1
- **Timestamp Precision**: `CURRENT_TIMESTAMP(0)` for consistency

### Best Practices
- **Naming Conventions**: `jb_` prefix for jobs
- **Audit Trail**: Every job must have audit logging
- **Error Handling**: All components need success/failure paths
- **Documentation**: Job descriptions in metadata