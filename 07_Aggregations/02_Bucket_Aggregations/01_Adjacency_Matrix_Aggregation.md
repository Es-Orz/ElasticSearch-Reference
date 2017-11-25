## Adjacency Matrix Aggregation

A bucket aggregation returning a form of [adjacency matrix](https://en.wikipedia.org/wiki/Adjacency_matrix). The request provides a collection of named filter expressions, similar to the `filters` aggregation request. Each bucket in the response represents a non-empty cell in the matrix of intersecting filters.

![Warning](images/icons/warning.png)

The `adjacency_matrix` aggregation is a new feature and we may evolve its design as we get feedback on its use. As a result, the API for this feature may change in non-backwards compatible ways 

Given filters named `A`, `B` and `C` the response would return buckets with the following names:

| A | B | C  
---|---|---|---  
  
 **A**

| 

A

| 

A&B

| 

A&C  
  
 **B**

| 

| 

B

| 

B&C  
  
 **C**

| 

| 

| 

C  
  
The intersecting buckets e.g `A&C` are labelled using a combination of the two filter names separated by the ampersand character. Note that the response does not also include a "C&A" bucket as this would be the same set of documents as "A&C". The matrix is said to be _symmetric_ so we only return half of it. To do this we sort the filter name strings and always use the lowest of a pair as the value to the left of the " &" separator.

An alternative `separator` parameter can be passed in the request if clients wish to use a separator string other than the default of the ampersand.

Example:
    
    
    PUT /emails/message/_bulk?refresh
    { "index" : { "_id" : 1 } }
    { "accounts" : ["hillary", "sidney"]}
    { "index" : { "_id" : 2 } }
    { "accounts" : ["hillary", "donald"]}
    { "index" : { "_id" : 3 } }
    { "accounts" : ["vladimir", "donald"]}
    
    GET emails/message/_search
    {
      "size": 0,
      "aggs" : {
        "interactions" : {
          "adjacency_matrix" : {
            "filters" : {
              "grpA" : { "terms" : { "accounts" : ["hillary", "sidney"] } },
              "grpB" : { "terms" : { "accounts" : ["donald", "mitt"] } },
              "grpC" : { "terms" : { "accounts" : ["vladimir", "nigel"] } }
            }
          }
        }
      }
    }

In the above example, we analyse email messages to see which groups of individuals have exchanged messages. We will get counts for each group individually and also a count of messages for pairs of groups that have recorded interactions.

Response:
    
    
    {
      "took": 9,
      "timed_out": false,
      "_shards": ...,
      "hits": ...,
      "aggregations": {
        "interactions": {
          "buckets": [
            {
              "key":"grpA",
              "doc_count": 2
            },
            {
              "key":"grpA&grpB",
              "doc_count": 1
            },
            {
              "key":"grpB",
              "doc_count": 2
            },
            {
              "key":"grpB&grpC",
              "doc_count": 1
            },
            {
              "key":"grpC",
              "doc_count": 1
            }
          ]
        }
      }
    }

### Usage

On its own this aggregation can provide all of the data required to create an undirected weighted graph. However, when used with child aggregations such as a `date_histogram` the results can provide the additional levels of data required to perform [dynamic network analysis](https://en.wikipedia.org/wiki/Dynamic_network_analysis) where examining interactions _over time_ becomes important.

### Limitations

For N filters the matrix of buckets produced can be N²/2 and so there is a default maximum imposed of 100 filters . This setting can be changed using the `index.max_adjacency_matrix_filters` index-level setting.
