
# HiveServer2 Configuration

The following properties are required by knox in secured mode:

*   hive.server2.enable.doAs
*   hive.server2.allow.user.substitution
*   hive.server2.transport.mode
*   hive.server2.thrift.http.port
*   hive.server2.thrift.http.path

Example:

```json
{ "ryba": {
    "hive": {
      "server2": {
        "heapsize": "4096",
        "opts": "-Dcom.sun.management.jmxremote -Djava.rmi.server.hostname=130.98.196.54 -Dcom.sun.management.jmxremote.rmi.port=9526 -Dcom.sun.management.jmxremote.port=9526 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false"
      },
      "site": {
        "hive.server2.thrift.port": "10001"
      }
    }
} }
```

    module.exports = (service) ->
      options = service.options

## Environment

      # Layout
      options.conf_dir ?= '/etc/hive/conf'
      options.log_dir ?= '/var/log/hive'
      options.pid_dir ?= '/var/run/hive'
      # Opts and Java
      options.java_home ?= service.deps.java.options.java_home
      options.opts ?= ''
      options.mode ?= 'local'
      throw Error 'Invalid Options mode: accepted value are "local" or "remote"' unless options.mode in ['local', 'remote']
      options.heapsize ?= '1024'
      # Misc
      options.fqdn = service.node.fqdn
      options.hostname = service.node.hostname
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.clean_logs ?= false
      options.ranger_admin ?= !!service.deps.ranger_admin

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      throw Error 'Required Options: "realm"' unless options.krb5.realm
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

      options.hdfs_krb5_user = service.deps.hdfs[0].options.hdfs.krb5_user
      # Kerberos Test Principal
      options.test_krb5_user ?= service.deps.test_user.options.krb5.user

## Identities

      # Group
      options.group ?= {}
      options.group = name: options.group if typeof options.group is 'string'
      options.group.name ?= 'hive'
      options.group.system ?= true
      # User
      options.user ?= {}
      options.user = name: options.user if typeof options.user is 'string'
      options.user.name ?= 'hive'
      options.user.gid = options.group.name
      options.user.system ?= true
      options.user.groups ?= 'hadoop'
      options.user.comment ?= 'Ambari Hive User'
      options.user.home ?= '/var/lib/hive'
      options.user.limits ?= {}
      options.user.limits.nofile ?= 64000
      options.user.limits.nproc ?= 64000

## Ambari REST API

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.stack_name ?= service.deps.ambari_server.options.stack_name
      options.stack_version ?= service.deps.ambari_server.options.stack_version

## Ambari Configurations

      options.configurations ?= {}
      options.configurations['hive-site'] ?= {}
      options.configurations['hive-site']['hadoop.security.credential.provider.path'] ?= 'jceks://file/usr/hdp/current/hive-server2/conf/conf.server/hive-site.jceks'
      options.configurations['hive-site']['hive.server2.tez.default.queues'] ?= 'default'
      options.configurations['hive-site']['hive.server2.tez.initialize.default.sessions'] ?= 'false'
      options.configurations['hive-site']['hive.server2.tez.sessions.per.default.queue'] ?= '4'
      options.configurations['hivemetastore-site'] ?= {}
      options.configurations['hive-env'] ?= {}

      options.configurations['webcat-site'] ?= {}
      options.configurations['webcat-env'] ?= {}
      
      
