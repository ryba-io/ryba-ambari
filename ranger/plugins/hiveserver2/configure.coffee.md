
# Ranger HIVE Plugin Configure

Ranger Hive plugin runs inside Hiveserver2's JVM

    module.exports = (service) ->
      options = service.options

## Identities

      options.group = merge {}, service.deps.ranger_admin.options.group, options.group or {}
      options.user = merge {}, service.deps.ranger_admin.options.user, options.user or {}
      options.hive_user = service.deps.hive_server2.options.user
      options.hive_group = service.deps.hive_server2.options.group
      options.hdfs_krb5_user = service.deps.hadoop_core.options.hdfs.krb5_user

## Kerberos

      options.krb5 ?= {}
      options.krb5.enabled ?= service.deps.hadoop_core.options.core_site['hadoop.security.authentication'] is 'kerberos'
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin = service.deps.krb5_client.options.admin[options.krb5.realm]

## Access

      options.ranger_admin ?= service.deps.ranger_admin.options.admin
      options.hdfs_install ?= service.deps.ranger_hdfs[0].options.install if service.deps.ranger_hdfs

## Plugin User

      options.plugin_user =
        "name": options.hive_user.name
        "firstName": ''
        "lastName": ''
        "emailAddress": ''
        'userSource': 1
        'userRoleList': ['ROLE_USER']
        'groups': []
        'status': 1

## Environment

      # Layout
      options.conf_dir ?= service.deps.hive_server2.options.conf_dir

## Configuration

      options.install ?= {}
      options.install['PYTHON_COMMAND_INVOKER'] ?= 'python'
      # Should Hive GRANT/REVOKE update XA policies?
      options.install['UPDATE_XAPOLICIES_ON_GRANT_REVOKE'] ?= 'true'
      options.install['CUSTOM_USER'] ?= "#{options.user.name}"
      options.install['CUSTOM_GROUP'] ?= "#{options.group.name}"

## Plugin - Ranger Admin SSL

Used only if SSL is enabled between Policy Admin Tool and Plugin. The path to
keystore is derived from Hive Server2. The path to the truststore is derived
from Hadoop Core.
    
      options.ssl = merge {}, service.deps.ssl?.options, options.ssl
      if service.deps.ranger_admin.options.site['ranger.service.https.attrib.ssl.enabled'] is 'true'
        options.install['SSL_KEYSTORE_FILE_PATH'] ?= service.deps.hive_server2.options.hive_site['hive.server2.keystore.path']
        options.install['SSL_KEYSTORE_PASSWORD'] ?= service.deps.hive_server2.options.hive_site['hive.server2.keystore.password']
        options.install['SSL_TRUSTSTORE_FILE_PATH'] ?= service.deps.hadoop_core.options.ssl_client['ssl.client.truststore.location']
        options.install['SSL_TRUSTSTORE_PASSWORD'] ?= service.deps.hadoop_core.options.ssl_client['ssl.client.truststore.password']

##Policy Admin Tool

The repository name should match the reposity name in web ui.

      # Build Hive Server2 URL
      
      port = if service.deps.hive_server2.options.hive_site['hive.server2.transport.mode'] is 'http'
      then service.deps.hive_server2.options.hive_site['hive.server2.thrift.http.port']
      else service.deps.hive_server2.options.hive_site['hive.server2.thrift.port']
      httpPath = service.deps.hive_server2.options.hive_site['hive.server2.thrift.http.path']
      hive_url = 'jdbc:hive2://'
      hive_url += "#{service.node.fqdn}:#{port}/"
      if service.deps.hive_server2.options.hive_site['hive.server2.authentication'] is 'KERBEROS'
        hive_url += ";principal=#{service.deps.hive_server2.options.hive_site['hive.server2.authentication.kerberos.principal']}"
      if service.deps.hive_server2.options.hive_site['hive.server2.use.SSL'] is 'true'
        hive_url += ";ssl=true"
        hive_url += ";sslTrustStore=#{service.deps.hadoop_core.options.ssl_client['ssl.client.truststore.location']}"
        hive_url += ";trustStorePassword=#{service.deps.hadoop_core.options.ssl_client['ssl.client.truststore.password']}"
      if service.deps.hive_server2.options.hive_site['hive.server2.transport.mode'] is 'http'
        hive_url += ";transportMode=#{service.deps.hive_server2.options.hive_site['hive.server2.transport.mode']}"
        hive_url += ";httpPath=#{httpPath}"

