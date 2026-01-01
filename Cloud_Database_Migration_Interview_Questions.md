# Database & Cloud Migration Interview Questions

## 1. How to Migrate Local MSSQL Database to AWS Aurora PostgreSQL
**Concept:** Migrating from Microsoft SQL Server (MSSQL) to AWS Aurora PostgreSQL involves schema conversion, data migration, and validation.

### Migration Approaches:

#### Approach 1: AWS Database Migration Service (DMS) - Recommended
AWS DMS is a managed service that makes database migration easy and automated.

**Steps:**

**1. Prerequisites:**
```sql
-- Enable CDC (Change Data Capture) on source MSSQL
USE YourDatabase;
EXEC sys.sp_cdc_enable_db;

-- Enable CDC on specific tables
EXEC sys.sp_cdc_enable_table
  @source_schema = 'dbo',
  @source_name = 'YourTable',
  @role_name = NULL;
```

**2. Schema Conversion using AWS SCT (Schema Conversion Tool):**
```bash
# Install AWS Schema Conversion Tool
# Download from: https://aws.amazon.com/dms/schema-conversion-tool/

# Steps in AWS SCT:
1. Connect to source MSSQL database
2. Connect to target Aurora PostgreSQL
3. Create mapping rules
4. Convert schema (handles data type conversions)
5. Review conversion report
6. Apply schema to target
```

**3. Create Replication Instance:**
```bash
# Using AWS CLI
aws dms create-replication-instance \
  --replication-instance-identifier my-replication-instance \
  --replication-instance-class dms.t3.medium \
  --allocated-storage 50 \
  --engine-version 3.4.6 \
  --vpc-security-group-ids sg-xxxxx \
  --replication-subnet-group-identifier my-subnet-group
```

**4. Create Source and Target Endpoints:**
```bash
# Source endpoint (MSSQL)
aws dms create-endpoint \
  --endpoint-identifier mssql-source \
  --endpoint-type source \
  --engine-name sqlserver \
  --username admin \
  --password YourPassword \
  --server-name your-mssql-server.example.com \
  --port 1433 \
  --database-name YourDatabase

# Target endpoint (Aurora PostgreSQL)
aws dms create-endpoint \
  --endpoint-identifier aurora-postgres-target \
  --endpoint-type target \
  --engine-name aurora-postgresql \
  --username postgres \
  --password YourPassword \
  --server-name your-aurora-cluster.cluster-xxxxx.us-east-1.rds.amazonaws.com \
  --port 5432 \
  --database-name YourDatabase
```

**5. Create and Run Migration Task:**
```bash
# Create migration task
aws dms create-replication-task \
  --replication-task-identifier migration-task \
  --source-endpoint-arn arn:aws:dms:us-east-1:xxx:endpoint:mssql-source \
  --target-endpoint-arn arn:aws:dms:us-east-1:xxx:endpoint:aurora-postgres-target \
  --replication-instance-arn arn:aws:dms:us-east-1:xxx:rep:xxxxx \
  --migration-type full-load-and-cdc \
  --table-mappings file://table-mappings.json \
  --replication-task-settings file://task-settings.json

# Start the task
aws dms start-replication-task \
  --replication-task-arn arn:aws:dms:us-east-1:xxx:task:xxxxx \
  --start-replication-task-type start-replication
```

**Table Mappings (table-mappings.json):**
```json
{
  "rules": [
    {
      "rule-type": "selection",
      "rule-id": "1",
      "rule-name": "include-all-tables",
      "object-locator": {
        "schema-name": "dbo",
        "table-name": "%"
      },
      "rule-action": "include"
    },
    {
      "rule-type": "transformation",
      "rule-id": "2",
      "rule-name": "convert-schema-name",
      "rule-target": "schema",
      "object-locator": {
        "schema-name": "dbo"
      },
      "rule-action": "rename",
      "value": "public"
    }
  ]
}
```

