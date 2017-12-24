## 集群重路由（cluster reroute）

重路由（reroute）请求允许显式执行一个集群重路由分配命令，举例：一个分片可能从一个节点移动到另外一个节点，同时一个分片还可以取消和指定没有分配的分片到一个指定的节点：

这里是一个简短的例子：
    
    
    POST /_cluster/reroute
    {
        "commands" : [
            {
                "move" : {
                    "index" : "test", 
                    "shard" : 0,
                    "from_node" : "node1",
                    "to_node" : "node2"
                }
            },
            {
              "allocate_replica" : {
                    "index" : "test", 
                    "shard" : 1,
                    "node" : "node3"
              }
            }
        ]
    }

要记住重要的一方面是：一旦发生重新路由，集群会重新分配到一个平衡的状态，例如：如果分片分配是从从`node1`到`node2，为了达到平衡状态，其他分片会从 `node2`移到`node1`.

可以禁用群集的自动分配功能，这意味着我们要手动显式地进行分片的分配任务。显然，只有所有命令执行成功，群集才会重新达到一个平衡的状态

另一个选项是运行`dry_run`(类似URI或请求休)，这会直接影响当前群集的状态，当命令执行成功之后，会返回一个命令执行后的集群状态。

如果`explain`参数被明确指定，一个更加详细的说明会在命令执行后进行返回。

重路由(reroute)请求支持的命令（commands）参数有如下:

* `move`

     从一个节点移动到另外一个节点. 
    
    * `index` 索引名称
    * `shard` 分片号,
    * `from_node` 从哪个节点上迁移
    * `to_node`  迁移到哪个节点
* `cancel`

     取消一个节点上一个分片

     * `index` 索引名称
     * `shard` 分片号
     * `node` 要取消分片所在结点
     * `allow_primary` 是否允许操作主分片,true of false 默认false,强制取消主分片，让群集重新执行标准的分配过程。

* `allocate_replica`

    分配一个还未进行分配的分片到一个群集结点，参考[群集分配决策](../14_Modules/01_Cluster.md)

    * `index` 索引名称
    * `shard` 分片号
    * `node` 分配到节点的名称

还有两个命令可用于主分片的分配，这两相命令要特别重视，ES本身会对主分片进行全自动分配，当然设置了如下的设置,ES就不会分配主分片：

  * 一个主分片被创建，但没有一个满足分配策略的节点可以进行分配 
  * 无法找到一个最新的分片的数据对当前数据集群中的节点。为了防止数据丢失,ES不会自动促进陈旧的复制分片到主分片。

下面两条命令允许强制迁移主分片：

    * `allocate_stale_primary`
    
    分配一个包含有旧数据的主分片。这命令可以会导到数据丢失。

        * `index` 索引名称
        * `shard` 分片号
        * `node` 指定分片所在结点
        * `accept_data_loss` 默认为true，可以接受数据丢失,会使用当前分片的数据覆盖群集中的主分配数据。这个参数中为了让命令更好的理解

    
    * `allocate_empty_primary`
    
    分配一个空的主分片节点
 
        * `index` 索引名称
        * `shard` 分片号
        * `node` 指定分片所在结点
        * `accept_data_loss` 默认为true，可以接受数据丢失，会使用空分片，替换集群上现在主分片的数据，这个参数中为了让命令更好的理解

### 重试失败的分片

在放弃分配一个失败的分片之前，集群会尝试`index.allocation.max_retries`(默认5)次，导致这个情况发生的原因可能是，分片使用的停用词文件不存在。

问题发生后，可以尝试[`reroute`](05_Cluster_Reroute.md "Cluster Reroute") API带`?retry_failed`参数进行手动重试分配，重试一次。