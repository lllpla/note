# mybatis的oracle自增主键配置 

mybatis对应mysql的自增主键非常简单，但是oracle则需要绑定序列。  

而且如果要返回自增主键的值，可以参考如下配置。 

## 目标

1. 查询序列并获得自增主键的值，设置为主键 
2. 根据获得的值插入新数据到表中 

xxxMap.xml代码如下 
```xml
<insert id="insert" parameterType="com.drpeng.taskframe.entity.TaskLog"> 

insert into TASK_LOG (TASK_LOG_ID, TASK_ID, RESULTS, 

PERCENT, START_DATE, FINISH_DATE,  

STATE, REMARKS) 

values (TASK_LOG$SEQ.nextval, #{taskId,jdbcType=DECIMAL}, #{results,jdbcType=VARCHAR}, 

#{percent,jdbcType=DECIMAL}, #{startDate,jdbcType=TIMESTAMP}, #{finishDate,jdbcType=TIMESTAMP},  

#{state,jdbcType=CHAR}, #{remarks,jdbcType=VARCHAR}) 

</insert> 

<insert id="insertSelective" parameterType="com.drpeng.taskframe.entity.TaskLog"> 

<selectKey resultType="java.lang.Long" order="BEFORE" keyProperty="taskLogId"> 

SELECT TASK_LOG$SEQ.nextval as taskLogId from DUAL 

</selectKey> 

insert into TASK_LOG 

<trim prefix="(" suffix=")" suffixOverrides=","> 

<if test="taskLogId != null"> 

TASK_LOG_ID, 

</if> 

<if test="taskId != null"> 

TASK_ID, 

</if> 
```