## Admin properties

      options.install['POLICY_MGR_URL'] ?= service.deps.ranger_admin.options.install['policymgr_external_url']
      options.install['REPOSITORY_NAME'] ?= 'hadoop-ryba-hive'

## Service Definition

      options.service_repo ?=
        'name': options.install['REPOSITORY_NAME']
        'description': 'Hive Repo'
        'type': 'hive'
        'isEnabled': true
        'configs':
          # 'username': 'ranger_plugin_hbase'
          # 'password': 'RangerPluginHive123!'
          'username': service.deps.ranger_admin.options.plugins.principal
          'password': service.deps.ranger_admin.options.plugins.password
          'jdbc.driverClassName': 'org.apache.hive.jdbc.HiveDriver'
          'jdbc.url': "#{hive_url}"
          "commonNameForCertificate": ''
          'policy.download.auth.users': "#{service.deps.hive_server2.options.user.name}" #from ranger 0.6
          'tag.download.auth.users': "#{service.deps.hive_server2.options.user.name}"

### Plugin Config - Audit to Solr

      options.install['XAAUDIT.HDFS.IS_ENABLED'] ?= 'true'
      if options.install['XAAUDIT.HDFS.IS_ENABLED'] is 'true'
        # migration: lucasbak 11102017
        # honored but not used by plugin
        # options.install['XAAUDIT.HDFS.LOCAL_BUFFER_DIRECTORY'] ?= "#{service.deps.ranger_admin.options.conf_dir}/%app-type%/audit"
        # options.install['XAAUDIT.HDFS.LOCAL_ARCHIVE_DIRECTORY'] ?= "#{service.deps.ranger_admin.options.conf_dir}/%app-type%/archive"
        options.install['XAAUDIT.HDFS.HDFS_DIR'] ?= "#{service.deps.hdfs_client.options.core_site['fs.defaultFS']}/#{options.user.name}/audit"
        options.install['XAAUDIT.HDFS.ENABLE'] ?= 'true'
        options.install['XAAUDIT.HDFS.DESTINATION_DIRECTORY'] ?= "#{service.deps.hdfs_client.options.core_site['fs.defaultFS']}/#{options.user.name}/audit/%app-type%/%time:yyyyMMdd%"
        options.install['XAAUDIT.HDFS.DESTINATION_FILE'] ?= '%hostname%-audit.log'
        options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR'] ?= "#{service.deps.hive_server2.options.log_dir}/audit/hdfs/spool"
        options.install['XAAUDIT.HDFS.DESTINATION_FLUSH_INTERVAL_SECONDS'] ?= '900'
        options.install['XAAUDIT.HDFS.DESTINATION_ROLLOVER_INTERVAL_SECONDS'] ?= '86400'
        options.install['XAAUDIT.HDFS.DESTINATION _OPEN_RETRY_INTERVAL_SECONDS'] ?= '60'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_FILE'] ?= '%time:yyyyMMdd-HHmm.ss%.log'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_FLUSH_INTERVAL_SECONDS'] ?= '60'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_ROLLOVER_INTERVAL_SECONDS'] ?= '600'
        options.install['XAAUDIT.HDFS.LOCAL_ARCHIVE _MAX_FILE_COUNT'] ?= '5'

# ## HDFS Policy
# 
#       if options.install['XAAUDIT.HDFS.IS_ENABLED'] is 'true'
#         options.policy_hdfs_audit ?=
#           'name': "hive-ranger-plugin-audit"
#           'service': "#{options.hdfs_install['REPOSITORY_NAME']}"
#           'repositoryType':"hdfs"
#           'description': 'Hive Ranger Plugin audit log policy'
#           'isEnabled': true
#           'isAuditEnabled': true
#           'resources':
#             'path':
#               'isRecursive': 'true'
#               'values': ['/ranger/audit/hiveServer2']
#               'isExcludes': false
#           'policyItems': [
#             'users': ["#{options.hive_user.name}"]
#             'groups': []
#             'delegateAdmin': true
#             'accesses': [
#                 "isAllowed": true
#                 "type": "read"
#             ,
#                 "isAllowed": true
#                 "type": "write"
#             ,
#                 "isAllowed": true
#                 "type": "execute"
#             ]
#             'conditions': []
#           ]

