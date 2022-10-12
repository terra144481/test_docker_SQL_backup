# test_svoboda
task from Radio Svoboda


1. Using Docker Compose (compose file) create service running basic MySQL instance with ersistent storage.  
```
version: '3.9'

services:
    mysql:
        image: mysql
        container_name: db-master
        volumes:
            - ./build_env/mysql/master.cnf:/etc/mysql/my.cnf
            - ./build_env/mysql/master.sql:/docker-entrypoint-initdb.d/start.sql
        environment:
            MYSQL_ROOT_PASSWORD: root
        ports:
            - 42333:3306

    mysqlread1:
        image: mysql
        container_name: db-slave1
        volumes:
            - ./build_env/mysql/slave1.cnf:/etc/mysql/my.cnf
            - ./build_env/mysql/slave.sql:/docker-entrypoint-initdb.d/start.sql
        depends_on:
            - mysql
        environment:
            MYSQL_ROOT_PASSWORD: root
        ports:
            - 43333:3306
 ```
 
 2. Make sure mysql is HA (at least two instances of mysql).  
```
ubuntu@ip-172-31-28-225:~/db-replication$ sudo docker compose ps
NAME                COMMAND                  SERVICE             STATUS              PORTS
db-master           "docker-entrypoint.s…"   mysql               running             33060/tcp, 0.0.0.0:42333->3306/tcp, :::42333->3306/tcp
db-slave1           "docker-entrypoint.s…"   mysqlread1          running             33060/tcp, 0.0.0.0:43333->3306/tcp, :::43333->3306/tcp
ubuntu@ip-172-31-28-225:~/db-replication$ sudo docker exec -it db-master mysql -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 22
Server version: 8.0.30 MySQL Community Server - GPL
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mydb_test          |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> exit
Bye

ubuntu@ip-172-31-28-225:~/db-replication$ sudo docker exec -it db-slave1 mysql -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 16
Server version: 8.0.30 MySQL Community Server - GPL
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mydb_test          |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```
3. Connect to this MySQL instance from whatever client and create at least two tables with sample data.  
![img](https://github.com/terra144481/test_svoboda/blob/19878694190ae416228cd814ec66342046890bac/Images/shema.png)  
Create `mydb_test` using Mysql workbanch

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mydb_test          |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use mydb_test;
```
Create 3 tables:
```
CREATE TABLE IF NOT EXISTS `mydb_test`.`usr` (
  `idusr` INT NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(45) NOT NULL,
  `surname` VARCHAR(45) NOT NULL,
  PRIMARY KEY (`idusr`))
ENGINE = InnoDB;
```

```
CREATE TABLE IF NOT EXISTS `mydb_test`.`product` (
  `idp` INT NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(45) NOT NULL,
  `description` VARCHAR(255) NOT NULL,
  PRIMARY KEY (`idp`))
ENGINE = InnoDB;
```
```
CREATE TABLE IF NOT EXISTS `mydb_test`.`invoces` (
  `idinvoces` INT NOT NULL AUTO_INCREMENT,
  `idusr` INT NOT NULL,
  `idp` INT NOT NULL,
  `price` DECIMAL(10,2) NOT NULL,
  PRIMARY KEY (`idinvoces`),
  INDEX `usr_idx` (`idusr` ASC) VISIBLE,
  INDEX `prod_idx` (`idp` ASC) VISIBLE,
  CONSTRAINT `usr`
    FOREIGN KEY (`idusr`)
    REFERENCES `mydb_test`.`usr` (`idusr`)
    ON DELETE CASCADE
    ON UPDATE CASCADE,
  CONSTRAINT `prod`
    FOREIGN KEY (`idp`)
    REFERENCES `mydb_test`.`product` (`idp`)
    ON DELETE CASCADE
    ON UPDATE CASCADE)
ENGINE = InnoDB;
```

```
mysql> show tables;
+---------------------+
| Tables_in_mydb_test |
+---------------------+
| invoces             |
| product             |
| usr                 |
+---------------------+
3 rows in set (0.00 sec)
```
Add data to table
```
mysql> select * from usr;
+-------+-------+---------+
| idusr | name  | surname |
+-------+-------+---------+
|     1 | Ivan  | Ivanov  |
|     2 | Petr  | Petrov  |
|     3 | Simon | Simonov |
+-------+-------+---------+
3 rows in set (0.00 sec)

mysql> select * from product;
+-----+--------+------------------+
| idp | name   | description      |
+-----+--------+------------------+
|   1 | lemon  | very sour fruit  |
|   2 | ananas | very sweet fruit |
|   3 | banan  | very nice fruit  |
+-----+--------+------------------+
3 rows in set (0.00 sec)

mysql> INSERT INTO invoces (idusr, idp, price) VALUES (3, 3, 14.78);
Query OK, 1 row affected (0.00 sec)

mysql> select * from invoces;
+-----------+-------+-----+-------+
| idinvoces | idusr | idp | price |
+-----------+-------+-----+-------+
|         1 |     2 |   1 | 10.25 |
|         2 |     1 |   2 |  8.75 |
|         3 |     3 |   3 | 14.78 |
+-----------+-------+-----+-------+
3 rows in set (0.00 sec)
```
4. Run SQL query on tables created at point 2. using INNER and LEFT JOIN.

```
mysql> SELECT invoces.idinvoces, usr.name FROM invoces INNER JOIN usr ON invoces.idusr=usr.idusr;
+-----------+-------+
| idinvoces | name  |
+-----------+-------+
|         2 | Ivan  |
|         1 | Petr  |
|         4 | Petr  |
|         3 | Simon |
+-----------+-------+
4 rows in set (0.00 sec)
```

```
mysql> SELECT usr.name, invoces.idinvoces FROM usr LEFT JOIN invoces ON usr.idusr = invoces.idusr ORDER BY usr.name;
+-------+-----------+
| name  | idinvoces |
+-------+-----------+
| Ivan  |         2 |
| Petr  |         1 |
| Petr  |         4 |
| Simon |         3 |
+-------+-----------+
4 rows in set (0.00 sec)
```


TASK 2
On Windows 10 system (not windows server) machine provide mechanism how to create keytab file for 
dummy@EXAMPLE.ORG domain user. Describe how can be this file used for the Linux container 
authentication against the domain resources using Kerberos. If any additional code or configuration is 
needed on the top of the docker compose functionality, use the bash / PowerShell scripting.  

1. Create a keytab file using ktab.sh at command line windows 10.  
```
#!/bin/sh

#create keytabfile for windows10
ktab -a dummy@EXAMPLE.ORG mypassword -n 0 -k c:\kerberos\dummy.keytab
```

2. Create /etc/krb5.conf file at container with MYSQL Linux.  
```
[libdefaults]
default_realm = EXAMPLE.ORG
```
3. Connect to container with MYSQL Linux using keytab `file MSSQLSvc/sqlcontainer.EXAMPLE.ORG:42333`.  




