
# Hadoop YARN Timeline Server Configure

```json
{ "ryba": { "yarn": { "ats": {
  "opts": "",
  "heapsize": "1024"
} } } }
```

    module.exports = (service) ->
      options = service.options

## Identities

      options.hadoop_group = merge {}, service.deps.hadoop_core.options.hadoop_group, options.hadoop_group
      options.group = merge {}, service.deps.hadoop_core.options.yarn.group, options.group
      options.user = merge {}, service.deps.hadoop_core.options.yarn.user, options.user

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      throw Error 'Required Options: "realm"' unless options.krb5.realm
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]
      
## Environment

      # Layout
      options.home ?= '/usr/hdp/current/hadoop-yarn-timelineserver'
      options.log_dir ?= service.deps.yarn[0].options.yarn.log_dir
      options.pid_dir ?= service.deps.yarn[0].options.yarn.pid_dir
      options.conf_dir ?= '/etc/hadoop/conf'
      # Java
      options.java_home ?= service.deps.java.options.java_home
      options.heapsize ?= '1024m'
      options.newsize ?= '200m'
      # Misc
      options.fqdn = service.node.fqdn
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.hdfs_krb5_user = service.deps.hdfs_nn[0].options.hdfs_krb5_user

## System Options

      options.opts ?= {}
      options.opts.base ?= ''
      options.opts.java_properties ?= {}
      options.opts.jvm ?= {}
      # options.opts.jvm['-Xms'] ?= options.heapsize
      # options.opts.jvm['-Xmx'] ?= options.heapsize
      # options.opts.jvm['-XX:NewSize='] ?= options.newsize #should be 1/8 of heapsize
      # options.opts.jvm['-XX:MaxNewSize='] ?= options.newsize #should be 1/8 of heapsize

## Configuration

      # Hadoop core "core-site.xml"
      options.core_site = merge {}, service.deps.hdfs_client[0].options.core_site, options.core_site or {}
      # HDFS client "hdfs-site.xml"
      options.hdfs_site = merge {}, service.deps.hdfs_client[0].options.hdfs_site, options.hdfs_site or {}
      # Yarn ATS "yarn-site.xml"
      options.yarn_site ?= {}
      # The hostname of the Timeline service web application.
      options.yarn_site['yarn.timeline-service.hostname'] ?= service.node.fqdn
      options.yarn_site['yarn.http.policy'] ?= 'HTTPS_ONLY' # HTTP_ONLY or HTTPS_ONLY or HTTP_AND_HTTPS
      # Advanced Configuration
      options.yarn_site['yarn.timeline-service.address'] ?= "#{service.node.fqdn}:10200"
      options.yarn_site['yarn.timeline-service.webapp.address'] ?= "#{service.node.fqdn}:8188"
      options.yarn_site['yarn.timeline-service.webapp.https.address'] ?= "#{service.node.fqdn}:8190"
      options.yarn_site['yarn.timeline-service.handler-thread-count'] ?= "100" # HDP default is "10"
      options.yarn_site['yarn.timeline-service.http-cross-origin.enabled'] ?= "true"
      options.yarn_site['yarn.timeline-service.http-cross-origin.allowed-origins'] ?= "*"
      options.yarn_site['yarn.timeline-service.http-cross-origin.allowed-methods'] ?= "GET,POST,HEAD"
      options.yarn_site['yarn.timeline-service.http-cross-origin.allowed-headers'] ?= "X-Requested-With,Content-Type,Accept,Origin"
      options.yarn_site['yarn.timeline-service.http-cross-origin.max-age'] ?= "1800"
      protocol = if options.yarn_site['yarn.http.policy'] is 'HTTP_ONLY' then '' else 'https.'
      options.yarn_site['yarn.log.server.web-service.url'] ?= if options.yarn_site['yarn.http.policy'] is 'HTTP_ONLY' 
      then 'http://' + options.yarn_site["yarn.timeline-service.webapp.#{protocol}address"] + '/ws/v1/applicationhistory'
      else 'https://' + options.yarn_site["yarn.timeline-service.webapp.#{protocol}address"] + '/ws/v1/applicationhistory'
      # Generic-data related Configuration
      # Yarn doc: yarn.timeline-service.generic-application-history.enabled = false
      options.yarn_site['yarn.timeline-service.generic-application-history.enabled'] ?= 'true'
      options.yarn_site['yarn.timeline-service.generic-application-history.save-non-am-container-meta-info'] ?= 'true'
      options.yarn_site['yarn.timeline-service.generic-application-history.store-class'] ?= "org.apache.hadoop.yarn.server.applicationhistoryservice.FileSystemApplicationHistoryStore"
      options.yarn_site['yarn.timeline-service.fs-history-store.uri'] ?= '/apps/ats' # Not documented, default to "$(hadoop.tmp.dir)/yarn/timeline/generic-history""
      options.yarn_site['yarn.timeline-service.generic-application-history.fs-history-store.uri'] ?= "#{options.yarn_site['yarn.timeline-service.fs-history-store.uri']}/generic-history/ApplicationHistoryDataRoot" #Not documented , default to /$(hadoop.tmp.dir)/yarn/timeline/generic-history/ApplicationHistoryDataRoot
      # Enabling Generic Data Collection (HDP specific)
      options.yarn_site['yarn.resourcemanager.system-metrics-publisher.enabled'] ?= "true"
      # Per-framework-date related Configuration
      # Indicates to clients whether or not the Timeline Server is enabled. If
      # it is enabled, the TimelineClient library used by end-users will post
      # entities and events to the Timeline Server.
      options.yarn_site['yarn.timeline-service.enabled'] ?= "true"
      # Timeline Server Store
      options.yarn_site['yarn.timeline-service.store-class'] ?= "org.apache.hadoop.yarn.server.timeline.LeveldbTimelineStore"
      options.yarn_site['yarn.timeline-service.leveldb-timeline-store.path'] ?= "/var/yarn/timeline"
      options.yarn_site['yarn.timeline-service.leveldb-state-store.path'] ?= '/var/yarn/timeline'
      options.yarn_site['yarn.timeline-service.ttl-enable'] ?= "true"
      options.yarn_site['yarn.timeline-service.ttl-ms'] ?= "#{604800000 * 2}" # 14 days, HDP default is "604800000"
      # Kerberos Authentication
      console.log 'TODO yarn ts princ/keytab'
      options.yarn_site['yarn.timeline-service.principal'] ?= "yarn/_HOST@#{options.krb5.realm}"
      options.yarn_site['yarn.timeline-service.keytab'] ?= '/etc/security/keytabs/yarn.service.keytab'
      options.yarn_site['yarn.timeline-service.http-authentication.type'] ?= "kerberos"
      options.yarn_site['yarn.timeline-service.http-authentication.kerberos.principal'] ?= "HTTP/_HOST@#{options.krb5.realm}"
      options.yarn_site['yarn.timeline-service.http-authentication.kerberos.keytab'] ?= options.core_site['hadoop.http.authentication.kerberos.keytab']
      # Timeline Server Authorization (ACLs)
      options.yarn_site['yarn.acl.enable'] ?= "true"

