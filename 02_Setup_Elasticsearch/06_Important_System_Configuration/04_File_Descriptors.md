## File Descriptors

![Note](images/icons/note.png)

This is only relevant for Linux and macOS and can be safely ignored if running Elasticsearch on Windows. On Windows that JVM uses an [API](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363858\(v=vs.85\).aspx) limited only by available resources.

Elasticsearch uses a lot of file descriptors or file handles. Running out of file descriptors can be disastrous and will most probably lead to data loss. Make sure to increase the limit on the number of open files descriptors for the user running Elasticsearch to 65,536 or higher.

For the `.zip` and `.tar.gz` packages, set [`ulimit -n 65536`](setting-system-settings.html#ulimit "ulimit") as root before starting Elasticsearch, or set `nofile` to `65536` in [`/etc/security/limits.conf`](setting-system-settings.html#limits.conf "/etc/security/limits.conf").

RPM and Debian packages already default the maximum number of file descriptors to 65536 and do not require further configuration.

You can check the `max_file_descriptors` configured for each node using the [_Nodes Stats_](cluster-nodes-stats.html "Nodes Stats") API, with:
    
    
    GET _nodes/stats/process?filter_path=**.max_file_descriptors