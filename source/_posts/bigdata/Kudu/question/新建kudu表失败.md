---
title: 新建kudu表失败
date: 2019-05-27 16:59:33
categories:
- 大数据
tags:
- Kudu
- 问题解决
---

**新建Kudu表失败**

解决办法：指定kudu.master_addresses

```sql
CREATE TABLE realtime_login_input_test2 (  
 secgamecode STRING ,  
 logintime TIMESTAMP ,  
 sign STRING ,   
 time BIGINT ,  
 ip STRING ,  
 deviceid STRING , 
 language STRING,  
 systemversion STRING , 
 mac STRING NULL ,  
 thirdplate STRING , 
 devicetype STRING ,
 appplatform STRING , 
 idfa STRING , 
 advertising_id STRING ,   
 imei STRING,
 gamecode STRING, 
 androidid STRING,  
 logintimestr BIGINT,
 userid BIGINT ,
 platform STRING , 
 countrycode STRING,
 regplatform STRING , 
 regcountrycode STRING,  
 PRIMARY KEY (secgamecode, logintime, sign) )
 PARTITION BY HASH (secgamecode) PARTITIONS 4, RANGE (logintime) (PARTITION VALUES < '2018-01-01 00:00:00') 
 STORED AS KUDU TBLPROPERTIES ('kudu.master_addresses'='ehdp-node-4');
```