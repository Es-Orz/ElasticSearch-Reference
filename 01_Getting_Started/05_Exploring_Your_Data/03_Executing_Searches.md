## Executing Searches

Now that we have seen a few of the basic search parameters, let’s dig in some more into the Query DSL. Let’s first take a look at the returned document fields. By default, the full JSON document is returned as part of all searches. This is referred to as the source (`_source` field in the search hits). If we don’t want the entire source document returned, we have the ability to request only a few fields from within source to be returned.

This example shows how to return two fields, `account_number` and `balance` (inside of `_source`), from the search:
    
    
    GET /bank/_search
    {
      "query": { "match_all": {} },
      "_source": ["account_number", "balance"]
    }

Note that the above example simply reduces the `_source` field. It will still only return one field named `_source` but within it, only the fields `account_number` and `balance` are included.

If you come from a SQL background, the above is somewhat similar in concept to the `SQL SELECT FROM` field list.

Now let’s move on to the query part. Previously, we’ve seen how the `match_all` query is used to match all documents. Let’s now introduce a new query called the [`match` query](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/query-dsl-match-query.html), which can be thought of as a basic fielded search query (i.e. a search done against a specific field or set of fields).

This example returns the account numbered 20:
    
    
    GET /bank/_search
    {
      "query": { "match": { "account_number": 20 } }
    }

This example returns all accounts containing the term "mill" in the address:
    
    
    GET /bank/_search
    {
      "query": { "match": { "address": "mill" } }
    }

This example returns all accounts containing the term "mill" or "lane" in the address:
    
    
    GET /bank/_search
    {
      "query": { "match": { "address": "mill lane" } }
    }

This example is a variant of `match` (`match_phrase`) that returns all accounts containing the phrase "mill lane" in the address:
    
    
    GET /bank/_search
    {
      "query": { "match_phrase": { "address": "mill lane" } }
    }

Let’s now introduce the [`bool`(Boolean) query](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/query-dsl-bool-query.html). The `bool` query allows us to compose smaller queries into bigger queries using boolean logic.

This example composes two `match` queries and returns all accounts containing "mill" and "lane" in the address:
    
    
    GET /bank/_search
    {
      "query": {
        "bool": {
          "must": [
            { "match": { "address": "mill" } },
            { "match": { "address": "lane" } }
          ]
        }
      }
    }

In the above example, the `bool must` clause specifies all the queries that must be true for a document to be considered a match.

In contrast, this example composes two `match` queries and returns all accounts containing "mill" or "lane" in the address:
    
    
    GET /bank/_search
    {
      "query": {
        "bool": {
          "should": [
            { "match": { "address": "mill" } },
            { "match": { "address": "lane" } }
          ]
        }
      }
    }

In the above example, the `bool should` clause specifies a list of queries either of which must be true for a document to be considered a match.

This example composes two `match` queries and returns all accounts that contain neither "mill" nor "lane" in the address:
    
    
    GET /bank/_search
    {
      "query": {
        "bool": {
          "must_not": [
            { "match": { "address": "mill" } },
            { "match": { "address": "lane" } }
          ]
        }
      }
    }

In the above example, the `bool must_not` clause specifies a list of queries none of which must be true for a document to be considered a match.

We can combine `must`, `should`, and `must_not` clauses simultaneously inside a `bool` query. Furthermore, we can compose `bool` queries inside any of these `bool` clauses to mimic any complex multi-level boolean logic.

This example returns all accounts of anybody who is 40 years old but doesn’t live in ID(aho):
    
    
    GET /bank/_search
    {
      "query": {
        "bool": {
          "must": [
            { "match": { "age": "40" } }
          ],
          "must_not": [
            { "match": { "state": "ID" } }
          ]
        }
      }
    }
