## Snapshot And Restore

The snapshot and restore module allows to create snapshots of individual indices or an entire cluster into a remote repository like shared file system, S3, or HDFS. These snapshots are great for backups because they can be restored relatively quickly but they are not archival because they can only be restored to versions of Elasticsearch that can read the index. That means that:

  * A snapshot of an index created in 2.x can be restored to 5.x. 
  * A snapshot of an index created in 1.x can be restored to 2.x. 
  * A snapshot of an index created in 1.x can **not** be restored to 5.x. 



To restore a snapshot of an index created in 1.x to 5.x you can restore it to a 2.x cluster and use [reindex-from-remote](docs-reindex.html#reindex-from-remote "Reindex from Remoteedit") to rebuild the index in a 5.x cluster. This is as time consuming as restoring from archival copies of the original data.

Note: If a repository is connected to a 2.x cluster, and you want to connect a 5.x cluster to the same repository, you will have to either first set the 2.x repository to `readonly` mode (see below for details on `readonly` mode) or create the 5.x repository in `readonly` mode. A 5.x cluster will update the repository to conform to 5.x specific formats, which will mean that any new snapshots written via the 2.x cluster will not be visible to the 5.x cluster, and vice versa. In fact, as a general rule, only one cluster should connect to the same repository location with write access; all other clusters connected to the same repository should be set to `readonly` mode. While setting all but one repositories to `readonly` should work with multiple clusters differing by one major version, it is not a supported configuration.

### Repositories

Before any snapshot or restore operation can be performed, a snapshot repository should be registered in Elasticsearch. The repository settings are repository-type specific. See below for details.
    
    
    PUT /_snapshot/my_backup
    {
      "type": "fs",
      "settings": {
            ... repository specific settings ...
      }
    }

Once a repository is registered, its information can be obtained using the following command:
    
    
    GET /_snapshot/my_backup

which returns:
    
    
    {
      "my_backup": {
        "type": "fs",
        "settings": {
          "compress": true,
          "location": "/mount/backups/my_backup"
        }
      }
    }

Information about multiple repositories can be fetched in one go by using a comma-delimited list of repository names. Star wildcards are supported as well. For example, information about repositories that start with `repo` or that contain `backup` can be obtained using the following command:
    
    
    GET /_snapshot/repo*,*backup*

If a repository name is not specified, or `_all` is used as repository name Elasticsearch will return information about all repositories currently registered in the cluster:
    
    
    GET /_snapshot

or
    
    
    GET /_snapshot/_all

##### Shared File System Repository

The shared file system repository (`"type": "fs"`) uses the shared file system to store snapshots. In order to register the shared file system repository it is necessary to mount the same shared filesystem to the same location on all master and data nodes. This location (or one of its parent directories) must be registered in the `path.repo` setting on all master and data nodes.

Assuming that the shared filesystem is mounted to `/mount/backups/my_backup`, the following setting should be added to `elasticsearch.yml` file:
    
    
    path.repo: ["/mount/backups", "/mount/longterm_backups"]

The `path.repo` setting supports Microsoft Windows UNC paths as long as at least server name and share are specified as a prefix and back slashes are properly escaped:
    
    
    path.repo: ["\\\\MY_SERVER\\Snapshots"]

After all nodes are restarted, the following command can be used to register the shared file system repository with the name `my_backup`:
    
    
    $ curl -XPUT 'http://localhost:9200/_snapshot/my_backup' -H 'Content-Type: application/json' -d '{
        "type": "fs",
        "settings": {
            "location": "/mount/backups/my_backup",
            "compress": true
        }
    }'

If the repository location is specified as a relative path this path will be resolved against the first path specified in `path.repo`:
    
    
    $ curl -XPUT 'http://localhost:9200/_snapshot/my_backup' -H 'Content-Type: application/json' -d '{
        "type": "fs",
        "settings": {
            "location": "my_backup",
            "compress": true
        }
    }'

The following settings are supported:

`location`

| 

Location of the snapshots. Mandatory.   
  
---|---  
  
`compress`

| 

Turns on compression of the snapshot files. Compression is applied only to metadata files (index mapping and settings). Data files are not compressed. Defaults to `true`.   
  
`chunk_size`

| 

Big files can be broken down into chunks during snapshotting if needed. The chunk size can be specified in bytes or by using size value notation, i.e. 1g, 10m, 5k. Defaults to `null` (unlimited chunk size).   
  
`max_restore_bytes_per_sec`

| 

Throttles per node restore rate. Defaults to `40mb` per second.   
  
`max_snapshot_bytes_per_sec`

| 

Throttles per node snapshot rate. Defaults to `40mb` per second.   
  
`readonly`

| 

Makes repository read-only. Defaults to `false`.   
  
##### Read-only URL Repository