**6. Monitor Migration:**
```bash
# Check task status
aws dms describe-replication-tasks \
  --filters Name=replication-task-arn,Values=arn:aws:dms:us-east-1:xxx:task:xxxxx

# View table statistics
aws dms describe-table-statistics \
  --replication-task-arn arn:aws:dms:us-east-1:xxx:task:xxxxx
```

---

#### Approach 2: Manual Migration (For Small Databases)

**1. Export from MSSQL:**
```sql
-- Generate INSERT scripts
-- Using SQL Server Management Studio (SSMS):
-- Right-click database → Tasks → Generate Scripts
-- Select specific objects or entire database
-- Advanced → Types of data to script → Data only or Schema and data
```

**2. Convert Data Types:**
| MSSQL Type | PostgreSQL Type |
|------------|----------------|
| `INT` | `INTEGER` |
| `BIGINT` | `BIGINT` |
| `VARCHAR(n)` | `VARCHAR(n)` |
| `NVARCHAR(n)` | `VARCHAR(n)` |
| `DATETIME` | `TIMESTAMP` |
| `BIT` | `BOOLEAN` |
| `UNIQUEIDENTIFIER` | `UUID` |
| `MONEY` | `DECIMAL` |
| `IMAGE` | `BYTEA` |

**3. Schema Conversion Example:**
```sql
-- MSSQL Schema
CREATE TABLE Users (
  Id INT IDENTITY(1,1) PRIMARY KEY,
  Username NVARCHAR(50) NOT NULL,
  Email NVARCHAR(100) UNIQUE,
  CreatedDate DATETIME DEFAULT GETDATE(),
  IsActive BIT DEFAULT 1
);

-- PostgreSQL Schema
CREATE TABLE Users (
  Id SERIAL PRIMARY KEY,
  Username VARCHAR(50) NOT NULL,
  Email VARCHAR(100) UNIQUE,
  CreatedDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  IsActive BOOLEAN DEFAULT TRUE
);
```

**4. Export Data to CSV:**
```sql
-- MSSQL: Export to CSV
bcp "SELECT * FROM Users" queryout "users.csv" -c -t, -S localhost -U sa -P password -d YourDatabase
```

**5. Import to PostgreSQL:**
```sql
-- PostgreSQL: Import from CSV
COPY Users(Id, Username, Email, CreatedDate, IsActive)
FROM '/path/to/users.csv'
DELIMITER ','
CSV HEADER;

-- Reset sequence after import
SELECT setval('users_id_seq', (SELECT MAX(id) FROM Users));
```

---

#### Approach 3: Using Third-Party Tools

**Tools:**
1. **AWS SCT (Schema Conversion Tool)** - Free, AWS provided
2. **pgloader** - Open source, command-line tool
3. **Flyway** - For version-controlled migrations
4. **Liquibase** - Database-independent migrations

**Example using pgloader:**
```bash
# Install pgloader
sudo apt-get install pgloader

# Create migration script (migration.load)
cat > migration.load << EOF
LOAD DATABASE
  FROM mssql://user:password@localhost/SourceDB
  INTO postgresql://user:password@aurora-endpoint/TargetDB

WITH include drop, create tables, create indexes, reset sequences

SET work_mem to '256MB', maintenance_work_mem to '512 MB'

CAST type datetime to timestamp
     drop default drop not null using zero-dates-to-null,
     type nvarchar to varchar drop typemod,
     type money to decimal drop typemod;
EOF

# Run migration
pgloader migration.load
```

---

### Post-Migration Validation:

**1. Verify Row Counts:**
```sql
-- MSSQL
SELECT 'Users' AS TableName, COUNT(*) AS RowCount FROM Users
UNION ALL
SELECT 'Orders', COUNT(*) FROM Orders;

-- PostgreSQL
SELECT 'Users' AS TableName, COUNT(*) AS RowCount FROM Users
UNION ALL
SELECT 'Orders', COUNT(*) FROM Orders;
```

**2. Compare Data Checksums:**
```sql
-- Sample records comparison
-- MSSQL
SELECT TOP 10 * FROM Users ORDER BY Id;

-- PostgreSQL
SELECT * FROM Users ORDER BY Id LIMIT 10;
```

