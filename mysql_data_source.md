# Ref https://sbcode.net/grafana/create-mysql-data-source/
# install Mysql in ubuntu machine
### update server
```
sudo apt update
```
### Install mysql
``` 
sudo apt install mysql-server -y
sudo service mysql status
mysql -V
```

# Collector
## All data source have some kind of collector

## Dashboard use in grafana is 
https://grafana.com/grafana/dashboards?dataSource=mysql
## Collector script 
https://grafana.com/grafana/dashboards?dataSource=mysql
## SSH onto your new MySQL server and download the script using wget
wget https://raw.githubusercontent.com/meob/my2Collector/master/my2_80.sql

### edit my2_80.sql
### uncomment last 3 lines
```
-- Use a specific user (suggested)
create user my2@'%' identified by 'password';
grant all on my2.* to my2@'%';
grant select on performance_schema.* to my2@'%';
```
### change the password
### Save the changes and then run the SQL script,
```
mysql < my2_80.sql
```

### Now open MySQL and do some checks.

```
mysql

> show databases;
> show variables where variable_name = 'event_scheduler';
> select host, user from mysql.user;
> use my2;
> show tables;
> select * from current;
> select * from status;
> quit
```
### Note: If when running the above lines, it shows that the event_scheduler is not enabled, then we will need to enable it so that the collector runs in the background. You can do this by editing the my.cnf file

#### Add lines to the end of the file my.cnf
```
sudo nano /etc/mysql/my.cnf
```

```
[mysqld]
event_scheduler = on
```
### Save and restart MySQL.
```
sudo service mysql restart
```
### We need to know the IP address of our Grafana server that will initiate the connection to the MySQL server.
```
mysql
```
```
> CREATE USER 'grafana'@'172.31.39.192' IDENTIFIED BY 'password';
> GRANT SELECT ON my2.* TO 'grafana'@'172.31.39.192';
> FLUSH PRIVILEGES;
> quit
```


#### To allow remote connections on the MySQL server.

#### Open the MySQL configuration file
```
sudo nano /etc/mysql/my.cnf
```
#### Change the bind address to 0.0.0.0 or add this text below to the end of the file if it doesn't already exist.
```
[mysqld]
bind-address    = 0.0.0.0
```
### Save and restart MySQL.
```
sudo service mysql restart
```

#### Run Query on Explore
Explore --> Select Data Source --> Select Code option
```
SELECT
  variable_value+0 as value,
  timest as time_sec
FROM my2.status
WHERE variable_name='THREADS_CONNECTED'
ORDER BY timest ASC;
```

Dashboard --> manage --> import
dashboard id 7991


Query Data

```
SELECT
  UNIX_TIMESTAMP(TIMEST) as time_sec,
  VARIABLE_VALUE as value,
  VARIABLE_NAME  as metric
FROM my2.status
WHERE $__timeFilter(TIMEST)
ORDER BY TIMEST ASC
```
***************************************************************
Graphing Non Time Series SQL Data in Grafana

mysql

> CREATE DATABASE exampledb;

> show databases;

Lets now create a simple table with some data that we can query

> CREATE TABLE `exampledb`.`simple_table` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`username` varchar(45) DEFAULT NULL,
`total` decimal(10,0) DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=latin1;

Insert Statement

> INSERT INTO `exampledb`.`simple_table`
(`username`,
`total`)
VALUES
('Cat',56),
('Dog',35),
('Lizard',41),
('Crocodile',22),
('Koala',26),
('Cassowary',29),
('Peacock',19),
('Emu',10),
('Kangaroo',13);

Check data exists
> SELECT * FROM `exampledb`.`simple_table`;

Now allow our grafana database user select permissions on the table.
```
> GRANT SELECT ON exampledb.simple_table TO 'grafana'@'172.31.39.192';

> FLUSH PRIVILEGES;

> quit;
```

Explore the Data in Grafana