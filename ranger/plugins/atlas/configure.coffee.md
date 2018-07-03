
# Ranger Atlas Plugin Configure
Ranger Atlas plugin runs inside Atlas Metadata server's JVM


    module.exports = (service) ->
      options = service.options

## Identities

      options.group = merge {}, service.deps.ranger_admin.options.group, options.group or {}
      options.user = merge {}, service.deps.ranger_admin.options.user, options.user or {}

## Kerberos

      options.krb5 ?= {}
      options.krb5.enabled ?= service.deps.hadoop_core.options.core_site['hadoop.security.authentication'] is 'kerberos'
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin = service.deps.krb5_client.options.admin[options.krb5.realm]
      options.jre_home ?= service.deps.java.options.jre_home

## Access

      options.ranger_admin ?= service.deps.ranger_admin.options.admin
      options.ranger_ranger_hdfs_install ?= service.deps.ranger_hdfs[0].options.install
      options.atlas_user = service.deps.atlas[0].options.user
      options.atlas_group = service.deps.atlas[0].options.group
      options.hdfs_client = service.deps.hdfs_client[0]
      options.ranger_hdfs_install = service.deps.ranger_hdfs
      options.hdfs_krb5_user = service.deps.hadoop_core.options.hdfs.krb5_user

## Plugin User

      options.plugin_user = 
        "name": options.atlas_user.name
        "firstName": ''
        "lastName": ''
        "emailAddress": ''
        "password": ''
        'userSource': 1
        'userRoleList': ['ROLE_USER']
        'groups': []
        'status': 1
        
## Configuration

      options.install ?= {}
      options.install['PYTHON_COMMAND_INVOKER'] ?= 'python'
      # Should Atlas GRANT/REVOKE update XA policies?
      options.install['UPDATE_XAPOLICIES_ON_GRANT_REVOKE'] ?= 'true'
      options.install['CUSTOM_USER'] ?= "#{options.atlas_user.name}"
      options.install['CUSTOM_GROUP'] ?= "#{options.atlas_group.name}"
      options.conf_dir ?= service.deps.atlas[0].options.conf_dir

## Admin properties

      options.install['POLICY_MGR_URL'] ?= service.deps.ranger_admin.options.install['policymgr_external_url']
      options.install['REPOSITORY_NAME'] ?= 'hadoop-ryba-atlas'

## Service Definition

      options.service_repo ?=
        'name': options.install['REPOSITORY_NAME']
        'description': 'Atlas Repo'
        'type': 'atlas'
        'isEnabled': true
        'configs':
          # 'username': 'ranger_plugin_atlas'
          # 'password': 'RangerPluginAtlas123!'
          'username': service.deps.ranger_admin.options.plugins.principal
          'password': service.deps.ranger_admin.options.plugins.password
          'atlas.rest.address': service.deps.atlas[0].options.application.properties['atlas.rest.address']
          'policy.download.auth.users': "#{options.atlas_user.name}" #from ranger 0.6
          'tag.download.auth.users': "#{options.atlas_group.name}"

### HDFS Storage

      options.install['XAAUDIT.HDFS.IS_ENABLED'] ?= 'true'
      if options.install['XAAUDIT.HDFS.IS_ENABLED'] is 'true'
        # migration: lucasbak 11102017
        # honored but not used by plugin
        # options.install['XAAUDIT.HDFS.LOCAL_BUFFER_DIRECTORY'] ?= "#{service.deps.ranger_admin.options.conf_dir}/%app-type%/audit"
        # options.install['XAAUDIT.HDFS.LOCAL_ARCHIVE_DIRECTORY'] ?= "#{service.deps.ranger_admin.options.conf_dir}/%app-type%/archive"
        options.install['XAAUDIT.HDFS.ENABLE'] ?= 'true'
        options.install['XAAUDIT.HDFS.HDFS_DIR'] ?= "#{options.hdfs_client.options.core_site['fs.defaultFS']}/#{options.user.name}/audit"
        options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR'] ?= "#{service.deps.atlas[0].options.log_dir}/audit/hdfs/spool"
        options.install['XAAUDIT.HDFS.DESTINATION_DIRECTORY'] ?= "#{options.hdfs_client.options.core_site['fs.defaultFS']}/#{options.user.name}/audit/%app-type%/%time:yyyyMMdd%"
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_DIRECTORY'] ?= '/var/log/ranger/%app-type%/audit'
        options.install['XAAUDIT.HDFS.LOCAL_ARCHIVE_DIRECTORY'] ?= '/var/log/ranger/%app-type%/archive'
        options.install['XAAUDIT.HDFS.DESTINATION_FILE'] ?= '%hostname%-audit.log'
        options.install['XAAUDIT.HDFS.DESTINATION_FLUSH_INTERVAL_SECONDS'] ?= '900'
        options.install['XAAUDIT.HDFS.DESTINATION_ROLLOVER_INTERVAL_SECONDS'] ?= '86400'
        options.install['XAAUDIT.HDFS.DESTINATION _OPEN_RETRY_INTERVAL_SECONDS'] ?= '60'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_FILE'] ?= '%time:yyyyMMdd-HHmm.ss%.log'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_FLUSH_INTERVAL_SECONDS'] ?= '60'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_ROLLOVER_INTERVAL_SECONDS'] ?= '600'
        options.install['XAAUDIT.HDFS.LOCAL_ARCHIVE _MAX_FILE_COUNT'] ?= '5'
        # AUDIT TO HDFS