**3. Test Application:**
```javascript
// Update connection strings in application
// From MSSQL:
// const config = {
//   user: 'sa',
//   password: 'password',
//   server: 'localhost',
//   database: 'YourDB'
// };

// To PostgreSQL:
const config = {
  user: 'postgres',
  password: 'password',
  host: 'your-aurora-cluster.cluster-xxxxx.us-east-1.rds.amazonaws.com',
  port: 5432,
  database: 'YourDB'
};
```

---

## 2. How to Sync Local MSSQL to AWS Aurora PostgreSQL Daily
**Concept:** Set up continuous data synchronization from on-premises MSSQL to Aurora PostgreSQL.

### Approach 1: AWS DMS with Ongoing Replication (CDC)

**1. Enable Change Data Capture (CDC) on MSSQL:**
```sql
-- Enable CDC on database
USE YourDatabase;
EXEC sys.sp_cdc_enable_db;

-- Enable CDC on tables
EXEC sys.sp_cdc_enable_table
  @source_schema = N'dbo',
  @source_name = N'Users',
  @role_name = NULL,
  @supports_net_changes = 1;

-- Verify CDC is enabled
SELECT name, is_cdc_enabled 
FROM sys.databases 
WHERE name = 'YourDatabase';
```

**2. Configure AWS DMS for Continuous Replication:**
```bash
# Create replication task with CDC
aws dms create-replication-task \
  --replication-task-identifier daily-sync-task \
  --source-endpoint-arn arn:aws:dms:us-east-1:xxx:endpoint:mssql-source \
  --target-endpoint-arn arn:aws:dms:us-east-1:xxx:endpoint:aurora-target \
  --replication-instance-arn arn:aws:dms:us-east-1:xxx:rep:xxxxx \
  --migration-type full-load-and-cdc \
  --table-mappings file://table-mappings.json \
  --replication-task-settings file://cdc-settings.json
```

**CDC Settings (cdc-settings.json):**
```json
{
  "TargetMetadata": {
    "TargetSchema": "public",
    "SupportLobs": true,
    "FullLobMode": false,
    "LobChunkSize": 64,
    "LimitedSizeLobMode": true,
    "LobMaxSize": 32
  },
  "FullLoadSettings": {
    "TargetTablePrepMode": "DROP_AND_CREATE",
    "CreatePkAfterFullLoad": true,
    "MaxFullLoadSubTasks": 8
  },
  "ChangeProcessingDdlHandlingPolicy": {
    "HandleSourceTableDropped": true,
    "HandleSourceTableTruncated": true,
    "HandleSourceTableAltered": true
  },
  "ChangeProcessingTuning": {
    "BatchApplyPreserveTransaction": true,
    "BatchApplyTimeoutMin": 1,
    "BatchApplyTimeoutMax": 30,
    "BatchApplyMemoryLimit": 500,
    "BatchSplitSize": 0,
    "MinTransactionSize": 1000,
    "CommitTimeout": 1,
    "MemoryLimitTotal": 1024,
    "MemoryKeepTime": 60,
    "StatementCacheSize": 50
  },
  "ValidationSettings": {
    "EnableValidation": true,
    "ValidationMode": "ROW_LEVEL",
    "ThreadCount": 5
  },
  "Logging": {
    "EnableLogging": true,
    "LogComponents": [
      {
        "Id": "TRANSFORMATION",
        "Severity": "LOGGER_SEVERITY_DEFAULT"
      },
      {
        "Id": "SOURCE_CAPTURE",
        "Severity": "LOGGER_SEVERITY_INFO"
      },
      {
        "Id": "TARGET_APPLY",
        "Severity": "LOGGER_SEVERITY_INFO"
      }
    ]
  }
}
```

**3. Monitor Replication Lag:**
```bash
# Check replication lag
aws dms describe-replication-tasks \
  --filters Name=replication-task-arn,Values=arn:aws:dms:xxx:task:xxxxx \
  --query 'ReplicationTasks[0].[CDCStartPosition,CDCStopPosition,ReplicationTaskStats]'
```

