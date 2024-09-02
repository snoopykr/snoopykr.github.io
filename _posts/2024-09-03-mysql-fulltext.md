---
layout: single
title: "MySQL FULLTEXT"
categories : Python
tag: [MySQL, FULLTEXT]
---

MySQL FULLTEXT 이해

### 준비

```bash
snoopy_kr@iMac ~ % docker run -p 3306:3306 --name test-db -e MYSQL_ROOT_PASSWORD=dbpass -d mysql:5.7
ba386180edf664a68f64a05111beee45673714f724cf3b95ee0907a4388e79d4
```
```bash
snoopy_kr@iMac ~ % docker exec -it test-db mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.42 MySQL Community Server (GPL)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

### SQL init

```sql
CREATE DATABASE test;

USE test;

CREATE TABLE articles (
        id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
        title VARCHAR(200),
        body TEXT,
        FULLTEXT idx_ft_title_and_body (title,body)
    ) ENGINE=InnoDB;
    
INSERT INTO articles (title,body) VALUES
    ('MySQL Tutorial','DBMS stands for DataBase ...'),
    ('How To Use MySQL Well','After you went through a ...'),
    ('Optimizing MySQL','In this tutorial we will show ...'),
    ('1001 MySQL Tricks','1. Never run mysqld as root. 2. ...'),
    ('MySQL vs. YourSQL','In the following database comparison ...'),
    ('MySQL Security','When configured properly, MySQL ...');
```

### 기본 셋팅

```bash
mysql> CREATE DATABASE test;
Query OK, 1 row affected (0.00 sec)
```

```bash
mysql> USE test;
Database changed
```

```bash
mysql> CREATE TABLE articles (
    ->         id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
    ->         title VARCHAR(200),
    ->         body TEXT,
    ->         FULLTEXT idx_ft_title_and_body (title,body)
    ->     ) ENGINE=InnoDB;
Query OK, 0 rows affected (0.04 sec)
```

```bash
mysql> INSERT INTO articles (title,body) VALUES
    ->     ('MySQL Tutorial','DBMS stands for DataBase ...'),
    ->     ('How To Use MySQL Well','After you went through a ...'),
    ->     ('Optimizing MySQL','In this tutorial we will show ...'),
    ->     ('1001 MySQL Tricks','1. Never run mysqld as root. 2. ...'),
    ->     ('MySQL vs. YourSQL','In the following database comparison ...'),
    ->     ('MySQL Security','When configured properly, MySQL ...');
Query OK, 6 rows affected (0.01 sec)
Records: 6  Duplicates: 0  Warnings: 0
```

### [참조] 이미 생성된 테이블에 index추가

```sql
mysql> CREATE FULLTEXT INDEX idx_ft_title_and_body on articles(title, body);

