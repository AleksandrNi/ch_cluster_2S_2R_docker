## runnung up
```
docker-compose up -d
```

### view node
```
docker exec -it clickhouse-01 /bin/bash
cat /var/log/clickhouse-server/clickhouse-server.err.log

clickhouse-client
show databases;
use system;

select cluster,replica_num,host_name,host_address,port,is_local from system.clusters
SELECT * FROM system.clusters where cluster='{cluster}';
SELECT * FROM system.zookeeper WHERE path IN ('/', '/clickhouse');
SELECT * FROM system.macros
SELECT * FROM system.users
SELECT * FROM system.roles
SELECT * FROM system.grants
SELECT * FROM system.quotas
SELECT * FROM system.quotas_usage
SELECT * FROM system.role_grants

SELECT user()
SELECT now()

SHOW CURRENT ROLES
SHOW USERS
SHOW PROFILES
SHOW QUOTAS
SHOW GRANTS
SHOW TABLES
CREATE USER foo IDENTIFIED WITH sha256_password BY 'secretPassword123!'


SELECT * FROM system.query_log WHERE type = 'QueryFinish' ORDER BY query_start_time DESC LIMIT 10;

```

## create database
```
CREATE DATABASE db1 ON CLUSTER '{cluster}';
```
## create table
```
CREATE TABLE db1.table1 on cluster '{cluster}' (
    `id` UInt64,
    `column1` String
)
ENGINE = ReplicatedMergeTree
ORDER BY id
```
## create distributed table
```
CREATE TABLE db1.table1_distributed
ON CLUSTER '{cluster}'
(
    `id` UInt64,
    `column1` String
)
ENGINE = Distributed('{cluster}', 'db1', 'table1', rand());
```
## insert data in distributed table
```
INSERT INTO db1.table1_distributed
    (id, column1)
VALUES
    (1, '1'),
    (2, '2'),
    (3, '3'),
    (4, '4'),
    (5, '5'),
    (6, '6');
```
## select data from local table
```
SELECT * FROM db1.table1;
```
## select data from distributed table
```
SELECT * FROM db1.table1_distributed;
```
## extra information
* [System tables with examples](https://github.com/ClickHouse/ClickHouse/blob/master/docs/en/operations/system-tables/replicas.md)
* [Clickhouse recipies](https://github.com/ClickHouse/examples/tree/main/docker-compose-recipes/recipes/cluster_2S_2R)
* [Using clickhouse-keeper](https://kb.altinity.com/altinity-kb-setup-and-maintenance/altinity-kb-zookeeper/clickhouse-keeper/)

* [Keeper configuration](https://clickhouse.com/docs/guides/sre/keeper/clickhouse-keeper)
* [Clickhouse-keeper-service](https://kb.altinity.com/altinity-kb-setup-and-maintenance/altinity-kb-zookeeper/clickhouse-keeper-service/)
```
mkdir -p /etc/clickhouse-keeper/config.d
mkdir -p /var/log/clickhouse-keeper
mkdir -p /var/lib/clickhouse-keeper/coordination/log
mkdir -p /var/lib/clickhouse-keeper/coordination/snapshots
mkdir -p /var/lib/clickhouse-keeper/cores

chown -R clickhouse.clickhouse /etc/clickhouse-keeper /var/log/clickhouse-keeper /var/lib/clickhouse-keeper
```
* [Tutorial for set up clickhouse server with zookeeper](https://github.com/jneo8/clickhouse-setup/blob/master/docker-compose.yml)
* [Server configuration files. recommended structure](https://kb.altinity.com/altinity-kb-setup-and-maintenance/altinity-kb-server-config-files/)
* [How to add new replica node](https://chistadata.com/knowledge-base/how-to-add-new-replica-node-in-clickhouse-cluster-setup-with-clickhouse-keeper/)
* [Setting Up a Replicated Cluster withKeeper](https://chistadata.com/knowledge-base/setting-up-a-replicated-clickhouse-cluster-with-clickhouse-keeper/)
* [Useful queries](https://kb.altinity.com/altinity-kb-useful-queries/)

```
Check table metadata in zookeeper
Compare query_log for 2 intervals
Debug hanging thing
Handy queries for system.query_log
Ingestion metrics from system.part_log
Remove block numbers from zookeeper for removed partitions
Removing tasks in the replication queue related to empty partitions
Can detached parts in ClickHouse® be dropped?
Database Size - Table - Column size
Number of active parts in a partition
Parts consistency
```
* [Useful queries for troubleshooting](https://clickhouse.com/docs/knowledgebase/useful-queries-for-troubleshooting)

## useful system tables

* system.tables: This table contains information about all the tables in the database, including their names, engine types, and creation time.
* system.columns: This table contains information about the columns in each table, including their names, data types, and default values.
* system.databases: This table contains information about all the databases in the system, including their names, engine types, and creation time.
* system.functions: This table contains information about all the functions available in the system, including their names, parameters, and return types.
* system.parts: This table contains information about the individual shards of each table, including their location, size, and partitioning key.
* system.settings: This table contains information about all the settings available in the system, including their names, data types, and default values.

```
SELECT name, engine FROM system.tables
SELECT name, type FROM system.columns WHERE table_name = 'my_table'
SELECT partition_id, path, size FROM system.parts WHERE table_name = 'my_table' AND partition_id = 1
SELECT value FROM system.settings WHERE name = 'max_threads'
```