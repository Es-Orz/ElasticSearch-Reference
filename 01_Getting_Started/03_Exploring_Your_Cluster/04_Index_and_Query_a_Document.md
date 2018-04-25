## 索引并查询一个文档

现在让我们放点内容到我们名称为`customer`索引中，回忆前面的内容，要建立一个文档，我们必须告诉ES文档的类型是什么。

我们索引一个简单的`customer`文档，它的类型是`external`,并主键（ID）为1，如下：
    
    
    PUT /customer/external/1?pretty
    {
      "name": "John Doe"
    }

响应:
    
    
    {
      "_index" : "customer",
      "_type" : "external",
      "_id" : "1",
      "_version" : 1,
      "result" : "created",
      "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
      },
      "created" : true
    }

从上面的内容，我们得知文档被成功建立。文档拥有一个内部的主键（id）为1，这是我们显示地指定的。

ES不要求你在索引一个文档的时候，先建立了一个索引，在上面的例子中，ES中如果不存在`customer`索引，ES会自动地创建一个名字为`customer`的索引。

我们现在获取一个我们刚刚索引的文档：
        
    GET /customer/external/1?pretty

响应如下：
    
    {
      "_index" : "customer",
      "_type" : "external",
      "_id" : "1",
      "_version" : 1,
      "found" : true,
      "_source" : { "name": "John Doe" }
    }

这里除了`found`字段表示了我们找到了一个主键（ID）为1有文档，还有其他字段`_source`字段的内容是我们在上一步中索引文档的全部的JSON内容
