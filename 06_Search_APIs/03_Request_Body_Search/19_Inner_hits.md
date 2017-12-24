## Inner hits

The [parent/child](mapping-parent-field.html "_parent field") and [nested](nested.html "Nested datatype") features allow the return of documents that have matches in a different scope. In the parent/child case, parent documents are returned based on matches in child documents or child documents are returned based on matches in parent documents. In the nested case, documents are returned based on matches in nested inner objects.

In both cases, the actual matches in the different scopes that caused a document to be returned is hidden. In many cases, it’s very useful to know which inner nested objects (in the case of nested) or children/parent documents (in the case of parent/child) caused certain information to be returned. The inner hits feature can be used for this. This feature returns per search hit in the search response additional nested hits that caused a search hit to match in a different scope.

Inner hits can be used by defining an `inner_hits` definition on a `nested`, `has_child` or `has_parent` query and filter. The structure looks like this:
    
    
    "<query>" : {
        "inner_hits" : {
            <inner_hits_options>
        }
    }

If `inner_hits` is defined on a query that supports it then each search hit will contain an `inner_hits` json object with the following structure:
    
    
    "hits": [
         {
            "_index": ...,
            "_type": ...,
            "_id": ...,
            "inner_hits": {
               "<inner_hits_name>": {
                  "hits": {
                     "total": ...,
                     "hits": [
                        {
                           "_type": ...,
                           "_id": ...,
                           ...
                        },
                        ...
                     ]
                  }
               }
            },
            ...
         },
         ...
    ]

### Options

Inner hits support the following options:

`from`

| 

The offset from where the first hit to fetch for each `inner_hits` in the returned regular search hits.   
  
---|---  
  
`size`

| 

The maximum number of hits to return per `inner_hits`. By default the top three matching hits are returned.   
  
`sort`

| 

How the inner hits should be sorted per `inner_hits`. By default the hits are sorted by the score.   
  
`name`

| 

The name to be used for the particular inner hit definition in the response. Useful when multiple inner hits have been defined in a single search request. The default depends in which query the inner hit is defined. For `has_child` query and filter this is the child type, `has_parent` query and filter this is the parent type and the nested query and filter this is the nested path.   
  
Inner hits also supports the following per document features:

  * [Highlighting](search-request-highlighting.html "Highlighting")
  * [Explain](search-request-explain.html "Explain")
  * [Source filtering](search-request-source-filtering.html "Source filtering")
  * [Script fields](search-request-script-fields.html "Script Fields")
  * [Doc value fields](search-request-docvalue-fields.html "Doc value Fields")
  * [Include versions](search-request-version.html "Version")



### Nested inner hits

The nested `inner_hits` can be used to include nested inner objects as inner hits to a search hit.

The example below assumes that there is a nested object field defined with the name `comments`:
    
    
    {
        "query" : {
            "nested" : {
                "path" : "comments",
                "query" : {
                    "match" : {"comments.message" : "[actual query]"}
                },
                "inner_hits" : {} ![](images/icons/callouts/1.png)
            }
        }
    }

![](images/icons/callouts/1.png)

| 

The inner hit definition in the nested query. No other options need to be defined.   
  
---|---  
  
An example of a response snippet that could be generated from the above search request:
    
    
    ...
    "hits": {
      ...
      "hits": [
         {
            "_index": "my-index",
            "_type": "question",
            "_id": "1",
            "_source": ...,
            "inner_hits": {
               "comments": { ![](images/icons/callouts/1.png)
                  "hits": {
                     "total": ...,
                     "hits": [
                        {
                           "_nested": {
                              "field": "comments",
                              "offset": 2
                           },
                           "_source": ...
                        },
                        ...
                     ]
                  }
               }
            }
         },
         ...

![](images/icons/callouts/1.png)

| 

The name used in the inner hit definition in the search request. A custom key can be used via the `name` option.   
  
---|---  
  
The `_nested` metadata is crucial in the above example, because it defines from what inner nested object this inner hit came from. The `field` defines the object array field the nested hit is from and the `offset` relative to its location in the `_source`. Due to sorting and scoring the actual location of the hit objects in the `inner_hits` is usually different than the location a nested inner object was defined.

By default the `_source` is returned also for the hit objects in `inner_hits`, but this can be changed. Either via `_source` filtering feature part of the source can be returned or be disabled. If stored fields are defined on the nested level these can also be returned via the `fields` feature.

An important default is that the `_source` returned in hits inside `inner_hits` is relative to the `_nested` metadata. So in the above example only the comment part is returned per nested hit and not the entire source of the top level document that contained the comment.

### Nested inner hits and _source

Nested document don’t have a ‘_source` field, because the entire source of document is stored with the root document under its `_source` field. To include the source of just the nested document, the source of the root document is parsed and just the relevant bit for the nested document is included as source in the inner hit. Doing this for each matching nested document has an impact on the time it takes to execute the entire search request, especially when `size` and the inner hits’ `size` are set higher than the default. To avoid the relatively expensive source extraction for nested inner hits, one can disable including the source and solely rely on stored fields.

Enabled stored field for fields under the nested object field in your mapping:
    
    
    {
        "properties": {
            "comment": {
              "type": "comments",
              "properties" : {
                "message" : {
                    "type" : "text",
                    "store" : true
                }
              }
            }
        }
    }

Disable including source and include specific stored fields in the inner hits definition:
    
    
    {
        "query" : {
            "nested" : {
                "path" : "comments",
                "query" : {
                    "match" : {"comments.message" : "[actual query]"}
                },
                "inner_hits" : {
                    "_source" : false,
                    "stored_fields" : ["comments.text"]
                }
            }
        }
    }

### Hierarchical levels of nested object fields and inner hits.

If a mapping has multiple levels of hierarchical nested object fields each level can be accessed via dot notated path. For example if there is a `comments` nested field that contains a `votes` nested field and votes should directly be returned with the root hits then the following path can be defined:
    
    
    {
       "query" : {
          "nested" : {
             "path" : "comments.votes",
             "query" : { ... },
             "inner_hits" : {}
          }
        }
    }

This indirect referencing is only supported for nested inner hits.

### Parent/child inner hits

The parent/child `inner_hits` can be used to include parent or child

The examples below assumes that there is a `_parent` field mapping in the `comment` type:
    
    
    {
        "query" : {
            "has_child" : {
                "type" : "comment",
                "query" : {
                    "match" : {"message" : "[actual query]"}
                },
                "inner_hits" : {} ![](images/icons/callouts/1.png)
            }
        }
    }

![](images/icons/callouts/1.png)

| 

The inner hit definition like in the nested example.   
  
---|---  
  
An example of a response snippet that could be generated from the above search request:
    
    
    ...
    "hits": {
      ...
      "hits": [
         {
            "_index": "my-index",
            "_type": "question",
            "_id": "1",
            "_source": ...,
            "inner_hits": {
               "comment": {
                  "hits": {
                     "total": ...,
                     "hits": [
                        {
                           "_type": "comment",
                           "_id": "5",
                           "_source": ...
                        },
                        ...
                     ]
                  }
               }
            }
         },
         ...