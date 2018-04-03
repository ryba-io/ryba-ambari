
## Ranger Knox Plugin Configure

    module.exports = (service) ->
      options = service.options

## Environment

      # Layout
      options.conf_dir ?= service.deps.knox[0].options.conf_dir

## Kerberos

      options.krb5 ?= {}
      options.krb5.enabled ?= service.deps.hadoop_core.options.core_site['hadoop.security.authentication'] is 'kerberos'
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin = service.deps.krb5_client.options.admin[options.krb5.realm]

## Identities

      options.group = merge {}, service.deps.ranger_admin.options.group, options.group or {}
      options.user = merge {}, service.deps.ranger_admin.options.user, options.user or {}
      options.knox_user = service.deps.knox[0].options.user
      options.knox_group = service.deps.knox[0].options.group
      options.hadoop_group = service.deps.hadoop_core.options.hadoop_group
      options.hdfs_krb5_user = service.deps.hadoop_core.options.hdfs.krb5_user

## Access

      options.ranger_admin ?= service.deps.ranger_admin.options.admin

## Configuration

      # Knox Plugin configuration
      options.install ?= {}
      options.install['PYTHON_COMMAND_INVOKER'] ?= 'python'
      options.install['CUSTOM_USER'] ?= "#{options.knox_user.name}"
      options.install['CUSTOM_GROUP'] ?= "#{options.knox_group.name}"
      options.install['KNOX_HOME'] ?= '/usr/hdp/current/knox-server'

## Admin properties

      options.install['POLICY_MGR_URL'] ?= service.deps.ranger_admin.options.install['policymgr_external_url']
      options.install['REPOSITORY_NAME'] ?= 'hadoop-ryba-knox'
        
## Plugin User

      options.plugin_user ?=
        'name': options.knox_user.name
        'firstName': ''
        'lastName': ''
        'emailAddress': ''
        'userSource': 1
        'userRoleList': ['ROLE_USER']
        'groups': []
        'status': 1

## Service Definition

      knox_protocol = if service.deps.knox_server.options.ssl then 'https' else 'http'
      knox_url = "#{knox_protocol}://#{service.deps.knox_server.node.fqdn}"
      knox_url += ":#{service.deps.knox_server.options.gateway_site['gateway.port']}/#{service.deps.knox_server.options.gateway_site['gateway.path']}"
      knox_url += '/admin/api/v1/topologies'
      options.service_repo ?=
        'name': options.install['REPOSITORY_NAME']
        'description': 'Knox Repository'
        'type': 'knox'
        'isEnabled': true
        'configs':
          'username': service.deps.ranger_admin.options.plugins.principal
          'password': service.deps.ranger_admin.options.plugins.password
          'knox.url': "#{knox_url}"
          'commonNameForCertificate': ''
          'policy.download.auth.users': "#{options.user.name}" #from ranger 0.6
          'tag.download.auth.users': "#{options.user.name}"

## HDFS Storage

      options.install['XAAUDIT.HDFS.IS_ENABLED'] ?= 'true'
      if options.install['XAAUDIT.HDFS.IS_ENABLED'] is 'true'
        # migration: lucasbak 11102017
        # honored but not used by plugin
        # options.install['XAAUDIT.HDFS.LOCAL_BUFFER_DIRECTORY'] ?= "#{service.deps.ranger_admin.options.conf_dir}/%app-type%/audit"
        # options.install['XAAUDIT.HDFS.LOCAL_ARCHIVE_DIRECTORY'] ?= "#{service.deps.ranger_admin.options.conf_dir}/%app-type%/archive"
        options.install['XAAUDIT.HDFS.ENABLE'] ?= 'true'
        options.install['XAAUDIT.HDFS.HDFS_DIR'] ?= "#{service.deps.hdfs_client.options.core_site['fs.defaultFS']}/#{options.user.name}/audit"
        options.install['XAAUDIT.HDFS.DESTINATION_DIRECTORY'] ?= "#{service.deps.hdfs_client.options.core_site['fs.defaultFS']}/#{options.user.name}/audit/%app-type%/%time:yyyyMMdd%"
        options.install['XAAUDIT.HDFS.DESTINATION_FILE'] ?= '%hostname%-audit.log'
        options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR'] ?= "#{service.deps.knox_server.options.log_dir}/audit/hdfs/spool"
        options.install['XAAUDIT.HDFS.DESTINATION_FILE'] ?= '%hostname%-audit.log'
        options.install['XAAUDIT.HDFS.DESTINATION_FLUSH_INTERVAL_SECONDS'] ?= '900'
        options.install['XAAUDIT.HDFS.DESTINATION_ROLLOVER_INTERVAL_SECONDS'] ?= '86400'
        options.install['XAAUDIT.HDFS.DESTINATION _OPEN_RETRY_INTERVAL_SECONDS'] ?= '60'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_FILE'] ?= '%time:yyyyMMdd-HHmm.ss%.log'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_FLUSH_INTERVAL_SECONDS'] ?= '60'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_ROLLOVER_INTERVAL_SECONDS'] ?= '600'
        options.install['XAAUDIT.HDFS.LOCAL_ARCHIVE _MAX_FILE_COUNT'] ?= '5'

