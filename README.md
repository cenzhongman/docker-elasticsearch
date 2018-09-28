# 使用Docker安装分布式的ElasticSearch

[参考](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/docker.html)

* 版本：6.4
* 日期：2018年9月5日、
* 系统：Linux虚拟机
* 前提：

## 单节点安装Docker

1. 查看自己最大文件限制

```sh
grep vm.max_map_count /etc/sysctl.conf
# 必须大于262144
```

2. 运行

* 可以加上参数 `-d`使程序在后台运行

```sh
docker run -d --name elasticsearch-simgle -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.4.0
```

* 9300用于ElasticSearch节点之间的通信、客户端通信
* 9200端口用于提供Rest API

## 多节点安装

使用`docker-compose.yml`文件

```yml
version: '2.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.4.0
    container_name: elasticsearch
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
      - 9300:9300
    networks:
      - esnet

  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.4.0
    container_name: elasticsearch2
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=elasticsearch"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata2:/usr/share/elasticsearch/data
    networks:
      - esnet

  elasticsearch3:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.4.0
    container_name: elasticsearch3
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=elasticsearch"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata3:/usr/share/elasticsearch/data
    networks:
      - esnet
  kibana:
    image: docker.elastic.co/kibana/kibana:6.4.0
    container_name: kibana
    environment:
      SERVER_NAME: kibana
      ELASTICSEARCH_URL: http://docker-cluster:9200
    ports:
      - 5601:5601
    networks:
      - esnet

volumes:
  esdata1:
    driver: local
  esdata2:
    driver: local
  esdata3:
    driver: local

networks:
  esnet:
```

执行

```sh
docker-compose up -d
```

* -d 表示后台运行