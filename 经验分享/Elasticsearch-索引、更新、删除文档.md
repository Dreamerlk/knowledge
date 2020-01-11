一、Elasticsearch 索引(新建)一个文档的命令：
```
curl XPUT ' http://localhost:9200/test_es_order_index/test_es_order_type/1 ' -d  '
{
"id": 5,
"name": "test555",
"skuName": "55",

"age":23

}
'
```
这里test_es_order_index 是_index名称，test_es_order_type是_type类型名称，1是_id号。

需要注意的是，如果不指定ID，那么需要使用POST命令，而不是PUT。

二、更新文档

除了索引和替换文档，ES还支持更新文档。更新文档其实是先删除旧的文档，再索引新的文档。

如果想要更新文档内容，可以按照下面的方式进行：
```
curl -XPOST 'localhost:9200/test_es_order_index/test_es_order_type/1/_update' -d '
{
  "doc": { "name": "Jane Doe" }
}'
```
由于是先删除再索引，因此可以额外增加新的字段：
```
curl -XPOST 'localhost:9200/test_es_order_index/test_es_order_type/1/_update' -d '
{
  "doc": { "name": "Jane Doe", "age": 20 }
}'
```
当然也支持使用脚本进行更新：
```
curl -XPOST 'localhost:9200/test_es_order_index/test_es_order_type/1/_update' -d '
{
  "script" : "ctx._source.age += 5"
}'
```
其中ctx._source代表了当前的文档，上面的意思 是 在当前文档的基础上age加5.

三、删除文档

删除文档就很简单了，只需要指定文档的索引、类型、ID就行了：
```
curl -XDELETE 'localhost:9200/test_es_order_index/test_es_order_type/1'
```
四、批量操作

除了索引、替换、更新和删除，ES为了减少来回的响应信息，可以一次性执行多个命令，最后统一返回执行结果。

例如：
```
curl -XPOST 'localhost:9200/test_es_order_index/test_es_order_type/_bulk?pretty' -d ' 

{"index":{"_id":"1"}} {"name": "John Doe" } {"index":{"_id":"2"}} {"name": "Jane Doe" } '
```

上面的命令可以同时插入两条数据。

_bulk命令不仅仅支持单个命令执行多条，还只是多种不同的命令执行多条。

```
curl -XPOST 'localhost:9200/test_es_order_index/test_es_order_type/_bulk?pretty' -d ' 
{"update":{"_id":"1"}} {"doc": { "name": "John Doe becomes Jane Doe" } } {"delete":{"_id":"2"}} '
```
上面的命令中，先更新id为1的文档，再删除id为2的文档。