The URL repository (`"type": "url"`) can be used as an alternative read-only way to access data created by the shared file system repository. The URL specified in the `url` parameter should point to the root of the shared filesystem repository. The following settings are supported:

`url`

| 

Location of the snapshots. Mandatory.   
  
---|---  
  
URL Repository supports the following protocols: "http", "https", "ftp", "file" and "jar". URL repositories with `http:`, `https:`, and `ftp:` URLs has to be whitelisted by specifying allowed URLs in the `repositories.url.allowed_urls` setting. This setting supports wildcards in the place of host, path, query, and fragment. For example:
    
    
    repositories.url.allowed_urls: ["http://www.example.org/root/*", "https://*.mydomain.com/*?*#*"]

URL repositories with `file:` URLs can only point to locations registered in the `path.repo` setting similar to shared file system repository.

##### Repository plugins

Other repository backends are available in these official plugins:

  * [repository-s3](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/repository-s3.html) for S3 repository support 
  * [repository-hdfs](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/repository-hdfs.html) for HDFS repository support in Hadoop environments 
  * [repository-azure](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/repository-azure.html) for Azure storage repositories 
  * [repository-gcs](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/repository-gcs.html) for Google Cloud Storage repositories 



##### Repository Verification

When a repository is registered, it’s immediately verified on all master and data nodes to make sure that it is functional on all nodes currently present in the cluster. The `verify` parameter can be used to explicitly disable the repository verification when registering or updating a repository:
    
    
    PUT /_snapshot/s3_repository?verify=false
    {
      "type": "s3",
      "settings": {
        "bucket": "my_s3_bucket",
        "region": "eu-west-1"
      }
    }

The verification process can also be executed manually by running the following command:
    
    
    POST /_snapshot/s3_repository/_verify

It returns a list of nodes where repository was successfully verified or an error message if verification process failed.

### Snapshot

A repository can contain multiple snapshots of the same cluster. Snapshots are identified by unique names within the cluster. A snapshot with the name `snapshot_1` in the repository `my_backup` can be created by executing the following command:
    
    
    PUT /_snapshot/my_backup/snapshot_1?wait_for_completion=true

The `wait_for_completion` parameter specifies whether or not the request should return immediately after snapshot initialization (default) or wait for snapshot completion. During snapshot initialization, information about all previous snapshots is loaded into the memory, which means that in large repositories it may take several seconds (or even minutes) for this command to return even if the `wait_for_completion` parameter is set to `false`.

By default a snapshot of all open and started indices in the cluster is created. This behavior can be changed by specifying the list of indices in the body of the snapshot request.
    
    
    PUT /_snapshot/my_backup/snapshot_1
    {
      "indices": "index_1,index_2",
      "ignore_unavailable": true,
      "include_global_state": false
    }

