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
```
ALTER TABLE TABLE_PARTITION DROP PARTITION TAB_PARTOTION_OTHER;   

ALTER TABLE TABLE_PARTITION ADD PARTITION TAB_PARTOTION_06 VALUES LESS THAN(2500000); 
```