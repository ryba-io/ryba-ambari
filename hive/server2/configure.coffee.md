
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
      options.conf_dir ?= service.deps.hive[0].options.conf_dir
      options.log_dir ?= service.deps.hive[0].options.log_dir
      options.pid_dir ?= service.deps.hive[0].options.pid_dir
      # Opts and Java
      options.java_home ?= service.deps.java.options.java_home
      options.mode ?= 'local'
      throw Error 'Invalid Options mode: accepted value are "local" or "remote"' unless options.mode in ['local', 'remote']
      options.heapsize ?= '1024'
      options.opts ?= {}
      # Misc
      options.fqdn = service.node.fqdn
      options.hostname = service.node.hostname
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.clean_logs ?= false

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      throw Error 'Required Options: "realm"' unless options.krb5.realm
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

## Identities

      options.group = merge {}, service.deps.hive[0].options.group, options.group
      options.user = merge {}, service.deps.hive[0].options.user, options.user
      options.hdfs_krb5_user ?= service.deps.hadoop_core.options.hdfs.krb5_user

## Configuration

      options.hive_site ?= {}
      properties = [ # Duplicate client, might remove
        'hive.metastore.sasl.enabled'
        'hive.security.authorization.enabled'
        # 'hive.security.authorization.manager'
        'hive.security.metastore.authorization.manager'
        'hive.security.authenticator.manager'
        'hive.optimize.mapjoin.mapreduce'
        'hive.enforce.bucketing'
        'hive.exec.dynamic.partition.mode'
        'hive.txn.manager'
        'hive.txn.timeout'
        'hive.txn.max.open.batch'
        # Transaction, read/write locks
        'hive.support.concurrency'
        'hive.cluster.delegation.token.store.zookeeper.connectString'
        # 'hive.cluster.delegation.token.store.zookeeper.znode'
        'hive.heapsize'
        'hive.exec.max.created.files'
        'hive.auto.convert.sortmerge.join.noconditionaltask'
        'hive.zookeeper.quorum'
      ]
      if options.mode is 'local'
        properties = properties.concat [
          'datanucleus.autoCreateTables'
          'hive.cluster.delegation.token.store.class'
          'hive.cluster.delegation.token.store.zookeeper.znode'
        ]
        options.hive_site['hive.metastore.uris'] = ' '
        options.hive_site['hive.compactor.initiator.on'] = 'false'
      else
        properties.push 'hive.metastore.uris'
      for property in properties
        options.hive_site[property] ?= service.deps.hive_hcatalog[0].options.hive_site[property]
      # Server2 specific properties
      options.hive_site['hive.server2.thrift.sasl.qop'] ?= 'auth'
      options.hive_site['hive.server2.enable.doAs'] ?= 'true'
      # options.hive_site['hive.server2.enable.impersonation'] ?= 'true' # Mention in CDH5.3 but hs2 logs complains it doesnt exist
      options.hive_site['hive.server2.allow.user.substitution'] ?= 'true'
      options.hive_site['hive.server2.transport.mode'] ?= 'http'
      options.hive_site['hive.server2.thrift.port'] ?= '10001'
      options.hive_site['hive.server2.thrift.http.port'] ?= '10001'
      options.hive_site['hive.server2.thrift.http.path'] ?= 'cliservice'
      # Bug fix: java properties are not interpolated
      # Default is "${system:java.io.tmpdir}/${system:user.name}/operation_logs"
      options.hive_site['hive.server2.logging.operation.log.location'] ?= "/tmp/#{options.user.name}/operation_logs"

