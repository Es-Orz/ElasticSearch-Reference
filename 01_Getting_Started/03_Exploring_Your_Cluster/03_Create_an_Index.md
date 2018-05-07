## 建立一个索引

现在我们建立一个索引将它命名为`customer`并再次简单查看全部的索引：
    
    PUT /customer?pretty
    GET /_cat/indices?v

上面的命令，第一行使用PUT动词是建立名称为`customer`的索引，我们简单地在URI上追加了`pretty`参数让响应的结果可以拥有一个漂亮的格式（如果存在响应结果）。

响应结果:
    
    
    health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
    yellow open   customer 95SQ4TSUT7mWBT7VNHH67A   5   1          0            0       260b           260b

响应结果的第二行告诉我们，我们现在拥有一个索引，名字是`customer`,它有5个主分片，并且拥有1个副本（默认配置），并包含了0个文档。

你可能注意到了我们当前集群索引的是一个黄色的状态。回忆一下，黄色状态表示副本没有进行分配。原因是ES集群默认给索引设置了1个副本。当前我们只有一个节点，副本与主分片不能分配到同一个节点上（为了高可用）,当其他节点没有起来的时候。一旦副本被分配到另外的一个节点上时，ES的健康状态就会转变为绿色。