## Solr Storage

      if service.deps.ranger_admin.options.install['audit_store'] is 'solr'
        options.install['XAAUDIT.SOLR.IS_ENABLED'] ?= 'true'
        options.install['XAAUDIT.SOLR.ENABLE'] ?= 'true'
        options.install['XAAUDIT.SOLR.URL'] ?= service.deps.ranger_admin.options.install['audit_solr_urls']
        options.install['XAAUDIT.SOLR.USER'] ?= service.deps.ranger_admin.options.install['audit_solr_user']
        options.install['XAAUDIT.SOLR.ZOOKEEPER'] ?= service.deps.ranger_admin.options.install['audit_solr_zookeepers']
        options.install['XAAUDIT.SOLR.PASSWORD'] ?= service.deps.ranger_admin.options.install['audit_solr_password']
        options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR'] ?= "#{service.deps.knox_server.options.log_dir}/audit/solr/spool"

## Database Storage

      # Deprecated
      options.install['XAAUDIT.DB.IS_ENABLED'] ?= 'false'
      options.install['SQL_CONNECTOR_JAR'] ?= '/usr/share/java/mysql-connector-java.jar'
      if options.install['XAAUDIT.DB.IS_ENABLED'] is 'true'
        options.install['XAAUDIT.DB.FLAVOUR'] ?= 'MYSQL'
        switch options.install['XAAUDIT.DB.FLAVOUR']
          when 'MYSQL'
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


## Ambari Config - Knox Plugin Audit

        options.configurations ?= {}
        options.configurations['ranger-knox-audit'] ?= {}
        options.configurations['ranger-knox-audit']['xasecure.audit.is.enabled'] ?= 'true'
        # audit to hdfs
        options.configurations['ranger-knox-audit']['xasecure.audit.destination.hdfs'] ?= options.install['XAAUDIT.HDFS.IS_ENABLED']
        options.configurations['ranger-knox-audit']['xasecure.audit.destination.hdfs.batch.filespool.dir'] ?= options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR']
        options.configurations['ranger-knox-audit']['xasecure.audit.destination.hdfs.dir'] ?= options.install['XAAUDIT.HDFS.HDFS_DIR']
        # audit to solr
        options.configurations['ranger-knox-audit']['xasecure.audit.destination.solr'] ?= options.install['XAAUDIT.SOLR.IS_ENABLED']
        options.configurations['ranger-knox-audit']['xasecure.audit.destination.solr.batch.filespool.dir'] ?= options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR']
        options.configurations['ranger-knox-audit']['xasecure.audit.destination.solr.zookeepers'] ?= options.install['XAAUDIT.SOLR.ZOOKEEPER']
        options.configurations['ranger-knox-audit']['xasecure.audit.solr.solr_url'] ?= options.install['XAAUDIT.SOLR.URL']
        options.configurations['ranger-knox-audit']['xasecure.audit.jaas.inmemory.loginModuleName'] ?= 'com.sun.security.auth.module.Krb5LoginModule'
        options.configurations['ranger-knox-audit']['xasecure.audit.jaas.inmemory.loginModuleControlFlag'] ?= 'required'
        options.configurations['ranger-knox-audit']['xasecure.audit.jaas.inmemory.Client.option.useKeyTab'] ?= 'true'
        options.configurations['ranger-knox-audit']['xasecure.audit.jaas.inmemory.Client.option.debug'] ?= 'true'
        options.configurations['ranger-knox-audit']['xasecure.audit.jaas.inmemory.Client.option.doNotPrompt'] ?= 'yes'
        options.configurations['ranger-knox-audit']['xasecure.audit.jaas.inmemory.Client.option.storeKey'] ?= 'yes'
        options.configurations['ranger-knox-audit']['xasecure.audit.jaas.inmemory.Client.option.serviceName'] ?= 'solr'
        options.configurations['ranger-knox-audit']['xasecure.audit.jaas.inmemory.Client.option.principal'] ?= service.deps.knox_server.options.krb5_user.principal
        options.configurations['ranger-knox-audit']['xasecure.audit.jaas.inmemory.Client.option.keyTab'] ?= service.deps.knox_server.options.krb5_user.keytab

## Knox Plugin SSL

      if service.deps.ranger_admin.options.site['ranger.service.https.attrib.ssl.enabled'] is 'true'
        # options.ssl = merge {}, service.deps.hadoop_core.options.ssl, options.ssl
        # options.install['SSL_KEYSTORE_FILE_PATH'] ?= service.deps.knox.options.ssl.keystore.target
        # options.install['SSL_KEYSTORE_PASSWORD'] ?= service.deps.knox.options.ssl.keystore.password
        # options.install['SSL_TRUSTSTORE_FILE_PATH'] ?= service.deps.hadoop_core.options.ssl_client['ssl.client.truststore.location']
        # options.install['SSL_TRUSTSTORE_PASSWORD'] ?= service.deps.hadoop_core.options.ssl_client['ssl.client.truststore.password']