## HDFS Policy

      if options.install['XAAUDIT.HDFS.IS_ENABLED'] is 'true'
        throw Error 'HDFS Ranger Plugin required' unless options.ranger_hdfs_install
        options.policy_hdfs_audit ?=
          'name': "atlas-ranger-plugin-audit"
          'service': "#{options.ranger_hdfs_install['REPOSITORY_NAME']}"
          'repositoryType':"hdfs"
          'description': 'Atlas Ranger Plugin audit log policy'
          'isEnabled': true
          'isAuditEnabled': true
          'resources':
            'path':
              'isRecursive': 'true'
              'values': ['/ranger/audit/atlas']
              'isExcludes': false
          'policyItems': [
            'users': ["#{options.atlas_user.name}"]
            'groups': []
            'delegateAdmin': true
            'accesses': [
                "isAllowed": true
                "type": "read"
            ,
                "isAllowed": true
                "type": "write"
            ,
                "isAllowed": true
                "type": "execute"
            ]
            'conditions': []
          ]

### Atlas Audit (database storage)

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

### Atlas Audit (to SOLR)

      if service.deps.ranger_admin.options.install['audit_store'] is 'solr'
        options.audit ?= {}
        options.install['XAAUDIT.SOLR.IS_ENABLED'] ?= 'true'
        options.install['XAAUDIT.SOLR.ENABLE'] ?= 'true'
        options.install['XAAUDIT.SOLR.URL'] ?= service.deps.ranger_admin.options.install['audit_solr_urls']
        options.install['XAAUDIT.SOLR.USER'] ?= service.deps.ranger_admin.options.install['audit_solr_user']
        options.install['XAAUDIT.SOLR.ZOOKEEPER'] ?= service.deps.ranger_admin.options.install['audit_solr_zookeepers']
        options.install['XAAUDIT.SOLR.PASSWORD'] ?= service.deps.ranger_admin.options.install['audit_solr_password']
        options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR'] ?= "#{service.deps.atlas[0].options.log_dir}/audit/solr/spool"
        options.audit['xasecure.audit.destination.solr.force.use.inmemory.jaas.config'] ?= 'true'
        options.audit['xasecure.audit.jaas.inmemory.loginModuleName'] ?= 'com.sun.security.auth.module.Krb5LoginModule'
        options.audit['xasecure.audit.jaas.inmemory.loginModuleControlFlag'] ?= 'required'
        options.audit['xasecure.audit.jaas.inmemory.Client.option.useKeyTab'] ?= 'true'
        options.audit['xasecure.audit.jaas.inmemory.Client.option.debug'] ?= 'true'
        options.audit['xasecure.audit.jaas.inmemory.Client.option.doNotPrompt'] ?= 'yes'
        options.audit['xasecure.audit.jaas.inmemory.Client.option.storeKey'] ?= 'yes'
        options.audit['xasecure.audit.jaas.inmemory.Client.option.serviceName'] ?= 'solr'
        atlas_princ = service.deps.atlas_server.options.application.properties['atlas.authentication.principal'].replace '_HOST', service.deps.atlas[0].node.fqdn
        options.audit['xasecure.audit.jaas.inmemory.Client.option.principal'] ?= atlas_princ
        options.audit['xasecure.audit.jaas.inmemory.Client.option.keyTab'] ?= service.deps.atlas[0].options.application.properties['atlas.authentication.keytab']

### Plugin Execution