## Optimizations

      # Tez
      # https://streever.atlassian.net/wiki/pages/viewpage.action?pageId=4390918
      options.hive_site['hive.execution.engine'] ?= if service.deps.tez then 'tez' else 'mr'
      options.hive_site['hive.server2.tez.default.queues'] ?= 'default'
      options.hive_site['hive.server2.tez.sessions.per.default.queue'] ?= '1'
      options.hive_site['hive.server2.tez.initialize.default.sessions'] ?= 'false'
      options.hive_site['hive.exec.post.hooks'] ?= 'org.apache.hadoop.hive.ql.hooks.ATSHook'
      options.hive_site['hive.prewarm.enabled'] ?= 'true'# hold containers to reduce latency
      options.hive_site['hive.prewarm.numcontainers'] ?= '3'
      # Permission inheritance
      # https://cwiki.apache.org/confluence/display/Hive/Permission+Inheritance+in+Hive
      # true unless ranger is the authorizer
      options.hive_site['hive.warehouse.subdir.inherit.perms'] ?= unless service.deps.ranger_admin then 'true' else 'false'
      options.hive_site['hive.auto.convert.join.noconditionaltask.size'] ?= '238026752'
      options.hive_site['hive.auto.convert.sortmerge.join'] ?= 'false'
      options.hive_site['hive.auto.convert.sortmerge.join.to.mapjoin'] ?= 'false'
      options.hive_site['hive.cbo.enable'] ?= 'true'
      options.hive_site['hive.stats.fetch.column.stats'] ?= 'false'
      options.hive_site['hive.compute.query.using.stats'] ?= 'true'
      options.hive_site['hive.conf.restricted.list'] ?= 'hive.security.authenticator.manager,hive.security.authorization.manager,hive.users.in.admin.role'
      options.hive_site['hive.convert.join.bucket.mapjoin.tez'] ?= 'false'
      options.hive_site['hive.default.fileformat'] ?= 'TextFile'
      options.hive_site['hive.default.fileformat.managed'] ?= 'TextFile'
      options.hive_site['hive.exec.orc.encoding.strategy'] ?= 'SPEED'
      options.hive_site['hive.exec.reducers.bytes.per.reducer'] ?= '67108864'
      # start default - required by ambari
      options.hive_site['hive.exec.compress.output'] ?= 'false'
      options.hive_site['hive.exec.dynamic.partition'] ?= 'true'
      options.hive_site['hive.exec.failure.hooks'] ?= 'org.apache.hadoop.hive.ql.hooks.ATSHook'
      options.hive_site['hive.exec.max.dynamic.partitions'] ?= '5000'
      options.hive_site['hive.exec.max.dynamic.partitions.pernode'] ?= '2000'
      options.hive_site['hive.exec.orc.compression.strategy'] ?= 'SPEED'
      options.hive_site['hive.exec.parallel'] ?= 'false'
      options.hive_site['hive.exec.parallel.thread.number'] ?= '8'
      options.hive_site['hive.exec.pre.hooks'] ?= 'org.apache.hadoop.hive.ql.hooks.ATSHook'
      options.hive_site['hive.exec.reducers.max'] ?= '1009'
      options.hive_site['hive.exec.submit.local.task.via.child'] ?= 'true'
      options.hive_site['hive.exec.submitviachild'] ?= 'false'
      options.hive_site['atlas.hook.hive.maxThreads'] ?= '1'
      options.hive_site['atlas.hook.hive.minThreads'] ?= '1'
      options.hive_site['hive.auto.convert.join.noconditionaltask'] ?= 'true'
      options.hive_site['hive.enforce.sorting'] ?= 'true'
      options.hive_site['hive.enforce.sortmergebucketmapjoin'] ?= 'true'
      options.hive_site['hive.fetch.task.aggr'] ?= 'false'
      options.hive_site['hive.fetch.task.conversion'] ?= 'more'
      options.hive_site['hive.fetch.task.conversion.threshold'] ?= '1073741824'
      options.hive_site['hive.limit.optimize.enable'] ?= 'true'
      options.hive_site['hive.limit.pushdown.memory.usage'] ?= '0.04'
      options.hive_site['hive.map.aggr'] ?= 'true'
      options.hive_site['hive.map.aggr.hash.force.flush.memory.threshold'] ?= '0.9'
      options.hive_site['hive.map.aggr.hash.min.reduction'] ?= '0.5'
      options.hive_site['hive.map.aggr.hash.percentmemory'] ?= '0.5'
      options.hive_site['hive.mapjoin.bucket.cache.size'] ?= '10000'
      options.hive_site['hive.mapjoin.optimized.hashtable'] ?= 'true'
      options.hive_site['hive.mapred.reduce.tasks.speculative.execution'] ?= 'false'
      options.hive_site['hive.merge.mapfiles'] ?= 'true'
      options.hive_site['hive.merge.mapredfiles'] ?= 'false'
      options.hive_site['hive.merge.orcfile.stripe.level'] ?= 'true'
      options.hive_site['hive.merge.rcfile.block.level'] ?= 'true'
      options.hive_site['hive.merge.size.per.task'] ?= '256000000'
      options.hive_site['hive.merge.smallfiles.avgsize'] ?= '16000000'
      options.hive_site['hive.merge.tezfiles'] ?= 'false'
      options.hive_site['hive.metastore.authorization.storage.checks'] ?= 'false'
      options.hive_site['hive.metastore.client.connect.retry.delay'] ?= '5s'
      options.hive_site['hive.metastore.client.socket.timeout'] ?= '1800s'
      options.hive_site['hive.metastore.connect.retries'] ?= '24'
      options.hive_site['hive.metastore.execute.setugi'] ?= 'true'
      options.hive_site['hive.metastore.failure.retries'] ?= '24'
      options.hive_site['hive.metastore.server.max.threads'] ?= '100000'
      options.hive_site['hive.optimize.bucketmapjoin'] ?= 'true'
      options.hive_site['hive.optimize.bucketmapjoin.sortedmerge'] ?= 'false'
      options.hive_site['hive.optimize.constant.propagation'] ?= 'true'
      options.hive_site['hive.optimize.index.filter'] ?= 'true'
      options.hive_site['hive.optimize.metadataonly'] ?= 'true'
      options.hive_site['hive.optimize.null.scan'] ?= 'true'
      options.hive_site['hive.optimize.reducededuplication'] ?= 'true'
      options.hive_site['hive.optimize.reducededuplication.min.reducer'] ?= '4'
      options.hive_site['hive.optimize.sort.dynamic.partition'] ?= 'false'
      options.hive_site['hive.orc.compute.splits.num.threads'] ?= '10'
      options.hive_site['hive.orc.splits.include.file.footer'] ?= 'false'
      options.hive_site['hive.security.metastore.authorization.auth.reads'] ?= 'true'
      options.hive_site['hive.server2.logging.operation.enabled'] ?= 'true'
      options.hive_site['hive.server2.max.start.attempts'] ?= '5'
      options.hive_site['hive.server2.table.type.mapping'] ?= 'CLASSIC'
      options.hive_site['hive.server2.thrift.max.worker.threads'] ?= '500'
      options.hive_site['hive.smbjoin.cache.rows'] ?= '10000'
      options.hive_site['hive.start.cleanup.scratchdir'] ?= 'false'
      options.hive_site['hive.stats.autogather'] ?= 'true'
      options.hive_site['hive.stats.dbclass'] ?= 'fs'
      options.hive_site['hive.stats.fetch.partition.stats'] ?= 'true'
      options.hive_site['hive.tez.auto.reducer.parallelism'] ?= 'true'
      options.hive_site['hive.tez.cpu.vcores'] ?= '-1'
      options.hive_site['hive.tez.dynamic.partition.pruning'] ?= 'true'
      options.hive_site['hive.tez.dynamic.partition.pruning.max.data.size'] ?= '104857600'
      options.hive_site['hive.tez.dynamic.partition.pruning.max.event.size'] ?= '1048576'
      options.hive_site['hive.tez.input.format'] ?= 'org.apache.hadoop.hive.ql.io.HiveInputFormat'
      options.hive_site['hive.tez.log.level'] ?= 'INFO'
      options.hive_site['hive.tez.max.partition.factor'] ?= '2.0'
      options.hive_site['hive.tez.min.partition.factor'] ?= '0.25'
      options.hive_site['hive.tez.smb.number.waves'] ?= '0.5'
      options.hive_site['hive.user.install.directory'] ?= '/user/'
      options.hive_site['hive.vectorized.execution.enabled'] ?= 'true'
      options.hive_site['hive.vectorized.execution.reduce.enabled'] ?= 'false'
      options.hive_site['hive.vectorized.groupby.checkinterval'] ?= '4096'
      options.hive_site['hive.vectorized.groupby.flush.percent'] ?= '0.1'
      options.hive_site['hive.vectorized.groupby.maxentries'] ?= '100000'
      options.hive_site['hive.zookeeper.client.port'] ?= '2181'
      options.hive_site['hive.zookeeper.namespace'] ?= 'hive_zookeeper_namespace'
      # end default - required by ambari

