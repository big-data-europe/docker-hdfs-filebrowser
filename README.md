[![Gitter chat](https://badges.gitter.im/gitterHQ/gitter.png)](https://gitter.im/big-data-europe/Lobby)

# bde2020/hdfs-filebrowser

This docker image runs HDFS FileBrowser from Cloudera Hue.

# Supported Versions

* 3.9
* 3.11

# Dependencies
* earthquakesan/ant
* earthquakesan/hue-build-env

# How To Use

This image is created for connecting to the HDFS cluster running inside Docker containers. To enable FileBrowser you need to:

* [Enable WebHDFS](https://hadoop.apache.org/docs/r1.0.4/webhdfs.html#HDFS+Configuration+Options) in hdfs-site.xml
* Create a [proxy user](https://hadoop.apache.org/docs/r2.7.1/hadoop-project-dist/hadoop-common/Superusers.html) with a name "hue" for FileBrowser in core-site.xml

This can be done with our [Hadoop docker image](https://github.com/big-data-europe/docker-hadoop) by setting environmental variables in hadoop.env as follows:
```
  CORE_CONF_hadoop_proxyuser_hue_hosts=*
  CORE_CONF_hadoop_proxyuser_hue_groups=*
  HDFS_CONF_dfs_webhdfs_enabled=true
```

Your HDFS docker container should run inside the same [docker network](https://docs.docker.com/engine/userguide/networking/dockernetworks/) as FileBrowser.
To create "hadoop" network type:
```
  docker network create hadoop
```

To start the FileBrowser run the container with NAMENODE_HOST env variable set to the namenode inside your docker network:
```
  docker run -e NAMENODE_HOST=namenode -p 8080:8088 bde2020/hdfs-filebrowser
```

Example docker-compose.yml file:
```
filebrowser:
  image: bde2020/hue
  hostname: filebrowser
  container_name: filebrowser
  domainname: hadoop
  net: hadoop
  environment:
    - NAMENODE_HOST=namenode
  ports:
    - "8088:8088"

namenode:
  image: bde2020/hadoop-namenode
  hostname: namenode
  container_name: namenode
  domainname: hadoop
  net: hadoop
  volumes:
    - ./data/namenode:/hadoop/dfs/name
  environment:
    - CLUSTER_NAME=test
  env_file:
    - ./hadoop.env

datanode1:
  image: bde2020/hadoop-datanode
  hostname: datanode1
  container_name: datanode1
  domainname: hadoop
  net: hadoop
  volumes:
    - ./data/datanode1:/hadoop/dfs/data
  env_file:
    - ./hadoop.env

datanode2:
  image: bde2020/hadoop-datanode
  hostname: datanode2
  container_name: datanode2
  domainname: hadoop
  net: hadoop
  volumes:
    - ./data/datanode2:/hadoop/dfs/data
  env_file:
    - ./hadoop.env
```

Example hadoop.env file:
```
#GANGLIA_HOST=ganglia.hadoop

CORE_CONF_fs_defaultFS=hdfs://namenode:8020
CORE_CONF_hadoop_http_staticuser_user=root
CORE_CONF_hadoop_proxyuser_hue_hosts=*
CORE_CONF_hadoop_proxyuser_hue_groups=*

HDFS_CONF_dfs_webhdfs_enabled=true
HDFS_CONF_dfs_permissions_enabled=false

YARN_CONF_yarn_log___aggregation___enable=true
YARN_CONF_yarn_resourcemanager_recovery_enabled=true
YARN_CONF_yarn_resourcemanager_store_class=org.apache.hadoop.yarn.server.resourcemanager.recovery.FileSystemRMStateStore
YARN_CONF_yarn_resourcemanager_fs_state___store_uri=/rmstate
YARN_CONF_yarn_nodemanager_remote___app___log___dir=/app-logs

YARN_CONF_yarn_log_server_url=http://historyserver.hadoop:8188/applicationhistory/logs/
YARN_CONF_yarn_timeline___service_enabled=true
YARN_CONF_yarn_timeline___service_generic___application___history_enabled=true
YARN_CONF_yarn_resourcemanager_system___metrics___publisher_enabled=true

YARN_CONF_yarn_resourcemanager_hostname=resourcemanager
YARN_CONF_yarn_timeline___service_hostname=historyserver.hadoop
```