mysql> ALTER TABLE articles ADD FULLTEXT INDEX idx_ft_title_and_body(title, body);
```

### FullText 검색

```bash
mysql> SELECT * FROM articles WHERE MATCH (title,body) AGAINST ('database' IN NATURAL LANGUAGE MODE);
+----+-------------------+------------------------------------------+
| id | title             | body                                     |
+----+-------------------+------------------------------------------+
|  1 | MySQL Tutorial    | DBMS stands for DataBase ...             |
|  5 | MySQL vs. YourSQL | In the following database comparison ... |
+----+-------------------+------------------------------------------+
2 rows in set (0.00 sec)
```

> 참고로 FullText 검색은 대/소문자를 구분하지 않습니다.

### 'data'는 검색이 안됨

```bash
mysql> SELECT * FROM articles WHERE MATCH (title, body) AGAINST ('data' IN NATURAL LANGUAGE MODE);
Empty set (0.00 sec)
```

### 저장된 단어 확인하기

```bash
mysql> SET GLOBAL innodb_ft_aux_table = 'test/articles';
Query OK, 0 rows affected (0.00 sec)
```

```bash
mysql> SET GLOBAL innodb_optimize_fulltext_only=ON;
Query OK, 0 rows affected (0.00 sec)
```

```bash
mysql> OPTIMIZE TABLE articles;
+---------------+----------+----------+----------+
| Table         | Op       | Msg_type | Msg_text |
+---------------+----------+----------+----------+
| test.articles | optimize | status   | OK       |
+---------------+----------+----------+----------+
1 row in set (0.00 sec)
```

```bash
mysql> SELECT WORD, DOC_COUNT, DOC_ID, POSITION FROM INFORMATION_SCHEMA.INNODB_FT_INDEX_TABLE;
+------------+-----------+--------+----------+
| WORD       | DOC_COUNT | DOC_ID | POSITION |
+------------+-----------+--------+----------+
| 1001       |         1 |      5 |        0 |
| after      |         1 |      3 |       22 |
| comparison |         1 |      6 |       44 |
| configured |         1 |      7 |       20 |
| database   |         2 |      2 |       31 |
| database   |         2 |      6 |       35 |
| dbms       |         1 |      2 |       15 |
| following  |         1 |      6 |       25 |
| mysql      |         6 |      2 |        0 |
| mysql      |         6 |      3 |       11 |
| mysql      |         6 |      4 |       11 |
| mysql      |         6 |      5 |        5 |
| mysql      |         6 |      6 |        0 |
| mysql      |         6 |      7 |        0 |
| mysql      |         6 |      7 |       41 |
| mysqld     |         1 |      5 |       31 |
| never      |         1 |      5 |       21 |
| optimizing |         1 |      4 |        0 |
| properly   |         1 |      7 |       31 |
| root       |         1 |      5 |       41 |
| run        |         1 |      5 |       27 |
| security   |         1 |      7 |        6 |
| show       |         1 |      4 |       42 |
| stands     |         1 |      2 |       20 |
| through    |         1 |      3 |       37 |
| tricks     |         1 |      5 |       11 |
| tutorial   |         2 |      2 |        6 |
| tutorial   |         2 |      4 |       25 |
| use        |         1 |      3 |        7 |
| well       |         1 |      3 |       17 |
| went       |         1 |      3 |       32 |
| you        |         1 |      3 |       28 |
| yoursql    |         1 |      6 |       10 |
+------------+-----------+--------+----------+
33 rows in set (0.00 sec)
```

### Rebuild Index

```bash
mysql> DROP INDEX idx_ft_title_and_body ON articles;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