## Storage

      options.hive_site['hive.exec.orc.default.stripe.size'] ?= '67108864'
      options.hive_site['hive.exec.orc.default.compress'] ?= 'SNAPPY' #ZLIB/ZLIB
      

## Database

Import database information from the Hive Metastore

      merge options.hive_site, service.deps.hive_metastore.options.hive_site

## Hive Server2 Environment

      options.opts.base ?= ''
      options.opts.java_properties ?= {}
      options.opts.jvm ?= {}
      options.opts.jvm['-Xms'] ?= options.heapsize
      options.opts.jvm['-Xmx'] ?= options.heapsize
      options.opts.jvm['-XX:NewSize='] ?= options.newsize #should be 1/8 of datanode heapsize
      options.opts.jvm['-XX:MaxNewSize='] ?= options.newsize #should be 1/8 of datanode heapsize
      #JMX Config
      if options.jmx_port?
        throw Error 'Missing options.jmx_rmi_port' unless options.jmx_rmi_port?
        throw Error 'Missing options.jmx_authenticate' unless options.jmx_authenticate?
        throw Error 'Missing options.jmx_ssl' unless options.jmx_ssl?
        options.opts.java_properties['com.sun.management.jmxremote.port'] ?= options.jmx_port
        options.opts.java_properties['com.sun.management.jmxremote.rmi.port'] ?= options.jmx_rmi_port
        options.opts.java_properties['com.sun.management.jmxremote.authenticate'] ?= options.jmx_authenticate
        options.opts.java_properties['com.sun.management.jmxremote.ssl'] ?= options.jmx_ssl
      # fix bug where phoenix-server and phoenix-client do not contain same
      # version of class used.
      options.aux_jars_paths ?= {}
      if service.deps.hbase_client
        options.aux_jars_paths['/usr/hdp/current/hbase-client/lib/hbase-server.jar'] ?= true
        options.aux_jars_paths['/usr/hdp/current/hbase-client/lib/hbase-client.jar'] ?= true
        options.aux_jars_paths['/usr/hdp/current/hbase-client/lib/hbase-common.jar'] ?= true
      if service.deps.phoenix_client
        options.aux_jars_paths['/usr/hdp/current/phoenix-client/phoenix-hive.jar'] ?= true
      for path, val of service.deps.hive_hcatalog[0].options.aux_jars_paths
        options.aux_jars_paths[path] ?= val
      #aux_jars forced by ryba to guaranty consistency
      options.aux_jars = "#{Object.keys(options.aux_jars_paths).join ':'}"

