##  存储 `store`

By default, field values are [indexed](mapping-index.html) to make them searchable, but they are not _stored_. This means that the field can be queried, but the original field value cannot be retrieved.

Usually this doesn’t matter. The field value is already part of the [`_source` field](mapping-source-field.html), which is stored by default. If you only want to retrieve the value of a single field or of a few fields, instead of the whole `_source`, then this can be achieved with [source filtering](search-request-source-filtering.html).

In certain situations it can make sense to `store` a field. For instance, if you have a document with a `title`, a `date`, and a very large `content` field, you may want to retrieve just the `title` and the `date` without having to extract those fields from a large `_source` field:
    
    
    PUT my_index
    {
      "mappings": {
        "my_type": {
          "properties": {
            "title": {
              "type": "text",
              "store": true <1>
            },
            "date": {
              "type": "date",
              "store": true <2>
            },
            "content": {
              "type": "text"
            }
          }
        }
      }
    }
    
    PUT my_index/my_type/1
    {
      "title":   "Some short title",
      "date":    "2015-01-01",
      "content": "A very long content field..."
    }
    
    GET my_index/_search
    {
      "stored_fields": [ "title", "date" ] <3>
    }

<1> <2>| The `title` and `date` fields are stored.     
---|---   
 <3>| This request will retrieve the values of the `title` and `date` fields.   
  
![Note](/images/icons/note.png)

### Stored fields returned as arrays

For consistency, stored fields are always returned as an _array_ because there is no way of knowing if the original field value was a single value, multiple values, or an empty array.

If you need the original value, you should retrieve it from the `_source` field instead.

Another situation where it can make sense to make a field stored is for those that don’t appear in the `_source` field (such as [`copy_to` fields](copy-to.html)).
