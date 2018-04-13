
# Ranger Kafka Plugin Configure

    module.exports = (service) ->
      options = service.options

## Identities

      options.group = merge {}, service.deps.ranger_admin.options.group, options.group or {}
      options.user = merge {}, service.deps.ranger_admin.options.user, options.user or {}
      options.kafka_user = service.deps.kafka_broker.options.user
      options.kafka_group = service.deps.kafka_broker.options.group
      options.hadoop_group = service.deps.hadoop_core.options.hadoop_group
      options.hdfs_krb5_user = service.deps.hadoop_core.options.hdfs.krb5_user

## Kerberos

      options.krb5 ?= {}
      options.krb5.enabled ?= service.deps.hadoop_core.options.core_site['hadoop.security.authentication'] is 'kerberos'
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin = service.deps.krb5_client.options.admin[options.krb5.realm]

## Environment

      # Layout
      options.conf_dir ?= service.deps.kafka_broker.options.conf_dir

## Access

      options.ranger_admin ?= service.deps.ranger_admin.options.admin
      # need hdfs plugin to create policy for audit logs (need when the nofallback policy is adopted)
      options.hdfs_install ?= service.deps.ranger_hdfs[0].options.install


## Register Authentication

      service.deps.kafka_broker.options.config['authorizer.class.name'] = 'org.apache.ranger.authorization.kafka.authorizer.RangerKafkaAuthorizer'
      for srv in service.deps.kafka_service
        srv.options.configurations ?= {}
        srv.options.configurations['kafka-broker'] ?= {}
        srv.options.configurations['kafka-broker']['authorizer.class.name'] = 'org.apache.ranger.authorization.kafka.authorizer.RangerKafkaAuthorizer'
        
## Plugin User

      options.plugin_user ?=
        'name': options.kafka_user.name
        'firstName': ''
        'lastName': ''
        'emailAddress': ''
        'userSource': 1
        'userRoleList': ['ROLE_USER']
        'groups': []
        'status': 1
      if 'PLAINTEXT' in service.deps.kafka_broker.options.protocols or 'SSL' in service.deps.kafka_broker.options.protocols
        options.plugin_user_anonymous ?=
          name: "ANONYMOUS"
          firstName: ''
          lastName: ''
          emailAddress: ''
          userSource: 1
          userRoleList: ['ROLE_USER']
          groups: []
          status: 1

## Configuration

      options.install ?= {}
      options.install['PYTHON_COMMAND_INVOKER'] ?= 'python'
      options.install['CUSTOM_USER'] ?= "#{service.deps.kafka_broker.options.user.name}"

## Ranger admin properties

The repository name should match the reposity name in web ui.
The properties can be found [here][kafka-repository]

      options.install['POLICY_MGR_URL'] ?= service.deps.ranger_admin.options.install['policymgr_external_url']
      options.install['REPOSITORY_NAME'] ?= 'hadoop-ryba-kafka'

## Service Definition

      options.service_repo ?=
        'name': options.install['REPOSITORY_NAME']
        'description': 'Kafka Repository'
        'type': 'kafka'
        'isEnabled': true
        'configs':
          'username': service.deps.ranger_admin.options.plugins.principal
          'password': service.deps.ranger_admin.options.plugins.password
          'hadoop.security.authentication': service.deps.hadoop_core.options.core_site['hadoop.security.authentication']
          'zookeeper.connect': service.deps.kafka_broker.options.config['zookeeper.connect'].join(',')
          'policy.download.auth.users': "#{service.deps.kafka_broker.options.user.name}" #from ranger 0.6
          'commonNameForCertificate': ''

## SSL

