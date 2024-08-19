# ElasticSearch入门教程By Gary
<img src="img/logo.png"  alt="无法显示该图片" />

## ES CURL常用命令：
> 1. 检查ES集群健康状态：curl 'localhost:9200/_cat/health?v&pretty'
> 2. 查看ES集群所有节点信息：curl 'localhost:9200/_cat/nodes?v&pretty'
> 3. 查找别名为Alias_name的Index：curl 'localhost:9200/_alias/Alias_name?pretty'
> 4. 查询Index下的所有数据（默认返回10条结果）：curl 'localhost:9200/Index/_search?q=*&pretty'
> 5. 根据Id查询单条数据：curl 'localhost:9200/Index/Type/Id?pretty'
> 6. 删除Index：curl -XDELETE 'localhost:9200/Index?pretty'
> 7. 新建Index：curl -XPUT 'localhost:9200/Index?pretty'
> 8. 插入单条数据：curl -XPUT 'localhost:9200/Index/Type/Id?pretty' -d '{"name":"gary"}'
> 9. 删除单条数据：curl -XDELETE 'localhost:9200/Index/Type/Id?pretty'
> 10. 查看ES集群所有Index：curl 'localhost:9200/_cat/indices?v&pretty'
> 11. 分页查询From&Size（每个Shard取100条数据，然后再排序取前10条）：curl 'localhost:9200/Index/Type/_search?pretty' -d '{"from":100,"size":10,"query":{"term":{"user":"gary"}}}'
> 12. 修改默认分页深度（默认分页深度为10000，超过就会报错）：curl -XPUT 'localhost:9200/Index/_settings' -d '{"index":{"max_result_window":1000000}}'
> 13. 分页查询scroll（scroll=30m：scroll保存时间，search_type=scan：表示不排序，size=400：每个分片一次返回多少条数据）：(1)curl 'localhost:9200/Index/Type/_search?pretty&scroll=30m&search_type=scan&size=400' -d 'query查询语句'  (2)curl 'localhost:9200/_search/scroll?scroll=30m&pretty' -d '(1)结果中的"_scroll_id"值'