## Ambari hive-env

      # Disbale Atlas Hook by default
      options.configurations['hive-env']['hive_security_authorization'] ?= 'SQLStdAuth' #SQLStdAuth, Ranger, NONE 
      options.configurations['hive-env']['hive_exec_orc_storage_strategy'] ?= 'SPEED' #SPEED or COMPRESSION
      options.configurations['hive-env']['hive_log_dir'] ?= options.log_dir #SPEED or COMPRESSION
      options.configurations['hive-env']['hive_pid_dir'] ?= options.pid_dir #SPEED or COMPRESSION
      options.configurations['hive-env']['hive_txn_acid'] ?= 'off' #on, off
      options.configurations['hive-env']['hive_user'] ?= options.user.name #on, off
      options.configurations['hive-env']['hive_user_nofile_limit'] ?= options.user.limits.nofile #on, off
      options.configurations['hive-env']['hive_user_nproc_limit'] ?= options.user.limits.nproc #on, off
      options.configurations['hive-env']['hive_timeline_logging_enabled'] ?= 'true' #Use ATS Logging
      options.configurations['hive-env']['enable_heap_dump'] ?= 'false' #Use ATS Logging
      options.configurations['hive-env']['hive.atlas.hook'] ?= 'false'
      options.configurations['hive-env']['hive.log.level'] ?= 'INFO'
      options.configurations['hive-env']['hive.atlas.hook'] ?= 'false'

## Ambari hive-interactive-site (LLAP)
Default properties
      
      options.configurations['hive-interactive-site'] ?= {}
      # START BLUEPRINT EXPORT
      options.configurations['hive-interactive-site']['hive.llap.client.consistent.splits'] ?= "true"
      options.configurations['hive-interactive-site']['hive.llap.daemon.yarn.container.mb'] ?= "341"
      options.configurations['hive-interactive-site']['hive.llap.io.memory.size'] ?= "0"
      options.configurations['hive-interactive-site']['hive.llap.task.scheduler.locality.delay'] ?= "-1"
      options.configurations['hive-interactive-site']['hive.vectorized.execution.reduce.enabled'] ?= "true"
      options.configurations['hive-interactive-site']['hive.llap.daemon.rpc.port'] ?= "15001"
      options.configurations['hive-interactive-site']['hive.execution.mode'] ?= "llap"
      options.configurations['hive-interactive-site']['hive.exec.orc.split.strategy'] ?= "HYBRID"
      options.configurations['hive-interactive-site']['hive.llap.daemon.allow.permanent.fns'] ?= "false"
      options.configurations['hive-interactive-site']['hive.llap.io.enabled'] ?= "true"
      options.configurations['hive-interactive-site']['hive.vectorized.execution.mapjoin.native.fast.hashtable.enabled'] ?= "true"
      options.configurations['hive-interactive-site']['hive.optimize.dynamic.partition.hashjoin'] ?= "true"
      options.configurations['hive-interactive-site']['hive.mapjoin.hybridgrace.hashtable'] ?= "false"
      options.configurations['hive-interactive-site']['hive.execution.engine'] ?= "tez"
      options.configurations['hive-interactive-site']['hive.llap.object.cache.enabled'] ?= "true"
      options.configurations['hive-interactive-site']['hive.llap.daemon.queue.name'] ?= "default"
      options.configurations['hive-interactive-site']['hive.metastore.uris'] ?= ""
      options.configurations['hive-interactive-site']['hive.llap.auto.allow.uber'] ?= "false"
      options.configurations['hive-interactive-site']['hive.server2.webui.port'] ?= "10502"
      options.configurations['hive-interactive-site']['hive.llap.daemon.num.executors'] ?= "1"
      options.configurations['hive-interactive-site']['hive.llap.daemon.task.scheduler.enable.preemption'] ?= "true"
      options.configurations['hive-interactive-site']['hive.llap.io.use.lrfu'] ?= "true"
      options.configurations['hive-interactive-site']['hive.vectorized.execution.mapjoin.native.enabled'] ?= "true"
      options.configurations['hive-interactive-site']['hive.metastore.event.listeners'] ?= ""
      options.configurations['hive-interactive-site']['hive.server2.tez.default.queues'] ?= "default"
      options.configurations['hive-interactive-site']['hive.llap.zk.sm.connectionString'] ?= "master01.metal.ryba:2181,master02.metal.ryba:2181"
      options.configurations['hive-interactive-site']['hive.server2.tez.sessions.per.default.queue'] ?= "1"
      options.configurations['hive-interactive-site']['hive.server2.webui.use.ssl'] ?= "false"
      options.configurations['hive-interactive-site']['hive.llap.management.rpc.port'] ?= "15004"
      options.configurations['hive-interactive-site']['hive.server2.thrift.http.port'] ?= "10501"
      options.configurations['hive-interactive-site']['hive.prewarm.enabled'] ?= "false"
      options.configurations['hive-interactive-site']['llap.shuffle.connection-keep-alive.timeout'] ?= "60"
      options.configurations['hive-interactive-site']['hive.llap.execution.mode'] ?= "all"
      options.configurations['hive-interactive-site']['hive.llap.daemon.yarn.shuffle.port'] ?= "15551"
      options.configurations['hive-interactive-site']['hive.llap.daemon.vcpus.per.instance'] ?= "${hive.llap.daemon.num.executors}"
      options.configurations['hive-interactive-site']['hive.llap.io.memory.mode'] ?= ""
      options.configurations['hive-interactive-site']['hive.server2.thrift.port'] ?= "10500"
      options.configurations['hive-interactive-site']['hive.driver.parallel.compilation'] ?= "true"
      options.configurations['hive-interactive-site']['hive.tez.exec.print.summary'] ?= "true"
      options.configurations['hive-interactive-site']['hive.tez.input.generate.consistent.splits'] ?= "true"
      options.configurations['hive-interactive-site']['hive.vectorized.execution.mapjoin.minmax.enabled'] ?= "true"
      options.configurations['hive-interactive-site']['hive.server2.enable.doAs'] ?= "false"
      options.configurations['hive-interactive-site']['hive.server2.zookeeper.namespace'] ?= "hiveserver2-hive2"
      options.configurations['hive-interactive-site']['hive.tez.bucket.pruning'] ?= "true"
      options.configurations['hive-interactive-site']['hive.llap.io.threadpool.size'] ?= "2"
      options.configurations['hive-interactive-site']['llap.shuffle.connection-keep-alive.enable'] ?= "true"
      options.configurations['hive-interactive-site']['hive.llap.daemon.service.hosts'] ?= "@llap0"
      options.configurations['hive-interactive-site']['hive.server2.tez.initialize.default.sessions'] ?= "true"
    

