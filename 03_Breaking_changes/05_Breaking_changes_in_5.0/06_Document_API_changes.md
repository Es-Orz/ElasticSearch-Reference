## 文档APIs变更 Document API changes

### `?refresh` no longer supports truthy and falsy values

The `?refresh` request parameter used to accept any value other than `false`, `0`, `off`, and `no` to mean).

### `created` field deprecated in the Index API

The `created` field has been deprecated in the Index API. It now returns `result`, returning `"result": "created"` when it created a document and `"result": "updated"` when it updated the document. This is also true for `index` bulk operations.

### `found` field deprecated in the Delete API

The `found` field has been deprecated in the Delete API. It now returns `result`, returning `"result": "deleted"` when it deleted a document and `"result": "not_found"` when it didn’t find the document. This is also true for `delete` bulk operations.

### Reindex and Update By Query

Before 5.0.0 `_reindex` and `_update_by_query` only retried bulk failures so they used the following response format:
    
    
    {
       ...
       "retries": 10
       ...
    }

Where `retries` counts the number of bulk retries. Now they retry on search failures as well and use this response format:
    
    
    {
       ...
       "retries": {
         "bulk": 10,
         "search": 1
       }
       ...
    }

Where `bulk` counts the number of bulk retries and `search` counts the number of search retries.

### get API

As of 5.0.0 the get API will issue a refresh if the requested document has been changed since the last refresh but the change hasn’t been refreshed yet. This will also make all other changes visible immediately. This can have an impact on performance if the same document is updated very frequently using a read modify update pattern since it might create many small segments. This behavior can be disabled by passing `realtime=false` to the get request.

The `fields` field has been renamed to `stored_fields`

### mget API

The `fields` field has been renamed to `stored_fields`

### update API

The `fields` field has been deprecated. You should use `_source` to load the field from _source.

### bulk API

The `fields` field has been deprecated. You should use `_source` to load the field from _source.
