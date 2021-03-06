## Foreach Processor

[experimental]  This processor may change or be replaced by something else that provides similar functionality. This processor executes in its own context, which makes it different compared to all other processors and for features like verbose simulation the subprocessor isn't visible. The reason we still expose this processor, is that it is the only processor that can operate on an array 

Processes elements in an array of unknown length.

All processors can operate on elements inside an array, but if all elements of an array need to be processed in the same way, defining a processor for each element becomes cumbersome and tricky because it is likely that the number of elements in an array is unknown. For this reason the `foreach` processor exists. By specifying the field holding array elements and a processor that defines what should happen to each element, array fields can easily be preprocessed.

A processor inside the foreach processor works in the array element context and puts that in the ingest metadata under the `_ingest._value` key. If the array element is a json object it holds all immediate fields of that json object. and if the nested object is a value is `_ingest._value` just holds that value. Note that if a processor prior to the `foreach` processor used `_ingest._value` key then the specified value will not be available to the processor inside the `foreach` processor. The `foreach` processor does restore the original value, so that value is available to processors after the `foreach` processor.

Note that any other field from the document are accessible and modifiable like with all other processors. This processor just puts the current array element being read into `_ingest._value` ingest metadata attribute, so that it may be pre-processed.

If the `foreach` processor fails to process an element inside the array, and no `on_failure` processor has been specified, then it aborts the execution and leaves the array unmodified.

 **Table 19. Foreach Options**

Name |  Required |  Default |  Description  
---|---|---|---  
`field`| yes| -| The array field    
`processor`| yes| -| The processor to execute against each field    
  


Assume the following document:
    
    
    {
      "values" : ["foo", "bar", "baz"]
    }

When this `foreach` processor operates on this sample document:
    
    
    {
      "foreach" : {
        "field" : "values",
        "processor" : {
          "uppercase" : {
            "field" : "_ingest._value"
          }
        }
      }
    }

Then the document will look like this after preprocessing:
    
    
    {
      "values" : ["FOO", "BAR", "BAZ"]
    }

Let’s take a look at another example:
    
    
    {
      "persons" : [
        {
          "id" : "1",
          "name" : "John Doe"
        },
        {
          "id" : "2",
          "name" : "Jane Doe"
        }
      ]
    }

In this case, the `id` field needs to be removed, so the following `foreach` processor is used:
    
    
    {
      "foreach" : {
        "field" : "persons",
        "processor" : {
          "remove" : {
            "field" : "_ingest._value.id"
          }
        }
      }
    }

After preprocessing the result is:
    
    
    {
      "persons" : [
        {
          "name" : "John Doe"
        },
        {
          "name" : "Jane Doe"
        }
      ]
    }

The wrapped processor can have a `on_failure` definition. For example, the `id` field may not exist on all person objects. Instead of failing the index request, you can use an `on_failure` block to send the document to the _failure_index_ index for later inspection:
    
    
    {
      "foreach" : {
        "field" : "persons",
        "processor" : {
          "remove" : {
            "field" : "_value.id",
            "on_failure" : [
              {
                "set" : {
                  "field", "_index",
                  "value", "failure_index"
                }
              }
            ]
          }
        }
      }
    }

In this example, if the `remove` processor does fail, then the array elements that have been processed thus far will be updated.

Another advanced example can be found in the [attachment processor documentation](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/ingest-attachment-with-arrays.html).
