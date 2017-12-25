## `analyzer`

The values of [`analyzed`](mapping-index.html) string fields are passed through an [analyzer](analysis.html) to convert the string into a stream of _tokens_ or _terms_. For instance, the string `"The quick Brown Foxes."` may, depending on which analyzer is used, be analyzed to the tokens: `quick`, `brown`, `fox`. These are the actual terms that are indexed for the field, which makes it possible to search efficiently for individual words _within_ big blobs of text.

This analysis process needs to happen not just at index time, but also at query time: the query string needs to be passed through the same (or a similar) analyzer so that the terms that it tries to find are in the same format as those that exist in the index.

Elasticsearch ships with a number of [pre-defined analyzers](analysis-analyzers.html), which can be used without further configuration. It also ships with many [character filters](analysis-charfilters.html), [tokenizers](analysis-tokenizers.html), and [_Token Filters_](analysis-tokenfilters.html) which can be combined to configure custom analyzers per index.

Analyzers can be specified per-query, per-field or per-index. At index time, Elasticsearch will look for an analyzer in this order:

  * The `analyzer` defined in the field mapping. 
  * An analyzer named `default` in the index settings. 
  * The [`standard`](analysis-standard-analyzer.html) analyzer. 



At query time, there are a few more layers:

  * The `analyzer` defined in a [full-text query](full-text-queries.html). 
  * The `search_analyzer` defined in the field mapping. 
  * The `analyzer` defined in the field mapping. 
  * An analyzer named `default_search` in the index settings. 
  * An analyzer named `default` in the index settings. 
  * The [`standard`](analysis-standard-analyzer.html) analyzer. 



The easiest way to specify an analyzer for a particular field is to define it in the field mapping, as follows:
    
    
    PUT /my_index
    {
      "mappings": {
        "my_type": {
          "properties": {
            "text": { ![](images/icons/callouts/1.png)
              "type": "text",
              "fields": {
                "english": { ![](images/icons/callouts/2.png)
                  "type":     "text",
                  "analyzer": "english"
                }
              }
            }
          }
        }
      }
    }
    
    GET my_index/_analyze ![](images/icons/callouts/3.png)
    {
      "field": "text",
      "text": "The quick Brown Foxes."
    }
    
    GET my_index/_analyze ![](images/icons/callouts/4.png)
    {
      "field": "text.english",
      "text": "The quick Brown Foxes."
    }

![](images/icons/callouts/1.png)

| 

The `text` field uses the default `standard` analyzer`.   
  
---|---  
  
![](images/icons/callouts/2.png)

| 

The `text.english` [multi-field](multi-fields.html) uses the `english` analyzer, which removes stop words and applies stemming.   
  
![](images/icons/callouts/3.png)

| 

This returns the tokens: [ `the`, `quick`, `brown`, `foxes` ].   
  
![](images/icons/callouts/4.png)

| 

This returns the tokens: [ `quick`, `brown`, `fox` ].   
  
### `search_quote_analyzer`

The `search_quote_analyzer` setting allows you to specify an analyzer for phrases, this is particularly useful when dealing with disabling stop words for phrase queries.

To disable stop words for phrases a field utilising three analyzer settings will be required:

  1. An `analyzer` setting for indexing all terms including stop words 
  2. A `search_analyzer` setting for non-phrase queries that will remove stop words 
  3. A `search_quote_analyzer` setting for phrase queries that will not remove stop words 


    
    
    PUT my_index
    {
       "settings":{
          "analysis":{
             "analyzer":{
                "my_analyzer":{ ![](images/icons/callouts/1.png)
                   "type":"custom",
                   "tokenizer":"standard",
                   "filter":[
                      "lowercase"
                   ]
                },
                "my_stop_analyzer":{ ![](images/icons/callouts/2.png)
                   "type":"custom",
                   "tokenizer":"standard",
                   "filter":[
                      "lowercase",
                      "english_stop"
                   ]
                }
             },
             "filter":{
                "english_stop":{
                   "type":"stop",
                   "stopwords":"_english_"
                }
             }
          }
       },
       "mappings":{
          "my_type":{
             "properties":{
                "title": {
                   "type":"text",
                   "analyzer":"my_analyzer", ![](images/icons/callouts/3.png)
                   "search_analyzer":"my_stop_analyzer", ![](images/icons/callouts/4.png)
                   "search_quote_analyzer":"my_analyzer" ![](images/icons/callouts/5.png)
                }
             }
          }
       }
    }
    
    
    PUT my_index/my_type/1
    {
       "title":"The Quick Brown Fox"
    }
    
    PUT my_index/my_type/2
    {
       "title":"A Quick Brown Fox"
    }
    
    GET my_index/my_type/_search
    {
       "query":{
          "query_string":{
             "query":"\"the quick brown fox\"" ![](images/icons/callouts/1.png)
          }
       }
    }

![](images/icons/callouts/1.png)

| 

`my_analyzer` analyzer which tokens all terms including stop words   
  
---|---  
  
![](images/icons/callouts/2.png)

| 

`my_stop_analyzer` analyzer which removes stop words   
  
![](images/icons/callouts/3.png)

| 

`analyzer` setting that points to the `my_analyzer` analyzer which will be used at index time   
  
![](images/icons/callouts/4.png)

| 

`search_analyzer` setting that points to the `my_stop_analyzer` and removes stop words for non-phrase queries   
  
![](images/icons/callouts/5.png)

| 

`search_quote_analyzer` setting that points to the `my_analyzer` analyzer and ensures that stop words are not removed from phrase queries   
  
![](images/icons/callouts/1.png)

| 

Since the query is wrapped in quotes it is detected as a phrase query therefore the `search_quote_analyzer` kicks in and ensures the stop words are not removed from the query. The `my_analyzer` analyzer will then return the following tokens [`the`, `quick`, `brown`, `fox`] which will match one of the documents. Meanwhile term queries will be analyzed with the `my_stop_analyzer` analyzer which will filter out stop words. So a search for either `The quick brown fox` or `A quick brown fox` will return both documents since both documents contain the following tokens [`quick`, `brown`, `fox`]. Without the `search_quote_analyzer` it would not be possible to do exact matches for phrase queries as the stop words from phrase queries would be removed resulting in both documents matching. 
