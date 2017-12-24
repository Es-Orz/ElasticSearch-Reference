## Geo Distance Range Query

[5.0.0] Deprecated in 5.0.0. Elasticsearch will continue to support geo_distance_range queries on indexes created prior to 5.0.0. Range searches on indexes created in 5.0.0 and later will no longer be supported. Distance aggregations or sorting should be used instead. 

Filters documents that exists within a range from a specific point:
    
    
    GET /_search
    {
        "query": {
            "bool" : {
                "must" : {
                    "match_all" : {}
                },
                "filter" : {
                    "geo_distance_range" : {
                        "from" : "200km",
                        "to" : "400km",
                        "pin.location" : {
                            "lat" : 40,
                            "lon" : -70
                        }
                    }
                }
            }
        }
    }

Supports the same point location parameter and query options as the [geo_distance](query-dsl-geo-distance-query.html "Geo Distance Query") filter. And also support the common parameters for range (lt, lte, gt, gte, from, to, include_upper and include_lower).

#### Ignore Unmapped

When set to `true` the `ignore_unmapped` option will ignore an unmapped field and will not match any documents for this query. This can be useful when querying multiple indexes which might have different mappings. When set to `false` (the default value) the query will throw an exception if the field is not mapped.