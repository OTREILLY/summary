### +++ elasticsearch restful api

#### 获取文档
```
curl GET /myindex/mytype/myid?pretty
curl GET /myindex/mytype/myid?_source=[field1,field2]  //获取_source的指定字段
curl GET /myindex/mytype/myid?_source				//只获取_source
curl -i -XGET /myindex/mytype/myid?pretty   //能够显示响应的头部
```

#### 检查文档是否存在
```
curl -i -XHEAD /myindex/mytype/myid
```

#### 更新文档
```
curl put /myindex/mytype/myid  //created 标志设置成 false ，表示相同的索引、类型和 ID 的文档已经存在
```

#### 创建新文档
```
curl POST /myindex/mytype/	//自增_id
{ ... }

PUT /myindex/mytype/myid?opmytype=created  		//自定义id，且文档不存在是才能成功创建
{ ... }	

```

#### 删除文档
```
curl DELETE /myindex/mytype/myid
```
#### 文档部分更新
```
curl POST /myindex/mytype/myid/_update
{ ... }
POST /myindex/mytype/myid/_update
{
   "script" : "ctx._source.views+=1"    //脚本
}
```

#### 取回多个文档
```
curl GET /_mget
curl GET /myindex/mytype/_mget
```


#### bulk API 

#### 检索文档


#### 分析
聚合(aggregations) </br>



#### samples
- shakespeare </br>
```
curl -X PUT "localhost:9200/shakespeare" -H 'Content-Type: application/json' -d'
{
 "mappings": {
  "doc": {
   "properties": {
    "speaker": {"type": "keyword"},
    "play_name": {"type": "keyword"},
    "linemyid": {"type": "integer"},
    "speech_number": {"type": "integer"}
   }
  }
 }
}
'

curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/shakespeare/doc/_bulk?pretty' --data-binary @shakespeare_6.0.json
```


