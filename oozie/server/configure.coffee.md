
# Oozie Server Configure

*   `oozie.user` (object|string)
    The Unix Oozie login name or a user object (see Nikita User documentation).
*   `oozie.group` (object|string)
    The Unix Oozie group name or a group object (see Nikita Group documentation).

Example

```json
    "oozie": {
      "user": {
        "name": "oozie", "system": true, "gid": "oozie",
        "comment": "Oozie User", "home": "/var/lib/oozie"
      },
      "group": {
        "name": "Oozie", "system": true
      },
      "db": {
        "password": "Oozie123!"
      }
    }
```

    module.exports = (service) ->
      options = service.options

## Identities
      
      options.user = service.deps.oozie[0].options.user
      options.group = service.deps.oozie[0].options.group
      options.hadoop_group ?= service.deps.hdfs[0].options.hadoop_group

## Supported Actions

      # Falcon
      options.has_falcon ?= !!service.deps.falcon_server
      # HBase
      options.hbase ?= {}
      options.hbase.enabled ?= !!service.deps.hbase_master
      options.fqdn ?= service.node.fqdn

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      throw Error 'Required Options: "realm"' unless options.krb5.realm
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]
      # Kerberos HDFS Admin
      options.hdfs_krb5_user = service.deps.hadoop_core.options.hdfs.krb5_user

## Environment

      # Layout
      options.conf_dir ?= '/etc/oozie/conf'
      options.data_dir ?= '/var/db/oozie'
      options.log_dir ?= '/var/log/oozie'
      options.pid_dir ?= '/var/run/oozie'
      options.tmp_dir ?= '/var/tmp/oozie'
      options.server_dir ?= '/usr/hdp/current/oozie-client/oozie-server'
      # Java
      options.heapsize ?= '1024m'
      options.newsize ?= '200m'
      # Misc
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.default_fs ?= service.deps.hdfs_nn[0].options.core_site['fs.defaultFS']
      options.java_home = service.deps.java.options.java_home
      options.hadoop_conf_dir ?= service.deps.hdfs[0].options.conf_dir
      options.hadoop_lib_home ?= '/usr/hdp/current/hadoop-client/lib'
      options.clean_logs ?= false


## Security

      options.ssl = merge {}, service.deps.ssl?.options, service.deps.oozie[0].options.ssl, options.ssl
      options.ssl.enabled ?= !!service.deps.ssl

## Configuration

      options.http_port ?= if options.ssl.enabled then 11443 else 11000
      options.admin_port ?= 11001
      options.oozie_site ?= {}
      options.oozie_site['oozie.base.url'] = if options.ssl.enabled
      then "https://#{service.node.fqdn}:#{options.http_port}/oozie"
      else "http://#{service.node.fqdn}:#{options.http_port}/oozie"

## Database

      options.db ?= {}
      options.db.engine ?= service.deps.db_admin.options.engine
      options.db = merge {}, service.deps.db_admin.options[options.db.engine], options.db
      options.db.database ?= 'oozie'
      options.db.username ?= 'oozie'
      throw Error "Required Option: db.password" unless options.db.password
      #jdbc provided by ryba/commons/db_admin
      #for now only setting the first host as Oozie fails to parse jdbc url.
      #JIRA: [OOZIE-2136]
      options.oozie_site['oozie.service.JPAService.jdbc.url'] ?= "jdbc:mysql://#{options.db.host}:#{options.db.port}/#{options.db.database}?createDatabaseIfNotExist=true"
      options.oozie_site['oozie.service.JPAService.jdbc.driver'] ?= 'com.mysql.jdbc.Driver'
      options.oozie_site['oozie.service.JPAService.jdbc.username'] = options.db.username
      options.oozie_site['oozie.service.JPAService.jdbc.password'] = options.db.password

