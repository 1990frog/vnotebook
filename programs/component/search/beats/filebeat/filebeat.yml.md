```yml
filebeat.inputs:
    # type有 log|stdin|redis|udp|docker
    - type: log
      enabled: true
      paths:
          - /home/cai/Code/Java/logs/error/*.log

filebeat.config.modules:
    path: ${path.config}/modules.d/*.yml
    reload.enabled: false

# 指定kibana的地址，用于导入dashboard
setup.kibana:
    host: "localhost:5601"

# output只能有一个，可以通过enbaled来开关
output.elasticsearch:
    hosts: ["localhost:9200"]
```