### Plugin Config - Audit to Solr

      if service.deps.ranger_admin.options.install['audit_store'] is 'solr'
        options.audit ?= {}
        options.install['XAAUDIT.SOLR.IS_ENABLED'] ?= 'true'
        options.install['XAAUDIT.SOLR.ENABLE'] ?= 'true'
        options.install['XAAUDIT.SOLR.URL'] ?= service.deps.ranger_admin.options.install['audit_solr_urls']
        options.install['XAAUDIT.SOLR.USER'] ?= service.deps.ranger_admin.options.install['audit_solr_user']
        options.install['XAAUDIT.SOLR.ZOOKEEPER'] ?= service.deps.ranger_admin.options.install['audit_solr_zookeepers']
        options.install['XAAUDIT.SOLR.PASSWORD'] ?= service.deps.ranger_admin.options.install['audit_solr_password']
        options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR'] ?= "#{service.deps.hive_server2.options.log_dir}/audit/solr/spool"
        options.audit['xasecure.audit.destination.solr.force.use.inmemory.jaas.config'] ?= 'true'
        options.audit['xasecure.audit.jaas.inmemory.loginModuleName'] ?= 'com.sun.security.auth.module.Krb5LoginModule'
        options.audit['xasecure.audit.jaas.inmemory.loginModuleControlFlag'] ?= 'required'
        options.audit['xasecure.audit.jaas.inmemory.Client.option.useKeyTab'] ?= 'true'
        options.audit['xasecure.audit.jaas.inmemory.Client.option.debug'] ?= 'true'
        options.audit['xasecure.audit.jaas.inmemory.Client.option.doNotPrompt'] ?= 'yes'
        options.audit['xasecure.audit.jaas.inmemory.Client.option.storeKey'] ?= 'yes'
        options.audit['xasecure.audit.jaas.inmemory.Client.option.serviceName'] ?= 'solr'
        options.audit['xasecure.audit.jaas.inmemory.Client.option.principal'] = service.deps.hive_server2.options.hive_site['hive.server2.authentication.kerberos.principal'].replace '_HOST', service.node.fqdn
        options.audit['xasecure.audit.jaas.inmemory.Client.option.keyTab'] ?= service.deps.hive_server2.options.hive_site['hive.server2.authentication.kerberos.keytab']

### Database storage

      #Deprecated
      options.install['XAAUDIT.DB.IS_ENABLED'] ?= 'false'
      if options.install['XAAUDIT.DB.IS_ENABLED'] is 'true'
        options.install['XAAUDIT.DB.FLAVOUR'] ?= 'MYSQL'
        switch options.install['XAAUDIT.DB.FLAVOUR']
          when 'MYSQL'
            options.install['SQL_CONNECTOR_JAR'] ?= '/usr/share/java/mysql-connector-java.jar'
            options.install['XAAUDIT.DB.HOSTNAME'] ?= service.deps.ranger_admin.options.install['db_host']
            options.install['XAAUDIT.DB.DATABASE_NAME'] ?= service.deps.ranger_admin.options.install['audit_db_name']
            options.install['XAAUDIT.DB.USER_NAME'] ?= service.deps.ranger_admin.options.install['audit_db_user']
            options.install['XAAUDIT.DB.PASSWORD'] ?= service.deps.ranger_admin.options.install['audit_db_password']
          when 'ORACLE'
            throw Error 'Ryba does not support ORACLE Based Ranger Installation'
          else
            throw Error "Apache Ranger does not support chosen DB FLAVOUR"
      else
          options.install['XAAUDIT.DB.HOSTNAME'] ?= 'NONE'
          options.install['XAAUDIT.DB.DATABASE_NAME'] ?= 'NONE'
          options.install['XAAUDIT.DB.USER_NAME'] ?= 'NONE'
          options.install['XAAUDIT.DB.PASSWORD'] ?= 'NONE'

## Wait

      options.wait_ranger_admin = service.deps.ranger_admin.options.wait

