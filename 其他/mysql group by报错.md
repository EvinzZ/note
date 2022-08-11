# mysql group by报错

## 错误日志

```Bash
### Error querying database.  Cause: java.sql.SQLSyntaxErrorException: Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'datacollection.source.id' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
### The error may exist in com/framework/control/api/mapper/SourceMapper.java (best guess)
### The error may involve defaultParameterMap
### The error occurred while setting parameters
### SQL: select id, name from source where type = 5 group by url
### Cause: java.sql.SQLSyntaxErrorException: Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'datacollection.source.id' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
; bad SQL grammar []; nested exception is java.sql.SQLSyntaxErrorException: Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'datacollection.source.id' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by

org.springframework.jdbc.BadSqlGrammarException: 
### Error querying database.  Cause: java.sql.SQLSyntaxErrorException: Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'datacollection.source.id' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
### The error may exist in com/framework/control/api/mapper/SourceMapper.java (best guess)
### The error may involve defaultParameterMap
### The error occurred while setting parameters
### SQL: select id, name from source where type = 5 group by url
### Cause: java.sql.SQLSyntaxErrorException: Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'datacollection.source.id' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
; bad SQL grammar []; nested exception is java.sql.SQLSyntaxErrorException: Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'datacollection.source.id' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
  at org.springframework.jdbc.support.SQLExceptionSubclassTranslator.doTranslate(SQLExceptionSubclassTranslator.java:93) ~[spring-jdbc-5.3.17.jar:5.3.17]
  at org.springframework.jdbc.support.AbstractFallbackSQLExceptionTranslator.translate(AbstractFallbackSQLExceptionTranslator.java:70) ~[spring-jdbc-5.3.17.jar:5.3.17]
  at org.springframework.jdbc.support.AbstractFallbackSQLExceptionTranslator.translate(AbstractFallbackSQLExceptionTranslator.java:79) ~[spring-jdbc-5.3.17.jar:5.3.17]
  at org.mybatis.spring.MyBatisExceptionTranslator.translateExceptionIfPossible(MyBatisExceptionTranslator.java:91) ~[mybatis-spring-2.0.7.jar:2.0.7]
  at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:441) ~[mybatis-spring-2.0.7.jar:2.0.7]
  at com.sun.proxy.$Proxy86.selectList(Unknown Source) ~[na:na]
  at org.mybatis.spring.SqlSessionTemplate.selectList(SqlSessionTemplate.java:224) ~[mybatis-spring-2.0.7.jar:2.0.7]
  at org.apache.ibatis.binding.MapperMethod.executeForMany(MapperMethod.java:147) ~[mybatis-3.5.9.jar:3.5.9]
  at org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:80) ~[mybatis-3.5.9.jar:3.5.9]
  at org.apache.ibatis.binding.MapperProxy$PlainMethodInvoker.invoke(MapperProxy.java:145) ~[mybatis-3.5.9.jar:3.5.9]
  at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:86) ~[mybatis-3.5.9.jar:3.5.9]
  at com.sun.proxy.$Proxy92.getSearchEngine(Unknown Source) ~[na:na]
  at com.framework.control.api.service.SearchSourceService.getSearchEngine(SearchSourceService.java:51) ~[classes/:na]
  at com.framework.control.api.controller.SearchSourceController.getSearchEngine(SearchSourceController.java:27) ~[classes/:na]
  at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_261]
  at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_261]
  at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_261]
  at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_261]
  
```

## 解决方法

一、永久方法
 my.ini添加（Windows中MySQL配置文件）springboot文件下载

```Ini
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

2.临时修改

```SQL
1、先使用SQL查询sql_mode
select @@global.sql_mode

2、重新设置sql_mode，删除ONLY_FULL_GROUP_BY
set @@global.sql_mode ='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'

3、退出重进
# 退出
quit
# 再次登录（命令行模式）
mysql -u root -p

4.再次执行查询语句，
```