## cat recovery

The `recovery` command is a view of index shard recoveries, both on-going and previously completed. It is a more compact view of the JSON [recovery](indices-recovery.html) API.

A recovery event occurs anytime an index shard moves to a different node in the cluster. This can happen during a snapshot recovery, a change in replication level, node failure, or on node startup. This last type is called a local store recovery and is the normal way for shards to be loaded from disk when a node starts up.

As an example, here is what the recovery state of a cluster may look like when there are no shards in transit from one node to another:
    
    
    GET _cat/recovery?v

The response of this request will be something like:
    
    
    index   shard time type  stage source_host source_node target_host target_node repository snapshot files files_recovered files_percent files_total bytes bytes_recovered bytes_percent bytes_total translog_ops translog_ops_recovered translog_ops_percent
    twitter 0     13ms store done  n/a         n/a         node0       node-0      n/a        n/a      0     0               100%          13          0     0               100%          9928        0            0                      100.0%

In the above case, the source and target nodes are the same because the recovery type was store, i.e. they were read from local storage on node start.

Now let’s see what a live recovery looks like. By increasing the replica count of our index and bringing another node online to host the replicas, we can see what a live shard recovery looks like.
    
    
    GET _cat/recovery?v&h=i,s,t,ty,st,shost,thost,f,fp,b,bp

This will return a line like:
    
    
    i       s t      ty   st    shost       thost       f     fp      b bp
    twitter 0 1252ms peer done  192.168.1.1 192.168.1.2 0     100.0%  0 100.0%

We can see in the above listing that our thw twitter shard was recovered from another node. Notice that the recovery type is shown as `peer`. The files and bytes copied are real-time measurements.

Finally, let’s see what a snapshot recovery looks like. Assuming I have previously made a backup of my index, I can restore it using the [snapshot and restore](modules-snapshots.html) API.
    
    
    GET _cat/recovery?v&h=i,s,t,ty,st,rep,snap,f,fp,b,bp

This will show a recovery of type snapshot in the response
    
    
    i       s t      ty       st    rep     snap   f  fp   b     bp
    twitter 0 1978ms snapshot done  twitter snap_1 79 8.0% 12086 9.0%