## Enrich configuration

      # Hive HCatalog
      for srv in service.deps.hive_hcatalog
        srv.options.warehouse_mode = '0000'
      for srv in service.deps.hive
        srv.options.configurations ?= {}
        srv.options.configurations['hive-env'] ?= {}
        srv.options.configurations['hiveserver2-site'] ?= {}
        # Hive Server2 Properties
        srv.options.configurations['hive-site']['hive.security.authorization.manager'] = 'org.apache.ranger.authorization.hive.authorizer.RangerHiveAuthorizerFactory'
        srv.options.configurations['hiveserver2-site']['hive.security.authorization.manager'] = 'org.apache.ranger.authorization.hive.authorizer.RangerHiveAuthorizerFactory'
        srv.options.configurations['hiveserver2-site']['hive.security.authorization.enabled '] = 'true'
        srv.options.configurations['hive-env']['hive_security_authorization'] = 'Ranger'
        srv.options.configurations['hiveserver2-site']['hive.security.authorization.enabled'] ?= 'true'
        # service.deps.hive_server2.options.hive_site['hive.security.authenticator.manager'] = 'org.apache.hadoop.hive.ql.security.SessionStateUserAuthenticator'
        # service.deps.hive_server2.options.opts ?= ''
        # service.deps.hive_server2.options.opts += " -Djavax.net.ssl.trustStore=#{service.deps.hadoop_core.options.ssl_client['ssl.client.truststore.location']} "
        # service.deps.hive_server2.options.opts += " -Djavax.net.ssl.trustStorePassword=#{service.deps.hadoop_core.options.ssl_client['ssl.client.truststore.password']}"

## Ambari Config - Hive Plugin Audit

        options.configurations ?= {}
        options.configurations['ranger-hive-audit'] ?= {}
        options.configurations['ranger-hive-audit']['xasecure.audit.is.enabled'] ?= 'true'
        # audit to hdfs
        options.configurations['ranger-hive-audit']['xasecure.audit.destination.hdfs'] ?= options.install['XAAUDIT.HDFS.IS_ENABLED']
        options.configurations['ranger-hive-audit']['xasecure.audit.destination.hdfs.batch.filespool.dir'] ?= options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR']
        options.configurations['ranger-hive-audit']['xasecure.audit.destination.hdfs.dir'] ?= options.install['XAAUDIT.HDFS.HDFS_DIR']
        # audit to solr
        options.configurations['ranger-hive-audit']['xasecure.audit.destination.solr'] ?= options.install['XAAUDIT.SOLR.IS_ENABLED']
        options.configurations['ranger-hive-audit']['xasecure.audit.destination.solr.batch.filespool.dir'] ?= options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR']
        options.configurations['ranger-hive-audit']['xasecure.audit.destination.solr.zookeepers'] ?= options.install['XAAUDIT.SOLR.ZOOKEEPER']
        options.configurations['ranger-hive-audit']['xasecure.audit.solr.solr_url'] ?= options.install['XAAUDIT.SOLR.URL']
        options.configurations['ranger-hive-audit']['xasecure.audit.jaas.inmemory.loginModuleName'] ?= 'com.sun.security.auth.module.Krb5LoginModule'
        options.configurations['ranger-hive-audit']['xasecure.audit.jaas.inmemory.loginModuleControlFlag'] ?= 'required'
        options.configurations['ranger-hive-audit']['xasecure.audit.jaas.inmemory.Client.option.useKeyTab'] ?= 'true'
        options.configurations['ranger-hive-audit']['xasecure.audit.jaas.inmemory.Client.option.debug'] ?= 'true'
        options.configurations['ranger-hive-audit']['xasecure.audit.jaas.inmemory.Client.option.doNotPrompt'] ?= 'yes'
        options.configurations['ranger-hive-audit']['xasecure.audit.jaas.inmemory.Client.option.storeKey'] ?= 'yes'
        options.configurations['ranger-hive-audit']['xasecure.audit.jaas.inmemory.Client.option.serviceName'] ?= 'solr'
        options.configurations['ranger-hive-audit']['xasecure.audit.jaas.inmemory.Client.option.keyTab'] ?= service.deps.hive_server2.options.hive_site['hive.server2.authentication.kerberos.keytab']
        options.configurations['ranger-hive-audit']['xasecure.audit.jaas.inmemory.Client.option.principal'] ?= service.deps.hive_server2.options.hive_site['hive.server2.authentication.kerberos.principal']

## Ambari Config - Hive Plugin SSL
SSL can be configured to use SSL if ranger admin has SSL enabled.
 
        options.ssl = merge {}, service.deps.ssl?.options, options.ssl
        options.configurations['ranger-hive-policymgr-ssl'] ?= {}
        if service.deps.ranger_admin.options.site['ranger.service.https.attrib.ssl.enabled'] is 'true'
          options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.keystore'] ?= "#{service.deps.hadoop_core.options.ssl.conf_dir}/ranger-hive-plugin-keystore"
          options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password'] ?= service.deps.hadoop_core.options.ssl_server['ssl.server.keystore.password']
          options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.truststore'] ?= "#{service.deps.hadoop_core.options.ssl.conf_dir}/ranger-hive-plugin-truststore"
          options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password'] ?= service.deps.hadoop_core.options.ssl_server['ssl.server.truststore.password']
          options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.credential.file'] ?= "jceks://file/etc/ranger/#{options.service_repo.name}/cred.jceks"
          options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.credential.file'] ?=  "jceks://file/etc/ranger/#{options.service_repo.name}/cred.jceks"

