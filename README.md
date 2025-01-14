# 2PC_Demo

To explore the phases of a distributed transaction in TiDB on the CLI and observe how data changes in the Lock, Data, and Write column families, you can use a combination of SQL commands and tools like pd-ctl or tikv-ctl for lower-level insights. Here’s a step-by-step guide:

### 1. Set Up the Scenario

Create a table, insert some data, and perform transactions to see how data flows through the column families.

## Create a Table
```
CREATE TABLE user_data (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    age INT,
    INDEX (age)
);
```

## Insert Data

Start by inserting some data.
```
INSERT INTO user_data (id, name, age) VALUES (1, 'Alice', 30), (2, 'Bob', 25);
```

### 2. Explore Data Changes Using SQL

TiDB uses MVCC (Multi-Version Concurrency Control) to store multiple versions of data in the Data and Write column families. You can observe this using specific SQL queries.

## Enable TiDB MVCC Debug Mode

Run this to see MVCC details:
```
ADMIN SHOW DDL JOBS;
```
## Check Data Versions

Use ADMIN INSPECT commands to view MVCC details.
```
ADMIN SHOW DDL JOBS FOR TABLE user_data;
```
## Perform a Transaction

Observe changes during a transaction.

Start a transaction:
```
BEGIN;
UPDATE user_data SET age = 35 WHERE id = 1;
```
Query the data during the transaction (before commit):
```
SELECT * FROM user_data WHERE id = 1;
```
Commit the transaction:
```
COMMIT;
```
Now check how the data looks after the transaction:
```
SELECT * FROM user_data WHERE id = 1;
```

### 3. Use TiKV Debug Tools

For deeper insights into column families (Lock, Data, Write), use tikv-ctl. This requires access to the TiKV nodes.

## Install tikv-ctl

Download and install tikv-ctl on a machine with access to the TiKV cluster.

## Inspect the Key-Value Data

To inspect the raw key-value data stored in the column families:
# Check Lock CF:
```
tikv-ctl --db /path/to/tikv/data db scan --cf lock
```
This shows locks for keys currently in a transaction.

# Check Data CF:
```
tikv-ctl --db /path/to/tikv/data db scan --cf default
```
This shows the actual key-value data stored with startTS.

# Check Write CF:
```
tikv-ctl --db /path/to/tikv/data db scan --cf write
```
This shows commit entries, linking startTS to commitTS.

## Observe Transactional Changes

Perform a transaction as before and then use tikv-ctl to check the column families. For example:
Start a transaction and make a change:
```
BEGIN;
UPDATE user_data SET name = 'Alice Updated' WHERE id = 1;
```
Before committing, inspect the Lock CF to see the lock.

Commit the transaction:
```
COMMIT;
```

Check the Write CF to see the commit record and Data CF to see the updated data.

## Query TiDB Internal Tables

TiDB provides system tables to explore metadata and transactions.

# View Transactions

Check currently running or completed transactions:
```
SELECT * FROM INFORMATION_SCHEMA.TIDB_TRX;
```
# View Locks

Inspect ongoing locks:
```
SELECT * FROM INFORMATION_SCHEMA.TIDB_LOCKS;
```
# Table Region Mapping

View which Regions are handling a specific table:
```
SELECT * FROM INFORMATION_SCHEMA.TIKV_REGION_STATUS WHERE TABLE_NAME = 'user_data';
```
5. Monitor Using Logs

Enable detailed logs on TiKV or TiDB to trace transaction phases. Use configuration settings to enable verbose logging of transactions.
 and how TiDB manages them internally.