## Configuration

      # Path to hadoop configuration is required when running 'sharelib upgrade'
      # or an error will complain that the hdfs url is invalid
      options.oozie_site['oozie.services.ext']?= []
      options.oozie_site['oozie.service.HadoopAccessorService.hadoop.configurations'] ?= '*=/etc/hadoop/conf'
      options.oozie_site['oozie.service.SparkConfigurationService.spark.configurations'] ?= '*=/etc/spark/conf/'
      #options.oozie_site['oozie.service.SparkConfigurationService.spark.configurations.ignore.spark.yarn.jar'] ?= 'true'
      options.oozie_site['oozie.service.AuthorizationService.authorization.enabled'] ?= 'true'
      options.oozie_site['oozie.service.HadoopAccessorService.kerberos.enabled'] ?= 'true'
      options.oozie_site['local.realm'] ?= "#{options.krb5.realm}"
      options.oozie_site['oozie.service.HadoopAccessorService.keytab.file'] ?= '/etc/security/keytabs/oozie.service.keytab'
      options.oozie_site['oozie.service.HadoopAccessorService.kerberos.principal'] ?= "oozie/_HOST@#{options.krb5.realm}"
      options.oozie_site['oozie.authentication.type'] ?= 'kerberos'
      options.oozie_site['oozie.authentication.kerberos.principal'] ?= "HTTP/_HOST@#{options.krb5.realm}"
      options.oozie_site['oozie.authentication.kerberos.keytab'] ?= '/etc/security/keytabs/spnego.service.keytab'
      options.oozie_site['oozie.authentication.kerberos.name.rules'] ?= service.deps.hdfs[0].options['configurations']['core-site']['hadoop.security.auth_to_local']
      options.oozie_site['oozie.service.HadoopAccessorService.nameNode.whitelist'] ?= '' # Fix space value
      options.oozie_site['oozie.credentials.credentialclasses'] ?= [
       'hcat=org.apache.oozie.action.hadoop.HCatCredentials'
       'hbase=org.apache.oozie.action.hadoop.HbaseCredentials'
       'hive2=org.apache.oozie.action.hadoop.Hive2Credentials'
      ]
      # Spark and Shell action dedicated configuration in each yarn container
      # To benefit from that feature in a ShellAction, one must specify the --config parameter
      # with the HADOOP_CONF_DIR env variable set by Oozie at runtime
      # eg : hadoop --config $HADOOP_CONF_DIR fs -ls /
      # see also OOZIE-2343, OOZIE-2481, OOZIE-2569 and OOZIE-2504, fixed by OOZIE-2739
      options.oozie_site['oozie.action.spark.setup.hadoop.conf.dir'] ?= 'true'
      options.oozie_site['oozie.action.shell.setup.hadoop.conf.dir'] ?= 'true'
      options.oozie_site['oozie.action.shell.setup.hadoop.conf.dir.write.log4j.properties'] ?= 'true'
      options.oozie_site['oozie.action.shell.setup.hadoop.conf.dir.log4j.content'] ?= '''
      log4j.rootLogger=INFO,console
      log4j.appender.console=org.apache.log4j.ConsoleAppender
      log4j.appender.console.target=System.err
      log4j.appender.console.layout=org.apache.log4j.PatternLayout
      log4j.appender.console.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{2}: %m%n
      '''
      # Sharelib add-ons
      options.upload_share_lib = service.instances[0].node.id is service.node.id
      options.sharelib ?= {}
      options.sharelib.distcp ?= []
      options.sharelib.hcatalog ?= []
      options.sharelib.hive ?= []
      options.sharelib.hive2 ?= []
      options.sharelib.mrstreaming ?= []
      options.sharelib.oozie ?= []
      options.sharelib.pig ?= []
      options.sharelib.spark ?= []
      options.sharelib.sqoop ?= []
      # https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.5.0/bk_command-line-upgrade/content/start-oozie-23.html
      # AMBARI-18383
      options.sharelib.spark.push '/usr/hdp/current/spark-client/lib/datanucleus-api-jdo-3.2.6.jar'
      options.sharelib.spark.push '/usr/hdp/current/spark-client/lib/datanucleus-core-3.2.10.jar'
      options.sharelib.spark.push '/usr/hdp/current/spark-client/lib/datanucleus-rdbms-3.2.9.jar'
      options.sharelib.spark.push '/usr/hdp/current/spark-client/lib/spark-assembly-1.6.2.2.5.3.0-37-hadoop2.7.3.2.5.3.0-37.jar'
      options.sharelib.spark.push '/usr/hdp/current/spark-client/python/lib/pyspark.zip'
      options.sharelib.spark.push '/usr/hdp/current/spark-client/python/lib/py4j-0.9-src.zip'
      # Oozie Notifications
      # see https://oozie.apache.org/docs/4.1.0/AG_Install.html#Notifications_Configuration
      if options.jms_url
        options.oozie_site['oozie.services.ext'].push [
          'org.apache.oozie.service.JMSAccessorService'
          'org.apache.oozie.service.JMSTopicService'
          'org.apache.oozie.service.EventHandlerService'
          'org.apache.oozie.sla.service.SLAService'
          ]
        options.oozie_site['oozie.service.EventHandlerService.event.listeners'] ?= [
          'org.apache.oozie.jms.JMSJobEventListener'
          'org.apache.oozie.sla.listener.SLAJobEventListener'
          'org.apache.oozie.jms.JMSSLAEventListener'
          'org.apache.oozie.sla.listener.SLAEmailEventListener'
          ]
        options.oozie_site['oozie.service.SchedulerService.threads'] ?= 15
        options.oozie_site['oozie.jms.producer.connection.properties'] ?= "java.naming.factory.initial#org.apache.activemq.jndi.ActiveMQInitialContextFactory;java.naming.provider.url#"+"#{options.jms_url}"+";connectionFactoryNames#ConnectionFactory"
        #options.oozie_site['oozie.service.JMSTopicService.topic.prefix'] ?= 'oozie.' # despite the docs, this parameter does not exist
        options.oozie_site['oozie.service.JMSTopicService.topic.name'] ?= [
          'default=${username}'
          'WORKFLOW=workflow'
          'COORDINATOR=coordinator'
          'BUNDLE=bundle'
          ].join(',')