Used only if SSL is enabled between Policy Admin Tool and Plugin

      if service.deps.ranger_admin.options.site['ranger.service.https.attrib.ssl.enabled'] is 'true'
        options.ssl = merge {}, service.deps.hadoop_core.options.ssl, options.ssl
        options.install['SSL_KEYSTORE_FILE_PATH'] ?= service.deps.hadoop_core.options.ssl_server['ssl.server.keystore.location']
        options.install['SSL_KEYSTORE_PASSWORD'] ?= service.deps.hadoop_core.options.ssl_server['ssl.server.keystore.password']
        options.install['SSL_TRUSTSTORE_FILE_PATH'] ?= service.deps.hadoop_core.options.ssl_server['ssl.server.truststore.location']
        options.install['SSL_TRUSTSTORE_PASSWORD'] ?= service.deps.hadoop_core.options.ssl_server['ssl.server.truststore.password']

## Ambari Configuration

        options.configurations ?= {}
        options.configurations['ranger-atlas-security'] ?= {}
        options.configurations['ranger-atlas-security']['ranger.plugin.atlas.service.name'] ?= options.service_repo.name
        options.configurations['ranger-atlas-security']['ranger.plugin.atlas.policy.rest.url'] ?= options.install['POLICY_MGR_URL']
        options.configurations['ranger-atlas-security']['ranger.plugin.atlas.policy.cache.dir'] ?= "/etc/ranger/#{options.service_repo.name}/policycache"
        options.configurations['ranger-atlas-security']['ranger.plugin.atlas.policy.pollIntervalMs'] ?= "30000"
        options.configurations['ranger-atlas-security']['ranger.plugin.atlas.policy.rest.ssl.config.file'] ?= "#{service.deps.atlas_server.options.conf_dir}/ranger-policymgr-ssl.xml"
        options.configurations['ranger-atlas-security']['ranger.plugin.atlas.policy.source.impl'] ?= 'org.apache.ranger.admin.client.RangerAdminRESTClient'

        options.configurations['ranger-atlas-plugin-properties'] ?= {}
        # options.configurations['ranger-atlas-plugin-properties'] ?= merge {}, options.service_repo.configs,
        #   options.install, options.configurations['ranger-atlas-plugin-properties']
        options.configurations['ranger-atlas-plugin-properties']['ranger-atlas-plugin-enabled'] ?= 'Yes' 
        options.configurations['ranger-atlas-plugin-properties']['REPOSITORY_CONFIG_USERNAME'] ?= options.service_repo.configs.username
        options.configurations['ranger-atlas-plugin-properties']['REPOSITORY_CONFIG_PASSWORD'] ?= options.service_repo.configs.password
        options.configurations['ranger-atlas-plugin-properties']['common.name.for.certificate'] ?= options.service_repo.configs['commonNameForCertificate']
        options.configurations['ranger-atlas-plugin-properties']['hadoop.rpc.protection'] ?= options.service_repo.configs['hadoop.rpc.protection']
        options.configurations['ranger-atlas-plugin-properties']['policy_user'] ?= options.service_repo.configs['policy.download.auth.users']
        for k, v of options.install
          if k.indexOf('XAAUDIT') isnt -1
            options.configurations['ranger-atlas-plugin-properties'][k] ?= v

## Plugin - Ranger Admin SSL
configure `policy-mgr-ssl` ambari configuration to make the plugin communicate overs tls with ranger-admin
 
 
        options.ssl = merge {}, service.deps.ssl?.options, options.ssl
        options.configurations['ranger-atlas-policymgr-ssl'] ?= {}
        if service.deps.ranger_admin.options.site['ranger.service.https.attrib.ssl.enabled'] is 'true'
          options.configurations['ranger-atlas-policymgr-ssl']['xasecure.policymgr.clientssl.keystore'] ?= "/usr/hdp/current/atlas-server/conf/ranger-plugin-keystore.jks"
          options.configurations['ranger-atlas-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password'] ?= service.deps.hadoop_core.options.ssl_server['ssl.server.keystore.password']
          options.configurations['ranger-atlas-policymgr-ssl']['xasecure.policymgr.clientssl.truststore'] ?= "/usr/hdp/current/atlas-server/conf/ranger-plugin-truststore.jks"
          options.configurations['ranger-atlas-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password'] ?= service.deps.hadoop_core.options.ssl_server['ssl.server.truststore.password']
          options.configurations['ranger-atlas-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.credential.file'] ?= "jceks://file{{credential_file}}"
          options.configurations['ranger-atlas-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.credential.file'] ?=  "jceks://file{{credential_file}}"

