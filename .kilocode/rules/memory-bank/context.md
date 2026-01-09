# Current Context

## Project State
The Matillion ETL project is a **mature, production system** running on version 1.68.19 in a Snowflake environment. The system is actively processing data from 20+ source systems into an enterprise data warehouse.

## Current Focus
- **Maintenance Mode**: The system is in production with established patterns and workflows
- **Data Pipeline Operations**: Daily/weekly scheduled jobs running for various business domains
- **Integration Management**: Multiple source systems feeding into Snowflake data warehouse

## Recent Understanding
Based on the initialization analysis:
- The project follows a well-established three-layer architecture (Landing → Dimension → Fact)
- All jobs follow standardized audit patterns for tracking and monitoring
- SCD Type 2 is implemented for historical tracking in dimension tables
- The system uses both orchestration and transformation job types in JSON format

## Key Observations

### Strengths
1. **Consistent Patterns**: All jobs follow similar audit and error handling patterns
2. **Comprehensive Coverage**: Covers finance, HR, marketing, sales, and operations domains
3. **Modular Design**: Jobs are organized by source system and functional area
4. **Audit Trail**: Complete logging and tracking of all ETL operations

### Areas of Complexity
1. **Multiple Source Systems**: 20+ different integration patterns to maintain
2. **Business Logic**: Complex hour conversion rules in fact aggregations
3. **SCD Type 2**: Historical tracking requires careful MERGE/INSERT logic
4. **Orchestration Hierarchy**: Multi-level job dependencies in EDW layer

## Next Steps
When working on this project, typical tasks might include:
- Adding new data source integrations
- Modifying existing transformation logic
- Creating new dimension or fact tables
- Updating audit or error handling patterns
- Optimizing performance of existing jobs
- Troubleshooting failed job executions

## Important Notes
- Always follow the established naming conventions (`jb_` prefix)
- Every job must include audit logging (Begin/Success/Failure)
- Use incremental loading where possible (OBJ_ID or date-based)
- Test SCD Type 2 logic carefully to avoid data corruption
- Verify environment variables are correctly set (${DB_Datalake}, ${SNFLKSCH}, etc.)