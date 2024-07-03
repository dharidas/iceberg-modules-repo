# Metadata Tables

## List of Tables

| Metadata Table         | Purpose                                                                          | Type of Data                                                                                                              |
|------------------------|----------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| `history`              | Shows the history of the table                                                   | Timestamp the snapshot was made, snapshot ID, parent ID, whether the snapshot is part of the current history              |
| `metadata_log_entries` | Shows metadata log entries for the table                                         | Timestamp, file path, latest snapshot ID, latest schema ID, latest sequence number                                        |
| `snapshots`            | Shows the valid snapshots for the table                                          | Timestamp the snapshot was committed, snapshot ID, parent ID, operation, manifest list, summary                            |
| `entries`              | Shows the current manifest entries for both data and delete files                | Status, snapshot ID, sequence number, file sequence number, data file, readable metrics                                   |
| `files`                | Shows the current files of the table                                             | Content, file path, file format, spec ID, record count, file size, column sizes, value counts, null value counts, etc.    |
| `manifests`            | Shows the current file manifests of the table                                    | Path, length, partition spec ID, added snapshot ID, added data files count, existing data files count, deleted files count |
| `partitions`           | Shows the current partitions of the table                                        | Partition, spec ID, record count, file count, total data file size, delete record count, delete file count, etc.          |
| `position_deletes`     | Shows all positional delete files from the current snapshot of the table         | File path, position, row, spec ID, delete file path                                                                       |
| `all_data_files`       | Shows all data files and their metadata across all snapshots                     | Content, file path, file format, partition, record count, file size, column sizes, value counts, null value counts, etc.  |
| `all_delete_files`     | Shows all delete files and their metadata across all snapshots                   | Content, file path, file format, spec ID, record count, file size, column sizes, value counts, null value counts, etc.    |
| `all_entries`          | Shows all manifest entries from all snapshots for both data and delete files     | Status, snapshot ID, sequence number, file sequence number, data file, readable metrics                                   |
| `all_manifests`        | Shows all manifest files of the table                                            | Path, length, partition spec ID, added snapshot ID, added data files count, existing data files count, deleted files count |
| `refs`                 | Shows the known snapshot references for the table                                | Reference name, type (branch or tag), snapshot ID, max reference age in ms, min snapshots to keep, max snapshot age in ms  |

## Querying the Tables in Spark

```sql
-- Query the history of the table
SELECT * FROM nessie.hr.employees.history;

-- Query the metadata log entries of the table
SELECT * FROM nessie.hr.employees.metadata_log_entries;

-- Query the valid snapshots for the table
SELECT * FROM nessie.hr.employees.snapshots;

-- Query the manifest entries for both data and delete files of the table
SELECT * FROM nessie.hr.employees.entries;

-- Query the current files of the table
SELECT * FROM nessie.hr.employees.files;

-- Query the current file manifests of the table
SELECT * FROM nessie.hr.employees.manifests;

-- Query the current partitions of the table
SELECT * FROM nessie.hr.employees.partitions;

-- Query all positional delete files from the current snapshot of the table
SELECT * FROM nessie.hr.employees.position_deletes;

-- Query all data files and their metadata across all snapshots of the table
SELECT * FROM nessie.hr.employees.all_data_files;

-- Query all delete files and their metadata across all snapshots of the table
SELECT * FROM nessie.hr.employees.all_delete_files;

-- Query all manifest entries from all snapshots for both data and delete files of the table
SELECT * FROM nessie.hr.employees.all_entries;

-- Query all manifest files of the table
SELECT * FROM nessie.hr.employees.all_manifests;

-- Query the known snapshot references for the table
SELECT * FROM nessie.hr.employees.refs;
```

## Querying in Dremio

```sql
-- Querying the data files metadata of an Iceberg table
SELECT * 
FROM TABLE(table_files('nessie.hr.employees'));

-- Querying the history of an Iceberg table
SELECT * 
FROM TABLE(table_history('nessie.hr.employees'));

-- Querying the manifests metadata of an Iceberg table
SELECT * 
FROM TABLE(table_manifests('nessie.hr.employees'));

-- Querying the partitions statistics of an Iceberg table
SELECT * 
FROM TABLE(table_partitions('nessie.hr.employees'));

-- Querying the snapshots metadata of an Iceberg table
SELECT * 
FROM TABLE(table_snapshot('nessie.hr.employees'));
```

