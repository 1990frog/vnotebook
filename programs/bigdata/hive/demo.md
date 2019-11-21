# test

load data local inpath '/home/cai/Desktop/test.log' overwrite into table people;


CREATE EXTERNAL TABLE people (
name STRING,
age STRING,
level STRING,
faction STRING )  
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe' 
LOCATION '/user/data';


CREATE EXTERNAL TABLE people ( 
name STRING,
age STRING,
level STRING,
faction STRING,
master array<string>,
student array<string>,
gest array<string>,
wife array<string>)  
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe' 
LOCATION '/user/data';


select  count(*)   from  xx
lateral view  explode(pair) ids_table1  as id1   lateral view  explode(pair) ids_table2  as id2  
where  year='2016' and month='02'  and day='23' and id2.type=68 and id1.type=64  and array_contains(dates,'20160223'); 
