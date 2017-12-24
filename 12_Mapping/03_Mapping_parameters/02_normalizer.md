## `normalizer`

The `normalizer` property of [`keyword`](keyword.html "Keyword datatype") fields is similar to [`analyzer`](analyzer.html "analyzer") except that it guarantees that the analysis chain produces a single token.

The `normalizer` is applied prior to indexing the keyword, as well as at search-time when the `keyword` field is searched via a query parser such as the [`match`](query-dsl-match-query.html "Match Query") query.
    
    
    PUT index
    {
      "settings": {
        "analysis": {
          "normalizer": {
            "my_normalizer": {
              "type": "custom",
              "char_filter": [],
              "filter": ["lowercase", "asciifolding"]
            }
          }
        }
      },
      "mappings": {
        "type": {
          "properties": {
            "foo": {
              "type": "keyword",
              "normalizer": "my_normalizer"
            }
          }
        }
      }
    }
    
    PUT index/type/1
    {
      "foo": "BÀR"
    }
    
    PUT index/type/2
    {
      "foo": "bar"
    }
    
    PUT index/type/3
    {
      "foo": "baz"
    }
    
    POST index/_refresh
    
    GET index/_search
    {
      "query": {
        "match": {
          "foo": "BAR"
        }
      }
    }

The above query matches documents 1 and 2 since `BÀR` is converted to `bar` at both index and query time.
    
    
    {
      "took": $body.took,
      "timed_out": false,
      "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
      },
      "hits": {
        "total": 2,
        "max_score": 0.2876821,
        "hits": [
          {
            "_index": "index",
            "_type": "type",
            "_id": "2",
            "_score": 0.2876821,
            "_source": {
              "foo": "bar"
            }
          },
          {
            "_index": "index",
            "_type": "type",
            "_id": "1",
            "_score": 0.2876821,
            "_source": {
              "foo": "BÀR"
            }
          }
        ]
      }
    }

Also, the fact that keywords are converted prior to indexing also means that aggregations return normalized values:
    
    
    GET index/_search
    {
      "size": 0,
      "aggs": {
        "foo_terms": {
          "terms": {
            "field": "foo"
          }
        }
      }
    }

returns
    
    
    {
      "took": 43,
      "timed_out": false,
      "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
      },
      "hits": {
        "total": 3,
        "max_score": 0.0,
        "hits": []
      },
      "aggregations": {
        "foo_terms": {
          "doc_count_error_upper_bound": 0,
          "sum_other_doc_count": 0,
          "buckets": [
            {
              "key": "bar",
              "doc_count": 2
            },
            {
              "key": "baz",
              "doc_count": 1
            }
          ]
        }
      }
    }