

    module.exports = (service) ->
      options = service.options

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      throw Error 'Required Options: "realm"' unless options.krb5.realm
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]


## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      throw Error 'Required Options: "realm"' unless options.krb5.realm
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

## Identities

      # Hadoop Group
      options.hadoop_group = merge {}, service.deps.hadoop_core.options.hadoop_group, options.hadoop_group
      options.group = merge service.deps.hbase[0].options.group, options.group
      options.user = merge service.deps.hbase[0].options.user, options.user

## Identities Kerberos

      # Kerberos HDFS Admin
      options.admin = merge service.deps.hbase[0].options.admin, options.admin
      options.hdfs_krb5_user = service.deps.hdfs[0].options.hdfs.krb5_user
      # Kerberos Test Principal
      options.test_krb5_user ?= service.deps.test_user.options.krb5.user

## Environment

      # Layout
      options.conf_dir ?= service.deps.hbase[0].options.conf_dir
      options.log_dir ?= service.deps.hbase[0].options.log_dir
      options.pid_dir ?= service.deps.hbase[0].options.pid_dir
      # Env
      options.env ?= {}
      options.env['HBASE_LOG_DIR'] ?= "#{options.log_dir}"
      options.env['HBASE_OPTS'] ?= '-XX:+UseConcMarkSweepGC ' # -XX:+CMSIncrementalMode is deprecated
      # Java
      # 'HBASE_MASTER_OPTS' ?= '-Xmx2048m' # Default in HDP companion file
      options.java_home ?= "#{service.deps.java.options.java_home}"
      options.heapsize ?= '1024m'
      options.newsize ?= '200m'
      # Misc
      options.fqdn ?= service.node.fqdn
      options.hostname = service.node.hostname
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.clean_logs ?= false
      # HDFS
      options.hdfs_conf_dir ?= service.deps.hadoop_core.options.conf_dir


## System Options

      options.opts ?= {}
      options.opts.base ?= ''
      options.opts.java_properties ?= {}
      options.opts.jvm ?= {}
      # options.opts.jvm['-Xms'] ?= options.heapsize
      # options.opts.jvm['-Xmx'] ?= options.heapsize
      # options.opts.jvm['-XX:NewSize='] ?= options.newsize #should be 1/8 of hbase regionserver heapsize
      # options.opts.jvm['-XX:MaxNewSize='] ?= options.newsize #should be 1/8 of hbase regionserver heapsize

## Registration

      for srv in service.deps.hbase_master
        srv.options.regionservers[service.node.fqdn] ?= true

      for srv in service.deps.hbase_regionserver
        srv.options.regionservers ?= {}
        srv.options.regionservers[service.node.fqdn] ?= true

## Configuration

      options.hbase_site ?= {}
      options.hbase_site['hbase.regionserver.port'] ?= '60020'
      options.hbase_site['hbase.regionserver.info.port'] ?= '60030'
      options.hbase_site['hbase.ssl.enabled'] ?= 'true'
      options.hbase_site['hbase.regionserver.handler.count'] ?= 60 # HDP default

## Security

      options.hbase_site['hbase.security.authentication'] ?= service.deps.hbase_master[0].options.hbase_site['hbase.security.authentication']
      if options.hbase_site['hbase.security.authentication'] is 'kerberos'
        options.hbase_site['hbase.master.kerberos.principal'] = service.deps.hbase_master[0].options.hbase_site['hbase.master.kerberos.principal']
        options.hbase_site['hbase.regionserver.kerberos.principal'] ?= service.deps.hbase_master[0].options.hbase_site['hbase.regionserver.kerberos.principal']
        options.hbase_site['hbase.regionserver.keytab.file'] ?= '/etc/security/keytabs/hbase.service.keytab'
        options.hbase_site['hbase.security.authentication.ui'] ?= 'kerberos'
        options.hbase_site['hbase.security.authentication.spnego.kerberos.principal'] ?= "HTTP/_HOST@#{options.krb5.realm}"
        options.hbase_site['hbase.security.authentication.spnego.kerberos.keytab'] ?= service.deps.hadoop_core.options.core_site['hadoop.http.authentication.kerberos.keytab']
      options.hbase_site['hbase.regionserver.global.memstore.upperLimit'] = null # Deprecated from HDP 2.3
      options.hbase_site['hbase.regionserver.global.memstore.size'] = '0.4' # Default in HDP Companion Files
      options.hbase_site['hbase.coprocessor.region.classes'] =  service.deps.hbase_master[0].options.hbase_site['hbase.coprocessor.region.classes']
      # Jaas file
      # options.opts.java_properties['java.security.auth.login.config'] ?= "#{options.conf_dir}/hbase_regionserver_jaas.conf"
      # Copy the Master keytab if a colocalised Master is found and if their principals are equal.
      local_master = service.deps.hbase_master.filter( (srv) -> srv.node.fqdn is service.node.fqdn)[0]
      are_principal_equal = local_master.options.hbase_site['hbase.master.kerberos.principal'] is options.hbase_site['hbase.regionserver.kerberos.principal'] if local_master
      if local_master and are_principal_equal
        options.copy_master_keytab ?= local_master.options.hbase_site['hbase.master.keytab.file']

## Configuration Distributed mode

      for property in [
        'zookeeper.znode.parent'
        'zookeeper.session.timeout'
        'hbase.cluster.distributed'
        'hbase.rootdir'
        'hbase.zookeeper.quorum'
        'hbase.zookeeper.property.clientPort'
        'dfs.domain.socket.path'
      ] then options.hbase_site[property] ?= service.deps.hbase_master[0].options.hbase_site[property]

## Configuration for HA Reads

