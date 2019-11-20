@echo off

set cmis_app_dir=E:/publish/cmis_java/cmis.jar
set log_out=E:/publish/cmis_java/logs/cmis/log.out
set server_port=8010

::for /f "tokens=2 delims=:" %%i in ('ipconfig^|findstr "Address"') do set ip=%%i
::setx /m deploy_host_ip "%ip%"
::echo localhost ip:%ip%.....
::setx /m deploy_host_name "cmis"

echo start cmis.......:"java -jar -Dinstance.flag=server1 -Xms256m  -Xmx1024m  -XX:MetaspaceSize=64m -XX:MaxMetaspaceSize=512m %cmis_app_dir% --server.port=%server_port% "
start javaw -jar -Dinstance.flag=cmis -Xms256m  -Xmx1024m  -XX:MetaspaceSize=64m -XX:MaxMetaspaceSize=512m %cmis_app_dir% --server.port=%server_port% 

exit
::pause & exit
