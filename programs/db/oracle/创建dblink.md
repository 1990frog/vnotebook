```sql
create public database link dbtest
    connect to "his_pengz_bak190909" identified by "123456"
    using '(DESCRIPTION =(ADDRESS_LIST =(ADDRESS =(PROTOCOL = TCP)(HOST = 192.168.0.251)(PORT = 1521)))(CONNECT_DATA =(SERVICE_NAME = orcl)))';
```