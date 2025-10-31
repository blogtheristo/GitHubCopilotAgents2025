# Skill: Data Processing

## Overview
Comprehensive data processing skill enabling agents to extract, transform, and load (ETL) data from various sources for analysis and business intelligence.

## Capabilities
- Data extraction from multiple sources
- Data cleaning and normalization
- Data transformation and enrichment
- Data validation and quality checks
- Real-time streaming data processing
- Batch data processing

## Technical Components

### 1. Data Extraction
```python
# Example: Multi-source data extraction
import pandas as pd
from typing import Dict, Any

class DataExtractor:
    """Extracts data from various sources"""
    
    def extract_from_database(self, connection_string: str, query: str) -> pd.DataFrame:
        """Extract data from SQL database"""
        import sqlalchemy
        engine = sqlalchemy.create_engine(connection_string)
        return pd.read_sql(query, engine)
    
    def extract_from_api(self, endpoint: str, params: Dict[str, Any]) -> pd.DataFrame:
        """Extract data from REST API"""
        import requests
        response = requests.get(endpoint, params=params)
        return pd.DataFrame(response.json())
    
    def extract_from_file(self, file_path: str, file_type: str) -> pd.DataFrame:
        """Extract data from files (CSV, JSON, Excel)"""
        if file_type == 'csv':
            return pd.read_csv(file_path)
        elif file_type == 'json':
            return pd.read_json(file_path)
        elif file_type == 'excel':
            return pd.read_excel(file_path)
        else:
            raise ValueError(f"Unsupported file type: {file_type}")
```

### 2. Data Transformation
```python
# Example: Data cleaning and transformation
class DataTransformer:
    """Transforms and cleans data"""
    
    def clean_data(self, df: pd.DataFrame) -> pd.DataFrame:
        """Remove duplicates, handle missing values, fix data types"""
        # Remove duplicates
        df = df.drop_duplicates()
        
        # Handle missing values
        numeric_columns = df.select_dtypes(include=['number']).columns
        df[numeric_columns] = df[numeric_columns].fillna(df[numeric_columns].mean())
        
        categorical_columns = df.select_dtypes(include=['object']).columns
        df[categorical_columns] = df[categorical_columns].fillna('Unknown')
        
        return df
    
    def enrich_data(self, df: pd.DataFrame, enrichment_source: str) -> pd.DataFrame:
        """Add additional information from external sources"""
        # Example: Add geographic data, demographic info, etc.
        # This is a placeholder - actual implementation would call external APIs
        return df
    
    def aggregate_data(self, df: pd.DataFrame, group_by: list, agg_functions: dict) -> pd.DataFrame:
        """Aggregate data by specified columns"""
        return df.groupby(group_by).agg(agg_functions).reset_index()
```

### 3. Data Validation
```python
# Example: Data quality checks
class DataValidator:
    """Validates data quality"""
    
    def validate_schema(self, df: pd.DataFrame, expected_schema: Dict[str, str]) -> Dict[str, Any]:
        """Check if dataframe matches expected schema"""
        issues = []
        
        # Check for missing columns
        missing_cols = set(expected_schema.keys()) - set(df.columns)
        if missing_cols:
            issues.append(f"Missing columns: {missing_cols}")
        
        # Check data types
        for col, expected_type in expected_schema.items():
            if col in df.columns:
                actual_type = str(df[col].dtype)
                if actual_type != expected_type:
                    issues.append(f"Column {col}: expected {expected_type}, got {actual_type}")
        
        return {
            "valid": len(issues) == 0,
            "issues": issues
        }
    
    def check_data_quality(self, df: pd.DataFrame) -> Dict[str, Any]:
        """Perform data quality checks"""
        return {
            "row_count": len(df),
            "null_percentages": (df.isnull().sum() / len(df) * 100).to_dict(),
            "duplicate_rows": df.duplicated().sum(),
            "unique_values": {col: df[col].nunique() for col in df.columns}
        }
```

## Integration Patterns

### ETL Pipeline Configuration
```json
{
  "pipeline": "customer_data_pipeline",
  "schedule": "daily_01:00",
  "steps": [
    {
      "type": "extract",
      "source": "postgresql",
      "query": "SELECT * FROM customers WHERE updated_at > :last_run",
      "connection": "prod_db"
    },
    {
      "type": "transform",
      "operations": [
        "clean_data",
        "normalize_addresses",
        "calculate_customer_lifetime_value"
      ]
    },
    {
      "type": "validate",
      "rules": [
        "no_null_email",
        "valid_phone_format",
        "positive_ltv"
      ]
    },
    {
      "type": "load",
      "destination": "data_warehouse",
      "table": "customers_processed",
      "mode": "upsert"
    }
  ],
  "error_handling": {
    "on_failure": "alert_and_retry",
    "max_retries": 3,
    "notification_channel": "slack"
  }
}
```

## Use Cases
1. **Customer Data Integration**: Merge data from CRM, support, and sales
2. **Financial Reporting**: Aggregate transaction data for reports
3. **Log Analysis**: Process and analyze application logs
4. **Real-time Monitoring**: Stream processing for live dashboards
5. **Data Migration**: Transfer data between systems

## Supported Data Sources
- SQL Databases (PostgreSQL, MySQL, SQL Server)
- NoSQL Databases (MongoDB, Cassandra)
- Cloud Storage (S3, Azure Blob, Google Cloud Storage)
- APIs (REST, GraphQL)
- Files (CSV, JSON, Excel, Parquet)
- Streaming (Kafka, RabbitMQ, AWS Kinesis)

## Dependencies
- Pandas, NumPy for data manipulation
- SQLAlchemy for database connections
- Apache Airflow or Prefect for orchestration
- Spark for big data processing
- Data validation libraries (Great Expectations)

## Performance Optimization
- Chunked processing for large datasets
- Parallel processing with multiprocessing/Dask
- Incremental loads (process only new/changed data)
- Caching frequently accessed data
- Index optimization for database queries
