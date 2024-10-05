Run docker:
```
docker compose up --detach --wait --wait-timeout 120
```
Set a password to StarRocks
```
SET PASSWORD = PASSWORD('secret');
```
Edit smt config file `smt/conf/config_prod.conf`
```
[db]
type = mysql
host = xxx.xx.xxx.xx
port = 3306
user = user1
password = xxxxxx

[other]
# Number of BEs in StarRocks
be_num = 3
# `decimal_v3` is supported since StarRocks-1.18.1.
use_decimal_v3 = true
# File to save the converted DDL SQL
output_dir = ./result

[table-rule.1]
# Pattern to match databases for setting properties
database = ^demo.*$
# Pattern to match tables for setting properties
table = ^.*$

############################################
### Flink sink configurations
### DO NOT set `connector`, `table-name`, `database-name`. They are auto-generated.
############################################
flink.starrocks.jdbc-url=jdbc:mysql://<fe_host>:<fe_query_port>
flink.starrocks.load-url= <fe_host>:<fe_http_port>
flink.starrocks.username=user2
flink.starrocks.password=xxxxxx
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator=\x01
flink.starrocks.sink.properties.row_delimiter=\x02
flink.starrocks.sink.buffer-flush.interval-ms=15000
```
Execute an interactive `sh` shell on the `flink-jobmanager` container.
```
docker exec -it flink-jobmanager sh
```
Inside the container, run the SMT
```
smt/starrocks-migrate-tool
```
Run the SMT to read the database & table schema in MySQL and generate SQL files in the `./result` directory based on the configuration file. The `starrocks-create.all.sql` file is used to create a database & table in StarRocks and the `flink-create.all.sql` file is used to submit a Flink job to the Flink cluster.

Connect to StarRocks and execute the `starrocks-create.all.sql` file to create a database and table in StarRocks.
```
mysql -h <fe_host> -P <fe_query_port> -u user2 -pxxxxxx < starrocks-create.all.sql
```

Run the Flink and submit a Flink job to continuously synchronize full and incremental data from MySQL to StarRocks.
```
./bin/sql-client.sh -f smt/result/flink-create.all.sql
```
If the following result is returned, the Flink job has been submitted for full and incremental synchronization.
```
[INFO] SQL update statement has been successfully submitted to the cluster:
Job ID: 9eb82c5cab67f92daff935331b61f672
```

Visit [http://localhost:8081](http://localhost:8081)