```bash
mysql> ALTER TABLE articles ADD FULLTEXT INDEX idx_ft_title_and_body(title, body) WITH PARSER NGRAM;
Query OK, 0 rows affected (0.09 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

### 'data'는 검색이 아직 안됨

```bash
mysql> SELECT * FROM articles WHERE MATCH (title, body) AGAINST ('data' IN NATURAL LANGUAGE MODE);
Empty set (0.00 sec)
```

### 저장된 단어 다시 확인하기

```bash
mysql> SELECT WORD, DOC_COUNT, DOC_ID, POSITION FROM INFORMATION_SCHEMA.INNODB_FT_INDEX_TABLE;
+------+-----------+--------+----------+
| WORD | DOC_COUNT | DOC_ID | POSITION |
+------+-----------+--------+----------+
| 00   |         1 |      5 |        1 |
| 01   |         1 |      5 |        2 |
| 10   |         1 |      5 |        0 |
| bm   |         1 |      2 |       16 |
| ck   |         1 |      5 |       14 |
| co   |         2 |      6 |       44 |
| co   |         2 |      7 |       20 |
| cu   |         1 |      7 |        8 |
| db   |         1 |      2 |       15 |
| ds   |         1 |      2 |       24 |
| ec   |         1 |      7 |        7 |
| ed   |         1 |      7 |       28 |
| el   |         1 |      3 |       18 |
| er   |         3 |      3 |       25 |
| er   |         3 |      5 |       24 |
| er   |         3 |      7 |       35 |
| ev   |         1 |      5 |       22 |
| fo   |         2 |      2 |       27 |
| fo   |         2 |      6 |       25 |
| ft   |         1 |      3 |       23 |
| gh   |         1 |      3 |       42 |
| gu   |         1 |      7 |       25 |
| he   |         2 |      6 |       22 |
| he   |         2 |      7 |       16 |
| ho   |         2 |      3 |        0 |
| ho   |         2 |      4 |       43 |
| hr   |         1 |      3 |       38 |
| ks   |         1 |      5 |       15 |
| ld   |         1 |      5 |       35 |
| ll   |         3 |      3 |       19 |
| ll   |         3 |      4 |       39 |
| ll   |         3 |      6 |       27 |
| lo   |         1 |      6 |       28 |
| ly   |         1 |      7 |       37 |
| mp   |         1 |      6 |       46 |
| ms   |         1 |      2 |       17 |
| my   |         6 |      2 |        0 |
| my   |         6 |      3 |       11 |
| my   |         6 |      4 |       11 |
| my   |         6 |      5 |        5 |
| my   |         6 |      5 |       26 |
| my   |         6 |      6 |        0 |
| my   |         6 |      7 |        0 |
| my   |         6 |      7 |       41 |
| nd   |         1 |      2 |       23 |
| ne   |         1 |      5 |       21 |
| nf   |         1 |      7 |       22 |
| ng   |         2 |      4 |        8 |
| ng   |         2 |      6 |       32 |
| nt   |         1 |      3 |       34 |
| ol   |         1 |      6 |       26 |
| om   |         1 |      6 |       45 |
| oo   |         1 |      5 |       42 |
| op   |         2 |      4 |        0 |
| op   |         2 |      7 |       33 |
| ot   |         1 |      5 |       43 |
| ou   |         2 |      3 |       29 |
| ou   |         2 |      3 |       11 |
| ou   |         2 |      6 |       11 |
| ow   |         3 |      3 |        1 |
| ow   |         3 |      4 |       44 |
| ow   |         3 |      6 |       29 |
| pe   |         1 |      7 |       34 |
| pr   |         1 |      7 |       31 |
| pt   |         1 |      4 |        1 |
| ql   |         6 |      2 |        3 |
| ql   |         6 |      3 |       14 |
| ql   |         6 |      4 |       14 |
| ql   |         6 |      5 |        8 |
| ql   |         6 |      5 |       26 |
| ql   |         6 |      6 |        3 |
| ql   |         6 |      6 |       12 |
| ql   |         6 |      7 |        3 |
| ql   |         6 |      7 |       41 |
| re   |         1 |      7 |       27 |
| rl   |         1 |      7 |       36 |
| ro   |         3 |      3 |       39 |
| ro   |         3 |      5 |       41 |
| ro   |         3 |      7 |       32 |
| rs   |         1 |      6 |       13 |
| ru   |         1 |      5 |       27 |
| se   |         4 |      2 |       37 |
| se   |         4 |      3 |        8 |
| se   |         4 |      6 |       41 |
| se   |         4 |      7 |        6 |
| sh   |         1 |      4 |       42 |
| so   |         1 |      6 |       51 |
| sq   |         6 |      2 |        2 |
| sq   |         6 |      3 |       13 |
| sq   |         6 |      4 |       13 |
| sq   |         6 |      5 |        7 |
| sq   |         6 |      5 |       26 |
| sq   |         6 |      6 |        2 |
| sq   |         6 |      6 |       12 |
| sq   |         6 |      7 |        2 |
| sq   |         6 |      7 |       41 |
| st   |         1 |      2 |       20 |
| te   |         1 |      3 |       24 |
| th   |         3 |      3 |       37 |
| th   |         3 |      4 |       20 |
| th   |         3 |      6 |       21 |
| tr   |         1 |      5 |       11 |
| tu   |         2 |      2 |        6 |
| tu   |         2 |      4 |       25 |
| ty   |         1 |      7 |       12 |
| ug   |         1 |      3 |       41 |
| un   |         1 |      5 |       28 |
| ur   |         2 |      6 |       12 |
| ur   |         2 |      7 |        9 |
| ur   |         2 |      7 |       17 |
| us   |         1 |      3 |        7 |
| ut   |         2 |      2 |        7 |
| ut   |         2 |      4 |       26 |
| ve   |         1 |      5 |       23 |
| vs   |         1 |      6 |        6 |
| we   |         2 |      3 |       17 |
| we   |         2 |      3 |       15 |
| we   |         2 |      4 |       34 |
| wh   |         1 |      7 |       15 |
| yo   |         2 |      3 |       28 |
| yo   |         2 |      6 |       10 |
| ys   |         6 |      2 |        1 |
| ys   |         6 |      3 |       12 |
| ys   |         6 |      4 |       12 |
| ys   |         6 |      5 |        6 |
| ys   |         6 |      5 |       26 |
| ys   |         6 |      6 |        1 |
| ys   |         6 |      7 |        1 |
| ys   |         6 |      7 |       41 |
+------+-----------+--------+----------+
129 rows in set (0.00 sec)
```

### Stopwords 확인

```bash
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_DEFAULT_STOPWORD;
+-------+
| value |
+-------+
| a     |
| about |
| an    |
| are   |
| as    |
| at    |
| be    |
| by    |
| com   |
| de    |
| en    |
| for   |
| from  |
| how   |
| i     |
| in    |
| is    |
| it    |
| la    |
| of    |
| on    |
| or    |
| that  |
| the   |
| this  |
| to    |
| was   |
| what  |
| when  |
| where |
| who   |
| will  |
| with  |
| und   |
| the   |
| www   |
+-------+
36 rows in set (0.00 sec)
```

### a’ 또는 ‘i’로 시작하거나 끝나는 쪼개진 단어들은 모두 저장되지 않음

```bash
mysql> SELECT WORD, DOC_COUNT, DOC_ID, POSITION FROM INFORMATION_SCHEMA.INNODB_FT_INDEX_TABLE WHERE WORD LIKE '%a%' OR WORD LIKE '%i%';
Empty set (0.00 sec)
```

### Stopwords를 비활성화 하는 방법

```bash
mysql> CREATE TABLE stopwords(value VARCHAR(30)) ENGINE = INNODB;
Query OK, 0 rows affected (0.02 sec)
```

```bash
mysql> SET GLOBAL innodb_ft_server_stopword_table = 'test/stopwords';
Query OK, 0 rows affected (0.00 sec)
```

```bash
mysql> DROP INDEX idx_ft_title_and_body ON articles;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