## Kerberos

      # https://cwiki.apache.org/confluence/display/Hive/Setting+up+HiveServer2
      # Authentication type
      options.hive_site['hive.server2.authentication'] ?= 'KERBEROS'
      # The keytab for the HiveServer2 service principal
      # 'options.authentication.kerberos.keytab': "/etc/security/keytabs/hcat.service.keytab"
      options.hive_site['hive.server2.authentication.kerberos.keytab'] ?= '/etc/security/keytabs/hive.service.keytab'
      # The service principal for the HiveServer2. If _HOST
      # is used as the hostname portion, it will be replaced.
      # with the actual hostname of the running instance.
      options.hive_site['hive.server2.authentication.kerberos.principal'] ?= "hive/_HOST@#{options.krb5.realm}"
      # SPNEGO
      options.hive_site['hive.server2.authentication.spnego.principal'] ?= service.deps.hadoop_core.options.core_site['hadoop.http.authentication.kerberos.principal']
      options.hive_site['hive.server2.authentication.spnego.keytab'] ?= service.deps.hadoop_core.options.core_site['hadoop.http.authentication.kerberos.keytab']
      # Ensure we dont create the same principal as with the Hive HCatalog or the kvno will be incremented
      hive_hcatalog_local_srv = service.deps.hive_hcatalog.filter((srv) -> srv.node.id is service.node.id)[0]
      options.principal_identical_to_hcatalog = hive_hcatalog_local_srv and hive_hcatalog_local_srv.options.hive_site['hive.metastore.kerberos.principal'] is options.hive_site['hive.server2.authentication.kerberos.principal']


## SSL

      options.ssl = merge {}, service.deps.hadoop_core.options.ssl, options.ssl
      options.hive_site['hive.server2.use.SSL'] ?= 'true'
      options.hive_site['hive.server2.keystore.path'] ?= "#{options.ssl.conf_dir}/keystore"
      options.hive_site['hive.server2.keystore.password'] ?= service.deps.hadoop_core.options.ssl.keystore.password

## HS2 High Availability & Rolling Upgrade