**4. Set up CloudWatch Alarms:**
```bash
# Create alarm for replication lag
aws cloudwatch put-metric-alarm \
  --alarm-name dms-replication-lag \
  --alarm-description "Alert when replication lag exceeds threshold" \
  --metric-name CDCLatencyTarget \
  --namespace AWS/DMS \
  --statistic Average \
  --period 300 \
  --threshold 300 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=ReplicationTaskIdentifier,Value=daily-sync-task
```

---

### Approach 2: Scheduled ETL Job (For Non-Real-Time Sync)

**Using AWS Lambda + EventBridge (CloudWatch Events):**

**1. Lambda Function (Python):**
```python
import boto3
import pyodbc
import psycopg2
from datetime import datetime, timedelta

def lambda_handler(event, context):
    # Source MSSQL connection
    mssql_conn = pyodbc.connect(
        'DRIVER={ODBC Driver 17 for SQL Server};'
        'SERVER=your-mssql-server;'
        'DATABASE=YourDB;'
        'UID=user;'
        'PWD=password'
    )
    
    # Target Aurora PostgreSQL connection
    pg_conn = psycopg2.connect(
        host='aurora-cluster.xxxxx.us-east-1.rds.amazonaws.com',
        port=5432,
        database='YourDB',
        user='postgres',
        password='password'
    )
    
    try:
        # Get data changed in last 24 hours
        yesterday = datetime.now() - timedelta(days=1)
        
        mssql_cursor = mssql_conn.cursor()
        query = f"""
            SELECT Id, Username, Email, UpdatedDate
            FROM Users
            WHERE UpdatedDate >= ?
        """
        mssql_cursor.execute(query, yesterday)
        
        rows = mssql_cursor.fetchall()
        
        # Upsert into PostgreSQL
        pg_cursor = pg_conn.cursor()
        
        for row in rows:
            upsert_query = """
                INSERT INTO Users (Id, Username, Email, UpdatedDate)
                VALUES (%s, %s, %s, %s)
                ON CONFLICT (Id) 
                DO UPDATE SET 
                    Username = EXCLUDED.Username,
                    Email = EXCLUDED.Email,
                    UpdatedDate = EXCLUDED.UpdatedDate
            """
            pg_cursor.execute(upsert_query, (row.Id, row.Username, row.Email, row.UpdatedDate))
        
        pg_conn.commit()
        
        return {
            'statusCode': 200,
            'body': f'Successfully synced {len(rows)} records'
        }
    
    except Exception as e:
        print(f"Error: {str(e)}")
        raise
    
    finally:
        mssql_conn.close()
        pg_conn.close()
```

**2. EventBridge Rule (Daily at 2 AM):**
```bash
# Create EventBridge rule
aws events put-rule \
  --name daily-db-sync \
  --schedule-expression "cron(0 2 * * ? *)" \
  --state ENABLED \
  --description "Daily database sync at 2 AM UTC"

# Add Lambda as target
aws events put-targets \
  --rule daily-db-sync \
  --targets "Id"="1","Arn"="arn:aws:lambda:us-east-1:xxx:function:db-sync-function"

# Grant EventBridge permission to invoke Lambda
aws lambda add-permission \
  --function-name db-sync-function \
  --statement-id allow-eventbridge \
  --action lambda:InvokeFunction \
  --principal events.amazonaws.com \
  --source-arn arn:aws:events:us-east-1:xxx:rule/daily-db-sync
```

---

### Approach 3: Using Apache NiFi (For Complex ETL)

**NiFi Flow:**
```
QueryDatabaseTable (MSSQL)
    ↓
ConvertRecord (Convert to JSON)
    ↓
RouteOnAttribute (Filter changed records)
    ↓
PutDatabaseRecord (Aurora PostgreSQL)
    ↓
LogAttribute (Success/Failure)
```

