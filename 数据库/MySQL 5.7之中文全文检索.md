# MySQL 5.7之中文全文检索

从MySQL 5.7开始，MySQL内置了ngram全文检索插件，用来支持中文分词，并且对MyISAM和InnoDB引擎有效。

在使用中文检索分词插件ngram之前，先得在MySQL配置文件里面设置他的分词大小，比如，

```mysql
[mysqld]
ngram_token_size=2
```

这里把分词大小设置为2。要记住，分词的SIZE越大，索引的体积就越大，所以要根据自身情况来设置合适的大小。示例表结构：

```mysql
CREATE TABLE articles (
   id   INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
   title VARCHAR(200),
   body  TEXT,
   FULLTEXT (title,body) WITH PARSER ngram
  ) ENGINE=InnoDB CHARACTER SET utf8mb4;
```

示例数据，有6行记录。

```mysql
mysql> select * from articles\G
***************************1. row ***************************
  id: 1
title: 数据库管理
 body: 在本教程中我将向你展示如何管理数据库
***************************2. row ***************************
  id: 2
title: 数据库应用开发
 body: 学习开发数据库应用程序
***************************3. row ***************************
  id: 3
title: MySQL完全手册
 body: 学习MySQL的一切
***************************4. row ***************************
  id: 4
title: 数据库与事务处理
 body: 系统的学习数据库的事务概论
***************************5. row ***************************
  id: 5
title: NoSQL精髓
 body: 学习了解各种非结构化数据库
***************************6. row ***************************
  id: 6
title: SQL 语言详解
 body: 详细了解如果使用各种SQL
6 rows inset (0.00 sec)
```

显式指定全文检索表源

```mysql
mysql> SETGLOBAL innodb_ft_aux_table="new_feature/articles";
Query OK, 0 rows affected (0.00 sec)
```

通过系统表，就可以查看到底是怎么划分articles里的数据。

```mysql
mysql> SELECT *FROM information_schema.INNODB_FT_INDEX_CACHE LIMIT 20,10;
+------+--------------+-------------+-----------+--------+----------+
| WORD | FIRST_DOC_ID | LAST_DOC_ID | DOC_COUNT | DOC_ID| POSITION  |
+------+--------------+-------------+-----------+--------+----------+
| 中我 |   2          |   2         |   1       |  2     |  28      |
| 习m  |   4          |   4         |   1       |  4     |  21      |
| 习了 |   6          |   6         |   1       |  6     |  16      |
| 习开 |   3          |   3         |   1       |  3     |  25      |
| 习数 |   5          |   5         |   1       |  5     |  37      |
| 了解 |   6          |   7         |   2       |  6     |  19      |
| 了解 |   6          |   7         |   2       |  7     |  23      |
| 事务 |   5          |   5         |   1       |  5     |  12      |
| 事务 |   5          |   5         |   1       |  5     |  40      |
| 何管 |   2          |   2         |   1       |  2     |  52      |
+------+--------------+-------------+-----------+--------+----------+
10 rows in set (0.00 sec)
```

这里可以看到，把分词长度设置为2，所有的数据都只有两个一组。上面数据还包含了行的位置，ID等等信息。接下来，我来进行一系列检索示范，使用方法和原来英文检索一致。

## 一、自然语言模式

###  1、得到符合条件的个数

```mysql
mysql>SELECT COUNT(*) FROM articles
-> WHERE MATCH (title,body) AGAINST ('数据库' IN NATURALLANGUAGE MODE);
+----------+
| COUNT(*) |
+----------+
|  4 |
+----------+
1 row in set (0.05 sec)
```

### 2、得到匹配的比率

```mysql
mysql>SELECT id, MATCH (title,body) AGAINST ('数据库' IN NATURAL LANGUAGE MODE)
 AS score FROM articles;
+----+----------------------+
| id| score               |
+----+----------------------+
| 1 | 0.12403252720832825 |
| 2 | 0.12403252720832825 |
| 3 | 0                   |
| 4 | 0.12403252720832825 |
| 5 | 0.062016263604164124|
| 6 | 0                   |
+----+----------------------+
6rows in set (0.00 sec)
```

## 二、布尔模式

这个就相对于自然模式搜索来的复杂些：

### 1、匹配既有管理又有数据库的记录

```mysql
mysql> SELECT * FROM articles WHERE MATCH (title,body)
  ->  AGAINST ('+数据库 +管理' IN BOOLEAN MODE);
+----+------------+--------------------------------------+
| id | title      | body                                 |
+----+------------+--------------------------------------+
| 1  | 数据库管理 | 在本教程中我将向你展示如何管理数据库 |
+----+------------+--------------------------------------+
1 rowin set (0.00 sec)
```

### 2、匹配有数据库，但是没有管理的记录

```mysql
mysql> ``SELECT` `* ``FROM` `articles ``WHERE` `MATCH (title,body)
 ``-> AGAINST (``'+数据库 -管理'` `IN` `BOOLEAN MODE);
+``----+------------------+----------------------------+
| id   | title            | body                       |
+``----+------------------+----------------------------+
| 2    | 数据库应用开发   | 学习开发数据库应用程序     |
| 4    | 数据库与事务处理 | 系统的学习数据库的事务概论 |
| 5    | NoSQL 精髓       | 学习了解各种非结构化数据库 |
+``----+------------------+----------------------------+
3 ``rows` `in` `set` `(0.00 sec)
```

### 3、匹配MySQL，但是把数据库的相关性降低

```mysql
mysql> SELECT * FROM articles WHERE MATCH (title,body)
  ->  AGAINST ('>数据库 +MySQL' INBOOLEAN MODE);
+----+---------------+-----------------+
| id | title         | body            |
+----+---------------+-----------------+
| 3  | MySQL完全手册 |学习MySQL的一切  |
+----+---------------+-----------------+
1 rowin set (0.00 sec)
```

## 三、查询扩展模式

比如要搜索数据库，那么MySQL，oracle，DB2也都将会被搜索到

```mysql
mysql> SELECT * FROM articles
  ->  WHERE MATCH (title,body)
  ->  AGAINST ('数据库' WITH QUERY EXPANSION);
+----+------------------+--------------------------------------+
| id | title            | body                                 |
+----+------------------+--------------------------------------+
| 1  | 数据库管理       | 在本教程中我将向你展示如何管理数据库 |
| 4  | 数据库与事务处理 | 系统的学习数据库的事务概论           |
| 2  | 数据库应用开发   | 学习开发数据库应用程序               |
| 5  | NoSQL 精髓       | 学习了解各种非结构化数据库           |
| 6  | SQL 语言详解     | 详细了解如果使用各种SQL              |
| 3  | MySQL完全手册    | 学习MySQL的一切                      |
+----+------------------+--------------------------------------+
6 rows in set (0.01 sec)
```

