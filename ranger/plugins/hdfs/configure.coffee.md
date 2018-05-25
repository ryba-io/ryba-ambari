
# Ranger HDFS Plugin Configure

For the HDFS plugin, the executed script already create the hdfs user to ranger admin
as external.

    module.exports = (service) ->
      options = service.options

## Identities

      options.group = merge {}, service.deps.ranger_admin.options.group, options.group or {}
      options.user = merge {}, service.deps.ranger_admin.options.user, options.user or {}
      options.hdfs_user = service.deps.hdfs_nn.options.user
      options.hdfs_group = service.deps.hdfs_nn.options.group
      options.hadoop_group = service.deps.hdfs_nn.options.hadoop_group

## Environment

      for srv in service.deps.hdfs
        srv.options.configurations['hdfs-site']['dfs.namenode.inode.attributes.provider.class'] ?= 'org.apache.ranger.authorization.hadoop.RangerHdfsAuthorizer'
      options.hdfs_conf_dir = service.deps.hdfs_nn.options.conf_dir

## Kerberos

      options.krb5 ?= {}
      options.krb5.enabled ?= service.deps.hadoop_core.options.core_site['hadoop.security.authentication'] is 'kerberos'
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin = service.deps.krb5_client.options.admin[options.krb5.realm]

## Access

      options.ranger_admin ?= service.deps.ranger_admin.options.admin
      # Wait for [#95](https://github.com/ryba-io/ryba/issues/95) to be answered
      # options.plugins ?= {}
      # options.plugins.principal ?= service.deps.ranger_admin.options.plugins.principal
      # options.plugins.password ?= service.deps.ranger_admin.options.plugins.password

## Setup

Repository creating is only executed from one NameNode.

      options.repo_create = service.deps.hdfs_nn.options.active_nn_host is service.node.fqdn

## Configuration

      options.install ?= {}
      options.install['PYTHON_COMMAND_INVOKER'] ?= 'python'

## Admin properties

      options.install['POLICY_MGR_URL'] ?= service.deps.ranger_admin.options.install['policymgr_external_url']
      options.install['REPOSITORY_NAME'] ?= 'hadoop-ryba-hdfs'

## Plugin User

      options.plugin_user =
        "name": options.hdfs_user.name
        "firstName": ''
        "lastName": ''
        "emailAddress": ''
        "password": ''
        'userSource': 1
        'userRoleList': ['ROLE_USER']
        'groups': []
        'status': 1

## Service Definition

      options.service_repo ?=
        'name': options.install['REPOSITORY_NAME']
        'description': 'HDFS Repo'
        'type': 'hdfs'
        'isEnabled': true
        'configs':
          # 'username': 'ranger_plugin_hdfs'
          # 'password': 'RangerPluginHDFS123!'
          'username': service.deps.hdfs[0].options.hdfs.krb5_user.principal
          'password': service.deps.hdfs[0].options.hdfs.krb5_user.password
          'fs.default.name': service.deps.hdfs_nn.options.core_site['fs.defaultFS']
          'hadoop.security.authentication': service.deps.hdfs_nn.options.core_site['hadoop.security.authentication']
          'dfs.namenode.kerberos.principal': service.deps.hdfs_nn.options.hdfs_site['dfs.namenode.kerberos.principal']
          'dfs.datanode.kerberos.principal': service.deps.hdfs_dn[0].options.hdfs_site['dfs.datanode.kerberos.principal']
          'hadoop.rpc.protection': service.deps.hdfs_nn.options.core_site['hadoop.rpc.protection']
          'hadoop.security.authorization': service.deps.hdfs_nn.options.core_site['hadoop.security.authorization']
          'hadoop.security.auth_to_local': service.deps.hdfs_nn.options.core_site['hadoop.security.auth_to_local']
          'commonNameForCertificate': ''
          'policy.download.auth.users': "#{service.deps.hdfs_nn.options.user.name}" #from ranger 0.6
          'tag.download.auth.users': "#{service.deps.hdfs_nn.options.user.name}"

## Audit Storage

      options.audit ?= {}