## Proxy Users

Hive Hcatalog, Hive Server2 and HBase retrieve their proxy users from the
hdfs_client configuration directory.

      enrich_proxy_user = (srv) ->
        srv.options.configurations['core-site']["hadoop.proxyuser.#{options.user.name}.groups"] ?= '*'
        hosts = srv.options.configurations['core-site']["hadoop.proxyuser.#{options.user.name}.hosts"] or []
        hosts = hosts.split ',' unless Array.isArray hosts
        for instance in service.instances
          hosts.push instance.node.fqdn unless instance.node.fqdn in hosts
        hosts = hosts.join ','
        srv.options.configurations['core-site']["hadoop.proxyuser.#{options.user.name}.hosts"] ?= hosts
      enrich_proxy_user srv for srv in service.deps.hdfs
      # migration: lucasbak 13112017
      # need hdfs_client for proxy_user
      # service.deps.hdfs_client.filter (srv) -> console.log service.node.id
      # .filter (srv) ->
      #   srv.node.id is
      #  srv for srv in service.deps.hive_server2

# ## Configuration for Hadoop
# 
#       options.hadoop_config ?= {}
#       options.hadoop_config['mapreduce.jobtracker.kerberos.principal'] ?= "mapred/_HOST@#{options.krb5.realm}"
#       options.hadoop_config['yarn.resourcemanager.principal'] ?= "yarn/_HOST@#{options.krb5.realm}"
#       options.hadoop_config['dfs.namenode.kerberos.principal'] ?= "hdfs/_HOST@#{options.krb5.realm}"
#       options.hadoop_config['mapreduce.framework.name'] ?= "yarn"

## Configuration for Log4J

      options.log4j = merge {}, service.deps.log4j?.options, options.log4j
      options.log4j.opts ?= {}# used to set variable in oozie-env.sh
      if options.log4j.server_port?
        options.log4j.opts['extra_appender'] = ",socket_server"
        options.log4j.opts['server_port'] = options.log4j.server_port
      if options.log4j.remote_host? && options.log4j.remote_port?
        options.log4j.opts['extra_appender'] = ",socket_client"
        options.log4j.opts.remote_host ?= options.log4j.remote_host
        options.log4j.opts.remote_port ?= options.log4j.remote_port
      options.log4j_opts = ""
      options.log4j_opts += " -Doozie.log4j.#{k}=#{v}" for k, v of options.log4j.opts

## High Availability
Config [High Availability][oozie-ha]. They should be configured against
the same database. It uses zookeeper for enabling HA.

      options.ha = if service.deps.zookeeper_server.length > 1 then true else false
      if options.ha
        quorum = service.deps.zookeeper_server
        .filter (srv) -> srv.options.config['peerType'] is 'participant'
        .map (srv) -> "#{srv.node.fqdn}:#{srv.options.config['clientPort']}"
        .join ','
        # quorum = for srv in service.deps.zookeeper_server
        #   continue unless  srv.options.config['peerType'] is 'participant'
        #   "#{srv.node.fqdn}:#{srv.options.config['clientPort']}"
        options.oozie_site['oozie.zookeeper.connection.string'] ?= quorum
        options.oozie_site['oozie.zookeeper.namespace'] ?= 'oozie-ha'
        options.oozie_site['oozie.services.ext'].push [
          'org.apache.oozie.service.ZKLocksService'
          'org.apache.oozie.service.ZKXLogStreamingService'
          'org.apache.oozie.service.ZKJobsConcurrencyService'
          'org.apache.oozie.service.ZKUUIDService'
        ]
      options.oozie_site['oozie.instance.id'] ?= service.node.fqdn
      #ACL On zookeeper
      options.oozie_site['oozie.zookeeper.secure'] ?= 'true'
      options.oozie_site['oozie.service.ZKUUIDService.jobid.sequence.max'] ?= '99999999990'

## Ambari REST API

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.stack_version ?= service.deps.ambari_server.options.stack_version
      options.stack_name ?= service.deps.ambari_server.options.stack_name
      options.takeover = service.deps.ambari_server.options.takeover