**Configuration:**
```yaml
# NiFi Processors Configuration
QueryDatabaseTable:
  Database Type: MS SQL 2012+
  Database Connection URL: jdbc:sqlserver://localhost:1433;databaseName=YourDB
  Table Name: Users
  Columns to Return: Id, Username, Email, UpdatedDate
  Maximum Value Columns: UpdatedDate
  Schedule: 0 0 2 * * ? (Daily at 2 AM)

PutDatabaseRecord:
  Database Type: PostgreSQL
  Database Connection URL: jdbc:postgresql://aurora-endpoint:5432/YourDB
  Table Name: Users
  Statement Type: UPSERT
```

---

### Approach 4: Using Cron Job + Custom Script

**Shell Script (sync-db.sh):**
```bash
#!/bin/bash

# Configuration
SOURCE_HOST="mssql-server"
SOURCE_DB="YourDB"
TARGET_HOST="aurora-cluster.xxxxx.us-east-1.rds.amazonaws.com"
TARGET_DB="YourDB"
LOG_FILE="/var/log/db-sync.log"

echo "[$(date)] Starting database sync..." >> $LOG_FILE

# Export changed data from MSSQL to CSV
bcp "SELECT * FROM Users WHERE UpdatedDate >= DATEADD(day, -1, GETDATE())" queryout /tmp/users_delta.csv -c -t, -S $SOURCE_HOST -U sa -P password -d $SOURCE_DB

# Import to PostgreSQL
psql -h $TARGET_HOST -U postgres -d $TARGET_DB << EOF
CREATE TEMP TABLE temp_users (LIKE Users INCLUDING ALL);

COPY temp_users FROM '/tmp/users_delta.csv' DELIMITER ',' CSV;

INSERT INTO Users (Id, Username, Email, UpdatedDate)
SELECT Id, Username, Email, UpdatedDate FROM temp_users
ON CONFLICT (Id) 
DO UPDATE SET 
    Username = EXCLUDED.Username,
    Email = EXCLUDED.Email,
    UpdatedDate = EXCLUDED.UpdatedDate;

DROP TABLE temp_users;
EOF

echo "[$(date)] Sync completed successfully" >> $LOG_FILE

# Clean up
rm /tmp/users_delta.csv
```

**Cron Job:**
```bash
# Edit crontab
crontab -e

# Add daily sync at 2 AM
0 2 * * * /path/to/sync-db.sh
```

---

### Monitoring & Alerting:

**1. CloudWatch Dashboard:**
```json
{
  "widgets": [
    {
      "type": "metric",
      "properties": {
        "metrics": [
          ["AWS/DMS", "CDCLatencyTarget", {"stat": "Average"}],
          [".", "CDCIncomingChanges", {"stat": "Sum"}],
          [".", "FullLoadThroughputRowsTarget", {"stat": "Sum"}]
        ],
        "period": 300,
        "stat": "Average",
        "region": "us-east-1",
        "title": "DMS Replication Metrics"
      }
    }
  ]
}
```

**2. SNS Notification:**
```bash
# Create SNS topic
aws sns create-topic --name db-sync-alerts

# Subscribe email
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:xxx:db-sync-alerts \
  --protocol email \
  --notification-endpoint admin@example.com
```

---

### Best Practices:

1. **Test in staging first** before production migration
2. **Monitor replication lag** - set alerts for delays > 5 minutes
3. **Validate data** after each sync (row counts, checksums)
4. **Handle schema changes** carefully (use SCT or manual scripts)
5. **Set up rollback plan** in case of failures
6. **Use VPN/Direct Connect** for secure on-premises to AWS connection
7. **Enable logging** for troubleshooting
8. **Schedule sync during low-traffic hours**
9. **Test failover scenarios** regularly

---

## Summary

| Approach | Best For | Real-Time | Complexity |
|----------|----------|-----------|------------|
| **AWS DMS with CDC** | Production, continuous sync | ✅ Yes | Medium |
| **Scheduled Lambda** | Small databases, daily sync | ❌ No | Low |
| **Apache NiFi** | Complex ETL, transformations | ⚠️ Near real-time | High |
| **Cron + Script** | Simple sync, budget-friendly | ❌ No | Low |
| **Manual Export/Import** | One-time migration | ❌ No | Low |