## Ambari Config - Knox Plugin SSL
SSL can be configured to use SSL if ranger admin has SSL enabled.
 
        options.ssl = merge {}, service.deps.ssl?.options, service.deps.hadoop_core.options.ssl, options.ssl
        options.configurations['ranger-knox-policymgr-ssl'] ?= {}
        if service.deps.ranger_admin.options.site['ranger.service.https.attrib.ssl.enabled'] is 'true'
          options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.keystore'] ?= "#{service.deps.hadoop_core.options.ssl.conf_dir}/ranger-knox-gateway-plugin-keystore"
          options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password'] ?= service.deps.hadoop_core.options.ssl_server['ssl.server.keystore.password']
          options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.truststore'] ?= "#{service.deps.hadoop_core.options.ssl.conf_dir}/ranger-knox-gateway-plugin-truststore"
          options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password'] ?= service.deps.hadoop_core.options.ssl_server['ssl.server.truststore.password']
          options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.credential.file'] ?= "jceks://file/etc/ranger/#{options.service_repo.name}/cred.jceks"
          options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.credential.file'] ?=  "jceks://file/etc/ranger/#{options.service_repo.name}/cred.jceks"

## Ambari Config - Knox Plugin Properties

        options.configurations['ranger-knox-plugin-properties'] ?= {}
        options.configurations['ranger-knox-plugin-properties']['ranger-knox-plugin-enabled'] ?= 'Yes' 
        options.configurations['ranger-knox-plugin-properties']['REPOSITORY_CONFIG_USERNAME'] ?= options.service_repo.configs.username
        options.configurations['ranger-knox-plugin-properties']['REPOSITORY_CONFIG_PASSWORD'] ?= options.service_repo.configs.password
        options.configurations['ranger-knox-plugin-properties']['common.name.for.certificate'] ?= options.service_repo.configs['commonNameForCertificate']
        options.configurations['ranger-knox-plugin-properties']['hadoop.rpc.protection'] ?= options.service_repo.configs['hadoop.rpc.protection']
        options.configurations['ranger-knox-plugin-properties']['policy_user'] ?= options.service_repo.configs['policy.download.auth.users']
        options.configurations['ranger-knox-plugin-properties']['knox.url'] ?= options.service_repo.configs['knox.url']
        for k, v of options.install
          if k.indexOf('XAAUDIT') isnt -1
            options.configurations['ranger-knox-plugin-properties'][k] ?= v

## Ambari Config - Knox Plugin Security

        options.configurations['ranger-knox-security'] ?= {}
        options.configurations['ranger-knox-security']['ranger.plugin.knox.service.name'] ?= options.service_repo.name
        options.configurations['ranger-knox-security']['ranger.plugin.knox.policy.rest.url'] ?= options.install['POLICY_MGR_URL']
        options.configurations['ranger-knox-security']['ranger.plugin.knox.policy.cache.dir'] ?= "/etc/ranger/#{options.service_repo.name}/policycache"
        options.configurations['ranger-knox-security']['ranger.plugin.knox.policy.pollIntervalMs'] ?= "30000"
        options.configurations['ranger-knox-security']['ranger.plugin.knox.policy.rest.ssl.config.file'] ?= '/usr/hdp/current/knox-server/conf/ranger-policymgr-ssl.xml'
        options.configurations['ranger-knox-security']['ranger.plugin.knox.policy.source.impl'] ?= 'org.apache.ranger.admin.client.RangerAdminRESTClient'

## Ambari Config REST Api

        #ambari server configuration
        options.post_component = service.instances[0].node.fqdn is service.node.fqdn
        options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
        options.ambari_url ?= service.deps.ambari_server.options.ambari_url
        options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
        options.cluster_name ?= service.deps.ambari_server.options.cluster_name

## Enable Plugin in Ranger Admin

        service.deps.ranger_admin.options.configurations['ranger-env']['ranger-knox-plugin-enabled'] = 'Yes'

## Enrich Knox Service  with Ranger Properties
For Knox, ranger related properties should be posted before any service is installed or
started, as Ambari required to configuration dictionnaries to exist `ranger-knox-plugin-properties` 
        
        for srv in service.deps.knox
          srv.options.configurations ?= {}
          srv.options.configurations['ranger-knox-security'] ?= merge {}, srv.options.configurations['ranger-knox-security'], options.configurations['ranger-knox-security']
          srv.options.configurations['ranger-knox-plugin-properties'] ?= merge {}, srv.options.configurations['ranger-knox-plugin-properties'], options.configurations['ranger-knox-plugin-properties']
          srv.options.configurations['ranger-knox-policymgr-ssl'] ?= merge {}, srv.options.configurations['ranger-knox-policymgr-ssl'], options.configurations['ranger-knox-policymgr-ssl']
          srv.options.configurations['ranger-knox-audit'] ?= merge {}, srv.options.configurations['ranger-knox-audit'], options.configurations['ranger-knox-audit']


## Wait

      options.wait_ranger_admin = service.deps.ranger_admin.options.wait

## Dependencies

    {merge} = require 'nikita/lib/misc'
