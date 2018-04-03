
# Hive Service Install

TODO: Implement lock for Hive Service
http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH4/4.2.0/CDH4-Installation-Guide/cdh4ig_topic_18_5.html

HDP 2.1 and 2.2 dont support secured Hive metastore in HA mode, see
[HIVE-9622](https://issues.apache.org/jira/browse/HIVE-9622).

Resources:
*   [Cloudera security instruction for CDH5](http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cdh_sg_hiveserver2_security.html)

    module.exports =  header: 'Ambari Hive Service Install', handler: (options) ->

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','hosts','add'], 'ryba-ambari-actions/lib/hosts/add'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"


## Identities

By default, the "hive" and "hive-hcatalog" packages create the following
entries:

```bash
cat /etc/passwd | grep hive
hive:x:493:493:Hive:/var/lib/hive:/sbin/nologin
cat /etc/group | grep hive
hive:x:493:
```
      # 
      # @system.group header: 'Group', options.group
      # @system.user header: 'User', options.user
      
      # @call ->
      #   missing = {}
      #   hive_site_ambari = 
      #     "ambari.hive.db.schema.name" : "hive"
      #     "atlas.hook.hive.maxThreads" : "1"
      #     "atlas.hook.hive.minThreads" : "1"
      #     "datanucleus.autoCreateSchema" : "false"
      #     "datanucleus.cache.level2.type" : "none"
      #     "datanucleus.fixedDatastore" : "true"
      #     "hive.auto.convert.join" : "true"
      #     "hive.auto.convert.join.noconditionaltask" : "true"
      #     "hive.auto.convert.join.noconditionaltask.size" : "71582788"
      #     "hive.auto.convert.sortmerge.join" : "false"
      #     "hive.auto.convert.sortmerge.join.to.mapjoin" : "false"
      #     "hive.cbo.enable" : "true"
      #     "hive.cli.print.header" : "false"
      #     "hive.cluster.delegation.token.store.class" : "org.apache.hadoop.hive.thrift.ZooKeeperTokenStore"
      #     "hive.cluster.delegation.token.store.zookeeper.connectString" : "master02.metal.ryba:2181,master01.metal.ryba:2181,master03.metal.ryba:2181"
      #     "hive.cluster.delegation.token.store.zookeeper.znode" : "/hive/cluster/delegation"
      #     "hive.compactor.abortedtxn.threshold" : "1000"
      #     "hive.compactor.check.interval" : "300L"
      #     "hive.compactor.delta.num.threshold" : "10"
      #     "hive.compactor.delta.pct.threshold" : "0.1f"
      #     "hive.compactor.initiator.on" : "true"
      #     "hive.compactor.worker.threads" : "0"
      #     "hive.compactor.worker.timeout" : "86400L"
      #     "hive.compute.query.using.stats" : "true"
      #     "hive.conf.restricted.list" : "hive.security.authenticator.manager,hive.security.authorization.manager,hive.users.in.admin.role"
      #     "hive.convert.join.bucket.mapjoin.tez" : "false"
      #     "hive.default.fileformat" : "TextFile"
      #     "hive.default.fileformat.managed" : "TextFile"
      #     "hive.enforce.bucketing" : "true"
      #     "hive.enforce.sorting" : "true"
      #     "hive.enforce.sortmergebucketmapjoin" : "true"
      #     "hive.exec.compress.intermediate" : "false"
      #     "hive.exec.compress.output" : "false"
      #     "hive.exec.dynamic.partition" : "true"
      #     "hive.exec.dynamic.partition.mode" : "strict"
      #     "hive.exec.failure.hooks" : "org.apache.hadoop.hive.ql.hooks.ATSHook"
      #     "hive.exec.max.created.files" : "100000"
      #     "hive.exec.max.dynamic.partitions" : "5000"
      #     "hive.exec.max.dynamic.partitions.pernode" : "2000"
      #     "hive.exec.orc.compression.strategy" : "SPEED"
      #     "hive.exec.orc.default.compress" : "ZLIB"
      #     "hive.exec.orc.default.stripe.size" : "67108864"
      #     "hive.exec.orc.encoding.strategy" : "SPEED"
      #     "hive.exec.parallel" : "false"
      #     "hive.exec.parallel.thread.number" : "8"
      #     "hive.exec.post.hooks" : "org.apache.hadoop.hive.ql.hooks.ATSHook"
      #     "hive.exec.pre.hooks" : "org.apache.hadoop.hive.ql.hooks.ATSHook"
      #     "hive.exec.reducers.bytes.per.reducer" : "67108864"
      #     "hive.exec.reducers.max" : "1009"
      #     "hive.exec.scratchdir" : "/tmp/hive"
      #     "hive.exec.submit.local.task.via.child" : "true"
      #     "hive.exec.submitviachild" : "false"
      #     "hive.execution.engine" : "tez"
      #     "hive.fetch.task.aggr" : "false"
      #     "hive.fetch.task.conversion" : "more"
      #     "hive.fetch.task.conversion.threshold" : "1073741824"
      #     "hive.limit.optimize.enable" : "true"
      #     "hive.limit.pushdown.memory.usage" : "0.04"
      #     "hive.map.aggr" : "true"
      #     "hive.map.aggr.hash.force.flush.memory.threshold" : "0.9"
      #     "hive.map.aggr.hash.min.reduction" : "0.5"
      #     "hive.map.aggr.hash.percentmemory" : "0.5"
      #     "hive.mapjoin.bucket.cache.size" : "10000"
      #     "hive.mapjoin.optimized.hashtable" : "true"
      #     "hive.mapred.reduce.tasks.speculative.execution" : "false"
      #     "hive.merge.mapfiles" : "true"
      #     "hive.merge.mapredfiles" : "false"
      #     "hive.merge.orcfile.stripe.level" : "true"
      #     "hive.merge.rcfile.block.level" : "true"
      #     "hive.merge.size.per.task" : "256000000"
      #     "hive.merge.smallfiles.avgsize" : "16000000"
      #     "hive.merge.tezfiles" : "false"
      #     "hive.metastore.authorization.storage.checks" : "false"
      #     "hive.metastore.cache.pinobjtypes" : "Table,Database,Type,FieldSchema,Order"
      #     "hive.metastore.client.connect.retry.delay" : "5s"
      #     "hive.metastore.client.socket.timeout" : "1800s"
      #     "hive.metastore.connect.retries" : "24"
      #     "hive.metastore.execute.setugi" : "true"
      #     "hive.metastore.failure.retries" : "24"
      #     "hive.metastore.kerberos.keytab.file" : "/etc/security/keytabs/hive.service.keytab"
      #     "hive.metastore.kerberos.principal" : "hive/_HOST@EXAMPLE.COM"
      #     "hive.metastore.pre.event.listeners" : "org.apache.hadoop.hive.ql.security.authorization.AuthorizationPreEventListener"
      #     "hive.metastore.sasl.enabled" : "false"
      #     "hive.metastore.server.max.threads" : "100000"
      #     "hive.metastore.uris" : "thrift://master01.metal.ryba:9083"
      #     "hive.metastore.warehouse.dir" : "/apps/hive/warehouse"
      #     "hive.optimize.bucketmapjoin" : "true"
      #     "hive.optimize.bucketmapjoin.sortedmerge" : "false"
      #     "hive.optimize.constant.propagation" : "true"
      #     "hive.optimize.index.filter" : "true"
      #     "hive.optimize.metadataonly" : "true"
      #     "hive.optimize.null.scan" : "true"
      #     "hive.optimize.reducededuplication" : "true"
      #     "hive.optimize.reducededuplication.min.reducer" : "4"
      #     "hive.optimize.sort.dynamic.partition" : "false"
      #     "hive.orc.compute.splits.num.threads" : "10"
      #     "hive.orc.splits.include.file.footer" : "false"
      #     "hive.prewarm.enabled" : "true"
      #     "hive.prewarm.numcontainers" : "3"
      #     "hive.security.authenticator.manager" : "org.apache.hadoop.hive.ql.security.ProxyUserAuthenticator"
      #     "hive.security.authorization.enabled" : "false"
      #     "hive.security.authorization.manager" : "org.apache.hadoop.hive.ql.security.authorization.plugin.sqlstd.SQLStdConfOnlyAuthorizerFactory"
      #     "hive.security.metastore.authenticator.manager" : "org.apache.hadoop.hive.ql.security.HadoopDefaultMetastoreAuthenticator"
      #     "hive.security.metastore.authorization.auth.reads" : "true"
      #     "hive.security.metastore.authorization.manager" : "org.apache.hadoop.hive.ql.security.authorization.StorageBasedAuthorizationProvider"
      #     "hive.server2.allow.user.substitution" : "true"
      #     "hive.server2.authentication" : "NONE"
      #     "hive.server2.authentication.kerberos.keytab" : "/etc/security/keytabs/hive.service.keytab"
      #     "hive.server2.authentication.kerberos.principal" : "hive/_HOST@EXAMPLE.COM"
      #     "hive.server2.authentication.spnego.keytab" : "HTTP/_HOST@EXAMPLE.COM"
      #     "hive.server2.authentication.spnego.principal" : "/etc/security/keytabs/spnego.service.keytab"
      #     "hive.server2.enable.doAs" : "true"
      #     "hive.server2.logging.operation.enabled" : "true"
      #     "hive.server2.logging.operation.log.location" : "/tmp/hive/operation_logs"
      #     "hive.server2.max.start.attempts" : "5"
      #     "hive.server2.support.dynamic.service.discovery" : "true"
      #     "hive.server2.table.type.mapping" : "CLASSIC"
      #     "hive.server2.tez.default.queues" : "default"
      #     "hive.server2.tez.initialize.default.sessions" : "false"
      #     "hive.server2.tez.sessions.per.default.queue" : "1"
      #     "hive.server2.thrift.http.path" : "cliservice"
      #     "hive.server2.thrift.http.port" : "10001"
      #     "hive.server2.thrift.max.worker.threads" : "500"
      #     "hive.server2.thrift.port" : "10000"
      #     "hive.server2.thrift.sasl.qop" : "auth"
      #     "hive.server2.transport.mode" : "binary"
      #     "hive.server2.use.SSL" : "true"
      #     "hive.server2.zookeeper.namespace" : "hiveserver2"
      #     "hive.smbjoin.cache.rows" : "10000"
      #     "hive.start.cleanup.scratchdir" : "false"
      #     "hive.stats.autogather" : "true"
      #     "hive.stats.dbclass" : "fs"
      #     "hive.stats.fetch.column.stats" : "true"
      #     "hive.stats.fetch.partition.stats" : "true"
      #     "hive.support.concurrency" : "false"
      #     "hive.tez.auto.reducer.parallelism" : "true"
      #     "hive.tez.container.size" : "256"
      #     "hive.tez.cpu.vcores" : "-1"
      #     "hive.tez.dynamic.partition.pruning" : "true"
      #     "hive.tez.dynamic.partition.pruning.max.data.size" : "104857600"
      #     "hive.tez.dynamic.partition.pruning.max.event.size" : "1048576"
      #     "hive.tez.input.format" : "org.apache.hadoop.hive.ql.io.HiveInputFormat"
      #     "hive.tez.java.opts" : "-server -Djava.net.preferIPv4Stack=true -XX:NewRatio=8 -XX:+UseNUMA -XX:+UseParallelGC -XX:+PrintGCDetails -verbose:gc -XX:+PrintGCTimeStamps"
      #     "hive.tez.log.level" : "INFO"
      #     "hive.tez.max.partition.factor" : "2.0"
      #     "hive.tez.min.partition.factor" : "0.25"
      #     "hive.tez.smb.number.waves" : "0.5"
      #     "hive.txn.manager" : "org.apache.hadoop.hive.ql.lockmgr.DummyTxnManager"
      #     "hive.txn.max.open.batch" : "1000"
      #     "hive.txn.timeout" : "300"
      #     "hive.user.install.directory" : "/user/"
      #     "hive.vectorized.execution.enabled" : "true"
      #     "hive.vectorized.execution.reduce.enabled" : "false"
      #     "hive.vectorized.groupby.checkinterval" : "4096"
      #     "hive.vectorized.groupby.flush.percent" : "0.1"
      #     "hive.vectorized.groupby.maxentries" : "100000"
      #     "hive.warehouse.subdir.inherit.perms" : "true"
      #     "hive.zookeeper.client.port" : "2181"
      #     "hive.zookeeper.namespace" : "hive_zookeeper_namespace"
      #     "hive.zookeeper.quorum" : "master02.metal.ryba:2181,master01.metal.ryba:2181,master03.metal.ryba:2181"
      #     "javax.jdo.option.ConnectionDriverName" : "com.mysql.jdbc.Driver"
      #     "javax.jdo.option.ConnectionPassword" : "SECRET:hive-site:1:javax.jdo.option.ConnectionPassword"
      #     "javax.jdo.option.ConnectionURL" : "jdbc:mysql://master01.metal.ryba/hive"
      #     "javax.jdo.option.ConnectionUserName" : "hive"
      # 
      #   hive_env_ambari =
      #     "alert_ldap_password" : ""
      #     "alert_ldap_username" : ""
      #     "content" : "\nexport HADOOP_USER_CLASSPATH_FIRST=true  #this prevents old metrics libs from mapreduce lib from bringing in old jar deps overriding HIVE_LIB\nif [ \"$SERVICE\" = \"cli\" ]; then\n  if [ -z \"$DEBUG\" ]; then\n    export HADOOP_OPTS=\"$HADOOP_OPTS -XX:NewRatio=12 -XX:MaxHeapFreeRatio=40 -XX:MinHeapFreeRatio=15 -XX:+UseNUMA -XX:+UseParallelGC -XX:-UseGCOverheadLimit\"\n  else\n    export HADOOP_OPTS=\"$HADOOP_OPTS -XX:NewRatio=12 -XX:MaxHeapFreeRatio=40 -XX:MinHeapFreeRatio=15 -XX:-UseGCOverheadLimit\"\n  fi\nfi\n\n# The heap size of the jvm stared by hive shell script can be controlled via:\n\nif [ \"$SERVICE\" = \"metastore\" ]; then\n  export HADOOP_HEAPSIZE={{hive_metastore_heapsize}} # Setting for HiveMetastore\nelse\n  export HADOOP_HEAPSIZE={{hive_heapsize}} # Setting for HiveServer2 and Client\nfi\n\nexport HADOOP_CLIENT_OPTS=\"$HADOOP_CLIENT_OPTS  -Xmx${HADOOP_HEAPSIZE}m\"\nexport HADOOP_CLIENT_OPTS=\"$HADOOP_CLIENT_OPTS{{heap_dump_opts}}\"\n\n# Larger heap size may be required when running queries over large number of files or partitions.\n# By default hive shell scripts use a heap size of 256 (MB).  Larger heap size would also be\n# appropriate for hive server (hwi etc).\n\n\n# Set HADOOP_HOME to point to a specific hadoop install directory\nHADOOP_HOME=${HADOOP_HOME:-{{hadoop_home}}}\n\nexport HIVE_HOME=${HIVE_HOME:-{{hive_home_dir}}}\n\n# Hive Configuration Directory can be controlled by:\nexport HIVE_CONF_DIR=${HIVE_CONF_DIR:-{{hive_config_dir}}}\n\n# Folder containing extra libraries required for hive compilation/execution can be controlled by:\nexport HIVE_AUX_JARS_PATH={{stack_root}}/current/ext/hive\nif [ \"${HIVE_AUX_JARS_PATH}\" != \"\" ]; then\n  if [ -f \"${HIVE_AUX_JARS_PATH}\" ] || [ -d \"${HIVE_AUX_JARS_PATH}\" ] ; then\n    export HIVE_AUX_JARS_PATH=${HIVE_AUX_JARS_PATH}\n  elif [ -d \"/usr/hdp/current/hive-webhcat/share/hcatalog\" ]; then\n    export HIVE_AUX_JARS_PATH=/usr/hdp/current/hive-webhcat/share/hcatalog/hive-hcatalog-core.jar\n  fi\nelif [ -d \"/usr/hdp/current/hive-webhcat/share/hcatalog\" ]; then\n  export HIVE_AUX_JARS_PATH=/usr/hdp/current/hive-webhcat/share/hcatalog/hive-hcatalog-core.jar\nfi\n\nexport METASTORE_PORT={{hive_metastore_port}}\n\n{% if sqla_db_used or lib_dir_available %}\nexport LD_LIBRARY_PATH=\"$LD_LIBRARY_PATH:{{jdbc_libs_dir}}\"\nexport JAVA_LIBRARY_PATH=\"$JAVA_LIBRARY_PATH:{{jdbc_libs_dir}}\"\n{% endif %}"
      #     "enable_heap_dump" : "false"
      #     "hcat_log_dir" : "/var/log/webhcat"
      #     "hcat_pid_dir" : "/var/run/webhcat"
      #     "hcat_user" : "hcat"
      #     "heap_dump_location" : "/tmp"
      #     "hive.atlas.hook" : "false"
      #     "hive.client.heapsize" : "1024"
      #     "hive.heapsize" : "1421"
      #     "hive.log.level" : "INFO"
      #     "hive.metastore.heapsize" : "512"
      #     "hive_ambari_database" : "MySQL"
      #     "hive_database" : "Existing MySQL / MariaDB Database"
      #     "hive_database_name" : "hive"
      #     "hive_database_type" : "mysql"
      #     "hive_exec_orc_storage_strategy" : "SPEED"
      #     "hive_log_dir" : "/var/log/hive"
      #     "hive_pid_dir" : "/var/run/hive"
      #     "hive_security_authorization" : "SQLStdAuth"
      #     "hive_timeline_logging_enabled" : "true"
      #     "hive_txn_acid" : "off"
      #     "hive_user" : "hive"
      #     "hive_user_nofile_limit" : "32000"
      #     "hive_user_nproc_limit" : "16000"
      #     "webhcat_user" : "hcat"
      #   logged_diff = []
      #   differences = {}
        # for k, v of options.configurations['hive-site']
        #   if hive_site_ambari[k]?
        #     # console.log "difference - #{k} - ambari: #{hive_site_ambari[k]}, ryba: #{options.configurations['hive-site'][k]}" unless "#{options.configurations['hive-site'][k]}" is "#{hive_site_ambari[k]}"
        #     differences[k] ?= hive_site_ambari[k] unless "#{options.configurations['hive-site'][k]}" is "#{hive_site_ambari[k]}"
        #     logged_diff.push k unless "#{options.configurations['hive-site'][k]}" is "#{hive_site_ambari[k]}"
        #   else
        #     # console.log "missing - in ambari: #{k}"
        #     missing["#{k}"] = v
        # console.log '------------------------------------------------------------'
        # for k, v of hive_site_ambari
        #   if options.configurations['hive-site'][k]?
        #     # console.log "difference - #{k} - ambari: #{hive_site_ambari[k]}, ryba: #{options.configurations['hive-site'][k]}"  unless ("#{options.configurations['hive-site'][k]}" is "#{hive_site_ambari[k]}" ) and (k not in logged_diff)
        #     differences[k] ?= hive_site_ambari[k] unless ("#{options.configurations['hive-site'][k]}" is "#{hive_site_ambari[k]}" )
        #   else
        #     # console.log 'missing - in RYBA', k 
        #     missing["#{k}"] = v
        # console.log differences
        # console.log missing
        # options.configurations['hive-env']['hive_security_authorization'] = 'SQLStdAuth'
        # 
        # for k, v of options.configurations['hive-env']
        #   if hive_env_ambari[k]?
        #     console.log "difference - #{k} - ambari: #{hive_env_ambari[k]}, ryba: #{options.configurations['hive-env'][k]}" unless "#{options.configurations['hive-env'][k]}" is "#{hive_env_ambari[k]}"
        #     differences[k] ?= hive_env_ambari[k] unless "#{options.configurations['hive-env'][k]}" is "#{hive_env_ambari[k]}"
        #   else
        #     console.log "missing - in ambari: #{k}"
        #     # missing["#{k}"] = v
        # console.log '------------------------------------------------------------'
        # for k, v of hive_env_ambari
        #   if options.configurations['hive-env'][k]?
        #     console.log "difference - #{k} - ambari: #{hive_env_ambari[k]}, ryba: #{options.configurations['hive-env'][k]}"  unless "#{options.configurations['hive-env'][k]}" is "#{hive_env_ambari[k]}"
        #     differences[k] ?= hive_env_ambari[k] unless "#{options.configurations['hive-env'][k]}" is "#{hive_env_ambari[k]}"
        #   else
        #     console.log 'missing - in RYBA', k 
        #     missing["#{k}"] = v
        # console.log differences
      # @call ->
      #   process.exit 1
      
## Layout

Create the directories to store the logs and pid information. The properties
"ryba.hive.server2.log\_dir" and "ryba.hive.server2.pid\_dir" may be modified.

      @call header: 'Layout', ->
        @system.mkdir
          target: options.log_dir
          uid: options.user.name
          gid: options.group.name
          parent: true
        @system.mkdir
          target: options.pid_dir
          uid: options.user.name
          gid: options.group.name
          parent: true



## Render Configuration

      @hconfigure
        header: 'Render hive-site'
        if: options.post_component
        source: "#{__dirname}/../resources/hive-site.xml"
        target: "#{options.cache_dir}/hive-site.xml"
        ssh: false
        properties: options.configurations['hive-site']
      # @hconfigure
      #   header: 'Render hive-interactive-site'
      #   if: options.post_component
      #   source: "#{__dirname}/../resources/hive-site.xml"
      #   target: "#{options.cache_dir}/hive-interactive-site.xml"
      #   ssh: false
      #   properties: options.configurations['hive-interactive-site']
      # @call ->
      #   CLIENT_OPTS = ''
      #   CLIENT_OPTS += " -D#{k}=#{v}" for k, v of options.client_opts.java_properties
      #   CLIENT_OPTS += " #{k}#{v}" for k, v of options.client_opts.jvm
      #   SERVER2_OPTS = ''
      #   SERVER2_OPTS += " -D#{k}=#{v}" for k, v of options.server2_opts.java_properties
      #   SERVER2_OPTS += " #{k}#{v}" for k, v of options.server2_opts.jvm
      #   HCATALOG_OPTS = ''
      #   HCATALOG_OPTS += " -D#{k}=#{v}" for k, v of options.hcatalog_opts.java_properties
      #   HCATALOG_OPTS += " #{k}#{v}" for k, v of options.hcatalog_opts.jvm
      #   WEBHCAT_OPTS = ''
      #   WEBHCAT_OPTS += " -D#{k}=#{v}" for k, v of options.webhcat_opts.java_properties
      #   WEBHCAT_OPTS += " #{k}#{v}" for k, v of options.webhcat_opts.jvm
      #   @file.render
      #     header: 'Render hive-env'
      #     if: options.post_component
      #     source: "#{__dirname}/../resources/hive-env.sh.j2"
      #     target: "#{options.cache_dir}/hive-env.sh"
      #     local: true
      #     ssh: false
      #     context:
      #       CLIENT_OPTS: CLIENT_OPTS
      #       SERVER2_OPTS: SERVER2_OPTS
      #       HCATALOG_OPTS: HCATALOG_OPTS
      #       WEBHCAT_OPTS: WEBHCAT_OPTS
      #       HIVE_CONF_DIR: options.conf_dir
      #       SERVER2_AUX_JARS: options.server2_aux_jars
      #       CLIENT_AUX_JARS: options.client_aux_jars
      #       HCATALOG_AUX_JARS: options.hcatalog_aux_jars
      #     eof: true
      #     backup: true
      #     mode: 0o0750
      #     ssh: false
      #   @file.render
      #     header: 'Render hive-interactive-env'
      #     if: options.post_component
      #     source: "#{__dirname}/../resources/hive-env.sh.j2"
      #     target: "#{options.cache_dir}/hive-interactive-env.sh"
      #     local: true
      #     ssh: false
      #     context:
      #       CLIENT_OPTS: CLIENT_OPTS
      #       SERVER2_OPTS: SERVER2_OPTS
      #       HCATALOG_OPTS: HCATALOG_OPTS
      #       WEBHCAT_OPTS: WEBHCAT_OPTS
      #       HIVE_CONF_DIR: options.conf_dir
      #       SERVER2_AUX_JARS: options.server2_aux_jars
      #       CLIENT_AUX_JARS: options.client_aux_jars
      #       HCATALOG_AUX_JARS: options.hcatalog_aux_jars
      #     eof: true
      #     backup: true
      #     mode: 0o0750
      #     ssh: false
        
      @file.render
        header: 'Render hive-exec-log4j2'
        if: options.post_component
        source: "#{__dirname}/../resources/hive-exec-log4j.properties.j2"
        local: true
        target: "#{options.cache_dir}/hive-exec-log4j.properties"
        context: options
        ssh: false
      @file.properties
        header: 'Render hive-log4j2'
        if: options.post_component
        target: "#{options.cache_dir}/hive-log4j.properties"
        content: options.hive_log4j
        backup: true
        ssh: false

      @call header: 'Render wehbcat-env', ->
        webhcat_opts = ''
        webhcat_opts += " -D#{k}=#{v}" for k, v of options.webhcat_opts.java_properties
        webhcat_opts += " #{k}#{v}" for k, v of options.webhcat_opts.jvm
        @file
          source: "#{__dirname}/../resources/webhcat-env.sh.j2"
          local: true
          target: "#{options.cache_dir}/webhcat-env.sh"
          ssh: false
          mode: 0o0755
          write: [
            match: RegExp "export HADOOP_OPTS=.*", 'm'
            replace: "export HADOOP_OPTS=\"${HADOOP_OPTS} #{webhcat_opts}\" # RYBA, DONT OVERWRITE"
            append: true
          ]
      @file
        header: 'Render webhcat-log4j'
        if: options.post_component and options.webhcat_log4j?
        target: "#{options.cache_dir}/webhcat-log4j.properties"
        source: "#{__dirname}/../resources/webhcat-log4j.properties"
        local: true
        ssh: false
        write: for k, v of options.webhcat_log4j
          match: RegExp "#{k}=.*", 'm'
          replace: "#{k}=#{v}"
          append: true
      @hconfigure
        header: 'Render webhcat-site'
        if: options.post_component
        source: "#{__dirname}/../resources/webhcat-site.xml"
        target: "#{options.cache_dir}/webhcat-site.xml"
        ssh: false
        properties: options.configurations['webhcat-site']

## Upload Configurations
Upload hive-env, hive-site, hive-exec-log4j2, hive-log4j2, webhcat-env, webhcat-site
and webhcat-log4j

      # @call
      #   header: 'Upload hive-env'
      #   if: options.post_component
      # , (_, callback) ->
      #     ssh2fs.readFile null, "#{options.cache_dir}/hive-env.sh", (err, content) =>
      #       try
      #         throw err if err
      #         content = content.toString()
      #         @ambari.configs.update
      #           url: options.ambari_url
      #           username: 'admin'
      #           merge: true
      #           password: options.ambari_admin_password
      #           config_type: 'hive-env'
      #           cluster_name: options.cluster_name
      #           properties: merge {},  options.configurations['hive-env'],
      #             content: content
      #         .next callback
      #       catch err
      #         callback err
      # 
      # @call
      #   header: 'Upload hive-interactive-env'
      #   if: options.post_component
      # , (_, callback) ->
      #     ssh2fs.readFile null, "#{options.cache_dir}/hive-interactive-env.sh", (err, content) =>
      #       try
      #         throw err if err
      #         content = content.toString()
      #         @ambari.configs.update
      #           url: options.ambari_url
      #           username: 'admin'
      #           merge: true
      #           password: options.ambari_admin_password
      #           config_type: 'hive-interactive-env'
      #           cluster_name: options.cluster_name
      #           properties: merge {},  options.configurations['hive-interactive-env'],
      #             content: content
      #         .next callback
      #       catch err
      #         callback err

      @call
        header: 'Upload hcat-env'
        if: options.post_component
      , (_, callback) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/hcat-env.sh.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                url: options.ambari_url
                username: 'admin'
                merge: true
                password: options.ambari_admin_password
                config_type: 'hcat-env'
                cluster_name: options.cluster_name
                properties: merge {},  options.configurations['hcat-env'],
                  content: content
              .next callback
            catch err
              callback err

      @call
        header: 'Upload hive-env'
        if: options.post_component
      , (_, callback) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/hive-env.sh.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                url: options.ambari_url
                username: 'admin'
                merge: true
                password: options.ambari_admin_password
                config_type: 'hive-env'
                cluster_name: options.cluster_name
                properties: merge {},  options.configurations['hive-env'],
                  content: content
              .next callback
            catch err
              callback err

      @call
        header: 'Upload hive-interactive-env'
        if: options.post_component
      , (_, callback) ->
          console.log 'TODO: CHECK|ING add merge hive-interactive-env'
          ssh2fs.readFile null, "#{__dirname}/../resources/hive-env.sh.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                url: options.ambari_url
                username: 'admin'
                merge: true
                password: options.ambari_admin_password
                config_type: 'hive-interactive-env'
                cluster_name: options.cluster_name
                properties: merge {},  options.configurations['hive-interactive-env']
                  # content: content
              .next callback
            catch err
              callback err



      @call
        header: 'Upload hive-site'
        if: options.post_component
      , (_, callback) ->
          properties.read null, "#{options.cache_dir}/hive-site.xml", (err, props) =>
            @ambari.configs.update
              header: 'config update hive-site'
              url: options.ambari_url
              username: 'admin'
              password: options.ambari_admin_password
              config_type: 'hive-site'
              cluster_name: options.cluster_name
              properties: props
            @ambari.configs.update
              header: 'config update hive-site'
              url: options.ambari_url
              username: 'admin'
              password: options.ambari_admin_password
              config_type: 'hivemetastore-site'
              cluster_name: options.cluster_name
              properties: props
            @next callback

      # @call
      #   header: 'Upload hive-site'
      #   if: options.post_component
      # , (_, callback) ->
      #     properties = JSON.parse fs.readFileSync '/home/bakalian/ryba/ryba-env-metal-ambari/hive-site-ambari-topost.json'
      #     @ambari.configs.update
      #       header: 'config update hive-site'
      #       url: options.ambari_url
      #       username: 'admin'
      #       password: options.ambari_admin_password
      #       config_type: 'hive-site'
      #       cluster_name: options.cluster_name
      #       properties: properties.properties
      #     @next callback

      # @call
      #   header: 'Upload hive-interactive-site'
      # , (_, callback) ->
      #     properties.read null, "#{options.cache_dir}/hive-interactive-site.xml", (err, props) =>
      @ambari.configs.update
        header: 'Upload hive-interactive-site'
        if: options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'hive-interactive-site'
        cluster_name: options.cluster_name
        properties: options.configurations['hive-interactive-site']
            # @next callback


      @call
        header: 'Upload hive-exec-log4j2'
        if: options.post_component
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/hive-exec-log4j.properties", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                url: options.ambari_url
                username: 'admin'
                merge: true
                password: options.ambari_admin_password
                config_type: 'hive-exec-log4j2'
                cluster_name: options.cluster_name
                properties: content: content
              .next callback
            catch err
              callback err

      @call
        header: 'Upload hive-log4j2'
        if: options.post_component
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/hive-log4j.properties", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                url: options.ambari_url
                username: 'admin'
                merge: true
                password: options.ambari_admin_password
                config_type: 'hive-log4j2'
                cluster_name: options.cluster_name
                properties: content: content
              .next callback
            catch err
              callback err

      @call
        header: 'Upload webhcat-env'
        if: options.post_component
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/webhcat-env.sh", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                url: options.ambari_url
                username: 'admin'
                merge: true
                password: options.ambari_admin_password
                config_type: 'webhcat-env'
                cluster_name: options.cluster_name
                properties: merge {},  options.configurations['webhcat-env'],
                  content: content
              .next callback
            catch err
              callback err

      @call
        header: 'Upload webhcat-site'
        if: options.post_component
      , (_, callback) ->
          properties.read null, "#{options.cache_dir}/webhcat-site.xml", (err, props) =>
            @ambari.configs.update
              header: 'config update webhcat-site'
              url: options.ambari_url
              username: 'admin'
              password: options.ambari_admin_password
              config_type: 'webhcat-site'
              cluster_name: options.cluster_name
              properties: props
            @next callback


      @call
        header: 'Upload webhcat-log4j'
        if: options.post_component
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/webhcat-log4j.properties", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                url: options.ambari_url
                username: 'admin'
                merge: true
                password: options.ambari_admin_password
                config_type: 'webhcat-log4j'
                cluster_name: options.cluster_name
                properties: content: content
              .next callback
            catch err
              callback err

      @ambari.configs.update
        header: 'Upload hiveserver2-site'
        if: options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'hiveserver2-site'
        cluster_name: options.cluster_name
        properties: options.configurations['hiveserver2-site']

## Upload Ranger Related Properties

      @ambari.configs.update
        header: 'Upload ranger-hive-plugin-properties'
        if : options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hive-plugin-properties'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hive-plugin-properties']

      @ambari.configs.update
        header: 'Upload ranger-hive-security'
        if : options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hive-security'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hive-security']

      @ambari.configs.update
        header: 'Upload ranger-hive-policymgr-ssl'
        if : options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hive-policymgr-ssl'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hive-policymgr-ssl']

      @ambari.configs.update
        header: 'Upload ranger-hive-audit'
        if : options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hive-audit'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hive-audit']

