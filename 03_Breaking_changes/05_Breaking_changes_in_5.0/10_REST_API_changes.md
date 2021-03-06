##  RESTFul API 变更 REST API changes

### Strict REST query string parameter parsing

Previous versions of Elasticsearch ignored unrecognized URL query string parameters. This means that extraneous parameters or parameters containing typographical errors would be silently accepted by Elasticsearch. This is dangerous from an end-user perspective because it means a submitted request will silently execute not as intended. This leniency has been removed and Elasticsearch will now fail any request that contains unrecognized query string parameters.

### id values longer than 512 bytes are rejected

When specifying an `_id` value longer than 512 bytes, the request will be rejected.

### `/_optimize` endpoint removed

The deprecated `/_optimize` endpoint has been removed. The `/_forcemerge` endpoint should be used in lieu of optimize.

The `GET` HTTP verb for `/_forcemerge` is no longer supported, please use the `POST` HTTP verb.

### Index creation endpoint only accepts `PUT`

It used to be possible to create an index by either calling `PUT index_name` or `POST index_name`. Only the former is now supported.

### `HEAD {index}/{type}` replaced with `HEAD {index}/_mapping/{type}`

The endpoint for checking whether a type exists has been changed from `{index}/{type}` to `{index}/_mapping/{type}` in order to prepare for the removal of types when `HEAD {index}/{id}` will be used to check whether a document exists in an index. The old endpoint will keep working until 6.0.

### Removed `mem` div from `/_cluster/stats` response

The `mem` div contained only the `total` value, which was actually the memory available throughout all nodes in the cluster. The div contains now `total`, `free`, `used`, `used_percent` and `free_percent`.

### Revised node roles aggregate returned by `/_cluster/stats`

The `client`, `master_only`, `data_only` and `master_data` fields have been removed in favor of `master`, `data`, `ingest` and `coordinating_only`. A node can contribute to multiple counts as it can have multiple roles. Every node is implicitly a coordinating node, so whenever a node has no explicit roles, it will be counted as coordinating only.

### Removed shard `version` information from `/_cluster/state` routing table

We now store allocation id’s of shards in the cluster state and use that to select primary shards instead of the version information.

### Node roles are not part of node attributes anymore

Node roles are now returned in a specific div, called `roles`, as part of nodes stats and nodes info response. The new div is an array that holds all the different roles that each node fulfills. In case the array is returned empty, that means that the node is a coordinating only node.

### Forbid unquoted JSON

Previously, JSON documents were allowed with unquoted field names, which isn’t strictly JSON and broke some Elasticsearch clients. If documents were already indexed with unquoted fields in a previous version of Elasticsearch, some operations may throw errors. To accompany this, a commented out JVM option has been added to the `jvm.options` file: `-Delasticsearch.json.allow_unquoted_field_names`.

Note that this option is provided solely for migration purposes and will be removed in Elasticsearch 6.0.0.

### Analyze API changes

The `filters` and `char_filters` parameters have been renamed `filter` and `char_filter`. The `token_filters` parameter has been removed. Use `filter` instead.

### `DELETE /_query` endpoint removed

The `DELETE /_query` endpoint provided by the Delete-By-Query plugin has been removed and replaced by the [Delete By Query API](docs-delete-by-query.html).

### Create stored script endpoint removed

The `PUT /_scripts/{lang}/{id}/_create` endpoint that previously allowed to create indexed scripts has been removed. Indexed scripts have been replaced by [stored scripts](modules-scripting-using.html#modules-scripting-stored-scripts).

### Create stored template endpoint removed

The `PUT /_search/template/{id}/_create` endpoint that previously allowed to create indexed template has been removed. Indexed templates have been replaced by [Pre-registered templates](search-template.html#pre-registered-templates).

### Remove properties support

Some REST endpoints (e.g., cluster update index settings) supported detecting content in the Java properties format (line-delimited key=value pairs). This support has been removed.

### `wait_for_relocating_shards` is now `wait_for_no_relocating_shards` in `/_cluster/health`

The `wait_for_relocating_shards` parameter that used to take a number is now simply a boolean flag `wait_for_no_relocating_shards`, which if set to true, means the request will wait (up until the configured timeout) for the cluster to have no shard relocations before returning. Defaults to false, which means the operation will not wait.
