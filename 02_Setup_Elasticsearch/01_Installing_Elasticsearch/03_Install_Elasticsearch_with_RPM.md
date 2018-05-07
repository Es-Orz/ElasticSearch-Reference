## 使用RPM安装包进行安装 Elasticsearch
RPM 安装包可以在[官网](rpm.html#install-rpm) 或 [RPM 仓库](rpm.html#rpm-repo) 进行下载. 可以在RPM系列的系统,如OpenSuSE,CentOS,SLES,Red Hat,和Oracle Enterprise等上进行安装.

![Note](/images/icons/note.png)

RPM 安装方式,不再支持老版本的RPM系列系统,如 SLES 11 和 CentOS 5.如果想在这些版本上进行安装请查看 [使用 `.zip` 或 `.tar.gz` 进行安装ES].

最新版本的ES可以在[这里](https://www.elastic.co//downloads/elasticsearch)进行下载, 其他的版本可到[过去版本页面](https://www.elastic.co//downloads/past-releases)进行下载.

![Note](/images/icons/note.png)

ES 要求 Java 8 或之后的版. 可以使用 [oracle 官风发行版本](http://www.oracle.com/technetwork/java/javase/downloads/index.html),也可以使用开源的版本,如[OpenJDK](http://openjdk.java.net).

### 导入 ES 的 PGP 密钥

我们使用了密钥对ES进行了签名,其密钥指纹接下,可以在 https://pgp.mit.edu 找到    
    
    4609 5ACC 8548 582C 1A26 99A9 D27D 666C D88E 42B4

下载并安装密钥:
    
    rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

### 从 RPM 仓库进行安装

在 Red Hat 系列的系统上在 `/etc/yum/repos.d/` 文件夹下 创建一个名为 `elasticseach.repo` 文件,或者在 `OpenSuSE` 系列的系统中的 `/etc/zypp/repos.d/` 目录下进行创建该文件. 其内容如下:
    
    [elasticsearch-5.x]
    name=Elasticsearch repository for 5.x packages
    baseurl=https://artifacts.elastic.co/packages/5.x/yum
    gpgcheck=1
    gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
    enabled=1
    autorefresh=1
    type=rpm-md

当 RPM 仓库配置好之后,就可以使用如下的命令进行安装 ES:
    
    sudo yum install elasticsearch <1>
    sudo dnf install elasticsearch <2>
    sudo zypper install elasticsearch <3>

<1>|在 CentOS 或者 老版本的 Red Hat 发行版中使用 `yum` 进行安装 
---|---    
<2>|在 Fedora 系统和新的 Red Hat 发行版,使用 `dnf` 进行安装 
<3>|在 OpenSuSE 发行版中,使用 `zypper` 进行安装
  
### Download and install the RPM manually

手动进行下载和安装的命令如下:    
    
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.4.3.rpm
    sha1sum elasticsearch-5.4.3.rpm <1>
    sudo rpm --install elasticsearch-5.4.3.rpm

<1>| 使用 [published SHA](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.4.3.rpm.sha1).进行校验 `sha1sum` 或 `shasum` 
---|---  
  
![Note](/images/icons/note.png)

在 `systemd` 的发行版系统中,安装脚本会尝试去设置内核参数(如:`vm.max_map_count`),你可以通过设置环境变量`ES_SKIP_SET_KERNEL_PARAMETERS=true`去忽略这个设置.

### SysV `init` vs `systemd`

ES 在安装后并不会进行自动启动. 如何去启动ES,这个取决于你的系统版本,是使用 `SysV init` 还是 `systemd`(新的发行版),你可以运行如下的命令的进行辨别系统:
    
    ps -p 1

### 使用 `SysV init` 运行 ES 

使用 `chkconfig` 命令进行配置ES当系统启动的时候进行自动启动    
    
    sudo chkconfig --add elasticsearch

ES 可以通过 `service` 命令进行启动和停止
    
    sudo -i service elasticsearch start
    sudo -i service elasticsearch stop

如果 ES 启动失败,它会打印标准输出,并记录日志在 `/var/log/elasticsearch/`.

### 使用 `systemd` 运行 ES

配置ES当系统启动的时候进行自动启动     
    
    sudo /bin/systemctl daemon-reload
    sudo /bin/systemctl enable elasticsearch.service

ES 可以通过如下的命令进行启动和暂停    
    
    sudo systemctl start elasticsearch.service
    sudo systemctl stop elasticsearch.service

这些命令并没有提供ES启动成功或失败的信息回馈.只会在 `/var/log/elasticsearch/` 文件夹下进行输出日志.

默认地,`systemed` 并不会记录ES服务的日志.可以通过在服务文件 `elasticsearch.searvice` 中在 `ExecStart` 命令后移除 `--quit` 参数

当启用 `systemd` 日志, 可以使用 `journalctl` 命令进行查看其日志信息.

`tail` 其日志命令如下:
    
    sudo journalctl -f

列出 ES 相关的日志信息
    
    sudo journalctl --unit elasticsearch

To list journal entries for the elasticsearch service starting from a given time:
    
    
    sudo journalctl --unit elasticsearch --since  "2016-10-30 18:17:16"

Check `man journalctl` or <https://www.freedesktop.org/software/systemd/man/journalctl.html> for more command line options.

### Checking that Elasticsearch is running

You can test that your Elasticsearch node is running by sending an HTTP request to port `9200` on `localhost`:
    
    
    GET /

which should give you a response something like this:
    
    
    {
      "name" : "Cp8oag6",
      "cluster_name" : "elasticsearch",
      "cluster_uuid" : "AT69_T_DTp-1qgIJlatQqA",
      "version" : {
        "number" : "5.4.3",
        "build_hash" : "f27399d",
        "build_date" : "2016-03-30T09:51:41.449Z",
        "build_snapshot" : false,
        "lucene_version" : "6.5.0"
      },
      "tagline" : "You Know, for Search"
    }

### Configuring Elasticsearch

Elasticsearch loads its configuration from the `/etc/elasticsearch/elasticsearch.yml` file by default. The format of this config file is explained in [_Configuring Elasticsearch_](settings.html).

The RPM also has a system configuration file (`/etc/sysconfig/elasticsearch`), which allows you to set the following parameters:

`ES_USER`| The user to run as, defaults to `elasticsearch`.     
---|---    
`ES_GROUP`| The group to run as, defaults to `elasticsearch`.     
`JAVA_HOME`| Set a custom Java path to be used.     
`MAX_OPEN_FILES`| Maximum number of open files, defaults to `65536`.     
`MAX_LOCKED_MEMORY`| Maximum locked memory size. Set to `unlimited` if you use the `bootstrap.memory_lock` option in elasticsearch.yml.     
`MAX_MAP_COUNT`| 

Maximum number of memory map areas a process may have. If you use `mmapfs` as index store type, make sure this is set to a high value. For more information, check the [linux kernel documentation](https://github.com/torvalds/linux/blob/master/Documentation/sysctl/vm.txt) about `max_map_count`. This is set via `sysctl` before starting elasticsearch. Defaults to `262144`.   
  
`LOG_DIR`| Log directory, defaults to `/var/log/elasticsearch`.     
`DATA_DIR`| Data directory, defaults to `/var/lib/elasticsearch`.     
`CONF_DIR`| Configuration file directory (which needs to include `elasticsearch.yml` and `log4j2.properties` files), defaults to `/etc/elasticsearch`.     
`ES_JAVA_OPTS`| Any additional JVM system properties you may want to apply.     
`RESTART_ON_UPGRADE`| 
Configure restart on package upgrade, defaults to `false`. This means you will have to restart your elasticsearch instance after installing a package manually. The reason for this is to ensure, that upgrades in a cluster do not result in a continuous shard reallocation resulting in high network traffic and reducing the response times of your cluster.   
  
![Note](/images/icons/note.png)

Distributions that use `systemd` require that system resource limits be configured via `systemd` rather than via the `/etc/sysconfig/elasticsearch` file. See [Systemd configuration](setting-system-settings.html#systemd) for more information.

### Directory layout of RPM

RPM软件包将配置文件，日志和数据目录放置在基于RPM的系统的适当位置：

类型| 说明| 默认位置| 设置
:---:|:---|:---|:---
**home**| ES的主目录 `$ES_HOME`| `/usr/share/elasticsearch`|      
**bin**| 二进制脚本包括`elasticsearch`启动节点，`elasticsearch-plugin`安装插件| `/usr/share/elasticsearch/bin`|      
**conf**| 配置文件包括`elasticsearch.yml`和 `log4j2.properties`| `/etc/elasticsearch`| `path.conf`     
**conf**| 环境变量包括堆大小，文件描述符.| ***`/etc/sysconfig/elasticsearch`***|      
**data**| 在节点上分配的每个索引/分片的数据文件的位置。 可以容纳多个地点.| `/var/lib/elasticsearch`| `path.data`     
**logs**| 日志文件存放位置.| `/var/log/elasticsearch`| `path.logs`     
**plugins**| 插件安装位置,每个插件都会是一个子目录.| `/usr/share/elasticsearch/plugins`| ``     
**repo**| 共享文件系统存储库位置。 可以容纳多个地点。 文件系统存储库可以放置在此处指定的任何目录的任何子目录中.|没有配置| `path.repo`     
**script**| 脚本文件存放目录.| `/etc/elasticsearch/scripts`| `path.scripts`  
  
### 接下来

您现在已经设置了一个测试Elasticsearch环境。 在您开始认真开发或者使用Elasticsearch进行生产之前，您需要进行一些额外的设置：

  * [配置ES](settings.html). 
  * [配置ES重要的设置](important-settings.html). 
  * [配置系统设置](system-config.html). 


