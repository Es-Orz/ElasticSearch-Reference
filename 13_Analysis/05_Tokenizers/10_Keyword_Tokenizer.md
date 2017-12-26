## Keyword Tokenizer

The `keyword` tokenizer is a “noop” tokenizer that accepts whatever text it is given and outputs the exact same text as a single term. It can be combined with token filters to normalise output, e.g. lower-casing email addresses.

### Example output
    
    
    POST _analyze
    {
      "tokenizer": "keyword",
      "text": "New York"
    }

The above sentence would produce the following term:
    
    
    [ New York ]

### Configuration

The `keyword` tokenizer accepts the following parameters:

`buffer_size`| The number of characters read into the term buffer in a single pass. Defaults to `256`. The term buffer will grow by this size until all the text has been consumed. It is advisable not to change this setting.     
---|---
