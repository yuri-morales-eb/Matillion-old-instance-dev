# System Architecture

## Project Structure

### Root Directory: `ROOT/PROD/`
All production ETL jobs are organized under this directory with subdirectories by data source or functional area.

### Key Directories

#### Data Source Directories
- **`Career Builder/`**: Job board data ingestion
- **`CELLULAR_SALES/`**: Cellular sales candidate, job order, and placement data
- **`Concur Data/`**: Expense management data
- **`DEMANDBASE/`**: Marketing intelligence data
- **`EB Competitors/`**: Competitor analysis data
- **`Essbase/`**: Financial planning and budgeting exports
- **`EQUIFAX/`**: Credit and background check data
- **`Google Analytics/`**: Web analytics (GA and GA4)
- **`Indeed/`**: Job board sponsored jobs data
- **`KITE/`**: Industry and client mapping data
- **`SalesForce/`**: CRM data
- **`UKG/`**: HR and workforce management data
- **`Zip Recruiter/`**: Job board data

#### Functional Directories
- **`LANDING_JOBS/`**: Landing layer jobs for raw data ingestion
- **`DIM_JOBS/`**: Dimension table population jobs
- **`FACT_JOBS/`**: Fact table population jobs
- **`EDW/`**: Enterprise Data Warehouse orchestration
- **`AUDIT_JOBS/`**: Audit and logging jobs
- **`Finance/`**: Financial reporting and allocations
- **`PowerBI/`**: Power BI security and access control

## File Naming Conventions

### Job Prefixes
- **`jb_`**: Standard job prefix (e.g., `jb_DIM_CUSTOMER.ORCHESTRATION`)
- **`Audit_`**: Audit-related transformation jobs
- **`FINAL_`**: Final orchestration jobs that coordinate multiple sub-jobs

### File Extensions
- **`.ORCHESTRATION`**: Workflow orchestration jobs (JSON format)
- **`.TRANSFORMATION`**: Data transformation jobs (JSON format)

### Naming Patterns
- **Landing Jobs**: `jb_[SOURCE]_DL_LDG_[TABLE].ORCHESTRATION`
  - Example: `jb_LAWSON_DL_LDG_EMPLOYEE.ORCHESTRATION`
- **Dimension Jobs**: `jb_DIM_[DIMENSION_NAME].ORCHESTRATION`
  - Example: `jb_DIM_CUSTOMER.ORCHESTRATION`
- **Fact Jobs**: `jb_FACT_[FACT_NAME].ORCHESTRATION`
  - Example: `jb_FACT_AGG_ACTUALS.ORCHESTRATION`
- **EDW Jobs**: `EDW_Orchestration_[AREA].ORCHESTRATION`
  - Example: `EDW_Orchestration_DIMS.ORCHESTRATION`

## Job Structure (JSON Format)

### Orchestration Job Components
```json
{
  "job": {
    "components": {
      "componentId": {
        "id": "unique_id",
        "implementationID": "component_type",
        "parameters": {},
        "inputConnectorIDs": [],
        "outputSuccessConnectorIDs": [],
        "outputFailureConnectorIDs": []
      }
    },
    "successConnectors": {},
    "failureConnectors": {},
    "unconditionalConnectors": {}
  }
}
```

### Common Component Types
- **`444132438`**: Start component
- **`1785813072`**: Run Orchestration component
- **`1896325668`**: Run Transformation component
- **`-798585337`**: SQL Script component
- **`-1032749985`**: Query component
- **`-741198691`**: Database Query (external source to Snowflake)
- **`-1946388514`**: End Success component
- **`515156205`**: End Failure component

### Transformation Job Components
- **`1354890871`**: Table Input component
- **`1716658327`**: Calculator component
- **`211954775`**: Table Output component

## Data Flow Architecture

### Three-Layer Architecture

#### 1. Landing Layer (LDG_*)
**Purpose**: Raw data ingestion with minimal transformation

**Pattern**:
```
Source System → Database Query → Add Audit Columns → Landing Table
```

**Audit Columns Added**:
- `DL_CREATED_DATE`: Timestamp of data load
- `DL_CREATED_BY`: User who executed the job
- `DL_JOBID`: Matillion job execution ID (${run_history_id})
- `IS_ACTIVE`: Active status flag ('Y'/'N')

**Example**: `jb_LAWSON_DL_LDG_EMPLOYEE.ORCHESTRATION`
- Reads from Lawson SQL Server
- Adds audit columns
- Writes to `LAWSON_DL_LDG_EMPLOYEE` table

#### 2. Dimension Layer (DIM_*)
**Purpose**: Master data with SCD Type 2 historical tracking

**Pattern**:
```
Landing Tables → Business Logic → MERGE/INSERT → Dimension Table
```

**SCD Type 2 Implementation**:
- `CREATED_DT`: Record creation timestamp
- `MODIFIED_DT`: Last modification timestamp
- `ACTIVE_STATUS`: 'Y' for current, 'N' for historical
- When changes detected: Set old record to 'N', insert new record with 'Y'

**Example**: `jb_DIM_CUSTOMER.ORCHESTRATION`
- Uses Python script with cursor.execute()
- MERGE statement to update existing records
- INSERT for new versions when changes detected

#### 3. Fact Layer (FACT_*)
**Purpose**: Transactional and aggregated metrics

**Pattern**:
```
Dimension Tables → Complex SQL → DELETE old data → INSERT new data → Fact Table
```

