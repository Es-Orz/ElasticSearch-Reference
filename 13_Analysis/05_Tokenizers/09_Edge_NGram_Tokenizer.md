## Edge NGram Tokenizer

The `edge_ngram` tokenizer first breaks text down into words whenever it encounters one of a list of specified characters, then it emits [N-grams](https://en.wikipedia.org/wiki/N-gram) of each word where the start of the N-gram is anchored to the beginning of the word.

Edge N-Grams are useful for _search-as-you-type_ queries.

![Tip](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/tip.png)

When you need _search-as-you-type_ for text which has a widely known order, such as movie or song titles, the [completion suggester](search-suggesters-completion.html) is a much more efficient choice than edge N-grams. Edge N-grams have the advantage when trying to autocomplete words that can appear in any order.

### Example output

With the default settings, the `edge_ngram` tokenizer treats the initial text as a single token and produces N-grams with minimum length `1` and maximum length `2`:
    
    
    POST _analyze
    {
      "tokenizer": "edge_ngram",
      "text": "Quick Fox"
    }

The above sentence would produce the following terms:
    
    
    [ Q, Qu ]

![Note](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/note.png)

These default gram lengths are almost entirely useless. You need to configure the `edge_ngram` before using it.

### Configuration

The `edge_ngram` tokenizer accepts the following parameters:

`min_gram`| Minimum length of characters in a gram. Defaults to `1`.     
---|---    
`max_gram`| Maximum length of characters in a gram. Defaults to `2`.     
`token_chars`| Character classes that should be included in a token. Elasticsearch will split on characters that don’t belong to the classes specified. Defaults to `[]` (keep all characters). 

Character classes may be any of the following:

  * `letter` —  for example `a`, `b`, `ï` or `京`
  * `digit` —  for example `3` or `7`
  * `whitespace` —  for example `" "` or `"\n"`
  * `punctuation` — for example `!` or `"`
  * `symbol` —  for example `$` or `√`

  
  
### Example configuration

In this example, we configure the `edge_ngram` tokenizer to treat letters and digits as tokens, and to produce grams with minimum length `2` and maximum length `10`:
    
    
    PUT my_index
    {
      "settings": {
        "analysis": {
          "analyzer": {
            "my_analyzer": {
              "tokenizer": "my_tokenizer"
            }
          },
          "tokenizer": {
            "my_tokenizer": {
              "type": "edge_ngram",
              "min_gram": 2,
              "max_gram": 10,
              "token_chars": [
                "letter",
                "digit"
              ]
            }
          }
        }
      }
    }
    
    POST my_index/_analyze
    {
      "analyzer": "my_analyzer",
      "text": "2 Quick Foxes."
    }

The above example produces the following terms:
    
    
    [ Qu, Qui, Quic, Quick, Fo, Fox, Foxe, Foxes ]

Usually we recommend using the same `analyzer` at index time and at search time. In the case of the `edge_ngram` tokenizer, the advice is different. It only makes sense to use the `edge_ngram` tokenizer at index time, to ensure that partial words are available for matching in the index. At search time, just search for the terms the user has typed in, for instance: `Quick Fo`.

Below is an example of how to set up a field for _search-as-you-type_ :
    
    
    PUT my_index
    {
      "settings": {
        "analysis": {
          "analyzer": {
            "autocomplete": {
              "tokenizer": "autocomplete",
              "filter": [
                "lowercase"
              ]
            },
            "autocomplete_search": {
              "tokenizer": "lowercase"
            }
          },
          "tokenizer": {
            "autocomplete": {
              "type": "edge_ngram",
              "min_gram": 2,
              "max_gram": 10,
              "token_chars": [
                "letter"
              ]
            }
          }
        }
      },
      "mappings": {
        "doc": {
          "properties": {
            "title": {
              "type": "text",
              "analyzer": "autocomplete",
              "search_analyzer": "autocomplete_search"
            }
          }
        }
      }
    }
    
    PUT my_index/doc/1
    {
      "title": "Quick Foxes" <1>
    }
    
    POST my_index/_refresh
    
    GET my_index/_search
    {
      "query": {
        "match": {
          "title": {
            "query": "Quick Fo", <2>
            "operator": "and"
          }
        }
      }
    }

<1>| The `autocomplete` analyzer indexes the terms `[qu, qui, quic, quick, fo, fox, foxe, foxes]`.     
---|---  
<2>| The `autocomplete_search` analyzer searches for the terms `[quick, fo]`, both of which appear in the index. 
