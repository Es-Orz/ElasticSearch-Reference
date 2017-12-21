## Breaking changes in 5.4

### Delete By Query API changes

Requests without an explicit query are deprecated and will be invalid in Elasticsearch 6.0.0.

### Default settings are no longer supported

Previous versions of Elasticsearch allowed a user to set a default setting for any setting. The default setting was only applied if the actual setting was not already set. This feature was trappy, and the complexity that it introduced was prone to bugs. Due to this, we have elected to make a breaking change in a minor release to remove this feature with the exception of `default.path.conf`, `default.path.data`, and `default.path.logs` which remain to support packaging. A future version of Elasticsearch will remove support for these as well, so users should stop relying on this functionality.

### Checking if indices exist

The endpoint `/{index}` with the verb `HEAD` can be used to check if an index matching `index` exists (`index` can be a comma-delimited list of index names or index patterns). This verb behaved differently than `GET` on the same endpoint in two ways. The first is that sending the same request to `GET` and `HEAD` could in some cases return different status codes (e.g., `GET /pattern-that-has-no-matches*` would 200 with an empty response and `HEAD /pattern-that-has-no-matches*` would 404 with an empty body (and a `content-length` header of zero). The second is that any `HEAD` request on this endpoint would return a `content-length` header of zero. Both of these are violations of the HTTP specification: if `HEAD` is supported, a `HEAD` request must return the same status as the same `GET` request would, and it must return a `content-length` header that is equal to the `content-length` header as the same `GET` request would. This behavior has been addressed so that `HEAD` always returns the same status code as a `GET` would have, and with the correct `content-length`. However, this means that `HEAD /pattern-that-has-no-matches*` which served as an index existence check no longer 404s, instead 200s as if it was a `GET` request. To obtain the previous behavior, you must add the parameter `?allow_no_indices=false` (this parameter defaults to `true`).