## Ambari Kerberos Principal and Keytab

      service.deps.ambari_server.options.identities ?= {}
      service.deps.ambari_server.options.identities ?= {}
      service.deps.ambari_server.options.identities['app_timeline_server_yarn'] ?= {}
      service.deps.ambari_server.options.identities['app_timeline_server_yarn']['principal'] ?= {}
      service.deps.ambari_server.options.identities['app_timeline_server_yarn']['principal']['configuration'] ?= 'yarn-site/yarn.timeline-service.principal'
      service.deps.ambari_server.options.identities['app_timeline_server_yarn']['principal']['type'] ?= 'service'
      service.deps.ambari_server.options.identities['app_timeline_server_yarn']['principal']['local_username'] ?= options.user.name
      service.deps.ambari_server.options.identities['app_timeline_server_yarn']['principal']['value'] ?= options.yarn_site['yarn.timeline-service.principal'] 
      service.deps.ambari_server.options.identities['app_timeline_server_yarn']['name'] ?= 'app_timeline_server_yarn'
      service.deps.ambari_server.options.identities['app_timeline_server_yarn']['keytab'] ?= {}
      service.deps.ambari_server.options.identities['app_timeline_server_yarn']['keytab']['owner'] ?= {}
      service.deps.ambari_server.options.identities['app_timeline_server_yarn']['keytab']['owner']['access'] ?= 'r' 
      service.deps.ambari_server.options.identities['app_timeline_server_yarn']['keytab']['owner']['name'] ?= options.user.name 
      service.deps.ambari_server.options.identities['app_timeline_server_yarn']['keytab']['group'] ?= {}
      service.deps.ambari_server.options.identities['app_timeline_server_yarn']['keytab']['group']['access'] ?= 'r'
      service.deps.ambari_server.options.identities['app_timeline_server_yarn']['keytab']['group']['name'] ?= options.group.name
      service.deps.ambari_server.options.identities['app_timeline_server_yarn']['keytab']['file'] ?= options.yarn_site['yarn.timeline-service.keytab'] 
      service.deps.ambari_server.options.identities['app_timeline_server_yarn']['keytab']['configuration'] ?= 'yarn-site/yarn.timeline-service.keytab'