## Add HIVE Service

      @ambari.services.add
        header: 'HIVE Service'
        if: options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HIVE'

# ## Add SQOOP Service
# 
#       @ambari.services.add
#         header: 'SQOOP Service'
#         if: options.post_component
#         url: options.ambari_url
#         username: 'admin'
#         password: options.ambari_admin_password
#         cluster_name: options.cluster_name
#         name: 'SQOOP'
# 

## Add and enable HIVE component
add `HIVE_SERVER`, `HCAT`, `HIVE_CLIENT`, `HIVE_METASTORE`, `HIVE_SERVER_INTERACTIVE` (LLAP)
 `WEBHCAT_SERVER` components to cluster in `INIT` state.

      @ambari.services.wait
        header: 'HIVE Service WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HIVE'

      @ambari.services.component_add
        if: options.post_component
        header: 'HIVE_SERVER'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HIVE_SERVER'
        service_name: 'HIVE'

      @ambari.services.component_add
        if: options.post_component
        header: 'HCAT'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HCAT'
        service_name: 'HIVE'
        
      @ambari.services.component_add
        if: options.post_component
        header: 'HIVE_CLIENT'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HIVE_CLIENT'
        service_name: 'HIVE'

      # @ambari.services.component_add
      #   if: options.post_component
      #   header: 'HCAT_CLIENT'
      #   url: options.ambari_url
      #   username: 'admin'
      #   password: options.ambari_admin_password
      #   cluster_name: options.cluster_name
      #   component_name: 'HCAT_CLIENT'
      #   service_name: 'HIVE'
        
      @ambari.services.component_add
        if: options.post_component
        header: 'HIVE_METASTORE'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HIVE_METASTORE'
        service_name: 'HIVE'

      @ambari.services.component_add
        if: options.post_component
        header: 'WEBHCAT_SERVER'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'WEBHCAT_SERVER'
        service_name: 'HIVE'

      for host in options.server2_hosts
        @ambari.hosts.component_add
          header: 'HIVE_SERVER'
          if: options.post_component
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'HIVE_SERVER'
          hostname: host

      # for host in options.metastore_hosts
      #   @ambari.hosts.component_add
      #     header: 'HIVE_METASTORE'
      #     if: options.post_component
      #     url: options.ambari_url
      #     username: 'admin'
      #     password: options.ambari_admin_password
      #     cluster_name: options.cluster_name
      #     component_name: 'HIVE_METASTORE'
      #     hostname: host

      for host in options.hcatalog_hosts
        @ambari.hosts.component_add
          header: 'HIVE_METASTORE'
          if: options.post_component
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'HIVE_METASTORE'
          hostname: host

      for host in options.hcatalog_hosts
        @ambari.hosts.component_add
          header: 'HIVE_CLIENT'
          if: options.post_component
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'HIVE_CLIENT'
          hostname: host

      # for host in [options.hcatalog_hosts]
      #   @ambari.hosts.component_add
      #     header: 'HIVE_CLIENT'
      #     if: options.post_component
      #     url: options.ambari_url
      #     username: 'admin'
      #     password: options.ambari_admin_password
      #     cluster_name: options.cluster_name
      #     component_name: 'HIVE_CLIENT'
      #     hostname: host


      for host in options.webhcat_hosts
        @ambari.hosts.component_add
          header: 'WEBHCAT_SERVER'
          if: options.post_component
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'WEBHCAT_SERVER'
          hostname: host

## Dependencies

    path = require 'path'
    {merge} = require 'nikita/lib/misc'
    fs = require 'fs'
    ssh2fs = require 'ssh2-fs'
    properties = require 'ryba/lib/properties'