## Ambari Oozie Configuration
Enrich `ryba-ambari-takeover/oozie/service` with master properties.
  
      enrich_config = (source, target) ->
        for k, v of source
          target[k] ?= v
      
      for srv in service.deps.oozie
        srv.options.configurations ?= {}
        srv.options.configurations['oozie-site'] ?= {}
        enrich_config options.oozie_site , srv.options.configurations['oozie-site']
        #add hosts
        srv.options.server_hosts ?= []
        srv.options.server_hosts.push options.fqdn if srv.options.server_hosts.indexOf(options.fqdn) is -1

## Ambari Oozie System Options
      
        # Env
        srv.options.configurations['oozie-env'] ?= {}
        # Ambari required
        srv.options.configurations['oozie-env']['oozie_admin_port'] ?= options.admin_port
        srv.options.configurations['oozie-env']['oozie_data_dir'] ?= options.data_dir
        srv.options.configurations['oozie-env']['oozie_pid_dir'] ?= options.pid_dir
        srv.options.configurations['oozie-env']['oozie_log_dir'] ?= options.log_dir
        srv.options.configurations['oozie-env']['oozie_tmp_dir'] ?= options.tmp_dir
        srv.options.configurations['oozie-env']['oozie_user'] ?= options.user.name
        srv.options.configurations['oozie-env']['oozie_user_nofile_limit'] ?= options.user.limits.nofile
        srv.options.configurations['oozie-env']['oozie_user_nproc_limit'] ?= options.user.limits.nproc
        srv.options.configurations['oozie-env']['oozie_database'] ?= 'Existing MySQL / MariaDB Database'
        srv.options.configurations['oozie-env']['oozie_heapsize'] ?= options.heapsize
        srv.options.configurations['oozie-env']['oozie_permsize'] ?= options.newsize
        # srv.options.configurations['oozie-env']['oozie_log_maxhistory'] ?= options.log_dir
        # opts
        #enrich_config options.opts , srv.options.master_opts

## Kerberos Descriptor

        srv.options.identities ?= {}
        srv.options.identities['oozie'] ?= {}
        srv.options.identities['oozie']['principal'] ?= {}
        srv.options.identities['oozie']['principal']['configuration'] ?= 'oozie-site/oozie.service.HadoopAccessorService.kerberos.principal'
        srv.options.identities['oozie']['principal']['type'] ?= 'user'
        srv.options.identities['oozie']['principal']['local_username'] ?= options.user.name
        srv.options.identities['oozie']['principal']['value'] ?= 'oozie/_HOST@${realm}'#options.spark.krb5_user.principal
        srv.options.identities['oozie']['name'] ?= 'oozie_server'
        srv.options.identities['oozie']['keytab'] ?= {}
        srv.options.identities['oozie']['keytab']['owner'] ?= {}
        srv.options.identities['oozie']['keytab']['owner']['access'] ?= 'r' 
        srv.options.identities['oozie']['keytab']['owner']['name'] ?= options.user.name 
        srv.options.identities['oozie']['keytab']['group'] ?= {}
        srv.options.identities['oozie']['keytab']['group']['access'] ?= 'r'
        srv.options.identities['oozie']['keytab']['group']['name'] ?= options.hadoop_group.name
        srv.options.identities['oozie']['keytab']['file'] ?= options.oozie_site['oozie.service.HadoopAccessorService.keytab.file']
        srv.options.identities['oozie']['keytab']['configuration'] ?= 'oozie-site/oozie.service.HadoopAccessorService.keytab.file'

## system Options

        srv.options.hadoop_lib_home ?= options.hadoop_lib_home

# ## Ambari Oozie Log4j Properties
# 
#         srv.options.hbase_log4j ?= {}
#         enrich_config options.log4j.properties, options.hbase_log4j if service.deps.metrics?

## Wait

      options.wait_krb5_client = service.deps.krb5_client.options.wait
      options.wait_zookeeper_server = service.deps.zookeeper_server[0].options.wait
      options.wait_hdfs_nn = service.deps.hdfs_nn[0].options.wait
      options.wait_hbase_master = service.deps.hbase_master[0].options.wait
      options.wait_hive_hcatalog = service.deps.hive_hcatalog[0].options.wait
      options.wait_hive_server2 = service.deps.hive_server2[0].options.wait
      options.wait_hive_webhcat = service.deps.hive_webhcat[0].options.wait
      options.wait = {}
      options.wait.http = for srv in service.deps.oozie_server
        # {fqdn, port} = url.parse oozie_ctx.config.ryba.oozie.site['oozie.base.url']
        host: srv.node.fqdn
        port: srv.options.http_port or if srv.options.ssl?.enabled or options.ssl.enabled then 11443 else 11000

## Dependencies

    {merge} = require 'nikita/lib/misc'

[oozie-ha]:(https://oozie.apache.org/docs/4.2.0/AG_Install.html#High_Availability_HA)