```bash
mysql> ALTER TABLE articles ADD FULLTEXT INDEX idx_ft_title_and_body(title, body) WITH PARSER NGRAM;
Query OK, 0 rows affected (0.06 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

### mysql.cnf (리셋되지 않기 위해 필요)

```
[mysqld]
innodb_ft_server_stopword_table='test/stopwords'
```

### 'data'의 검색 결과 확인

```bash
mysql> SELECT * FROM articles WHERE MATCH (title, body) AGAINST ('data' IN NATURAL LANGUAGE MODE);
+----+-------------------+------------------------------------------+
| id | title             | body                                     |
+----+-------------------+------------------------------------------+
|  1 | MySQL Tutorial    | DBMS stands for DataBase ...             |
|  5 | MySQL vs. YourSQL | In the following database comparison ... |
+----+-------------------+------------------------------------------+
2 rows in set (0.00 sec)
```

### 'database'의 검색 결과 확인

```bash
mysql> SELECT * FROM articles WHERE MATCH (title,body) AGAINST ('database' IN NATURAL LANGUAGE MODE);
+----+-----------------------+------------------------------------------+
| id | title                 | body                                     |
+----+-----------------------+------------------------------------------+
|  1 | MySQL Tutorial        | DBMS stands for DataBase ...             |
|  5 | MySQL vs. YourSQL     | In the following database comparison ... |
|  4 | 1001 MySQL Tricks     | 1. Never run mysqld as root. 2. ...      |
|  2 | How To Use MySQL Well | After you went through a ...             |
|  6 | MySQL Security        | When configured properly, MySQL ...      |
+----+-----------------------+------------------------------------------+
5 rows in set (0.00 sec)
```

### 'database'의 검색 결과 해결 방법

```bash
mysql> SELECT * FROM articles WHERE MATCH (title, body) AGAINST ('database' IN BOOLEAN MODE);
+----+-------------------+------------------------------------------+
| id | title             | body                                     |
+----+-------------------+------------------------------------------+
|  1 | MySQL Tutorial    | DBMS stands for DataBase ...             |
|  5 | MySQL vs. YourSQL | In the following database comparison ... |
+----+-------------------+------------------------------------------+
2 rows in set (0.00 sec)
```
