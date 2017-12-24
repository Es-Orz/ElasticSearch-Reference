## Pending cluster tasks

The pending cluster tasks API returns a list of any cluster-level changes (e.g. create index, update mapping, allocate or fail shard) which have not yet been executed.

![Note](images/icons/note.png)

This API returns a list of any pending updates to the cluster state. These are distinct from the tasks reported by the [Task Management API](tasks.html "Task Management API") which include periodic tasks and tasks initiated by the user, such as node stats, search queries, or create index requests. However, if a user-initiated task such as a create index command causes a cluster state update, the activity of this task might be reported by both task api and pending cluster tasks API.
    
    
    $ curl -XGET 'http://localhost:9200/_cluster/pending_tasks'

Usually this will return an empty list as cluster-level changes are usually fast. However if there are tasks queued up, the output will look something like this:
    
    
    {
       "tasks": [
          {
             "insert_order": 101,
             "priority": "URGENT",
             "source": "create-index [foo_9], cause [api]",
             "time_in_queue_millis": 86,
             "time_in_queue": "86ms"
          },
          {
             "insert_order": 46,
             "priority": "HIGH",
             "source": "shard-started ([foo_2][1], node[tMTocMvQQgGCkj7QDHl3OA], [P], s[INITIALIZING]), reason [after recovery from shard_store]",
             "time_in_queue_millis": 842,
             "time_in_queue": "842ms"
          },
          {
             "insert_order": 45,
             "priority": "HIGH",
             "source": "shard-started ([foo_2][0], node[tMTocMvQQgGCkj7QDHl3OA], [P], s[INITIALIZING]), reason [after recovery from shard_store]",
             "time_in_queue_millis": 858,
             "time_in_queue": "858ms"
          }
      ]
    }