HA properties must be available to masters and regionservers.

      properties = [
        'hbase.regionserver.storefile.refresh.period'
        'hbase.regionserver.meta.storefile.refresh.period'
        'hbase.region.replica.replication.enabled'
        'hbase.master.hfilecleaner.ttl'
        'hbase.master.loadbalancer.class'
        'hbase.meta.replica.count'
        'hbase.region.replica.wait.for.primary.flush'
        'hbase.region.replica.storefile.refresh.memstore.multiplier'
      ]
      for property in properties then options.hbase_site[property] ?= service.deps.hbase_master[0].options.hbase_site[property]

## Configuration for security

      options.hbase_site['hbase.security.authorization'] ?= service.deps.hbase_master[0].options.hbase_site['hbase.security.authorization']
      options.hbase_site['hbase.rpc.engine'] ?= service.deps.hbase_master[0].options.hbase_site['hbase.rpc.engine']
      options.hbase_site['hbase.superuser'] ?= service.deps.hbase_master[0].options.hbase_site['hbase.superuser']
      options.hbase_site['hbase.bulkload.staging.dir'] ?= service.deps.hbase_master[0].options.hbase_site['hbase.bulkload.staging.dir']

## Configuration Quota

      options.hbase_site['hbase.quota.enabled'] ?= service.deps.hbase_master[0].options.hbase_site['hbase.quota.enabled']
      options.hbase_site['hbase.quota.refresh.period'] ?= service.deps.hbase_master[0].options.hbase_site['hbase.quota.refresh.period']

## Configuration for metrics

Metrics information are entirely derived from the Master.

      options.metrics ?= service.deps.hbase_master[0].options.metrics

## Configuration for Log4J

      options.log4j = merge {}, service.deps.log4j?.options, options.log4j

      options.opts.java_properties['hbase.security.log.file'] ?= 'SecurityAuth-Regional.audit'
      #HBase bin script use directly environment bariables
      options.env['HBASE_ROOT_LOGGER'] ?= 'INFO,RFA'
      options.env['HBASE_SECURITY_LOGGER'] ?= 'INFO,RFAS'
      if options.log4j.remote_host? and options.log4j.remote_port?
        # adding SOCKET appender
        options.log4j.socket_client ?= "SOCKET"
        # Root logger
        if options.env['HBASE_ROOT_LOGGER'].indexOf(options.log4j.socket_client) is -1
        then options.env['HBASE_ROOT_LOGGER'] += ",#{options.log4j.socket_client}"
        # Security Logger
        if options.env['HBASE_SECURITY_LOGGER'].indexOf(options.log4j.socket_client) is -1
        then options.env['HBASE_SECURITY_LOGGER']+= ",#{options.log4j.socket_client}"

        options.opts.java_properties['hbase.log.application'] = 'hbase-regionserver'
        options.opts.java_properties['hbase.log.remote_host'] = options.log4j.remote_host
        options.opts.java_properties['hbase.log.remote_port'] = options.log4j.remote_port

        options.log4j.socket_opts ?=
          Application: '${hbase.log.application}'
          RemoteHost: '${hbase.log.remote_host}'
          Port: '${hbase.log.remote_port}'
          ReconnectionDelay: '10000'

        options.log4j.properties = merge options.log4j.properties, appender
          type: 'org.apache.log4j.net.SocketAppender'
          name: options.log4j.socket_client
          logj4: options.log4j.properties
          properties: options.log4j.socket_opts

## Wait

      options.wait_krb5_client = service.deps.krb5_client.options.wait
      options.wait_zookeeper_server = service.deps.zookeeper_server[0].options.wait
      options.wait_hdfs_nn = service.deps.hdfs_nn[0].options.wait
      options.wait_hbase_master = service.deps.hbase_master[0].options.wait
      options.wait = {}
      for srv in service.deps.hbase_regionserver
        srv.options.hbase_site ?= {}
        srv.options.hbase_site['hbase.regionserver.port'] ?= '60020'
        srv.options.hbase_site['hbase.regionserver.info.port'] ?= '60030'
      options.wait.rpc = for srv in service.deps.hbase_regionserver
        host: srv.node.fqdn
        port: srv.options.hbase_site['hbase.regionserver.port']
      options.wait.info = for srv in service.deps.hbase_regionserver
        host: srv.node.fqdn
        port: srv.options.hbase_site['hbase.regionserver.info.port']

## Ambari REST API

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.takeover = service.deps.ambari_server.options.takeover
      options.baremetal = service.deps.ambari_server.options.baremetal

## HBase Configuration
Enrich `ryba-ambari-takeover/hbase/master` with regionservers properties.
  
      enrich_config = (source, target) ->
        for k, v of source
          target[k] ?= v
      
      for srv in service.deps.hbase
        srv.options.configurations ?= {}
        srv.options.configurations['hbase-site'] ?= {}
        enrich_config options.hbase_site , srv.options.configurations['hbase-site']
        #add hosts
        srv.options.regionserver_hosts ?= []
        srv.options.regionserver_hosts.push options.fqdn if srv.options.regionserver_hosts.indexOf(options.fqdn) is -1

## System Options
      
        # Env
        srv.options.configurations['hbase-env'] ?= {}
        # Ambari required
        srv.options.configurations['hbase-env']['regionserver_heapsize'] ?= options.heapsize
        # opts
        enrich_config options.opts , srv.options.regionserver_opts

## Log4j Properties

        srv.options.hbase_log4j ?= {}
        enrich_config options.log4j.properties, options.hbase_log4j if service.deps.metrics?

## Dependencies

    appender = require 'ryba/lib/appender'
    {merge} = require 'nikita/lib/misc'