Used only if SSL is enabled between Policy Admin Tool and Plugin. The path to
keystore is derived from Kafka server. The path to the truststore is derived
from Hadoop Core.

      if service.deps.ranger_admin.options.site['ranger.service.https.attrib.ssl.enabled'] is 'true'
        options.install['SSL_KEYSTORE_FILE_PATH'] ?= service.deps.kafka_broker.options.config['ssl.keystore.location']
        options.install['SSL_KEYSTORE_PASSWORD'] ?= service.deps.kafka_broker.options.config['ssl.keystore.password']
        options.install['SSL_KEY_PASSWORD'] ?= service.deps.kafka_broker.options.config['ssl.key.password']
        options.install['SSL_TRUSTSTORE_FILE_PATH'] ?= service.deps.kafka_broker.options.config['ssl.truststore.location']
        options.install['SSL_TRUSTSTORE_PASSWORD'] ?= service.deps.kafka_broker.options.config['ssl.truststore.password']

## Audit

      options.install['XAAUDIT.SUMMARY.ENABLE'] ?= 'true'

## HDFS storage

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
        options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR'] ?= "#{service.deps.kafka_broker.options.log_dir}/audit/hdfs/spool"
        options.install['XAAUDIT.HDFS.DESTINATION_FILE'] ?= '%hostname%-audit.log'
        options.install['XAAUDIT.HDFS.DESTINATION_FLUSH_INTERVAL_SECONDS'] ?= '900'
        options.install['XAAUDIT.HDFS.DESTINATION_ROLLOVER_INTERVAL_SECONDS'] ?= '86400'
        options.install['XAAUDIT.HDFS.DESTINATION _OPEN_RETRY_INTERVAL_SECONDS'] ?= '60'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_FILE'] ?= '%time:yyyyMMdd-HHmm.ss%.log'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_FLUSH_INTERVAL_SECONDS'] ?= '60'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_ROLLOVER_INTERVAL_SECONDS'] ?= '600'
        options.install['XAAUDIT.HDFS.LOCAL_ARCHIVE _MAX_FILE_COUNT'] ?= '5'
        options.policy_hdfs_audit ?=
          'name': "kafka-ranger-plugin-audit"
          'service': "#{options.hdfs_install['REPOSITORY_NAME']}"
          'repositoryType':"hdfs"
          'description': 'Kafka Ranger Plugin audit log policy'
          'isEnabled': true
          'isAuditEnabled': true
          'resources':
            'path':
              'isRecursive': 'true'
              'values': ['/ranger/audit/kafka']
              'isExcludes': false
          'policyItems': [
            'users': ["#{options.kafka_user.name}"]
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

## Solr storage

      if service.deps.ranger_admin.options.install['audit_store'] is 'solr'
        options.install['XAAUDIT.SOLR.IS_ENABLED'] ?= 'true'
        options.install['XAAUDIT.SOLR.ENABLE'] ?= 'true'
        options.install['XAAUDIT.SOLR.URL'] ?= service.deps.ranger_admin.options.install['audit_solr_urls']
        options.install['XAAUDIT.SOLR.USER'] ?= service.deps.ranger_admin.options.install['audit_solr_user']
        options.install['XAAUDIT.SOLR.ZOOKEEPER'] ?= service.deps.ranger_admin.options.install['audit_solr_zookeepers']
        options.install['XAAUDIT.SOLR.PASSWORD'] ?= service.deps.ranger_admin.options.install['audit_solr_password']
        options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR'] ?= "#{service.deps.kafka_broker.options.log_dir}/audit/solr/spool"

## Database storage

      #Deprecated
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


## Ambari Config - Hive Plugin Audit

        options.configurations ?= {}
        options.configurations['ranger-kafka-audit'] ?= {}
        options.configurations['ranger-kafka-audit']['xasecure.audit.is.enabled'] ?= 'true'
        # audit to hdfs
        options.configurations['ranger-kafka-audit']['xasecure.audit.destination.hdfs'] ?= options.install['XAAUDIT.HDFS.IS_ENABLED']
        options.configurations['ranger-kafka-audit']['xasecure.audit.destination.hdfs.batch.filespool.dir'] ?= options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR']
        options.configurations['ranger-kafka-audit']['xasecure.audit.destination.hdfs.dir'] ?= options.install['XAAUDIT.HDFS.HDFS_DIR']
        # audit to solr
        options.configurations['ranger-kafka-audit']['xasecure.audit.destination.solr'] ?= options.install['XAAUDIT.SOLR.IS_ENABLED']
        options.configurations['ranger-kafka-audit']['xasecure.audit.destination.solr.batch.filespool.dir'] ?= options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR']
        options.configurations['ranger-kafka-audit']['xasecure.audit.destination.solr.zookeepers'] ?= options.install['XAAUDIT.SOLR.ZOOKEEPER']
        options.configurations['ranger-kafka-audit']['xasecure.audit.solr.solr_url'] ?= options.install['XAAUDIT.SOLR.URL']
        options.configurations['ranger-kafka-audit']['xasecure.audit.jaas.inmemory.loginModuleName'] ?= 'com.sun.security.auth.module.Krb5LoginModule'
        options.configurations['ranger-kafka-audit']['xasecure.audit.jaas.inmemory.loginModuleControlFlag'] ?= 'required'
        options.configurations['ranger-kafka-audit']['xasecure.audit.jaas.inmemory.Client.option.useKeyTab'] ?= 'true'
        options.configurations['ranger-kafka-audit']['xasecure.audit.jaas.inmemory.Client.option.debug'] ?= 'true'
        options.configurations['ranger-kafka-audit']['xasecure.audit.jaas.inmemory.Client.option.doNotPrompt'] ?= 'yes'
        options.configurations['ranger-kafka-audit']['xasecure.audit.jaas.inmemory.Client.option.storeKey'] ?= 'yes'
        options.configurations['ranger-kafka-audit']['xasecure.audit.jaas.inmemory.Client.option.serviceName'] ?= 'solr'
        options.configurations['ranger-kafka-audit']['xasecure.audit.jaas.inmemory.Client.option.keyTab'] ?= service.deps.kafka_broker.options.kerberos['principal']
        options.configurations['ranger-kafka-audit']['xasecure.audit.jaas.inmemory.Client.option.principal'] ?= service.deps.kafka_broker.options.kerberos['keyTab']

## Ambari Config - Kafka Plugin SSL
SSL can be configured to use SSL if ranger admin has SSL enabled.
 
        options.ssl = merge {}, service.deps.ssl?.options, options.ssl
        options.configurations['ranger-kafka-policymgr-ssl'] ?= {}
        if service.deps.ranger_admin.options.site['ranger.service.https.attrib.ssl.enabled'] is 'true'
          options.configurations['ranger-kafka-policymgr-ssl']['xasecure.policymgr.clientssl.keystore'] ?= "#{service.deps.hadoop_core.options.ssl.conf_dir}/ranger-kafka-plugin-keystore"
          options.configurations['ranger-kafka-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password'] ?= service.deps.hadoop_core.options.ssl_server['ssl.server.keystore.password']
          options.configurations['ranger-kafka-policymgr-ssl']['xasecure.policymgr.clientssl.truststore'] ?= "#{service.deps.hadoop_core.options.ssl.conf_dir}/ranger-kafka-plugin-truststore"
          options.configurations['ranger-kafka-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password'] ?= service.deps.hadoop_core.options.ssl_server['ssl.server.truststore.password']
          options.configurations['ranger-kafka-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.credential.file'] ?= "jceks://file/etc/ranger/#{options.service_repo.name}/cred.jceks"
          options.configurations['ranger-kafka-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.credential.file'] ?=  "jceks://file/etc/ranger/#{options.service_repo.name}/cred.jceks"

## Ambari Config - Hive Plugin Properties

        options.configurations['ranger-kafka-plugin-properties'] ?= {}
        options.configurations['ranger-kafka-plugin-properties']['ranger-kafka-plugin-enabled'] ?= 'Yes' 
        options.configurations['ranger-kafka-plugin-properties']['REPOSITORY_CONFIG_USERNAME'] ?= options.service_repo.configs.username
        options.configurations['ranger-kafka-plugin-properties']['REPOSITORY_CONFIG_PASSWORD'] ?= options.service_repo.configs.password
        options.configurations['ranger-kafka-plugin-properties']['common.name.for.certificate'] ?= options.service_repo.configs['commonNameForCertificate']
        options.configurations['ranger-kafka-plugin-properties']['hadoop.rpc.protection'] ?= options.service_repo.configs['hadoop.rpc.protection']
        options.configurations['ranger-kafka-plugin-properties']['policy_user'] ?= options.service_repo.configs['policy.download.auth.users']
        options.configurations['ranger-kafka-plugin-properties']['zookeeper.connect'] ?= options.service_repo.configs['zookeeper.connect']
        for k, v of options.install
          if k.indexOf('XAAUDIT') isnt -1
            options.configurations['ranger-kafka-plugin-properties'][k] ?= v

## Ambari Config - Hive Plugin Security

        options.configurations['ranger-kafka-security'] ?= {}
        options.configurations['ranger-kafka-security']['ranger.plugin.kafka.service.name'] ?= options.service_repo.name
        options.configurations['ranger-kafka-security']['ranger.plugin.kafka.policy.rest.url'] ?= options.install['POLICY_MGR_URL']
        options.configurations['ranger-kafka-security']['ranger.plugin.kafka.policy.cache.dir'] ?= "/etc/ranger/#{options.service_repo.name}/policycache"
        options.configurations['ranger-kafka-security']['ranger.plugin.kafka.policy.pollIntervalMs'] ?= "30000"
        options.configurations['ranger-kafka-security']['ranger.plugin.kafka.policy.rest.ssl.config.file'] ?= "#{options.conf_dir}/conf/ranger-policymgr-ssl.xml"
        options.configurations['ranger-kafka-security']['ranger.plugin.kafka.policy.source.impl'] ?= 'org.apache.ranger.admin.client.RangerAdminRESTClient'

## Ambari Config REST Api

        #ambari server configuration
        options.post_component = service.instances[0].node.fqdn is service.node.fqdn
        options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
        options.ambari_url ?= service.deps.ambari_server.options.ambari_url
        options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
        options.cluster_name ?= service.deps.ambari_server.options.cluster_name
        options.takeover = service.deps.ambari_server.options.takeover

## Enable Plugin in Ranger Admin

        service.deps.ranger_admin.options.configurations['ranger-env']['ranger-kafka-plugin-enabled'] = 'Yes'

## Enrich Kafka Service  with Ranger Properties
For Kafka, ranger related properties should be posted before any service is installed or
started, as Ambari required to configuration dictionnaries to exist `ranger-kafka-plugin-properties` 
        
        for srv in service.deps.kafka_service
          srv.options.configurations ?= {}
          srv.options.configurations['ranger-kafka-security'] ?= merge {}, srv.options.configurations['ranger-kafka-security'], options.configurations['ranger-kafka-security']
          srv.options.configurations['ranger-kafka-plugin-properties'] ?= merge {}, srv.options.configurations['ranger-kafka-plugin-properties'], options.configurations['ranger-kafka-plugin-properties']
          srv.options.configurations['ranger-kafka-policymgr-ssl'] ?= merge {}, srv.options.configurations['ranger-kafka-policymgr-ssl'], options.configurations['ranger-kafka-policymgr-ssl']
          srv.options.configurations['ranger-kafka-audit'] ?= merge {}, srv.options.configurations['ranger-kafka-audit'], options.configurations['ranger-kafka-audit']

## Wait

      options.wait_ranger_admin = service.deps.ranger_admin.options.wait

## Dependencies

    {merge} = require 'nikita/lib/misc'
