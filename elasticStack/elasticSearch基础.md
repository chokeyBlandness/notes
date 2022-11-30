# ElasticSearch基础

### 一、安装启动

兼容性：[https](https://www.elastic.co/cn/support/matrix)[://www.elastic.co/cn/support/matrix](https://www.elastic.co/cn/support/matrix)

#### 1.1 启动es

配置文件 config/elasticsearch.yml

```yaml
# 默认只能通过localhost访问
network.host: 0.0.0.0
http.port: 9200

cluster.initial_master_nodes: ["node-1"]
```

启动elasticSearch

```shell
./bin/elasticsearch -d
```

> 问题一：启动异常，max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
>
> 解决方法：
>
> 通过 `sudo vi /etc/sysctl.conf` 添加配置 `vm.max_map_count=262144` ，再执行 `sysctl -p`。
>
> 问题二：启动异常。the default discovery settings are unsuitable for production use;at least one of [discovery.seed_hosts, discovery.seed_providers. cluster.initial_master_nodes] must be configured。
>
> 解决方法：
>
> 在config/elasticsearch.yml中配置 discovery.seed_hosts 等配置

#### 1.2 启动kibana

配置文件 config/kibana.yml

```yaml
# 访问端口
server.port: 5601
# 配置host，默认localhost无法对外提供服务
server.host: "0.0.0.0"

elasticsearch.hosts: ["http://localhost:9200"]

```

启动kibana

```shell
nohup ./bin/kibana >kibana.out &
```

#### 1.3 安装ElasticSearch-head插件

一个用来查看ES集群信息的前端页面。

git地址拉取源码本地安装：https://github.com/mobz/elasticsearch-head。

或者直接安装chrome插件，可在上面git仓库的crx目录下找到插件压缩包，直接插件启动。

#### 1.4 健康值检查

head插件查看，或kibana中devTools命令查看 `GET _cat/health?v` 或`GET _cluster/health` 。

### 二、DSL

ES支持RESTful风格的API调用

在kibana的devTools工具命令行中进行操作

索引的增删改查：`GET/PUT/POST/DELETE /indexName` 。
GET _cat/indices?v ：带表头显示所有索引信息。
PUT /indexName/_doc/id ：向索引中插入doc数据。

全文检索match：
```
get index/_search
{
	"query": {
		"match": {
			"name": "ali book"
		}
	}
}
```

精准查询term：不会将搜索内容分词搜索

term和match_phrase的区别：match_phrase会分词搜索

```
get index/_search
{
	"query": {
		"term": {
			"name": "ali book"
		}
	}
}
```

过滤查找filter：

```
get index/_search
{
	"query": {
		"constant_score": {
			"filter": {
				"term": {
					"name": "ali book"
				}
			}
		}
	}
}
```

组合查找bool：must和should

must是和的关系，should是或的关系。should在有其他限制条件的情况下，默认满足0个或多个

```
get index/_search
{
	"query": {
		"bool": {
			"must": [{
				"match": {
					"name": "ali book"
				}
			}],
			"should": [{
				"match_phase": {
					"desc": "machine learning"
				}
			}]
		}
	}
}
```

