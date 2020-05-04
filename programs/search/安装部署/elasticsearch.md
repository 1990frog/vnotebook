[TCO]

# config/elasticsearch.yml
```yml
# 集群名称
cluster.name: my-cluster
# 节点名称        
node.name: node-1
#node.attr.rack: r1
path.data: /usr/data/elk/node1
path.logs: /usr/logs/elk/node1
#bootstrap.memory_lock: true
# 配置访问本节点的ip
network.host: 0.0.0.0  
# 设置对外服务的http端口，默认为9200
http.port: 9200
# 设置节点间交互的tcp端口,默认是9300
transport.tcp.port: 9300
# 配置所有用来组建集群的机器的IP地址     
discovery.seed_hosts: ["localhost:9300", "localhost:9301"]
#cluster.initial_master_nodes: ["node-1", "node-2"]
#gateway.recover_after_nodes: 3
#action.destructive_requires_name: true
```