[TOC]

# Arch Linux搭建Elastic Search环境

## 安装Elastic Search

可以在Arch官方库中直接安装：  

```bash
yay -S elasticsearch
```

## 启动Elastic Search

开启elastic search服务：  

```bash
# 本次有效
systemctl start elasticsearch
# 开机自启
systemctl enable elasticsearch
```

## 测试Elastic Search

测试Rest API是否可用，直接访问 `9200` 端口：  

```bash
curl http://127.0.0.1:9200
```

```
{
  "name" : "GLaDOS",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "l4g7d-GvQ3qDL8kWdSopzA",
  "version" : {
    "number" : "7.3.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "1c1faf1",
    "build_date" : "2019-09-06T14:40:30.409026Z",
    "build_snapshot" : false,
    "lucene_version" : "8.1.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

## 自定义Elastic Search配置

主配置文件存放在 `/etc/elasticsearch/elasticsearch.yml` 。  

若需要修改分配的内存，配置文件在 `/etc/elasticsearch/jvm.options` 。  

## 安装Kibana

`KIbana` 用于监控Elastic Search集群的状态，它提供了良好的UI界面。  

可以在Arch官方库中直接安装：  

```bash
yay -S kibana
```

## 启动Kibana

开启Kibana服务：  

```bash
# 本次有效
systemctl start kibana
# 开机自启
systemctl enable kibana
```

## 测试Kibana

直接访问 `5601` 端口：  `http://localhost:5601` 。  



## 自定义Kibana配置

主配置文件放在 `/etc/kibana/kibana.yml` 。  

修改配置后，需要restart Kibana服务。  