HS2 use Zookeepper to track registered servers. The znode address is
"/<hs2_namespace>/serverUri=<host:port>;version=<versionInfo>; sequence=<sequence_number>"
and its value is the server "host:port".

      zookeeper_quorum = for srv in service.deps.zookeeper_server
        continue unless srv.options.config['peerType'] is 'participant'
        "#{srv.node.fqdn}:#{srv.options.config['clientPort']}"
      options.hive_site['hive.zookeeper.quorum'] ?= zookeeper_quorum.join ','
      options.hive_site['hive.server2.support.dynamic.service.discovery'] ?= if service.deps.hive_server2.length > 1 then 'true' else 'false'
      options.hive_site['hive.zookeeper.session.timeout'] ?= '600000' # Default is "600000"
      options.hive_site['hive.server2.zookeeper.namespace'] ?= 'hiveserver2' # Default is "hiveserver2"

# Configure Log4J

      options.log4j = merge {}, service.deps.log4j?.options, options.log4j
      options.log4j.properties ?= {}
      options.log4j.properties['hive.log.file'] ?= 'hiveserver2.log'
      options.log4j.properties['hive.log.dir'] ?= "#{options.log_dir}"
      options.log4j.properties['log4j.appender.EventCounter'] ?= 'org.apache.hadoop.hive.shims.HiveEventCounter'
      options.log4j.properties['log4j.appender.console'] ?= 'org.apache.log4j.ConsoleAppender'
      options.log4j.properties['log4j.appender.console.target'] ?= 'System.err'
      options.log4j.properties['log4j.appender.console.layout'] ?= 'org.apache.log4j.PatternLayout'
      options.log4j.properties['log4j.appender.console.layout.ConversionPattern'] ?= '%d{yy/MM/dd HH:mm:ss} %p %c{2}: %m%n'
      options.log4j.properties['log4j.appender.console.encoding'] ?= 'UTF-8'
      options.log4j.properties['log4j.appender.RFAS'] ?= 'org.apache.log4j.RollingFileAppender'
      options.log4j.properties['log4j.appender.RFAS.File'] ?= '${hive.log.dir}/${hive.log.file}'
      options.log4j.properties['log4j.appender.RFAS.MaxFileSize'] ?= '20MB'
      options.log4j.properties['log4j.appender.RFAS.MaxBackupIndex'] ?= '10'
      options.log4j.properties['log4j.appender.RFAS.layout'] ?= 'org.apache.log4j.PatternLayout'
      options.log4j.properties['log4j.appender.RFAS.layout.ConversionPattern'] ?= '%d{ISO8601} %-5p %c{2} - %m%n'
      options.log4j.properties['log4j.appender.DRFA'] ?= 'org.apache.log4j.DailyRollingFileAppender'
      options.log4j.properties['log4j.appender.DRFA.File'] ?= '${hive.log.dir}/${hive.log.file}'
      options.log4j.properties['log4j.appender.DRFA.DatePattern'] ?= '.yyyy-MM-dd'
      options.log4j.properties['log4j.appender.DRFA.layout'] ?= 'org.apache.log4j.PatternLayout'
      options.log4j.properties['log4j.appender.DRFA.layout.ConversionPattern'] ?= '%d{ISO8601} %-5p %c{2} (%F:%M(%L)) - %m%n'
      options.log4j.properties['log4j.appender.DAILY'] ?= 'org.apache.log4j.rolling.RollingFileAppender'
      options.log4j.properties['log4j.appender.DAILY.rollingPolicy'] ?= 'org.apache.log4j.rolling.TimeBasedRollingPolicy'
      options.log4j.properties['log4j.appender.DAILY.rollingPolicy.ActiveFileName'] ?= '${hive.log.dir}/${hive.log.file}'
      options.log4j.properties['log4j.appender.DAILY.rollingPolicy.FileNamePattern'] ?= '${hive.log.dir}/${hive.log.file}.%d{yyyy-MM-dd}'
      options.log4j.properties['log4j.appender.DAILY.layout'] ?= 'org.apache.log4j.PatternLayout'
      options.log4j.properties['log4j.appender.DAILY.layout.ConversionPattern'] ?= '%d{dd MMM yyyy HH:mm:ss,SSS} %-5p [%t] (%C.%M:%L) %x - %m%n'
      options.log4j.properties['log4j.appender.AUDIT'] ?= 'org.apache.log4j.RollingFileAppender'
      options.log4j.properties['log4j.appender.AUDIT.File'] ?= '${hive.log.dir}/hiveserver2_audit.log'
      options.log4j.properties['log4j.appender.AUDIT.MaxFileSize'] ?= '20MB'
      options.log4j.properties['log4j.appender.AUDIT.MaxBackupIndex'] ?= '10'
      options.log4j.properties['log4j.appender.AUDIT.layout'] ?= 'org.apache.log4j.PatternLayout'
      options.log4j.properties['log4j.appender.AUDIT.layout.ConversionPattern'] ?= '%d{ISO8601} %-5p %c{2} (%F:%M(%L)) - %m%n'

      options.log4j.appenders = ',RFAS'
      options.log4j.audit_appenders = ',AUDIT'
      if options.log4j.remote_host and options.log4j.remote_port
        options.log4j.appenders = options.log4j.appenders + ',SOCKET'
        options.log4j.audit_appenders = options.log4j.audit_appenders + ',SOCKET'
        options.log4j.properties['log4j.appender.SOCKET'] ?= 'org.apache.log4j.net.SocketAppender'
        options.log4j.properties['log4j.appender.SOCKET.Application'] ?= 'hiveserver2'
        options.log4j.properties['log4j.appender.SOCKET.RemoteHost'] ?= options.log4j.remote_host
        options.log4j.properties['log4j.appender.SOCKET.Port'] ?= options.log4j.remote_port

      options.log4j.properties['log4j.category.DataNucleus'] ?= 'ERROR' + options.log4j.appenders
      options.log4j.properties['log4j.category.Datastore'] ?= 'ERROR' + options.log4j.appenders
      options.log4j.properties['log4j.category.Datastore.Schema'] ?= 'ERROR' + options.log4j.appenders
      options.log4j.properties['log4j.category.JPOX.Datastore'] ?= 'ERROR' + options.log4j.appenders
      options.log4j.properties['log4j.category.JPOX.Plugin'] ?= 'ERROR' + options.log4j.appenders
      options.log4j.properties['log4j.category.JPOX.MetaData'] ?= 'ERROR' + options.log4j.appenders
      options.log4j.properties['log4j.category.JPOX.Query'] ?= 'ERROR' + options.log4j.appenders
      options.log4j.properties['log4j.category.JPOX.General'] ?= 'ERROR' + options.log4j.appenders
      options.log4j.properties['log4j.category.JPOX.Enhancer'] ?= 'ERROR' + options.log4j.appenders
      options.log4j.properties['log4j.logger.org.apache.hadoop.conf.Configuration'] ?= 'ERROR' + options.log4j.appenders
      options.log4j.properties['log4j.logger.org.apache.zookeeper'] ?= 'INFO' + options.log4j.appenders
      options.log4j.properties['log4j.logger.org.apache.zookeeper.server.ServerCnxn'] ?= 'WARN' + options.log4j.appenders
      options.log4j.properties['log4j.logger.org.apache.zookeeper.server.NIOServerCnxn'] ?= 'WARN' + options.log4j.appenders
      options.log4j.properties['log4j.logger.org.apache.zookeeper.ClientCnxn'] ?= 'WARN' + options.log4j.appenders
      options.log4j.properties['log4j.logger.org.apache.zookeeper.ClientCnxnSocket'] ?= 'WARN' + options.log4j.appenders
      options.log4j.properties['log4j.logger.org.apache.zookeeper.ClientCnxnSocketNIO'] ?= 'WARN' + options.log4j.appenders
      options.log4j.properties['log4j.logger.org.apache.hadoop.hive.ql.log.PerfLogger'] ?= '${hive.ql.log.PerfLogger.level}'
      options.log4j.properties['log4j.logger.org.apache.hadoop.hive.ql.exec.Operator'] ?= 'INFO' + options.log4j.appenders
      options.log4j.properties['log4j.logger.org.apache.hadoop.hive.serde2.lazy'] ?= 'INFO' + options.log4j.appenders
      options.log4j.properties['log4j.logger.org.apache.hadoop.hive.metastore.ObjectStore'] ?= 'INFO' + options.log4j.appenders
      options.log4j.properties['log4j.logger.org.apache.hadoop.hive.metastore.MetaStore'] ?= 'INFO' + options.log4j.appenders
      options.log4j.properties['log4j.logger.org.apache.hadoop.hive.metastore.HiveMetaStore'] ?= 'INFO' + options.log4j.appenders
      options.log4j.properties['log4j.logger.org.apache.hadoop.hive.metastore.HiveMetaStore.audit'] ?= 'INFO' + options.log4j.audit_appenders
      options.log4j.properties['log4j.additivity.org.apache.hadoop.hive.metastore.HiveMetaStore.audit'] ?= false
      options.log4j.properties['log4j.logger.server.AsyncHttpConnection'] ?= 'OFF'
      options.log4j.properties['hive.log.threshold'] ?= 'ALL'
      options.log4j.properties['hive.root.logger'] ?= 'INFO' + options.log4j.appenders
      options.log4j.properties['log4j.rootLogger'] ?= '${hive.root.logger}, EventCounter'
      options.log4j.properties['log4j.threshold'] ?= '${hive.log.threshold}'