The list of indices that should be included into the snapshot can be specified using the `indices` parameter that supports [multi index syntax](search-search.html#search-multi-index-type "Multi-Index, Multi-Typeedit"). The snapshot request also supports the `ignore_unavailable` option. Setting it to `true` will cause indices that do not exist to be ignored during snapshot creation. By default, when `ignore_unavailable` option is not set and an index is missing the snapshot request will fail. By setting `include_global_state` to false it’s possible to prevent the cluster global state to be stored as part of the snapshot. By default, the entire snapshot will fail if one or more indices participating in the snapshot don’t have all primary shards available. This behaviour can be changed by setting `partial` to `true`.

The index snapshot process is incremental. In the process of making the index snapshot Elasticsearch analyses the list of the index files that are already stored in the repository and copies only files that were created or changed since the last snapshot. That allows multiple snapshots to be preserved in the repository in a compact form. Snapshotting process is executed in non-blocking fashion. All indexing and searching operation can continue to be executed against the index that is being snapshotted. However, a snapshot represents the point-in-time view of the index at the moment when snapshot was created, so no records that were added to the index after the snapshot process was started will be present in the snapshot. The snapshot process starts immediately for the primary shards that has been started and are not relocating at the moment. Before version 1.2.0, the snapshot operation fails if the cluster has any relocating or initializing primaries of indices participating in the snapshot. Starting with version 1.2.0, Elasticsearch waits for relocation or initialization of shards to complete before snapshotting them.

Besides creating a copy of each index the snapshot process can also store global cluster metadata, which includes persistent cluster settings and templates. The transient settings and registered snapshot repositories are not stored as part of the snapshot.

Only one snapshot process can be executed in the cluster at any time. While snapshot of a particular shard is being created this shard cannot be moved to another node, which can interfere with rebalancing process and allocation filtering. Elasticsearch will only be able to move a shard to another node (according to the current allocation filtering settings and rebalancing algorithm) once the snapshot is finished.

Once a snapshot is created information about this snapshot can be obtained using the following command:
    
    
    GET /_snapshot/my_backup/snapshot_1

This command returns basic information about the snapshot including start and end time, version of elasticsearch that created the snapshot, the list of included indices, the current state of the snapshot and the list of failures that occurred during the snapshot. The snapshot `state` can be

`IN_PROGRESS`

| 

The snapshot is currently running.   
  
---|---  
  
`SUCCESS`

| 

The snapshot finished and all shards were stored successfully.   
  
`FAILED`

| 

The snapshot finished with an error and failed to store any data.   
  
`PARTIAL`

| 

The global cluster state was stored, but data of at least one shard wasn’t stored successfully. The `failure` div in this case should contain more detailed information about shards that were not processed correctly.   
  
`INCOMPATIBLE`

| 

The snapshot was created with an old version of elasticsearch and therefore is incompatible with the current version of the cluster.   
  
Similar as for repositories, information about multiple snapshots can be queried in one go, supporting wildcards as well:
    
    
    GET /_snapshot/my_backup/snapshot_*,some_other_snapshot

All snapshots currently stored in the repository can be listed using the following command:
    
    
    GET /_snapshot/my_backup/_all

The command fails if some of the snapshots are unavailable. The boolean parameter `ignore_unavailable` can be used to return all snapshots that are currently available.

A currently running snapshot can be retrieved using the following command:
    
    
    $ curl -XGET "localhost:9200/_snapshot/my_backup/_current"

A snapshot can be deleted from the repository using the following command:
    
    
    DELETE /_snapshot/my_backup/snapshot_1

When a snapshot is deleted from a repository, Elasticsearch deletes all files that are associated with the deleted snapshot and not used by any other snapshots. If the deleted snapshot operation is executed while the snapshot is being created the snapshotting process will be aborted and all files created as part of the snapshotting process will be cleaned. Therefore, the delete snapshot operation can be used to cancel long running snapshot operations that were started by mistake.

A repository can be deleted using the following command:
    
    
    DELETE /_snapshot/my_backup

When a repository is deleted, Elasticsearch only removes the reference to the location where the repository is storing the snapshots. The snapshots themselves are left untouched and in place.

### Restore

A snapshot can be restored using the following command:
    
    
    POST /_snapshot/my_backup/snapshot_1/_restore

By default, all indices in the snapshot are restored, and the cluster state is **not** restored. It’s possible to select indices that should be restored as well as to allow the global cluster state from being restored by using `indices` and `include_global_state` options in the restore request body. The list of indices supports [multi index syntax](search-search.html#search-multi-index-type "Multi-Index, Multi-Typeedit"). The `rename_pattern` and `rename_replacement` options can be also used to rename indices on restore using regular expression that supports referencing the original text as explained [here](http://docs.oracle.com/javase/6/docs/api/java/util/regex/Matcher.html#appendReplacement\(java.lang.StringBuffer,%20java.lang.String\)). Set `include_aliases` to `false` to prevent aliases from being restored together with associated indices
    
    
    POST /_snapshot/my_backup/snapshot_1/_restore
    {
      "indices": "index_1,index_2",
      "ignore_unavailable": true,
      "include_global_state": true,
      "rename_pattern": "index_(.+)",
      "rename_replacement": "restored_index_$1"
    }

The restore operation can be performed on a functioning cluster. However, an existing index can be only restored if it’s [closed](indices-open-close.html "Open / Close Index API") and has the same number of shards as the index in the snapshot. The restore operation automatically opens restored indices if they were closed and creates new indices if they didn’t exist in the cluster. If cluster state is restored with `include_global_state` (defaults to `false`), the restored templates that don’t currently exist in the cluster are added and existing templates with the same name are replaced by the restored templates. The restored persistent settings are added to the existing persistent settings.

#### Partial restore

By default, the entire restore operation will fail if one or more indices participating in the operation don’t have snapshots of all shards available. It can occur if some shards failed to snapshot for example. It is still possible to restore such indices by setting `partial` to `true`. Please note, that only successfully snapshotted shards will be restored in this case and all missing shards will be recreated empty.

#### Changing index settings during restore

Most of index settings can be overridden during the restore process. For example, the following command will restore the index `index_1` without creating any replicas while switching back to default refresh interval:
    
    
    POST /_snapshot/my_backup/snapshot_1/_restore
    {
      "indices": "index_1",
      "index_settings": {
        "index.number_of_replicas": 0
      },
      "ignore_index_settings": [
        "index.refresh_interval"
      ]
    }

Please note, that some settings such as `index.number_of_shards` cannot be changed during restore operation.

#### Restoring to a different cluster

The information stored in a snapshot is not tied to a particular cluster or a cluster name. Therefore it’s possible to restore a snapshot made from one cluster into another cluster. All that is required is registering the repository containing the snapshot in the new cluster and starting the restore process. The new cluster doesn’t have to have the same size or topology. However, the version of the new cluster should be the same or newer (only 1 major version newer) than the cluster that was used to create the snapshot. For example, you can restore a 1.x snapshot to a 2.x cluster, but not a 1.x snapshot to a 5.x cluster.

If the new cluster has a smaller size additional considerations should be made. First of all it’s necessary to make sure that new cluster have enough capacity to store all indices in the snapshot. It’s possible to change indices settings during restore to reduce the number of replicas, which can help with restoring snapshots into smaller cluster. It’s also possible to select only subset of the indices using the `indices` parameter. Prior to version 1.5.0 elasticsearch didn’t check restored persistent settings making it possible to accidentally restore an incompatible `discovery.zen.minimum_master_nodes` setting, and as a result disable a smaller cluster until the required number of master eligible nodes is added. Starting with version 1.5.0 incompatible settings are ignored.

If indices in the original cluster were assigned to particular nodes using [shard allocation filtering](shard-allocation-filtering.html "Shard Allocation Filtering"), the same rules will be enforced in the new cluster. Therefore if the new cluster doesn’t contain nodes with appropriate attributes that a restored index can be allocated on, such index will not be successfully restored unless these index allocation settings are changed during restore operation.

### Snapshot status

A list of currently running snapshots with their detailed status information can be obtained using the following command:
    
    
    GET /_snapshot/_status

In this format, the command will return information about all currently running snapshots. By specifying a repository name, it’s possible to limit the results to a particular repository:
    
    
    GET /_snapshot/my_backup/_status

If both repository name and snapshot id are specified, this command will return detailed status information for the given snapshot even if it’s not currently running:
    
    
    GET /_snapshot/my_backup/snapshot_1/_status

Multiple ids are also supported:
    
    
    GET /_snapshot/my_backup/snapshot_1,snapshot_2/_status

### Monitoring snapshot/restore progress

There are several ways to monitor the progress of the snapshot and restores processes while they are running. Both operations support `wait_for_completion` parameter that would block client until the operation is completed. This is the simplest method that can be used to get notified about operation completion.

The snapshot operation can be also monitored by periodic calls to the snapshot info:
    
    
    GET /_snapshot/my_backup/snapshot_1

Please note that snapshot info operation uses the same resources and thread pool as the snapshot operation. So, executing a snapshot info operation while large shards are being snapshotted can cause the snapshot info operation to wait for available resources before returning the result. On very large shards the wait time can be significant.

To get more immediate and complete information about snapshots the snapshot status command can be used instead:
    
    
    GET /_snapshot/my_backup/snapshot_1/_status

While snapshot info method returns only basic information about the snapshot in progress, the snapshot status returns complete breakdown of the current state for each shard participating in the snapshot.

The restore process piggybacks on the standard recovery mechanism of the Elasticsearch. As a result, standard recovery monitoring services can be used to monitor the state of restore. When restore operation is executed the cluster typically goes into `red` state. It happens because the restore operation starts with "recovering" primary shards of the restored indices. During this operation the primary shards become unavailable which manifests itself in the `red` cluster state. Once recovery of primary shards is completed Elasticsearch is switching to standard replication process that creates the required number of replicas at this moment cluster switches to the `yellow` state. Once all required replicas are created, the cluster switches to the `green` states.

The cluster health operation provides only a high level status of the restore process. It’s possible to get more detailed insight into the current state of the recovery process by using [indices recovery](indices-recovery.html "Indices Recovery") and [cat recovery](cat-recovery.html "cat recovery") APIs.

### Stopping currently running snapshot and restore operations

The snapshot and restore framework allows running only one snapshot or one restore operation at a time. If a currently running snapshot was executed by mistake, or takes unusually long, it can be terminated using the snapshot delete operation. The snapshot delete operation checks if the deleted snapshot is currently running and if it does, the delete operation stops that snapshot before deleting the snapshot data from the repository.
    
    
    DELETE /_snapshot/my_backup/snapshot_1

The restore operation uses the standard shard recovery mechanism. Therefore, any currently running restore operation can be canceled by deleting indices that are being restored. Please note that data for all deleted indices will be removed from the cluster as a result of this operation.

### Effect of cluster blocks on snapshot and restore operations

Many snapshot and restore operations are affected by cluster and index blocks. For example, registering and unregistering repositories require write global metadata access. The snapshot operation requires that all indices and their metadata as well as the global metadata were readable. The restore operation requires the global metadata to be writable, however the index level blocks are ignored during restore because indices are essentially recreated during restore. Please note that a repository content is not part of the cluster and therefore cluster blocks don’t affect internal repository operations such as listing or deleting snapshots from an already registered repository.