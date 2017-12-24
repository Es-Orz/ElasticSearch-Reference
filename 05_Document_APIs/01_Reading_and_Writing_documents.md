## Reading and Writing documents

### Introduction

Each index in Elasticsearch is [divided into shards](gs-basic-concepts.html#getting-started-shards-and-replicas "Shards & Replicasedit") and each shard can have multiple copies. These copies are known as a _replication group_ and must be kept in sync when documents are added or removed. If we fail to do so, reading from one copy will result in very different results than reading from another. The process of keeping the shard copies in sync and serving reads from them is what we call the _data replication model_.

Elasticsearch’s data replication model is based on the _primary-backup model_ and is described very well in the [PacificA paper](https://www.microsoft.com/en-us/research/publication/pacifica-replication-in-log-based-distributed-storage-systems/) of Microsoft Research. That model is based on having a single copy from the replication group that acts as the primary shard. The other copies are called _replica shards_. The primary serves as the main entry point for all indexing operations. It is in charge of validating them and making sure they are correct. Once an index operation has been accepted by the primary, the primary is also responsible for replicating the operation to the other copies.

This purpose of this div is to give a high level overview of the Elasticsearch replication model and discuss the implications it has for various interactions between write and read operations.

### Basic write model

Every indexing operation in Elasticsearch is first resolved to a replication group using [routing](docs-index_.html#index-routing "Routingedit"), typically based on the document ID. Once the replication group has been determined, the operation is forwarded internally to the current _primary shard_ of the group. The primary shard is responsible for validating the operation and forwarding it to the other replicas. Since replicas can be offline, the primary is not required to replicate to all replicas. Instead, Elasticsearch maintains a list of shard copies that should receive the operation. This list is called the _in-sync copies_ and is maintained by the master node. As the name implies, these are the set of "good" shard copies that are guaranteed to have processed all of the index and delete operations that have been acknowledged to the user. The primary is responsible for maintaining this invariant and thus has to replicate all operations to each copy in this set.

The primary shard follows this basic flow:

  1. Validate incoming operation and reject it if structurally invalid (Example: have an object field where a number is expected) 
  2. Execute the operation locally i.e. indexing or deleting the relevant document. This will also validate the content of fields and reject if needed (Example: a keyword value is too long for indexing in Lucene). 
  3. Forward the operation to each replica in the current in-sync copies set. If there are multiple replicas, this is done in parallel. 
  4. Once all replicas have successfully performed the operation and responded to the primary, the primary acknowledges the successful completion of the request to the client. 



#### Failure handling

Many things can go wrong during indexing — disks can get corrupted, nodes can be disconnected from each other, or some configuration mistake could cause an operation to fail on a replica despite it being successful on the primary. These are infrequent but the primary has to respond to them.

In the case that the primary itself fails, the node hosting the primary will send a message to the master about it. The indexing operation will wait (up to 1 minute, by [default](index-modules.html#dynamic-index-settings "Dynamic index settingsedit")) for the master to promote one of the replicas to be a new primary. The operation will then be forwarded to the new primary for processing. Note that the master also monitors the health of the nodes and may decide to proactively demote a primary. This typically happens when the node holding the primary is isolated from the cluster by a networking issue. See [here](docs-replication.html#demoted-primary) for more details.

Once the operation has been successfully performed on the primary, the primary has to deal with potential failures when executing it on the replica shards. This may be caused by an actual failure on the replica or due to a network issue preventing the operation from reaching the replica (or preventing the replica from responding). All of these share the same end result: a replica which is part of the in-sync replica set misses an operation that is about to be acknowledged. In order to avoid violating the invariant, the primary sends a message to the master requesting that the problematic shard be removed from the in-sync replica set. Only once removal of the shard has been acknowledged by the master does the primary acknowledge the operation. Note that the master will also instruct another node to start building a new shard copy in order to restore the system to a healthy state.

While forwarding an operation to the replicas, the primary will use the replicas to validate that it is still the active primary. If the primary has been isolated due to a network partition (or a long GC) it may continue to process incoming indexing operations before realising that it has been demoted. Operations that come from a stale primary will be rejected by the replicas. When the primary receives a response from the replica rejecting its request because it is no longer the primary then it will reach out to the master and will learn that it has been replaced. The operation is then routed to the new primary.

 **What happens if there are no replicas?**

This is a valid scenario that can happen due to index configuration or simply because all the replicas have failed. In that case the primary is processing operations without any external validation, which may seem problematic. On the other hand, the primary cannot fail other shards on its own but request the master to do so on its behalf. This means that the master knows that the primary is the only single good copy. We are therefore guaranteed that the master will not promote any other (out-of-date) shard copy to be a new primary and that any operation indexed into the primary will not be lost. Of course, since at that point we are running with only single copy of the data, physical hardware issues can cause data loss. See [Wait For Active Shards for some mitigation options.

### Basic read model

Reads in Elasticsearch can be very lightweight lookups by ID or a heavy search request with complex aggregations that take non-trivial CPU power. One of the beauties of the primary-backup model is that it keeps all shard copies identical (with the exception of in-flight operations). As such, a single in-sync copy is sufficient to serve read requests.

When a read request is received by a node, that node is responsible for forwarding it to the nodes that hold the relevant shards, collating the responses, and responding to the client. We call that node the _coordinating node_ for that request. The basic flow is as follows:

  1. Resolve the read requests to the relevant shards. Note that since most searches will be sent to one or more indices, they typically need to read from multiple shards, each representing a different subset of the data. 
  2. Select an active copy of each relevant shard, from the shard replication group. This can be either the primary or a replica. By default, Elasticsearch will simply round robin between the shard copies. 
  3. Send shard level read requests to the selected copies. 
  4. Combine the results and respond. Note that in the case of get by ID look up, only one shard is relevant and this step can be skipped. 



#### Failure handling

When a shard fails to respond to a read request, the coordinating node will select another copy from the same replication group and send the shard level search request to that copy instead. Repetitive failures can result in no shard copies being available. In some cases, such as `_search`, Elasticsearch will prefer to respond fast, albeit with partial results, instead of waiting for the issue to be resolved (partial results are indicated in the `_shards` header of the response).

### A few simple implications

Each of these basic flows determines how Elasticsearch behaves as a system for both reads and writes. Furthermore, since read and write requests can be executed concurrently, these two basic flows interact with each other. This has a few inherent implications:

Efficient reads 
     Under normal operation each read operation is performed once for each relevant replication group. Only under failure conditions do multiple copies of the same shard execute the same search. 
Read unacknowledged 
     Since the primary first indexes locally and then replicates the request, it is possible for a concurrent read to already see the change before it has been acknowledged. 
Two copies by default 
     This model can be fault tolerant while maintaining only two copies of the data. This is in contrast to quorum-based system where the minimum number of copies for fault tolerance is 3. 

### Failures

Under failures, the following is possible:

A single shard can slow down indexing 
     Because the primary waits for all replicas in the in-sync copies set during each operation, a single slow shard can slow down the entire replication group. This is the price we pay for the read efficiency mentioned above. Of course a single slow shard will also slow down unlucky searches that have been routed to it. 
Dirty reads 
     An isolated primary can expose writes that will not be acknowledged. This is caused by the fact that an isolated primary will only realize that it is isolated once it sends requests to its replicas or when reaching out to the master. At that point the operation is already indexed into the primary and can be read by a concurrent read. Elasticsearch mitigates this risk by pinging the master every second (by default) and rejecting indexing operations if no master is known. 

### The Tip of the Iceberg

This document provides a high level overview of how Elasticsearch deals with data. Of course, there is much much more going on under the hood. Things like primary terms, cluster state publishing and master election all play a role in keeping this system behaving correctly. This document also doesn’t cover known and important bugs (both closed and open). We recognize that [GitHub is hard to keep up with](https://github.com/elastic/elasticsearch/issues?q=label%3Aresiliency). To help people stay on top of those and we maintain a dedicated [resiliency page](https://www.elastic.co/guide/en/elasticsearch/resiliency/current/index.html) on our website. We strongly advise reading it.