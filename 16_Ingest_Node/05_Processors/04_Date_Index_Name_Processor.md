## Date Index Name Processor

The purpose of this processor is to point documents to the right time based index based on a date or timestamp field in a document by using the [date math index name support](date-math-index-names.html "Date math support in index names").

The processor sets the `_index` meta field with a date math index name expression based on the provided index name prefix, a date or timestamp field in the documents being processed and the provided date rounding.

First, this processor fetches the date or timestamp from a field in the document being processed. Optionally, date formatting can be configured on how the field’s value should be parsed into a date. Then this date, the provided index name prefix and the provided date rounding get formatted into a date math index name expression. Also here optionally date formatting can be specified on how the date should be formatted into a date math index name expression.

An example pipeline that points documents to a monthly index that starts with a `myindex-` prefix based on a date in the `date1` field:
    
    
    PUT _ingest/pipeline/monthlyindex
    {
      "description": "monthly date-time index naming",
      "processors" : [
        {
          "date_index_name" : {
            "field" : "date1",
            "index_name_prefix" : "myindex-",
            "date_rounding" : "M"
          }
        }
      ]
    }

Using that pipeline for an index request:
    
    
    PUT /myindex/type/1?pipeline=monthlyindex
    {
      "date1" : "2016-04-25T12:02:01.789Z"
    }
    
    
    {
      "_index" : "myindex-2016-04-01",
      "_type" : "type",
      "_id" : "1",
      "_version" : 1,
      "result" : "created",
      "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
      },
      "created" : true
    }

The above request will not index this document into the `myindex` index, but into the `myindex-2016-04-01` index because it was rounded by month. This is because the date-index-name-processor overrides the `_index` property of the document.

To see the date-math value of the index supplied in the actual index request which resulted in the above document being indexed into `myindex-2016-04-01` we can inspect the effects of the processor using a simulate request.
    
    
    POST _ingest/pipeline/_simulate
    {
      "pipeline" :
      {
        "description": "monthly date-time index naming",
        "processors" : [
          {
            "date_index_name" : {
              "field" : "date1",
              "index_name_prefix" : "myindex-",
              "date_rounding" : "M"
            }
          }
        ]
      },
      "docs": [
        {
          "_source": {
            "date1": "2016-04-25T12:02:01.789Z"
          }
        }
      ]
    }

and the result:
    
    
    {
      "docs" : [
        {
          "doc" : {
            "_id" : "_id",
            "_index" : "<myindex-{2016-04-25||/M{yyyy-MM-dd|UTC} }>",
            "_type" : "_type",
            "_source" : {
              "date1" : "2016-04-25T12:02:01.789Z"
            },
            "_ingest" : {
              "timestamp" : "2016-11-08T19:43:03.850+0000"
            }
          }
        }
      ]
    }

The above example shows that `_index` was set to `<myindex-{2016-04-25||/M{yyyy-MM-dd|UTC} }>`. Elasticsearch understands this to mean `2016-04-01` as is explained in the [date math index name documentation](date-math-index-names.html "Date math support in index names")

 **Table 17. Date index name options**

Name |  Required |  Default |  Description  
---|---|---|---  
  
`field`

| 

yes

| 

-

| 

The field to get the date or timestamp from.  
  
`index_name_prefix`

| 

no

| 

-

| 

A prefix of the index name to be prepended before the printed date.  
  
`date_rounding`

| 

yes

| 

-

| 

How to round the date when formatting the date into the index name. Valid values are: `y` (year), `M` (month), `w` (week), `d` (day), `h` (hour), `m` (minute) and `s` (second).  
  
`date_formats `

| 

no

| 

yyyy-MM-dd’T'HH:mm:ss.SSSZ

| 

An array of the expected date formats for parsing dates / timestamps in the document being preprocessed. Can be a Joda pattern or one of the following formats: ISO8601, UNIX, UNIX_MS, or TAI64N.  
  
`timezone`

| 

no

| 

UTC

| 

The timezone to use when parsing the date and when date math index supports resolves expressions into concrete index names.  
  
`locale`

| 

no

| 

ENGLISH

| 

The locale to use when parsing the date from the document being preprocessed, relevant when parsing month names or week days.  
  
`index_name_format`

| 

no

| 

yyyy-MM-dd

| 

The format to be used when printing the parsed date into the index name. An valid Joda pattern is expected here.  
  
  