## Ambari hive-interactive-env (LLAP)
Default properties
      
      options.configurations['hive-interactive-env'] ?= {}
      options.configurations['hive-interactive-env']['enable_hive_interactive'] ?= 'false'
      options.configurations['hive-interactive-env']['num_llap_nodes'] ?= '4'
      options.configurations['hive-interactive-env']['hive.llap.daemon.yarn.container.mb'] ?= '600'
      options.configurations['hive-interactive-env']['hive.llap.io.memory.size'] ?= '6000'
      options.configurations['hive-interactive-env']['hive.llap.daemon.num.executors'] ?= '2'
      options.configurations['hive-interactive-env']['llap_heap_size'] ?= "0"
      options.configurations['hive-interactive-env']['llap_headroom_space'] ?= "6144"
      options.configurations['hive-interactive-env']['llap_app_name'] ?= "llap0"
      options.configurations['hive-interactive-env']['num_retries_for_checking_llap_status'] ?= "10"
      options.configurations['hive-interactive-env']['llap_queue_capacity'] ?= "0"
      options.configurations['hive-interactive-env']['content'] ?= "\n      if [ \"$SERVICE\" = \"cli\" ]; then\n      if [ -z \"$DEBUG\" ]; then\n      export HADOOP_OPTS=\"$HADOOP_OPTS -XX:NewRatio=12 -XX:MaxHeapFreeRatio=40 -XX:MinHeapFreeRatio=15 -XX:+UseParNewGC -XX:-UseGCOverheadLimit\"\n      else\n      export HADOOP_OPTS=\"$HADOOP_OPTS -XX:NewRatio=12 -XX:MaxHeapFreeRatio=40 -XX:MinHeapFreeRatio=15 -XX:-UseGCOverheadLimit\"\n      fi\n      fi\n\n      # The heap size of the jvm stared by hive shell script can be controlled via:\n\n      if [ \"$SERVICE\" = \"metastore\" ]; then\n      export HADOOP_HEAPSIZE={{hive_metastore_heapsize}} # Setting for HiveMetastore\n      else\n      export HADOOP_HEAPSIZE={{hive_heapsize}} # Setting for HiveServer2 and Client\n      fi\n\n      export HADOOP_CLIENT_OPTS=\"$HADOOP_CLIENT_OPTS  -Xmx${HADOOP_HEAPSIZE}m\"\n\n      # Larger heap size may be required when running queries over large number of files or partitions.\n      # By default hive shell scripts use a heap size of 256 (MB).  Larger heap size would also be\n      # appropriate for hive server (hwi etc).\n\n\n      # Set HADOOP_HOME to point to a specific hadoop install directory\n      HADOOP_HOME=${HADOOP_HOME:-{{hadoop_home}}}\n\n      # Hive Configuration Directory can be controlled by:\n      export HIVE_CONF_DIR={{hive_server_interactive_conf_dir}}\n\n      # Add additional hcatalog jars\n      if [ \"${HIVE_AUX_JARS_PATH}\" != \"\" ]; then\n      export HIVE_AUX_JARS_PATH=${HIVE_AUX_JARS_PATH}\n      else\n      export HIVE_AUX_JARS_PATH=/usr/hdp/current/hive-server2-hive2/lib/hive-hcatalog-core.jar\n      fi\n\n      export METASTORE_PORT={{hive_metastore_port}}\n\n      # Spark assembly contains a conflicting copy of HiveConf from hive-1.2\n      export HIVE_SKIP_SPARK_ASSEMBLY=true"
      options.configurations['hive-interactive-env']['llap_log_level'] ?= "INFO"
      options.configurations['hive-interactive-env']['slider_am_container_mb'] ?= "341"
      options.configurations['hive-interactive-env']['llap_java_opts'] ?= "-XX:+AlwaysPreTouch {% if java_version > 7 %}-XX:+UseG1GC -XX:TLABSize=8m -XX:+ResizeTLAB -XX:+UseNUMA -XX:+AggressiveOpts -XX:MetaspaceSize=1024m -XX:InitiatingHeapOccupancyPercent=80 -XX:MaxGCPauseMillis=200{% else %}-XX:+PrintGCDetails -verbose:gc -XX:+PrintGCTimeStamps -XX:+UseNUMA -XX:+UseParallelGC{% endif %}"
      # options.configurations['hive-interactive-env']['hive_server_interactive_conf_dir'] ?= "/etc/hive2/conf.server"
      
## Ambari hiveserver2-interactive-site (LLAP)
Default Properties

      options.configurations['hiveserver2-interactive-site'] ?= {}
      options.configurations['hiveserver2-interactive-site']["hive.async.log.enabled"] ?= "false"
      options.configurations['hiveserver2-interactive-site']["hive.service.metrics.hadoop2.component"] ?= "hiveserver2"
      options.configurations['hiveserver2-interactive-site']["hive.metastore.metrics.enabled"] ?= "true"
      options.configurations['hiveserver2-interactive-site']["hive.service.metrics.reporter"] ?= "JSON_FILE, JMX, HADOOP2"
      options.configurations['hiveserver2-interactive-site']["hive.service.metrics.file.location"] ?= "/var/log/hive/hiveserver2Interactive-report.json"
      
      
## Ambari Agent
Register users to ambari agent's user list.

      for srv in service.deps.ambari_agent
        srv.options.users ?= {}
        srv.options.users['hive'] ?= options.user
        srv.options.groups ?= {}
        srv.options.groups['hive'] ?= options.group

## Dependencies

    {merge} = require 'nikita/lib/misc'
