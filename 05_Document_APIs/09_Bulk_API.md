## Bulk API

The bulk API makes it possible to perform many index/delete operations in a single API call. This can greatly increase the indexing speed.

 **Client support for bulk requests**

Some of the officially supported clients provide helpers to assist with bulk requests and reindexing of documents from one index to another:

Perl 
     See [Search::Elasticsearch::Client::5_0::Bulk](https://metacpan.org/pod/Search::Elasticsearch::Client::5_0::Bulk) and [Search::Elasticsearch::Client::5_0::Scroll](https://metacpan.org/pod/Search::Elasticsearch::Client::5_0::Scroll)
Python 
     See [elasticsearch.helpers.*](http://elasticsearch-py.readthedocs.org/en/master/helpers.html)

The REST API endpoint is `/_bulk`, and it expects the following newline delimited JSON (NDJSON) structure:
    
    
    action_and_meta_data\n
    optional_source\n
    action_and_meta_data\n
    optional_source\n
    ....
    action_and_meta_data\n
    optional_source\n

 **NOTE** : the final line of data must end with a newline character `\n`. Each newline character may be preceded by a carriage return `\r`. When sending requests to this endpoint the `Content-Type` header should be set to `application/x-ndjson`.

The possible actions are `index`, `create`, `delete` and `update`. `index` and `create` expect a source on the next line, and have the same semantics as the `op_type` parameter to the standard index API (i.e. create will fail if a document with the same index and type exists already, whereas index will add or replace a document as necessary). `delete` does not expect a source on the following line, and has the same semantics as the standard delete API. `update` expects that the partial doc, upsert and script and its options are specified on the next line.

If you’re providing text file input to `curl`, you **must** use the `--data-binary` flag instead of plain `-d`. The latter doesn’t preserve newlines. Example:
    
    
    $ cat requests
    { "index" : { "_index" : "test", "_type" : "type1", "_id" : "1" } }
    { "field1" : "value1" }
    $ curl -s -H "Content-Type: application/x-ndjson" -XPOST localhost:9200/_bulk --data-binary "@requests"; echo
    {"took":7, "errors": false, "items":[{"index":{"_index":"test","_type":"type1","_id":"1","_version":1,"result":"created","forced_refresh":false} }]}

Because this format uses literal `\n`'s as delimiters, please be sure that the JSON actions and sources are not pretty printed. Here is an example of a correct sequence of bulk commands:
    
    
    POST _bulk
    { "index" : { "_index" : "test", "_type" : "type1", "_id" : "1" } }
    { "field1" : "value1" }
    { "delete" : { "_index" : "test", "_type" : "type1", "_id" : "2" } }
    { "create" : { "_index" : "test", "_type" : "type1", "_id" : "3" } }
    { "field1" : "value3" }
    { "update" : {"_id" : "1", "_type" : "type1", "_index" : "test"} }
    { "doc" : {"field2" : "value2"} }

The result of this bulk operation is:
    
    
    {
       "took": 30,
       "errors": false,
       "items": [
          {
             "index": {
                "_index": "test",
                "_type": "type1",
                "_id": "1",
                "_version": 1,
                "result": "created",
                "_shards": {
                   "total": 2,
                   "successful": 1,
                   "failed": 0
                },
                "created": true,
                "status": 201
             }
          },
          {
             "delete": {
                "found": false,
                "_index": "test",
                "_type": "type1",
                "_id": "2",
                "_version": 1,
                "result": "not_found",
                "_shards": {
                   "total": 2,
                   "successful": 1,
                   "failed": 0
                },
                "status": 404
             }
          },
          {
             "create": {
                "_index": "test",
                "_type": "type1",
                "_id": "3",
                "_version": 1,
                "result": "created",
                "_shards": {
                   "total": 2,
                   "successful": 1,
                   "failed": 0
                },
                "created": true,
                "status": 201
             }
          },
          {
             "update": {
                "_index": "test",
                "_type": "type1",
                "_id": "1",
                "_version": 2,
                "result": "updated",
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "status": 200
             }
          }
       ]
    }

The endpoints are `/_bulk`, `/{index}/_bulk`, and `{index}/{type}/_bulk`. When the index or the index/type are provided, they will be used by default on bulk items that don’t provide them explicitly.

A note on the format. The idea here is to make processing of this as fast as possible. As some of the actions will be redirected to other shards on other nodes, only `action_meta_data` is parsed on the receiving node side.

Client libraries using this protocol should try and strive to do something similar on the client side, and reduce buffering as much as possible.

The response to a bulk action is a large JSON structure with the individual results of each action that was performed. The failure of a single action does not affect the remaining actions.

There is no "correct" number of actions to perform in a single bulk call. You should experiment with different settings to find the optimum size for your particular workload.

If using the HTTP API, make sure that the client does not send HTTP chunks, as this will slow things down.

### Versioning

Each bulk item can include the version value using the `_version`/`version` field. It automatically follows the behavior of the index / delete operation based on the `_version` mapping. It also support the `version_type`/`_version_type` (see [versioning](docs-index_.html#index-versioning "Versioningedit"))

### Routing

Each bulk item can include the routing value using the `_routing`/`routing` field. It automatically follows the behavior of the index / delete operation based on the `_routing` mapping.

### Parent

Each bulk item can include the parent value using the `_parent`/`parent` field. It automatically follows the behavior of the index / delete operation based on the `_parent` / `_routing` mapping.

### Wait For Active Shards

When making bulk calls, you can set the `wait_for_active_shards` parameter to require a minimum number of shard copies to be active before starting to process the bulk request. See [here](docs-index_.html#index-wait-for-active-shards "Wait For Active Shardsedit") for further details and a usage example.

### Refresh

Control when the changes made by this request are visible to search. See [refresh](docs-refresh.html "?refresh").

### Update

When using `update` action `_retry_on_conflict` can be used as field in the action itself (not in the extra payload line), to specify how many times an update should be retried in the case of a version conflict.

The `update` action payload, supports the following options: `doc` (partial document), `upsert`, `doc_as_upsert`, `script` and `_source`. See [update](docs-update.html "Update API") documentation for details on the options. Example with update actions:
    
    
    POST _bulk
    { "update" : {"_id" : "1", "_type" : "type1", "_index" : "index1", "_retry_on_conflict" : 3} }
    { "doc" : {"field" : "value"} }
    { "update" : { "_id" : "0", "_type" : "type1", "_index" : "index1", "_retry_on_conflict" : 3} }
    { "script" : { "inline": "ctx._source.counter += params.param1", "lang" : "painless", "params" : {"param1" : 1} }, "upsert" : {"counter" : 1} }
    { "update" : {"_id" : "2", "_type" : "type1", "_index" : "index1", "_retry_on_conflict" : 3} }
    { "doc" : {"field" : "value"}, "doc_as_upsert" : true }
    { "update" : {"_id" : "3", "_type" : "type1", "_index" : "index1", "_source" : true} }
    { "doc" : {"field" : "value"} }
    { "update" : {"_id" : "4", "_type" : "type1", "_index" : "index1"} }
    { "doc" : {"field" : "value"}, "_source": true}

### Security

See [_URL-based access control_](url-access-control.html "URL-based access control")