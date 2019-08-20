# test

load data local inpath '/home/cai/Desktop/test.log' overwrite into table people;


CREATE EXTERNAL TABLE people ( name STRING,age STRING,level STRING,faction STRING )  ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe' LOCATION '/user/data';