# Hive On HBase

Add Hive user as proxyuser

      for srv in service.deps.hdfs
        srv.options.core_site ?= {}
        srv.options.core_site["hadoop.proxyuser.#{options.user.name}.hosts"] ?= '*'
        srv.options.core_site["hadoop.proxyuser.#{options.user.name}.groups"] ?= '*'

## Wait

      options.wait_krb5_client ?= service.deps.krb5_client.options.wait
      options.wait_zookeeper_server ?= service.deps.zookeeper_server[0].options.wait
      options.wait = {}
      options.wait.thrift = for srv in service.deps.hive_server2
        srv.options.hive_site ?= {}
        srv.options.hive_site['hive.server2.transport.mode'] ?= 'http'
        srv.options.hive_site['hive.server2.thrift.http.port'] ?= '10001'
        srv.options.hive_site['hive.server2.thrift.port'] ?= '10001'
        host: srv.node.fqdn
        port: if srv.options.hive_site['hive.server2.transport.mode'] is 'http'
        then srv.options.hive_site['hive.server2.thrift.http.port']
        else srv.options.hive_site['hive.server2.thrift.port']

## Ambari REST API

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name

## Ambari Required Configurations

      options.hiveserver2_site ?= {}
      options.hiveserver2_site['hive.service.metrics.hadoop2.component'] ?= "hiveserver2"
      options.hiveserver2_site['hive.security.authorization.enabled'] ?= "false"
      options.hiveserver2_site['hive.metastore.metrics.enabled'] ?= "true"
      options.hiveserver2_site['hive.service.metrics.reporter'] ?= "JSON_FILE, JMX, HADOOP2"
      options.hiveserver2_site['hive.service.metrics.file.location'] ?= '/var/log/hive/hiveserver2-report.json'

