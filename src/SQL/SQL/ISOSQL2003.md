# ISO SQL 2003 常用语法

## 数据库创建
语法：
```SQL
CREATE DATABASE <databaseName>;
```

## 创建表
语法：
```SQL
CREATE TABLE <tableName> (
    <columnName> <dataType> <check>,
    <columnName> <dataType> <check>,
    <columnName> <dataType> <check>
)
```
数据类型：
* 数字型  SMALLINT、INTEGER、BIGINT、NUMERIC(p,s) 精确数字
* 字符型  CHAR(n) 固定长度字符、VARCHAR(n)可变长字符
* 日期型  DATE日期、TIME 时间、TIMESTAMP 时间戳

## 修改表名
```SQL
ALTER TABLE <tableName> RENAME TO <newTableName>;
```

## 删除表
```SQL
DROP TABLE <tableName>;
```

## 表更新
添加列
```SQL
ALTER TABLE <tableName> ADD COLUMN <columnName>;
```
删除列
```SQL
ALTER TABLE <tableName> DROP <columnName>;
```


case 表达式
语法格式
```sql
CASE 
    WHEN  THEN  
    ELSE  
END
```
注意事项：
1. 统一各个分支的数据返回类型
2. 不要忘记写 END
3. 养成写 ELSE 的习惯