## YARN ATS 1.5

      options.yarn_site['yarn.timeline-service.version'] ?= '1.0'
      if options.yarn_site['yarn.timeline-service.version'] is '1.5'
        options.yarn_site['yarn.timeline-service.store-class'] = 'org.apache.hadoop.yarn.server.timeline.EntityGroupFSTimelineStore'
        options.yarn_site['yarn.timeline-service.entity-group-fs-store.active-dir'] ?= '/ats/active/'
        options.yarn_site['yarn.timeline-service.entity-group-fs-store.done-dir'] ?= '/ats/done'
        options.yarn_site['yarn.timeline-service.entity-group-fs-store.group-id-plugin-classes'] ?= 'org.apache.tez.dag.history.logging.ats.TimelineCachePluginImpl'
        options.yarn_site['yarn.timeline-service.entity-group-fs-store.summary-store'] ?= 'org.apache.hadoop.yarn.server.timeline.RollingLevelDBTimelineStore'


## SSL

      options.ssl = merge {}, service.deps.hadoop_core.options.ssl, options.ssl
      options.ssl_server = merge {}, service.deps.hadoop_core.options.ssl_server, options.ssl_server or {},
        'ssl.server.keystore.location': "#{options.conf_dir}/keystore"
        'ssl.server.truststore.location': "#{options.conf_dir}/truststore"
      options.ssl_client = merge {}, service.deps.hadoop_core.options.ssl_client, options.ssl_client or {},
        'ssl.client.truststore.location': "#{options.conf_dir}/truststore"

## Metrics

      options.metrics = merge {}, service.deps.hadoop_core.options.metrics, options.metrics

## Export to Yarn NodeManager

      for srv in service.deps.yarn_nm
        for property in [
          'yarn.timeline-service.enabled'
          'yarn.timeline-service.address'
          'yarn.timeline-service.webapp.address'
          'yarn.timeline-service.webapp.https.address'
          'yarn.log.server.web-service.url'
          'yarn.timeline-service.principal'
          'yarn.timeline-service.http-authentication.type'
          'yarn.timeline-service.http-authentication.kerberos.principal'
        ]
          srv.options.yarn_site ?= {}
          srv.options.yarn_site[property] ?= options.yarn_site[property]

## Wait

      options.wait_krb5_client = service.deps.krb5_client.options.wait
      options.wait_hdfs_nn = service.deps.hdfs_nn[0].options.wait
      options.wait = {}
      options.wait.webapp = for srv in service.deps.yarn_ts
        srv.options.yarn_site['yarn.http.policy'] ?= options.yarn_site['yarn.http.']
        srv.options.yarn_site['yarn.timeline-service.webapp.address'] ?= "#{srv.node.fqdn}:8188"
        srv.options.yarn_site['yarn.timeline-service.webapp.https.address'] ?= "#{srv.node.fqdn}:8190"
        protocol = if srv.options.yarn_site['yarn.http.policy'] is 'HTTP_ONLY' then '' else 'https.'
        [host, port] = srv.options.yarn_site["yarn.timeline-service.webapp.#{protocol}address"].split ':'
        host: host, port: port

## Hadoop Site Configuration
Enrich `ryba-ambari-takeover/hadoop/hdfs` with hdfs_nn properties.
  
      enrich_config = (source, target) ->
        for k, v of source
          target[k] ?= v
      
      for srv in service.deps.yarn
        srv.options.configurations ?= {}
        srv.options.configurations['core-site'] ?= {}
        srv.options.configurations['hdfs-site'] ?= {}
        srv.options.configurations['yarn-site'] ?= {}
        srv.options.configurations['mapred-site'] ?= {}
        srv.options.configurations['ssl-server'] ?= {}
        srv.options.configurations['ssl-client'] ?= {}

        # enrich_config options.core_site, srv.options.configurations['core-site']
        # enrich_config options.hdfs_site, srv.options.configurations['hdfs-site']
        enrich_config options.yarn_site, srv.options.configurations['yarn-site']
        # enrich_config options.mapred_site, srv.options.configurations['mapred-site']
        # enrich_config options.ssl_server, srv.options.configurations['ssl-server']
        # enrich_config options.ssl_client, srv.options.configurations['ssl-client']
        
        #add hosts
        srv.options.ts_hosts ?= []
        srv.options.ts_hosts.push options.fqdn if srv.options.ts_hosts.indexOf(options.fqdn) is -1

## System Options
      
        # Env
        srv.options.configurations['yarn-env'] ?= {}
        srv.options.configurations['yarn-env']['apptimelineserver_heapsize'] ?= options.heapsize
        # opts
        srv.options.yarn_ts_opts = options.opts

## Metrics Properties

        srv.options.configurations['hadoop-metrics-properties'] ?= {}
        enrich_config options.metrics.config, srv.options.configurations['hadoop-metrics-properties'] if service.deps.metrics?

## Log4j Properties

        srv.options.yarn_log4j ?= {}
        enrich_config options.log4j.properties, options.yarn_log4j if service.deps.log4j?

## Ambari

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.takeover = service.deps.ambari_server.options.takeover
      options.baremetal = service.deps.ambari_server.options.baremetal

## Dependencies

    {merge} = require 'nikita/lib/misc'
