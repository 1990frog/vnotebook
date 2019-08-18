# test


CREATE TABLE people
(
  name STRING,
  age        STRING,
  level   STRING,
  faction      STRING)
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
STORED AS TEXTFILE;


load data local inpath '/home/cai/Desktop/test.log' overwrite into table people;

CREATE EXTERNAL TABLE test ( text string ) ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe' LOCATION '/test.json';
