# 简单ORACLE分区表、分区索引
## 1、类型说明： 
range分区方式，也算是最常用的分区方式，其通过某字段或几个字段的组合的值，从小到大，按照指定的范围说明进行分区，我们在INSERT数据的时候就会存储到指定的分区中。 

List分区方式，一般是在range基础上做的二级分区较多，是一种列举方式进行分区，一般讲某些地区、状态或指定规则的编码等进行划分。 

Hash分区方式，它没有固定的规则，由ORACLE管理，只需要将值INSERT进去，ORACLE会自动去根据一套HASH算法去划分分区，只需要告诉ORACLE要分几个区即可。   

## 2、分区应用： 

一般一张表超过2G的大小，ORACLE是推荐使用分区表的，分区一般都需要创建索引，说到分区索引，就可以分为：全局索引、分区索引，即：global索引和local索引，前者为默认情况下在分区表上创建索引时的索引方式，并不对索引进行分区（索引也是表结构，索引大了也需要分区，关于索引以后专门写点）而全局索引可修饰为分区索引，但是和local索引有所区别，前者的分区方式完全按照自定义方式去创建，和表结构完全无关，所以对于分区表的全局索引有以下两幅网上常用的图解： 

### 2.1、对于分区表的不分区索引（这个有点绕，不过就是表分区，但其索引不分区）： 
![title](https://raw.githubusercontent.com/lllpla/img/master/gitnote/2020/04/11/1586600325966-1586600325970.png)
### 2.2、对于分区表的分区索引: 
![title](https://raw.githubusercontent.com/lllpla/img/master/gitnote/2020/04/11/1586600355358-1586600355365.png)
全局分区索引 

创建语法为： 
```sql
CREATE INDEX INX_TAB_PARTITION_COL1 ON TABLE_PARTITION(COL1)   
GLOBAL PARTITION BY RANGE(COL1)(   
         PARTITION IDX_P1 values less than (1000000),   
         PARTITION IDX_P2 values less than (2000000),   
         PARTITION IDX_P3 values less than (MAXVALUE)   
  );   
```
### 2.3、LOCAL索引结构： 
![title](https://raw.githubusercontent.com/lllpla/img/master/gitnote/2020/04/11/1586600435996-1586600436002.png)
创建语法为： 
```sql
CREATE INDEX INX_TAB_PARTITION_COL1 ON TABLE_PARTITION(COL1) LOCAL;   
```
也可按照分区表的的分区结构给与一一定义，索引的分区将得到重命名。 

分区上的位图索引只能为LOCAL索引，不能为GLOBAL全局索引。 
### 2.4、对比索引方式： 

  一般使用LOCAL索引较为方便，而且维护代价较低，并且LOCAL索引是在分区的基础上去创建索引，类似于在一个子表内部去创建索引，这样开销主要是区分分区上，很规范的管理起来，在OLAP系统中应用很广泛；而相对的GLOBAL索引是全局类型的索引，根据实际情况可以调整分区的类别，而并非按照分区结构一一定义，相对维护代价较高一些，在OLTP环境用得相对较多，这里所谓OLTP和OLAP也是相对的，不是特殊的项目，没有绝对的划分概念，在应用过程中依据实际情况而定，来提高整体的运行性能。 
## 3、常用视图
```sql
1、查询当前用户下有哪些是分区表：   
SELECT * FROM USER_PART_TABLES;       
2、查询当前用户下有哪些分区索引：   
SELECT * FROM USER_PART_INDEXES;       
3、查询当前用户下分区索引的分区信息：   
SELECT * FROM USER_IND_PARTITIONS T   
WHERE T.INDEX_NAME=?       
4、查询当前用户下分区表的分区信息：   
SELECT * FROM USER_TAB_PARTITIONS T   
WHERE T.TABLE_NAME=?;       
5、查询某分区下的数据量：   
SELECT COUNT(*) FROM TABLE_PARTITION PARTITION(TAB_PARTOTION_01);   
6、查询索引、表上在那些列上创建了分区：   
SELECT * FROM USER_PART_KEY_COLUMNS;       
7、查询某用户下二级分区的信息（只有创建了二级分区才有数据）：   
SELECT * FROM USER_TAB_SUBPARTITIONS;  
```
## 4、维护操作
### 4.1、删除分区 
```sql
ALTER TABLE TABLE_PARTITION DROP PARTITION TAB_PARTOTION_03;   
```
如果是全局索引，因为全局索引的分区结构和表可以不一致，若不一致的情况下，会导致整个全局索引失效，在删除分区的时候，语句修改为：   
```sql
ALTER TABLE TABLE_PARTITION DROP PARTITION TAB_PARTOTION_03 UPDATE GLOBAL INDEXES   
```
### 4.2、分区合并（从中间删除掉一个分区，或者两个分区需要合并后减少分区数量） 
合并分区和删除中间的RANGE有点像，但是合并分区是不会删除数据的，对于LIST、HASH分区也是和RANGE分区不一样的，其语法为： 
```sql
ALTER TABLE TABLE_PARTITION MERGE PARTITIONS    TAB_PARTOTION_01,TAB_PARTOTION_02 INTO PARTITION MERGED_PARTITION;   
```
### 4.3、分隔分区（一般分区从扩展分区从分隔）  
```sql
ALTER TABLE TABLE_PARTITION SPLIT PARTITION TAB_PARTOTION_OTHERE AT(2500000)    

INTO (PARTITION TAB_PARTOTION_05,PARTITION TAB_PARTOTION_OTHERE);   
```
### 4.4、创建新的分区（分区数据若不能提供范围，则插入时会报错，需要增加分区来扩大范围） 

一般有扩展分区的是都是用分隔的方式，若上述创建表时没有创建TAB_PARTOTION_OTHER分区时，在插入数据较大时（按照上述建立规则，超过1800000就应该创建新的分区来存储），就可以创建新的分区，如： 

为了试验，我们将扩展分区先删除掉再创建新的分区（因为ORACLE要求，分区的数据不允许重叠，即按照分区字段同样的数据不能同时存储在不同的分区中）： 
```sql
ALTER TABLE TABLE_PARTITION DROP PARTITION TAB_PARTOTION_OTHER;   

ALTER TABLE TABLE_PARTITION ADD PARTITION TAB_PARTOTION_06 VALUES LESS THAN(2500000); 
```
在分区下创建新的子分区大致如下（RANGE分区，若为LIST或HASH分区，将创建方式修改为对应的方式即可）： 
```sql
ALTER TABLE <table_name> MODIFY PARTITION <partition_name> ADD SUBPARTITION <user_define_subpartition_name> VALUES LESS THAN(....);   
```
### 4.5、修改分区名称（修改相关的属性信息） 
```sql
ALTER TABLE TABLE_PARTITION RENAME PARTITION MERGED_PARTITION TO MERGED_PARTITION02;   
```
### 4.6、交换分区（快速交换数据，其实是交换段名称指针） 
  首先创建一个交换表，和原表结构相同，如果有数据，必须符合所交换对应分区的条件： 
```sql 
CREATE TABLE TABLE_PARTITION_2   
  AS SELECT * FROM TABLE_PARTITION WHERE 1=2;   
```
  然后将第一个分区的数据交换出去： 
```sql
ALTER TABLE TABLE_PARTITION EXCHANGE PARTITION TAB_PARTOTION_01    
WITH TABLE TABLE_PARTITION_2 INCLUDING INDEXES;   
```
  此时会发现第一个分区的数据和表TABLE_PARTITION_2做了瞬间交换，比TRUNCATE还要快，因为这个过程没有进行数据转存，只是段名称的修改过程，和实际的数据量没有关系。 

  如果是子分区也可以与外部的表进行交换，只需要将关键字修改为：SUBPARTITION 即可。 
### 4.7、清空分区数据 
```sql
ALTER TABLE <table_name> TRUNCATE PARTITION <partition_name>;   
   ALTER TABLE <table_name> TRUNCATE subpartition <subpartition_name>;   
```
### 4.8、磁盘碎片压缩 
   对分区表的某分区进行磁盘压缩，当对分区内部数据进行了大量的UPDATE、DELETE操作后，一定时间需要进行磁盘压缩，否则在查询时，若通过FULL SCAN扫描数据，将会把空块也会扫描到，对表进行磁盘压缩需要进行行迁移操作，所以首先需要操作： 
```sql
ALTER TABLE <table_name> ENABLE ROW MOVEMENT ;   
```
对分区表的某分区压缩语法为：   
```sql
ALTER TABLE <table_name>   
modify partition <partition_name> shrink space;   
```
对普通表压缩：   
```sql
ALTER TABLE <table_name> shrink space;   
```
  对于索引也需要进行压缩，索引也是表：   
```sql
ALTER INDEX <index_name> shrink space;   
```
### 4.9.分区表重新分析以及索引重新分析 
  对表进行压缩后，需要对表和索引进行重新分析，对表进行重新分析，一般有两种方式： 

  在ORACLE 10G以前，使用： 
```sql
BEGIN   
   dbms_stats.gather_table_stats(USER,UPPER('<table_name>'));   

END;   

   

ORACLE 10G后，可以使用：   

ANALYZE TABLE <table_name> COMPUTE STATISTICS;   

  

  索引重新分析，将上述两种方式分别修改一下，如第一种可以使用：gather_index_stats，而第二种修改为：ANALYZE INDEX即可，不过一般比较常用的是重新编译： 

  对于分区表并进行了索引分区的情况，需要对每个分区的索引进行重新编译,这里以LOCAL索引为例子（其每个索引的分区和表分区结构相同，默认分区名称和表分区名称相同）： 

[sql] view plain copy 

ALTER INDEX <index_name> REBUILD PARTITION <partition_name>;   

 对于全局索引，根据全局索引锁定义的分区名称修改即可，若没有分区，和普通单表索引重新编译方式相同：   

ALTER INDEX <index_name> REBUILD;   

  

11、关联对象重新编译 

  上述对表、索引进行重新编译，尤其对表进行了压缩后会产生行迁移，这个过程可能会导致一些视图、过程对象的失效，此时要将其重新编译一次。 

  

  

12、扩展：HASH分区中，如果创建了新的分区，可以将其进行重新HASH分布： 

[sql] view plain copy 

ALTER TABLE <table_name> COALESCA PARTITION   