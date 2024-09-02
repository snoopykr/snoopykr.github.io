---
layout: single
title: "MySQL FULLTEXT"
categories : Python
tag: [MySQL, FULLTEXT]
---

MySQL FULLTEXT 이해

### 설치

```bash
ubuntu@ip-172-31-37-56:~$ sudo apt update
```

```bash
ubuntu@ip-172-31-37-56:~$ sudo apt install mysql-server
```

### 외부 접속 가능 설정

```bash
ubuntu@ip-172-31-37-56:~$ sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
bind-address = 0.0.0.0
```

### root 추가 준비

```bash
ubuntu@ip-172-31-37-56:~$ sudo mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.34-0ubuntu0.22.04.1 (Ubuntu)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

```bash
mysql> use mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
```

```bash
mysql>  select host, user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| localhost | debian-sys-maint |
| localhost | mysql.infoschema |
| localhost | mysql.session    |
| localhost | mysql.sys        |
| localhost | root             |
+-----------+------------------+
5 rows in set (0.00 sec)
```

### root 추가

```bash
mysql> create user 'root'@'%' identified by '123qwe!@#';
Query OK, 0 rows affected (0.02 sec)
```

```bash
mysql>  select host, user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| %         | root             |
| localhost | debian-sys-maint |
| localhost | mysql.infoschema |
| localhost | mysql.session    |
| localhost | mysql.sys        |
| localhost | root             |
+-----------+------------------+
6 rows in set (0.00 sec)
```

### root 권한 설정

```bash
mysql> grant all privileges on *.* to 'root'@'%';
Query OK, 0 rows affected (0.02 sec)
```

```bash
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

### 종료

```bash
mysql> quit
Bye
```

> MySQL 8 은 기본 character set 과 collation 이 utf8mb4과 utf8mb4_0900_ai_ci이므로 별도 설정 불필요