### Database Storage

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
          # This properties are needed even if they are not user
          # We set it to NONE to let the script execute
          options.install['XAAUDIT.DB.HOSTNAME'] ?= 'NONE'
          options.install['XAAUDIT.DB.DATABASE_NAME'] ?= 'NONE'
          options.install['XAAUDIT.DB.USER_NAME'] ?= 'NONE'
          options.install['XAAUDIT.DB.PASSWORD'] ?= 'NONE'

### HDFS Storage

      options.install['XAAUDIT.HDFS.IS_ENABLED'] ?= 'true'
      if options.install['XAAUDIT.HDFS.IS_ENABLED'] is 'true'
        # migration: lucasbak 11102017
        # honored but not used by plugin
        # options.install['XAAUDIT.HDFS.LOCAL_BUFFER_DIRECTORY'] ?= "#{service.deps.ranger_admin.options.conf_dir}/%app-type%/audit"
        # options.install['XAAUDIT.HDFS.LOCAL_ARCHIVE_DIRECTORY'] ?= "#{service.deps.ranger_admin.options.conf_dir}/%app-type%/archive"
        options.install['XAAUDIT.HDFS.ENABLE'] ?= 'true'
        options.install['XAAUDIT.HDFS.DESTINATION_DIRECTORY'] ?= "#{service.deps.hdfs_nn.options.core_site['fs.defaultFS']}/#{service.deps.ranger_admin.options.user.name}/audit/%app-type%/%time:yyyyMMdd%"
        options.install['XAAUDIT.HDFS.HDFS_DIR'] ?= "#{service.deps.hdfs_nn.options.core_site['fs.defaultFS']}/#{service.deps.ranger_admin.options.user.name}/audit"
        options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR'] ?= "#{service.deps.hdfs_nn.options.log_dir}/audit/hdfs/spool"
        options.install['XAAUDIT.HDFS.DESTINATION_FILE'] ?= '%hostname%-audit.log'
        options.install['XAAUDIT.HDFS.DESTINATION_FLUSH_INTERVAL_SECONDS'] ?= '900'
        options.install['XAAUDIT.HDFS.DESTINATION_ROLLOVER_INTERVAL_SECONDS'] ?= '86400'
        options.install['XAAUDIT.HDFS.DESTINATION _OPEN_RETRY_INTERVAL_SECONDS'] ?= '60'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_FILE'] ?= '%time:yyyyMMdd-HHmm.ss%.log'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_FLUSH_INTERVAL_SECONDS'] ?= '60'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_ROLLOVER_INTERVAL_SECONDS'] ?= '600'
        options.install['XAAUDIT.HDFS.LOCAL_ARCHIVE _MAX_FILE_COUNT'] ?= '5'
        options.policy_hdfs_audit ?=
          'name': "hdfs-ranger-plugin-audit"
          'service': "#{options.install['REPOSITORY_NAME']}"
          'repositoryType':"hdfs"
          'description': 'HDFS Ranger Plugin audit log policy'
          'isEnabled': true
          'isAuditEnabled': true
          'resources':
            'path':
              'isRecursive': 'true'
              'values': [options.install['XAAUDIT.HDFS.HDFS_DIR']]
              'isExcludes': false
          'policyItems': [
            'users': ["#{options.hdfs_user.name}"]
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

### Solr Storage

      if service.deps.ranger_admin.options.install['audit_store'] is 'solr'
        options.install['XAAUDIT.SOLR.IS_ENABLED'] ?= 'true'
        options.install['XAAUDIT.SOLR.ENABLE'] ?= 'true'
        options.install['XAAUDIT.SOLR.URL'] ?= service.deps.ranger_admin.options.install['audit_solr_urls']
        options.install['XAAUDIT.SOLR.USER'] ?= service.deps.ranger_admin.options.install['audit_solr_user']
        options.install['XAAUDIT.SOLR.ZOOKEEPER'] ?= service.deps.ranger_admin.options.install['audit_solr_zookeepers']
        options.install['XAAUDIT.SOLR.PASSWORD'] ?= service.deps.ranger_admin.options.install['audit_solr_password']
        options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR'] ?= "#{service.deps.hdfs_nn.options.log_dir}/audit/solr/spool"
        
## Ambari Configuration

        options.configurations ?= {}
        options.configurations['ranger-hdfs-security'] ?= {}
        options.configurations['ranger-hdfs-security']['ranger.plugin.hdfs.service.name'] ?= options.service_repo.name
        options.configurations['ranger-hdfs-security']['ranger.plugin.hdfs.policy.rest.url'] ?= options.install['POLICY_MGR_URL']
        options.configurations['ranger-hdfs-security']['ranger.plugin.hdfs.policy.cache.dir'] ?= "/etc/ranger/#{options.service_repo.name}/policycache"
        options.configurations['ranger-hdfs-security']['ranger.plugin.hdfs.policy.pollIntervalMs'] ?= "30000"
        options.configurations['ranger-hdfs-security']['ranger.plugin.hdfs.policy.rest.ssl.config.file'] ?= "#{service.deps.hdfs_nn.options.hadoop_conf_dir}/ranger-policymgr-ssl.xml"
        options.configurations['ranger-hdfs-security']['ranger.plugin.hdfs.policy.source.impl'] ?= 'org.apache.ranger.admin.client.RangerAdminRESTClient'
        options.configurations['ranger-hdfs-security']['xasecure.add-hadoop-authorization'] ?= 'true'

        options.configurations['ranger-hdfs-plugin-properties'] ?= {}
        # options.configurations['ranger-hdfs-plugin-properties'] ?= merge {}, options.service_repo.configs,
        #   options.install, options.configurations['ranger-hdfs-plugin-properties']
        options.configurations['ranger-hdfs-plugin-properties']['ranger-hdfs-plugin-enabled'] ?= 'Yes' 
        options.configurations['ranger-hdfs-plugin-properties']['REPOSITORY_CONFIG_USERNAME'] ?= options.service_repo.configs.username
        options.configurations['ranger-hdfs-plugin-properties']['REPOSITORY_CONFIG_PASSWORD'] ?= options.service_repo.configs.password
        options.configurations['ranger-hdfs-plugin-properties']['common.name.for.certificate'] ?= options.service_repo.configs['commonNameForCertificate']
        options.configurations['ranger-hdfs-plugin-properties']['hadoop.rpc.protection'] ?= options.service_repo.configs['hadoop.rpc.protection']
        options.configurations['ranger-hdfs-plugin-properties']['policy_user'] ?= options.service_repo.configs['policy.download.auth.users']
        for k, v of options.install
          if k.indexOf('XAAUDIT') isnt -1
            options.configurations['ranger-hdfs-plugin-properties'][k] ?= v

## Plugin to Ranger admin SSL
configure `policy-mgr-ssl` ambari configuration to make the plugin communicate overs tls with ranger-admin
 
        options.configurations['ranger-hdfs-policymgr-ssl'] ?= {}
        if service.deps.ranger_admin.options.site['ranger.service.https.attrib.ssl.enabled'] is 'true'
          options.configurations['ranger-hdfs-policymgr-ssl']['xasecure.policymgr.clientssl.keystore'] ?= service.deps.hdfs_nn.options.ssl_server['ssl.server.keystore.location']
          options.configurations['ranger-hdfs-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password'] ?= service.deps.hdfs_nn.options.ssl_server['ssl.server.keystore.password']
          options.configurations['ranger-hdfs-policymgr-ssl']['xasecure.policymgr.clientssl.truststore'] ?= service.deps.hdfs_nn.options.ssl_server['ssl.server.truststore.location']
          options.configurations['ranger-hdfs-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password'] ?= service.deps.hdfs_nn.options.ssl_server['ssl.server.truststore.password']
          options.configurations['ranger-hdfs-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.credential.file'] ?= "jceks://file/etc/ranger/#{options.service_repo.name}/cred.jceks"
          options.configurations['ranger-hdfs-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.credential.file'] ?=  "jceks://file/etc/ranger/#{options.service_repo.name}/cred.jceks"

## Plugin Audit

        options.configurations['ranger-hdfs-audit'] ?= {}
        options.configurations['ranger-hdfs-audit']['xasecure.audit.is.enabled'] ?= 'true'
        # audit to hdfs
        options.configurations['ranger-hdfs-audit']['xasecure.audit.destination.hdfs'] ?= options.install['XAAUDIT.HDFS.IS_ENABLED']
        options.configurations['ranger-hdfs-audit']['xasecure.audit.destination.hdfs.batch.filespool.dir'] ?= options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR']
        options.configurations['ranger-hdfs-audit']['xasecure.audit.destination.hdfs.dir'] ?= options.install['XAAUDIT.HDFS.HDFS_DIR']
        # audit to solr
        options.configurations['ranger-hdfs-audit']['xasecure.audit.destination.solr'] ?= options.install['XAAUDIT.SOLR.IS_ENABLED']
        options.configurations['ranger-hdfs-audit']['xasecure.audit.destination.solr.batch.filespool.dir'] ?= options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR']
        options.configurations['ranger-hdfs-audit']['xasecure.audit.destination.solr.zookeepers'] ?= options.install['XAAUDIT.SOLR.ZOOKEEPER']
        options.configurations['ranger-hdfs-audit']['xasecure.audit.solr.solr_url'] ?= options.install['XAAUDIT.SOLR.URL']
        ## JAAS in memory configuration
        options.configurations['ranger-hdfs-audit']['xasecure.audit.destination.solr.force.use.inmemory.jaas.config'] ?= 'true'
        options.configurations['ranger-hdfs-audit']['xasecure.audit.jaas.inmemory.loginModuleName'] ?= 'com.sun.security.auth.module.Krb5LoginModule'
        options.configurations['ranger-hdfs-audit']['xasecure.audit.jaas.inmemory.loginModuleControlFlag'] ?= 'required'
        options.configurations['ranger-hdfs-audit']['xasecure.audit.jaas.inmemory.Client.option.useKeyTab'] ?= 'true'
        options.configurations['ranger-hdfs-audit']['xasecure.audit.jaas.inmemory.Client.option.debug'] ?= 'true'
        options.configurations['ranger-hdfs-audit']['xasecure.audit.jaas.inmemory.Client.option.doNotPrompt'] ?= 'yes'
        options.configurations['ranger-hdfs-audit']['xasecure.audit.jaas.inmemory.Client.option.storeKey'] ?= 'yes'
        options.configurations['ranger-hdfs-audit']['xasecure.audit.jaas.inmemory.Client.option.serviceName'] ?= 'solr'
        options.configurations['ranger-hdfs-audit']['xasecure.audit.jaas.inmemory.Client.option.keyTab'] ?= service.deps.hdfs_nn.options.hdfs_site['dfs.namenode.keytab.file']
        options.configurations['ranger-hdfs-audit']['xasecure.audit.jaas.inmemory.Client.option.principal'] ?= service.deps.hdfs_nn.options.hdfs_site['dfs.namenode.kerberos.principal']

## Ambari

        #ambari server configuration
        options.post_component = service.instances[0].node.fqdn is service.node.fqdn
        options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
        options.ambari_url ?= service.deps.ambari_server.options.ambari_url
        options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
        options.cluster_name ?= service.deps.ambari_server.options.cluster_name
        options.takeover = service.deps.ambari_server.options.takeover

## Enable Plugin in Ranger Admin

        service.deps.ranger_admin.options.configurations['ranger-env']['ranger-hdfs-plugin-enabled'] = 'Yes'
        
## Enrich HDFS Service  with Ranger Properties
For Hive, ranger related properties should be posted before any service is installed or
started, as Ambari required to configuration dictionnaries to exist `ranger-hdfs-plugin-properties` 
        
        for srv in service.deps.hdfs
          srv.options.configurations ?= {}
          srv.options.configurations['ranger-hdfs-security'] ?= merge {}, srv.options.configurations['ranger-hdfs-security'], options.configurations['ranger-hdfs-security']
          srv.options.configurations['ranger-hdfs-plugin-properties'] ?= merge {}, srv.options.configurations['ranger-hdfs-plugin-properties'], options.configurations['ranger-hdfs-plugin-properties']
          srv.options.configurations['ranger-hdfs-policymgr-ssl'] ?= merge {}, srv.options.configurations['ranger-hdfs-policymgr-ssl'], options.configurations['ranger-hdfs-policymgr-ssl']
          srv.options.configurations['ranger-hdfs-audit'] ?= merge {}, srv.options.configurations['ranger-hdfs-audit'], options.configurations['ranger-hdfs-audit']
## Wait

      options.wait_ranger_admin = service.deps.ranger_admin.options.wait

## Dependencies

    {merge} = require 'nikita/lib/misc'