## Plugin Audit

        options.configurations['ranger-atlas-audit'] ?= {}
        options.configurations['ranger-atlas-audit']['xasecure.audit.is.enabled'] ?= 'true'
        # audit to hdfs
        options.configurations['ranger-atlas-audit']['xasecure.audit.destination.hdfs'] ?= options.install['XAAUDIT.HDFS.IS_ENABLED']
        options.configurations['ranger-atlas-audit']['xasecure.audit.destination.hdfs.batch.filespool.dir'] ?= options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR']
        options.configurations['ranger-atlas-audit']['xasecure.audit.destination.hdfs.dir'] ?= options.install['XAAUDIT.HDFS.HDFS_DIR']
        # audit to solr
        options.configurations['ranger-atlas-audit']['xasecure.audit.destination.solr'] ?= options.install['XAAUDIT.SOLR.IS_ENABLED']
        options.configurations['ranger-atlas-audit']['xasecure.audit.destination.solr.batch.filespool.dir'] ?= options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR']
        options.configurations['ranger-atlas-audit']['xasecure.audit.destination.solr.zookeepers'] ?= options.install['XAAUDIT.SOLR.ZOOKEEPER']
        options.configurations['ranger-atlas-audit']['xasecure.audit.solr.solr_url'] ?= options.install['XAAUDIT.SOLR.URL']
        ## JAAS in memory configuration
        options.configurations['ranger-atlas-audit']['xasecure.audit.destination.solr.force.use.inmemory.jaas.config'] ?= 'true'
        options.configurations['ranger-atlas-audit']['xasecure.audit.jaas.inmemory.loginModuleName'] ?= 'com.sun.security.auth.module.Krb5LoginModule'
        options.configurations['ranger-atlas-audit']['xasecure.audit.jaas.inmemory.loginModuleControlFlag'] ?= 'required'
        options.configurations['ranger-atlas-audit']['xasecure.audit.jaas.inmemory.Client.option.useKeyTab'] ?= 'true'
        options.configurations['ranger-atlas-audit']['xasecure.audit.jaas.inmemory.Client.option.debug'] ?= 'true'
        options.configurations['ranger-atlas-audit']['xasecure.audit.jaas.inmemory.Client.option.doNotPrompt'] ?= 'yes'
        options.configurations['ranger-atlas-audit']['xasecure.audit.jaas.inmemory.Client.option.storeKey'] ?= 'yes'
        options.configurations['ranger-atlas-audit']['xasecure.audit.jaas.inmemory.Client.option.serviceName'] ?= 'solr'
        options.configurations['ranger-atlas-audit']['xasecure.audit.jaas.Client.option.keyTab '] ?= service.deps.atlas_server.options.application.properties['atlas.authentication.keytab']
        options.configurations['ranger-atlas-audit']['xasecure.audit.jaas.inmemory.Client.option.keyTab'] ?= service.deps.atlas_server.options.application.properties['atlas.authentication.keytab']
        options.configurations['ranger-atlas-audit']['xasecure.audit.jaas.inmemory.Client.option.principal'] ?= service.deps.atlas_server.options.application.properties['atlas.authentication.principal']
        options.configurations['ranger-atlas-audit']['xasecure.audit.provider.summary.enabled'] ?= 'true'

## Ambari

        #ambari server configuration
        options.post_component = service.instances[0].node.fqdn is service.node.fqdn
        options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
        options.ambari_url ?= service.deps.ambari_server.options.ambari_url
        options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
        options.cluster_name ?= service.deps.ambari_server.options.cluster_name
        options.takeover = service.deps.ambari_server.options.takeover

## Enable Plugin in Ranger Admin

        service.deps.ranger_admin.options.configurations['ranger-env']['ranger-atlas-plugin-enabled'] = 'Yes'

## Enrich HBase Service  with Ranger Properties
For HBase, ranger related properties should be posted before any service is installed or
started, as Ambari required to configuration dictionnaries to exist `ranger-atlas-plugin-properties` 
        
        for srv in service.deps.atlas
          srv.options.configurations ?= {}
          srv.options.configurations['ranger-atlas-security'] ?= merge {}, srv.options.configurations['ranger-atlas-security'], options.configurations['ranger-atlas-security']
          srv.options.configurations['ranger-atlas-plugin-properties'] ?= merge {}, srv.options.configurations['ranger-atlas-plugin-properties'], options.configurations['ranger-atlas-plugin-properties']
          srv.options.configurations['ranger-atlas-policymgr-ssl'] ?= merge {}, srv.options.configurations['ranger-atlas-policymgr-ssl'], options.configurations['ranger-atlas-policymgr-ssl']
          srv.options.configurations['ranger-atlas-audit'] ?= merge {}, srv.options.configurations['ranger-atlas-audit'], options.configurations['ranger-atlas-audit']


## Wait

      options.wait_ranger_admin = service.deps.ranger_admin.options.wait

## Dependencies

    {merge} = require 'nikita/lib/misc'