## Ambari Configurations
Enrich `ryba-ambari-takeover/hive/service` with hive/server2 properties.
  
      enrich_config = (source, target) ->
        for k, v of source
          target[k] ?= v
          
      for srv in service.deps.hive
        srv.options.configurations ?= {}
        #hive-site
        srv.options.configurations['hive-site'] ?= {}
        enrich_config options.hive_site, srv.options.configurations['hive-site']
        #hive-env
        srv.options.configurations['hive-env'] ?= {}
        srv.options.configurations['hive-env']['hive.heapsize'] ?= '1024'
        srv.options.configurations['hive-env']['hive_timeline_logging_enabled'] ?= 'true'
        srv.options.configurations['hive-env']['heap_dump_location'] ?= '/tmp'
        # srv.options.configurations['hive-env']['alert_ldap_username'] ?= ''
        # srv.options.configurations['hive-env']['alert_ldap_password'] ?= 'admin123'
        # srv.options.configurations['hive-env']['alert_ldap_url'] ?= ''
        
        srv.options.server2_opts ?= options.opts
        srv.options.server2_aux_jars ? options.aux_jars
        #add hosts
        srv.options.server2_hosts ?= []
        srv.options.server2_hosts.push service.node.fqdn if srv.options.server2_hosts.indexOf(service.node.fqdn) is -1


## Log4j Properties

        srv.options.hive_log4j ?= {}
        enrich_config options.log4j.properties, options.hive_log4j if service.deps.log4j?

## Log4j Properties

        srv.options.configurations['hiveserver2-site'] ?= {}
        enrich_config options.hiveserver2_site, srv.options.configurations['hiveserver2-site']


## Dependencies

    {merge} = require 'nikita/lib/misc'