## Ambari Config - Hive Plugin Properties

        options.configurations['ranger-hive-plugin-properties'] ?= {}
        options.configurations['ranger-hive-plugin-properties']['ranger-hive-plugin-enabled'] ?= 'Yes' 
        options.configurations['ranger-hive-plugin-properties']['REPOSITORY_CONFIG_USERNAME'] ?= options.service_repo.configs.username
        options.configurations['ranger-hive-plugin-properties']['REPOSITORY_CONFIG_PASSWORD'] ?= options.service_repo.configs.password
        options.configurations['ranger-hive-plugin-properties']['common.name.for.certificate'] ?= options.service_repo.configs['commonNameForCertificate']
        options.configurations['ranger-hive-plugin-properties']['hadoop.rpc.protection'] ?= options.service_repo.configs['hadoop.rpc.protection']
        options.configurations['ranger-hive-plugin-properties']['policy_user'] ?= options.service_repo.configs['policy.download.auth.users']
        options.configurations['ranger-hive-plugin-properties']['jdbc.driverClassName'] ?= options.service_repo.configs['jdbc.driverClassName']
        options.configurations['ranger-hive-plugin-properties']['jdbc.url'] ?= options.service_repo.configs['jdbc.url']
        for k, v of options.install
          if k.indexOf('XAAUDIT') isnt -1
            options.configurations['ranger-hive-plugin-properties'][k] ?= v

## Ambari Config - Hive Plugin Security

        options.configurations['ranger-hive-security'] ?= {}
        options.configurations['ranger-hive-security']['ranger.plugin.hive.service.name'] ?= options.service_repo.name
        options.configurations['ranger-hive-security']['ranger.plugin.hive.policy.rest.url'] ?= options.install['POLICY_MGR_URL']
        options.configurations['ranger-hive-security']['ranger.plugin.hive.policy.cache.dir'] ?= "/etc/ranger/#{options.service_repo.name}/policycache"
        options.configurations['ranger-hive-security']['ranger.plugin.hive.policy.pollIntervalMs'] ?= "30000"
        options.configurations['ranger-hive-security']['ranger.plugin.hive.policy.rest.ssl.config.file'] ?= "#{options.conf_dir}/conf.server/ranger-policymgr-ssl.xml"
        options.configurations['ranger-hive-security']['ranger.plugin.hive.policy.source.impl'] ?= 'org.apache.ranger.admin.client.RangerAdminRESTClient'

## Ambari Config REST Api

        #ambari server configuration
        options.post_component = service.instances[0].node.fqdn is service.node.fqdn
        options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
        options.ambari_url ?= service.deps.ambari_server.options.ambari_url
        options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
        options.cluster_name ?= service.deps.ambari_server.options.cluster_name
        options.takeover = service.deps.ambari_server.options.takeover

## Enable Plugin in Ranger Admin

        service.deps.ranger_admin.options.configurations['ranger-env']['ranger-hive-plugin-enabled'] = 'Yes'

## Enrich Hive Service  with Ranger Properties
For Hive, ranger related properties should be posted before any service is installed or
started, as Ambari required to configuration dictionnaries to exist `ranger-hive-plugin-properties` 
        
        for srv in service.deps.hive
          srv.options.configurations ?= {}
          srv.options.configurations['ranger-hive-security'] ?= merge {}, srv.options.configurations['ranger-hive-security'], options.configurations['ranger-hive-security']
          srv.options.configurations['ranger-hive-plugin-properties'] ?= merge {}, srv.options.configurations['ranger-hive-plugin-properties'], options.configurations['ranger-hive-plugin-properties']
          srv.options.configurations['ranger-hive-policymgr-ssl'] ?= merge {}, srv.options.configurations['ranger-hive-policymgr-ssl'], options.configurations['ranger-hive-policymgr-ssl']
          srv.options.configurations['ranger-hive-audit'] ?= merge {}, srv.options.configurations['ranger-hive-audit'], options.configurations['ranger-hive-audit']

## Dependencies

    {merge} = require 'nikita/lib/misc'