**Example**: `jb_FACT_AGG_ACTUALS.ORCHESTRATION`
- Deletes existing actuals data
- Aggregates from DIM_TRANS and DIM_GL_TRANS
- Applies business rules for hour conversions
- Inserts aggregated results

#### 4. EDW Layer
**Purpose**: Enterprise-wide consolidated views

**Orchestration Hierarchy**:
```
EDW_Orchestration.ORCHESTRATION (Master)
├── EDW_Orchestration_ODS.ORCHESTRATION
├── EDW_Orchestration_DIMS.ORCHESTRATION
├── EDW_Orchestration_FACTS_BRIDGES.ORCHESTRATION
├── EDW_Orchestration_TRANSACTIONS.ORCHESTRATION
├── EDW_Orchestration_ASSIGNMENTS.ORCHESTRATION
├── EDW_Orchestration_MARKETING_DIMS.ORCHESTRATION
├── EDW_Orchestration_MARKETING_FACTS_BRIDGES.ORCHESTRATION
└── EDW_Orchestration_KPI_FACTS.ORCHESTRATION
```

## Audit Pattern

### Standard Audit Flow
Every orchestration job follows this pattern:

```
Start → Audit Begin → Main Processing → Audit Success/Failure → End
```

### Audit Components
1. **Audit Begin**: Logs job start
   - Sets `audit_status = 'RUNNING'`
   - Records `audit_started` timestamp
   
2. **Main Processing**: Core business logic
   - Exports `audit_rowcount`, `audit_status`, `audit_component`
   
3. **Audit Success**: Logs successful completion
   - Sets `audit_status = 'SUCCESS'`
   - Records row counts and completion time
   
4. **Audit Failure**: Logs failure
   - Sets `audit_status = 'FAILURE'`
   - Records error details

### Audit Transformation Jobs
- `_Audit_Datalake_Public_Begin`: Start logging
- `_Audit_Datalake_Public_End`: End logging
- `Audit_[JOB_NAME]`: Job-specific audit transformations

## Database Architecture

### Snowflake Databases
- **`PROD_DATALAKE`**: Main data lake database
  - Schema: `LAWPROD` (Lawson data)
  - Schema: `CRM_MSCRM` (CRM data)
  - Schema: Various source-specific schemas

- **`PROD_NPG_VIP`**: NPG VIP data
  - Schema: `LDG` (Landing)

### Environment Variables
- `${DB_Datalake}`: Database name
- `${DB_DatalakeOrig}`: Original database reference
- `${SNFLKDB}`: Snowflake database
- `${SNFLKSCH}`: Snowflake schema
- `${Sch_[SOURCE]}`: Source-specific schema names
- `${run_history_id}`: Current job execution ID
- `${environment_username}`: Current user

## Integration Patterns

### External Database Connections
**SQL Server (Lawson)**:
- Connection: `${SQLConnLaw}`
- Credentials: `DATALAKE_PROD`
- Pattern: Query → Stage → Transform → Load

### File-Based Integrations
**Career Builder, Indeed, Zip Recruiter**:
- Pattern: File → Landing Table → Parse → Transform → Target Table
- File naming convention includes date: `filename_YYYYMMDD.csv`

### API-Based Integrations
**Google Analytics, UKG**:
- Pattern: API Call → JSON/XML → Parse → Landing → Transform

### Essbase Exports
**Financial Planning**:
- Pattern: Snowflake Query → Format → Export to Essbase
- Jobs: LPM, WBS, Headcount exports
- Frequency: Weekly/Monthly

## Key Technical Decisions

### Incremental Loading Strategy
- **OBJ_ID-based**: Uses `WHERE OBJ_ID > ${Max_ObjId}` for Lawson data
- **Date-based**: Uses posting dates for time-bound queries
- **Full Refresh**: Some dimensions use full refresh with SCD Type 2

### Data Type Handling
- Explicit CAST operations for type conversions
- `Fix Data Type Mismatches = No` (manual control)
- Timestamp conversions: `TO_TIMESTAMP_NTZ(CURRENT_TIMESTAMP(0))`

### Performance Optimization
- Warehouse sizing: `WAREHOUSE_RESIZE.ORCHESTRATION`
- Parallel execution: Multiple jobs run concurrently in orchestrations
- Staging: Snowflake Managed staging for bulk loads
- Compression: Gzip compression for data transfer

### Error Handling
- Success/Failure connectors on all components
- Separate audit paths for success and failure
- Error aggregation component: `-1343684451` (And/Or component)
- Graceful degradation: Jobs continue on non-critical failures

## Critical Implementation Paths

### Daily Data Refresh Flow
1. **Landing Jobs** (LANDING_JOBS directory)
   - Extract from source systems
   - Load to landing tables
   
2. **Dimension Jobs** (DIM_JOBS directory)
   - Update master data
   - Apply SCD Type 2 logic
   
3. **Fact Jobs** (FACT_JOBS directory)
   - Aggregate transactional data
   - Calculate metrics
   
4. **EDW Jobs** (EDW directory)
   - Consolidate enterprise views
   - Update Power BI security

### Complete Job Orchestrations
- **`FINAL_BOARD_COMPLETE_JOB.ORCHESTRATION`**: Job board data complete workflow
- **`FINAL_FPR_COMPLETE_JOB.ORCHESTRATION`**: FPR complete workflow
- **`BOARD_COMPLETE_JOB.ORCHESTRATION`**: Board data orchestration

## Version Information
- **Matillion Version**: 1.68.19
- **Environment**: Snowflake
- **Build File**: `.build_version`