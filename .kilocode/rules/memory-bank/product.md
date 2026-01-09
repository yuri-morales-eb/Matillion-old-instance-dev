# Product Overview

## Purpose
This is a **Matillion ETL (Extract, Transform, Load) data pipeline system** designed to ingest, transform, and load data from multiple heterogeneous data sources into Snowflake data warehouse. The system serves as the central data integration platform for enterprise data warehousing and analytics.

## Problems It Solves

### Data Integration Challenges
- **Multi-Source Data Consolidation**: Integrates data from 20+ different source systems including:
  - ERP Systems (Lawson)
  - CRM Systems (Microsoft Dynamics CRM, Salesforce)
  - HR Systems (UKG)
  - Marketing Platforms (Google Analytics, Indeed, Career Builder, Zip Recruiter)
  - Financial Systems (Essbase, Concur)
  - Third-party Services (Equifax, Demandbase, Kite)

- **Data Quality & Consistency**: Ensures data quality through:
  - Standardized transformation patterns
  - Audit logging and tracking
  - Data validation and error handling
  - SCD Type 2 (Slowly Changing Dimensions) for historical tracking

- **Enterprise Data Warehouse**: Builds a comprehensive EDW (Enterprise Data Warehouse) with:
  - Dimensional modeling (Facts and Dimensions)
  - Landing layer for raw data ingestion
  - Transformation layer for business logic
  - Presentation layer for analytics and reporting

## How It Works

### Architecture Layers

1. **Landing Layer (LDG_*)**: 
   - Raw data ingestion from source systems
   - Minimal transformation
   - Audit columns added (DL_CREATED_DATE, DL_CREATED_BY, DL_JOBID)

2. **Dimension Layer (DIM_*)**: 
   - Master data and reference tables
   - Customer, Employee, Activity, Branch, Region dimensions
   - SCD Type 2 implementation for historical tracking

3. **Fact Layer (FACT_*)**: 
   - Transactional and aggregated data
   - Financial actuals, budgets, headcount
   - Performance metrics and KPIs

4. **EDW Layer**: 
   - Enterprise-wide consolidated views
   - Cross-functional analytics
   - Power BI security and access control

### Job Types

**Orchestration Jobs (.ORCHESTRATION)**:
- Workflow coordination and sequencing
- Error handling and branching logic
- Parallel execution management
- Audit logging integration

**Transformation Jobs (.TRANSFORMATION)**:
- Data transformation logic
- Column mapping and calculations
- Data type conversions
- Business rule implementation

### Key Patterns

1. **Audit Pattern**: Every job includes:
   - Start audit logging
   - Main processing
   - Success/Failure audit logging
   - Row count tracking

2. **Incremental Loading**: 
   - Uses OBJ_ID or timestamp-based incremental logic
   - Avoids full table reloads
   - Optimizes performance

3. **Error Handling**:
   - Success/Failure connectors
   - Separate audit paths for success and failure
   - Graceful degradation

## User Experience Goals

### For Data Engineers
- **Maintainability**: Clear naming conventions (jb_* prefix for jobs)
- **Debuggability**: Comprehensive audit trails
- **Reusability**: Standardized transformation patterns
- **Scalability**: Modular job design for easy extension

### For Business Users
- **Data Availability**: Scheduled daily/weekly data refreshes
- **Data Quality**: Validated and cleansed data
- **Historical Tracking**: Point-in-time analysis capabilities
- **Performance**: Optimized queries through proper dimensional modeling

### For Analysts
- **Consistent Data Model**: Standardized dimensions and facts
- **Power BI Integration**: Security and access control
- **Comprehensive Coverage**: All business domains represented
- **Timely Updates**: Near real-time to daily refresh cycles

## Key Business Domains

1. **Finance & Accounting**: GL transactions, budgets, actuals, allocations
2. **Human Resources**: Employee data, headcount, payroll, requisitions
3. **Sales & Marketing**: Job board performance, campaigns, conversions
4. **Operations**: Activity tracking, resource utilization, assignments
5. **Customer Management**: Client hierarchies, account types, relationships