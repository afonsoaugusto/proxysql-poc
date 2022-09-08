# proxysql-poc

```sh
docker network create banco

docker run \
    --network banco \
    --name mysql1 \
    -e MYSQL_ROOT_PASSWORD=my-secret-pw \
    -p 3306:3306 \
    mysql:5.7

docker run \
    --network banco \
    --name mysql2 \
    -e MYSQL_ROOT_PASSWORD=my-secret-pw \
    -p 3307:3306 \
    mysql:5.7


docker run \
    -p 16032:6032 \
    -p 16033:6033 \
    -p 16070:6070 \
    --name proxysql \
    --network banco \
    --rm -it \
    -v ${PWD}/proxysql.cnf:/etc/proxysql.cnf \
    proxysql/proxysql:2.4.3


docker exec -it proxysql bash -c 'mysql -u admin -padmin -h 127.0.0.1 -P6032 --prompt="Admin> "'    

```

```sql

------- ProxySQL

CREATE USER 'monitor'@'%' IDENTIFIED BY 'monitor';
GRANT USAGE ON *.* TO 'monitor'@'%';
FLUSH PRIVILEGES;

CREATE USER 'proxysql'@'%' IDENTIFIED BY 'ProxySQLPa55';
GRANT USAGE ON *.* TO 'proxysql'@'%';

show databases;

CREATE DATABASE T;
USE T;

CREATE TABLE P (ID INT);
INSERT INTO T.P VALUES (3);
SELECT * FROM T.P;

GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, INDEX, ALTER, CREATE ROUTINE, ALTER ROUTINE, EVENT, TRIGGER ON T.* TO 'proxysql'@'%';


FLUSH TABLES WITH READ LOCK;
SET GLOBAL read_only = 1;

SET GLOBAL read_only = 0;
UNLOCK TABLES;

show global variables like '%read_only%';

select rule_id,active,match_digest,destination_hostgroup,apply from mysql_query_rules;
select * from mysql_replication_hostgroups mrh ;
select rule_id, username, match_digest, destination_hostgroup  from mysql_query_rules mqr ;


SELECT @@global.read_only read_only;
```

## Config proxysql in runtime

```sql
UPDATE mysql_users SET default_hostgroup=1;
LOAD MYSQL USERS TO RUNTIME;
SAVE MYSQL USERS TO DISK;
INSERT INTO mysql_query_rules (rule_id,active,match_digest,destination_hostgroup,apply)
VALUES
(1,1,'^SELECT.*FOR UPDATE',1,1),
(2,1,'^SELECT',2,1);
LOAD MYSQL QUERY RULES TO RUNTIME;
SAVE MYSQL QUERY RULES TO DISK;
```

SELECT * FROM mysql_aws_aurora_hostgroups;

INSERT INTO mysql_servers (hostgroup_id, hostname) VALUES (1, 'mysql1'), (2,'mysql2');
LOAD MYSQL SERVERS TO RUNTIME;
SAVE MYSQL SERVERS TO DISK;
SELECT *FROM mysql_servers;
SELECT* FROM runtime_mysql_servers;
