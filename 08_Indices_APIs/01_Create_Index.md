## Create Index

The create index API allows to instantiate an index. Elasticsearch provides support for multiple indices, including executing operations across several indices.

### Index Settings

Each index created can have specific settings associated with it.
    
    
    PUT twitter
    {
        "settings" : {
            "index" : {
                "number_of_shards" : 3, ![](images/icons/callouts/1.png)
                "number_of_replicas" : 2 ![](images/icons/callouts/2.png)
            }
        }
    }

![](images/icons/callouts/1.png)

| 

Default for `number_of_shards` is 5   
  
---|---  
  
![](images/icons/callouts/2.png)

| 

Default for `number_of_replicas` is 1 (ie one replica for each primary shard)   
  
The above second curl example shows how an index called `twitter` can be created with specific settings for it using [YAML](http://www.yaml.org). In this case, creating an index with 3 shards, each with 2 replicas. The index settings can also be defined with [JSON](http://www.json.org):
    
    
    PUT twitter
    {
        "settings" : {
            "index" : {
                "number_of_shards" : 3,
                "number_of_replicas" : 2
            }
        }
    }

or more simplified
    
    
    PUT twitter
    {
        "settings" : {
            "number_of_shards" : 3,
            "number_of_replicas" : 2
        }
    }

![Note](images/icons/note.png)

You do not have to explicitly specify `index` div inside the `settings` div.

For more information regarding all the different index level settings that can be set when creating an index, please check the [index modules](index-modules.html "Index Modules") div.

### Mappings

The create index API allows to provide a set of one or more mappings:
    
    
    PUT test
    {
        "settings" : {
            "number_of_shards" : 1
        },
        "mappings" : {
            "type1" : {
                "properties" : {
                    "field1" : { "type" : "text" }
                }
            }
        }
    }

### Aliases

The create index API allows also to provide a set of [aliases](indices-aliases.html "Index Aliases"):
    
    
    PUT test
    {
        "aliases" : {
            "alias_1" : {},
            "alias_2" : {
                "filter" : {
                    "term" : {"user" : "kimchy" }
                },
                "routing" : "kimchy"
            }
        }
    }

### Wait For Active Shards

By default, index creation will only return a response to the client when the primary copies of each shard have been started, or the request times out. The index creation response will indicate what happened:
    
    
    {
        "acknowledged": true,
        "shards_acknowledged": true
    }

`acknowledged` indicates whether the index was successfully created in the cluster, while `shards_acknowledged` indicates whether the requisite number of shard copies were started for each shard in the index before timing out. Note that it is still possible for either `acknowledged` or `shards_acknowledged` to be `false`, but the index creation was successful. These values simply indicate whether the operation completed before the timeout. If `acknowledged` is `false`, then we timed out before the cluster state was updated with the newly created index, but it probably will be created sometime soon. If `shards_acknowledged` is `false`, then we timed out before the requisite number of shards were started (by default just the primaries), even if the cluster state was successfully updated to reflect the newly created index (i.e. `acknowledged=true`).

We can change the default of only waiting for the primary shards to start through the index setting `index.write.wait_for_active_shards` (note that changing this setting will also affect the `wait_for_active_shards` value on all subsequent write operations):
    
    
    PUT test
    {
        "settings": {
            "index.write.wait_for_active_shards": "2"
        }
    }

or through the request parameter `wait_for_active_shards`:
    
    
    PUT test?wait_for_active_shards=2

A detailed explanation of `wait_for_active_shards` and its possible values can be found [here](docs-index_.html#index-wait-for-active-shards "Wait For Active Shardsedit").