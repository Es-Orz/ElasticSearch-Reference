## Stats Bucket Aggregation

![Warning](/images/icons/warning.png)

This functionality is experimental and may be changed or removed completely in a future release. Elastic will take a best effort approach to fix any issues, but experimental features are not subject to the support SLA of official GA features.

A sibling pipeline aggregation which calculates a variety of stats across all bucket of a specified metric in a sibling aggregation. The specified metric must be numeric and the sibling aggregation must be a multi-bucket aggregation.

### Syntax

A `stats_bucket` aggregation looks like this in isolation:
    
    
    {
        "stats_bucket": {
            "buckets_path": "the_sum"
        }
    }

 **Table 6. `stats_bucket` Parameters**
 
Parameter Name| Description| Required| Default Value    
---|---|---|---    
`buckets_path`| The path to the buckets we wish to calculate stats for (see [`buckets_path` Syntax| Required|     
`gap_policy`| The policy to apply when gaps are found in the data (see [Dealing with gaps in the data| Optional| `skip`    
`format`| format to apply to the output value of this aggregation| Optional| `null`  
  
  


The following snippet calculates the stats for monthly `sales`:
    
    
    POST /sales/_search
    {
        "size": 0,
        "aggs" : {
            "sales_per_month" : {
                "date_histogram" : {
                    "field" : "date",
                    "interval" : "month"
                },
                "aggs": {
                    "sales": {
                        "sum": {
                            "field": "price"
                        }
                    }
                }
            },
            "stats_monthly_sales": {
                "stats_bucket": {
                    "buckets_path": "sales_per_month>sales" <1>
                }
            }
        }
    }

<1>| `bucket_paths` instructs this `stats_bucket` aggregation that we want the calculate stats for the `sales` aggregation in the `sales_per_month` date histogram.     
---|---  
  
And the following may be the response:
    
    
    {
       "took": 11,
       "timed_out": false,
       "_shards": ...,
       "hits": ...,
       "aggregations": {
          "sales_per_month": {
             "buckets": [
                {
                   "key_as_string": "2015/01/01 00:00:00",
                   "key": 1420070400000,
                   "doc_count": 3,
                   "sales": {
                      "value": 550.0
                   }
                },
                {
                   "key_as_string": "2015/02/01 00:00:00",
                   "key": 1422748800000,
                   "doc_count": 2,
                   "sales": {
                      "value": 60.0
                   }
                },
                {
                   "key_as_string": "2015/03/01 00:00:00",
                   "key": 1425168000000,
                   "doc_count": 2,
                   "sales": {
                      "value": 375.0
                   }
                }
             ]
          },
          "stats_monthly_sales": {
             "count": 3,
             "min": 60.0,
             "max": 550.0,
             "avg": 328.3333333333333,
             "sum": 985.0
          }
       }
    }
